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