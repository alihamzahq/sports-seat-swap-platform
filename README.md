# Sports Seat Swapping Marketplace — Backend Platform

A production-grade backend system powering a sports seat swapping marketplace, built with Laravel and designed to scale across multiple sports, regions, and mobile clients.

This repository represents a **real-world commercial project**. Source code is private due to confidentiality, but this documentation outlines the system architecture, technical decisions, and my contributions as Lead Backend Developer.

---

## Overview

The platform enables sports fans to:
- List stadium seats for upcoming events
- Discover and exchange seats with other users
- Communicate via real-time chat
- Subscribe to premium access tiers
- Receive push notifications and transactional emails

It serves **iOS and Android mobile applications** via a versioned REST API and includes a full-featured **Admin CMS** for internal operations.

---

## Tech Stack

**Backend**
- Laravel 10 · PHP 8.2
- MySQL · Redis
- RESTful APIs (versioned)
- Laravel Sanctum (API authentication)
- Stripe (subscriptions & billing)
- Firebase Cloud Messaging (push notifications)

**Admin CMS**
- Blade Templates
- Server-side DataTables
- Vite asset bundling

**DevOps & Quality**
- CI/CD pipeline
- PHPUnit testing
- Static code analysis
- Queues & scheduled tasks

---

## Key Capabilities

- 80+ REST API endpoints for mobile clients
- Subscription billing with Stripe (full lifecycle)
- Seat listings, swap offers, and approval workflows
- Real-time chat and push notifications
- Identity verification with admin moderation
- Automated background jobs and scheduled tasks
- Multi-sport data ingestion from external APIs

---

## Architecture

> Diagrams reflect the real production system. Source code is confidential.

### System Architecture

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

### Stripe Payment Flow

```mermaid
sequenceDiagram
    autonumber
    participant U as User / Mobile App
    participant API as REST API
    participant REPO as Subscription<br/>Repository
    participant GW as Payment Gateway
    participant WH as Webhook Handler
    participant Q as Queue Worker
    participant DB as Database
    participant MAIL as Email Service

    note over U, MAIL: Card Payment Flow
    U->>API: POST /subscription (card + plan + duration)
    API->>REPO: createSubscription()
    REPO->>GW: Create subscription with payment token
    GW-->>REPO: Subscription active
    REPO->>DB: Save subscription record
    REPO->>Q: Dispatch SubscriptionCreated event
    Q->>MAIL: Send confirmation email
    REPO-->>API: Return subscription details
    API-->>U: 200 OK — Subscription active

    note over U, MAIL: Bank Transfer Flow
    U->>API: POST /subscription (bank_transfer + plan)
    API->>REPO: createSubscription()
    REPO->>GW: Create subscription + send invoice
    GW-->>REPO: Subscription incomplete (awaiting payment)
    REPO->>DB: Save subscription (status: incomplete)
    REPO->>Q: Dispatch BankTransferCreated event
    Q->>GW: Customize & finalize invoice
    Q->>MAIL: Send invoice PDF to user
    REPO-->>API: Return subscription details
    API-->>U: 200 OK — Invoice sent

    note over U, MAIL: Webhook — Payment Received
    GW->>WH: invoice.payment_succeeded
    WH->>DB: Update status → active

    note over U, MAIL: Webhook — Payment Past Due
    GW->>WH: subscription.updated (past_due)
    WH->>DB: Update status + timestamp
    WH->>Q: Send payment reminder notification

    note over U, MAIL: Plan Upgrade
    U->>API: POST /subscription/update (new plan)
    API->>REPO: updateSubscription()
    REPO->>REPO: Validate upgrade (no downgrades)
    REPO->>GW: Swap plan + prorate invoice
    REPO->>Q: Dispatch PlanUpgrade event
    Q->>MAIL: Send upgrade confirmation + invoice

    note over U, MAIL: Cancellation
    U->>API: POST /subscription/cancel
    API->>REPO: cancelSubscription()
    REPO->>GW: Cancel at period end
    REPO->>Q: Dispatch SubscriptionCancel event
    Q->>MAIL: Send cancellation email

    note over U, MAIL: Auto-Cancel Past Due (Scheduled Job — Daily)
    Q->>DB: Find past_due > 14 days
    Q->>GW: Cancel immediately
    Q->>DB: Update status → cancelled
```

📁 **More diagrams:** [API Architecture](./docs/api-architecture.md) · [Firebase Notifications](./docs/firebase-notifications.md) · [Chat Architecture](./docs/chat-architecture.md) · [Database Schema](./docs/database-schema.md)

---

## My Role

**Lead Backend Developer**

- Designed the system architecture and API structure
- Built the REST API and Admin CMS
- Designed the database schema and domain models
- Implemented Stripe billing and webhook handling
- Integrated Firebase push notifications and real-time chat
- Built background jobs, schedulers, and data pipelines
- Led backend development and technical decision-making

---

## Documentation

Detailed technical documentation is available here:

📁 **[`/docs`](./docs)**  
- Architecture & design patterns  
- API structure  
- Database schema  
- Background processing  
- Security & integrations  

---

## Note on Source Code

This is a **proprietary commercial project**.  
The source code is not publicly available.  
This repository is shared **for portfolio and technical demonstration purposes only**.

---

**Author:** Ali Hamza  
**Role:** Senior / Lead Backend Engineer
