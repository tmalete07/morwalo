# 12. Deployment diagram

Where each part of Morwalo runs, and how code goes from a laptop to the cloud.
Hosting options below are **candidates** (free or low-cost tiers) to be confirmed before building.

## 12.1 Environments

| Environment | Purpose | Where |
|---|---|---|
| Local | Day-to-day development and tests | Developer laptop, Docker Compose |
| Demo (staging) | Shared prototype for demos and portfolio | Cloud, sandbox payments, test data |
| Production | Real customers (later, at pilot stage) | Cloud, real payments, backups, monitoring |

## 12.2 Prototype deployment

```mermaid
flowchart LR
    subgraph dev["💻 Developer laptop (VS Code)"]
        direction TB
        code["Source code<br/><small>backend · mobile · admin · docs</small>"]
        subgraph compose["Docker Compose (local)"]
            direction TB
            lbe["Backend container<br/><small>Spring Boot</small>"]
            ldb[("PostgreSQL + PostGIS")]
            lfs[("MinIO<br/><small>local S3 storage</small>")]
        end
    end

    subgraph gh["GitHub"]
        direction TB
        repo["morwalo repo"]
        ci["GitHub Actions<br/><small>build · test · Docker image</small>"]
        reg["Container registry<br/><small>GHCR</small>"]
    end

    subgraph cloud["☁️ Cloud (demo environment)"]
        direction TB
        be["Backend container<br/><small>Spring Boot, HTTPS + WebSocket<br/>e.g. Render / Railway / Fly.io</small>"]
        db[("Managed PostgreSQL + PostGIS<br/><small>e.g. Neon / Supabase</small>")]
        fs[("Object storage<br/><small>e.g. Cloudflare R2</small>")]
        adminweb["Admin website<br/><small>Next.js on Vercel</small>"]
    end

    subgraph phones["📱 Android phones"]
        direction TB
        capp["Customer app<br/><small>APK via Expo EAS</small>"]
        dapp["Driver app<br/><small>APK via Expo EAS</small>"]
    end

    subgraph ext["External services"]
        direction TB
        pay["💳 Payment provider (sandbox)"]
        maps["🗺️ Google Maps"]
        fcm["🔔 Firebase Cloud Messaging"]
        mail["📧 Email service"]
    end

    code -- "git push" --> repo
    repo --> ci
    ci -- "tests pass" --> reg
    reg -- "deploy" --> be
    ci -- "deploy" --> adminweb
    be --> db
    be --> fs
    capp -- "HTTPS / WSS" --> be
    dapp -- "HTTPS / WSS" --> be
    adminweb -- "HTTPS" --> be
    be --> pay
    be --> maps
    be --> fcm
    be --> mail
    pay -- "webhook" --> be

    classDef local fill:#F4F6F7,stroke:#8A9BA5,color:#0E2A3B
    classDef cloudc fill:#EAF8FB,stroke:#0B7B93,color:#0E2A3B
    classDef store fill:#0E2A3B,stroke:#0E2A3B,color:#FFFFFF
    classDef extc fill:#FFFFFF,stroke:#8A9BA5,color:#0E2A3B,stroke-dasharray: 4 3
    class code,lbe local
    class be,adminweb,capp,dapp,repo,ci,reg cloudc
    class ldb,lfs,db,fs store
    class pay,maps,fcm,mail extc
```

## 12.3 CI/CD pipeline

```mermaid
flowchart LR
    push(["git push /<br/>pull request"]) --> build["Build<br/><small>Maven</small>"]
    build --> unit["Unit tests<br/><small>JUnit 5</small>"]
    unit --> api["API + acceptance tests<br/><small>RestAssured, Cucumber<br/>Testcontainers DB</small>"]
    api --> gate{"All tests pass?"}
    gate -- No --> fail(["❌ Block merge,<br/>notify developer"])
    gate -- Yes --> img["Build Docker image"]
    img --> branch{"Branch = main?"}
    branch -- No --> stop(["✅ Ready for review"])
    branch -- Yes --> deploy["Deploy to demo<br/>environment"]
    deploy --> smoke["Smoke tests<br/><small>health check, quote API</small>"]
    smoke --> live(["✅ Live on demo"])

    classDef bad fill:#FDECEC,stroke:#E5484D,color:#0E2A3B
    classDef good fill:#EAF8FB,stroke:#0B7B93,color:#0E2A3B
    class fail bad
    class live,stop good
```

## Notes

| Choice | Reason |
|---|---|
| Docker everywhere | The backend runs the same on a laptop and in the cloud |
| Testcontainers in CI | API tests run against a real PostgreSQL + PostGIS, not a fake |
| Secrets in GitHub / host settings | API keys and passwords are never committed to the repo |
| APKs via Expo EAS | Testers install the apps directly; Google Play internal testing later |
| Backups and monitoring | Added for production at the pilot stage |