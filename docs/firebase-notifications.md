# Push Notification Flow

Push notifications are dispatched asynchronously via a queue job. When a triggering event occurs (swap offer, seat approval, chat message, account status change), the system saves an in-app notification to the database and dispatches a background job that sends a multicast push to all of the user's registered devices via Firebase Cloud Messaging.

```mermaid
sequenceDiagram
    autonumber
    participant SRC as Event Source<br/>(Controller / Listener)
    participant TRAIT as FCM Trait
    participant Q as Queue
    participant JOB as Push Notification<br/>Job
    participant DB as Database
    participant FCM as Cloud Messaging<br/>Service
    participant DEV as User Devices

    note over SRC, DEV: Notification Trigger Events
    note right of SRC: • Swap offer sent/accepted/rejected<br/>• Seat approved/rejected<br/>• Chat message received<br/>• Account status changed<br/>• Profile verified

    SRC->>DB: Save in-app notification (notifications table)
    SRC->>TRAIT: sendNotification(user, title, body, category, entityId)
    TRAIT->>Q: Dispatch PushNotificationJob

    Q->>JOB: Process job
    JOB->>DB: Fetch user's device tokens
    DB-->>JOB: Return FCM token list

    alt Has registered devices
        JOB->>FCM: sendMulticast(message, deviceTokens)
        note right of JOB: Payload includes:<br/>• notification: {title, body}<br/>• data: {entity_id, category, datetime}
        FCM->>DEV: Deliver to all registered devices
        DEV-->>FCM: Delivery receipt
    else No devices registered
        JOB->>JOB: Skip — no tokens found
    end

    note over SRC, DEV: Notification Categories
    note right of DEV: seat_approved · seat_rejected<br/>swap_offer_sent · swap_offer_received<br/>swap_offer_accepted · swap_offer_rejected<br/>chat · profile_approved
```

## Device Token Management

- Tokens are registered during **login** and **signup** (one per device)
- Multiple tokens per user supported (multi-device)
- Stored in a dedicated `device_tokens` table
- `updateOrCreate` ensures no duplicates

## Notification Channels

| Channel | Purpose | Delivery |
|---------|---------|----------|
| Database | In-app notification feed | Synchronous |
| FCM Push | Mobile device alerts | Async via queue |
| Email | Subscription & account events | Async via queue |
