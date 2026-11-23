# 7. Sequence diagram – dispatch

How a booking gets a driver. Covers FR-D03, FR-D04, FR-A02 and BR-35.

## Dispatch settings (admin-editable)

| Setting | Starting value |
|---|---|
| First search radius (Express) | 10 km |
| Second search radius (Express) | 20 km |
| Drivers offered per round | Up to 5 nearest |
| Time to accept an Express offer | 60 seconds |
| Rounds before Operations is alerted | 2 |
| Route planning run (Standard and Saver) | Daily at 17:00, for the next day |
| Saver forced dispatch | Day 5 of the 7-day window |

## 7.1 Express – live offers

```mermaid
sequenceDiagram
    autonumber
    participant B as Bookings module
    participant D as Dispatch module
    participant DB as PostgreSQL + PostGIS
    participant WS as WebSocket / push
    actor D1 as Driver 1
    actor D2 as Driver 2
    participant Ops as Admin website<br/>(Operations)

    B-)D: Event BookingConfirmed (Express)
    D->>DB: Find drivers: APPROVED, online, not on a job,<br/>right vehicle, within 10 km (PostGIS), nearest first
    DB-->>D: Driver 1, Driver 2, ...
    D->>DB: Create job offers (OFFERED, expires in 60 s)
    D->>WS: Send offer to each driver
    WS-)D1: New job: payout, distance, items, customer rating
    WS-)D2: New job: payout, distance, items, customer rating

    Note over D1,D2: Drivers may decline customers rated below 3.5 with no penalty (BR-35)

    par Both drivers tap Accept
        D1->>D: Accept offer
    and
        D2->>D: Accept offer
    end

    D->>DB: UPDATE booking SET driver = Driver 1<br/>WHERE status = 'BOOKED' (atomic)
    DB-->>D: 1 row updated ✅
    D->>DB: UPDATE booking SET driver = Driver 2<br/>WHERE status = 'BOOKED'
    DB-->>D: 0 rows updated ❌ (already taken)

    D->>B: assignDriver(Driver 1)
    B->>B: transitionTo(DRIVER_ASSIGNED)
    D->>DB: Driver 1 offer ACCEPTED, all other offers EXPIRED
    D-->>D1: Job confirmed, navigate to pickup
    D-->>D2: Job already taken
    B-)WS: Customer: driver assigned (name, vehicle, rating)

    alt Nobody accepts within 60 s
        D->>DB: Mark offers EXPIRED
        D->>DB: Search again within 20 km
        D->>WS: Second round of offers
        opt Still nobody after round 2
            D-)Ops: Alert: Express booking has no driver
            Ops->>D: Assign a driver manually (see 7.3)
        end
    end
```

## 7.2 Standard and Saver – route packages

```mermaid
sequenceDiagram
    autonumber
    participant Job as Scheduled job<br/>(daily 17:00)
    participant D as Dispatch module
    participant DB as PostgreSQL + PostGIS
    participant WS as Push notifications
    actor Drv as Drivers
    participant Ops as Admin website<br/>(Operations)

    Job->>D: planRoutes(tomorrow)
    D->>DB: Unassigned Standard bookings due tomorrow<br/>+ Saver bookings that fit tomorrow
    DB-->>D: Bookings with pickup / drop-off points and weights

    loop For each area and vehicle type
        D->>D: Group nearby pickups and drop-offs (PostGIS clustering)
        D->>D: Fill vehicle up to max load (1,000 / 1,500 kg)
        D->>D: Order stops (pickups before their drop-offs)
        D->>DB: Save route (PLANNED) with its bookings and payout
    end

    opt Saver booking reaches day 5 without a route
        D->>D: Put it on a route even if the van is not full
    end

    D->>DB: Find approved drivers with the right vehicle near each route's area
    D->>WS: Offer route packages (stops, km, total payout)
    WS-)Drv: New route for tomorrow

    alt A driver accepts
        Drv->>D: Accept route
        D->>DB: Atomic update: route ACCEPTED, driver set (first accept wins)
        D->>DB: Every booking on the route → DRIVER_ASSIGNED
        D-)WS: Customers: driver assigned, delivery date confirmed
    else No driver by 20:00
        D-)Ops: Alert: route not taken
        Ops->>D: Assign manually or split the route
    end
```

## 7.3 Manual assignment by Operations

```mermaid
sequenceDiagram
    autonumber
    actor Op as Operations admin
    participant Web as Admin website
    participant API as Backend API
    participant D as Dispatch module
    participant B as Bookings module
    participant A as Audit log
    actor Drv as Driver

    Op->>Web: Open booking, click "Assign driver"
    Web->>API: GET /api/v1/admin/bookings/{id}/eligible-drivers
    API->>D: Approved drivers, right vehicle, nearest first
    D-->>Web: List of drivers
    Op->>Web: Choose a driver
    Web->>API: POST /api/v1/admin/bookings/{id}/assign
    API->>API: Check role (Operations or Super admin)
    API->>D: assign(booking, driver, by = admin)
    D->>B: transitionTo(DRIVER_ASSIGNED)
    D->>A: Record "JOB_ASSIGNED" (who, when, which driver)
    D-)Drv: Push: new job assigned to you
    D-->>Web: Assigned ✅
```

## Notes

| Design choice | Reason |
|---|---|
| Atomic `UPDATE ... WHERE status = ...` | Prevents two drivers getting the same job, without slow locks |
| Offers expire after 60 s | Customers get a driver fast; unanswered offers don't block the job |
| Routes planned the evening before | Drivers can plan their day; customers get a confirmed date |
| Saver forced on day 5 | Keeps the "within 7 days" promise even when loads don't fill up |
| Every manual assignment audited | Admin actions are traceable (FR-A10) |