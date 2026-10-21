# Todo API

A RESTful **Todo API** built with **ASP.NET Core 9**, **Entity Framework Core**, and **MySQL**. Users register and log in to receive an access token, then manage their own private list of todos. Each user can only see and modify their own items.

## Features

- **User authentication**: register, log in, and log out
- **Secure passwords** hashed with BCrypt
- **Opaque access tokens**: 32-byte random tokens, stored only as SHA-256 hashes, valid for 7 days, and revocable on logout
- **Custom authentication handler** (`TokenAuth` scheme) that validates `Authorization: Bearer <token>` headers
- **Per-user todos**: full CRUD with partial updates (`PATCH`)
- **DTOs + validation** with Data Annotations
- **AutoMapper** for entity ↔ DTO mapping
- **Swagger UI** with bearer-token support in development
- Clean layering: Controllers → Services → EF Core `DbContext`

## Tech Stack

| Concern        | Technology |
| -------------- | ---------- |
| Framework      | ASP.NET Core 9 (Web API, controllers) |
| ORM            | Entity Framework Core 9 + Pomelo MySQL provider |
| Database       | MySQL |
| Auth           | Custom token scheme, BCrypt.Net-Next, SHA-256 |
| Mapping        | AutoMapper |
| API docs       | Swashbuckle (Swagger) |

## Project Structure

```
todo-api/
├── Controllers/
│   ├── AuthController.cs           # /api/Auth: register, login, logout
│   └── TodoController.cs           # /api/Todo: CRUD (requires auth)
├── Services/                        # Business logic (IAuthService, ITodoService)
├── Data/AppDbContext.cs             # EF Core context and relationships
├── Models/                          # User, Todo, AccessToken entities
├── DTOs/                            # Request/response contracts
├── Mapping/                         # AutoMapper profiles
├── Middlewares/
│   └── TokenAuthenticationHandler.cs
├── Migrations/                      # EF Core migrations
├── Program.cs                       # DI, auth, Swagger, pipeline
├── appsettings.Example.json         # Configuration template
└── TodoApi.csproj
```

## Getting Started

### Prerequisites

- [.NET 9 SDK](https://dotnet.microsoft.com/download)
- MySQL 8+
- EF Core CLI tools:

  ```bash
  dotnet tool install --global dotnet-ef
  ```

### Installation

```bash
git clone https://github.com/SifeYassine/todo-api.git
cd todo-api
dotnet restore
```

### Configuration

Copy the example settings and fill in your MySQL connection string:

```bash
cp appsettings.Example.json appsettings.json
```

```json
"ConnectionStrings": {
  "DefaultConnection": "Server=localhost;Port=3306;Database=todo_api;User=root;Password=yourpassword;"
}
```

> `appsettings.json` and `appsettings.Development.json` are git-ignored, so your credentials stay local.

### Database

Apply the migrations to create the schema:

```bash
dotnet ef database update
```

### Run

```bash
dotnet run
```

The API listens on `http://localhost:5106` (or `https://localhost:7267` with the `https` profile).
In development, Swagger UI is available at **`/swagger`**.

## API Reference

### Auth

| Method | Endpoint             | Auth | Description |
| ------ | -------------------- | :--: | ----------- |
| POST   | `/api/Auth/register` |  ✗   | Create an account |
| POST   | `/api/Auth/login`    |  ✗   | Log in and receive an access token |
| POST   | `/api/Auth/logout`   |  ✓   | Revoke the current token |

**Register**

```json
{ "username": "jane", "email": "jane@example.com", "password": "secret123" }
```

**Login**

```json
{ "email": "jane@example.com", "password": "secret123" }
```

Response (login):

```json
{
  "id": 1,
  "username": "jane",
  "email": "jane@example.com",
  "createdAt": "2026-01-12T17:43:01Z",
  "token": "<access-token>"
}
```

### Todos

All endpoints require `Authorization: Bearer <access-token>`.

| Method | Endpoint          | Description |
| ------ | ----------------- | ----------- |
| GET    | `/api/Todo`       | List your todos |
| GET    | `/api/Todo/{id}`  | Get one todo |
| POST   | `/api/Todo`       | Create a todo |
| PATCH  | `/api/Todo/{id}`  | Partially update a todo |
| DELETE | `/api/Todo/{id}`  | Delete a todo |

**Create**

```json
{ "title": "Buy milk", "description": "2 liters" }
```

**Update** (all fields optional)

```json
{ "isComplete": true }
```

### Validation rules

| Field                 | Rule |
| --------------------- | ---- |
| `username`            | 3–20 characters, unique |
| `email`               | Valid email, max 50 characters, unique |
| `password`            | 6–20 characters |
| `title`               | 1–50 characters |
| `description`         | Max 100 characters |

## Example with cURL

```bash
# Log in
curl -X POST http://localhost:5106/api/Auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"jane@example.com","password":"secret123"}'

# Create a todo
curl -X POST http://localhost:5106/api/Todo \
  -H "Authorization: Bearer <access-token>" \
  -H "Content-Type: application/json" \
  -d '{"title":"Buy milk"}'
```

## Author

**Sifeddine Yassine** · [@SifeYassine](https://github.com/SifeYassine)
