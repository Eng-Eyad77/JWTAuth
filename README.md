# JwtAuth

JwtAuth is an educational ASP.NET Core Web API that demonstrates user registration and JWT-based authentication with Entity Framework Core.

## Technologies

- .NET 10 (`net10.0`)
- ASP.NET Core Web API
- Entity Framework Core 10
- SQL Server via `Microsoft.EntityFrameworkCore.SqlServer`
- JWT bearer authentication
- Scalar and ASP.NET Core OpenAPI for development-time API documentation

## Implemented features

- User registration at `POST /api/Auth/register`
- Password hashing and verification with ASP.NET Core `PasswordHasher<User>`
- User login at `POST /api/Auth/login`
- JWT access-token generation with issuer, audience, expiration, and HMAC-SHA512 signing
- Refresh-token generation and replacement at `POST /api/Auth/refresh-token`
- Authenticated-user endpoint at `GET /api/Auth`
- Role-protected admin endpoint at `GET /api/Auth/admin-only`
- SQL Server persistence for users, roles, refresh tokens, and refresh-token expiration times

Newly registered users receive the default empty role from the current entity implementation. The admin endpoint therefore requires a user whose stored role is `Admin`; registration does not accept a role.

## Authentication flow

1. Register with a username and password.
2. Log in with the same credentials. The API verifies the stored password hash and returns an access token and refresh token.
3. Send the access token as a bearer token in the `Authorization` header when calling protected endpoints.
4. When an access token needs to be renewed, send the user's ID and refresh token to `POST /api/Auth/refresh-token`.
5. The API validates the refresh token and its expiration, then issues a new access token and replaces the stored refresh token. Refresh tokens expire after seven days, while access tokens expire after one day.

## Project structure

```text
JwtAuth/
├── Controllers/
│   └── AuthController.cs
├── Data/
│   └── UserDbContext.cs
├── Entities/
│   └── User.cs
├── Migrations/
│   ├── 20260930105358_InitialCreate.cs
│   ├── 20261001083119_userRole.cs
│   ├── 20261001122918_addRefeshToken.cs
│   └── UserDbContextModelSnapshot.cs
├── Models/
│   ├── RefreshTokenRequestDto.cs
│   ├── TokenResponseDto.cs
│   └── UserDto.cs
├── Properties/
│   └── launchSettings.json
├── Services/
│   ├── AuthService.cs
│   └── IAuthService.cs
├── appsettings.json
├── appsettings.Development.json
├── JwtAuth.csproj
└── JwtAuth.http
```

## Prerequisites

- .NET 10 SDK
- SQL Server or SQL Server Express
- Entity Framework Core CLI tools, if you need to create or apply migrations

## Setup

From the directory containing `JwtAuth.csproj`, restore dependencies:

```powershell
dotnet restore
```

Configure a local SQL Server connection string and JWT settings without committing secrets. The application reads these keys:

- `ConnectionStrings:UserDatabase`
- `AppSettings:Token`
- `AppSettings:Issuer`
- `AppSettings:Audience`

For example, environment variables use double underscores for nested configuration keys:

```powershell
$env:ConnectionStrings__UserDatabase = "Server=localhost\SQLEXPRESS;Database=JwtAuth;Trusted_Connection=True;TrustServerCertificate=True;"
$env:AppSettings__Token = "<long-random-signing-key>"
$env:AppSettings__Issuer = "<your-issuer>"
$env:AppSettings__Audience = "<your-audience>"
```

Use a long, randomly generated signing key and keep local credentials outside source control. Do not publish real connection strings or signing keys.

## Database and migrations

The project uses `UserDbContext` with SQL Server and includes EF Core migrations in `Migrations/`. Apply the existing migrations with:

```powershell
dotnet ef database update
```

To create a new migration after changing the data model:

```powershell
dotnet ef migrations add <MigrationName>
dotnet ef database update
```

The connection string must be available through the `ConnectionStrings:UserDatabase` configuration key before applying migrations.

## Run the API

```powershell
dotnet run
```

The configured launch profiles use:

- HTTP: `http://localhost:5259`
- HTTPS: `https://localhost:7103`

In Development, OpenAPI and Scalar are mapped by the application. The included `JwtAuth.http` file also contains a basic weather forecast request at `/weatherforecast`.

## Test the authentication endpoints

Register a user:

```http
POST https://localhost:7103/api/Auth/register
Content-Type: application/json

{
  "username": "demo-user",
  "password": "<local-password>"
}
```

Log in:

```http
POST https://localhost:7103/api/Auth/login
Content-Type: application/json

{
  "username": "demo-user",
  "password": "<local-password>"
}
```

Use the returned access token:

```http
GET https://localhost:7103/api/Auth
Authorization: Bearer <access-token>
```

Refresh the token pair with the user ID returned by registration and the current refresh token:

```http
POST https://localhost:7103/api/Auth/refresh-token
Content-Type: application/json

{
  "userId": "<user-id>",
  "refreshToken": "<refresh-token>"
}
```

The admin endpoint is:

```http
GET https://localhost:7103/api/Auth/admin-only
Authorization: Bearer <access-token-for-a-user-with-the-Admin-role>
```

## Disclaimer

This is an educational project demonstrating authentication concepts. It has not necessarily been hardened for production use and should receive a thorough security, configuration, error-handling, and operational review before being deployed publicly.
