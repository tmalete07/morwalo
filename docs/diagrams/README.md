# Morwalo – System Design Diagrams

Diagrams for the Morwalo courier platform, written in [Mermaid](https://mermaid.js.org/) so GitHub renders them directly.

> Morwalo is an app-based courier for Gauteng that moves parcels and big items (fridges, furniture, bicycles) for less, by combining loads and using independent drivers.

## Diagrams

| # | Diagram | Type | File |
|---|---|---|---|
| 1 | System context | Flowchart | [01-system-context.md](01-system-context.md) |
| 2 | Architecture (containers) | Flowchart | 02-architecture.md |
| 3 | Use cases | Flowchart (UML style) | 03-use-cases.md |
| 4 | Entity relationship diagram | erDiagram | 04-erd.md |
| 5 | Domain class diagram | UML class diagram | 05-class-diagram.md |
| 6 | Booking and payment | UML sequence diagram | 06-seq-booking-payment.md |
| 7 | Dispatch | UML sequence diagram | 07-seq-dispatch.md |
| 8 | Pickup and delivery | UML sequence diagram | 08-seq-pickup-delivery.md |
| 9 | Booking status | UML state diagram | 09-state-booking.md |
| 10 | Driver account status | UML state diagram | 10-state-driver.md |
| 11 | Price calculation | UML activity diagram | 11-activity-pricing.md |
| 12 | Deployment | Flowchart | 12-deployment.md |

## Related documents

- [Business Requirements Document (BRD)](../BRD/Morwalo_BRD_v0.1.pdf)

## Tech stack (planned)

- **Backend:** Java 21, Spring Boot (modular monolith)
- **Database:** PostgreSQL + PostGIS
- **Mobile apps:** React Native (Expo), TypeScript – Customer app and Driver app
- **Admin website:** Next.js
- **Live updates:** WebSockets, Firebase Cloud Messaging
- **CI:** GitHub Actions