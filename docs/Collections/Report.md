8. Report Collection 🚨

Now our final collection.

A Report is created when a user reports another user or a clothing listing.

Our relationship is:

| Field          | Data Type   | Required? | Purpose                                  |
| -------------- | ----------- | --------- | ---------------------------------------- |
| `_id`          | ObjectId    | Yes       | Unique report ID                         |
| `reportedBy`   | ObjectId    | Yes       | User who submitted report                |
| `reportedUser` | ObjectId    | No        | User being reported                      |
| `clothingId`   | ObjectId    | No        | Clothing being reported                  |
| `reason`       | Enum/String | Yes       | Reason for report                        |
| `description`  | String      | No        | Additional details                       |
| `status`       | Enum        | Yes       | PENDING / REVIEWED / RESOLVED / REJECTED |
| `createdAt`    | Date        | Yes       | Report creation time                     |
| `updatedAt`    | Date        | Yes       | Last update                              |


{
  "_id": "ObjectId(...)",
  "reportedBy": "ObjectId(raj123)",
  "reportedUser": "ObjectId(aman456)",
  "clothingId": "ObjectId(clothing789)",
  "reason": "FAKE_LISTING",
  "description": "The uploaded clothing does not match the actual item.",
  "status": "PENDING",
  "createdAt": "2026-09-10T12:00:00Z",
  "updatedAt": "2026-09-10T12:00:00Z"
}