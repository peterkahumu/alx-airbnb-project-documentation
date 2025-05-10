# Backend Feature Requirement Specifications

## 1. User Authentication

### API Endpoints

* `POST /api/auth/register/`
* `POST /api/auth/login/`
* `GET /api/auth/profile/`
* `PUT /api/auth/profile/update/`

### Input Specifications

#### Register

```json
{
  "email": "user@example.com",
  "password": "Password123!",
  "full_name": "John Doe",
  "role": "guest"  // or "host"
}
```

#### Login

```json
{
  "email": "user@example.com",
  "password": "Password123!"
}
```

### Output Specifications

**Success (201 Created):**

```json
{
  "message": "Registration successful",
  "token": "jwt_token"
}
```

**Failure (400 Bad Request):**

```json
{
  "error": "Email already exists"
}
```

### Validation Rules

* Email must be unique and in a valid format.
* Password must be at least 8 characters, with letters, numbers, and special characters.
* Role must be either `"guest"` or `"host"`.

### Performance Criteria

* Maximum response time: 500ms
* Rate limiting: 5 registration/login attempts per minute per IP address

---

## 2. Property Management

### API Endpoints

* `POST /api/properties/`
* `GET /api/properties/`
* `GET /api/properties/:id/`
* `PUT /api/properties/:id/`
* `DELETE /api/properties/:id/`

### Input Specifications (Create/Update)

```json
{
  "title": "Beach House",
  "description": "Ocean view property",
  "location": "California",
  "price_per_night": 150.0,
  "amenities": ["wifi", "pool"],
  "max_guests": 4,
  "images": ["url1", "url2"]
}
```

### Output Specifications

**Success (200 OK or 201 Created):**

```json
{
  "id": "prop123",
  "title": "Beach House",
  "location": "California",
  "price_per_night": 150.0,
  ...
}
```

**Failure (400 or 403):**

```json
{
  "error": "Invalid input" or "Unauthorized"
}
```

### Validation Rules

* Price must be a positive float.
* `max_guests` must be 1 or greater.
* Only the host who owns the listing can update or delete it.

### Performance Criteria

* GET endpoints must support pagination (e.g., 20 listings per page).
* Use Redis or equivalent caching for frequently queried listings.
* Response time should average under 300ms.

---

## 3. Booking System

### API Endpoints

* `POST /api/bookings/`
* `GET /api/bookings/`
* `DELETE /api/bookings/:id/`

### Input Specifications (Create Booking)

```json
{
  "property_id": "123",
  "check_in": "2025-06-01",
  "check_out": "2025-06-05"
}
```

### Output Specifications

**Success (201 Created):**

```json
{
  "booking_id": "B123",
  "status": "pending",
  "total_price": 600.0
}
```

**Failure (400 or 409 Conflict):**

```json
{
  "error": "Dates are not available"
}
```

### Validation Rules

* Check-in must be before check-out.
* Property must not have overlapping bookings in the selected range.
* Only users with the `"guest"` role can create bookings.

### Performance Criteria

* Booking availability queries must complete in under 200ms.
* Use transactions or DB-level locks to prevent double booking.
