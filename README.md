# Meeting Scheduler

.NET 9 clean architecture sample focused on meeting scheduling. Uses minimal APIs + CQRS + domain events.

Features:
- Create users
- Schedule a meeting (finds earliest slot)
- List a user's meetings

## What it's using
- .NET 9
- Minimal APIs + controllers
- Serilog for logging
- Health check endpoint
- Docker support

## Project layout 
```
src/
  Domain        -> core entities and domain events
  Application   -> handlers, DTOs, commands & queries
  Infrastructure-> implementations (stuff that talks to outside world later)
  SharedKernel  -> shared base classes (Result, Error, Entity, etc.)
  Web.Api       -> actual API endpoints
tests/
  Unit + Integration + Architecture tests
```

## Prerequisites
Install .NET 9 SDK. (Optional) Docker Desktop if you want containers.
Check version:
```powershell
dotnet --version
```

## Build the solution
```powershell
dotnet restore
dotnet build
```
Warnings are treated as errors, so fix them if build fails.

## Run the API
```powershell
cd src/Web.Api
dotnet run
```
Look in the console for the actual URL (something like http://localhost:5080). In Development you also get Swagger (try adding /swagger).

## Run with Docker
```powershell
docker compose build
docker compose up
```
Then open: http://localhost:8080/swagger

Rebuild after changes:
```powershell
docker compose build --no-cache web-api
```

## Config files
Main ones are in `src/Web.Api/appsettings*.json`. Environment-specific overrides use the usual pattern.

## Tests
Run everything:
```powershell
dotnet test
```
Run one project:
```powershell
dotnet test tests/Domain.UnitTests/Domain.UnitTests.csproj
```
With coverage (simple way):
```powershell
dotnet test --collect:"XPlat Code Coverage"
```

## Health check
```
GET /health
```
Returns JSON with the health status.

## API Endpoints

### 1. Create user
```
POST /users
{
  "name": "Alice"
}
Returns 201 + user data
```

### 2. Get a user's meetings
```
GET /users/{userId}/meetings
Returns 200 + list (can be empty)
```

### 3. Schedule a meeting
```
POST /meetings
{
  "participantIds": [1,2,3],
  "durationMinutes": 30,
  "earliestStart": "2025-08-22T09:00:00Z",
  "latestEnd": "2025-08-22T17:00:00Z"
}
Returns 201 + meeting info
```

## Basic DTO shapes
CreateUserRequest
```json
{ "name": "string" }
```
ScheduleMeetingRequest
```json
{
  "participantIds": [1,2],
  "durationMinutes": 30,
  "earliestStart": "2025-08-22T09:00:00Z",
  "latestEnd": "2025-08-22T17:00:00Z"
}
```
UserDto
```json
{ "id": 1, "name": "Alice" }
```
MeetingDto
```json
{
  "id": 10,
  "startTime": "2025-08-22T09:30:00Z",
  "endTime": "2025-08-22T10:00:00Z",
  "participants": [ { "id": 1, "name": "Alice" } ]
}
```

## Logging
Serilog is set up already. You get structured logs in the console. Middleware adds some extra stuff (like correlation ids and timing).

## How things are wired
- Endpoints implement a small `IEndpoint` interface and get auto-registered.
- Commands/Queries go through handlers (CQRS style).
- There's a `Result<T>` type for success/error instead of throwing everywhere.