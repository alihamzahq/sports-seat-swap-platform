# Real-Time Chat Architecture

Chat is scoped to swap offers — when two users have an active swap, a chat channel opens between them. Messages are persisted in Firebase Realtime Database, enabling instant sync on the client side. The Laravel API handles message creation, delivery/read receipts, and chat lifecycle (soft-delete with business rule enforcement). Each new message also triggers an in-app notification and a push notification to the recipient.

## System Overview

```mermaid
graph TB
    subgraph "Sender"
        S_APP[Mobile App<br/>Sender]
    end

    subgraph "Laravel API"
        CHAT_CTRL[Chat Controller]
        CHAT_REPO[Chat Repository]
        NOTIF_TRAIT[Notification Trait]
    end

    subgraph "Data Stores"
        RTDB[(Realtime Database<br/>Firebase RTDB)]
        DB[(MySQL<br/>Notifications + Offers)]
    end

    subgraph "Notification Pipeline"
        Q[Queue Worker]
        FCM[Cloud Messaging<br/>Service]
    end

    subgraph "Receiver"
        R_APP[Mobile App<br/>Receiver]
    end

    S_APP -->|"POST /chat/send<br/>{offer_id, message}"| CHAT_CTRL
    CHAT_CTRL --> CHAT_REPO

    CHAT_REPO -->|"Save message at<br/>chat/{offerId}/{msgKey}"| RTDB
    CHAT_REPO -->|"Save DB notification"| DB
    CHAT_REPO --> NOTIF_TRAIT
    NOTIF_TRAIT -->|"Dispatch push job"| Q
    Q -->|"Send push"| FCM
    FCM -->|"Push notification"| R_APP

    RTDB -.->|"Realtime sync<br/>(client SDK listener)"| R_APP

    R_APP -->|"POST /chat/messages/mark-as-delivered"| CHAT_CTRL
    CHAT_CTRL -->|"Update delivered_at"| RTDB

    R_APP -->|"POST /chat/messages/mark-as-read"| CHAT_CTRL
    CHAT_CTRL -->|"Update read_at"| RTDB
```

## Message Lifecycle

```mermaid
sequenceDiagram
    autonumber
    participant S as Sender App
    participant API as Laravel API
    participant RTDB as Realtime DB
    participant Q as Queue Worker
    participant FCM as Push Service
    participant R as Receiver App

    S->>API: POST /chat/send {offer_id, message}
    API->>RTDB: Store message at chat/{offerId}/{key}
    API->>API: Save in-app notification to DB
    API->>Q: Dispatch push notification job
    API-->>S: 200 OK

    RTDB-->>R: Realtime sync — new message appears
    Q->>FCM: Send push notification
    FCM->>R: Push: "User sent you a message"

    R->>API: POST /chat/messages/mark-as-delivered
    API->>RTDB: Update delivered_at timestamp

    R->>API: POST /chat/messages/mark-as-read
    API->>RTDB: Update read_at timestamp
```

## Message Data Structure

```json
{
  "sender_id": 42,
  "receiver_id": 17,
  "message": "Is the seat still available?",
  "delivered_at": "",
  "read_at": "",
  "send_at": 1700000000,
  "deleted_by_sender": false,
  "deleted_by_receiver": false
}
```

## Chat Business Rules

| Rule | Description |
|------|-------------|
| Scoped to swap offers | Chat only exists between users with an active swap offer |
| Soft delete | Messages marked as `deleted_by_sender/receiver`, not removed |
| Delete protection | Cannot delete chat on accepted offers until both matches finish (120+ min) |
| Active flags | `sender_chat_active` / `receiver_chat_active` control visibility |

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/chat/list` | Get all chat conversations |
| POST | `/chat/send` | Send a message |
| POST | `/chat/messages/mark-as-delivered` | Mark messages as delivered |
| POST | `/chat/messages/mark-as-read` | Mark messages as read |
| DELETE | `/chat/delete/{offer_id}` | Soft-delete a chat conversation |
