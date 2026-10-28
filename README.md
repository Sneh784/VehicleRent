# Vehicle Rent — local setup and implementation notes

This repository retains the existing React + TypeScript + Vite frontend, Java 21 + Spring Boot backend, and PostgreSQL database. It adds backend foundations for fleet manager role access, vehicle maintenance schedules, GPS location ingestion/history, customer complaints, and rental agreements, and a fleet operations UI for maintenance scheduling.

## Requirements
- Node.js 22+ and npm
- Java 21
- PostgreSQL 15+ (or compatible managed PostgreSQL)
- Maven 3.9+ or the included `vehicleBackend/mvnw` wrapper

## 1. Configure PostgreSQL
Create a database and user (example values; use your own password):

```sql
CREATE USER vehicle_rent WITH PASSWORD 'change-this-password';
CREATE DATABASE vehicle_rent OWNER vehicle_rent;
```

Copy `.env.example` as a reference; do not commit real values. Set environment variables in the terminal that will run the backend. `JWT_SECRET` must be a stable random value at least 32 bytes long. The backend uses `DB_URL`, `DB_USERNAME`, and `DB_PASSWORD`; it no longer embeds a remote database credential in `application.properties`.

For a disposable local database, set `JPA_DDL_AUTO=update` to create/update entity tables. For an existing database, back it up, review `vehicleBackend/db/manual_migration_postgres.sql`, apply it, and use `JPA_DDL_AUTO=validate`.

## 2. Run backend (PowerShell)
From the repository root:

```powershell
$env:DB_URL = 'jdbc:postgresql://localhost:5432/vehicle_rent'
$env:DB_USERNAME = 'vehicle_rent'
$env:DB_PASSWORD = 'your-local-password'
$env:JWT_SECRET = 'replace-with-a-random-secret-of-at-least-32-characters'
$env:JPA_DDL_AUTO = 'update'
cd vehicleBackend
./mvnw.cmd spring-boot:run
```

If the wrapper cannot download Maven, install Maven 3.9+ and run `mvn spring-boot:run`. Bootstrap admin/vendor accounts are optional; set `ADMIN_EMAIL` + `ADMIN_PASSWORD` or `VENDOR_EMAIL` + `VENDOR_PASSWORD` only when needed. Change/remove bootstrap credentials after initial setup.

## 3. Run frontend
In a second terminal:

```powershell
cd vehicle-Frontend
Copy-Item .env.example .env
npm ci
npm run dev
```

Open the Vite URL shown in the terminal (usually `http://localhost:5173`). `VITE_API_BASE_URL` should point to the backend API root, normally `http://localhost:8080/api`.

## New API foundations
- `POST /api/tracking/locations` — authenticated driver/vendor/fleet-manager/admin can ingest a GPS point; latitude/longitude are validated.
- `GET /api/tracking/vehicles/{vehicleId}` — authenticated location history endpoint.
- `/api/maintenance` — fleet manager/admin maintenance records.
- `GET/POST /api/complaints`, `PATCH /api/complaints/{id}` — user complaint submission and admin resolution.
- `GET/POST /api/agreements`, `POST /api/agreements/{id}/accept` — rental agreement draft and acceptance records.

## Build and test

```powershell
cd vehicle-Frontend
npm ci
npm run build
cd ..\vehicleBackend
./mvnw.cmd test
```

The code requires a live PostgreSQL instance to test end-to-end behavior. Automated browser tests are not included.

## Integration boundaries and production checklist
The backend now provides data models and API foundations, not complete external-provider integrations. Before production, implement and configure payment gateway order creation/webhook signature verification and refunds, secure file uploads to S3/Cloudinary (do not accept arbitrary public file URLs as trusted documents), OTP/Google/Apple identity providers, email/SMS delivery, Socket.IO/WebSocket or GPS device ingestion, Google Maps API restrictions, geofencing alerts, payout settlement, and booking-scoped authorization for live location history. Review the newly added endpoints with security tests, add pagination to admin reports, and rotate any database credential that was previously present in the original project archive.
