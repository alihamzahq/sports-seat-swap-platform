# Subscription & Payment Flow

The platform supports two payment methods: **card payments** (immediate charge) and **bank transfer** (invoice-based). Subscriptions are managed through a payment gateway SDK with full lifecycle support — creation, upgrades, cancellation, and automatic past-due handling. Webhook events keep the system in sync with the payment provider's state, and a scheduled job auto-cancels subscriptions that remain unpaid after 14 days.

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

## Subscription Plans

| Duration | Billing Interval | Description |
|----------|-----------------|-------------|
| Monthly | 1 month | Rolling monthly subscription |
| Half Season | 6 months | Mid-season commitment |
| Full Season | 12 months | Full year with discount |

## Webhook Events Handled

| Event | Action |
|-------|--------|
| `invoice.payment_succeeded` | Activate subscription (bank transfers) |
| `invoice.payment_failed` | Log failure |
| `subscription.updated` | Sync status; send reminder if past_due |
| `subscription.deleted` | Mark as cancelled |
