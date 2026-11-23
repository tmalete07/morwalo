# 5. Domain class diagram

The main classes in the Morwalo backend (Java), showing where each business rule lives.
Database tables are in [04-erd.md](04-erd.md); this diagram is about behaviour.

## 5.1 Domain model

```mermaid
classDiagram
    direction LR

    class User {
        -UUID id
        -String phone
        -String email
        -String fullName
        -LocalDate dateOfBirth
        -UserType userType
        -AccountStatus status
        +isAdult() boolean
        +suspend(String reason) void
    }

    class Customer {
        -UUID id
        -Rating averageRating
        -int ratedJobsCount
        +isNew() boolean
        +isLowRated() boolean
        +needsReview() boolean
    }

    class Driver {
        -UUID id
        -DriverStatus status
        -String licenceCode
        -LocalDate licenceExpiry
        -boolean online
        -GeoPoint currentLocation
        -Rating averageRating
        +goOnline() void
        +goOffline() void
        +canTakeJobs() boolean
        +approve(User admin) void
        +reject(User admin) void
        +suspend(String reason) void
        +canReapply(LocalDate today) boolean
    }

    class Vehicle {
        -UUID id
        -VehicleType type
        -String registrationNumber
        -int gvmKg
        -int maxLoadKg
        -LocalDate roadworthyExpiry
        +canCarry(Weight weight) boolean
        +isRoadworthy(LocalDate today) boolean
    }

    class Helper {
        -UUID id
        -String fullName
        -HelperStatus status
    }

    class Booking {
        -UUID id
        -String reference
        -BookingStatus status
        -SpeedTier speedTier
        -Address pickup
        -Address dropoff
        -Receiver receiver
        -String deliveryPinHash
        -int pinAttempts
        -PriceBreakdown price
        -Money declaredValue
        +transitionTo(BookingStatus next, User by) void
        +cancel(User by) Money
        +cancellationFee() Money
        +verifyPin(String pin) boolean
        +requestPriceChange(PriceBreakdown newPrice) PriceChangeRequest
        +chargeableWeight() Weight
        +isCancellable() boolean
    }

    class BookingItem {
        -UUID id
        -String description
        -int quantity
        -Weight actualWeight
        -Dimensions dimensions
        +volumetricWeight() Weight
        +chargeableWeight() Weight
        +needsTwoHelpers() boolean
    }

    class AddOn {
        -AddOnType type
        -int quantity
        -Money unitPrice
        +total() Money
    }

    class PriceChangeRequest {
        -Money oldTotal
        -Money newTotal
        -String reason
        -RequestStatus status
        +approve() void
        +reject() void
    }

    class Payment {
        -UUID id
        -PaymentType type
        -Money amount
        -String idempotencyKey
        -PaymentStatus status
        +markSucceeded(String providerRef) void
        +markFailed() void
    }

    class Rating {
        -int score
        -List~String~ reasons
        -String comment
        +isValid() boolean
    }

    class Address {
        <<value object>>
        -String line
        -GeoPoint location
        -int floor
        -boolean hasLift
        +floorsToCarry() int
    }

    class Money {
        <<value object>>
        -BigDecimal amount
        +plus(Money other) Money
        +minus(Money other) Money
        +times(BigDecimal factor) Money
        +roundUpToNearest10() Money
    }

    class BookingStatus {
        <<enumeration>>
        QUOTE
        PENDING_PAYMENT
        BOOKED
        DRIVER_ASSIGNED
        EN_ROUTE_TO_PICKUP
        ARRIVED_AT_PICKUP
        AWAITING_PRICE_APPROVAL
        REFUSED
        PICKED_UP
        IN_TRANSIT
        ARRIVED_AT_DROPOFF
        DELIVERED
        FAILED_DELIVERY
        RETURNED
        CANCELLED
        +canMoveTo(BookingStatus next) boolean
    }

    class DriverStatus {
        <<enumeration>>
        APPLIED
        UNDER_REVIEW
        APPROVED
        REJECTED
        SUSPENDED
        DEACTIVATED
    }

    class SpeedTier {
        <<enumeration>>
        EXPRESS
        STANDARD
        SAVER
    }

    class VehicleType {
        <<enumeration>>
        PANEL_VAN
        MINI_TRUCK
    }

    User "1" --> "0..1" Customer : profile
    User "1" --> "0..1" Driver : profile
    Driver "1" *-- "1..*" Vehicle : owns
    Driver "1" *-- "0..*" Helper : works with
    Customer "1" --> "0..*" Booking : places
    Driver "0..1" --> "0..*" Booking : delivers
    Booking "1" *-- "1..*" BookingItem : contains
    Booking "1" *-- "0..*" AddOn : has
    Booking "1" *-- "0..*" PriceChangeRequest : may have
    Booking "1" --> "1..*" Payment : paid by
    Booking "1" --> "0..2" Rating : rated in
    Booking --> Address : pickup / dropoff
    Booking --> BookingStatus
    Booking --> SpeedTier
    Driver --> DriverStatus
    Vehicle --> VehicleType
```

### Where the business rules live

| Rule | Class and method |
|---|---|
| Only allowed status changes (section 9 of BRD) | `BookingStatus.canMoveTo()`, called by `Booking.transitionTo()` |
| Cancellation fees (BR-29) | `Booking.cancellationFee()` |
| Delivery PIN, 3 attempts (BR-31) | `Booking.verifyPin()` |
| Volumetric and chargeable weight (BR-06, BR-07) | `BookingItem.volumetricWeight()`, `chargeableWeight()` |
| 2 helpers for items over 120 kg (BR-10) | `BookingItem.needsTwoHelpers()` |
| New customer until 5 rated jobs (BR-34) | `Customer.isNew()` |
| Low rating thresholds 3.5 / 3.0 (BR-35, BR-36) | `Customer.isLowRated()`, `needsReview()` |
| Reapply after 30 days (BR-40) | `Driver.canReapply()` |
| Rounding up to nearest R10 (BR-09) | `Money.roundUpToNearest10()` |

## 5.2 Pricing and promotions

```mermaid
classDiagram
    direction TB

    class QuoteService {
        -PriceCalculator calculator
        -DiscountSelector discountSelector
        -DistanceService distanceService
        +createQuote(QuoteRequest request) Quote
    }

    class PriceCalculator {
        -PricingSettings settings
        +deliveryFee(Weight chargeable, BigDecimal km, VehicleType vehicle, SpeedTier tier) Money
        +addOnsTotal(List~AddOn~ addOns) Money
        +weightBandFee(Weight chargeable) Money
    }

    class PricingSettings {
        -Map~VehicleType, VehicleRate~ vehicleRates
        -List~WeightBand~ weightBands
        -Map~SpeedTier, BigDecimal~ multipliers
        -Map~AddOnType, Money~ addOnPrices
    }

    class DistanceService {
        <<interface>>
        +drivingDistanceKm(GeoPoint from, GeoPoint to) BigDecimal
    }

    class GoogleMapsDistanceService {
        +drivingDistanceKm(GeoPoint from, GeoPoint to) BigDecimal
    }

    class DiscountRule {
        <<interface>>
        +isApplicable(DiscountContext ctx) boolean
        +discountFor(DiscountContext ctx) Money
    }

    class PromoCodeDiscount {
        -PromoCode promoCode
        +isApplicable(DiscountContext ctx) boolean
        +discountFor(DiscountContext ctx) Money
    }

    class SpendDiscount {
        -List~SpendTier~ tiers
        +isApplicable(DiscountContext ctx) boolean
        +discountFor(DiscountContext ctx) Money
    }

    class DiscountSelector {
        +bestDiscount(List~DiscountRule~ rules, DiscountContext ctx) Money
    }

    class PromoCode {
        -String code
        -DiscountType type
        -BigDecimal value
        -Money maxDiscount
        -Money minOrderAmount
        -boolean firstOrderOnly
        -int maxTotalUses
        -int maxUsesPerCustomer
        +isActiveOn(Instant now) boolean
    }

    class Quote {
        -UUID id
        -Weight chargeableWeight
        -BigDecimal distanceKm
        -Money deliveryFee
        -Money discount
        -Money addOnsTotal
        -Money total
        -Instant expiresAt
        +isExpired(Instant now) boolean
    }

    QuoteService ..> PriceCalculator : uses
    QuoteService ..> DiscountSelector : uses
    QuoteService ..> DistanceService : uses
    QuoteService ..> Quote : creates
    PriceCalculator --> PricingSettings : reads
    GoogleMapsDistanceService ..|> DistanceService
    PromoCodeDiscount ..|> DiscountRule
    SpendDiscount ..|> DiscountRule
    PromoCodeDiscount --> PromoCode
    DiscountSelector ..> DiscountRule : compares
```

### Design notes

- **Strategy pattern:** `PromoCodeDiscount` and `SpendDiscount` both implement `DiscountRule`. `DiscountSelector` asks each applicable rule for its discount and keeps the **biggest one** (BR-22: no stacking, customer gets whichever saves more). A new kind of discount later (e.g. a business volume discount) is just a new class, with no changes to existing code.
- **Interface for maps:** `DistanceService` is an interface, so tests can use a fake that returns a fixed distance (no Google API calls, no cost, repeatable tests).
- **Settings, not constants:** `PriceCalculator` reads every price from `PricingSettings`, which is loaded from the database (NFR-19).
- **Money as a value object:** amounts are never `double`. `Money` wraps `BigDecimal` so rounding is always correct.