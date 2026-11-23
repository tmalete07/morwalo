# Morwalo – System Design Diagrams

Diagrams for the Morwalo courier platform, written in [Mermaid](https://mermaid.js.org/) so GitHub renders them directly.

> Morwalo is an app-based courier for Gauteng that moves parcels and big items (fridges, furniture, bicycles) for less, by combining loads and using independent drivers.

## Where to start

| If you want to… | Read |
|---|---|
| Understand Morwalo in 2 minutes | [1. System context](01-system-context.md) |
| See how the system is built | [2. Architecture](02-architecture.md) |
| See what each user can do | [3. Use cases](03-use-cases.md) |
| Understand the data | [4. ERD](04-erd.md) and [5. Class diagram](05-class-diagram.md) |
| Follow a delivery end to end | [6](06-seq-booking-payment.md) → [7](07-seq-dispatch.md) → [8](08-seq-pickup-delivery.md) |
| Design tests | [9. Booking status](09-state-booking.md), [10. Driver status](10-state-driver.md), [11. Pricing](11-activity-pricing.md) |
| See where it runs | [12. Deployment](12-deployment.md) |

## All diagrams

| # | Diagram | Type | What it shows |
|---|---|---|---|
| 1 | [System context](01-system-context.md) | Flowchart | Morwalo, its users and outside services |
| 2 | [Architecture (containers)](02-architecture.md) | Flowchart | Apps, backend modules, data stores, external services |
| 3 | [Use cases](03-use-cases.md) | Flowchart (UML style) | What customers, drivers and admins can do |
| 4 | [Entity relationship diagram](04-erd.md) | erDiagram | Database tables, columns, keys and relationships |
| 5 | [Domain class diagram](05-class-diagram.md) | UML class diagram | Main classes and where business rules live |
| 6 | [Booking and payment](06-seq-booking-payment.md) | UML sequence diagram | Quote → promo → pay → booked |
| 7 | [Dispatch](07-seq-dispatch.md) | UML sequence diagram | Express offers, Saver routes, manual assignment |
| 8 | [Pickup and delivery](08-seq-pickup-delivery.md) | UML sequence diagram | Photos, mismatch, refusal, PIN, failed delivery |
| 9 | [Booking status](09-state-booking.md) | UML state diagram | Every booking status and allowed move |
| 10 | [Driver account status](10-state-driver.md) | UML state diagram | Applied → Approved → Suspended, online / on a job |
| 11 | [Price calculation](11-activity-pricing.md) | UML activity diagram | Pricing logic step by step, with test cases |
| 12 | [Deployment](12-deployment.md) | Flowchart | Environments, hosting and CI/CD pipeline |

## Traceability

Rule and requirement IDs (BR-xx, FR-xx, NFR-xx) used in these diagrams come from the BRD, so each design decision can be traced back to a business requirement and forward to a test.

## Related documents

- [Business Requirements Document (BRD)](../BRD/Morwalo_BRD_v0.1.pdf)

## Tech stack (planned)

- **Backend:** Java 21, Spring Boot (modular monolith)
- **Database:** PostgreSQL + PostGIS
- **Mobile apps:** React Native (Expo), TypeScript – Customer app and Driver app
- **Admin website:** Next.js
- **Live updates:** WebSockets, Firebase Cloud Messaging
- **CI/CD:** GitHub Actions, Docker
- **Testing:** JUnit 5, RestAssured, Cucumber, Testcontainers, Playwright, Appium