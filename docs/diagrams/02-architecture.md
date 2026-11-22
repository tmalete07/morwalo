# 2. Architecture (containers)

What's inside the Morwalo platform: three client apps, one modular Spring Boot backend, the data stores and the external services.

```mermaid
flowchart TB
    subgraph clients["Client apps"]
        direction LR
        capp["📱 Customer app<br/><small>React Native (Expo)<br/>Android first</small>"]
        dapp["📱 Driver app<br/><small>React Native (Expo)<br/>Android first, offline photo queue</small>"]
        web["💻 Admin website<br/><small>Next.js<br/>Browser</small>"]
    end

    subgraph backend["Morwalo backend · Java 21 + Spring Boot (modular monolith)"]
        direction TB
        subgraph api["API layer"]
            direction LR
            rest["REST API<br/><small>/api/v1 · JSON over HTTPS</small>"]
            ws["WebSocket<br/><small>Live tracking, job offers</small>"]
            auth["Security<br/><small>JWT tokens, roles, rate limits</small>"]
        end
        subgraph modules["Business modules"]
            direction LR
            identity["Identity<br/><small>Users, OTP, roles</small>"]
            drivers["Drivers<br/><small>Onboarding, documents,<br/>vehicles, helpers</small>"]
            pricing["Pricing<br/><small>Quotes, catalogue,<br/>settings</small>"]
            promos["Promotions<br/><small>Codes, spend discounts</small>"]
            bookings["Bookings<br/><small>Status flow, cancellations</small>"]
            dispatch["Dispatch<br/><small>Offers, Saver routes</small>"]
            tracking["Tracking<br/><small>Live location, PIN, proof</small>"]
            payments["Payments<br/><small>Charges, refunds,<br/>tips, payouts</small>"]
            ratings["Ratings &amp; claims<br/><small>Ratings, disputes, damage</small>"]
            notify["Notifications<br/><small>Push, SMS, email</small>"]
            reporting["Admin &amp; audit<br/><small>Dashboard, reports, audit log</small>"]
        end
        jobs["⏱️ Scheduled jobs<br/><small>Saver route grouping · weekly payouts<br/>document expiry checks</small>"]
    end

    subgraph data["Data"]
        direction LR
        db[("PostgreSQL + PostGIS<br/><small>Main database</small>")]
        files[("File storage (S3-compatible)<br/><small>Photos, driver documents</small>")]
    end

    subgraph external["External services"]
        direction LR
        pay["💳 Payment provider<br/><small>Sandbox</small>"]
        maps["🗺️ Google Maps"]
        sms["✉️ SMS provider<br/><small>Simulated in prototype</small>"]
        fcm["🔔 Firebase Cloud Messaging"]
        mail["📧 Email service"]
    end

    capp -- "HTTPS + WebSocket" --> api
    dapp -- "HTTPS + WebSocket" --> api
    web -- "HTTPS" --> api
    api --> modules
    jobs --> modules
    modules -- "SQL (JPA)" --> db
    modules -- "Upload / signed URLs" --> files
    payments --> pay
    pricing --> maps
    notify --> sms
    notify --> fcm
    notify --> mail

    classDef app fill:#EAF8FB,stroke:#0B7B93,color:#0E2A3B
    classDef core fill:#FFFFFF,stroke:#3EC6E0,color:#0E2A3B
    classDef store fill:#0E2A3B,stroke:#0E2A3B,color:#FFFFFF
    classDef ext fill:#F4F6F7,stroke:#8A9BA5,color:#0E2A3B,stroke-dasharray: 4 3
    classDef job fill:#FDECEC,stroke:#E5484D,color:#0E2A3B

    class capp,dapp,web app
    class rest,ws,auth,identity,drivers,pricing,promos,bookings,dispatch,tracking,payments,ratings,notify,reporting core
    class db,files store
    class pay,maps,sms,fcm,mail ext
    class jobs job
```

## Why a modular monolith?

One deployable Spring Boot app, split into modules with clear boundaries. It is simpler to build, test and deploy for a small team than microservices. Each module owns its own tables and talks to other modules only through their public service interfaces, so any module can later be moved into its own service if Morwalo grows.

## Modules

| Module | Owns (data) | Main responsibilities | Related rules |
|---|---|---|---|
| Identity | users, roles, otp_codes | Sign-up, OTP login, JWT tokens, admin roles | BR-38, NFR-01–03 |
| Drivers | drivers, vehicles, helpers, documents | Onboarding, approval, suspension, document expiry | BR-39, BR-40 |
| Pricing | item_catalogue, pricing_settings, quotes | Volumetric weight, price formula, add-ons | BR-06–BR-14 |
| Promotions | promo_codes, promo_redemptions | Code validation, spend discounts, best-saving rule | BR-19–BR-25 |
| Bookings | bookings, booking_items, status_history | Status flow, cancellations, price changes | BR-27–BR-30 |
| Dispatch | job_offers, routes | Express offers, Saver route grouping, manual assign | FR-D03, FR-D04 |
| Tracking | location_pings, proofs | Live location, photos, delivery PIN | BR-31, NFR-07 |
| Payments | payments, refunds, tips, payouts | Charges, idempotency, splits, weekly payouts | BR-15–BR-18, BR-26 |
| Ratings & claims | ratings, claims, disputes | Two-way ratings, thresholds, damage claims | BR-32–BR-37 |
| Notifications | notifications | Push, SMS and email on status changes | FR-C12 |
| Admin & audit | audit_log | Dashboard, reports, CSV export, audit log | FR-A01, FR-A10 |

## Communication

| From → To | How |
|---|---|
| Apps → backend | REST over HTTPS (JSON), with a JWT token in each request |
| Backend → apps (live) | WebSocket for driver location and job offers; push notifications when the app is closed |
| Module → module | Direct Java service calls for queries; internal events (e.g. `BookingDelivered`) for side effects like notifications and payouts |
| Backend → database | Spring Data JPA; PostGIS for "drivers near this point" |