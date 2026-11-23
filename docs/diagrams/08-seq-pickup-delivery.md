# 8. Sequence diagram – pickup and delivery

From the driver starting the trip to a completed delivery. Covers BR-04, BR-13, BR-27–BR-31, FR-D05–FR-D13 and NFR-07.

## 8.1 Pickup

```mermaid
sequenceDiagram
    autonumber
    actor Drv as Driver
    participant DApp as Driver app
    participant API as Backend API
    participant B as Bookings module
    participant T as Tracking module
    participant Pay as Payments module
    participant Files as File storage
    participant N as Notifications
    actor C as Customer

    Drv->>DApp: Tap "Start trip"
    DApp->>API: POST /bookings/{id}/start
    API->>B: transitionTo(EN_ROUTE_TO_PICKUP)
    B-)N: Customer: driver on the way
    loop Every 10 seconds while on the job
        DApp->>T: Location (WebSocket)
        T-)C: Driver position on map
    end

    Drv->>DApp: Tap "Arrived"
    DApp->>API: POST /bookings/{id}/arrived-pickup
    API->>B: transitionTo(ARRIVED_AT_PICKUP)
    B->>B: Start waiting timer (15 min free, then R50 per 15 min)
    B-)N: Customer: driver has arrived

    Drv->>DApp: Checklist: items match booking? prohibited items? packed?
    Drv->>DApp: Take item photos
    DApp->>Files: Upload photos (queued on phone if no signal)

    alt Items match the booking
        DApp->>API: POST /bookings/{id}/pickup (photo refs)
        API->>B: transitionTo(PICKED_UP)
        B->>B: Stop waiting timer, add waiting fee if any
        B-)N: Customer: item picked up

    else Items don't match (heavier, bigger or extra items)
        Drv->>DApp: Flag mismatch, enter real items, take photos
        DApp->>API: POST /bookings/{id}/price-change
        API->>B: requestPriceChange(new items)
        B->>B: Recalculate price (same formula and settings)
        B->>B: transitionTo(AWAITING_PRICE_APPROVAL)
        B-)N: Customer: approve new price? (photos + old vs new total)
        N-)C: Push notification
        alt Customer approves
            C->>API: POST /price-changes/{id}/approve
            API->>Pay: Charge the difference (new Idempotency-Key)
            Pay-->>API: Payment succeeded
            API->>B: transitionTo(PICKED_UP)
            B-)N: Driver: approved, load the items
        else Customer rejects, or no answer in 15 minutes
            API->>B: cancel() → fee = 50% of delivery fee (driver arrived, BR-29)
            B->>Pay: Refund the rest
            B->>B: transitionTo(CANCELLED)
            B-)N: Driver: job cancelled, cancellation share paid
        end

    else Prohibited item found
        Drv->>DApp: Refuse item, choose reason, take photos
        DApp->>API: POST /bookings/{id}/refuse
        API->>B: transitionTo(REFUSED)
        B->>Pay: Keep R100 call-out fee, refund the rest (BR-28)
        B->>B: transitionTo(CANCELLED)
        B-)N: Customer: item refused, reason and refund
    end
```

## 8.2 Transit and delivery

```mermaid
sequenceDiagram
    autonumber
    actor Drv as Driver
    participant DApp as Driver app
    participant API as Backend API
    participant B as Bookings module
    participant T as Tracking module
    participant Pay as Payments module
    participant N as Notifications
    actor R as Receiver
    actor C as Customer
    participant Ops as Admin website<br/>(Operations)

    Drv->>DApp: Tap "Leave pickup"
    DApp->>API: POST /bookings/{id}/in-transit
    API->>B: transitionTo(IN_TRANSIT)
    B-)N: Receiver SMS: on the way + tracking link

    loop Every 10 seconds
        DApp->>T: Location (WebSocket)
        T-)C: Live map
        T-)R: Live map (tracking link)
    end

    Drv->>DApp: Tap "Arrived"
    DApp->>API: POST /bookings/{id}/arrived-dropoff
    API->>B: transitionTo(ARRIVED_AT_DROPOFF)
    B->>B: Start 15-minute wait timer
    B-)N: Receiver: driver has arrived

    alt Receiver is there
        R->>Drv: Gives 4-digit PIN
        Drv->>DApp: Enter PIN, take delivery photos
        DApp->>API: POST /bookings/{id}/deliver (PIN, photo refs)
        API->>B: verifyPin(pin)
        alt PIN correct
            B->>B: transitionTo(DELIVERED)
            B-)Pay: Event BookingDelivered
            Pay->>Pay: Record driver earnings<br/>(75% delivery fee, 90% helpers, 100% add-ons)
            B-)N: Customer: delivered + photos + rate and tip
            B-)N: Driver: rate the customer
            T->>T: Stop location tracking for this job (NFR-07)
        else PIN wrong (attempt 1 or 2)
            B->>B: pinAttempts + 1
            B-->>DApp: Wrong PIN, try again
        else PIN wrong 3 times
            B-)Ops: Alert: PIN failed 3 times (BR-31)
            Ops->>C: Call customer to confirm the receiver
        end

    else Receiver not there after 15 minutes
        Drv->>DApp: Record failed delivery + photo of location
        DApp->>API: POST /bookings/{id}/failed
        API->>B: transitionTo(FAILED_DELIVERY)
        B-)N: Customer: delivery failed, choose return or rebook
        C->>API: Choose "Return" or "Rebook"
        API->>Pay: Charge the customer for the extra trip (BR-30)
        alt Return
            API->>B: transitionTo(RETURNED)
        else Rebook
            API->>B: Create new booking linked to this one
        end
    end
```

## Notes

| Design choice | Reason |
|---|---|
| Photos at pickup and delivery, with GPS and phone time | Evidence for damage claims and disputes (BR-04, BR-32) |
| Offline photo queue | Drivers can finish jobs in areas with poor signal (FR-D13) |
| Price recalculated with the same formula | The customer sees a fair, explainable new price (BR-27) |
| 15-minute limit to approve a price change | The driver is not kept waiting forever |
| PIN checked by the backend, never by the app | The app never knows the real PIN, so it can't be bypassed |
| Earnings recorded on `BookingDelivered` | Drivers are only paid for completed jobs; payouts run weekly |
| Tracking stops at delivery | Location is tracked only while on a job (POPIA, NFR-07) |