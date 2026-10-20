# Car Rental Service

Online car rental management system for customers and staff. The service replaces manual fleet tracking, paper rental contracts, phone calls, and cash-only payments with a browsable API, inventory management, Stripe checkout, and Telegram notifications.

## Project goals

- Let customers register, authenticate, browse available cars, create and manage rentals, and pay online.
- Let staff manage the car fleet and oversee rentals and payments.
- Notify staff about new rentals, overdue returns, and successful payments.
- Provide the complete user experience through the browsable API; no frontend is included in this project.

## Functional requirements

### Cars

Staff can create, view, update, and delete cars. Customers can browse cars and filter the list by brand, fuel type, and year range. Rental creation and return update fleet availability.

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/cars/` | Add a car (staff) |
| `GET` | `/cars/` | List cars; supports `brand`, `fuel_type`, `min_year`, and `max_year` filters |
| `GET` | `/cars/<id>/` | Retrieve car details |
| `PUT`, `PATCH` | `/cars/<id>/` | Update car information or availability (staff) |
| `DELETE` | `/cars/<id>/` | Delete a car (staff) |

### Users and authentication

Customers can register, obtain and refresh JWT tokens, view their profile, and update their profile information.

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/users/` | Register a customer |
| `POST` | `/users/token/` | Obtain JWT access and refresh tokens |
| `POST` | `/users/token/refresh/` | Refresh an access token |
| `GET` | `/users/me/` | Retrieve the authenticated user's profile |
| `PUT`, `PATCH` | `/users/me/` | Update the authenticated user's profile |

### Rentals

Customers can create rentals, inspect rental details and history, return cars, or cancel eligible bookings. Creating a rental reduces available inventory; returning a car restores it. Cancellation is available only before the rental start date. A late cancellation may incur a fee.

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/rentals/` | Create a rental |
| `GET` | `/rentals/` | List rentals; supports `user_id` and `status` filters |
| `GET` | `/rentals/<id>/` | Retrieve rental details |
| `POST` | `/rentals/<id>/return/` | Return a rented car and record the actual return date |
| `POST` | `/rentals/<id>/cancel/` | Cancel a booking when eligible |

### Payments

Stripe handles checkout for rental charges, cancellation fees, and overdue-return fines. Payment records track the checkout session and its payment status.

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/success/` | Payment success return page |
| `GET` | `/cancel/` | Payment cancellation return page |

### Telegram notifications

The system sends staff notifications about new rentals, overdue rentals, and successful payments. Notifications are intended to run in the background using a task queue such as Django-Q or Celery and a Telegram bot.

## Data model

### Car

- `brand`, `model`: text
- `year`: integer
- `fuel_type`: `GAS`, `DIESEL`, `HYBRID`, or `ELECTRIC`
- `daily_rate`: decimal amount in USD
- `inventory`: positive integer

### User (customer)

- `email`: email address used for registration/authentication
- `first_name`, `last_name`: text
- `password`: stored as a securely hashed credential
- `is_staff`: staff access flag

### Rental

- `car_id`, `user_id`: associated car and customer
- `start_date`, `end_date`: planned rental period
- `actual_return_date`: actual return date, if returned
- `status`: `BOOKED`, `COMPLETED`, `CANCELLED`, or `OVERDUE`
- `price_per_day`: decimal amount in USD captured for the rental

### Payment

- `status`: `PENDING`, `PAID`, or `EXPIRED`
- `type`: `RENTAL`, `CANCELLATION_FEE`, or `OVERDUE_FEE`
- `rental_id`: associated rental
- `session_url`: Stripe Checkout URL
- `session_id`: Stripe Checkout session identifier
- `money_to_pay`: decimal amount in USD

## Non-functional targets

- Up to 5 concurrent users
- Fleet of up to 300 cars
- Approximately 80,000 rentals per year
- Approximately 40 MB of data per year

## API usage

The service is designed to be used through an HTTP API and its browsable interface. Authenticate with a JWT access token for protected operations. Staff-only operations require a staff account. Exact permissions and response schemas depend on the API implementation.