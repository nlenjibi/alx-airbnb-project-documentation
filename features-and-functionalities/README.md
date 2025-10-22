# Features and Functionalities — Airbnb Clone Backend

This document lists and explains the backend features the Airbnb Clone must support. Use this as the canonical reference while you create diagrams and technical specs.

Core areas:

- User Management
- Property Listings Management
- Search & Filtering
- Booking Management
- Payment Integration
- Reviews & Ratings
- Notifications
- Admin Dashboard

Detailed feature descriptions

1. User Management

- Registration: Sign up as guest or host. Required fields: name, email, password, role (guest|host). Optional: profile photo, phone number.
- Authentication: JWT-based sessions. Support OAuth providers (Google, Facebook) as optional flows.
- Login: Email + password and OAuth.
- Profile management: Update profile info, upload profile picture, change password, view booking history.

2. Property Listings Management

- Create listing: Hosts provide title, description, location (address + lat/lng), price (per night), currency, amenities, rules, photos, maximum guests, property type, and availability calendar.
- Edit/Delete listing: Hosts can update or remove their own listings.
- Media handling: Images should be uploaded to cloud storage (e.g., S3 or Cloudinary). Store URLs in the database.

3. Search & Filtering

- Search by city, state, country, or geo-coordinates.
- Filters: price range, number of guests, amenities, property type, availability dates.
- Pagination: limit & offset or cursor-based for large datasets.

4. Booking Management

- Create booking: Guests request booking for date range; price calculation should account for nightly rate, cleaning fee, taxes, and service fees.
- Prevent double-booking: Validate against existing confirmed bookings.
- Booking lifecycle: statuses such as pending, confirmed, canceled, completed.
- Cancellation: Enforce cancellation policies; handle refunds.

5. Payment Integration

- Use a secure payment gateway like Stripe (recommended) or PayPal.
- Charge guests on booking confirmation; hold funds in escrow and payout hosts after checkout or per payout schedule.
- Support multiple currencies; perform server-side currency normalization.

6. Reviews & Ratings

- Guests leave reviews and star ratings tied to a completed booking.
- Hosts can reply to reviews.
- Enforce one review per booking per reviewer.

7. Notifications

- Send email and in-app notifications for booking confirmations, cancellations, payout events, messages.
- Use services like SendGrid or Mailgun for email delivery.

8. Admin Dashboard

- Admin users can manage users, listings, bookings, payments, and review reports and analytics.

