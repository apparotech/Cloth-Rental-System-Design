# Clothes Rental System - High-Level Architecture

## 1. Overview

This document describes the high-level architecture of the
Clothes Rental System.

The system is designed as a scalable peer-to-peer clothes
rental platform where users can both list clothes and rent
clothes from other users.

The architecture is designed around the estimated scale of:

- 10 million registered users
- 2 million daily active users
- Approximately 3,000 peak API requests per second
- Approximately 580 peak search requests per second
- Approximately 12 peak booking creations per second
- Approximately 25 TB of clothing image storage

The architecture focuses on scalability, availability,
performance, consistency, and future extensibility.

---

## 2. Architecture Style

The initial system uses a **Modular Monolith** architecture.

The backend is deployed as multiple stateless API server
instances behind a Load Balancer.

The application is internally divided into clear business
modules:

- Authentication & User
- Clothing
- Booking
- Review
- Subscription
- Payment
- Notification
- Report
- Admin

Although these modules initially run within the same application,
their boundaries are kept clear so that high-scale or
independently evolving modules can later be extracted into
separate services.

### Why Modular Monolith?

A microservices architecture is not required from the beginning
because the estimated traffic can initially be handled through
horizontal scaling of stateless API servers.

Starting with microservices would introduce additional
operational complexity such as:

- Service-to-service communication
- Distributed transactions
- Service discovery
- Multiple deployments
- Distributed tracing
- Increased infrastructure complexity

Therefore, the initial architecture keeps the application
simple while maintaining clear domain boundaries for future
service extraction.

---

## 3. High-Level Components

The system contains the following major components:

1. Web / Mobile Client
2. Load Balancer
3. API Servers
4. Modular Backend
5. MongoDB
6. Redis
7. Object Storage
8. CDN
9. Message Queue
10. Background Workers
11. Search System
12. Payment Gateway
13. Monitoring System

---

## 4. High-Level Request Flow

The basic request flow is:

Client

↓

Load Balancer

↓

API Server

↓

Modular Backend

↓

Database / Cache / Queue / External Services

Multiple API servers are deployed behind the Load Balancer
so that incoming traffic can be distributed across available
instances.

---

## 5. Client Layer

The system can be accessed through:

- Web Application
- Mobile Application

The client communicates with the backend through REST APIs.

The client does not directly communicate with the database
or internal infrastructure components.

---

## 6. Load Balancer

The Load Balancer distributes incoming requests across
multiple API server instances.

Example:

Client

↓

Load Balancer

↓

┌──────────┬──────────┬──────────┐
↓          ↓          ↓
API 1      API 2      API 3

If traffic increases, additional API servers can be added.

Example:

3 API servers

↓

10 API servers

↓

20 API servers

This is known as **horizontal scaling**.

### Responsibilities

- Distribute incoming requests
- Prevent overloading a single API server
- Route traffic to healthy instances
- Support horizontal scaling
- Remove unhealthy instances from traffic

---

## 7. API Server Layer

API servers handle requests from clients and execute the
business logic defined by the modular backend.

The API servers are designed to be **stateless**.

This means that no important user session or application state
should depend on the memory of a particular API server.

Shared state is stored in systems such as:

- MongoDB
- Redis
- Message Queue

This allows any API server to handle any incoming request.

---

## 8. Modular Backend

The backend is divided into business modules.

### Authentication & User

Responsible for:

- Registration
- Login
- Authentication
- User profile
- Password management
- Account status

### Clothing

Responsible for:

- Clothing creation
- Clothing updates
- Clothing deletion
- Clothing details
- Clothing availability
- Listing management

### Booking

Responsible for:

- Booking creation
- Availability checks
- Double-booking prevention
- Booking acceptance/rejection
- Booking cancellation
- Booking lifecycle

### Review

Responsible for:

- Creating reviews
- Updating reviews
- Deleting reviews
- Clothing ratings

### Subscription

Responsible for:

- Subscription plans
- Free trial
- Subscription activation
- Subscription expiration
- Subscription history

### Payment

Responsible for:

- Subscription payment creation
- Payment status
- Payment verification
- Payment webhook processing

### Notification

Responsible for:

- Booking notifications
- Subscription notifications
- Review notifications
- System notifications

### Report

Responsible for:

- User reports
- Clothing reports
- Report status
- Admin review

### Admin

Responsible for:

- User management
- Clothing moderation
- Reports
- Subscription plans
- System statistics

---

## 9. MongoDB

MongoDB is the primary database for the system.

It stores structured application data such as:

- Users
- Clothing
- Bookings
- Reviews
- Notifications
- Reports
- Subscriptions
- Subscription Plans

The database design is documented separately in:

`database-design.md`

MongoDB indexes are used for frequently accessed query fields.

Examples include:

- User email
- Clothing owner
- Clothing category
- Clothing status
- Booking clothing
- Booking renter
- Booking owner
- Subscription user
- Notification user + read status

---

## 10. Redis

Redis is used as a high-speed caching layer.

The system is expected to be read-heavy, especially for:

- Clothing browsing
- Clothing details
- Search results
- Frequently accessed listing data

Basic flow:

Client

↓

API Server

↓

Redis

↓

MongoDB

### Cache Hit

If the requested data exists in Redis:

Client

↓

API Server

↓

Redis

↓

Return Response

### Cache Miss

If the requested data does not exist in Redis:

Client

↓

API Server

↓

Redis

↓

MongoDB

↓

Store Result in Redis

↓

Return Response

Caching reduces repeated database reads and improves
response latency.

Detailed caching strategy will be documented separately.

---

## 11. Object Storage

Clothing images are stored in object storage rather than
inside MongoDB.

MongoDB stores only the image URL or object reference.

Example:

Client

↓

Object Storage

↓

Image URL

↓

MongoDB

This prevents large image files from increasing the size
of the primary database.

The scale estimation predicts approximately 25 TB of image
storage for the initial dataset.

---

## 12. CDN

A Content Delivery Network (CDN) is placed in front of
object storage for serving frequently requested images.

Request flow:

Client

↓

CDN

↓

Object Storage

### Cache Hit

CDN returns the image directly.

### Cache Miss

CDN retrieves the image from object storage and then serves
it to the client.

The CDN helps:

- Reduce image latency
- Reduce object-storage requests
- Reduce application-server load
- Improve performance for geographically distributed users

---

## 13. Message Queue

A Message Queue is used for asynchronous operations.

Instead of making the API server wait for every secondary
operation, the system can publish an event to the queue.

Example:

API Server

↓

Message Queue

↓

Background Worker

This can be used for:

- Booking notifications
- Email notifications
- Image processing
- Search indexing
- Subscription reminders
- Booking expiration jobs

The queue also helps isolate background workloads from
user-facing API traffic.

---

## 14. Background Workers

Background workers consume messages from the Message Queue
and perform asynchronous operations.

Example:

Booking Created

↓

Message Queue

↓

Notification Worker

↓

Notify Clothing Owner

If the notification service is temporarily unavailable,
the API request does not necessarily need to fail.

The queue can retain the message for retry processing.

---

## 15. Search System

Search traffic is expected to reach approximately
580 requests per second during peak traffic.

The initial system can use database indexes for basic
search and filtering.

As the number of listings and search traffic grows, a
dedicated search system can be introduced.

Possible responsibilities:

- Full-text search
- Filtering
- Sorting
- Category search
- Location-based search
- Price filtering

The search system can be updated asynchronously through
the Message Queue.

Example:

Clothing Created

↓

MongoDB

↓

Clothing Created Event

↓

Message Queue

↓

Search Index Worker

↓

Search System

---

## 16. Payment Gateway

The system uses an external payment gateway for
subscription payments.

The payment flow is:

User

↓

API Server

↓

Payment Gateway

↓

Payment Processing

↓

Payment Webhook

↓

API Server

↓

Update Subscription

Payment webhook requests must be verified before changing
subscription or payment status.

Rental payments between renter and clothing owner are
handled directly between the two parties and are not
processed as platform commission transactions.

---

## 17. Monitoring

The system requires monitoring for:

- API latency
- Error rate
- Request rate
- Database performance
- Cache hit/miss ratio
- Queue depth
- Worker failures
- Payment failures
- Booking failures
- Server health

Logs, metrics, and alerts should be collected centrally
to help identify production issues.

---

## 18. Horizontal Scaling

The API layer can scale horizontally.

Example:

3 API servers

↓

10 API servers

↓

20 API servers

The Load Balancer distributes requests across these
instances.

Because the API servers are stateless, adding or removing
instances does not require moving user session state between
servers.

Other components can also scale independently based on
their workload.

---

## 19. Initial Architecture

The initial architecture can be summarized as:

Client

↓

Load Balancer

↓

Multiple API Servers

↓

Modular Backend

↓

┌────────────┬────────────┬──────────────┐
↓            ↓            ↓
Redis      MongoDB       Message Queue
                            ↓
                         Workers

For media:

Client

↓

CDN

↓

Object Storage

This architecture provides a foundation for scaling the
system while keeping the initial operational complexity
manageable.

---

## 20. Future Evolution

As the system grows, individual components can be scaled
or extracted independently.

### Stage 1

Modular Monolith

+

MongoDB

+

Redis

+

Message Queue

+

Background Workers

### Stage 2

Modular Monolith

+

Dedicated Search System

+

Redis

+

Database Read Replicas

+

Message Queue

### Stage 3

Selected high-scale modules can be extracted into
independent services.

Potential candidates include:

- Booking Service
- Search Service
- Notification Service
- Payment Service
- User Service

The system does not need to become microservices-first.
Service extraction should be driven by actual scale,
independent deployment requirements, and operational needs.

---

## 21. Architecture Principles

The architecture follows these principles:

- Stateless API servers
- Horizontal scaling
- Database indexing
- Caching for read-heavy workloads
- Object storage for large media
- CDN for image delivery
- Asynchronous processing using queues
- Background workers for non-critical operations
- Strong consistency for booking operations
- Idempotent processing for background jobs
- Clear modular boundaries
- Independent scalability where required
- Monitoring and observability