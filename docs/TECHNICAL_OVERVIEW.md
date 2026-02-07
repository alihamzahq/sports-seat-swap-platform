# Technical Overview — Sports Seat Swapping Marketplace Backend

This document provides a detailed technical overview of a **production-grade Laravel backend system** powering a sports seat swapping marketplace. The platform serves mobile applications (iOS & Android) via REST APIs and includes an internal Admin CMS for operations and moderation.

> **Note:** This is a proprietary system. Source code is not publicly available. This documentation demonstrates architecture, engineering decisions, and technical depth.

---

## System Overview

The platform enables sports fans to **list, discover, and swap stadium seats** for live sporting events across multiple sports. It supports real-time interactions, subscription-based access, identity verification, and automated sports data synchronization.

### Core Components

1. **REST API Layer**
   - Versioned JSON APIs (`/api/v1`)
   - Serves mobile clients
   - Token-based authentication (Laravel Sanctum)

2. **Admin CMS**
   - Server-rendered Blade application
   - Used by internal operations teams
   - Moderation, verification, analytics, and billing management

---

## Architecture

### High-Level Flow

HTTP Request
→ Middleware (Auth, Admin, Rate Limiting, CSRF)
→ Controller (Thin)
→ Repository (Business Logic)
→ Eloquent Models
→ Database

Side Effects:
→ Events & Listeners (Emails, Notifications)
→ Queued Jobs (Async processing)
→ Scheduled Tasks (Automation)


### Architectural Principles

- **Separation of Concerns** — Controllers handle HTTP, repositories handle domain logic
- **Domain-Oriented Design** — Features grouped by business domain
- **Event-Driven Side Effects** — Non-blocking, decoupled processing
- **Scalability First** — Async queues and scheduled jobs
- **Maintainability** — Predictable structure and reusable abstractions

---

## Design Patterns & Practices

### Repository Pattern
- 20+ repositories organized by domain
- Centralized business rules
- Thin controllers, testable logic

### Event-Driven Architecture
- Domain events trigger listeners for:
  - Email dispatch
  - Push notifications
  - Webhook handling
- Core workflows remain clean and synchronous

### Traits for Shared Logic
Reusable traits encapsulate:
- API response formatting
- File uploads
- Payment handling
- External API communication
- Notification dispatching

### PHP 8.1+ Enums
Type-safe enums used for:
- Offer lifecycle states
- Listing approval statuses
- Identity verification stages
- Subscription states
- Document types

### Validation & Transformation
- **Form Request classes** validate all incoming input
- **API Resource transformers** standardize JSON responses
- Version-safe response envelopes

---

## Technology Stack

### Backend

| Technology | Purpose |
|----------|--------|
| Laravel 10 | Core framework |
| PHP 8.2 | Language runtime |
| MySQL | Relational database |
| Redis | Cache, queues, sessions |
| Laravel Sanctum | API authentication |
| Laravel Cashier | Stripe subscriptions |
| Firebase (FCM) | Push notifications |
| Guzzle | External API communication |
| DomPDF | Invoice PDF generation |

### DevOps & Quality

- Redis-backed queue workers
- Scheduled artisan commands
- CI/CD pipeline
- PHPUnit tests
- Static code analysis
- Automated code formatting

---

## Core Domain Features

### Seat Marketplace
- CRUD seat listings with media uploads
- Match-specific seat mapping
- Admin approval workflow
- Automated listing expiration
- Verification document handling

### Swap Offer System
- Peer-to-peer seat swap lifecycle
- Status-based state machine
- Offer expiration automation
- Digital ticket exchange
- Scheduled archival

### Search & Discovery
- Multi-criteria filtering
- Geographic drill-down
- Popular item surfacing
- Autocomplete search

### User & Identity Management
- Authentication & profiles
- Multi-step identity verification
- Status lifecycle notifications
- Moderation & reporting system

### Subscriptions & Billing (Stripe)
- Multi-tier plans
- Full subscription lifecycle
- Payment methods management
- PDF invoices
- Webhook processing
- Bank transfer fallback

### Real-Time Communication
- In-app messaging
- Delivery & read receipts
- Firebase push notifications
- WebSocket broadcasting

---

## Background Processing

### Queued Jobs
- Sports data ingestion
- Push notification delivery
- Async processing of heavy tasks

### Scheduled Tasks

| Frequency | Task |
|--------|------|
| Every 30 minutes | Expire stale swap offers |
| Daily | Archive expired listings |
| Daily | Sync sports fixtures |
| Daily | Subscription enforcement |
| Twice daily | Archive completed swaps |

---

## Database Design

### Schema Overview

- 45+ migrations
- 25+ models
- 6 domain modules
- Indexed foreign keys
- Pivot tables for many-to-many relations

#### Relationship Highlights
- Users → Listings → Seats
- Users ↔ Swap Offers
- Teams ↔ Leagues ↔ Matches
- Users ↔ Subscriptions (Stripe Billable)

---

## API Design

### Route Groups

| Group | Auth | Purpose |
|----|----|------|
| Public | None | Auth & onboarding |
| Authenticated | Token | Core marketplace |
| Admin | Session | CMS & moderation |
| Webhooks | Signature | Stripe events |

### API Principles
- Versioned endpoints
- Standard response envelope
- Request validation at boundary
- Rate limiting & security middleware

---

## Project Structure

The backend follows a **domain-oriented Laravel structure** optimized for scale and clarity.

```
├── Controllers/
│   ├── Api/                  # REST API controllers (mobile app)
│   │   ├── Auth/             # Authentication & registration
│   │   ├── Marketplace/      # Seat listings & swap offers
│   │   ├── Subscriptions/    # Billing & payment methods
│   │   ├── Messaging/        # Real-time chat
│   │   └── ...               # Notifications, search, user profile
│   └── Admin/                # CMS controllers (admin panel)
│       ├── Users/            # User & document moderation
│       ├── Listings/         # Seat listing management
│       ├── Subscriptions/    # Plan & billing management
│       └── ...               # Teams, leagues, reports, settings
├── Models/                   # 25+ Eloquent models with relationships
├── Repositories/             # 20+ domain-organized data access classes
├── Events & Listeners/       # Event-driven email & notification dispatch
├── Jobs/                     # Queued background processing
├── Mail/                     # 10 transactional email templates
├── Notifications/            # 12 push notification classes
├── Enums/                    # PHP 8.1 backed enums for domain states
├── Traits/                   # 6 shared traits (API responses, uploads, payments, etc.)
├── Requests/                 # 18 form request validators
├── Resources/                # 20 JSON API resource transformers
├── Middleware/                # Auth, admin, CSRF, rate limiting
├── Console/                  # 8 scheduled artisan commands
├── Routes/
│   ├── api                   # Versioned REST API routes (mobile)
│   ├── admin                 # Admin CMS routes (web panel)
│   └── webhooks              # Stripe webhook handler
├── Views/
│   ├── admin/                # Blade templates for CMS dashboard
│   └── emails/               # Email notification templates
├── Database/
│   ├── migrations/           # 45+ schema migrations
│   └── seeders/              # Initial data seeders
└── Config & DevOps           # CI/CD pipeline, static analysis, tests
```

---


### Structure Rationale
- Controllers remain thin
- Business logic centralized
- Side effects fully decoupled
- Clear separation of API and Admin layers

---

## Security Considerations

- Token-based API authentication
- Admin authorization gates
- CSRF protection
- Rate limiting
- Input validation
- Secure Stripe webhook verification

---

## My Role & Contributions

As **Lead Backend Developer**, I was responsible for:

- System architecture and technical direction
- REST API design (80+ endpoints)
- Admin CMS development
- Database schema design
- Stripe subscription system
- Real-time messaging & notifications
- Sports data ingestion pipeline
- Identity verification workflow
- Background job & scheduler architecture
- CI/CD, testing, and code quality enforcement

---

## Summary

This project demonstrates expertise in:

- Scalable Laravel architecture
- REST API design
- Payment systems (Stripe)
- Real-time communication
- Background processing
- Event-driven design
- Secure, maintainable backend systems

---

*This document is intended for technical evaluation and portfolio demonstration purposes.*
