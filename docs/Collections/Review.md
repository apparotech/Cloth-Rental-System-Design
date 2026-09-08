Review Collection ⭐



A Review represents feedback given by a renter after they have rented a clothing item.
Our relationships are:


| Field        | Data Type | Required? | Purpose                            |
| ------------ | --------- | --------- | ---------------------------------- |
| `_id`        | ObjectId  | Yes       | Unique review ID                   |
| `userId`     | ObjectId  | Yes       | User who wrote the review          |
| `clothingId` | ObjectId  | Yes       | Clothing being reviewed            |
| `bookingId`  | ObjectId  | Yes       | Booking associated with the review |
| `rating`     | Number    | Yes       | Rating, usually 1–5                |
| `comment`    | String    | No        | User's written feedback            |
| `createdAt`  | Date      | Yes       | Review creation time               |
| `updatedAt`  | Date      | Yes       | Last update time                   |


{
  "_id": "ObjectId(...)",
  "userId": "ObjectId(raj123)",
  "clothingId": "ObjectId(suit456)",
  "bookingId": "ObjectId(booking789)",
  "rating": 5,
  "comment": "Very good quality and clean.",
  "createdAt": "2026-09-20T10:00:00Z",
  "updatedAt": "2026-09-20T10:00:00Z"
}
