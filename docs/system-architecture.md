# High-Level System Architecture

The platform follows a service-oriented architecture with a Laravel monolith at its core. The API serves mobile clients via token-based auth, while the Admin CMS uses session-based auth. Background workers handle sports data ingestion, notification delivery, and subscription lifecycle management. External integrations include a payment gateway, push notification service, and a third-party sports data feed.

```mermaid
graph TB
    subgraph Clients
        MA[Mobile App]
        AB[Admin Browser]
    end

    subgraph "Application Layer"
        API[REST API<br/>Token Auth]
        CMS[Admin CMS<br/>Session Auth]
    end

    subgraph "Core Services"
        AUTH[Auth Service<br/>Sanctum Tokens]
        SEAT[Seat Management<br/>CRUD + Approval]
        SWAP[Swap Engine<br/>Offer Lifecycle]
        SUB[Subscription Service<br/>Plans + Billing]
        CHAT[Chat Service<br/>Messaging]
        NOTIF[Notification Service<br/>Push + Email + DB]
        SEARCH[Search Service<br/>Seats + Teams]
    end

    subgraph "Background Workers"
        QW[Queue Workers<br/>Job Processing]
        SCHED[Task Scheduler<br/>Cron Jobs]
    end

    subgraph "Data Stores"
        DB[(MySQL<br/>Primary Database)]
        RTDB[(Realtime Database<br/>Chat Messages)]
        QUEUE[(Queue Store<br/>Job Queue)]
    end

    subgraph "External Services"
        PAY[Payment Gateway<br/>Subscriptions + Invoicing]
        PUSH[Push Notification<br/>Service FCM]
        SPORT[Sports Data API<br/>Fixtures + Teams]
        MAIL[Email Service<br/>Transactional Mail]
    end

    MA -->|HTTPS / JSON| API
    AB -->|HTTPS| CMS

    API --- AUTH
    API --- SEAT
    API --- SWAP
    API --- SUB
    API --- CHAT
    API --- NOTIF
    API --- SEARCH
    CMS --- SEAT
    CMS --- SUB
    CMS --- AUTH

    AUTH --> DB
    SEAT --> DB
    SWAP --> DB
    SUB --> DB
    SUB -->|Charges + Webhooks| PAY
    CHAT --> RTDB
    NOTIF --> PUSH
    NOTIF --> MAIL
    NOTIF --> DB
    SEARCH --> DB

    QW --> PUSH
    QW --> MAIL
    QW --> SPORT
    QW <--> QUEUE
    SCHED -->|Dispatches| QUEUE

    SPORT -.->|Daily Sync| DB
    PAY -.->|Webhooks| API
```

## Key Components

| Component | Technology | Purpose |
|-----------|-----------|---------|
| REST API | Laravel + Sanctum | Mobile client endpoints |
| Admin CMS | Laravel + Blade | Content & user management |
| Database | MySQL | Primary data store |
| Realtime DB | Firebase RTDB | Chat message persistence |
| Queue | Database driver | Async job processing |
| Payment | Stripe via Cashier | Subscriptions & invoicing |
| Push Notifications | Firebase Cloud Messaging | Mobile push delivery |
| Sports Data | SportCC API | Fixture & team data sync |
| Email | Configured mail driver | Transactional emails |
