# 6. Sequence diagram – booking and payment

From getting a quote to a confirmed booking. Covers BR-06–BR-09 (pricing), BR-19–BR-25 (promotions) and BR-15–BR-17 (payment).

## 6.1 Get a quote

```mermaid
sequenceDiagram
    autonumber
    actor C as Customer
    participant App as Customer app
    participant API as Backend API
    participant Q as Pricing module<br/>(QuoteService)
    participant Maps as Google Maps
    participant P as Promotions module
    participant DB as PostgreSQL

    C->>App: Enter pickup, drop-off, items, add-ons
    App->>API: POST /api/v1/quotes
    API->>API: Check JWT token (logged in?)
    API->>Q: createQuote(request)
    Q->>Maps: Driving distance (pickup → drop-off)
    Maps-->>Q: 25.0 km
    Q->>DB: Load pricing settings, catalogue items
    DB-->>Q: Vehicle rates, weight bands, multipliers
    Q->>Q: Chargeable weight = max(actual, L×W×H ÷ 5000), min 5 kg
    Q->>Q: Delivery fee for each speed tier
    Q->>P: Best discount (spend discount, no code yet)
    P-->>Q: R0 (fee below R1,000)
    Q->>Q: Add add-ons, round up to nearest R10
    Q->>DB: Save quote (expires in 15 minutes)
    Q-->>API: Quote with Express / Standard / Saver prices
    API-->>App: 201 Created + price breakdown
    App-->>C: Show prices and suggested vehicle
```

## 6.2 Book and pay

```mermaid
sequenceDiagram
    autonumber
    actor C as Customer
    participant App as Customer app
    participant API as Backend API
    participant P as Promotions module
    participant B as Bookings module
    participant Pay as Payments module
    participant PP as Payment provider<br/>(sandbox)
    participant N as Notifications module
    participant DB as PostgreSQL

    C->>App: Choose Standard, enter promo code "FIRST50"
    App->>API: POST /api/v1/promotions/validate
    API->>P: validate("FIRST50", quote, customer, device)
    P->>DB: Code active? Uses left? First order? Same device or card used before?
    alt Code valid
        P->>P: Compare code discount with spend discount, keep the bigger one
        P-->>API: Discount R50
        API-->>App: New total shown
    else Code invalid or expired
        P-->>API: Rejected + reason
        API-->>App: Show reason, keep old price
    end

    C->>App: Accept item rules, add receiver, tap "Pay"
    App->>API: POST /api/v1/bookings<br/>(quoteId, promo, receiver, Idempotency-Key)
    API->>B: createBooking(...)
    B->>DB: Quote still valid? (not expired, not used)
    alt Quote expired
        B-->>API: 409 Quote expired
        API-->>App: Ask customer to refresh the quote
    else Quote valid
        B->>DB: Save booking (status PENDING_PAYMENT),<br/>items, add-ons, promo redemption
        B->>Pay: startPayment(booking, Idempotency-Key)
        Pay->>DB: Key already used?
        alt Key already used (double tap)
            Pay-->>B: Return the existing payment, no new charge
        else New key
            Pay->>PP: Create checkout (amount, reference)
            PP-->>Pay: Checkout URL
            Pay->>DB: Save payment (PENDING)
        end
        Pay-->>API: Checkout URL
        API-->>App: 201 Created + checkout URL
    end

    App->>PP: Open secure checkout page
    C->>PP: Enter card details
    PP-->>App: Redirect back to app

    PP->>API: Webhook: payment result (signed)
    API->>Pay: handleWebhook(event)
    Pay->>Pay: Verify signature, ignore if already processed
    alt Payment succeeded
        Pay->>DB: Payment SUCCEEDED
        Pay->>B: paymentSucceeded(bookingId)
        B->>B: transitionTo(BOOKED)
        B->>B: Generate 4-digit delivery PIN (store hash only)
        B->>DB: Save status + status history
        B-)N: Event BookingConfirmed
        N-)C: Push + email: booking confirmed, receipt
        N-)C: SMS to receiver: tracking link + PIN
        Note over B: Dispatch starts (see 07-seq-dispatch.md)
    else Payment failed
        Pay->>DB: Payment FAILED
        Pay->>B: paymentFailed(bookingId)
        B->>DB: Keep PENDING_PAYMENT, restore promo code
        N-)C: Push: payment failed, try again
    end

    App->>API: GET /api/v1/bookings/{id}
    API-->>App: Current status (BOOKED)
    App-->>C: Show booking reference and tracking screen
```

## Notes

| Step | Why it matters |
|---|---|
| Quote expires after 15 minutes | Prices and settings may change; the customer always pays a fresh, locked price (BR-16) |
| Promo checked against phone, device and card | Stops first-order promo abuse (BR-25) |
| `Idempotency-Key` header | Double taps or retries never create a second charge (BR-17) |
| Signed webhook | Only the real payment provider can confirm a payment |
| PIN stored as a hash | Even the database never holds the plain PIN (NFR-01) |
| `-)` arrows (async) | Notifications are sent in the background, so a slow SMS never delays the booking |