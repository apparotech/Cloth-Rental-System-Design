
# Clothes Rental System - API Design

## 1. Overview

The Clothes Rental System exposes RESTful APIs for mobile and web clients.

The APIs are versioned using `/api/v1` to allow future API evolution without breaking existing clients.

Base URL:

/api/v1

---

# 2. API Design Principles

The API follows these principles:

- RESTful API design
- API versioning
- JWT-based authentication
- Role-based authorization for admin APIs
- Standard HTTP status codes
- JSON request and response format
- Pagination for large collections
- Filtering and sorting for search APIs
- Consistent error responses
- Validation at the API boundary

---

# 3. Authentication & User APIs

## 3.1 Authentication APIs

| Endpoint | Method | Authentication | Purpose |
|---|---|---|---|
| `/auth/register` | POST | No | Register a new user |
| `/auth/login` | POST | No | Authenticate user |
| `/auth/refresh` | POST | Refresh Token | Generate new access token |
| `/auth/logout` | POST | Yes | Logout user |
| `/auth/forgot-password` | POST | No | Request password reset |
| `/auth/reset-password` | POST | No | Reset password |

---

## 3.2 Register User

### Endpoint

POST `/api/v1/auth/register`

### Authentication

Not required.

### Request

```json
{
  "name": "Raj Aryan",
  "email": "raj@example.com",
  "phone": "9876543210",
  "password": "********"
}
````

### Business Logic

1. Validate request data.
2. Check whether email already exists.
3. Hash the password.
4. Create the user.
5. Create a 7-day trial subscription.
6. Return authentication information.

### Important Rule

Every newly registered user receives a 7-day free trial.

The trial is represented using the Subscription collection instead of creating a separate Trial collection.

Example:

```text
status    = TRIAL
startDate = registration date
endDate   = registration date + 7 days
```

### Possible Responses

`201 Created`

```json
{
  "success": true,
  "message": "Registration successful"
}
```

`400 Bad Request`

```json
{
  "success": false,
  "message": "Invalid registration data"
}
```

`409 Conflict`

```json
{
  "success": false,
  "message": "Email already exists"
}
```

---

# 4. Login

### Endpoint

POST `/api/v1/auth/login`

### Authentication

Not required.

### Request

```json
{
  "email": "raj@example.com",
  "password": "********"
}
```

### Business Logic

1. Find user by email.
2. Verify password.
3. Check account status.
4. Generate access token.
5. Generate refresh token.
6. Return authentication information.

### Possible Responses

`200 OK`

```json
{
  "success": true,
  "data": {
    "accessToken": "access-token",
    "refreshToken": "refresh-token"
  }
}
```

`401 Unauthorized`

```json
{
  "success": false,
  "message": "Invalid credentials"
}
```

`403 Forbidden`

```json
{
  "success": false,
  "message": "Account is blocked"
}
```

---

# 5. Refresh Token

### Endpoint

POST `/api/v1/auth/refresh`

### Authentication

Refresh token required.

### Purpose

Generate a new access token when the existing access token expires.

---

# 6. Logout

### Endpoint

POST `/api/v1/auth/logout`

### Authentication

Required.

### Purpose

Invalidate the user's authentication session/refresh token.

---

# 7. Forgot Password

### Endpoint

POST `/api/v1/auth/forgot-password`

### Authentication

Not required.

### Request

```json
{
  "email": "raj@example.com"
}
```

### Purpose

Initiate the password reset process.

---

# 8. Reset Password

### Endpoint

POST `/api/v1/auth/reset-password`

### Authentication

Password reset token required.

### Request

```json
{
  "token": "reset-token",
  "newPassword": "********"
}
```

---

# 9. User APIs

| Endpoint             | Method | Authentication | Purpose                  |
| -------------------- | ------ | -------------- | ------------------------ |
| `/users/me`          | GET    | Yes            | Get current user profile |
| `/users/me`          | PUT    | Yes            | Update profile           |
| `/users/me/password` | PUT    | Yes            | Change password          |

---

## 9.1 Get Profile

### Endpoint

GET `/api/v1/users/me`

### Authentication

Required.

### Purpose

Returns the authenticated user's profile information.

---

## 9.2 Update Profile

### Endpoint

PUT `/api/v1/users/me`

### Authentication

Required.

### Request

```json
{
  "name": "Raj Aryan",
  "phone": "9876543210",
  "profileImage": "image-url"
}
```

---

## 9.3 Change Password

### Endpoint

PUT `/api/v1/users/me/password`

### Authentication

Required.

### Request

```json
{
  "currentPassword": "********",
  "newPassword": "********"
}
```

---

# 10. Clothing APIs

Clothing is the core resource of the platform.

A user can upload multiple clothing items and can also rent clothing uploaded by other users.

| Endpoint                      | Method | Authentication | Purpose              |
| ----------------------------- | ------ | -------------- | -------------------- |
| `/clothing`                   | POST   | Yes            | Upload clothing      |
| `/clothing/{id}`              | GET    | No/Yes         | Get clothing details |
| `/clothing/{id}`              | PUT    | Yes            | Update clothing      |
| `/clothing/{id}`              | DELETE | Yes            | Delete clothing      |
| `/clothing/my`                | GET    | Yes            | Get user's clothing  |
| `/clothing/{id}/availability` | GET    | No/Yes         | Check availability   |

---

## 10.1 Create Clothing

### Endpoint

POST `/api/v1/clothing`

### Authentication

Required.

### Request

```json
{
  "title": "Black Formal Shirt",
  "description": "Black formal shirt in good condition",
  "category": "SHIRT",
  "size": "L",
  "gender": "MEN",
  "brand": "XYZ",
  "condition": "GOOD",
  "images": [
    "image-url-1",
    "image-url-2"
  ],
  "pricePerDay": 800,
  "securityDeposit": 3000,
  "location": "Delhi"
}
```

### Business Logic

1. Authenticate user.
2. Validate clothing information.
3. Upload/store image references.
4. Create clothing record.
5. Set status to `AVAILABLE`.

---

## 10.2 Get Clothing

### Endpoint

GET `/api/v1/clothing/{id}`

### Purpose

Returns details of a specific clothing item.

---

## 10.3 Update Clothing

### Endpoint

PUT `/api/v1/clothing/{id}`

### Authentication

Required.

### Authorization

Only the owner of the clothing can update it.

---

## 10.4 Delete Clothing

### Endpoint

DELETE `/api/v1/clothing/{id}`

### Authentication

Required.

### Authorization

Only the owner can delete the clothing.

Historical booking information should not be affected by deleting/deactivating a listing.

---

## 10.5 Get My Clothing

### Endpoint

GET `/api/v1/clothing/my`

### Authentication

Required.

### Purpose

Returns all clothing items uploaded by the authenticated user.

---

## 10.6 Check Clothing Availability

### Endpoint

GET `/api/v1/clothing/{id}/availability`

### Query Parameters

```text
startDate
endDate
```

Example:

```text
GET /api/v1/clothing/123/availability?startDate=2026-09-10&endDate=2026-09-13
```

The system checks existing bookings for overlapping dates.

---

# 11. Search & Filter APIs

### Endpoint

GET `/api/v1/clothing`

### Example

```text
GET /api/v1/clothing?category=SHIRT&gender=MEN&location=Delhi&status=AVAILABLE
```

### Supported Filters

* Category
* Gender
* Size
* Brand
* Condition
* Location
* Price range
* Availability

### Sorting

Examples:

```text
sort=price_asc
sort=price_desc
sort=newest
```

### Pagination

Example:

```text
?page=1&limit=20
```

The API should use pagination because returning thousands of clothing records in one response is inefficient.

---

# 12. Booking APIs

Booking is one of the most important parts of the system.

| Endpoint                | Method | Authentication | Purpose                         |
| ----------------------- | ------ | -------------- | ------------------------------- |
| `/bookings`             | POST   | Yes            | Request rental                  |
| `/bookings/{id}`        | GET    | Yes            | Get booking details             |
| `/bookings/my`          | GET    | Yes            | Get renter bookings             |
| `/bookings/owner`       | GET    | Yes            | Get bookings for owned clothing |
| `/bookings/{id}/accept` | PUT    | Yes            | Accept booking                  |
| `/bookings/{id}/reject` | PUT    | Yes            | Reject booking                  |
| `/bookings/{id}/cancel` | PUT    | Yes            | Cancel booking                  |

---

## 12.1 Create Booking

### Endpoint

POST `/api/v1/bookings`

### Authentication

Required.

### Request

```json
{
  "clothingId": "clothing123",
  "startDate": "2026-09-10",
  "endDate": "2026-09-13"
}
```

### Business Logic

1. Authenticate renter.
2. Check user's rental access/subscription.
3. Check clothing exists.
4. Check clothing is available.
5. Validate rental dates.
6. Check overlapping bookings.
7. Calculate rental amount.
8. Create booking.
9. Notify clothing owner.

### Rental Calculation

If:

```text
pricePerDay = ₹800
startDate   = Sept 10
endDate     = Sept 13
```

Rental duration:

```text
3 days
```

Rental amount:

```text
800 × 3 = ₹2400
```

Security deposit:

```text
₹3000
```

The booking stores these values as a snapshot.

---

# 13. Double Booking Prevention

The system must prevent two users from successfully booking the same clothing for overlapping dates.

Overlap condition:

```text
existing.startDate < requested.endDate
AND
existing.endDate > requested.startDate
```

If an overlapping booking exists, the request is rejected.

### Important Concurrency Problem

Checking availability alone is not enough.

Two requests could arrive at almost the same time:

```text
Request A → Check availability → Available
Request B → Check availability → Available

Request A → Create booking
Request B → Create booking
```

This could result in double booking.

Therefore, booking creation must use appropriate transaction/concurrency control so that the availability check and booking creation are performed safely.

### Conflict Response

```text
409 Conflict
```

Example:

```json
{
  "success": false,
  "message": "Clothing is already booked for the requested dates"
}
```

---

# 14. Booking Status Flow

```text
PENDING
   |
   ├── ACCEPTED
   │      |
   │      ↓
   │    ACTIVE
   │      |
   │      ↓
   │   COMPLETED
   |
   ├── REJECTED
   |
   └── CANCELLED
```

The owner can accept or reject a pending rental request.

---

# 15. Review APIs

| Endpoint                 | Method | Authentication | Purpose              |
| ------------------------ | ------ | -------------- | -------------------- |
| `/reviews`               | POST   | Yes            | Create review        |
| `/clothing/{id}/reviews` | GET    | No/Yes         | Get clothing reviews |
| `/reviews/{id}`          | PUT    | Yes            | Update review        |
| `/reviews/{id}`          | DELETE | Yes            | Delete review        |

A review should be associated with a completed booking.

A unique constraint on `bookingId` prevents multiple reviews for the same booking.

---

# 16. Notification APIs

| Endpoint                   | Method | Authentication | Purpose                        |
| -------------------------- | ------ | -------------- | ------------------------------ |
| `/notifications`           | GET    | Yes            | Get notifications              |
| `/notifications/{id}/read` | PUT    | Yes            | Mark notification as read      |
| `/notifications/read-all`  | PUT    | Yes            | Mark all notifications as read |

Notifications can be generated for:

* Booking request
* Booking accepted
* Booking rejected
* Booking cancelled
* Booking completed
* Subscription expiring
* Subscription expired
* New review
* Listing blocked

---

# 17. Report APIs

| Endpoint      | Method | Authentication | Purpose               |
| ------------- | ------ | -------------- | --------------------- |
| `/reports`    | POST   | Yes            | Create report         |
| `/reports/my` | GET    | Yes            | Get submitted reports |

Users can report:

* Another user
* Clothing
* Both

Example:

```json
{
  "reportedUser": "user123",
  "clothingId": "clothing123",
  "reason": "MISLEADING_LISTING",
  "description": "The actual clothing does not match the listing."
}
```

---

# 18. Subscription APIs

| Endpoint                     | Method | Authentication | Purpose                  |
| ---------------------------- | ------ | -------------- | ------------------------ |
| `/subscription-plans`        | GET    | No             | Get available plans      |
| `/subscriptions/me`          | GET    | Yes            | Get current subscription |
| `/subscriptions`             | POST   | Yes            | Subscribe to a plan      |
| `/subscriptions/history`     | GET    | Yes            | Get subscription history |
| `/subscriptions/{id}/cancel` | PUT    | Yes            | Cancel subscription      |

---

## 18.1 Get Subscription Plans

### Endpoint

GET `/api/v1/subscription-plans`

Returns active subscription plans.

Example:

```json
{
  "data": [
    {
      "id": "plan123",
      "name": "Monthly",
      "price": 199,
      "duration": 30
    }
  ]
}
```

---

# 19. Payment APIs

Platform subscription payments are handled through a payment gateway.

Rental payments between renter and clothing owner are outside the platform.

| Endpoint            | Method | Authentication | Purpose                        |
| ------------------- | ------ | -------------- | ------------------------------ |
| `/payments/create`  | POST   | Yes            | Initiate subscription payment  |
| `/payments/webhook` | POST   | No*            | Receive gateway payment events |
| `/payments/{id}`    | GET    | Yes            | Get payment status             |

`/payments/webhook` must be authenticated/verified using the payment provider's webhook signature rather than a normal user JWT.

---

# 20. Admin APIs

Admin APIs require administrator authorization.

| Endpoint                         | Method | Purpose             |
| -------------------------------- | ------ | ------------------- |
| `/admin/users`                   | GET    | List users          |
| `/admin/users/{id}/block`        | PUT    | Block user          |
| `/admin/users/{id}/unblock`      | PUT    | Unblock user        |
| `/admin/clothing`                | GET    | Manage listings     |
| `/admin/clothing/{id}/block`     | PUT    | Block listing       |
| `/admin/reports`                 | GET    | View reports        |
| `/admin/reports/{id}`            | PUT    | Update report       |
| `/admin/subscription-plans`      | POST   | Create plan         |
| `/admin/subscription-plans/{id}` | PUT    | Update plan         |
| `/admin/statistics`              | GET    | Platform statistics |

---

# 21. Standard HTTP Status Codes

| Status | Meaning               | Example                          |
| ------ | --------------------- | -------------------------------- |
| 200    | OK                    | Successful request               |
| 201    | Created               | User/booking created             |
| 400    | Bad Request           | Invalid input                    |
| 401    | Unauthorized          | Authentication missing/invalid   |
| 403    | Forbidden             | User doesn't have permission     |
| 404    | Not Found             | Resource doesn't exist           |
| 409    | Conflict              | Double booking/email conflict    |
| 422    | Unprocessable Entity  | Validation/business-rule failure |
| 500    | Internal Server Error | Unexpected server error          |

---

# 22. API Security Considerations

The API should implement:

* Password hashing
* JWT authentication
* Refresh token rotation/invalidation strategy
* Role-based authorization
* Input validation
* Rate limiting
* HTTPS
* Secure HTTP headers
* Payment webhook signature verification
* Protection against unauthorized resource access
* Sensitive information should not be returned in API responses

---

# 23. Future API Improvements

As the system scales, the API layer can be extended with:

* API Gateway
* Rate limiting service
* Distributed caching
* Search service
* Message queues
* Read replicas
* Service decomposition

These decisions will be covered in later system-design phases.

````

## ✅ After pasting

Your project should now look like:

```text
clothes-rental-system/
│
└── docs/
    ├── 01-requirements.md
    ├── 02-scale-estimation.md
    ├── 03-database-design.md
    └── 04-api-design.md   ← NEW
````

**Don't start the next architecture phase yet.** First save this document.

Then we'll do a **Phase 3 API design review**, mainly checking whether any important APIs/business flows are missing. After that, we'll move to **Scale Estimation**, which is important because this is where we'll establish the numbers behind your eventual “scalable system” claim on LinkedIn.
