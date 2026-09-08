SubscriptionPlan Fields

| Field         | Data Type     | Required? | Purpose             |
| ------------- | ------------- | --------- | ------------------- |
| `_id`         | ObjectId      | Yes       | Unique plan ID      |
| `name`        | String        | Yes       | Plan name           |
| `description` | String        | No        | Description of plan |
| `price`       | Number        | Yes       | Plan price          |
| `duration`    | Number        | Yes       | Duration in days    |
| `features`    | Array[String] | No        | Features included   |
| `status`      | Enum          | Yes       | ACTIVE / INACTIVE   |
| `createdAt`   | Date          | Yes       | Creation time       |
| `updatedAt`   | Date          | Yes       | Last update time    |


Example
{
  "_id": "ObjectId(...)",
  "name": "Monthly",
  "description": "Monthly rental access",
  "price": 199,
  "duration": 30,
  "features": [
    "Upload clothes",
    "Rent clothes",
    "Unlimited bookings"
  ],
  "status": "ACTIVE",
  "createdAt": "2026-09-01T10:00:00Z",
  "updatedAt": "2026-09-01T10:00:00Z"
}