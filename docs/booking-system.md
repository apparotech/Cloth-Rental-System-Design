# Clothes Rental System - Booking System

## 1. Overview

The Booking System is one of the most consistency-sensitive
parts of the Clothes Rental System.

A booking represents a renter's request to rent a specific
clothing item from an owner for a specific date range.

The booking system must handle:

- Booking creation
- Availability checking
- Double-booking prevention
- Booking acceptance and rejection
- Booking cancellation
- Booking expiration
- Booking state transitions
- Concurrent booking requests
- Background processing
- Reliable notifications

The system must ensure that two users cannot successfully
book the same clothing item for overlapping dates.

---

## 2. Booking Lifecycle

A booking follows a defined lifecycle.

The normal flow is:

PENDING

↓

ACCEPTED

↓

ACTIVE

↓

COMPLETED

A booking can also follow alternative paths:

PENDING → REJECTED

PENDING → CANCELLED

PENDING → EXPIRED

ACCEPTED → CANCELLED

### State Descriptions

### PENDING

The renter has created a booking request, but the clothing
owner has not accepted or rejected it yet.

### ACCEPTED

The owner has accepted the booking request.

### ACTIVE

The rental period has started and the renter currently has
the clothing.

### COMPLETED

The rental period has ended successfully.

### REJECTED

The owner rejected the booking request.

### CANCELLED

The booking was cancelled by an authorized user or system
operation.

### EXPIRED

The booking remained pending beyond the allowed response
time and was automatically expired by the system.

---

## 3. Booking State Machine

The system allows only valid state transitions.

### Valid Transitions

PENDING → ACCEPTED

PENDING → REJECTED

PENDING → CANCELLED

PENDING → EXPIRED

ACCEPTED → ACTIVE

ACCEPTED → CANCELLED

ACTIVE → COMPLETED

### Invalid Transitions

The system must prevent invalid state changes such as:

COMPLETED → ACCEPTED

REJECTED → ACTIVE

CANCELLED → ACCEPTED

EXPIRED → ACCEPTED

COMPLETED → ACTIVE

This prevents inconsistent booking records.

### State Machine

PENDING

├── ACCEPTED

│     └── ACTIVE

│           └── COMPLETED

│

├── REJECTED

│

├── CANCELLED

│

└── EXPIRED

---

## 4. Pending Booking Expiration

A pending booking should not remain pending indefinitely.

If the owner does not respond within a configured time,
the booking is automatically expired.

For this design, we assume a pending booking timeout of
30 minutes.

Example:

Booking created

↓

PENDING

↓

30 minutes

↓

EXPIRED

The timeout value is a business configuration and can be
changed without changing the overall architecture.

### Why Expiration is Required

Without expiration:

- A pending booking could remain indefinitely.
- The requested dates could remain blocked.
- Other renters may be unable to request the clothing.
- The system could accumulate stale pending bookings.

Expiration releases the booking and allows the system to
continue handling future requests.

---

## 5. Message Queue and Background Worker

The API server should not depend on an in-memory timer such
as `setTimeout()` for booking expiration.

The system contains multiple API servers:

Load Balancer

↓

API Server 1

API Server 2

API Server 3

An API server can restart, crash, or be replaced.

Therefore, booking expiration should be handled through
background processing.

### Expiration Flow

Booking API

↓

Create PENDING Booking

↓

Publish Expiration Job

↓

Message Queue

↓

Background Worker

↓

Check Booking

↓

Expire if necessary

The expiration job contains information such as:

- Booking ID
- Expiration time

### Worker Processing

The worker checks the current booking state before changing
it.

If the booking is still PENDING:

PENDING → EXPIRED

If the booking has already changed to another state:

Do nothing.

This prevents an old background job from overwriting a newer
booking state.

---

## 6. Double-Booking Prevention

The system must prevent two renters from successfully
booking the same clothing for overlapping dates.

For a requested booking:

`requestedStartDate`

`requestedEndDate`

An existing booking overlaps when:

`existing.startDate < requestedEndDate`

AND

`existing.endDate > requestedStartDate`

### Example

Existing booking:

September 10 → September 13

Requested booking:

September 12 → September 15

Check:

`10 < 15` → TRUE

`13 > 12` → TRUE

Both conditions are true.

Therefore:

The bookings overlap.

The requested booking cannot be successfully created.

### Adjacent Bookings

Adjacent bookings are allowed.

Example:

Existing booking:

September 10 → September 13

Requested booking:

September 13 → September 16

Check:

`10 < 16` → TRUE

`13 > 13` → FALSE

Therefore, there is no overlap.

The second booking can be allowed.

### Blocking Booking Statuses

Only active reservation-related states participate in
availability checks.

Blocking statuses:

- PENDING
- ACCEPTED
- ACTIVE

Non-blocking statuses:

- REJECTED
- CANCELLED
- EXPIRED
- COMPLETED

---

## 7. Race Condition During Booking Creation

A simple availability check is not sufficient.

Two renters may send booking requests at nearly the same
time.

### Request A

Check availability.

Result:

Available.

### Request B

Check availability.

Result:

Available.

Both requests may then attempt to create a booking.

Without concurrency protection:

Request A → Create Booking

Request B → Create Booking

Both bookings could be created successfully.

This would result in a double booking.

### Problem

The issue occurs because the following operations are
treated separately:

Check availability

↓

Create booking

Another request can execute between these operations.

This is known as a race condition.

Therefore, availability checking and booking creation must
be protected as a consistency-sensitive operation.

---

## 8. Booking Creation with Concurrency Control

The booking creation process should conceptually work as:

Start Transaction

↓

Check overlapping bookings

↓

Is the clothing available?

├── No → Abort Transaction

└── Yes

↓

Create Booking

↓

Commit Transaction

The goal is to ensure that two concurrent requests cannot
both successfully reserve the same clothing for overlapping
dates.

If a conflicting booking is detected, the request should
fail rather than creating another overlapping booking.

The API can return:

`409 Conflict`

for an availability conflict.

### Important Principle

The availability check and booking creation should be treated
as one consistency-sensitive operation.

The exact concurrency mechanism can depend on the database
deployment and implementation strategy.

---

## 9. Booking Acceptance

Availability must also be checked when an owner accepts a
pending booking.

The system should not assume that the clothing is still
available simply because it was available when the booking
request was initially created.

### Acceptance Flow

Owner clicks Accept

↓

Start protected operation

↓

Recheck availability

↓

Is the clothing still available?

├── Yes → PENDING → ACCEPTED

└── No → Reject acceptance

### Why Recheck Availability?

Consider the following situation:

Booking A is created:

PENDING

↓

Another booking changes the availability

↓

Owner attempts to accept Booking A

If the system does not perform another availability check,
it could accept a booking that now conflicts with another
booking.

Therefore, availability should be verified again during
the acceptance operation.

---

## 10. Atomic State Transitions

Booking state changes should be conditional.

Consider the expiration worker.

A booking is initially:

PENDING

The worker wants to change it to:

EXPIRED

However, the owner may accept the booking just before the
worker executes.

The state could become:

PENDING → ACCEPTED

The expiration worker must not then perform:

ACCEPTED → EXPIRED

### Conditional Update

Conceptually:

`UPDATE booking`

`WHERE bookingId = X`

`AND status = PENDING`

If the update succeeds:

PENDING → EXPIRED

If no record is updated, the booking has already changed
state.

The worker should then stop processing that booking.

### Why This Matters

This prevents an old background job from overwriting a newer
valid state transition.

The same principle can be applied to other booking state
changes where concurrent operations are possible.

---

## 11. Idempotent Background Jobs

Background jobs may be delivered more than once.

For example:

Expiration Job

↓

Worker processes job

↓

PENDING → EXPIRED

The same message may later be delivered again.

The worker checks the booking status:

EXPIRED

Since the booking is no longer PENDING, the worker does
nothing.

The final state remains:

EXPIRED

This is called idempotent processing.

### Why Can Messages Be Delivered More Than Once?

A queue may retry a message when:

- A worker crashes
- Processing fails
- Network communication fails
- The worker does not acknowledge the message
- Processing times out

Therefore, background workers should be designed so that
processing the same message multiple times does not produce
an incorrect final state.

---

## 12. Booking Consistency Rules

The following rules must always be maintained.

### Rule 1 — No Double Booking

A clothing item must not have two successfully reserved
overlapping bookings when both bookings have blocking
statuses.

---

### Rule 2 — Valid State Transitions

Only valid booking state transitions are allowed.

For example:

PENDING → ACCEPTED

PENDING → REJECTED

PENDING → CANCELLED

PENDING → EXPIRED

Invalid transitions must be rejected.

---

### Rule 3 — Expired Bookings Cannot Be Accepted

Once a booking has expired:

EXPIRED → ACCEPTED

is not allowed.

The renter would need to create a new booking request.

---

### Rule 4 — Rejected or Cancelled Bookings Cannot Become Active

For example:

REJECTED → ACTIVE

and

CANCELLED → ACTIVE

are invalid transitions.

---

### Rule 5 — Booking Price is a Historical Snapshot

The booking stores the price that was applicable when the
booking was created.

Example:

Clothing price:

₹800/day

Booking:

`pricePerDay = ₹800`

Later, the owner changes the clothing price:

₹800/day → ₹1,000/day

The existing booking still uses:

`pricePerDay = ₹800`

This prevents changes to the clothing listing from altering
historical bookings.

---

### Rule 6 — Availability is Determined from Booking Records

Availability should not depend on a simple field such as:

`isAvailable = true`

because availability depends on the requested date range.

Example:

Clothing:

September 10 → September 15 = Booked

September 20 → September 25 = Available

Therefore, availability must be calculated based on booking
records and date-overlap rules.

---

### Rule 7 — Critical Booking Operations Require Concurrency Control

Booking creation and acceptance are consistency-sensitive
operations.

The system must protect them against concurrent requests so
that multiple users cannot successfully reserve the same
clothing for overlapping dates.

---

### Rule 8 — Background Jobs Must Be Idempotent

Expiration and other background operations may be retried.

Processing the same job multiple times should not create an
incorrect booking state.

---

### Rule 9 — MongoDB is the Source of Truth

MongoDB is the authoritative source for persistent booking
state.

Redis is used as a performance and caching layer.

Redis should not replace MongoDB as the source of truth for
booking state.

---

### Rule 10 — Redis Must Not Decide Booking Availability

Redis can be used for:

- Clothing details
- Popular listings
- Search results
- Frequently accessed public data

However, the final booking availability decision must use
the authoritative database state and concurrency-control
logic.

This prevents stale cache data from causing incorrect
booking decisions.

---

## 13. Booking Failure Scenarios

The system should also handle common failure scenarios.

### Scenario 1 — Worker Crashes

Expiration Worker

↓

Starts processing job

↓

Worker crashes

↓

Message remains available for retry

↓

Another worker processes the job

The booking is safely processed.

---

### Scenario 2 — Duplicate Expiration Job

The same expiration job is delivered twice.

First execution:

PENDING → EXPIRED

Second execution:

Booking is already EXPIRED

↓

No action

The final state remains correct.

---

### Scenario 3 — Owner Accepts an Expired Booking

Owner attempts to accept a booking after it has expired.

The API checks the current state:

EXPIRED

↓

Acceptance rejected

The system does not allow:

EXPIRED → ACCEPTED

---

### Scenario 4 — Booking Conflict

Two users attempt to reserve the same clothing for
overlapping dates.

The system detects the conflict and one request fails with:

`409 Conflict`

Only one booking can successfully reserve the conflicting
time period.

---

### Scenario 5 — Notification Failure

Booking state is successfully updated:

PENDING → ACCEPTED

but notification delivery fails.

The booking should remain ACCEPTED.

The notification operation can be retried asynchronously
through the Message Queue.

This prevents a notification failure from incorrectly
changing the booking state.

---

## 14. Booking System Summary

The booking system uses:

- Date-overlap checks for availability
- Concurrency control for booking creation
- Availability rechecks during acceptance
- Conditional state transitions
- Message Queue for asynchronous processing
- Background Workers for expiration
- Idempotent job processing
- MongoDB as the authoritative booking data store
- Redis only as a performance optimization

The overall booking flow is:

Renter

↓

Create Booking Request

↓

Authentication + Authorization

↓

Check Subscription

↓

Check Availability

↓

Create PENDING Booking

↓

Publish Background Jobs / Events

↓

Notify Owner

↓

Owner Accepts / Rejects

↓

If Accepted

↓

ACTIVE

↓

COMPLETED

If Owner Does Not Respond:

PENDING

↓

Expiration Worker

↓

EXPIRED

The design prioritizes consistency for booking operations
while using asynchronous processing for non-critical
background operations.