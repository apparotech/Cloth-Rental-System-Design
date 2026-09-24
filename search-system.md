
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



## 3. MongoDB Search - Initial Approach

In the initial version of the system, MongoDB can be used
directly for clothing search.

This keeps the architecture simple and avoids introducing a
dedicated search system before it is actually required.

The initial flow is:

Client
   ↓
Load Balancer
   ↓
API Server
   ↓
MongoDB
   ↓
Search Results
   ↓
Client

---

## 3.1 Why Start with MongoDB?

The Clothing collection already contains the fields required
for many search and filtering operations.

For example:

- category
- gender
- size
- brand
- condition
- location
- pricePerDay
- status
- createdAt

MongoDB indexes can be created on frequently queried fields.

Example:

```text
category → INDEX
gender → INDEX
location → INDEX
status → INDEX
pricePerDay → INDEX
createdAt → INDEX
````

This allows the database to efficiently handle many
structured queries.

---

## 3.2 Example MongoDB Search

Suppose a user searches for:

```text
Category = SHIRT
Gender = MEN
Location = Delhi
Price <= ₹500
Status = AVAILABLE
```

The database query is conceptually:

```text
category = SHIRT
AND
gender = MEN
AND
location = Delhi
AND
pricePerDay <= 500
AND
status = AVAILABLE
```

MongoDB can use appropriate indexes to locate matching
documents instead of scanning the entire Clothing collection.

---

## 3.3 Example Search Flow

```text
User
 ↓
GET /api/v1/clothing
 ↓
Load Balancer
 ↓
API Server
 ↓
Validate Query Parameters
 ↓
Build MongoDB Query
 ↓
MongoDB
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

## 3.4 MongoDB Indexes

Indexes are important because the system contains
approximately 5 million clothing listings.

Without an index, MongoDB may need to inspect a large number
of documents to find matching results.

For example:

```text
5,000,000 Clothing Documents
          ↓
     Search Query
          ↓
    Scan many documents
```

With an appropriate index:

```text
5,000,000 Clothing Documents
          ↓
        Index
          ↓
    Matching Documents
```

This can significantly reduce the amount of data that MongoDB
needs to examine.

---

## 3.5 Single-Field Indexes

The initial design can use indexes such as:

```text
category
gender
location
status
pricePerDay
createdAt
```

However, the system will often query multiple fields
together.

For example:

```text
category = SHIRT
gender = MEN
location = Delhi
status = AVAILABLE
```

Creating an individual index for every possible combination
is not practical.

This leads to the need for carefully designed compound
indexes.

---

## 3.6 Compound Indexes

A compound index contains multiple fields in a specific
order.

For example:

```text
(gender, category, status, location)
```

This can help queries that frequently filter using these
fields.

However, index order matters.

A compound index should be designed based on actual query
patterns rather than simply adding every searchable field
into one index.

For example, if the most common query is:

```text
gender
category
status
```

a compound index such as:

```text
(gender, category, status)
```

may be more useful than creating several unrelated indexes.

The exact index strategy should be validated using query
patterns and database query analysis.

---

## 3.7 Sorting with MongoDB

MongoDB can also perform sorting.

Example:

```text
sort = price_asc
```

Conceptually:

```text
pricePerDay ASC
```

Or:

```text
sort = newest
```

Conceptually:

```text
createdAt DESC
```

Indexes can also help with sorting when the query and index
patterns are designed appropriately.

---

## 3.8 Pagination with MongoDB

The initial implementation can use pagination.

Example:

```text
page = 1
limit = 20
```

The API returns 20 results.

A later request retrieves the next page.

For relatively small result sets, traditional page-based
pagination can be acceptable.

However, at very large offsets, using:

```text
skip()
```

for deep pages can become inefficient because the database
may still need to walk through skipped records.

For high-scale search, cursor-based pagination can be
considered.

---

## 3.9 Keyword Search Limitation

Structured filtering is not the only requirement.

The system also needs keyword search.

Example:

```text
black shirt
```

A simple structured query works well for:

```text
category = SHIRT
gender = MEN
location = Delhi
```

But text relevance is more complicated.

The system may need to:

* Match words across multiple fields
* Handle partial matches
* Rank relevant results
* Handle synonyms
* Support typo tolerance
* Calculate relevance

These requirements are where a dedicated search engine becomes
more useful.

---

## 3.10 MongoDB Search at Initial Scale

MongoDB can be a reasonable initial search solution when:

* Search requirements are simple.
* Most queries are structured filters.
* Dataset size is manageable.
* Search traffic can be handled by the database.
* The team wants to minimize infrastructure complexity.

For the initial version, MongoDB keeps the architecture
simpler:

```text
Client
   ↓
API Server
   ↓
MongoDB
```

There is no need to introduce another infrastructure
component before the search requirements justify it.

---

## 3.11 Limitation at Larger Scale

The system is expected to eventually handle:

* 5 million clothing listings
* 10 million search requests/day
* approximately 580 peak search RPS

At this scale, search can become a significant workload.

The same MongoDB cluster is also responsible for:

* Clothing writes
* Booking data
* User data
* Reviews
* Reports
* Subscription data
* Other application operations

If search traffic becomes heavy, running complex search
queries directly against the primary database can increase
database load.

This can affect other important operations.

---

## 3.12 Why We Need a Dedicated Search System

A dedicated search engine can separate search workload from
the main transactional database.

The architecture can evolve from:

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
API Server
   ↓
Search Engine
```

while MongoDB remains the source of truth.

The search engine can be optimized specifically for:

* Keyword search
* Full-text search
* Filtering
* Sorting
* Relevance
* Fast read queries
* High search traffic

---

## 3.13 Architecture Evolution

### Initial Architecture

```text
Client
   ↓
Load Balancer
   ↓
API Server
   ↓
MongoDB
```

### Scaled Search Architecture

```text
Client
   ↓
Load Balancer
   ↓
API Server
   ↓
Search Engine
```

MongoDB remains connected to the system as the primary
transactional database.

```text
                  ┌──────────────┐
                  │   MongoDB    │
                  │ Source of    │
                  │    Truth     │
                  └──────┬───────┘
                         │
                         │ Index Updates
                         ↓
                  ┌──────────────┐
                  │ Search Index │
                  └──────┬───────┘
                         │
                         ↓
                      Search
```

This allows the search workload to scale independently.

---

## 3.14 Important Design Principle

The system should not introduce a dedicated search engine
simply because it is commonly used.

The architecture should evolve based on actual requirements.

Therefore:

### Initial Stage

Use MongoDB for structured search and filtering.

### Later Stage

Introduce a dedicated search engine when:

* Search traffic increases.
* Full-text search becomes important.
* Relevance ranking is required.
* Search queries become more complex.
* Search workload starts affecting MongoDB performance.



## 4. Dedicated Search Engine

As the system grows, search can become a significant workload.

The system is expected to handle:

- Approximately 5 million clothing listings
- Approximately 10 million search requests per day
- Approximately 580 peak search requests per second

At this scale, a dedicated search engine can be introduced
to handle search workloads independently from MongoDB.

Possible technologies include:

- OpenSearch
- Elasticsearch

For this system design, we will use **OpenSearch** as the
example search engine.

The concepts are similar to Elasticsearch.

---

## 4.1 Why Use a Dedicated Search Engine?

MongoDB is primarily being used as the transactional
database.

It stores the authoritative clothing data.

However, search requires additional capabilities such as:

- Full-text search
- Keyword matching
- Relevance ranking
- Filtering
- Sorting
- Fast search over large datasets
- Search-specific indexing

A dedicated search engine is optimized for these workloads.

Therefore, the architecture can evolve to:

```text
Client
   ↓
Load Balancer
   ↓
API Server
   ↓
OpenSearch
   ↓
Search Results
````

MongoDB remains the source of truth.

---

## 4.2 MongoDB vs Search Engine

The responsibilities are separated.

### MongoDB

MongoDB is responsible for:

* Permanent clothing data
* Booking data
* User data
* Reviews
* Reports
* Subscription data
* Transactional operations

MongoDB is the source of truth.

### OpenSearch

OpenSearch is responsible for:

* Keyword search
* Full-text search
* Filtering
* Sorting
* Relevance ranking
* Fast search queries

OpenSearch is a searchable representation of clothing
data, not the primary source of truth.

---

## 4.3 Search Index

A search engine does not query MongoDB directly for every
search request.

Instead, searchable clothing data is copied into a
search index.

For example, a MongoDB clothing document may contain:

```text
{
    _id: "123",
    ownerId: "456",
    title: "Black Cotton Shirt",
    description: "Comfortable cotton shirt",
    category: "SHIRT",
    size: "L",
    gender: "MEN",
    brand: "Example Brand",
    condition: "GOOD",
    pricePerDay: 400,
    location: "Delhi",
    status: "AVAILABLE"
}
```

The search system maintains a corresponding searchable
document.

Conceptually:

```text
MongoDB Clothing Document
          ↓
     Indexing Process
          ↓
OpenSearch Search Document
```

---

## 4.4 Search Document

The OpenSearch document should contain the fields required
for searching, filtering, sorting, and displaying search
results.

Example:

```text
{
    "id": "123",
    "title": "Black Cotton Shirt",
    "description": "Comfortable cotton shirt",
    "category": "SHIRT",
    "size": "L",
    "gender": "MEN",
    "brand": "Example Brand",
    "condition": "GOOD",
    "pricePerDay": 400,
    "location": "Delhi",
    "status": "AVAILABLE"
}
```

The search document does not need to contain every field from
the MongoDB document.

Only fields required for search and result presentation
should be indexed.

---

## 4.5 Search Request Flow

With OpenSearch, the search flow becomes:

```text
Client
   ↓
Load Balancer
   ↓
API Server
   ↓
Validate Search Request
   ↓
Build OpenSearch Query
   ↓
OpenSearch
   ↓
Search Index
   ↓
Matching Results
   ↓
API Server
   ↓
Client
```

The API server acts as the entry point for search requests.

The client should not directly access OpenSearch.

---

## 4.6 Example Search

Suppose the user searches:

```text
black shirt
```

with:

```text
Gender = MEN
Location = Delhi
Maximum Price = ₹500
```

The API server converts these parameters into an OpenSearch
query.

Conceptually:

```text
Keyword:
black shirt

AND

Gender:
MEN

AND

Location:
Delhi

AND

Price:
<= 500
```

OpenSearch searches its index and returns matching documents.

---

## 4.7 Full-Text Search

One major advantage of a dedicated search engine is
full-text search.

For example, a user searches:

```text
black cotton shirt
```

The search engine can analyze the text and find relevant
documents containing related terms.

The search engine can also calculate relevance so that more
relevant results can appear earlier.

Conceptually:

```text
Search Query
     ↓
Text Analysis
     ↓
Term Matching
     ↓
Relevance Calculation
     ↓
Sorted Results
```

This is more sophisticated than a simple structured database
filter.

---

## 4.8 Structured Filters

OpenSearch can also apply structured filters.

Example:

```text
category = SHIRT
gender = MEN
location = Delhi
condition = GOOD
pricePerDay <= 500
status = AVAILABLE
```

The search engine can combine:

```text
Full-text search
+
Structured filters
+
Sorting
+
Pagination
```

within a single search request.

---

## 4.9 Relevance Ranking

Search results should not always be returned in arbitrary
order.

For keyword searches, the system can rank results based on
relevance.

For example, the user searches:

```text
black shirt
```

Possible results:

```text
1. Black Cotton Shirt
2. Black Formal Shirt
3. Black Casual Shirt
4. Blue Shirt with Black Pattern
```

The search engine calculates a relevance score and can return
more relevant matches earlier.

The exact ranking algorithm can be tuned later based on
product requirements.

---

## 4.10 Search Engine Does Not Replace MongoDB

Introducing OpenSearch does not mean MongoDB is removed.

The architecture becomes:

```text
                    ┌──────────────┐
                    │   MongoDB    │
                    │ Source of    │
                    │    Truth     │
                    └──────┬───────┘
                           │
                           │ Index Updates
                           ↓
                    ┌──────────────┐
                    │  OpenSearch  │
                    │ Search Index │
                    └──────┬───────┘
                           ↑
                           │
                      Search API
                           ↑
                           │
                         Client
```

MongoDB stores the actual application data.

OpenSearch stores an optimized searchable representation.

---

## 4.11 Search Result vs Source Data

The search engine can return enough information to display
the search result.

For example:

```text
id
title
image
pricePerDay
category
size
location
```

If the user opens the complete clothing details page, the
API can retrieve the authoritative clothing document from
MongoDB.

Conceptually:

```text
Search Page

Client
   ↓
OpenSearch
   ↓
Search Results


Clothing Details

Client
   ↓
API Server
   ↓
MongoDB
   ↓
Authoritative Clothing Data
```

This keeps MongoDB as the source of truth.

---

## 4.12 Benefits

Using a dedicated search engine provides:

* Faster full-text search
* Relevance-based results
* Efficient filtering
* Efficient sorting
* Independent search scaling
* Reduced search load on MongoDB
* Better support for complex search requirements

---

## 4.13 Trade-offs

A dedicated search engine also introduces additional
complexity.

The system now has another infrastructure component to
manage.

Additional concerns include:

* Search index synchronization
* Index failures
* Reindexing
* Monitoring
* Capacity planning
* Replication
* Search-engine availability

There can also be temporary differences between MongoDB and
OpenSearch because search indexing is generally asynchronous.

Therefore, the system must be designed to handle eventual
consistency.

---

## 4.14 Final Architecture for Search

The scaled search architecture becomes:

```text
                         Client
                           |
                           ↓
                    Load Balancer
                           |
                           ↓
                       API Servers
                           |
                           ↓
                    Search Module
                           |
                           ↓
                     OpenSearch
                           |
                           ↓
                    Search Results


                  MongoDB
               Source of Truth
                    |
                    |
              Index Updates
                    ↓
                OpenSearch
```

The important separation is:

```text
MongoDB
    ↓
Source of Truth

OpenSearch
    ↓
Search Optimization
```

This architecture allows the search workload to scale
independently while keeping the transactional data in
MongoDB.


## 5. Search Index Design

The Search Index contains a searchable representation of
clothing data stored in MongoDB.

MongoDB remains the source of truth.

OpenSearch stores only the fields required for:

- Keyword search
- Filtering
- Sorting
- Search result display

The goal is to keep the search document focused and avoid
indexing unnecessary data.

---

## 5.1 Clothing Search Document

A clothing document in MongoDB may contain:

```text
_id
ownerId
title
description
category
size
gender
brand
condition
images
pricePerDay
securityDeposit
location
status
createdAt
updatedAt
````

The OpenSearch document can contain:

```text
id
title
description
category
size
gender
brand
condition
image
pricePerDay
location
status
createdAt
```

The `ownerId`, `securityDeposit`, and other fields that are
not required for search do not necessarily need to be indexed.

---

## 5.2 Field Types

Different fields have different search requirements.

For example:

### Text Fields

Used for keyword search:

```text
title
description
brand
```

These fields may require text analysis.

Example:

```text
"Black Cotton Shirt"
```

can be analyzed into searchable terms.

---

### Keyword / Exact-Match Fields

Used for filtering:

```text
category
gender
size
condition
status
location
```

Example:

```text
gender = MEN
```

The system should match the exact filter value.

---

### Numeric Fields

Used for range filtering and sorting:

```text
pricePerDay
```

Example:

```text
pricePerDay <= 500
```

---

### Date Fields

Used for sorting and other time-based operations:

```text
createdAt
updatedAt
```

For example:

```text
Sort by newest listings
```

can use:

```text
createdAt DESC
```

---

## 5.3 Example Search Document

Conceptually, an OpenSearch document could look like:

```text
{
    "id": "clothing123",
    "title": "Black Cotton Shirt",
    "description": "Comfortable cotton shirt for rent",
    "category": "SHIRT",
    "size": "L",
    "gender": "MEN",
    "brand": "Example Brand",
    "condition": "GOOD",
    "image": "https://example.com/image.jpg",
    "pricePerDay": 400,
    "location": "Delhi",
    "status": "AVAILABLE",
    "createdAt": "2026-09-20T10:00:00Z"
}
```

The document is optimized for search operations.

---

## 5.4 Text Search Fields

The following fields are useful for keyword search:

```text
title
description
brand
category
```

For example, a user searches:

```text
black cotton shirt
```

The search engine can look for relevant terms across these
fields.

The search engine can then calculate relevance and return
the most relevant results.

---

## 5.5 Exact Filter Fields

Fields such as:

```text
gender
category
size
condition
status
```

are primarily used for exact filtering.

Example:

```text
gender = MEN
```

Another example:

```text
category = SHIRT
```

These filters can be combined with text search.

Example:

```text
Keyword:
black shirt

Filters:
gender = MEN
category = SHIRT
status = AVAILABLE
```

---

## 5.6 Numeric Fields

`pricePerDay` should support numeric operations.

Example:

```text
pricePerDay >= 200
AND
pricePerDay <= 500
```

This allows users to search within a price range.

It can also support sorting:

```text
pricePerDay ASC
```

or:

```text
pricePerDay DESC
```

---

## 5.7 Date Fields

`createdAt` can be used for sorting listings.

Example:

```text
sort = newest
```

Conceptually:

```text
createdAt DESC
```

This allows recently added clothing items to appear first.

---

## 5.8 Location

Location can be represented in different ways depending on
the final product requirements.

For the initial design, a location value can be used for
exact or normalized location filtering.

Example:

```text
location = Delhi
```

If the product later requires geographic search such as:

```text
Find clothes within 10 km
```

the search index can use geographic coordinates.

Conceptually:

```text
latitude
longitude
```

This would allow geo-distance queries.

The exact geographic implementation can be added when the
product requires location-radius search.

---

## 5.9 Search Result Fields

The search index should contain enough information to return
a useful search result without requiring another database
query for every result.

For example:

```text
id
title
image
pricePerDay
category
size
condition
location
```

The search API can return these fields directly.

Example:

```text
[
    {
        "id": "123",
        "title": "Black Cotton Shirt",
        "image": "...",
        "pricePerDay": 400,
        "category": "SHIRT",
        "size": "L",
        "location": "Delhi"
    }
]
```

---

## 5.10 Search Index Does Not Store Everything

The search index should not blindly copy the entire MongoDB
document.

For example, fields such as:

```text
password
payment information
private user information
```

must never be placed into the clothing search index.

Only data required for search and result presentation should
be indexed.

This reduces:

* Index size
* Storage requirements
* Unnecessary data duplication
* Security risk

---

## 5.11 Index Identifier

The OpenSearch document ID should correspond to the MongoDB
clothing ID.

Example:

```text
MongoDB:

_id = clothing123
```

OpenSearch:

```text
_document_id = clothing123
```

This makes it easier to:

* Update a search document
* Delete a search document
* Rebuild a specific document
* Identify the original MongoDB record

Conceptually:

```text
MongoDB Clothing ID
        =
OpenSearch Document ID
```

---

## 5.12 Search Index and MongoDB Relationship

The relationship is:

```text
                 MongoDB
              Source of Truth
                    |
                    |
              Indexing Events
                    |
                    ↓
                OpenSearch
               Search Index
```

MongoDB owns the actual clothing record.

OpenSearch maintains a searchable copy.

If the OpenSearch document becomes incorrect or is lost, it
can be rebuilt from MongoDB.

---

## 5.13 Reindexing

The system should support rebuilding the search index.

For example, if:

* Search index is corrupted
* Search mapping changes
* Search infrastructure is replaced
* New searchable fields are added

the system can read clothing records from MongoDB and rebuild
the OpenSearch index.

Conceptually:

```text
MongoDB
   ↓
Read Clothing Records
   ↓
Transform Documents
   ↓
OpenSearch Bulk Index
   ↓
New Search Index
```

This is another reason MongoDB must remain the source of
truth.

---

## 5.14 Search Index Design Principle

The Search Index should be:

* Optimized for search queries
* Smaller than the complete application dataset
* Focused on searchable fields
* Independent from transactional operations
* Rebuildable from MongoDB





## 6. Search Query Flow

The Search Query Flow describes how a user's search request
travels through the system and how OpenSearch returns the
matching clothing items.

For example, suppose a user searches for:

```text
Keyword: black shirt
Gender: MEN
Location: Delhi
Maximum Price: ₹500
Sort: price_asc
Limit: 20
````

The request passes through several components before the
results are returned.

---

## 6.1 High-Level Search Flow

```text
Client
   ↓
Load Balancer
   ↓
API Server
   ↓
Authentication / Validation
   ↓
Build Search Query
   ↓
OpenSearch
   ↓
Search Index
   ↓
Matching Results
   ↓
API Server
   ↓
Client
```

The client does not communicate directly with OpenSearch.

The API server acts as the controlled entry point to the
search system.

---

## 6.2 Step 1 — User Sends Search Request

The client sends a request such as:

```text
GET /api/v1/clothing
    ?keyword=black+shirt
    &gender=MEN
    &location=Delhi
    &maxPrice=500
    &sort=price_asc
    &page=1
    &limit=20
```

The request contains:

* Keyword
* Filters
* Sorting
* Pagination

---

## 6.3 Step 2 — Load Balancer

The request first reaches the Load Balancer.

```text
Client
   ↓
Load Balancer
```

The Load Balancer distributes the request across available
API servers.

For example:

```text
                    Load Balancer
                   /      |      \
                  ↓       ↓       ↓
               API 1    API 2    API 3
```

This allows the API layer to scale horizontally.

---

## 6.4 Step 3 — API Server Receives Request

The selected API server receives the search request.

The API server is responsible for:

* Validating query parameters
* Applying default values
* Validating allowed filter values
* Validating sorting options
* Validating pagination parameters
* Building the search query

For example:

```text
limit = 20
sort = price_asc
gender = MEN
```

The API should reject invalid values before sending the query
to OpenSearch.

---

## 6.5 Step 4 — Build Search Query

The API server converts the request parameters into a search
query.

User request:

```text
keyword = black shirt
gender = MEN
location = Delhi
maxPrice = 500
sort = price_asc
```

Conceptually becomes:

```text
Keyword:
black shirt

Filters:
gender = MEN
location = Delhi
pricePerDay <= 500

Status:
AVAILABLE

Sort:
pricePerDay ASC

Pagination:
20 results
```

The API server sends this query to OpenSearch.

---

## 6.6 Step 5 — OpenSearch Searches the Index

OpenSearch receives the search query.

```text
API Server
    ↓
OpenSearch
    ↓
Search Index
```

OpenSearch searches the indexed clothing documents.

It can combine:

* Full-text search
* Exact filters
* Numeric filters
* Sorting
* Pagination

in one search request.

---

## 6.7 Step 6 — Keyword Matching

The keyword:

```text
black shirt
```

is searched against relevant text fields such as:

```text
title
description
brand
category
```

The search engine identifies documents containing relevant
terms.

For example:

```text
Black Cotton Shirt
Black Formal Shirt
Black Casual Shirt
```

can be considered relevant matches.

---

## 6.8 Step 7 — Apply Filters

After identifying relevant documents, the search query applies
the requested filters.

Example:

```text
gender = MEN
```

Then:

```text
location = Delhi
```

Then:

```text
pricePerDay <= 500
```

And the listing should satisfy the required searchable
status.

Conceptually:

```text
Keyword Match
      ↓
Gender Filter
      ↓
Location Filter
      ↓
Price Filter
      ↓
Status Filter
```

The remaining documents are returned as search results.

---

## 6.9 Step 8 — Sorting

The user requested:

```text
sort = price_asc
```

Therefore, matching results are sorted by:

```text
pricePerDay ASC
```

Example:

```text
₹250
₹300
₹400
₹450
₹500
```

If the user instead selects:

```text
sort = price_desc
```

the order becomes:

```text
₹500
₹450
₹400
₹300
₹250
```

---

## 6.10 Step 9 — Pagination

The API request contains:

```text
page = 1
limit = 20
```

The search system returns only the required result set.

This prevents the API from returning thousands of documents
in a single response.

For large-scale search, cursor-based pagination can be used
instead of very deep page offsets.

---

## 6.11 Step 10 — OpenSearch Returns Results

OpenSearch returns the matching search documents.

Example:

```text
[
    {
        "id": "clothing123",
        "title": "Black Cotton Shirt",
        "image": "...",
        "pricePerDay": 300,
        "category": "SHIRT",
        "size": "L",
        "location": "Delhi"
    },
    {
        "id": "clothing456",
        "title": "Black Formal Shirt",
        "image": "...",
        "pricePerDay": 450,
        "category": "SHIRT",
        "size": "M",
        "location": "Delhi"
    }
]
```

---

## 6.12 Step 11 — API Server Formats Response

The API server receives the search results and converts them
into the application's response format.

The API may also include pagination metadata.

Example:

```text
{
    "data": [
        {
            "id": "clothing123",
            "title": "Black Cotton Shirt",
            "pricePerDay": 300,
            "location": "Delhi"
        }
    ],
    "pagination": {
        "page": 1,
        "limit": 20,
        "hasNextPage": true
    }
}
```

The exact response format can be standardized across the
application.

---

## 6.13 Step 12 — Return Results to Client

The API server sends the response back to the client.

```text
OpenSearch
    ↓
API Server
    ↓
Load Balancer
    ↓
Client
```

The user can then see the matching clothing listings.

---

## 6.14 Complete Search Flow

The complete flow is:

```text
User
  ↓
Search Request
  ↓
Load Balancer
  ↓
API Server
  ↓
Validate Parameters
  ↓
Build Search Query
  ↓
OpenSearch
  ↓
Keyword Matching
  ↓
Apply Filters
  ↓
Apply Sorting
  ↓
Apply Pagination
  ↓
Return Search Results
  ↓
API Server
  ↓
Client
```

---

## 6.15 Search Result and Booking Flow

Search and booking remain separate operations.

Example:

```text
User
  ↓
Search "black shirt"
  ↓
OpenSearch
  ↓
Search Results
  ↓
User selects clothing
  ↓
View Clothing Details
  ↓
Booking Request
  ↓
Booking System
  ↓
Final Availability Check
  ↓
Concurrency Control
  ↓
Create Booking
```

The search result does not guarantee that the clothing can
still be booked.

Another user may have created a conflicting booking between
the search request and the booking request.

Therefore, the Booking System must always perform the final
availability check.

---

## 6.16 Why Search is Separate from Booking

The two systems have different consistency requirements.

### Search

Optimized for:

* Speed
* Relevance
* Filtering
* Sorting
* High read traffic

Search data can tolerate a small amount of eventual
consistency.

### Booking

Optimized for:

* Correct availability
* Strong consistency
* Concurrency control
* Double-booking prevention

Therefore:

```text
Search
  ↓
OpenSearch

Booking
  ↓
MongoDB
```

This separation is an important part of the architecture.

---

## 6.17 Search Query Flow Summary

The search request follows:

```text
Client
   ↓
Load Balancer
   ↓
API Server
   ↓
Validate
   ↓
Build Query
   ↓
OpenSearch
   ↓
Search + Filter + Sort + Pagination
   ↓
Results
   ↓
API Server
   ↓
Client
```

The search engine is responsible for efficient search
operations, while the API server controls access and converts
client parameters into the appropriate search query.


### Next: Section 7 — Search Filtering

Add this to `docs/search-system.md`:

````markdown
## 7. Search Filtering

Search filtering allows users to narrow down clothing results
based on specific attributes.

For example, a user may search for:

- Men's shirts
- Size M
- Black color
- Good condition
- Location Delhi
- Price between ₹200 and ₹500

The search system should allow multiple filters to be applied
together in a single query.

---

### 7.1 Supported Filters

The clothing search supports the following filters:

| Filter | Example |
|---|---|
| Category | SHIRT |
| Gender | MEN |
| Size | M |
| Brand | Nike |
| Condition | GOOD |
| Location | Delhi |
| Price | ₹200 - ₹500 |
| Status | AVAILABLE |

These filters can be combined with keyword search.

Example:

```text
keyword = black shirt
gender = MEN
category = SHIRT
size = M
location = Delhi
minPrice = 200
maxPrice = 500
condition = GOOD
````

---

### 7.2 Filter Processing

The API server receives the search parameters and
converts them into a search query.

Example:

```text
Client
   |
   | Search Request
   ↓
API Server
   |
   | Validate Filters
   ↓
Build Search Query
   |
   ↓
OpenSearch
   |
   | Apply Filters
   ↓
Matching Documents
   |
   ↓
API Server
   |
   ↓
Client
```

OpenSearch applies the filters against the indexed clothing
documents.

---

### 7.3 Keyword Search vs Filters

Keyword search and filters serve different purposes.

#### Keyword Search

Used for text-based matching.

Example:

```text
"black shirt"
```

The search engine may match:

* title
* description
* brand
* category

#### Filters

Used for exact or structured conditions.

Example:

```text
gender = MEN
size = M
condition = GOOD
location = Delhi
```

A typical search request may combine both.

```text
Keyword:
"black shirt"

Filters:
gender = MEN
size = M
location = Delhi
maxPrice = 500
```

This allows the search engine to first identify relevant
text matches and then apply the required structured filters.

---

### 7.4 Price Filtering

Price filtering uses a range.

Example:

```text
minPrice = 200
maxPrice = 500
```

The search system returns clothing where:

```text
200 <= pricePerDay <= 500
```

The price range should be handled by the search engine rather
than retrieving a large number of documents and filtering them
inside the API server.

This reduces unnecessary data transfer and application-server
processing.

---

### 7.5 Location Filtering

Initially, location can be treated as a structured field.

Example:

```text
location = Delhi
```

Later, the system can support geographic coordinates:

```text
latitude
longitude
```

This would allow more advanced queries such as:

```text
Find clothes within 10 km of the user
```

For example:

```text
User Location
     |
     ↓
Latitude / Longitude
     |
     ↓
OpenSearch Geo Query
     |
     ↓
Clothing within specified distance
```

This can be useful when the platform grows to support
location-based rental discovery.

---

### 7.6 Availability Filtering

Availability is different from normal filters because
availability depends on booking dates.

For example:

```text
startDate = 2026-09-10
endDate   = 2026-09-13
```

The search system can help identify clothing that is
potentially available, but it should not be treated as the
final authority for booking availability.

The Booking System and MongoDB remain the source of truth
for preventing double booking.

Therefore:

```text
Search System
     |
     | Potential availability
     ↓
Search Results
     |
     ↓
Booking Request
     |
     ↓
Booking System
     |
     | Authoritative availability check
     ↓
MongoDB
```

This separation prevents stale search data from causing
incorrect bookings.

---

### 7.7 Combining Multiple Filters

Users should be able to combine multiple filters.

Example request:

```text
GET /api/v1/clothing?
keyword=black+shirt
&category=SHIRT
&gender=MEN
&size=M
&location=Delhi
&minPrice=200
&maxPrice=500
&condition=GOOD
&sort=price_asc
```

Conceptually, the search query becomes:

```text
keyword matches "black shirt"
AND
category = SHIRT
AND
gender = MEN
AND
size = M
AND
location = Delhi
AND
condition = GOOD
AND
pricePerDay >= 200
AND
pricePerDay <= 500
```

The search engine performs these filtering operations
before returning the final result set.

---

### 7.8 Why Filtering is Handled by the Search Engine

At larger scale, filtering inside the application server
would require retrieving many documents before applying
conditions.

For example:

```text
MongoDB
   |
   | 100,000 documents
   ↓
API Server
   |
   | Filter documents
   ↓
1,000 matching results
```

This creates unnecessary network traffic and application
processing.

With a dedicated search engine:

```text
API Server
   |
   | Search + Filters
   ↓
OpenSearch
   |
   | Filter internally
   ↓
20 matching results
   |
   ↓
API Server
```

Only the required search results are returned to the
application.

This improves search performance and allows the search
workload to scale independently from the transactional
database.

---

### 7.9 Important Design Principle

The Search System is responsible for:

* Keyword search
* Filtering
* Sorting
* Pagination
* Search relevance

The Booking System is responsible for:

* Final availability validation
* Double-booking prevention
* Booking creation
* Booking state transitions

MongoDB remains the authoritative source for transactional
booking data.

This separation allows the search system to remain optimized
for fast discovery without weakening booking consistency.



