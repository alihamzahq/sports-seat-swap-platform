# API Architecture

The API uses a versioned RESTful design with Sanctum token authentication. Requests pass through a middleware pipeline before reaching controllers, which delegate to a Repository layer for business logic. API responses follow a consistent envelope format via a shared response trait. The admin panel runs as a separate route group with session-based auth and role-based access control.

```mermaid
graph TB
    subgraph "Client Request"
        REQ[HTTP Request]
    end

    subgraph "Middleware Pipeline"
        CORS[CORS]
        RATE[Rate Limiting]
        SANC[Auth: Sanctum<br/>Token Validation]
        ADMIN_MW[Auth: Admin<br/>Role Check]
    end

    subgraph "API Routes — /api"
        direction TB
        AUTH_R["/auth<br/>Register · Login · Password Reset"]
        USER_R["/user<br/>Profile · Documents · Settings"]
        SEAT_R["/seats<br/>CRUD · Types · Photos"]
        SWAP_R["/swap-seats<br/>Offers · Accept · Reject · Tickets"]
        SUB_R["/subscription<br/>Plans · Create · Cancel · Resume"]
        PAY_R["/payment-methods<br/>CRUD · Set Default"]
        INV_R["/invoices<br/>List · Customer Details"]
        CHAT_R["/chat<br/>List · Send · Read Status"]
        NOTIF_R["/notifications<br/>List · Mark Read"]
        SEARCH_R["/search<br/>Seats · Teams · Leagues"]
    end

    subgraph "Admin Routes — /admin"
        direction TB
        DASH["/dashboard<br/>Analytics Overview"]
        USR_MGMT["/users<br/>Verify Docs · Ban · Activate"]
        SEAT_MGMT["/seats<br/>Approve · Reject"]
        SUB_MGMT["/subscriptions<br/>Plans CRUD · User Subs"]
        CONTENT["/content<br/>Countries · Leagues · Teams"]
        REPORTS["/reports<br/>Reported Users · Contact"]
    end

    subgraph "Application Layer"
        CTRL[Controllers]
        REPO[Repositories<br/>Business Logic]
        TRAIT[Traits<br/>API Response · File Upload · FCM]
        RES[API Resources<br/>Response Transformation]
    end

    subgraph "Data Layer"
        MODEL[Eloquent Models<br/>Scopes · Relations · Enums]
        DB[(Database)]
    end

    REQ --> CORS --> RATE

    RATE --> SANC --> AUTH_R
    SANC --> USER_R
    SANC --> SEAT_R
    SANC --> SWAP_R
    SANC --> SUB_R
    SANC --> PAY_R
    SANC --> INV_R
    SANC --> CHAT_R
    SANC --> NOTIF_R
    SANC --> SEARCH_R

    RATE --> ADMIN_MW --> DASH
    ADMIN_MW --> USR_MGMT
    ADMIN_MW --> SEAT_MGMT
    ADMIN_MW --> SUB_MGMT
    ADMIN_MW --> CONTENT
    ADMIN_MW --> REPORTS

    AUTH_R --> CTRL
    USER_R --> CTRL
    SEAT_R --> CTRL
    SWAP_R --> CTRL
    SUB_R --> CTRL
    PAY_R --> CTRL
    INV_R --> CTRL
    CHAT_R --> CTRL
    NOTIF_R --> CTRL
    SEARCH_R --> CTRL
    DASH --> CTRL
    USR_MGMT --> CTRL
    SEAT_MGMT --> CTRL
    SUB_MGMT --> CTRL
    CONTENT --> CTRL
    REPORTS --> CTRL

    CTRL --> REPO
    REPO --> TRAIT
    REPO --> MODEL
    CTRL --> RES
    MODEL --> DB
```

## Design Patterns

| Pattern | Implementation | Purpose |
|---------|---------------|---------|
| Repository | `app/Repositories/` | Separates business logic from controllers |
| API Resource | `app/Http/Resources/` | Consistent response transformation |
| Trait-based composition | `ApiResponseTrait`, `FileUploadTrait`, `SendFcmNotificationTrait` | Reusable cross-cutting concerns |
| Form Request validation | `app/Http/Requests/` | Request validation separated from controllers |
| Enum status management | `SeatStatus`, `SwapSeatOfferStatus` | Type-safe status handling |
