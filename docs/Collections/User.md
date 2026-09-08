1. User Collection

The User represents every person using our platform.

A user can be both:

👕 Clothing Owner — uploads clothes
🛍️ Renter — rents clothes

So we do not need separate Owner and Renter collections.

User fields
| Field          | Data Type | Required? | Purpose                    |
| -------------- | --------- | --------- | -------------------------- |
| `_id`          | ObjectId  | Yes       | Unique user ID             |
| `name`         | String    | Yes       | User's name                |
| `email`        | String    | Yes       | Login/contact email        |
| `phone`        | String    | Yes       | User's phone number        |
| `password`     | String    | Yes       | Hashed password            |
| `profileImage` | String    | No        | Profile image URL          |
| `status`       | Enum      | Yes       | ACTIVE / BLOCKED / DELETED |
| `createdAt`    | Date      | Yes       | Account creation time      |
| `updatedAt`    | Date      | Yes       | Last update time           |


