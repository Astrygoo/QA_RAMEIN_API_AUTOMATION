# Ramein Backend API Contract (Reverse-Engineered)

**Version:** 0.1  
**Source:** Ramein backend source code  
**Base URL:** `https://ramein.fun/service/api/v1`

> This contract was reconstructed from the backend implementation (`routes`, `controllers`, `services`, middleware, and the existing `api_doc.json`). It describes the API that is actually implemented, rather than inventing a separate API design.

---

## 1. General API Convention

### Request

JSON endpoints use:

```http
Content-Type: application/json
```

Protected endpoints require:

```http
Authorization: Bearer <access_token>
```

### Success response

The backend uses a common response envelope:

```json
{
  "success": true,
  "message": "Human readable message",
  "data": {},
  "meta": null
}
```

`data` depends on the endpoint.

### Error response

```json
{
  "success": false,
  "message": "Error message"
}
```

Common validation/authentication responses:

- `400` — Bad request / validation / business rule error
- `401` — Unauthorized / invalid token
- `403` — Forbidden / insufficient role
- `404` — Resource not found
- `409` — Conflict, e.g. duplicate email
- `500` — Internal server error

---

# 2. Authentication

## POST `/auth/first-user`

Creates the first user and assigns the `admin` role.

**Authentication:** None

### Request

```json
{
  "name": "Initial Admin",
  "email": "firstadmin@example.com",
  "password": "password123"
}
```

### Required fields

| Field | Type | Rule |
|---|---|---|
| name | string | required |
| email | string | valid email |
| password | string | minimum 6 characters |

### Success

`201 Created`

```json
{
  "success": true,
  "message": "First user initialized",
  "data": {
    "id": "uuid",
    "name": "Initial Admin",
    "email": "firstadmin@example.com",
    "role": "admin"
  },
  "meta": null
}
```

---

## POST `/auth/register`

Registers a normal user.

**Authentication:** None

### Request

```json
{
  "name": "User Test",
  "email": "user@example.com",
  "password": "password123",
  "phone": "08123456789"
}
```

### Required

- `name`
- `email`
- `password`

`phone` is optional.

### Success

`201 Created`

Returns the created user's:

- `id`
- `name`
- `email`
- `role`

---

## POST `/auth/login`

Authenticates a user/admin.

**Authentication:** None

### Request

```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

### Required

- `email`
- `password`

### Success

`200 OK`

Returns the login result including an access token and refresh token.

The Postman collection stores:

```text
data.accessToken
data.user.id
```

for subsequent requests.

---

## POST `/auth/google`

Google authentication.

**Authentication:** None

### Request

Either:

```json
{
  "credential": "<GOOGLE_ID_TOKEN>"
}
```

or:

```json
{
  "idToken": "<GOOGLE_ID_TOKEN>"
}
```

At least one is required.

### Success

- `200 OK` — existing user
- `201 Created` — new user

---

## POST `/auth/google/register`

Alias of Google authentication.

**Authentication:** None

Same request and response behavior as `/auth/google`.

---

## POST `/auth/google/login`

Alias of Google authentication.

**Authentication:** None

Same request and response behavior as `/auth/google`.

---

## POST `/auth/refresh-token`

Refreshes an access token.

**Authentication:** None

### Request

```json
{
  "refreshToken": "<REFRESH_TOKEN>"
}
```

### Success

`200 OK`

---

## POST `/auth/logout`

Logs out the current session at API level.

**Authentication:** None in the current implementation.

### Success

`200 OK`

```json
{
  "success": true,
  "message": "Logout success",
  "data": null,
  "meta": null
}
```

---

# 3. Users

## GET `/users/me`

Returns the authenticated user's profile.

**Authentication:** User/Admin

`200 OK`

---

## PATCH `/users/me`

Updates the current user's profile.

**Authentication:** User/Admin

### Request

```json
{
  "name": "Updated Name",
  "phone": "08123456789"
}
```

Both fields are optional, but at least one meaningful update should be supplied.

### Success

`200 OK`

---

## GET `/users/admin/list`

Returns all users.

**Authentication:** Admin

`200 OK`

---

## GET `/users/admin/admins`

Returns users with the `admin` role.

**Authentication:** Admin

`200 OK`

---

## POST `/users/admin/admins`

Creates an admin user.

**Authentication:** Admin

### Request

```json
{
  "name": "Admin Baru",
  "email": "admin@example.com",
  "password": "secret123",
  "phone": "081299998888"
}
```

Required:

- `name`
- `email`
- `password`

Optional:

- `phone`

### Success

`201 Created`

---

## GET `/users/admin/users`

Returns users with the `user` role.

**Authentication:** Admin

`200 OK`

---

# 4. Master Data

The same route pattern is used for:

- `categories`
- `cities`
- `organizers`

## GET `/master/{resource}`

Public list endpoint.

Example:

```http
GET /master/categories
GET /master/cities
GET /master/organizers
```

**Authentication:** None

---

## GET `/master/{resource}/{id}`

Returns one master-data record.

**Authentication:** None

`id` must be a UUID.

---

## POST `/master/{resource}`

Creates a category/city/organizer.

**Authentication:** Admin

### Common request fields

```json
{
  "name": "Example"
}
```

Optional fields:

```json
{
  "provinsi": "Jawa Barat",
  "description": "Description",
  "contactName": "Contact Person",
  "contactEmail": "contact@example.com",
  "contactPhone": "08123456789"
}
```

`contactEmail`, when supplied, must be a valid email.

### Success

`201 Created`

---

## PATCH `/master/{resource}/{id}`

Updates master data.

**Authentication:** Admin

Supported fields:

- `name`
- `provinsi`
- `description`
- `contactName`
- `contactEmail`
- `contactPhone`
- `isActive`

All are optional.

---

## PUT `/master/{resource}/{id}`

Same implementation as PATCH.

**Authentication:** Admin

---

## DELETE `/master/{resource}/{id}`

Deactivates the master-data record.

**Authentication:** Admin

`200 OK`

> Note: the backend calls this operation "deactivated"; it is not necessarily a physical database delete.

---

# 5. Events

## POST `/events`

Creates an event.

**Authentication:** User/Admin

### Request

```json
{
  "title": "Konser Akhir Tahun",
  "description": "Konser musik dengan banyak artis.",
  "categoryId": "<UUID>",
  "organizerId": "<UUID>",
  "cityId": "<UUID>",
  "addressDetail": "Gate A, lantai 1",
  "paymentType": "paid",
  "startDateTime": "2030-12-20T19:00:00.000Z",
  "endDateTime": "2030-12-20T22:00:00.000Z",
  "ticketTypes": [
    {
      "name": "Regular",
      "price": 150000,
      "quota": 100,
      "saleStartAt": "2030-11-01T00:00:00.000Z",
      "saleEndAt": "2030-12-20T18:00:00.000Z"
    }
  ]
}
```

### Required fields

- `title`
- `description`
- `categoryId`
- `organizerId`
- `cityId`
- `addressDetail`
- `startDateTime`
- `endDateTime`
- `ticketTypes`

### Optional fields

- `banner`
- `eventType`
- `labelOnline`
- `urlOnline`
- `paymentType`

`paymentType` accepts:

```text
free
paid
```

### Ticket type rules

Each ticket type requires:

- `name`
- `price` >= 0
- `quota` >= 0
- `saleStartAt`
- `saleEndAt`

### Success

`201 Created`

---

## GET `/events`

Lists events.

**Authentication:** Optional.

Supported query parameters:

```text
status
organizerId
search
createdBy
```

`createdBy=me` requires authentication.

---

## GET `/events/me`

Lists events created by the authenticated user.

**Authentication:** User/Admin

Supports:

```text
status
organizerId
search
```

---

## GET `/events/me/purchased`

Lists purchased events.

### Query

```text
userId=<UUID>
```

`userId` is required and must be a UUID.

**Authentication:** None in current route implementation.

---

## GET `/events/trending`

Returns trending/public events.

**Authentication:** None

---

## GET `/events/explore`

Public event exploration/filter endpoint.

**Authentication:** None

Supported query parameters include:

```text
search
q
category
kategory
categoryName
category_name
categoryId
category_id
wilayah
provinsi
province
region
kota
city
cityName
city_name
cityId
city_id
date
eventDate
event_date
startDate
start_date
```

---

## GET `/events/recommended`

Returns recommended events.

**Authentication:** None

Query:

```text
interest=<category-name>
```

`category` is also supported by the service.

If no category is supplied, the service defaults to `"Konser"`.

---

## GET `/events/interest`

Returns events matching selected interests.

**Authentication:** None

Example:

```http
GET /events/interest?categories=Music,Workshop
```

Supported aliases include:

```text
categories
category
kategory
interest
interests
```

---

## GET `/events/{id}`

Returns event detail.

**Authentication:** None

`id` must be UUID.

### Not found

`404 Event not found`

---

## PATCH `/events/{id}`

Updates an event.

**Authentication:** User/Admin

Only the event creator or admin can manage the event.

### Supported fields

Same event fields as create, plus:

```text
status
isPublished
is_published
ticketTypes
ticket_types
```

Supported status values:

```text
draft
pending
published
rejected
completed
cancelled
```

The backend accepts both camelCase and snake_case aliases for several fields.

---

## PUT `/events/{id}`

Same implementation as PATCH.

**Authentication:** User/Admin

---

## DELETE `/events/{id}`

Deletes an event.

**Authentication:** User/Admin

Only the creator or admin can delete it.

The backend rejects deletion with `400` when the event already has purchases.

---

# 6. Creator

## GET `/creators/{id}`

Returns creator information.

**Authentication:** None

`id` must be UUID.

`200 OK`

---

# 7. Event Chat

## POST `/event-chat`

Sends a message to the event assistant.

**Authentication:** None

### Request

```json
{
  "message": "What events are available?",
  "visitorName": "Astry"
}
```

Required:

- `message`

Optional:

- `visitorName`

### Success

`200 OK`

---

# 8. Transactions

## POST `/transactions`

Creates a transaction.

**Authentication:** User/Admin

### Request

```json
{
  "eventId": "<EVENT_UUID>",
  "items": [
    {
      "ticketTypeId": "<TICKET_TYPE_UUID>",
      "quantity": 2
    }
  ]
}
```

Required:

- `eventId` — UUID
- `items` — array, minimum 1 item
- `items[].ticketTypeId` — UUID
- `items[].quantity` — integer >= 1

### Success

`201 Created`

---

## GET `/transactions/me`

Returns transactions belonging to the authenticated user.

**Authentication:** User/Admin

`200 OK`

---

## GET `/transactions/statistic/event/{event_id}`

Returns transaction statistics for an event.

**Authentication:** User/Admin

`event_id` must be UUID.

`200 OK`

---

## GET `/transactions/admin/all`

Returns all transactions.

**Authentication:** Admin

`200 OK`

---

# 9. Payments

## POST `/payments/midtrans/notification`

Receives Midtrans payment notification.

**Authentication:** None

The existing Postman documentation uses:

```json
{
  "order_id": "<ORDER_ID>",
  "status_code": "200",
  "gross_amount": "<GROSS_AMOUNT>",
  "transaction_status": "settlement",
  "fraud_status": "accept",
  "signature_key": "<SIGNATURE>"
}
```

### Success

`200 OK`

> This endpoint should be tested carefully because a valid Midtrans notification depends on a real transaction/signature. For CI testing, do not put real production credentials or secrets in the repository.

---

# 10. Tickets

## GET `/ticket`

Returns the authenticated user's tickets.

**Authentication:** User/Admin

`200 OK`

---

## GET `/ticket/me`

Alias of `/ticket`.

**Authentication:** User/Admin

---

## GET `/ticket/event-ticket/{eventId}/{status?}`

Returns tickets for an event.

**Authentication:** User/Admin

`eventId` must be UUID.

The optional status is passed to the service.

Examples:

```http
GET /ticket/event-ticket/<EVENT_ID>/all
GET /ticket/event-ticket/<EVENT_ID>/hadir
GET /ticket/event-ticket/<EVENT_ID>/tidak_hadir
```

---

## POST `/ticket/qr-code/scan`

Scans a QR code.

**Authentication:** User/Admin

### Request

```json
{
  "qrCode": "TICKET_QR_CODE"
}
```

`qrCode` is required.

---

## POST `/ticket/qr-code/{qrCode}/scan`

QR code scan using a path parameter.

**Authentication:** User/Admin

---

# 11. Withdraw

## POST `/withdraw`

Creates a withdrawal request.

**Authentication:** User/Admin

### Request

The backend accepts camelCase and snake_case aliases.

Example:

```json
{
  "event_id": "<EVENT_UUID>",
  "total_amount": 120000,
  "bank_name": "BCA",
  "bank_account": "Account Owner",
  "account_number": "1234567890"
}
```

Required:

- `eventId` OR `event_id`
- `totalAmount` OR `total_amount`

Optional:

- `bankName` / `bank_name`
- `bankAccount` / `bank_account`
- `accountNumber` / `account_number`

`totalAmount` must be >= 0.

---

## GET `/withdraw/me`

Returns withdrawal requests belonging to the authenticated user.

**Authentication:** User/Admin

---

## GET `/withdraw/all`

Returns all withdrawals.

**Authentication:** Admin

---

## POST `/withdraw/status`

Updates withdrawal status.

**Authentication:** Admin

### Request

```json
{
  "id": "<WITHDRAW_UUID>",
  "status": "approved"
}
```

`id` or `withdraw_id` is required.

Allowed status:

```text
pending
approved
rejected
```

---

# 12. Finance

## GET `/finance/{published_by}/{organizer_id?}`

Returns finance data.

**Authentication:** Admin

`published_by` must be:

```text
admin
user
```

`organizer_id` is optional but, when supplied, must be UUID.

Examples:

```http
GET /finance/admin
GET /finance/user
GET /finance/user/<ORGANIZER_UUID>
GET /finance/admin/<ORGANIZER_UUID>
```

---

# 13. Event-Me

## GET `/event-me/me`

Returns events associated with the authenticated user.

**Authentication:** User/Admin

---

## GET `/event-me/me/purchased`

Returns events purchased by the authenticated user.

**Authentication:** User/Admin

---

# 14. Feedback

## GET `/feedback`

Returns feedback list.

**Authentication:** None

Optional query:

```text
rating
```

Allowed values:

```text
Sangat Puas
Puas
Cukup Puas
Tidak Puas
Sangat Tidak Puas
```

---

## GET `/feedback/{id}`

Returns feedback detail.

**Authentication:** None

`id` must be UUID.

---

## POST `/feedback`

Creates feedback.

**Authentication:** User/Admin

### Request

```json
{
  "rating": "Puas",
  "review": "Aplikasinya mudah dipakai."
}
```

`rating` is required.

`review` is optional and may be null.

---

## DELETE `/feedback/{id}`

Deletes feedback.

**Authentication:** Admin

`id` must be UUID.

---

# 15. Creator Feedback

These endpoints exist in the backend but were missing from the existing `api_doc.json` Postman collection.

## GET `/feedback-creators`

Lists creator feedback.

**Authentication:** None

Optional query parameters:

```text
rating
creatorType
creator_type
creatorId
creator_id
```

`rating` must be integer `1-5`.

`creatorType` must be:

```text
organizer
user
```

---

## GET `/feedback-creators/creator/{creatorId}`

Gets feedback for a specific creator.

**Authentication:** None

`creatorId` must be UUID.

Optional:

```text
creatorType
creator_type
```

---

## GET `/feedback-creators/{id}`

Gets creator feedback detail.

**Authentication:** None

`id` must be UUID.

---

## POST `/feedback-creators`

Creates creator feedback.

**Authentication:** User/Admin

### Request

```json
{
  "rating": 5,
  "review": "Organizer sangat responsif.",
  "creatorType": "organizer",
  "creatorId": "<CREATOR_UUID>"
}
```

Required:

- `rating`
- `creatorType`
- `creatorId`

`rating` must be integer `1-5`.

`creatorType` must be `organizer` or `user`.

---

## PATCH `/feedback-creators/{id}`

Updates creator feedback.

**Authentication:** User/Admin

At least one of:

- `rating`
- `review`

must be supplied.

---

## PUT `/feedback-creators/{id}`

Same implementation as PATCH.

**Authentication:** User/Admin

---

## DELETE `/feedback-creators/{id}`

Deletes creator feedback.

**Authentication:** User/Admin

---

# 16. Health Check

## GET `/health`

Returns API health status.

**Authentication:** None

### Success

`200 OK`

```json
{
  "success": true,
  "message": "OK",
  "data": null,
  "meta": null
}
```

---

# 17. Authentication Matrix

| Module | Public | User | Admin |
|---|---:|---:|---:|
| Health | ✓ | ✓ | ✓ |
| Auth | ✓ | ✓ | ✓ |
| User profile | | ✓ | ✓ |
| User administration | | | ✓ |
| Master-data read | ✓ | ✓ | ✓ |
| Master-data write | | | ✓ |
| Event public read | ✓ | ✓ | ✓ |
| Event create/update/delete | | ✓ | ✓ |
| Transactions | | ✓ | ✓ |
| Admin transactions | | | ✓ |
| Tickets | | ✓ | ✓ |
| Withdraw | | ✓ | ✓ |
| Admin withdrawals | | | ✓ |
| Finance | | | ✓ |
| Feedback read | ✓ | ✓ | ✓ |
| Feedback create | | ✓ | ✓ |
| Feedback delete | | | ✓ |
| Creator feedback read | ✓ | ✓ | ✓ |
| Creator feedback write | | ✓ | ✓ |

