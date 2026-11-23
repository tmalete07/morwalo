# 3. Use cases

What each actor can do in Morwalo. Use cases are drawn as rounded shapes inside the system boundary.
`«include»` = always happens as part of the use case. `«extend»` = optional extra behaviour.

## 3.1 Customer app

```mermaid
flowchart LR
    customer["👤 Customer"]
    receiver["👤 Receiver"]
    payprov["💳 Payment provider"]

    subgraph sys["Morwalo – Customer app"]
        direction TB
        uc1(["Sign up / log in with OTP"])
        uc2(["Get a quote"])
        uc3(["Book a delivery"])
        uc4(["Pay for booking"])
        uc5(["Apply promo code"])
        uc6(["Add add-ons<br/>(helper, stairs, packing, cover)"])
        uc7(["Track delivery live"])
        uc8(["Approve price change"])
        uc9(["Cancel booking"])
        uc10(["Rate and tip driver"])
        uc11(["Report problem / damage claim"])
        uc12(["Download receipt"])
        uc13(["Give delivery PIN"])
    end

    customer --- uc1
    customer --- uc2
    customer --- uc3
    customer --- uc7
    customer --- uc8
    customer --- uc9
    customer --- uc10
    customer --- uc11
    customer --- uc12
    receiver --- uc7
    receiver --- uc13

    uc3 -. "«include»" .-> uc2
    uc3 -. "«include»" .-> uc4
    uc5 -. "«extend»" .-> uc4
    uc6 -. "«extend»" .-> uc3
    uc4 --- payprov
    uc9 --- payprov

    classDef actor fill:#EAF8FB,stroke:#0B7B93,color:#0E2A3B
    classDef usecase fill:#FFFFFF,stroke:#3EC6E0,color:#0E2A3B
    classDef ext fill:#F4F6F7,stroke:#8A9BA5,color:#0E2A3B,stroke-dasharray: 4 3
    class customer,receiver actor
    class payprov ext
    class uc1,uc2,uc3,uc4,uc5,uc6,uc7,uc8,uc9,uc10,uc11,uc12,uc13 usecase
```

## 3.2 Driver app

```mermaid
flowchart LR
    driver["🚚 Driver"]
    customer["👤 Customer"]

    subgraph sys["Morwalo – Driver app"]
        direction TB
        d1(["Sign up and upload documents"])
        d2(["Go online / offline"])
        d3(["Accept Express job offer"])
        d4(["Accept Saver / Standard route"])
        d5(["Navigate to address"])
        d6(["Confirm pickup with photos"])
        d7(["Flag item mismatch"])
        d8(["Refuse prohibited item"])
        d9(["Confirm delivery with PIN and photos"])
        d10(["Record failed delivery"])
        d11(["Rate customer"])
        d12(["View earnings and payouts"])
        d13(["Report emergency"])
    end

    driver --- d1
    driver --- d2
    driver --- d3
    driver --- d4
    driver --- d5
    driver --- d6
    driver --- d9
    driver --- d11
    driver --- d12
    driver --- d13

    d7 -. "«extend»" .-> d6
    d8 -. "«extend»" .-> d6
    d10 -. "«extend»" .-> d9
    d7 --- customer

    classDef actor fill:#EAF8FB,stroke:#0B7B93,color:#0E2A3B
    classDef usecase fill:#FFFFFF,stroke:#3EC6E0,color:#0E2A3B
    class driver,customer actor
    class d1,d2,d3,d4,d5,d6,d7,d8,d9,d10,d11,d12,d13 usecase
```

## 3.3 Admin website

```mermaid
flowchart LR
    superA["🛠️ Super admin"]
    ops["🛠️ Operations admin"]
    support["🛠️ Support agent"]
    finance["🛠️ Finance admin"]

    subgraph sys["Morwalo – Admin website"]
        direction TB
        a1(["Manage prices, settings, catalogue"])
        a2(["Manage promotions"])
        a3(["Manage admin users and roles"])
        a4(["Approve / reject / suspend drivers"])
        a5(["Assign or reassign jobs"])
        a6(["View and cancel bookings"])
        a7(["Handle claims and rating disputes"])
        a8(["Request refund"])
        a9(["Approve refund"])
        a10(["Run weekly payouts"])
        a11(["View reports and export CSV"])
        a12(["View audit log"])
    end

    superA --- a1
    superA --- a2
    superA --- a3
    superA --- a12
    ops --- a4
    ops --- a5
    ops --- a6
    support --- a6
    support --- a7
    support --- a8
    finance --- a9
    finance --- a10
    finance --- a11

    a9 -. "«extend»" .-> a8

    classDef actor fill:#EAF8FB,stroke:#0B7B93,color:#0E2A3B
    classDef usecase fill:#FFFFFF,stroke:#3EC6E0,color:#0E2A3B
    class superA,ops,support,finance actor
    class a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12 usecase
```

> **Note:** the Super admin can do **every** admin use case. Only their unique ones are linked above, to keep the diagram readable. The full permission matrix is in the BRD, section 5.