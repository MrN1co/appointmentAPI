# Appointment API

A simple API for CRUD operations on appointments made based on provided availability.

## Overview

appointmentAPI is an ASP.NET Core-based REST API built with C# that manages appointment reservations. It provides endpoints for managing clients, visits (appointment types), and reservations, with role-based access control and authentication.

## Technology Stack

- Framework: ASP.NET Core (net8.0)
- Language: C#
- Database: SQLite
- Authentication: ASP.NET Core Identity
- ORM: Entity Framework Core

## Getting Started

### Prerequisites

- .NET 8.0 SDK or later
- SQLite (included with the project)

### Installation

1. Clone the repository:
```bash
git clone https://github.com/MrN1co/appointmentAPI.git
cd appointmentAPI
```

2. Navigate to the project directory:
```bash
cd AppointmentAPI
```

3. Install dependencies and build:
```bash
dotnet restore
dotnet build
```

4. Run the application:
```bash
dotnet run
```

The API will be available at `https://localhost:7000` by default.

## Authentication

This API uses ASP.NET Core Identity for authentication and role-based authorization.

### Available Roles

- **Admin**: Full access to all endpoints including user deletion
- **User**: Standard access to create and manage their own resources

### Authentication Endpoints

The API provides standard Identity endpoints for user management:

- `POST /login` - User login
- `POST /register` - User registration
- `POST /refresh` - Refresh authentication token

Most endpoints require authorization. Use the Bearer token obtained from login in the Authorization header:

```
Authorization: Bearer {your_token}
```

## API Endpoints

### Visit Management

Visits represent appointment types available in the system.

#### GET /visit

Retrieve all available visits.

- Authentication: Not required
- Response: List of Visit objects

**Example Response:**
```json
[
  {
    "visitId": 1,
    "type": "Consultation",
    "cost": 50.0,
    "timeLength": 30.0
  },
  {
    "visitId": 2,
    "type": "Follow-up",
    "cost": 35.0,
    "timeLength": 20.0
  }
]
```

#### GET /visit/{visitId}

Retrieve a specific visit by ID.

- Authentication: Required
- Path Parameter: `visitId` (integer)
- Response: Visit object
- Status Codes:
  - 200 OK: Visit found
  - 404 Not Found: Visit does not exist

**Example Request:**
```
GET /visit/1
Authorization: Bearer {token}
```

#### POST /visit

Create a new visit type.

- Authentication: Required
- Request Body:
  ```json
  {
    "type": "Consultation",
    "cost": 50.0,
    "timeLength": 30.0
  }
  ```
- Response: 201 Created with location header
- Status Codes:
  - 201 Created: Visit created successfully
  - 400 Bad Request: Invalid input

**Example Request:**
```
POST /visit
Authorization: Bearer {token}
Content-Type: application/json

{
  "type": "Consultation",
  "cost": 50.0,
  "timeLength": 30.0
}
```

#### PUT /visit

Update an existing visit.

- Authentication: Required
- Request Body:
  ```json
  {
    "visitId": 1,
    "type": "Updated Consultation",
    "cost": 60.0,
    "timeLength": 45.0
  }
  ```
- Response: 204 No Content
- Status Codes:
  - 204 No Content: Update successful
  - 400 Bad Request: Invalid visit or input

**Example Request:**
```
PUT /visit
Authorization: Bearer {token}
Content-Type: application/json

{
  "visitId": 1,
  "type": "Extended Consultation",
  "cost": 65.0,
  "timeLength": 45.0
}
```

#### DELETE /visit

Delete a visit by ID.

- Authentication: Required
- Query Parameter: `id` (integer)
- Response: 204 No Content
- Status Codes:
  - 204 No Content: Visit deleted successfully
  - 404 Not Found: Visit does not exist

**Example Request:**
```
DELETE /visit?id=1
Authorization: Bearer {token}
```

### Client Management

Clients are individuals who can make appointment reservations.

#### GET /client

Retrieve all clients.

- Authentication: Required
- Response: List of Client objects
- Status Code: 200 OK

**Example Response:**
```json
[
  {
    "clientId": "JohDoe1a2b",
    "name": "John",
    "surname": "Doe",
    "email": "john@example.com",
    "phoneNumber": "+1234567890"
  },
  {
    "clientId": "JanSmi3c4d",
    "name": "Jane",
    "surname": "Smith",
    "email": "jane@example.com",
    "phoneNumber": "+0987654321"
  }
]
```

#### GET /client/{clientId}

Retrieve a specific client by ID.

- Authentication: Required
- Path Parameter: `clientId` (string)
- Response: ClientDto object
- Status Codes:
  - 200 OK: Client found
  - 404 Not Found: Client does not exist

**Example Request:**
```
GET /client/JohDoe1a2b
Authorization: Bearer {token}
```

#### POST /client

Create a new client.

- Authentication: Required
- Request Body:
  ```json
  {
    "name": "John",
    "surname": "Doe",
    "email": "john@example.com",
    "phoneNumber": "+1234567890"
  }
  ```
- Response: 201 Created
- Status Codes:
  - 201 Created: Client created successfully
  - 400 Bad Request: Invalid input

**Notes:**
- Client ID is automatically generated using the first 3 characters of name and surname plus a random hex string
- Email is required
- Phone number is optional

**Example Request:**
```
POST /client
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "John",
  "surname": "Doe",
  "email": "john@example.com",
  "phoneNumber": "+1234567890"
}
```

**Example Response:**
```json
{
  "clientId": "JohDoe1a2b",
  "name": "John",
  "surname": "Doe",
  "email": "john@example.com",
  "phoneNumber": "+1234567890"
}
```

#### PUT /client

Update an existing client.

- Authentication: Required
- Request Body:
  ```json
  {
    "clientId": "JohDoe1a2b",
    "name": "John",
    "surname": "Doe",
    "email": "john.doe@example.com",
    "phoneNumber": "+1111111111"
  }
  ```
- Response: 204 No Content
- Status Codes:
  - 204 No Content: Update successful
  - 400 Bad Request: Invalid client or input

**Example Request:**
```
PUT /client
Authorization: Bearer {token}
Content-Type: application/json

{
  "clientId": "JohDoe1a2b",
  "name": "John",
  "surname": "Doe",
  "email": "john.updated@example.com",
  "phoneNumber": "+1111111111"
}
```

#### DELETE /client/{clientId}

Delete a client by ID.

- Authentication: Required
- Authorization: Admin role required
- Path Parameter: `clientId` (string)
- Response: 204 No Content
- Status Codes:
  - 204 No Content: Client deleted successfully
  - 404 Not Found: Client does not exist
  - 403 Forbidden: Insufficient permissions

**Example Request:**
```
DELETE /client/JohDoe1a2b
Authorization: Bearer {admin_token}
```

### Reservation Management

Reservations represent scheduled appointments between clients and visit types.

#### GET /reservation

Retrieve all reservations.

- Authentication: Required
- Response: List of Reservation objects
- Status Code: 200 OK

**Example Response:**
```json
[
  {
    "reservationId": "res1a2b3c4d",
    "clientId": "JohDoe1a2b",
    "visitId": 1,
    "date": "2026-02-15T10:00:00"
  },
  {
    "reservationId": "res5e6f7g8h",
    "clientId": "JanSmi3c4d",
    "visitId": 2,
    "date": "2026-02-16T14:30:00"
  }
]
```

#### GET /reservation/{reservationId}

Retrieve a specific reservation by ID.

- Authentication: Required
- Path Parameter: `reservationId` (string)
- Response: Reservation object
- Status Codes:
  - 200 OK: Reservation found
  - 404 Not Found: Reservation does not exist

**Example Request:**
```
GET /reservation/res1a2b3c4d
Authorization: Bearer {token}
```

#### POST /reservation

Create a new reservation.

- Authentication: Required
- Request Body:
  ```json
  {
    "clientId": "JohDoe1a2b",
    "visitId": 1,
    "date": "2026-02-15T10:00:00"
  }
  ```
- Response: 201 Created
- Status Codes:
  - 201 Created: Reservation created successfully
  - 400 Bad Request: Invalid input (missing client or visit)

**Example Request:**
```
POST /reservation
Authorization: Bearer {token}
Content-Type: application/json

{
  "clientId": "JohDoe1a2b",
  "visitId": 1,
  "date": "2026-02-15T10:00:00"
}
```

**Example Response:**
```json
{
  "reservationId": "res1a2b3c4d",
  "clientId": "JohDoe1a2b",
  "visitId": 1,
  "date": "2026-02-15T10:00:00"
}
```

#### PUT /reservation

Update an existing reservation.

- Authentication: Required
- Request Body:
  ```json
  {
    "reservationId": "res1a2b3c4d",
    "clientId": "JohDoe1a2b",
    "visitId": 2,
    "date": "2026-02-15T11:00:00"
  }
  ```
- Response: 204 No Content
- Status Codes:
  - 204 No Content: Update successful
  - 400 Bad Request: Invalid reservation or input

**Example Request:**
```
PUT /reservation
Authorization: Bearer {token}
Content-Type: application/json

{
  "reservationId": "res1a2b3c4d",
  "clientId": "JohDoe1a2b",
  "visitId": 1,
  "date": "2026-02-15T11:30:00"
}
```

#### DELETE /reservation/{reservationId}

Delete a reservation by ID.

- Authentication: Required
- Path Parameter: `reservationId` (string)
- Response: 204 No Content
- Status Codes:
  - 204 No Content: Reservation deleted successfully
  - 404 Not Found: Reservation does not exist

**Example Request:**
```
DELETE /reservation/res1a2b3c4d
Authorization: Bearer {token}
```

## Data Models

### Visit

Represents a type of appointment available in the system.

```json
{
  "visitId": 1,
  "type": "Consultation",
  "cost": 50.0,
  "timeLength": 30.0
}
```

**Fields:**
- `visitId` (integer): Unique identifier (auto-generated)
- `type` (string): Visit type name (max 30 characters)
- `cost` (float): Cost of the visit
- `timeLength` (float): Duration in minutes

### Client

Represents an individual who can make reservations.

```json
{
  "clientId": "JohDoe1a2b",
  "name": "John",
  "surname": "Doe",
  "email": "john@example.com",
  "phoneNumber": "+1234567890"
}
```

**Fields:**
- `clientId` (string): Unique identifier (auto-generated from name, surname, and random hex)
- `name` (string): First name (required, max 30 characters)
- `surname` (string): Last name (required, max 30 characters)
- `email` (string): Email address (required, must be valid email, max 100 characters)
- `phoneNumber` (string): Phone number (optional, max 15 characters)

### Reservation

Represents a scheduled appointment linking a client to a visit.

```json
{
  "reservationId": "res1a2b3c4d",
  "clientId": "JohDoe1a2b",
  "visitId": 1,
  "date": "2026-02-15T10:00:00"
}
```

**Fields:**
- `reservationId` (string): Unique identifier (auto-generated)
- `clientId` (string): Reference to the client (required)
- `visitId` (integer): Reference to the visit type (required)
- `date` (datetime): Date and time of the appointment (required)

## HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | OK - Request successful |
| 201 | Created - Resource created successfully |
| 204 | No Content - Request successful with no response body |
| 400 | Bad Request - Invalid input or missing required fields |
| 401 | Unauthorized - Authentication required or failed |
| 403 | Forbidden - Insufficient permissions for the resource |
| 404 | Not Found - Resource does not exist |

## Error Handling

The API returns appropriate HTTP status codes and error messages. Always check the response status code to determine if a request was successful.

**Example Error Response:**
```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Email": ["The Email field is required."]
  }
}
```

## Database

The application uses two SQLite databases:

- `ReservationData.db` - Contains Clients, Visits, and Reservations
- `AuthenticationData.db` - Contains user authentication and role information

Databases are automatically created on first run.

## Validation Rules

All input data is validated according to the following rules:

### Client Validation

- **Name**: Required, max 30 characters
- **Surname**: Required, max 30 characters
- **Email**: Required, must be a valid email format, max 100 characters
- **Phone Number**: Optional, max 15 characters, must be valid phone format

### Visit Validation

- **Type**: Required, max 30 characters
- **Cost**: Required, must be a positive float
- **Time Length**: Required, must be a positive float (in minutes)

### Reservation Validation

- **Client ID**: Required, must reference an existing client
- **Visit ID**: Required, must reference an existing visit
- **Date**: Required, must be a valid datetime format

## Usage Examples

### Complete Workflow Example

Here's a step-by-step example of creating a reservation:

#### Step 1: Register and Login

```bash
curl -X POST https://localhost:7000/register \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"SecurePass123!"}'
```

```bash
curl -X POST https://localhost:7000/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"SecurePass123!"}'
```

Response includes access token.

#### Step 2: Create a Visit Type (Admin)

```bash
curl -X POST https://localhost:7000/visit \
  -H "Authorization: Bearer {admin_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "Dental Cleaning",
    "cost": 75.0,
    "timeLength": 45.0
  }'
```

#### Step 3: Create a Client

```bash
curl -X POST https://localhost:7000/client \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John",
    "surname": "Doe",
    "email": "john@example.com",
    "phoneNumber": "+1234567890"
  }'
```

Response includes the generated `clientId`.

#### Step 4: Create a Reservation

```bash
curl -X POST https://localhost:7000/reservation \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "clientId": "JohDoe1a2b",
    "visitId": 1,
    "date": "2026-02-20T10:00:00"
  }'
```

#### Step 5: View All Reservations

```bash
curl -X GET https://localhost:7000/reservation \
  -H "Authorization: Bearer {token}"
```

## Development

### Swagger UI

When running in development mode, Swagger UI is available at:
```
https://localhost:7000/swagger
```

This provides an interactive interface for testing all API endpoints.

### Testing Endpoints

You can use tools like Postman, Insomnia, or curl to test the API:

```bash
# Test if server is running
curl https://localhost:7000

# Get all visits (no auth required)
curl https://localhost:7000/visit
```

### Logging

The application includes logging configuration. Check the console output during development for debugging information.

### Project Structure

```
AppointmentAPI/
├── Controllers/          # API endpoint controllers
│   ├── ClientController.cs
│   ├── ReservationController.cs
│   ├── VisitController.cs
│   └── RoleController.cs
├── Models/              # Data models
│   ├── Client.cs
│   ├── Visit.cs
│   └── Reservation.cs
├── Services/            # Business logic services
│   ├── ClientService.cs
│   ├── ReservationService.cs
│   └── VisitService.cs
├── DTOs/               # Data transfer objects
│   ├── ClientDto.cs
│   ├── VisitDto.cs
│   └── ReservationDto.cs
├── Data/               # Database context and initialization
│   ├── ReservationContext.cs
│   ├── AuthenticationContext.cs
│   ├── DbInitializer.cs
│   └── Extensions.cs
├── Migrations/         # Entity Framework migrations
├── Program.cs          # Application startup configuration
└── appointmentAPI.csproj
```

### Database Initialization

The application automatically creates and initializes the SQLite databases on first run using the `DbInitializer` class. This includes:

- Creating database tables
- Setting up relationships
- Populating initial data (if configured)

## Performance Notes

- All database queries use Entity Framework Core with efficient querying
- Indexes are created on foreign keys and primary keys
- The application uses SQLite, suitable for small to medium deployments

## License

This project is open source and available under the MIT License.

## Changelog

- Initial release
- Complete CRUD operations for Clients, Visits, and Reservations
- ASP.NET Core Identity authentication
- Role-based authorization
- SQLite database integration
- Swagger/OpenAPI documentation

### Issues to fix
- Not all endpoints properly filter users
