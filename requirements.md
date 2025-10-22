# Requirements Specification — Airbnb Clone Backend

This file documents technical and functional specifications for three key backend features: User Authentication, Property Management, and Booking System. Each feature includes API endpoints, request/response shapes, validation rules, error codes, and performance criteria.

---

## 1. User Authentication

Purpose: manage user signup, login, profile, and roles (guest, host, admin).

Authentication mechanism: JWT (access token + refresh token). Passwords hashed with bcrypt.

Endpoints:

- POST /api/auth/register

  - Request body: { name, email, password, role }
  - Response (201): { id, name, email, role, accessToken, refreshToken }
  - Validation: email format, password >= 8 chars, role in ["guest","host"].
  - Errors: 400 (validation), 409 (email exists)

- POST /api/auth/login

  - Request body: { email, password }
  - Response (200): { accessToken, refreshToken, user: { id, name, email, role } }
  - Errors: 401 (invalid credentials)

- POST /api/auth/oauth

  - For OAuth logins (Google/Facebook). Accept provider token and return JWT on success.

- POST /api/auth/refresh

  - Request body: { refreshToken }
  - Response: { accessToken }

- GET /api/users/me

  - Headers: Authorization: Bearer <accessToken>
  - Response (200): { id, name, email, role, profilePhoto, phone }

- PUT /api/users/me
  - Update profile fields. Multipart/form-data when uploading profilePhoto.

Security & rules:

- Access tokens short-lived (e.g., 15m). Refresh tokens long-lived and revocable.
- Rate limit authentication endpoints (e.g., 5 attempts per minute per IP) to prevent brute force.

Performance:

- 95th percentile auth response time < 200ms under moderate load.

---

## 2. Property Management

Purpose: allow hosts to create, edit, list, and remove property listings.

Data model (simplified):

Property {
id: uuid,
host_id: uuid (FK -> users.id),
title: string,
description: text,
address: string,
location: { lat: float, lng: float },
price_per_night: decimal,
currency: string,
max_guests: int,
amenities: array[string],
photos: array[string] (URLs),
created_at, updated_at
}

Endpoints:

- POST /api/properties

  - Auth: Host only
  - Body: JSON + multipart for photos (or accept pre-signed URLs)
  - Response (201): property object
  - Validation: required fields: title, address, price_per_night, max_guests

- GET /api/properties

  - Query params: q (text), lat,lng,radius, price_min, price_max, guests, amenities, available_from, available_to, page, limit
  - Response (200): { results: [property], pagination: { page, limit, total } }

- GET /api/properties/:id

  - Response: property detail

- PUT /api/properties/:id

  - Auth: Host who owns listing or Admin
  - Update fields

- DELETE /api/properties/:id
  - Auth: Host who owns listing or Admin

Media handling:

- Use cloud storage (S3/Cloudinary). Prefer direct-to-cloud uploads (pre-signed URLs) to reduce server bandwidth.

Validation & constraints:

- Max images per listing (e.g., 50). Image size limit (e.g., 5MB).
- Price >= 0, max_guests > 0.

Performance:

- Search queries should use indexed columns (location, price) and full-text index for title/description.
- Cache popular search results in Redis for 60 seconds to reduce DB load.

---

## 3. Booking System

Purpose: handle booking lifecycle, availability checks, and payment coordination.

Data model (simplified):

Booking {
id: uuid,
property_id: uuid,
guest_id: uuid,
start_date: date,
end_date: date,
total_price: decimal,
currency: string,
status: enum(pending, confirmed, canceled, completed),
payment_status: enum(unpaid, paid, refunded),
created_at, updated_at
}

Endpoints:

- POST /api/bookings

  - Auth: Guest
  - Body: { property_id, start_date, end_date, guests, payment_method } or payment intent token
  - Flow:
    1. Validate dates and guests against property constraints.
    2. Check availability (no overlapping confirmed bookings).
    3. Calculate total price (rate \* nights + fees + taxes).
    4. Create booking record with status=pending.
    5. Create payment intent with payment provider (Stripe) and capture payment.
    6. On successful payment, set booking.status=confirmed, payment_status=paid.
  - Response (201): booking object
  - Errors: 400 (validation), 409 (dates unavailable), 402 (payment required/failed)

- GET /api/bookings/:id

  - Auth: Guest (owner), Host (owner's property), or Admin

- PUT /api/bookings/:id/cancel
  - Auth: Guest or Host
  - Enforce cancellation policy and compute refunds where applicable.

Availability check (server-side):

- Use an availability table or query confirmed bookings for the property where date ranges overlap. For scale, consider a booking calendar aggregated per day or interval tree.

Concurrency & double-booking prevention:

- Use database transactions and row-level locks when creating bookings, or optimistic concurrency tokens.
- Alternatively, use an availability service using Redis locks keyed to property and date-range to prevent race conditions.

Payment integration:

- Use Stripe Payment Intents to handle authorization and capture flows. Store payment provider transaction IDs in Payments table.

Performance:

- Booking creation must complete quickly (median < 500ms) — heavy operations (reporting, analytics) should be offloaded to background workers.

---

## Error Handling & Logging

- Use structured logging (JSON) with correlation IDs for requests to trace flows across microservices.
- Global error handler should return consistent error shapes: { error: { code, message, details? } }
- Map common HTTP codes: 400, 401, 403, 404, 409, 422, 500.

## Database

- Recommended: PostgreSQL for relational integrity and advanced features (e.g., PostGIS for geospatial queries).
- Use migrations (e.g., Flyway, Alembic, Django migrations, or Sequelize migrations).

## Testing

- Unit tests for services and controllers.
- Integration tests for API endpoints (pytest, supertest, or similar).
- End-to-end tests for critical flows (register -> create listing -> book -> payment -> review).
