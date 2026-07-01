# CatalogService Package

## What is included

- Source for `CatalogService`
- Shared contracts used by the service
- External database script: `database/CatalogDb.sql`

## Connection contract

This repo is part of the 3-service library system. To connect correctly with the other teams, keep these values aligned across all services:

- `Jwt__Issuer` = `LibraryAuth`
- `Jwt__Audience` = `LibraryUsers`
- `Jwt__Key` = same secret in every service
- `InternalApi__Key` = same internal secret in every service

Catalog also needs to know where `CirculationService` is reachable:

- inside Docker Compose: `http://circulationservice:8080/integration/events/book-availability-changed`
- when deployed separately: replace with the other team's public host/IP and port

If these values do not match, stock update events will not reach circulation and protected requests will fail.

## Database setup

1. Open SQL Server Management Studio or Azure Data Studio.
2. Connect to your SQL Server instance.
3. Run `database/CatalogDb.sql`.
4. Verify the database name matches the service connection string.

## Default connection string

```json
Server=localhost,1433;Database=CatalogDb;User Id=sa;Password=Your_password123;TrustServerCertificate=True;Encrypt=False
```

## Required environment variables

If you run the service outside the main `docker compose` stack, set these values explicitly:

```bash
ConnectionStrings__CatalogDb=Server=YOUR_SQL_SERVER;Database=CatalogDb;User Id=sa;Password=...;TrustServerCertificate=True;Encrypt=False
Jwt__Issuer=LibraryAuth
Jwt__Audience=LibraryUsers
Jwt__Key=ChangeThisKeyToSomethingAtLeast32CharsLong!
InternalApi__Key=LibraryInternalSecretChangeMe!
IntegrationEvents__Subscribers__book.availability.changed__0=http://YOUR_CIRCULATION_HOST:5002/integration/events/book-availability-changed
```

## Run locally

```bash
dotnet restore
dotnet run --project src/CatalogService/CatalogService.csproj
```

## Run with Docker

```bash
docker build -f src/CatalogService/Dockerfile -t catalogservice .
docker run --rm -p 5001:8080 catalogservice
```

## Notes

- This service only needs SQL Server to start.
- When the whole system runs, it publishes stock changes to CirculationService.
- Do not change the JWT or internal API secrets in only one repo; all 3 services must share the same values.
- If the service is moved to another machine, update the stock-change webhook URL so Circulation still receives inventory updates.
