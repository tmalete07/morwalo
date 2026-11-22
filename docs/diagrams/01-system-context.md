# 1. System context

Morwalo as one system, the people who use it, and the outside services it depends on.

```mermaid
flowchart TB
    customer["👤 Customer<br/><small>Individual or business<br/>books and tracks deliveries</small>"]
    receiver["👤 Receiver<br/><small>Gets SMS tracking link<br/>gives delivery PIN</small>"]
    driver["🚚 Driver<br/><small>Independent partner<br/>picks up and delivers</small>"]
    admin["🛠️ Admin staff<br/><small>Super, Operations,<br/>Support, Finance</small>"]

    morwalo[["<b>Morwalo platform</b><br/><small>Customer app · Driver app · Admin website<br/>Quotes, bookings, dispatch, tracking, payments</small>"]]

    payments["💳 Payment provider<br/><small>Card payments, refunds<br/>(sandbox in prototype)</small>"]
    maps["🗺️ Google Maps Platform<br/><small>Address search,<br/>distance, maps</small>"]
    sms["✉️ SMS provider<br/><small>OTP codes,<br/>tracking links</small>"]
    push["🔔 Firebase Cloud Messaging<br/><small>Push notifications</small>"]
    email["📧 Email service<br/><small>Receipts, updates</small>"]

    customer -- "Gets quotes, books, pays,<br/>tracks, rates, tips" --> morwalo
    receiver -- "Opens tracking link" --> morwalo
    driver -- "Accepts jobs, uploads proof,<br/>shares location" --> morwalo
    admin -- "Manages drivers, bookings,<br/>prices, refunds" --> morwalo

    morwalo -- "Takes payments,<br/>issues refunds" --> payments
    morwalo -- "Looks up addresses,<br/>calculates km" --> maps
    morwalo -- "Sends OTPs and links" --> sms
    morwalo -- "Sends app notifications" --> push
    morwalo -- "Sends receipts" --> email

    classDef person fill:#EAF8FB,stroke:#0B7B93,color:#0E2A3B
    classDef system fill:#3EC6E0,stroke:#0E2A3B,stroke-width:2px,color:#0E2A3B
    classDef external fill:#F4F6F7,stroke:#8A9BA5,color:#0E2A3B,stroke-dasharray: 4 3

    class customer,receiver,driver,admin person
    class morwalo system
    class payments,maps,sms,push,email external
```

## Legend

| Style | Meaning |
|---|---|
| Light cyan box | A person who uses Morwalo |
| Cyan box | The Morwalo platform (what we build) |
| Grey dashed box | An outside service Morwalo depends on (not built by us) |

## Notes

- In the prototype, payments run in **sandbox (test) mode** and SMS/OTP is **simulated** with a fixed test code.
- The receiver doesn't need the app. They only use the SMS tracking link and give the PIN to the driver.