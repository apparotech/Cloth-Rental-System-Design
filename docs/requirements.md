# Clothes Rental System

## 1. Project Overview

The Clothes Rental System is a subscription-based platform where users
can upload their clothes for rent and rent clothes uploaded by other
users.

The platform earns revenue through user subscriptions.

The platform does NOT charge any commission on rental transactions.

Rental payments are handled directly between the renter and the owner
outside the platform.

Every new user receives a free 7-day trial.

After the free trial expires, the user must have an active paid
subscription to use restricted rental features.

---

# 2. Actors

## 2.1 User

A user can act as both:

- Clothing Owner
- Renter

A user does not need separate owner and renter accounts.

A subscribed user can:

- Upload clothes
- Manage their clothes
- Browse clothes
- Search clothes
- View clothing details
- Rent clothes
- Manage bookings
- View rental history
- Manage their subscription
- Manage their profile

---

## 2.2 Admin

The admin manages and monitors the platform.

Admin can:

- Manage users
- View user details
- Block/unblock users
- Manage reported content
- Manage clothing listings
- Manage subscriptions
- View platform statistics
- Manage subscription plans

---

# 3. Subscription Model

The platform follows a subscription-based business model.

Users pay the platform for access to rental features.

The platform does not charge commission on individual rental
transactions.

---

## 3.1 Free Trial

Every newly registered user receives a free 7-day trial.

Example:

User registers:

1 August

Free trial:

1 August → 8 August

During the free trial, the user can use the features allowed
to subscribed users.

After the trial expires, the user must purchase a subscription
to continue using restricted features.

---

## 3.2 Subscription Plans

The platform can provide multiple subscription plans.

Example:

- Monthly plan
- Quarterly plan
- Yearly plan

The actual prices and features of each plan will be decided later.

---

## 3.3 Subscription Features

Users can:

- View available subscription plans
- Start a subscription
- Renew a subscription
- Cancel a subscription
- View current subscription
- View subscription history
- View subscription expiry date

The system must maintain the subscription status.

Possible subscription states:

- TRIAL
- ACTIVE
- EXPIRED
- CANCELLED

---

# 4. Functional Requirements

## 4.1 Authentication

The system should allow users to:

- Register
- Login
- Logout
- Refresh authentication token
- Reset password
- Change password
- View their profile
- Update their profile

The system should securely store user passwords.

---

# 5. User Management

Users should have:

- Name
- Email
- Phone number
- Profile image
- Password
- Account status
- Subscription status
- Created date
- Updated date

Possible account states:

- ACTIVE
- BLOCKED
- DELETED

---

# 6. Clothing Management

A subscribed user can upload clothing items for rent.

A clothing item can contain:

- Title
- Description
- Category
- Size
- Gender
- Brand
- Condition
- Images
- Price per day
- Security deposit
- Location
- Availability status
- Owner
- Created date
- Updated date

---

## 6.1 Clothing Operations

Users can:

- Add clothing
- Upload clothing images
- View their clothes
- Update clothing
- Delete clothing
- Enable/disable a listing
- View clothing details

---

# 7. Browse and Search

Users can:

- Browse available clothes
- Search clothes
- Filter clothes
- Sort clothes
- View clothing details

Possible filters:

- Category
- Size
- Gender
- Price range
- Location
- Availability
- Brand

---

# 8. Clothing Availability

The system must maintain the availability of each clothing item.

A clothing item cannot be rented by two users during overlapping
rental periods.

Example:

Clothing:

Black Sherwani

Already booked:

10 September → 13 September

Another user cannot book:

11 September → 14 September

But another user can book:

15 September → 18 September

The system must check availability before creating a booking.

---

# 9. Rental / Booking System

A user can request to rent a clothing item.

The user selects:

- Clothing
- Start date
- End date

The system calculates:

- Rental Days
- Rental Amount
- Security Deposit

---

## 9.1 Rental Calculation

Example:

Price per day = ₹800

Start date = 10 September

End date = 13 September

Rental days = 3

Rental amount:

3 × ₹800 = ₹2400

Security deposit = ₹3000

Total rental amount:

₹2400 + ₹3000 = ₹5400

The platform does not collect this rental amount or security deposit.

The renter and owner handle rental payment directly outside
the platform.

---

# 10. Booking Lifecycle

A booking can have different statuses.

Possible states:

- PENDING
- ACCEPTED
- REJECTED
- CANCELLED
- ACTIVE
- COMPLETED

Example:

User creates booking:

PENDING

Owner accepts:

ACCEPTED

Rental period starts:

ACTIVE

Rental period finishes:

COMPLETED

---

# 11. Booking Operations

## Renter

Renter can:

- Create booking request
- View booking
- View booking history
- Cancel booking
- View booking status

## Owner

Owner can:

- View incoming booking requests
- Accept booking
- Reject booking
- View rental history

---

# 12. Rental Payment

Rental payments are NOT processed by the platform.

The renter and owner handle rental payment directly outside
the platform.

Example:

Renter
   |
   | Rental payment
   ↓
Owner

The platform does not:

- Take rental commission
- Process rental payment
- Hold rental money
- Split rental payments
- Provide seller payouts
- Provide rental payment settlement

The platform may store the rental amount for booking information,
but it does not process the rental payment.

---

# 13. Subscription Payment

Subscription payments are processed by the platform.

Flow:

User
   |
   | Subscription payment
   ↓
Payment Gateway
   |
   ↓
Platform
   |
   ↓
Subscription ACTIVE

The system should:

- Create subscription
- Verify payment
- Activate subscription
- Track subscription expiry
- Handle subscription renewal
- Handle failed payments
- Handle payment webhooks
- Maintain payment history

---

# 14. Subscription Access Control

The system must check the user's subscription or trial status
before allowing restricted features.

---

## 14.1 New User

When a user registers, the system automatically starts a
7-day free trial.

Example:

User Registration
       |
       ↓
7-Day Free Trial
       |
       ↓
TRIAL ACTIVE

During the active trial, the user can use the same rental features
available to a paid subscriber.

---

## 14.2 Trial Expired

When the 7-day trial expires:

7-Day Trial
     |
     ↓
Trial Expired
     |
     ↓
Active Paid Subscription?
     |
   +---+---+
   |       |
  YES      NO
   |       |
   ↓       ↓
Full      Restricted
Access    Access

If the user has an active paid subscription:

- User can upload clothes
- User can rent clothes
- User can create bookings
- User can manage their clothes
- User can manage their bookings

If the user does not have an active subscription:

- User can login
- User can view their profile
- User can view subscription plans
- User can purchase a subscription
- User can browse/view clothes if allowed

Restricted users cannot:

- Upload new clothes
- Create new rental bookings

Existing clothes and booking history should not be deleted when
the subscription expires.

---

# 15. Reviews and Ratings

After completing a rental, the renter can:

- Give a rating
- Write a review

Users can:

- View ratings
- View reviews

The system should prevent a user from reviewing a clothing item
that they have not rented.

A user should only be able to review a clothing item after
completing a rental.

---

# 16. Notifications

The system should notify users about important events.

Notifications may include:

- New booking request
- Booking accepted
- Booking rejected
- Booking cancelled
- Booking started
- Booking completed
- Subscription activated
- Subscription expiring
- Subscription expired
- Payment successful
- Payment failed

Possible notification channels:

- In-app notification
- Push notification
- Email

---

# 17. Admin Features

The admin manages and monitors the platform.

Admin can:

- View users
- View user details
- Block users
- Unblock users
- View clothing listings
- Remove inappropriate listings
- View bookings
- View subscriptions
- Manage subscription plans
- View reports
- Manage reported users
- Manage reported clothing
- View platform statistics

---

# 18. Reporting System

Users can report:

- Inappropriate clothing
- Fake clothing listing
- Fraudulent user
- Inappropriate content
- Other users

Admin can:

- View reports
- Investigate reports
- Take action
- Block users
- Remove listings
- Close reports

Possible report statuses:

- PENDING
- UNDER_REVIEW
- RESOLVED
- REJECTED

---

# 19. Non-Functional Requirements

## 19.1 Performance

Normal API requests should respond quickly.

Target:

Most normal API requests should respond within approximately
500 milliseconds under normal load.

This target may change after load testing.

---

## 19.2 Scalability

The system should be designed so that it can grow from:

1,000 users
      ↓
10,000 users
      ↓
100,000 users
      ↓
1,000,000+ users

The architecture should support horizontal scaling when required.

---

## 19.3 Availability

The system should remain available even if one application
server fails.

The system should avoid having a single point of failure
where possible.

---

## 19.4 Security

The system should provide:

- Secure password hashing
- JWT authentication
- Authorization
- Input validation
- HTTPS
- Rate limiting
- Secure API endpoints
- Protection against unauthorized access
- Protection against common API attacks

---

## 19.5 Data Consistency

Important operations must maintain correct data.

The system must prevent:

- Double booking
- Invalid subscription status
- Unauthorized booking
- Unauthorized clothing management
- Invalid booking dates

---

## 19.6 Reliability

The system should handle failures gracefully.

Examples:

- Payment gateway failure
- Database failure
- Server failure
- Network failure
- Payment webhook failure

Temporary failures should not create incorrect booking or
subscription states.

---

# 20. Business Rules

## Rule 1: Free Trial

Every newly registered user receives one free 7-day trial.

A user should not be able to repeatedly create accounts just
to receive unlimited free trials.

---

## Rule 2: Subscription Access

A user needs an active trial or active paid subscription to
use restricted rental features.

---

## Rule 3: One User, Two Roles

A user can act as both:

- Owner
- Renter

No separate owner or renter account is required.

---

## Rule 4: No Rental Commission

The platform takes ₹0 commission from rental transactions.

---

## Rule 5: Rental Payment Outside Platform

Rental payments are handled directly between the renter and owner.

Example:

Renter
   |
   | Rental Payment
   ↓
Owner

The platform does not process the rental payment.

---

## Rule 6: Subscription Payment

Subscription payments are processed by the platform through
a supported payment gateway.

---

## Rule 7: No Double Booking

A clothing item cannot have overlapping active bookings.

---

## Rule 8: Booking Confirmation

A booking request must be accepted by the owner before it becomes
confirmed.

---

## Rule 9: Reviews

Only a user who has completed a rental can review that clothing.

---

## Rule 10: Own Clothing

A user cannot rent their own clothing.

---

## Rule 11: Blocked Users

Blocked users cannot perform restricted platform actions.

Their existing data should not automatically be deleted.

---

## Rule 12: Expired Subscription

When a subscription expires, the user's existing clothes,
bookings, reviews and account data remain in the system.

The user loses access to restricted features until they start
another subscription.

---

## Rule 13: Clothing Availability

A clothing item cannot be booked for overlapping rental periods.

---

## Rule 14: Rental Amount

The rental amount is calculated using:

Rental Amount = Number of Rental Days × Price Per Day

Security Deposit is calculated separately.

The platform does not collect the rental amount or security deposit.

---

# 21. Initial Technology Stack

## Backend

- Node.js
- Express.js

## Database

- MongoDB
- Mongoose

## Authentication

- JWT
- bcrypt

## API Testing

- Postman

## API Documentation

- Swagger / OpenAPI

## File Storage

- Cloudinary or Amazon S3

Used for:

- Clothing images
- User profile images

## Subscription Payment

A payment gateway will be used only for subscription payments.

Rental payments are handled outside the platform.

## Cache

- Redis

Redis will be introduced later when the system requires caching
and higher performance.

## Message Queue

- RabbitMQ or Apache Kafka

A message queue will be introduced later for asynchronous
processing such as notifications and other background tasks.

## Deployment

- Docker
- Linux Server

## Monitoring

- Application logs
- Error monitoring
- Performance metrics

## Version Control

- Git
- GitHub

---

# 22. Initial Version Scope

The first version of the system will focus on:

- User registration and authentication
- 7-day free trial
- Subscription management
- Clothing management
- Browse and search
- Clothing availability
- Booking management
- Rental history
- Subscription payment
- Reviews and ratings
- Basic notifications
- Admin management

The following features will be considered later:

- Chat
- Advanced recommendations
- AI features
- Delivery/logistics
- Advanced analytics
- Referral system
- Coupons
- Loyalty program
- Advanced fraud detection

---

# 23. Future System Scaling

The initial system will start with a simple architecture.

As the number of users and requests increases, the system may
introduce:

- Redis caching
- Load balancing
- Multiple application servers
- Database indexes
- Database replication
- Database sharding
- Message queues
- Background workers
- CDN
- Object storage
- Monitoring and alerting

These technologies will be introduced only when required by
specific scaling problems.

---

# 24. Project Goal

The goal of this project is not only to build a clothes rental
application.

The goal is also to learn how to design and scale a real-world
system.

The system will initially be built as a simple application and
then gradually improved to support larger traffic and more users.

The development process will follow:

Requirements
     ↓
Database Design
     ↓
API Design
     ↓
High-Level Architecture
     ↓
Backend Implementation
     ↓
Testing
     ↓
Scaling
     ↓
System Design Improvements