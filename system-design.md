# Final Project: Car booking

## Functinal requirements
- User Registration and Login:
    - The system should allow users to create an account by providing password, phone number, and personal details (e.g., name, dob information).
    - Users should be able to log in using their registered username and password.
    - Users can modify their information
    - Users can be either Drivers or Customers
    - Driver have to created with their vehicle information
- Admin control:
    - Admins should be able to block/delete users
    - Admins should be able to manage user accounts, view booking details, and access reports on booking statistics, revenue, and user feedback.
- Booking a Car:
    - Users can book a vehical from current position to destination position
    - The system should calculate and display the total rental cost based on the selected dates and daily rental rate.
    - Users can wait max up to 30 seconds to check if any driver is available
    - Users can received notifications after the ride is finished.
- Booking History and Status Tracking:
    - Users should be able to view their booking history, showing past and current bookings, dates, total charges, and booking statuses.
    - Users should be able to track active bookings, including pickup and drop-off schedules.
- Review:
    - Users can write review and rate driver after every transaction

## Non Functional requirements
- The system should log all booking transactions with a retention period of 1 year.
- The system should ensure that no bookings are duplicated or lost during transactions, even in case of system failures.
- The system should priority drivers with higher rate point
- The system should priority customers with longer wait time.

## Technical Details
- User Registration and Login:
    - The system support login by token (JWT)
    - The system save hased passwords
    - The system return both refresh token and access token for every login. Refresh token can be used to re-create an access token.
    - Admin is just a user with high priority. No need to seperated between entities user and admin
- Booking a Car:
    - Drivers update their position every 30 seconds
    - When a user books a drive, the system looks for nearest driver and assign that driver for user.
        -  If the distant of nearest driver and user > MAX_ALLOW_DISTANCE, cancel that transaction. Keep looking again
        -  The distance between user and driver = distance(pos(user) - post(driver)) + driver_average_rate. It means that even a driver 
           is nearer to a customer, the system can assign to a farther driver if that driver is rated low
    - The system will support long polling for this api (up to 30 seconds)
    - The system searches for matched driver up to 2 seconds. If timeout, the system, searches again for one with higher priority
    - Users booking request and matched drivers will be exchanged over Message Queue (Even service are monolithic). If the system is monolithic, the system could use an in-memory Message Queue (implemented by Queue) else the system use a distributed Message Queue (RabbitMQ)
    - Notifications will be sent async (Message Queue). The system will use 3rd providers to send notifications.
- Booking History and Status Tracking:
    - The system saved car history and allow user to view
    - The system cached information about transaction in the last 1 year.
    - The system indexing for searching

## Components diagram
![Diagram](images/component-diagrams.svg)

- User: user can be either Driver/Customer
- Load Balancer: system load balancer
- Authentication & Authorization Service: Service do authentication and authorization
- Customer Service: Service save and get information about Customer/Admin
- Driver Service: Service save and get information about Driver
- Ride Transaction Service: Service save and get information about every ride
- Notification Service: Service call 3rd provider to send notification
- Notification Provider: 3rd party that do the sending notifications
- Message Queue: Message queue that do async requests
- Database: RDMS database
- Booking Service: Service that coordinating a ride (check user location, find matched driver location)

## Main flows
### Booking a ride flow
[![Sequence Diagram](images/booking-a-ride-flow.svg)](https://sequencediagram.org/index.html?presentationMode=readOnly#initialData=C4S2BsFMAICEHt4GsQDsDm0CG0BOIATGAM3HgHcAoSrAY2Hl2gFUBnSXSgBy11FpA9UwaACIAggFdgAC0jCQtLKHipsqAtCmzGIAF7KQq6AGUOAN0WRR2VlukyzuS7Ujde-QVmFxEKDE4ubjx8il4+AMKSrAwAthyBVu6hAkIiACL45gkWSSGeadAAKrjerHQqqImuyQXeIqIAspCs5egwAIqSkN02WHaNHZQEylgARv0w6bDUbBzQALQAfPay1ZAAXNAAVNsAEpBYRLisu1sAUuQiDEjyADqoADzkjATkpVwbAKwADEu7CAIAE8ztBohwAPqEAA00FoklwuHkwAhZCUlVhRBiaEMqlR8HRRlQsOApVQrC4jGAuNQEOAQK4kEeAHoXrg3h8ltQsOARAA1HmEYrIeSUbSOXKuRYrBDINDodYbZ6vd5YT6-Jbg3BQgiw+GI5H4wmqTEtUCoGlGmkkskUqmW+mMllsjlqrmy-wKyUwZbQKIxeDxXCK6AAcUgIi10DGQLB7G1hEo-riOWcVml0GmW2V7NV6r+4cj8eg5DAMmgQoAvHHIULvJoYspotBqwAxABKAFFO+pNGikNBZMoa7hnSrOUnoing96M8nA6mglszEX5o3gM3q7AAPLbgDSAEkAHKhycBoPrDNZvCQKCTEdn6eX30e+UhuZMUuyCuaatanW9tA66bnAu6HielBkPAXDQAA6jIIBQNABBZPMqDwCIxDwJIGiAaA8TQI8xQHo0nbbswRSUK+ASzr6gzZi6ebfH8Zi4QA2v+MJwgiSLCFaGLIWaOKVPxRI2mUlJ8A6DKQAAuoO8DQFw+C6PS0AAI7dN0JZlkpKn4Gp1b6rxKLKUYBlAncADU1ljrmE6ZCA2QzmmUqPAs0D0WGEbQBx8Y6nqPGGmi1qCdiFoiSFAmkhJ9oiY6cmUI5znPis145q6+ZLIWyGoZ+ukxeSknUvFMktoOtrFdJjKAcBdhtl2Pb1shICsAAFJS7UoU5HAAJSBQafFRUSvXlY0x6Af2g4yMO3XOXZmVcjyGR5dAWE4QQSV5almbbd6WzMFwIzADAc1riVIE7vunbpFtPUuUEGZee2EYImo60aJAmhnZwt7sLt93QOhmHYRod0pbRKzPa9uBqKgkjgOAlDyJt1Feq5MDuZ5HRbDlH3fXllBjEiWADsl8z41RfhvpDvhyjRGPLj5P1raDRCaG1xOHAOUFcL1yNgyjVP0+jj2+iUZQVESIYRCTJ1A5A5AVZL9BEr5nG6rl90Bdxg0osNJphealoG6gsmUBL5JS6oO3XrLhzy6givK1bqvGH5taaz9OvGcFBKhVixuRf7lTm2jO0flsL0brDWvOTqyPgP9B6oOYgqaEUIqoGKDgR-GWwACw-AAzFsraMGMhBEGoAsEEAA)
