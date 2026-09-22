
# Clothes Rental System - Search System

## 1. Search System Overview

The Search System allows users to discover clothing items
available for rental.

Users should be able to search and filter clothing based on
different attributes such as:

- Keyword
- Category
- Gender
- Size
- Brand
- Condition
- Location
- Price range
- Availability

Users can also sort the search results based on different
criteria.

Examples:

- Search for "black shirt"
- Find men's shirts in Delhi
- Find clothes below ₹500/day
- Find available dresses in a particular location
- Sort clothes by lowest price
- Sort clothes by newest listings

---

## 1.1 Search Scale

According to the system's scale estimation:

- Clothing listings: approximately 5 million
- Search requests: approximately 10 million/day
- Average search traffic: approximately 116 RPS
- Peak search traffic: approximately 580 RPS

Therefore, search is one of the highest-traffic operations
in the system.

The search system should be designed to handle high read
traffic without putting excessive load on the primary
database.

---

## 1.2 Search Requirements

The search system should support:

### Keyword Search

Users can search using keywords.

Example:

```text
black shirt
````

The system should search relevant clothing information such
as:

* Title
* Description
* Brand
* Category

---

### Filtering

Users can filter results using attributes such as:

```text
Category
Gender
Size
Brand
Condition
Location
Price
Availability
```

Example:

```text
Category = SHIRT
Gender = MEN
Location = Delhi
Price <= ₹500/day
```

---

### Sorting

Users should be able to sort search results.

Examples:

```text
Price: Low to High
Price: High to Low
Newest Listings
```

---

### Pagination

Search results should be returned in pages rather than
returning all matching clothing items at once.

Example:

```text
page = 1
limit = 20
```

The API returns only the required number of results.

This reduces:

* Network payload
* API memory usage
* Database/search-engine workload
* Client-side processing

---

## 1.3 Initial Search Approach

In the initial version of the system, search can be performed
directly using MongoDB.

Basic flow:

```text
Client
   ↓
API Server
   ↓
MongoDB
   ↓
Search Results
   ↓
Client
```

MongoDB can handle structured filtering using fields such as:

```text
category
gender
size
brand
condition
location
pricePerDay
status
```

Indexes can be created for frequently used filters.

For example:

```text
category → INDEX
gender → INDEX
location → INDEX
status → INDEX
```

This approach is suitable for the initial version of the
system.

---

## 1.4 Search Evolution

As the number of clothing listings and search requests
increases, the search workload can become significant.

The system can evolve from:

```text
Client
   ↓
API Server
   ↓
MongoDB
```

to:

```text
Client
   ↓
Load Balancer
   ↓
API Server
   ↓
Search System
   ↓
Search Index
```

A dedicated search engine such as OpenSearch or Elasticsearch
can be introduced when search requirements and scale justify
the additional infrastructure.

The search engine maintains a separate search index optimized
for fast searching, filtering, and sorting.

MongoDB remains the source of truth for clothing data.

---

## 1.5 Source of Truth

The search index should not become the primary source of
clothing data.

MongoDB remains the authoritative database.

The search engine contains a searchable representation of
clothing data.

Conceptually:

```text
MongoDB
   |
   | Index / Update
   ↓
Search Index
```

If a clothing document is updated in MongoDB, the
corresponding search index should also be updated.

---

## 1.6 Design Goal

The main goals of the Search System are:

1. Provide fast clothing discovery.
2. Support keyword search.
3. Support multiple filters.
4. Support sorting.
5. Support pagination.
6. Handle high search traffic.
7. Reduce unnecessary load on MongoDB.
8. Allow the search infrastructure to scale independently.
9. Keep MongoDB as the source of truth.
10. Handle search-index failures without losing the actual
    clothing data.



