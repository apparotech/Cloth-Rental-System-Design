# Clothes Rental System - High-Level Architecture

## 1. Overview

The system uses a horizontally scalable architecture
designed to handle high read traffic and large-scale
image storage.

The architecture separates synchronous API operations
from asynchronous background processing.

---

## 2. Major Components

The initial architecture contains:

1. Clients
2. Load Balancer
3. API Servers
4. Redis Cache
5. MongoDB Database
6. Object Storage
7. CDN
8. Message Queue
9. Background Workers
10. Search System
11. Payment Gateway
12. Monitoring System

---

## 3. High-Level Request Flow

Client
  |
  ↓
Load Balancer
  |
  ↓
API Servers
  |
  ├── Redis
  ├── MongoDB
  ├── Search System
  └── Message Queue
          |
          ↓
      Background Workers

---

## 4. Load Balancer

The Load Balancer distributes incoming requests
across multiple API server instances.

Example:

Load Balancer
    |
    ├── API Server 1
    ├── API Server 2
    └── API Server 3

This allows horizontal scaling by adding more API
server instances as traffic increases.

---

## 5. Stateless API Servers

API servers should remain stateless.

Important shared state should not depend on the local
memory of a single API server.

Shared state can be stored in:

- Redis
- MongoDB
- Other shared infrastructure

Stateless API servers make horizontal scaling easier.

---

## 6. Redis Cache

Redis is used to reduce database load for frequently
accessed data.

Potential cache candidates include:

- Popular clothing listings
- Clothing details
- Search results
- Frequently accessed metadata

Basic cache flow:

Client
  |
  ↓
API Server
  |
  ↓
Redis
  |
  ├── Cache HIT → Return data
  |
  └── Cache MISS
          ↓
       MongoDB
          ↓
       Redis
          ↓
       Return data

---

## 7. Object Storage

Clothing images are stored in object storage instead
of MongoDB.

MongoDB stores image URLs/references.

This separates large media storage from transactional
database data.

---

## 8. CDN

A CDN is used to serve frequently accessed clothing
images.

Request flow:

Client
  |
  ↓
CDN
  |
  ├── Cache HIT → Return image
  |
  └── Cache MISS
          ↓
     Object Storage

This reduces latency and object-storage access.

---

## 9. Message Queue

A message queue is used for asynchronous processing.

Examples:

- Notifications
- Email
- Image processing
- Search indexing
- Background jobs

This prevents non-critical background operations from
blocking synchronous API requests.

---

## 10. Background Workers

Workers consume messages from the queue and perform
background tasks independently of the API servers.

---

## 11. Scalability

The API layer can scale horizontally:

3 API servers
      ↓
10 API servers
      ↓
50 API servers

Additional infrastructure such as caching, read
replicas, search systems, and queues can be introduced
as traffic and data requirements increase.

---

# 12. Architecture Style

## 12.1 Initial Architecture

The system will initially follow a **Modular Monolith**
architecture.

The backend will be deployed as a single application
but internally divided into well-defined business modules.

### Backend Modules

- Auth & User
- Clothing
- Booking
- Review
- Notification
- Report
- Subscription
- Payment
- Admin

---

## 12.2 Why Modular Monolith?

The estimated peak traffic is approximately 3,000 RPS.

This traffic can initially be handled by horizontally
scaling stateless API servers.

Starting with microservices would introduce additional
operational complexity such as:

- Service-to-service communication
- Distributed transactions
- Service discovery
- Distributed tracing
- Multiple deployments
- Increased infrastructure management

Therefore, the initial architecture prioritizes clear
module boundaries while keeping deployment and operations
relatively simple.

---

## 12.3 Future Microservice Migration

The modular boundaries allow individual modules to be
extracted into independent services if required.

Potential candidates include:

- Booking Service
- Search Service
- Notification Service
- Payment Service
- User Service

For example:

Modular Monolith
      |
      ↓
Booking Module
      |
      ↓
Booking Service

This allows the architecture to evolve based on actual
traffic, scaling requirements, and business needs.

---

## 12.4 Architecture Evolution

### Stage 1

Modular Monolith
+ Redis
+ MongoDB
+ Message Queue
+ Background Workers

### Stage 2

Modular Monolith
+ Dedicated Search
+ Redis
+ Database Read Replicas
+ Message Queue
+ Background Workers

### Stage 3

API Gateway
+ Independent domain services
+ Dedicated databases where required
+ Redis
+ Message Queue
+ Search Infrastructure