# Clothes Rental System - Scale Estimation

## 1. Overview

This document estimates the expected traffic, storage,
and resource requirements for the Clothes Rental System.

The numbers below are design assumptions used to evaluate
the scalability of the system.

---

## 2. User Scale

| Metric | Assumption |
|---|---:|
| Registered users | 10 million |
| Daily active users (DAU) | 2 million |
| Peak concurrent users | 200,000 |
| Clothing listings | 5 million |
| Bookings per day | 200,000 |
| Search requests per day | 10 million |

---

## 3. API Traffic

Assumptions:

- Daily active users = 2 million
- Average API requests per active user = 20/day

### Daily API Requests

2,000,000 × 20

= 40,000,000 requests/day

### Average Requests Per Second

40,000,000 / 86,400

≈ 463 RPS

### Peak Requests Per Second

Assuming peak traffic is approximately 5× average traffic:

463 × 5

≈ 2,315 RPS

For capacity planning, we target approximately:

**3,000 RPS**


---

## 4. Database Storage Estimation

The system is expected to contain approximately 5 million clothing listings.

### Clothing Data

Assumption:

- Average clothing document size ≈ 2 KB
- Number of clothing listings = 5 million

Calculation:

5,000,000 × 2 KB

≈ 10 GB

This represents only the raw clothing document data.

The database will also contain:

- Users
- Bookings
- Reviews
- Notifications
- Reports
- Subscriptions
- Subscription plans
- Indexes
- Database overhead

Therefore, the initial database capacity should be planned
with additional headroom rather than provisioning only 10 GB.

For this design, we estimate approximately:

**50–100 GB of initial database storage capacity.**

---

## 5. Image Storage Estimation

Clothing images are stored separately from the database.

Assumptions:

- Clothing listings = 5 million
- Average images per listing = 5
- Average image size = 1 MB

### Total Images

5,000,000 × 5

= 25,000,000 images

### Total Image Storage

25,000,000 × 1 MB

≈ 25 TB

Therefore, approximately **25 TB of object storage**
would be required for the initial image dataset.

---

## 6. Object Storage

Images should not be stored directly inside MongoDB.

Instead, the system uses object storage for clothing images.

Example architecture:

Client
  |
  ↓
Object Storage
  |
  ↓
Image URL
  |
  ↓
MongoDB

MongoDB stores image URLs/references rather than
the actual image files.

---

## 7. CDN

A CDN is placed in front of object storage for frequently
requested clothing images.

Request flow:

Client
  |
  ↓
CDN
  |
  ├── Cache HIT → Return image
  |
  └── Cache MISS
          |
          ↓
     Object Storage

Benefits:

- Lower image latency
- Reduced object-storage requests
- Reduced application-server load
- Better performance for geographically distributed users


---

## 8. Booking Traffic Estimation

Assumption:

- Bookings per day = 200,000

### Average Booking Rate

200,000 / 86,400

≈ 2.3 bookings/second

### Peak Booking Rate

Assuming peak traffic is approximately 5× average traffic:

2.3 × 5

≈ 11.5 bookings/second

Therefore, the system should support approximately:

**12 booking creations/second during peak traffic.**

Although booking traffic is relatively low compared to
overall API traffic, booking is a critical operation because
incorrect concurrency handling can result in double booking
or inconsistent booking state.

---

## 9. Search Traffic Estimation

Assumption:

- Search requests per day = 10 million

### Average Search Rate

10,000,000 / 86,400

≈ 116 RPS

### Peak Search Rate

116 × 5

≈ 580 RPS

Therefore, the search system should support approximately:

**580 search requests/second during peak traffic.**

---

## 10. Read vs Write Pattern

The system is expected to be predominantly read-heavy.

### Read-heavy operations

- Clothing search
- Clothing details
- Browse listings
- Reviews
- User profile
- Notifications
- Subscription information

### Write-heavy operations

- Clothing uploads
- Booking creation
- Booking status updates
- Reviews
- Reports
- Subscription changes

Search traffic is significantly higher than booking
creation traffic.

This makes caching, database indexes, read replicas,
and potentially a dedicated search system important
for future scalability.