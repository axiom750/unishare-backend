# Unishare — Database Schema Documentation

> **Database:** PostgreSQL via Supabase  
> **Total Tables:** 22 (+ 4 admin view aliases)

---

## Table of Contents

- [users](#1-users)
- [user_profiles](#2-user_profiles)
- [shared_rides](#3-shared_rides)
- [ride_requests](#4-ride_requests)
- [tickets](#5-tickets)
- [ticket_views](#6-ticket_views)
- [ticket_inquiries](#7-ticket_inquiries)
- [ticket_requests](#8-ticket_requests)
- [item_sell](#9-item_sell)
- [item_requests](#10-item_requests)
- [lost_found_items](#11-lost_found_items)
- [lostfound_contacts](#12-lostfound_contacts)
- [lostfound_requests](#13-lostfound_requests)
- [rooms](#14-rooms)
- [room_requests](#15-room_requests)
- [announcements](#16-announcements)
- [notices](#17-notices)
- [contacts](#18-contacts)
- [resources](#19-resources)
- [notifications](#20-notifications)
- [user_notification_reads](#21-user_notification_reads)
- [reports](#22-reports)
- [Supabase Storage Buckets](#supabase-storage-buckets)
- [Entity Relationship Diagram](#entity-relationship-diagram)

---

## 1. `users`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | text | **PK** | Format: `user_{timestamp}_{random}` (email) or Google provider ID |
| `name` | text | NOT NULL | `{firstName} {lastName}` |
| `email` | text | UNIQUE, NOT NULL | Lowercased on insert |
| `password` | text | nullable | Bcrypt hash; null for Google OAuth users |
| `picture` | text | nullable | Google profile picture URL |
| `university` | text | nullable | Optional during registration |
| `role` | text | default `'user'` | `'user'` or `'admin'` |
| `is_active` | boolean | default `true` | Account status |
| `created_at` | timestamptz | NOT NULL | ISO string |
| `updated_at` | timestamptz | | ISO string |
| `last_login` | timestamptz | | Set on registration and login |

**Relationships:** Referenced as FK by nearly every other table via `user_id`.

---

## 2. `user_profiles`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `user_id` | text | **PK**, FK → `users.id` | Upsert conflict column |
| `display_name` | text | nullable | Searchable via ilike |
| `bio` | text | nullable | |
| `phone_number` | text | nullable | 10–15 digits validated |
| `campus_name` | text | nullable | 2–100 chars validated |
| `profile_image_url` | text | nullable | URL from `profile-images` bucket |
| `custom_user_id` | text | UNIQUE, nullable | Format: `@[a-zA-Z0-9_]{2,20}` |
| `created_at` | timestamptz | | |
| `updated_at` | timestamptz | | |

---

## 3. `shared_rides`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | uuid | **PK** | Auto-generated |
| `user_id` | text | FK → `users.id`, NOT NULL | Ride poster |
| `driver_name` | text | NOT NULL | Defaults to user name or `'Anonymous'` |
| `from_location` | text | NOT NULL | Searchable via ilike |
| `to_location` | text | NOT NULL | Searchable via ilike |
| `date` | date | NOT NULL | Filterable with eq/gt/lt |
| `time` | time | NOT NULL | Filterable with gt |
| `seats` | integer | NOT NULL | Total seats |
| `available_seats` | integer | NOT NULL | Initially equals `seats`; decremented on confirm |
| `price` | numeric | NOT NULL | Filterable with gte/lte |
| `vehicle_info` | text | NOT NULL | |
| `description` | text | nullable | |
| `contact_info` | jsonb | NOT NULL | Object with contact type → value mappings |
| `status` | text | NOT NULL, default `'active'` | `'active'`, `'completed'`, `'cancelled'` |
| `created_at` | timestamptz | NOT NULL | |
| `updated_at` | timestamptz | | |

**Relationships:**
- `user_id` → `users.id`
- Referenced by `ride_requests.ride_id`

---

## 4. `ride_requests`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | uuid | **PK** | |
| `ride_id` | uuid | FK → `shared_rides.id`, NOT NULL | |
| `requester_id` | text | FK → `users.id`, NOT NULL | Passenger |
| `driver_id` | text | FK → `users.id`, NOT NULL | Ride owner |
| `message` | text | NOT NULL | |
| `seats_requested` | integer | NOT NULL | |
| `contact_method` | text | NOT NULL | |
| `pickup_location` | text | nullable | |
| `status` | text | NOT NULL, default `'pending'` | `'pending'`, `'confirmed'`, `'declined'`, `'cancelled'` |
| `response_message` | text | nullable | Driver's response |
| `responded_at` | timestamptz | nullable | |
| `created_at` | timestamptz | NOT NULL | |
| `updated_at` | timestamptz | | |

**Relationships:**
- `ride_id` → `shared_rides.id` (alias: `ride:ride_id`)
- `requester_id` → `users.id` (alias: `requester:requester_id`)
- `driver_id` → `users.id` (alias: `driver:driver_id`)

---

## 5. `tickets`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | uuid | **PK** | |
| `user_id` | text | FK → `users.id`, NOT NULL | Seller |
| `title` | text | NOT NULL | Searchable via ilike |
| `price` | numeric | NOT NULL | Filterable with gte/lte |
| `category` | text | NOT NULL | `'event'`, `'travel'`, `'other'` |
| `event_type` | text | default `'other'` | Filterable |
| `event_date` | date | nullable | Sortable |
| `venue` | text | nullable | Searchable via ilike |
| `location` | text | nullable | Searchable via ilike |
| `quantity_available` | integer | NOT NULL | Decremented on accepted request |
| `ticket_type` | text | default `'Standard'` | |
| `description` | text | nullable | Searchable via ilike |
| `contact_info` | jsonb | NOT NULL | |
| `status` | text | NOT NULL, default `'active'` | `'active'`, `'sold'`, `'expired'` |
| `origin` | text | nullable | Travel category |
| `destination` | text | nullable | Travel category |
| `transport_mode` | text | nullable | Travel category |
| `travel_date` | date | nullable | Travel category |
| `item_type` | text | nullable | Other category field |
| `created_at` | timestamptz | NOT NULL | |
| `updated_at` | timestamptz | | |

**Relationships:**
- `user_id` → `users.id`
- Referenced by `ticket_views`, `ticket_inquiries`, `ticket_requests`

---

## 6. `ticket_views`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `ticket_id` | uuid | FK → `tickets.id` | Composite unique |
| `user_id` | text | FK → `users.id` | Composite unique |
| `viewed_at` | timestamptz | | Upserted on conflict |

**Constraints:** Composite unique on `(ticket_id, user_id)`

---

## 7. `ticket_inquiries`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | uuid | **PK** | |
| `ticket_id` | uuid | FK → `tickets.id`, NOT NULL | |
| `user_id` | text | FK → `users.id`, NOT NULL | Inquiring user |
| `message` | text | NOT NULL | |
| `status` | text | default `'pending'` | |

**Relationships:**
- `user_id` → `users.id` (alias: `users:user_id`)
- `ticket_id` → `tickets.id`

---

## 8. `ticket_requests`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | uuid | **PK** | |
| `ticket_id` | uuid | FK → `tickets.id`, NOT NULL | |
| `requester_id` | text | FK → `users.id`, NOT NULL | Buyer |
| `seller_id` | text | FK → `users.id`, NOT NULL | Ticket owner |
| `message` | text | NOT NULL | |
| `contact_method` | text | NOT NULL | |
| `quantity_requested` | integer | NOT NULL | |
| `offered_price` | numeric | nullable | |
| `pickup_preference` | text | nullable | |
| `status` | text | NOT NULL, default `'pending'` | `'pending'`, `'accepted'`, `'rejected'`, `'cancelled'` |
| `response_message` | text | nullable | |
| `agreed_price` | numeric | nullable | Set when accepted |
| `agreed_quantity` | integer | nullable | Set when accepted |
| `responded_at` | timestamptz | nullable | |
| `created_at` | timestamptz | NOT NULL | |
| `updated_at` | timestamptz | | |

**Relationships:**
- `requester_id` → `users.id` (alias: `requester:requester_id`)
- `seller_id` → `users.id` (alias: `seller:seller_id`)
- `ticket_id` → `tickets.id` (alias: `ticket:ticket_id`)

---

## 9. `item_sell`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | uuid | **PK** | |
| `user_id` | text | FK → `users.id`, NOT NULL | Seller |
| `title` | text | NOT NULL | Min 3 chars; searchable via ilike |
| `price` | numeric | NOT NULL | Filterable with gte/lte |
| `category` | text | NOT NULL | Lowercased; filterable |
| `condition` | text | NOT NULL | `'new'`, `'like-new'`, `'good'`, `'fair'`, `'damaged'` |
| `location` | text | NOT NULL | Searchable via ilike |
| `available_from` | date | NOT NULL | |
| `description` | text | nullable | Searchable via ilike |
| `photos` | jsonb (array) | default `[]` | Legacy field, always `[]` |
| `image_url` | text | nullable | Single image URL from storage |
| `contact_info` | jsonb | NOT NULL | |
| `created_at` | timestamptz | | |
| `updated_at` | timestamptz | | |

**Relationships:**
- `user_id` → `users.id`
- Referenced by `item_requests.item_id`

---

## 10. `item_requests`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | uuid | **PK** | |
| `item_id` | uuid | FK → `item_sell.id`, NOT NULL | |
| `requester_id` | text | FK → `users.id`, NOT NULL | Buyer |
| `seller_id` | text | FK → `users.id`, NOT NULL | Item owner |
| `message` | text | NOT NULL | |
| `contact_method` | text | NOT NULL | |
| `offered_price` | numeric | nullable | |
| `pickup_preference` | text | nullable | |
| `status` | text | NOT NULL, default `'pending'` | `'pending'`, `'accepted'`, `'rejected'`, `'cancelled'` |
| `response_message` | text | nullable | |
| `agreed_price` | numeric | nullable | |
| `responded_at` | timestamptz | nullable | |
| `created_at` | timestamptz | NOT NULL | |
| `updated_at` | timestamptz | | |

**Relationships:**
- `item_id` → `item_sell.id` (alias: `item:item_id`)
- `requester_id` → `users.id` (alias: `requester:requester_id`)
- `seller_id` → `users.id` (alias: `seller:seller_id`)

---

## 11. `lost_found_items`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | uuid | **PK** | |
| `user_id` | text | FK → `users.id`, NOT NULL | Reporter |
| `item_name` | text | NOT NULL | Searchable via ilike |
| `description` | text | NOT NULL | Searchable via ilike |
| `mode` | text | NOT NULL | `'lost'` or `'found'` |
| `where_last_seen` | text | nullable | For lost items; searchable |
| `where_found` | text | nullable | For found items; searchable |
| `date_lost` | date | nullable | For lost items |
| `time_lost` | time | nullable | For lost items |
| `date_found` | date | nullable | For found items |
| `time_found` | time | nullable | For found items |
| `contact_info` | jsonb | NOT NULL | |
| `image_urls` | jsonb (array) | default `[]` | Array of image URLs |
| `status` | text | NOT NULL, default `'active'` | `'active'`, `'resolved'`, `'expired'` |
| `resolution_notes` | text | nullable | Added on status update |
| `resolved_at` | timestamptz | nullable | Set when resolved |
| `created_at` | timestamptz | NOT NULL | |
| `updated_at` | timestamptz | | |

**Relationships:**
- `user_id` → `users.id`
- Referenced by `lostfound_contacts.item_id`, `lostfound_requests.item_id`

---

## 12. `lostfound_contacts`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | uuid | **PK** | |
| `item_id` | uuid | FK → `lost_found_items.id`, NOT NULL | |
| `from_user_id` | text | FK → `users.id`, NOT NULL | Contacting user |
| `to_user_id` | text | FK → `users.id`, NOT NULL | Item owner |
| `message` | text | NOT NULL | |
| `contact_method` | text | default `'message'` | |
| `status` | text | default `'pending'` | |

**Relationships:**
- `from_user_id` → `users.id` (alias: `from_user:from_user_id`)
- `item_id` → `lost_found_items.id` (alias: `item:item_id`)

---

## 13. `lostfound_requests`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | uuid | **PK** | |
| `item_id` | uuid | FK → `lost_found_items.id`, NOT NULL | |
| `requester_id` | text | FK → `users.id`, NOT NULL | Claimant / reporter |
| `owner_id` | text | FK → `users.id`, NOT NULL | Item poster |
| `message` | text | NOT NULL | |
| `contact_method` | text | NOT NULL | |
| `proof_description` | text | nullable | For claim verification |
| `status` | text | NOT NULL, default `'pending'` | `'pending'`, `'accepted'`, `'rejected'`, `'cancelled'` |
| `response_message` | text | nullable | |
| `responded_at` | timestamptz | nullable | |
| `created_at` | timestamptz | NOT NULL | |
| `updated_at` | timestamptz | | |

**Relationships:**
- `requester_id` → `users.id` (alias: `requester:requester_id`)
- `owner_id` → `users.id` (alias: `owner:owner_id`)
- `item_id` → `lost_found_items.id` (alias: `item:item_id`)

---

## 14. `rooms`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | uuid | **PK** | |
| `user_id` | text | FK → `users.id`, NOT NULL | Landlord |
| `title` | text | NOT NULL | |
| `description` | text | nullable | Max 1000 chars |
| `rent` | integer | NOT NULL | Filterable with gte/lte |
| `location` | text | NOT NULL | Searchable via ilike |
| `beds` | integer | NOT NULL | Filterable |
| `move_in_date` | date | NOT NULL | Validated to be non-past |
| `contact_info` | jsonb | | Keys: `mobile`, `email`, `instagram` |
| `photos` | jsonb (array) | default `[]` | Array of photo URLs |
| `created_at` | timestamptz | NOT NULL | |
| `updated_at` | timestamptz | | |

**Relationships:**
- `user_id` → `users.id`
- Referenced by `room_requests.room_id`

---

## 15. `room_requests`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | uuid | **PK** | |
| `room_id` | uuid | FK → `rooms.id`, NOT NULL | |
| `requester_id` | text | FK → `users.id`, NOT NULL | Prospective tenant |
| `landlord_id` | text | FK → `users.id`, NOT NULL | Room owner |
| `message` | text | NOT NULL | Min 10, max 1000 chars |
| `contact_method` | text | NOT NULL | |
| `move_in_date` | date | NOT NULL | Preferred move-in date |
| `stay_duration` | text | nullable | Max 100 chars |
| `occupants` | integer | default `1` | Range: 1–20 |
| `status` | text | NOT NULL, default `'pending'` | `'pending'`, `'approved'`, `'rejected'`, `'cancelled'` |
| `response_message` | text | nullable | |
| `responded_at` | timestamptz | nullable | |
| `created_at` | timestamptz | NOT NULL | |
| `updated_at` | timestamptz | | |

**Relationships:**
- `room_id` → `rooms.id` (alias: `room:room_id`)
- `requester_id` → `users.id` (alias: `requester:requester_id`)
- `landlord_id` → `users.id` (alias: `landlord:landlord_id`)

---

## 16. `announcements`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | integer | **PK**, auto-increment | |
| `title` | text | NOT NULL | Max 200 chars |
| `body` | text | NOT NULL | Max 2000 chars |
| `priority` | text | default `'normal'` | `'high'`, `'normal'`, `'low'` |
| `tags` | jsonb (array) | default `['general']` | Array of tag strings |
| `active` | boolean | default `false` | User submissions start inactive; admin activates |
| `user_id` | text | FK → `users.id` | Submitter |
| `created_at` | timestamptz | NOT NULL | |
| `updated_at` | timestamptz | | |

---

## 17. `notices`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | integer | **PK** | |
| `heading` | text | NOT NULL | |
| `body` | text | NOT NULL | |
| `priority` | integer/text | | Used for ordering |
| `active` | boolean | | Filtered: `active = true` |
| `user_id` | text | FK → `users.id` | Admin who created |
| `created_at` | timestamptz | NOT NULL | |
| `updated_at` | timestamptz | | |

---

## 18. `contacts`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | integer | **PK** | |
| `name` | text | NOT NULL | Searchable via ilike |
| `role` | text | | Searchable via ilike |
| `emails` | jsonb (array) | | Array of email strings |
| `phones` | jsonb (array) | | Array of phone strings |
| `category` | text | | Filterable |
| `location` | text | | |
| `hours` | text | | |
| `active` | boolean | | Filtered: `active = true` |
| `user_id` | text | FK → `users.id` | Admin who created |
| `created_at` | timestamptz | | |
| `updated_at` | timestamptz | | |

---

## 19. `resources`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | integer/uuid | **PK** | |
| `title` | text | NOT NULL | |
| `desc` | text | | Description |
| `url` | text | | Resource link |
| `category` | text | | Filterable |
| `type` | text | | |
| `tags` | jsonb (array) | | |
| `active` | boolean | default `false` | Suggestions start inactive; admin toggles |
| `user_id` | text | FK → `users.id` | Submitter |
| `created_at` | timestamptz | | |
| `updated_at` | timestamptz | | |

---

## 20. `notifications`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | integer | **PK**, auto-increment | |
| `title` | text | nullable | |
| `message` | text | NOT NULL | |
| `type` | text | NOT NULL | `'info'`, `'success'`, `'warning'`, `'error'` |
| `recipient_type` | text | NOT NULL | `'all'` (broadcast) or `'user'` (targeted) |
| `recipient_id` | text | FK → `users.id`, nullable | null when `recipient_type = 'all'` |
| `sender_id` | text | FK → `users.id` | Admin who sent it |
| `read` | boolean | default `false` | |
| `created_at` | timestamptz | NOT NULL | |
| `updated_at` | timestamptz | | |

**Relationships:**
- `recipient_id` → `users.id`
- `sender_id` → `users.id`
- Referenced by `user_notification_reads`

---

## 21. `user_notification_reads`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | integer/uuid | **PK** | |
| `user_id` | text | FK → `users.id` | Composite unique |
| `notification_id` | integer | FK → `notifications.id` | Composite unique |
| `read_at` | timestamptz | NOT NULL | |

**Constraints:** Composite unique on `(user_id, notification_id)` — duplicate key error `23505` handled.

---

## 22. `reports`

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | uuid/integer | **PK** | |
| `type` | text | | `'inappropriate_content'`, `'spam'`, `'harassment'`, `'scam'`, `'misinformation'` |
| `category` | text | | `'rideshare'`, `'marketplace'`, `'lost-found'`, `'user-profile'` |
| `priority` | text | | `'high'`, etc. |
| `status` | text | | `'pending'`, `'reviewing'`, `'resolved'`, `'dismissed'` |
| `description` | text | nullable | |
| `reason` | text | nullable | |
| `content_id` | text | nullable | ID of reported content |
| `screenshots` | jsonb (array) | default `[]` | |
| `reported_by` | text | FK → `users.id` | Reporter |
| `reported_user` | text | FK → `users.id` | Reported user |
| `resolved_by` | text | FK → `users.id`, nullable | Admin who resolved |
| `resolution_action` | text | nullable | |
| `resolution_notes` | text | nullable | |
| `resolved_at` | timestamptz | nullable | |
| `created_at` | timestamptz | NOT NULL | |
| `updated_at` | timestamptz | | |

**Relationships:**
- `reported_by` → `users.id` (alias: `reporter:reported_by`)
- `reported_user` → `users.id` (alias: `reported:reported_user`)
- `resolved_by` → `users.id` (alias: `resolver:resolved_by`)

---

## Admin Alternate Table Names

The admin route files use different table names that may be database views or aliases:

| Admin Route File | Table Name Used | Likely Maps To |
|------------------|-----------------|----------------|
| `routes/admin/shareride.js` | `shareride` | `shared_rides` |
| `routes/admin/ticketsell.js` | `ticketsell` | `tickets` |
| `routes/admin/itemsell.js` | `itemsell` | `item_sell` |
| `routes/admin/lostfound.js` | `lostfound` | `lost_found_items` |

---

## Supabase Storage Buckets

| Bucket Name | Used For | Source |
|-------------|----------|--------|
| `marketplace-items` | Item sell images | `routes/api/itemsell.js`, `routes/upload.js` |
| `lostfound-images` | Lost/found item images | `routes/api/lostfound.js` |
| `room-photos` | Room listing photos | `routes/api/rooms.js` |
| `profile-images` | User profile images | `routes/api/profile.js` |

---

## Entity Relationship Diagram

```
users (PK: id)
 │
 ├── user_profiles.user_id ──────────────── 1:1
 │
 ├── shared_rides.user_id ───────────────── 1:N
 │    └── ride_requests.ride_id ─────────── 1:N
 │         ├── .requester_id → users.id
 │         └── .driver_id → users.id
 │
 ├── tickets.user_id ────────────────────── 1:N
 │    ├── ticket_views (ticket_id, user_id) ── N:N
 │    ├── ticket_inquiries.ticket_id ────── 1:N
 │    │    └── .user_id → users.id
 │    └── ticket_requests.ticket_id ─────── 1:N
 │         ├── .requester_id → users.id
 │         └── .seller_id → users.id
 │
 ├── item_sell.user_id ──────────────────── 1:N
 │    └── item_requests.item_id ─────────── 1:N
 │         ├── .requester_id → users.id
 │         └── .seller_id → users.id
 │
 ├── lost_found_items.user_id ───────────── 1:N
 │    ├── lostfound_contacts.item_id ────── 1:N
 │    │    ├── .from_user_id → users.id
 │    │    └── .to_user_id → users.id
 │    └── lostfound_requests.item_id ────── 1:N
 │         ├── .requester_id → users.id
 │         └── .owner_id → users.id
 │
 ├── rooms.user_id ──────────────────────── 1:N
 │    └── room_requests.room_id ─────────── 1:N
 │         ├── .requester_id → users.id
 │         └── .landlord_id → users.id
 │
 ├── announcements.user_id ──────────────── 1:N
 ├── notices.user_id ────────────────────── 1:N
 ├── contacts.user_id ───────────────────── 1:N
 ├── resources.user_id ──────────────────── 1:N
 │
 ├── notifications
 │    ├── .recipient_id → users.id
 │    ├── .sender_id → users.id
 │    └── user_notification_reads ────────── N:N
 │         ├── .user_id → users.id
 │         └── .notification_id → notifications.id
 │
 └── reports
      ├── .reported_by → users.id
      ├── .reported_user → users.id
      └── .resolved_by → users.id
```
