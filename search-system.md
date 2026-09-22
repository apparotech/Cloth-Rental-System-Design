
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



## 2. Search Requirements & Query Design

The Search System needs to support different types of search
queries.

A user may search using only a keyword, only filters, or a
combination of keyword and filters.

The search API should convert these user inputs into a
structured search query.

---

## 2.1 Keyword Search

Users should be able to search using text.

Example:

```text
black shirt
````

The system should search relevant fields such as:

* title
* description
* brand
* category

Example:

```text
Search Query:
"black shirt"
```

Possible matching listings:

```text
Black Cotton Shirt
Black Formal Shirt
Black Casual Shirt
Black Linen Shirt
```

The search system should return the most relevant matching
results.

---

## 2.2 Filter Search

Users should also be able to search without entering a
keyword.

For example:

```text
category = SHIRT
gender = MEN
location = Delhi
```

The system should return clothing items matching these
conditions.

Example:

```text
GET /api/v1/clothing?category=SHIRT&gender=MEN&location=Delhi
```

---

## 2.3 Combined Search

Users can combine keyword search with filters.

Example:

```text
Keyword:
black shirt

Filters:
Gender = MEN
Location = Delhi
Price <= ₹500
Condition = GOOD
```

Conceptually:

```text
Search
AND
Gender = MEN
AND
Location = Delhi
AND
Price <= ₹500
AND
Condition = GOOD
```

Only clothing items matching the required conditions should
be returned.

---

## 2.4 Availability Filter

Availability is date-dependent.

A clothing item may be available for one date range but
unavailable for another date range.

Example:

```text
Requested dates:

September 20 → September 25
```

The search system should identify clothing items that do not
have conflicting blocking bookings for the requested dates.

The booking system remains responsible for the final
availability decision during booking creation.

Therefore:

```text
Search
   ↓
Show potentially available clothing
   ↓
User selects clothing
   ↓
Booking System performs authoritative availability check
```

Search results should not be treated as a reservation
guarantee.

---

## 2.5 Price Filter

Users should be able to specify a price range.

Example:

```text
Minimum Price = ₹200
Maximum Price = ₹500
```

Conceptually:

```text
pricePerDay >= 200
AND
pricePerDay <= 500
```

This allows users to discover clothing within their budget.

---

## 2.6 Sorting

The search system should support sorting.

Supported examples:

```text
price_asc
price_desc
newest
```

### Price Ascending

```text
₹200
₹300
₹400
₹500
```

### Price Descending

```text
₹1000
₹900
₹800
₹700
```

### Newest

Recently created clothing listings appear first.

Sorting should be performed by the search system rather than
fetching a large result set and sorting it inside the API
server.

---

## 2.7 Pagination

The API should not return every matching clothing item.

Example:

```text
page = 1
limit = 20
```

The response contains approximately 20 results.

A later request can retrieve the next page.

Example:

```text
page = 2
limit = 20
```

This prevents large responses and reduces unnecessary resource
usage.

---

## 2.8 Example Search Request

A complete search request could look like:

```text
GET /api/v1/clothing
    ?keyword=black+shirt
    &category=SHIRT
    &gender=MEN
    &location=Delhi
    &minPrice=200
    &maxPrice=500
    &condition=GOOD
    &sort=price_asc
    &page=1
    &limit=20
```

The API server receives these parameters and builds the
appropriate search query.

---

## 2.9 Search Query Processing

The high-level flow is:

```text
Client
   ↓
Search Request
   ↓
Load Balancer
   ↓
API Server
   ↓
Validate Search Parameters
   ↓
Build Search Query
   ↓
Search Database / Search Engine
   ↓
Apply Filters
   ↓
Apply Sorting
   ↓
Apply Pagination
   ↓
Return Results
```

---

## 2.10 Search Parameters

The main search parameters are:

| Parameter | Purpose                      |
| --------- | ---------------------------- |
| keyword   | Text-based search            |
| category  | Filter by clothing category  |
| gender    | Filter by target gender      |
| size      | Filter by clothing size      |
| brand     | Filter by brand              |
| condition | Filter by clothing condition |
| location  | Filter by location           |
| minPrice  | Minimum rental price         |
| maxPrice  | Maximum rental price         |
| sort      | Sorting option               |
| page      | Result page                  |
| limit     | Number of results            |

---

## 2.11 Example Query

Suppose the user searches:

```text
Keyword = black shirt
Gender = MEN
Location = Delhi
Max Price = ₹500
Sort = price_asc
Limit = 20
```

The system conceptually performs:

```text
Find clothing where:

keyword matches
AND
gender = MEN
AND
location = Delhi
AND
pricePerDay <= 500
AND
status = AVAILABLE

Sort by:

pricePerDay ASC

Return:

20 results
```

The exact query implementation will depend on whether the
system is using MongoDB or a dedicated search engine.

---

## 2.12 Search and Booking Responsibility

The Search System and Booking System have different
responsibilities.

### Search System

Responsible for:

* Finding relevant clothing
* Applying search filters
* Sorting results
* Pagination
* Fast discovery

### Booking System

Responsible for:

* Final availability verification
* Preventing double booking
* Creating the booking
* Maintaining booking state
* Concurrency control

Therefore, even if a search result says a clothing item is
available, the booking system must perform a final
availability check.

Example:

```text
User searches
      ↓
Search System
      ↓
Clothing appears available
      ↓
User clicks Rent
      ↓
Booking System
      ↓
Final availability check
      ↓
Create booking
```

This separation prevents the search layer from becoming the
source of truth for booking availability.

---

## 2.13 Search Design Principle

The Search System should optimize for:

* Fast read performance
* Flexible filtering
* Relevant keyword matching
* Efficient sorting
* Scalable pagination
* High search throughput

The Booking System should optimize for:

* Strong consistency
* Correct availability
* Concurrency control
* No double booking

This separation allows each part of the system to use the
appropriate architecture for its requirements.

````
