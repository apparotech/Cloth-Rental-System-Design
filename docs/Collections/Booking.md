 Booking Collection 📅

The Booking collection represents a rental request/booking made by a renter for a particular clothing item.

| Field             | Data Type | Required? | Purpose                    |
| ----------------- | --------- | --------- | -------------------------- |
| `_id`             | ObjectId  | Yes       | Unique booking ID          |
| `clothingId`      | ObjectId  | Yes       | Clothing being rented      |
| `renterId`        | ObjectId  | Yes       | User requesting the rental |
| `ownerId`         | ObjectId  | Yes       | Owner of the clothing      |
| `startDate`       | Date      | Yes       | Rental start date          |
| `endDate`         | Date      | Yes       | Rental end date            |
| `totalDays`       | Number    | Yes       | Number of rental days      |
| `pricePerDay`     | Number    | Yes       | Price at time of booking   |
| `rentalAmount`    | Number    | Yes       | Total rental cost          |
| `securityDeposit` | Number    | Yes       | Security deposit           |
| `status`          | Enum      | Yes       | Booking state              |
| `createdAt`       | Date      | Yes       | Booking creation time      |
| `updatedAt`       | Date      | Yes       | Last update time           |


{
  "_id": "ObjectId(...)",
  "clothingId": "ObjectId(clothing123)",
  "renterId": "ObjectId(user123)",
  "ownerId": "ObjectId(owner456)",

  "startDate": "2026-09-10",
  "endDate": "2026-09-13",
  "totalDays": 3,

  "pricePerDay": 800,
  "rentalAmount": 2400,
  "securityDeposit": 3000,

  "status": "PENDING",

  "createdAt": "2026-09-03T10:00:00Z",
  "updatedAt": "2026-09-03T10:00:00Z"
}