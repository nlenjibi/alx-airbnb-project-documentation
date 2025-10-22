# User Stories — Airbnb Clone Backend

Below are user stories converted from the use case diagram. There are at least 5 core user stories capturing important interactions.

1. As a guest, I want to register an account so that I can search and book properties.

2. As a host, I want to create and manage property listings so that I can rent my space to guests.

3. As a guest, I want to search for properties by location, price, and amenities so that I can find a place that fits my needs.

4. As a guest, I want to make a booking for specified dates so that I can reserve a property.

5. As a guest, I want to pay securely for my booking so that my reservation is confirmed.

6. As a guest, I want to leave a review after my stay so that I can share feedback with other users.

7. As an admin, I want to manage users and listings so that I can ensure platform safety and quality.

Mapping to acceptance criteria (example):

- Registration: valid email, password with minimum length 8, unique email, optional OAuth.
- Create listing: host must be authenticated and verified; required fields: title, location, price, max_guests.
- Booking: date range must be available; payment required for confirmation.
