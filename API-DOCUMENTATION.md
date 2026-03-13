# Unishare Backend — API Documentation

> **Base URL (Development):** `http://localhost:4000`

---

## Table of Contents

- [Auth](#auth-auth)
- [Upload](#upload-upload)
- [Share Ride](#share-ride-apishareride)
- [Ticket Sell](#ticket-sell-apiticketsell)
- [Marketplace](#marketplace-apiitemsell)
- [Lost & Found](#lost--found-apilostfound)
- [Housing](#housing-apirooms)
- [Announcements](#announcements-apiannouncements)
- [Notice](#notice-apinotice)
- [Contacts](#contacts-apicontacts)
- [Resources](#resources-apiresources)
- [Notifications](#notifications-apinotifications)
- [Profile](#profile-apiprofile)
- [Admin](#admin-admin)

---

## Auth (`/auth`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/auth/google` | No | Start Google OAuth |
| GET | `/auth/google/callback` | No | Google OAuth callback |
| GET | `/auth/me` | No | Get current logged-in user |
| GET | `/auth/logout` | No | Logout & destroy session |
| GET | `/auth/health` | No | Auth health check |
| POST | `/auth/login` | No | Email/password login |
| POST | `/auth/register` | No | Email/password registration |

---

## Upload (`/upload`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/upload/item-image` | Yes | Upload item image to Supabase Storage |
| DELETE | `/upload/item-image` | Yes | Delete item image from storage |

---

## Share Ride (`/api/shareride`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/shareride` | Optional | Search/list active rides |
| GET | `/api/shareride/my` | Yes | Get my posted rides |
| GET | `/api/shareride/my/requests` | Yes | Requests received on my rides (driver) |
| GET | `/api/shareride/my/requested` | Yes | Requests I sent (passenger) |
| GET | `/api/shareride/requests/sent` | Yes | Alias for `/my/requested` |
| GET | `/api/shareride/stats` | Yes | User's ride statistics |
| GET | `/api/shareride/:id` | Optional | Single ride details |
| POST | `/api/shareride/create` | Yes | Create new ride |
| POST | `/api/shareride/:id/request` | Yes | Request to join a ride |
| PUT | `/api/shareride/requests/:requestId/respond` | Yes | Respond to ride request (confirm/decline) |
| PUT | `/api/shareride/:id` | Yes | Full update of a ride |
| PATCH | `/api/shareride/:id` | Yes | Partial update of a ride |
| DELETE | `/api/shareride/:id` | Yes | Delete a ride |
| DELETE | `/api/shareride/requests/:requestId` | Yes | Cancel a ride request |

---

## Ticket Sell (`/api/ticketsell`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/ticketsell` | Optional | List/search active tickets |
| GET | `/api/ticketsell/my` | Yes | Get my tickets |
| GET | `/api/ticketsell/my/requests` | Yes | Requests received (seller) |
| GET | `/api/ticketsell/requests/sent` | Yes | Requests I sent (buyer) |
| GET | `/api/ticketsell/stats` | Yes | Ticket statistics |
| GET | `/api/ticketsell/:id` | Optional | Single ticket details |
| POST | `/api/ticketsell/create` | Yes | Create new ticket listing |
| POST | `/api/ticketsell/:id/inquire` | Yes | Send inquiry about a ticket |
| POST | `/api/ticketsell/:id/request` | Yes | Request to buy tickets |
| PUT | `/api/ticketsell/:id` | Yes | Update ticket listing |
| PUT | `/api/ticketsell/requests/:requestId/respond` | Yes | Respond to request (accept/reject) |
| DELETE | `/api/ticketsell/:id` | Yes | Delete ticket listing |
| DELETE | `/api/ticketsell/requests/:requestId` | Yes | Cancel a ticket request |

---

## Marketplace (`/api/itemsell`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/itemsell` | Optional | List/search items |
| GET | `/api/itemsell/mine` | Yes | Get my items |
| GET | `/api/itemsell/my/requests` | Yes | Requests received (seller) |
| GET | `/api/itemsell/requests/sent` | Yes | Requests I sent (buyer) |
| GET | `/api/itemsell/:id` | Optional | Single item details |
| POST | `/api/itemsell` | Yes | Create new item (with image upload) |
| POST | `/api/itemsell/:id/request` | Yes | Request to buy an item |
| PUT | `/api/itemsell/:id` | Yes | Update item (with optional new image) |
| PUT | `/api/itemsell/requests/:requestId/respond` | Yes | Respond to request (accept/reject) |
| DELETE | `/api/itemsell/:id` | Yes | Delete item and image |
| DELETE | `/api/itemsell/requests/:requestId` | Yes | Cancel an item request |

---

## Lost & Found (`/api/lostfound`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/lostfound` | Optional | List/search lost & found items |
| GET | `/api/lostfound/my` | Yes | Get my lost/found items |
| GET | `/api/lostfound/my/requests` | Yes | Requests received (owner) |
| GET | `/api/lostfound/requests/sent` | Yes | Requests I sent (claimant) |
| GET | `/api/lostfound/stats` | Yes | Lost/found statistics |
| GET | `/api/lostfound/:id` | Optional | Single item details |
| POST | `/api/lostfound/create` | Yes | Create new item (multi-image upload) |
| POST | `/api/lostfound/:id/contact` | Yes | Send contact message |
| POST | `/api/lostfound/:id/request` | Yes | Claim found / report sighting |
| PUT | `/api/lostfound/:id` | Yes | Update item (resolve, etc.) |
| PUT | `/api/lostfound/requests/:requestId/respond` | Yes | Respond to request (accept/reject) |
| DELETE | `/api/lostfound/:id` | Yes | Delete item and images |
| DELETE | `/api/lostfound/requests/:requestId` | Yes | Cancel a request |

---

## Housing (`/api/rooms`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/rooms` | No | List rooms with filters/pagination |
| GET | `/api/rooms/mine` | Yes | Get my rooms |
| GET | `/api/rooms/my/requests` | Yes | Requests received (owner) |
| GET | `/api/rooms/requests/sent` | Yes | Requests I sent (tenant) |
| GET | `/api/rooms/:id` | No | Single room details |
| POST | `/api/rooms` | Yes | Create room listing (with photo upload) |
| POST | `/api/rooms/:id/request` | Yes | Request a room |
| PUT | `/api/rooms/:id` | Yes | Update room listing |
| PUT | `/api/rooms/requests/:requestId/respond` | Yes | Respond to room request |
| DELETE | `/api/rooms/:id` | Yes | Delete room and photos |
| DELETE | `/api/rooms/requests/:requestId` | Yes | Cancel a room request |

---

## Announcements (`/api/announcements`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/announcements` | No | Get active announcements |
| GET | `/api/announcements/my` | Yes | Get my announcements |
| GET | `/api/announcements/:id` | No | Get specific announcement |
| POST | `/api/announcements` | Yes | Submit announcement (needs admin approval) |
| PUT | `/api/announcements/:id` | Yes | Update own announcement |
| DELETE | `/api/announcements/:id` | Yes | Delete own announcement |

---

## Notice (`/api/notice`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/notice` | No | Get active notices (read-only) |

---

## Contacts (`/api/contacts`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/contacts` | No | Get contacts (filterable by category) |
| GET | `/api/contacts/:id` | No | Get specific contact |
| GET | `/api/contacts/search/:term` | No | Search contacts by name/role/category |

---

## Resources (`/api/resources`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/resources` | No | Get resources (filterable by category) |
| GET | `/api/resources/my` | Yes | Get my resource suggestions |
| GET | `/api/resources/categories` | No | Get unique resource categories |
| GET | `/api/resources/:id` | No | Get specific resource |
| POST | `/api/resources/suggest` | Yes | Submit resource suggestion (admin approval) |
| PUT | `/api/resources/my/:id` | Yes | Update own resource suggestion |
| DELETE | `/api/resources/my/:id` | Yes | Delete own resource suggestion |

---

## Notifications (`/api/notifications`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/notifications` | Yes | Get user's notifications |
| GET | `/api/notifications/unread-count` | Yes | Get unread count |
| PATCH | `/api/notifications/:id/read` | Yes | Mark as read |
| PATCH | `/api/notifications/mark-all-read` | Yes | Mark all as read |
| PATCH | `/api/notifications/public/:id/read` | No | Mark as read (public, needs `user_id` in body) |
| PATCH | `/api/notifications/public/:id/toggle` | No | Toggle read/unread (public, needs `user_id` in body) |
| DELETE | `/api/notifications/:id` | Yes | Delete notification |

---

## Profile (`/api/profile`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/profile` | Yes | Get current user's profile |
| GET | `/api/profile/search/:query` | No (rate-limited) | Search users by name or custom ID |
| GET | `/api/profile/check-availability/:customUserId` | No (rate-limited) | Check if custom user ID is available |
| GET | `/api/profile/suggest-username/:baseUsername` | No (rate-limited) | Suggest available usernames |
| GET | `/api/profile/:customUserId` | No (rate-limited) | Get public profile by custom user ID |
| POST | `/api/profile` | Yes | Create or update profile (with optional image) |
| PUT | `/api/profile` | Yes | Update profile (delegates to POST) |
| DELETE | `/api/profile/image` | Yes | Delete profile image |

---

## Admin (`/admin`)

> **All admin routes require admin role.**

### Core Admin

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/admin/users` | Get all users with activity counts |
| GET | `/admin/analytics` | Platform analytics |
| GET | `/admin/reports` | Content reports / moderation |
| GET | `/admin/activity` | Recent platform activity feed |
| GET | `/admin/cleanup/status` | Cleanup service status |

### Admin Module Routes

Each service has a full set of admin CRUD routes mounted at:

| Module | Base Path |
|--------|-----------|
| Share Ride | `/admin/shareride` |
| Ticket Sell | `/admin/ticketsell` |
| Marketplace | `/admin/itemsell` |
| Housing | `/admin/rooms` |
| Lost & Found | `/admin/lostfound` |
| Announcements | `/admin/announcements` |
| Resources | `/admin/resources` |
| Contacts | `/admin/contacts` |
| Notice | `/admin/notice` |
| Notifications | `/admin/notifications` |

---

## Notes

- **Auth:** "Yes" means the request must include a valid session cookie. "Optional" means the endpoint works for both authenticated and unauthenticated users (with richer data for authenticated ones).
- **Owner/Admin:** PUT and DELETE on user-created content typically require the requester to be the original creator or an admin.
- **Rate-limited:** Some public profile endpoints are rate-limited to prevent abuse.
