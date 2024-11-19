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
- Booking a Car:
    - Drivers update their position every 30 seconds
    - When a user books a drive, the system looks for nearest driver and assign that driver for user.
        -  If the distant of nearest driver and user > MAX_ALLOW_DISTANCE, cancel that transaction. Keep looking again
        -  The distance between user and driver = distance(pos(user) - post(driver)) + driver_average_rate. It means that even a driver is nearer to a customer, the system can assign to a farer driver if that driver is rated low
    - The system will support long polling for this api (up to 30 seconds)
    - The system searches for matched driver up to 2 seconds. If timeout, the syste, searches again for one with higher priority
    - Users booking request and matched drivers will be exchanged over Message Queue (Even service are monolithics). If the system is monolithics, the system could use an in-memory Message Queue (implemented by Queue) else the system use a distributed Message Queue (RabbitMQ)
    - Notifications will be sent async (Message Queue). The system will use 3rd providers to send nofitications.
- Booking History and Status Tracking:
    - The system saved car history and allow user to view
    - The system cached information about transaction in the last 1 years.
    - The system indexing for searching
