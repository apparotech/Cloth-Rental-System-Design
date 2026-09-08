
Notification Collection 

A Notification stores messages that the system sends to users.

For example:

"Your booking has been accepted."
"Your subscription expires in 2 days."
"Someone booked your clothing."
"Your clothing listing was blocked."

| Field       | Data Type | Required? | Purpose                         |
| ----------- | --------- | --------- | ------------------------------- |
| `_id`       | ObjectId  | Yes       | Unique notification ID          |
| `userId`    | ObjectId  | Yes       | User receiving the notification |
| `type`      | Enum      | Yes       | Type of notification            |
| `title`     | String    | Yes       | Notification title              |
| `message`   | String    | Yes       | Notification content            |
| `read`      | Boolean   | Yes       | Whether user has read it        |
| `createdAt` | Date      | Yes       | When notification was created   |


{
  "_id": "ObjectId(...)",
  "userId": "ObjectId(user123)",
  "type": "BOOKING_ACCEPTED",
  "title": "Booking Accepted",
  "message": "Your rental request for Black Suit has been accepted.",
  "read": false,
  "createdAt": "2026-09-10T10:30:00Z"
}