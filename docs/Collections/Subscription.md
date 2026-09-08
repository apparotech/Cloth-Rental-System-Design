Subscription Collection

| Field       | Data Type | Required? | Purpose                              |
| ----------- | --------- | --------- | ------------------------------------ |
| `_id`       | ObjectId  | Yes       | Unique subscription ID               |
| `userId`    | ObjectId  | Yes       | User who subscribed                  |
| `planId`    | ObjectId  | Yes       | Selected plan                        |
| `status`    | Enum      | Yes       | TRIAL / ACTIVE / EXPIRED / CANCELLED |
| `startDate` | Date      | Yes       | Subscription start                   |
| `endDate`   | Date      | Yes       | Subscription expiry                  |
| `paymentId` | String    | No        | Payment gateway transaction ID       |
| `createdAt` | Date      | Yes       | Creation time                        |
| `updatedAt` | Date      | Yes       | Last update time                     |


{
  "_id": "ObjectId(...)",
  "userId": "ObjectId(user123)",
  "planId": "ObjectId(plan456)",
  "status": "ACTIVE",
  "startDate": "2026-09-01T00:00:00Z",
  "endDate": "2026-10-01T00:00:00Z",
  "paymentId": "pay_123456",
  "createdAt": "2026-09-01T10:00:00Z",
  "updatedAt": "2026-09-01T10:00:00Z"
}