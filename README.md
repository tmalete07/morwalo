<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="docs/assets/morwalo-logo-dark.svg">
    <img src="docs/assets/morwalo-logo-light.svg" alt="Morwalo" width="360">
  </picture>
</p>

<p align="center"><b>Big or small, we rwala it.</b></p>

<p align="center">
  <img src="https://img.shields.io/badge/status-prototype%20in%20development-3EC6E0" alt="Status: prototype in development">
  <img src="https://img.shields.io/badge/region-Gauteng%2C%20South%20Africa-0B7B93" alt="Region: Gauteng, South Africa">
  <img src="https://img.shields.io/badge/backend-Java%2021%20%7C%20Spring%20Boot-0E2A3B" alt="Backend: Java 21, Spring Boot">
  <img src="https://img.shields.io/badge/apps-React%20Native%20%7C%20Next.js-0E2A3B" alt="Apps: React Native, Next.js">
</p>

---

**Morwalo** (Sepedi for *load*) is an app-based courier for Gauteng that moves parcels **and big items** such as fridges, furniture and bicycles, at prices ordinary people and small businesses can afford.

Customers get an instant price, book a vetted driver with a panel van or mini truck, pay in the app and track the delivery live. Customers who can wait choose **Saver**, where their item travels with other loads going the same way, so they pay less and drivers earn more per trip.

## The problem

Moving one big item is expensive and awkward. Parcel couriers won't take a fridge or a couch, and movers price for a dedicated truck and crew. At the same time, many vans drive half-empty or return empty after a job.

## How it works

1. **Get a price.** Enter pickup and drop-off, pick items from a catalogue (or enter size and weight), and see prices for every speed option instantly.
2. **Book and pay.** Add a helper or stairs if needed, apply a promo code and pay in the app.
3. **Track it.** Follow the driver live on a map. The receiver gives a 4-digit PIN to confirm delivery, and photos are taken at pickup and drop-off.

| Speed | Gauteng delivery time | Price |
|---|---|---|
| **Express** | Same or next day | Highest |
| **Standard** | 2–3 days | Medium |
| **Saver** | Within 7 days, combined with other loads | Cheapest |

## What's in the system

| App | For | Highlights |
|---|---|---|
| 📱 **Customer app** | Individuals and businesses | Instant quotes, promo codes, live tracking, delivery PIN, ratings and tips, damage claims |
| 🚚 **Driver app** | Independent drivers | Job offers, Saver route packages, pickup and delivery photos (works offline), earnings and weekly payouts |
| 💻 **Admin website** | Morwalo staff | Driver approvals, dispatch, pricing and promotions, refunds, payouts, reports and audit log |

## Project status

This is a **prototype in development**: it uses test (sandbox) payments and demo data, and is designed to go live later.

- [x] Business requirements (BRD)
- [x] Branding (name, logo, colours)
- [x] System design diagrams (12 diagrams)
- [ ] API design
- [ ] Real-time tracking and notifications design
- [ ] Security design
- [ ] System Design Document
- [ ] Backend (Spring Boot)
- [ ] Customer app and Driver app (React Native)
- [ ] Admin website (Next.js)
- [ ] Automated tests and CI/CD

## Documentation

| Document | Description |
|---|---|
| 📄 [Business Requirements Document](docs/BRD/Morwalo_BRD_v0.1.pdf) | Mission, vision, business rules (BR-xx), functional (FR-xx) and non-functional (NFR-xx) requirements |
| 🧭 [System design diagrams](docs/diagrams/README.md) | Context, architecture, use cases, ERD, class, sequence, state, activity and deployment diagrams |

## Tech stack (planned)

| Layer | Technology |
|---|---|
| Backend | Java 21, Spring Boot (modular monolith) |
| Database | PostgreSQL + PostGIS |
| Mobile apps | React Native (Expo), TypeScript – Android first |
| Admin website | Next.js |
| Live updates | WebSockets, Firebase Cloud Messaging |
| Maps | Google Maps Platform |
| CI/CD | GitHub Actions, Docker |
| Testing | JUnit 5, RestAssured, Cucumber, Testcontainers, Playwright, Appium |

## Repository structure (planned)

```
morwalo/
├── backend/        # Spring Boot API (Java 21)
├── mobile/
│   ├── customer/   # Customer app (React Native, Expo)
│   └── driver/     # Driver app (React Native, Expo)
├── admin/          # Admin website (Next.js)
└── docs/
    ├── BRD/        # Business Requirements Document
    ├── diagrams/   # System design diagrams (Mermaid)
    └── assets/     # Logo and brand files
```

## Roadmap

1. **Prototype:** all three apps working with test payments and demo data
2. **Pilot:** real payments and a small group of vetted drivers in one area
3. **Gauteng launch:** all of Gauteng, first furniture-shop partners
4. **Grow:** business accounts, iOS, booking website, more vehicle types
5. **Nationwide:** partner long-haul carriers and backloads

## Author

**Tapiso Malete**: founder and developer. Software engineering student at WeThinkCode_, focused on Java development, QA and test automation.

[LinkedIn](www.linkedin.com/in/tapiso-malete-9a0379206) · [GitHub](https://github.com/tmalete07)

---

<sub>© 2026 Morwalo. All rights reserved. This repository is shared for portfolio and demonstration purposes.</sub>