# Clothes Rental System - Database Design

## 1. Database

Database: MongoDB

## 2. Collections

The system will initially contain the following collections:

1. User
2. SubscriptionPlan
3. Subscription
4. Clothing
5. Booking
6. Review
7. Notification
8. Report

                              ┌──────────────────────┐
                              │  SubscriptionPlan    │
                              │──────────────────────│
                              │ _id                  │
                              │ name                 │
                              │ price                │
                              │ duration             │
                              │ features[]           │
                              │ status               │
                              └──────────┬───────────┘
                                         │
                                        1:N
                                         │
                                         ▼
┌──────────────────────┐       ┌──────────────────────┐
│        User          │       │    Subscription      │
│──────────────────────│       │──────────────────────│
│ _id                  │1     N│ _id                  │
│ name                 ├───────┤ userId               │
│ email                │       │ planId               │
│ phone                │       │ status               │
│ password             │       │ startDate            │
│ profileImage         │       │ endDate              │
│ status               │       │ paymentId            │
│ createdAt            │       └──────────────────────┘
└──────────┬───────────┘
           │
     ┌─────┼───────────────┬────────────────┬─────────────────┐
     │     │               │                │                 │
    1:N   1:N             1:N              1:N               1:N
     │     │               │                │                 │
     ▼     ▼               ▼                ▼                 ▼
┌────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌──────────────┐
│Clothing│ │  Booking   │ │   Review   │ │Notification│ │    Report    │
│────────│ │────────────│ │────────────│ │────────────│ │──────────────│
│_id     │ │_id         │ │_id         │ │_id         │ │_id           │
│ownerId │ │renterId    │ │userId      │ │userId      │ │reportedBy    │
│title   │ │clothingId  │ │clothingId  │ │type        │ │reportedUser  │
│category│ │ownerId     │ │bookingId   │ │title       │ │clothingId    │
│size    │ │startDate   │ │rating      │ │message     │ │reason        │
│price   │ │endDate     │ │comment     │ │read        │ │description   │
│images[]│ │status      │ │createdAt   │ │createdAt   │ │status        │
│status  │ │totalAmount │ └────────────┘ └────────────┘ └──────────────┘
└────┬───┘ └─────┬──────┘
     │            │
    1:N          1:0..1
     │            │
     └────────────┘
          │
          ▼
       Review




## Phase 2 — Step 8: Finalize `database-design.md`

Now that we've decided **collections, relationships, fields, data types, indexes, and double-booking prevention**, we should document everything properly before moving to APIs.

Your `database-design.md` should have these sections:

```text
# Clothes Rental System - Database Design

1. Database Choice
2. Collections
3. Relationships
4. Collection Fields
5. Indexes
6. Double Booking Prevention
7. Important Design Decisions
```

### Important design decisions we should document

**1. MongoDB**

> We are using MongoDB as the primary database.

**2. Images**

> Clothing images will be stored in object/file storage. MongoDB stores only the image URLs.

**3. Subscription history**

> A user can have multiple subscription records, so we keep subscription history.

**4. Booking price snapshot**

> Booking stores `pricePerDay` so that if the owner changes the clothing price later, an old booking's price doesn't change.

**5. Availability**

> We don't store rental dates directly inside Clothing. Availability is determined using Booking records.

**6. Double booking**

> Before creating a booking, we check for overlapping bookings and use transactional/concurrency control so two users cannot successfully book the same clothing for the same dates.

---

### Phase 2 status

```text
✅ Collections
✅ Relationships
✅ Fields
✅ Data types
✅ Required / Optional
✅ Indexes
✅ Double-booking prevention
🔄 Final database documentation
⬜ Final database review
⬜ Phase 3 — API Design
```


