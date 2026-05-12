# CarRenter

CarRenter is a microservices-based ASP.NET vehicle rental reservation system with JWT authentication, role-based access (User/Admin), and an API Gateway.

## Core Features

- User registration and login (JWT)
- Role-based access (User/Admin)
- Browse available vehicles
- Create reservations for selected vehicles and date ranges
- View and cancel reservations
- Admin panel for managing users, vehicles, and reservations

## Architecture

The project is split into independent services:

- **CarRenter.Auth (UserService)** – authentication, registration, JWT, user accounts, roles
- **VehicleService** – vehicle catalog CRUD
- **ReservationService** – reservation CRUD, vehicle-based filtering, reservation dates
- **CarRenter.Gateway (API Gateway)** – Ocelot-based routing between frontend and services
- **CarRenter (Frontend)** – ASP.NET Core MVC app that aggregates data and handles user session/UI

Communication between modules is done through REST APIs.

## Technologies and Packages

- ASP.NET Core (Backend + Frontend MVC)
- Entity Framework Core
- MySQL via Pomelo provider
- JWT authentication/authorization
- Swagger / OpenAPI
- Ocelot API Gateway
- Stripe.net

Main package set used in the solution:

- `Microsoft.AspNetCore.Authentication.JwtBearer`
- `Microsoft.EntityFrameworkCore`
- `Microsoft.EntityFrameworkCore.Design`
- `Pomelo.EntityFrameworkCore.MySql`
- `Stripe.net`
- `Swashbuckle.AspNetCore`
- `Microsoft.AspNetCore.OpenApi`
- `Ocelot`

## Services and Local Ports

- **API Gateway**: `http://localhost:6000`
- **Auth/User service**: `http://localhost:6001`
- **Reservation service**: `http://localhost:5007`
- **Vehicle service**: `http://localhost:5012`

## Database Model (ERD)

### Users
- `Id` (PK)
- `Username`
- `Email`
- `PasswordHash`
- `Role`

### Vehicles
- `Id` (PK)
- `Brand` (`Marka`)
- `Model`
- `Year` (`Rok`)
- `PricePerDay` (`KwotaZaDzien`)
- `Description` (`Opis`)
- `ImageUrl` (`UrlObrazka`)

### Reservations
- `ReservationId` (PK)
- `VehicleId` (FK)
- `UserId` (FK)
- `StartDate`
- `EndDate`
- `CreatedAt`
- `Notes`

## API Endpoints

### User/Auth Service (`CarRenter.Auth`)

#### AuthController (`/api/auth`)

| Method | Path | Body / Params | Auth | Response |
|---|---|---|---|---|
| POST | `/api/auth/register` | `Username`, `Email`, `Password` | No | `200 OK` / `400 BadRequest` |
| POST | `/api/auth/login` | `Username`, `Password` | No | JWT token + role |

#### UserController (`/api/user`)

| Method | Path | Body / Params | Auth (project assumption) | Response |
|---|---|---|---|---|
| GET | `/api/user` | - | JWT (Admin) | List of users |
| GET | `/api/user/{id}` | `id` | JWT | User details |
| PUT | `/api/user/{id}` | `Username`, `Email`, `Role` | JWT (Admin) | `200 OK` |
| PUT | `/api/user/editwithpassword/{id}` | `Username`, `Email`, `NewPassword` | JWT | `200 OK` |
| DELETE | `/api/user/{id}` | `id` | JWT (Admin) | `200 OK` / `404` |

### Vehicle Service (`VehicleService`)

| Method | Path | Body / Params | Auth (project assumption) | Response |
|---|---|---|---|---|
| GET | `/api/vehicles` | - | No | Vehicle list |
| GET | `/api/vehicles/{id}` | `id` | No | Vehicle details |
| POST | `/api/vehicles` | Vehicle payload | JWT (Admin) | `201 Created` |
| PUT | `/api/vehicles/{id}` | `id`, vehicle payload | JWT (Admin) | `204 NoContent` |
| DELETE | `/api/vehicles/{id}` | `id` | JWT (Admin) | `204 NoContent` |

### Reservation Service (`ReservationService`)

| Method | Path | Body / Params | Auth (project assumption) | Response |
|---|---|---|---|---|
| GET | `/api/reservation` | - | JWT | Reservation list |
| GET | `/api/reservation/{id}` | `id` | JWT | Reservation details |
| GET | `/api/reservation/by-vehicle/{vehicleId}` | `vehicleId` | JWT | Reservations for vehicle |
| POST | `/api/reservation` | `VehicleId`, `UserId`, `StartDate`, `EndDate`, `Notes` | JWT | `201 Created` |
| PUT | `/api/reservation/{id}` | `id`, reservation payload | JWT | `204 NoContent` |
| DELETE | `/api/reservation/{id}` | `id` | JWT | `204 NoContent` |

### Gateway Upstream Routes (`CarRenter.Gateway`)

| Upstream | Downstream |
|---|---|
| `/auth/{everything}` | `/api/auth/{everything}` |
| `/user/{everything}` | `/api/user/{everything}` |
| `/vehicles/{everything}` | `/api/vehicles/{everything}` |
| `/reservation/{everything}` *(configured as `/Reservation/{everything}` in `ocelot.json`)* | `/api/reservation/{everything}` *(configured as `/api/Reservation/{everything}` in `ocelot.json`)* |

## Validation and Error Handling

- Reservation UI checks for date conflicts before submit.
- Reservation conflict attempts should return readable error messages.
- Reservation deletion is intended for owner/admin.
- Typical errors are handled (unauthorized, missing resource, invalid input).
- IDs are generated server-side during entity creation.

## Admin Panel

Admin capabilities include:

- Managing users, vehicles, and reservations
- Filtering reservations by vehicle
- Deleting reservations and vehicles

## Summary

CarRenter implements a modern vehicle reservation platform with microservices architecture.  
The service split, JWT security model, API Gateway, and separate data ownership per service support scalability, maintainability, and secure data flow.
