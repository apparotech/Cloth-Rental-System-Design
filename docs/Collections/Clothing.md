
 Clothing Fields


| Field             | Data Type     | Required? | Purpose                         |
| ----------------- | ------------- | --------- | ------------------------------- |
| `_id`             | ObjectId      | Yes       | Unique clothing ID              |
| `ownerId`         | ObjectId      | Yes       | User who owns the clothing      |
| `title`           | String        | Yes       | Name/title of clothing          |
| `description`     | String        | No        | Detailed description            |
| `category`        | Enum/String   | Yes       | Shirt, Jeans, Saree, etc.       |
| `size`            | Enum/String   | Yes       | S, M, L, XL, etc.               |
| `gender`          | Enum/String   | Yes       | Men, Women, Unisex              |
| `brand`           | String        | No        | Brand name                      |
| `condition`       | Enum          | Yes       | NEW, LIKE_NEW, GOOD, FAIR       |
| `images`          | Array[String] | Yes       | Clothing image URLs             |
| `pricePerDay`     | Number        | Yes       | Rental price per day            |
| `securityDeposit` | Number        | Yes       | Security deposit amount         |
| `location`        | String/Object | Yes       | Where the clothing is available |
| `status`          | Enum          | Yes       | AVAILABLE, UNAVAILABLE, BLOCKED |
| `createdAt`       | Date          | Yes       | Creation time                   |
| `updatedAt`       | Date          | Yes       | Last update time                |


{
  "_id": "ObjectId(...)",
  "ownerId": "ObjectId(user123)",
  "title": "Black Formal Suit",
  "description": "Black formal suit suitable for weddings and events.",
  "category": "SUIT",
  "size": "L",
  "gender": "MEN",
  "brand": "Van Heusen",
  "condition": "LIKE_NEW",
  "images": [
    "https://example.com/suit1.jpg",
    "https://example.com/suit2.jpg"
  ],
  "pricePerDay": 800,
  "securityDeposit": 3000,
  "location": "Silchar",
  "status": "AVAILABLE",
  "createdAt": "2026-09-03T10:00:00Z",
  "updatedAt": "2026-09-03T10:00:00Z"
}