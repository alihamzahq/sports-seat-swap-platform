# Database Schema (ERD)

The schema centers around **Users** who list **Seats** at sporting venues, linked to **Matches** from an external sports data pipeline. Users create **Swap Offers** to trade seats for specific matches. The system also manages **Subscriptions** (with plan tiers), **Chat Messages** (via external realtime DB), **Notifications**, and admin-managed reference data like **Countries**, **Leagues**, and **Teams**.

```mermaid
erDiagram
    USER {
        bigint id PK
        string first_name
        string last_name
        string email UK
        bigint role_id FK
        bigint country_id FK
        string stripe_id
        string documents_status
        boolean is_active
        boolean is_ban
    }

    ROLE {
        bigint id PK
        string name
    }

    COUNTRY {
        bigint id PK
        string name
        string code
        boolean is_popular
        boolean is_active
    }

    CITY {
        bigint id PK
        string name
        bigint country_id FK
    }

    LEAGUE {
        bigint id PK
        bigint country_id FK
        string name
        string season
        boolean is_popular
        boolean is_active
    }

    TEAM {
        bigint id PK
        string name
        string nationality
        bigint country_id FK
        bigint team_venue_id FK
        boolean is_popular
    }

    TEAM_VENUE {
        bigint id PK
        string venue_name
        bigint city_id FK
        integer capacity
    }

    VENUE {
        bigint id PK
        string name
        integer spectators
    }

    MATCH {
        bigint id PK
        bigint league_id FK
        bigint venue_id FK
        string status
        datetime date
    }

    MATCH_TEAM {
        bigint match_id FK
        bigint team_id FK
        boolean is_home_team
    }

    LEAGUE_TEAM {
        bigint league_id FK
        bigint team_id FK
    }

    SEAT {
        bigint id PK
        bigint user_id FK
        bigint country_id FK
        bigint league_id FK
        bigint team_id FK
    }

    SEAT_DETAIL {
        bigint id PK
        bigint seat_id FK
        bigint user_id FK
        bigint seat_type_id FK
        bigint food_type_id FK
        string stadium_name
        string stand
        string season
        date date_from
        date date_to
        string duration
        string status
    }

    SEAT_TYPE {
        bigint id PK
        string title
        string icon
    }

    FOOD_TYPE {
        bigint id PK
        string title
        string icon
    }

    SEAT_PHOTO {
        bigint id PK
        bigint seat_detail_id FK
        string image
    }

    SEAT_MATCH {
        bigint id PK
        bigint user_id FK
        bigint match_id FK
        bigint seat_detail_id FK
        string ticket
        boolean is_active
    }

    SWAP_OFFER {
        bigint id PK
        bigint sender_id FK
        bigint receiver_id FK
        bigint swapped_for FK
        bigint swapped_with FK
        string status
        boolean is_archived
        datetime expired_at
    }

    SWAP_OFFER_SEAT {
        bigint id PK
        bigint swap_offer_id FK
        bigint seat_id FK
    }

    SWAP_OFFER_SEAT_DETAIL {
        bigint id PK
        bigint offer_seat_id FK
        bigint seat_detail_id FK
    }

    SWAP_OFFER_MATCH {
        bigint id PK
        bigint offer_seat_detail_id FK
        bigint seat_match_id FK
    }

    SUBSCRIPTION {
        bigint id PK
        bigint user_id FK
        string stripe_id UK
        string stripe_status
        string stripe_price
        bigint country_id FK
        datetime ends_at
    }

    PLAN {
        bigint id PK
        string name
        string slug
        decimal monthly_price
        decimal half_season_price
        decimal full_season_price
        string stripe_product_id
    }

    INVOICE_DETAIL {
        bigint id PK
        bigint user_id FK
        string company_name
        string address
        string vat_number
    }

    DEVICE_TOKEN {
        bigint id PK
        bigint user_id FK
        text token
    }

    NOTIFICATION {
        uuid id PK
        string type
        bigint notifiable_id FK
        text data
        datetime read_at
    }

    CONTACT_MESSAGE {
        bigint id PK
        bigint user_id FK
        string subject
        text message
    }

    REPORTED_USER {
        bigint id PK
        bigint reported_by FK
        bigint reported_to FK
        string subject
        text issue
    }

    USER ||--o{ SEAT : "lists"
    USER ||--o{ SEAT_DETAIL : "owns"
    USER ||--o{ SEAT_MATCH : "assigns"
    USER ||--o{ SWAP_OFFER : "sends"
    USER ||--o{ SUBSCRIPTION : "subscribes"
    USER ||--o{ DEVICE_TOKEN : "registers"
    USER ||--o{ NOTIFICATION : "receives"
    USER ||--o{ INVOICE_DETAIL : "has"
    USER ||--o{ CONTACT_MESSAGE : "submits"
    USER ||--o{ REPORTED_USER : "reports"
    USER }o--|| ROLE : "has"
    USER }o--|| COUNTRY : "from"

    COUNTRY ||--o{ LEAGUE : "hosts"
    COUNTRY ||--o{ CITY : "contains"
    CITY ||--o{ TEAM_VENUE : "has"

    LEAGUE ||--o{ MATCH : "schedules"
    LEAGUE }o--o{ TEAM : "participates"

    TEAM }o--o{ MATCH : "plays in"
    TEAM }o--o| TEAM_VENUE : "home at"

    MATCH }o--o| VENUE : "held at"

    SEAT }o--|| COUNTRY : "in"
    SEAT }o--|| LEAGUE : "for"
    SEAT }o--|| TEAM : "supports"
    SEAT ||--o{ SEAT_DETAIL : "has"

    SEAT_DETAIL }o--|| SEAT_TYPE : "is type"
    SEAT_DETAIL }o--|| FOOD_TYPE : "includes"
    SEAT_DETAIL ||--o{ SEAT_PHOTO : "has"
    SEAT_DETAIL ||--o{ SEAT_MATCH : "available for"

    SEAT_MATCH }o--|| MATCH : "for"

    SWAP_OFFER }o--|| SEAT_MATCH : "swapped for"
    SWAP_OFFER }o--|| SEAT_MATCH : "swapped with"
    SWAP_OFFER ||--o{ SWAP_OFFER_SEAT : "includes"

    SWAP_OFFER_SEAT }o--|| SEAT : "references"
    SWAP_OFFER_SEAT ||--o{ SWAP_OFFER_SEAT_DETAIL : "has"

    SWAP_OFFER_SEAT_DETAIL }o--|| SEAT_DETAIL : "references"
    SWAP_OFFER_SEAT_DETAIL ||--o{ SWAP_OFFER_MATCH : "for"

    SWAP_OFFER_MATCH }o--|| SEAT_MATCH : "references"
```

## Table Summary

| Domain | Tables | Description |
|--------|--------|-------------|
| Users | `users`, `roles`, `device_tokens` | User accounts, roles, push tokens |
| Geography | `countries`, `cities` | Location reference data |
| Sports | `leagues`, `teams`, `matches`, `venues`, `team_venues` | External sports data |
| Pivot | `match_team`, `league_team` | Many-to-many relationships |
| Seats | `seats`, `seat_details`, `seat_types`, `seat_food_types`, `seat_photos`, `seat_matches` | User seat listings |
| Swaps | `swap_offers`, `swap_offer_seats`, `swap_offer_seat_details`, `swap_offer_matches` | Swap negotiation chain |
| Billing | `subscriptions`, `subscription_items`, `plans`, `invoice_customer_details` | Payment & subscription management |
| System | `notifications`, `jobs`, `job_batches`, `failed_jobs`, `contact_messages`, `reported_users` | Platform operations |
