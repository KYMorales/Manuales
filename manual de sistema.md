# 📘 SYSTEM DOCUMENTATION

## 1. Project Information

Project Name: Vet KYMS  
Student Name: Katerin Morales  
Course: Base de Datos II  
Semester: 7  
Date: 20/11/25  
Instructor: Jaider Quintero  

Short Project Description: Full web solution for end-to-end veterinary clinic management. The Angular SPA handles owners, pets, clinical workflows, treatments and payments, while a Django REST backend exposes secured CRUD APIs over a relational database (default MySQL) that the SPA consumes.

## 2. System Architecture Overview

### 2.1 Architecture Description
- Single Page Application built with Angular 20 and PrimeNG served locally on port 4200.  
- Django 5 + Django REST Framework expose RESTful endpoints under `http://localhost:8000/api/`, plus JWT auth endpoints under `/api/auth/`.  
- Frontend communicates with backend via HTTPS/HTTP JSON, expecting DRF-style pagination `{count, next, previous, results}` with trailing slashes.  
- Database is configured via env vars; default engine is MySQL with utf8mb4 charset. Connectors for PostgreSQL/SQLite are available if configured.  
- CORS is enabled for local development; JWT lifetimes are set (60 min access, 1 day refresh) though DRF is currently permissive (`AllowAny`) for ease of development.

### 2.2 Technologies Used
- Frontend: Angular 20 (standalone components), PrimeNG 20 + PrimeIcons, Tailwind CSS v4, RxJS 7, TypeScript 5.8.
- Backend: Django 5.0.9, Django REST Framework 3.15, SimpleJWT, django-cors-headers.
- Database Engine: MySQL (env-driven `DATABASE_ENGINE`/`MYSQL_*`); connectors available for PostgreSQL/SQLite if envs are changed.
- Additional Libraries / Tools: BehaviorSubject caching in services, Karma/Jasmine for unit tests (front), DRF viewsets with custom permission hooks, REST Client `.http` samples, Faker for seeding (optional), Black formatter.

### 2.3 Visual explanation of the system’s operation

```
[ User Browser ]
        |
        v
 [ Angular 20 SPA ] --(HTTPS/HTTP JSON)--> [ Django REST API ]
        |                                         |
        |                              [+ JWT endpoints for login/refresh ]
        v                                         |
  PrimeNG UI, forms                     Business logic & serializers
        |                                         |
        '-----------------(ORM queries)----------> [ MySQL DB ]
```

## 3. Database Documentation (ENGLISH)

### 3.1 Database Description
Relational model that groups entities by domain: Core (owners, veterinarians, pets, medical records), Clinical (appointments, procedures, laboratories), Treatments (vaccines, applications, medications, prescriptions, prescription_medications), and Billing (payments). All entities share a status field (`active`|`inactive`) for lifecycle control. Dates use `DateField` or `DateTimeField` depending on context.

### 3.2 ERD – Entity Relationship Diagram
- Owner 1..* Pet  
- Pet 1..1 MedicalRecord  
- Pet 1..* Appointment  
- Appointment 0..* Procedure, 0..* Laboratory, 0..* VaccineApplication, 0..* Prescription  
- Prescription 0..* PrescriptionMedication and many-to-many with Medication through PrescriptionMedication  
- Owner 1..* Payment

### 3.3 Logical Model
- Core entities maintain clinic master data and link to clinical/billing flows.  
- Clinical entities capture visits (appointments) and related procedures/lab tests.  
- Treatments capture vaccines, medications, and prescriptions tied to appointments/pets.  
- Billing links payments to owners and can be expanded to invoices.  
- Status controls visibility and business actions; foreign keys cascade deletes unless otherwise noted (appointments keep veterinarian nullable).

### 3.4 Physical Model (Tables)

| Table | Column | Type | PK/FK | Description |
|-------|--------|------|-------|-------------|
| owners | id | BIGINT | PK | Auto increment |
| owners | first_name | VARCHAR(100) |  | Owner first name |
| owners | last_name | VARCHAR(100) |  | Owner last name |
| owners | phone | VARCHAR(20) |  | Contact phone |
| owners | email | VARCHAR |  | Optional email |
| owners | address | VARCHAR(255) |  | Optional address |
| owners | status | VARCHAR(10) |  | active/inactive |
| veterinarians | id | BIGINT | PK | Auto increment |
| veterinarians | first_name | VARCHAR(100) |  | Vet first name |
| veterinarians | last_name | VARCHAR(100) |  | Vet last name |
| veterinarians | license_id | VARCHAR(50) |  | Unique license number |
| veterinarians | phone | VARCHAR(20) |  | Optional phone |
| veterinarians | email | VARCHAR |  | Optional email |
| veterinarians | status | VARCHAR(10) |  | active/inactive |
| pets | id | BIGINT | PK | Auto increment |
| pets | owner_id | BIGINT | FK → owners.id | Pet owner |
| pets | name | VARCHAR(100) |  | Pet name |
| pets | species | VARCHAR(50) |  | Species |
| pets | breed | VARCHAR(100) |  | Optional breed |
| pets | sex | VARCHAR(10) |  | Optional sex |
| pets | birth_date | DATE |  | Optional birth date |
| pets | status | VARCHAR(10) |  | active/inactive |
| medical_records | id | BIGINT | PK | Auto increment |
| medical_records | pet_id | BIGINT | FK → pets.id (unique) | One-to-one pet record |
| medical_records | created_at | DATETIME |  | Auto timestamp |
| medical_records | notes | TEXT |  | Optional notes |
| medical_records | status | VARCHAR(10) |  | active/inactive |
| appointments | id | BIGINT | PK | Auto increment |
| appointments | pet_id | BIGINT | FK → pets.id | Visit pet |
| appointments | veterinarian_id | BIGINT | FK → veterinarians.id (nullable) | Assigned vet |
| appointments | date | DATETIME |  | Visit date/time |
| appointments | reason | VARCHAR(255) |  | Optional reason |
| appointments | notes | TEXT |  | Optional notes |
| appointments | status | VARCHAR(10) |  | active/inactive |
| procedures | id | BIGINT | PK | Auto increment |
| procedures | appointment_id | BIGINT | FK → appointments.id | Parent appointment |
| procedures | name | VARCHAR(100) |  | Procedure name |
| procedures | description | TEXT |  | Optional description |
| procedures | status | VARCHAR(10) |  | active/inactive |
| laboratories | id | BIGINT | PK | Auto increment |
| laboratories | appointment_id | BIGINT | FK → appointments.id | Related appointment |
| laboratories | test_name | VARCHAR(100) |  | Lab test name |
| laboratories | result | TEXT |  | Optional result |
| laboratories | date | DATE |  | Auto date |
| laboratories | status | VARCHAR(10) |  | active/inactive |
| vaccines | id | BIGINT | PK | Auto increment |
| vaccines | name | VARCHAR(100) |  | Vaccine name |
| vaccines | description | TEXT |  | Optional description |
| vaccines | status | VARCHAR(10) |  | active/inactive |
| vaccine_applications | id | BIGINT | PK | Auto increment |
| vaccine_applications | appointment_id | BIGINT | FK → appointments.id | Applied in visit |
| vaccine_applications | vaccine_id | BIGINT | FK → vaccines.id | Vaccine used |
| vaccine_applications | date_applied | DATE |  | Auto date |
| vaccine_applications | status | VARCHAR(10) |  | active/inactive |
| medications | id | BIGINT | PK | Auto increment |
| medications | name | VARCHAR(100) |  | Medication name |
| medications | description | TEXT |  | Optional description |
| medications | status | VARCHAR(10) |  | active/inactive |
| prescriptions | id | BIGINT | PK | Auto increment |
| prescriptions | pet_id | BIGINT | FK → pets.id | Prescribed to pet |
| prescriptions | appointment_id | BIGINT | FK → appointments.id | Linked visit |
| prescriptions | date | DATETIME |  | Auto timestamp |
| prescriptions | notes | TEXT |  | Optional notes |
| prescriptions | status | VARCHAR(10) |  | active/inactive |
| prescription_medications | id | BIGINT | PK | Auto increment |
| prescription_medications | prescription_id | BIGINT | FK → prescriptions.id | Parent prescription |
| prescription_medications | medication_id | BIGINT | FK → medications.id | Medication |
| prescription_medications | status | VARCHAR(10) |  | active/inactive |
| payments | id | BIGINT | PK | Auto increment |
| payments | owner_id | BIGINT | FK → owners.id | Paying owner |
| payments | amount | DECIMAL(10,2) |  | Payment amount |
| payments | date | DATETIME |  | Auto timestamp |
| payments | description | VARCHAR(255) |  | Optional description |
| payments | status | VARCHAR(10) |  | active/inactive |

## 4. Use Cases – CRUD

### 4.1 Use Case: Create Entity (example: Owner)
Actor: Clinic receptionist or admin  
Description: Register a new owner to link pets and payments.  
Preconditions: User has access to SPA; backend reachable; required fields provided.  
Postconditions: Owner stored with status active; list view refreshed.  
Main Flow: Navigate to `/owners/create` → fill form → submit → REST `POST /api/owners/` → toast success → redirect to `/owners`.

### 4.2 Use Case: Read Entity
Actor: Staff  
Description: View paginated lists or detail pages.  
Preconditions: Data exists; backend reachable.  
Main Flow: Open `/owners` → SPA calls `GET /api/owners/` → renders table with pagination and actions.

### 4.3 Use Case: Update Entity
Actor: Staff  
Description: Edit existing record while preserving relationships.  
Preconditions: Record exists; id present in route.  
Main Flow: `/owners/update/:id` → load data with `GET /api/owners/{id}/` → edit → `PUT /api/owners/{id}/` → refresh list and show toast.

### 4.4 Use Case: Delete Entity
Actor: Staff or Admin  
Description: Remove obsolete record.  
Preconditions: Record exists; user confirms deletion.  
Main Flow: Click delete in table → confirm dialog → `DELETE /api/owners/{id}/` → toast success → table refresh via `refresh*()` in service.

## 5. Backend Documentation

### 5.1 Backend Architecture
- Django project `vet_kyms` with four domain apps under `MyApps`: core, clinical, treatments, billing.  
- Each app exposes DRF `ModelViewSet` endpoints via a shared `BaseModelViewSet` that adds optional group-based filters, status restrictions, and statistics endpoints.  
- JWT endpoints available (`/api/auth/login`, `/refresh`, `/verify`, `/logout`) though DRF permissions are set to `AllowAny` for local testing.  
- Pagination: DRF `PageNumberPagination`, `PAGE_SIZE=20`, responses expose `results`.  
- CORS enabled (`corsheaders.middleware`) for SPA access.

### 5.2 Folder Structure

```
vet_kyms_back/
├── manage.py
├── requirements.txt
├── vet_kyms/                # Django project settings and urls
├── MyApps/
│   ├── core/                # Owners, veterinarians, pets, medical records
│   ├── clinical/            # Appointments, procedures, laboratories
│   ├── treatments/          # Vaccines, applications, medications, prescriptions
│   ├── billing/             # Payments
│   └── utils/permissions.py # Shared permission helpers (currently unused)
└── db.sqlite3 (local) or MySQL (env-configured)
```

### 5.3 API Documentation (REST)
- Base URL: `http://localhost:8000/api/` (trailing slash enabled).  
- Core: `owners/`, `veterinarians/`, `pets/`, `medical-records/`.  
- Clinical: `appointments/`, `procedures/`, `laboratories/`.  
- Treatments: `vaccines/`, `vaccine-applications/`, `medications/`, `prescriptions/`, `prescription-medications/`.  
- Billing: `payments/`.  
- Auth: `POST /api/auth/login/` (JWT), `POST /api/auth/refresh/`, `POST /api/auth/logout/`, `POST /api/auth/verify/`.  
- Extra per-model endpoints: `GET /api/<entity>/statistics/`, `GET /api/<entity>/admin_only_stats/`, `GET /api/<entity>/my_permissions/`.

Example create (Owner):
```
POST /api/owners/
{
  "first_name": "Ana",
  "last_name": "Gomez",
  "phone": "3001234567",
  "email": "ana@example.com",
  "address": "Calle 1",
  "status": "active"
}
```
Responses:  
- 200/201 on success (returns created/updated entity).  
- 400 on validation errors.  
- 403 on group-based restrictions (e.g., managers setting status inactive).  
- 204 on delete.

### 5.4 REST Client
- VS Code REST Client samples: `MyApps/core/auth/auth.http` and `MyApps/core/auth/owner.http`.  
- cURL example:
```
curl -X GET http://localhost:8000/api/pets/ -H "Accept: application/json"
```

## 6. Frontend Documentation

### 6.1 Technical Frontend Documentation
Framework Used: Angular 20 (standalone).  
Theme/UI: PrimeNG 20 (Aura preset), PrimeIcons, Tailwind CSS v4.  

Folder Structure:
```
vet_kyms_front/
├── src/
│   ├── main.ts               # bootstrapApplication
│   ├── app/
│   │   ├── app.config.ts     # providers: router, http, PrimeNG, animations
│   │   ├── app.routes.ts     # declarative routes for all CRUD screens
│   │   ├── app.html          # layout (header, aside, footer, router-outlet)
│   │   ├── components/       # layout + CRUD components grouped by domain
│   │   ├── services/         # HTTP services with BehaviorSubject cache
│   │   └── models/           # TS interfaces per domain
│   └── styles.css            # Tailwind + PrimeIcons imports
```
Models, services and Components:
- Models mirror backend schema: OwnerI, PetI, VeterinarianI, MedicalRecordI; AppointmentI/ProcedureI/LaboratoryI; VaccineI/VaccineApplicationI/MedicationI/PrescriptionI/PrescriptionMedicationI; PaymentI plus Response interfaces.  
- Services wrap CRUD with DRF pagination (map `results`) and expose `BehaviorSubject` streams (e.g., `owners$`).  
- Components follow `show/create/update` pattern using reactive forms, PrimeNG tables, Toast and ConfirmDialog. Routes cover all entities (`/owners`, `/appointments`, `/vaccines`, `/payments`, etc.).

### 6.2 Visual explanation of the system’s operation
- Persistent layout: header (brand, quick nav, mobile menu), aside with `p-panelMenu` organized by domain, main router outlet for CRUD pages, footer bar.  
- Tables show paginated/sortable data and action buttons; forms include validation and status select; success/error feedback via toasts.

## 7. Frontend–Backend Integration
- Base API URL expected: `http://localhost:8000/api/`; endpoints require trailing slash.  
- List calls expect DRF pagination and map `results` to local caches.  
- After create/update/delete, services call `refresh*()` to re-fetch data.  
- Status fields must be provided by the frontend; backend enforces group-based restrictions when authentication is enabled.  
- CORS is enabled; JWT headers (`Authorization: Bearer <token>`) can be added if auth is turned on.

## 8. Conclusions & Recommendations
- The stack already covers full CRUD across clinical, treatment, and billing domains with consistent patterns.  
- Improve security by re-enabling DRF authentication classes and enforcing JWT in production.  
- Externalize frontend base URL via `environment.ts` to support dev/QA/prod.  
- Add unit tests: DRF serializers/viewsets and Angular services/forms.  
- Consider migrations toward role-based UI (disable actions when `my_permissions` denies) and add search/filter for large datasets.

## 9. Annexes (Optional)
- Environment variables (backend): `DATABASE_ENGINE`, `MYSQL_NAME`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_HOST`, `MYSQL_PORT`, `SECRET_KEY` (override), `DEBUG` (production: False).  
- Development run commands: `python manage.py migrate && python manage.py runserver 0.0.0.0:8000`; frontend `npm install && npm start` inside `vet_kyms_front`.

---

## 10. Detailed Use Cases by Domain (Front + Back)

### 10.1 Owners
- Create: `POST /api/owners/` from `/owners/create` form. Validation: `first_name`/`last_name` required, `phone` regex `^\d{7,15}$`, status required.  
- Read: `GET /api/owners/?page=n` to fill PrimeNG table with pagination and sorting.  
- Update: `PUT /api/owners/{id}/` from `/owners/update/:id`.  
- Delete: `DELETE /api/owners/{id}/` with ConfirmDialog; service refreshes list.  
- Audit: `GET /api/owners/statistics/` shows total/active/inactive.

### 10.2 Pets
- Create: `POST /api/pets/` from `/pets/create`; requires `owner_id`, `name`, `species`.  
- Read: `GET /api/pets/` shows pet list and owner relation shorthand.  
- Update: `PUT /api/pets/{id}/`; cascades to medical record link if exists.  
- Delete: `DELETE /api/pets/{id}/`; cascades remove medical record per DB FK.  
- Relationship: One-to-one medical record created separately; front can add if needed.

### 10.3 Veterinarians
- Create: `POST /api/veterinarians/` with unique `license_id`.  
- Read: `GET /api/veterinarians/`.  
- Update/Delete: `PUT/DELETE /api/veterinarians/{id}/`.  
- Used by: appointments optional `veterinarian_id`.

### 10.4 Medical Records
- Create: `POST /api/medical-records/` with `pet_id` (unique).  
- Read: `GET /api/medical-records/`.  
- Update/Delete: `PUT/DELETE /api/medical-records/{id}/`.  
- Notes field allows free text; status controls visibility.

### 10.5 Appointments
- Create: `POST /api/appointments/` from `/appointments/create`; requires `pet_id`, `date`.  
- Read: `GET /api/appointments/` paginated; table may display pet and vet names.  
- Update/Delete: `PUT/DELETE /api/appointments/{id}/`.  
- Children: procedures, lab tests, vaccine applications, prescriptions reference appointment id.

### 10.6 Procedures
- Create: `POST /api/procedures/` under an appointment.  
- Read: `GET /api/procedures/`.  
- Update/Delete: `PUT/DELETE /api/procedures/{id}/`.  
- Optional description string; status for workflow visibility.

### 10.7 Laboratories
- Create: `POST /api/laboratories/`; auto `date` on creation.  
- Read: `GET /api/laboratories/`.  
- Update/Delete: `PUT/DELETE /api/laboratories/{id}/`.  
- Fields: `test_name`, `result` optional text.

### 10.8 Vaccines & Applications
- Vaccine CRUD: `/api/vaccines/`; simple name/description/status.  
- Applications: `/api/vaccine-applications/` with `appointment_id`, `vaccine_id`, auto `date_applied`.  
- Used to track immunization history tied to visits.

### 10.9 Medications & Prescriptions
- Medications: `/api/medications/` catalog.  
- Prescriptions: `/api/prescriptions/` referencing `pet_id` + `appointment_id`; many-to-many with medications via `prescription-medications`.  
- Junction CRUD: `/api/prescription-medications/`; ensures explicit status per link.

### 10.10 Payments
- Create: `POST /api/payments/` from `/payments/create`; requires `owner_id`, `amount`.  
- Read: `GET /api/payments/`; shows amount/owner/date.  
- Update/Delete: `PUT/DELETE /api/payments/{id}/`.  
- Suggest linking to invoices in future (not implemented).

---

## 11. API Endpoint Catalogue

### 11.1 Core
- `GET /api/owners/` — list (paginated)  
- `POST /api/owners/` — create  
- `GET /api/owners/{id}/` — retrieve  
- `PUT /api/owners/{id}/` — full update  
- `PATCH /api/owners/{id}/` — partial update  
- `DELETE /api/owners/{id}/` — delete  
- `GET /api/owners/statistics/` — total/active/inactive  
- `GET /api/owners/admin_only_stats/` — admin stats  
- `GET /api/owners/my_permissions/` — permission snapshot  
- Repeat same pattern for `veterinarians`, `pets`, `medical-records`.

### 11.2 Clinical
- `.../appointments/`, `.../procedures/`, `.../laboratories/` all expose the same set of CRUD plus `statistics`, `admin_only_stats`, `my_permissions`.

### 11.3 Treatments
- Vaccines: CRUD + stats endpoints.  
- Vaccine Applications: CRUD + stats endpoints.  
- Medications: CRUD + stats endpoints.  
- Prescriptions: CRUD + stats endpoints.  
- Prescription Medications: CRUD + stats endpoints.

### 11.4 Billing
- Payments: CRUD + stats endpoints.

### 11.5 Auth
- `POST /api/auth/login/` — returns access + refresh.  
- `POST /api/auth/refresh/` — refresh token.  
- `POST /api/auth/logout/` — blacklist refresh token when enabled.  
- `POST /api/auth/verify/` — validate token.  
- Current configuration leaves `DEFAULT_AUTHENTICATION_CLASSES` empty for ease of local use; enable JWT classes for production.

---

## 12. Validation Rules (Frontend + Backend)

- All entities: `status` required, limited to `active | inactive`.  
- Owners: `first_name`/`last_name` min length 2; `phone` 7–15 digits; `email` optional but must be valid format; `address` optional min length 5 when provided.  
- Pets: `owner_id`, `name`, `species` required; `birth_date` in `YYYY-MM-DD`.  
- Veterinarians: `license_id` unique; phone/email optional but validated formats.  
- Appointments: ISO datetime `date`; `pet_id` required; `veterinarian_id` optional.  
- Procedures/Laboratories: name/test_name required; `date` auto on lab creation.  
- Vaccines/Medications: name required.  
- Vaccine Applications: `appointment_id`, `vaccine_id` required; `date_applied` auto.  
- Prescriptions: `pet_id`, `appointment_id` required; `date` auto.  
- Prescription Medications: `prescription_id`, `medication_id` required.  
- Payments: `owner_id`, `amount` numeric; `date` auto; description optional.

---

## 13. Sample Payloads

### 13.1 Appointment Create
```
POST /api/appointments/
{
  "pet": 1,
  "veterinarian": 2,
  "date": "2025-01-10T14:30:00Z",
  "reason": "Vaccination booster",
  "notes": "Bring vaccination card",
  "status": "active"
}
```
Response `201 Created` with full entity JSON.

### 13.2 Prescription with Medications
```
POST /api/prescriptions/
{
  "pet": 1,
  "appointment": 3,
  "notes": "Administer twice daily",
  "status": "active"
}
```
Then link medications:
```
POST /api/prescription-medications/
{
  "prescription": 5,
  "medication": 2,
  "status": "active"
}
```

### 13.3 Payment Create
```
POST /api/payments/
{
  "owner": 1,
  "amount": "85.50",
  "description": "Consultation + vaccines",
  "status": "active"
}
```

### 13.4 JWT Login
```
POST /api/auth/login/
{
  "username": "admin",
  "password": "admin123"
}
```
Response:
```
{
  "refresh": "<token>",
  "access": "<token>"
}
```

---

## 14. Deployment & Environment

### 14.1 Backend Environment Variables
- `DATABASE_ENGINE=mysql` (default).  
- `MYSQL_NAME`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_HOST`, `MYSQL_PORT`.  
- Alternative engines via `DATABASE_ENGINE=postgresql` or `sqlite3` if added.  
- `SECRET_KEY` override; `DEBUG=False` for production; `ALLOWED_HOSTS` list required when not localhost.  
- CORS: tighten for prod (`CORS_ALLOW_ALL_ORIGINS = False`, set whitelist).  
- JWT: adjust lifetimes in `SIMPLE_JWT`.

### 14.2 Backend Commands
- Install deps: `pip install -r requirements.txt`.  
- Migrate: `python manage.py migrate`.  
- Create superuser: `python manage.py createsuperuser`.  
- Run dev server: `python manage.py runserver 0.0.0.0:8000`.  
- Load sample data: use Faker or custom management commands (not included yet).

### 14.3 Frontend Environment
- Node 20+, npm 10+.  
- Install: `cd vet_kyms_front && npm install`.  
- Dev server: `npm start` at `http://localhost:4200/`.  
- Build: `npm run build` outputs `dist/vet-kyms`.  
- Env config: recommended `src/environments/environment.ts` with `apiUrl`.

### 14.4 Production Considerations
- Serve Angular build via Nginx/Apache; configure SPA fallback to `index.html`.  
- Serve Django via WSGI/ASGI (Gunicorn/Uvicorn) behind reverse proxy; configure HTTPS and database credentials.  
- Run `collectstatic` if static handling is added.  
- Set proper CORS/CSRF and JWT enforcement for secured deployments.

---

## 15. Testing Strategy

- Frontend: Karma/Jasmine (`npm test`); add specs for services (HttpTestingController) and forms (validators).  
- Backend: Add DRF test cases per app (serializers, viewsets, permissions). Use `python manage.py test MyApps.core` etc.  
- Integration: Manual smoke via REST Client `.http` files or cURL; optional Postman collection.  
- Future: E2E via Playwright/Cypress targeting mock or QA API.  
- Data validation: Include boundary tests for phone regex, date formats, status filtering.

---

## 16. Operations Runbook (Dev/QA)

- Start DB (MySQL) and ensure credentials present.  
- Backend up: `python manage.py migrate && python manage.py runserver 0.0.0.0:8000`.  
- Frontend up: `npm start`.  
- Health check: `GET http://localhost:8000/api/owners/` returns paginated JSON.  
- Login test (if auth enabled): `POST /api/auth/login/` with valid user.  
- Create smoke data: POST one owner, one pet, one appointment, one payment; verify list screens.  
- Logs: Django dev server stdout shows CRUD messages for create/update/delete.  
- Backup: use DB-native dump (mysqldump) per environment policy.

---

## 17. Troubleshooting Matrix

- 400 Bad Request: check required fields, phone regex, date format.  
- 401/403: JWT missing/expired or group restriction (managers cannot set status inactive).  
- 404: wrong endpoint or missing trailing slash; confirm id exists.  
- 500: inspect backend logs; verify DB connectivity (`DATABASE_ENGINE` and credentials).  
- CORS errors: configure allowed origins; ensure frontend uses correct `apiUrl`.  
- Empty tables: confirm backend returns `results` in pagination; services map `results`.  
- UNIQUE constraint (license_id): ensure distinct veterinarian license.  
- Foreign key constraint: deleting parent with children (e.g., owner with pets) cascades; confirm intended.

---

## 18. Security Notes

- Development defaults to `AllowAny`; enable JWT auth in DRF `DEFAULT_AUTHENTICATION_CLASSES` and set `DEFAULT_PERMISSION_CLASSES` to `IsAuthenticated` for production.  
- Use HTTPS and secure cookies if sessions are later used.  
- Rotate `SECRET_KEY` and disable `DEBUG` outside local.  
- Limit CORS origins; disable `CORS_ALLOW_ALL_ORIGINS` in prod.  
- Add rate limiting (e.g., Django REST throttling) if exposed publicly.  
- Validate inputs server-side; current serializers rely on DRF model validation.  
- Consider soft deletes if compliance requires data retention; currently hard deletes via CASCADE.

---

## 19. UX Patterns (Frontend)

- Listing pages: `p-table` with pagination, sort, empty state template, action buttons (edit, delete) with tooltips.  
- Forms: reactive; `form.markAllAsTouched()` on submit failures; inline error messages; status select.  
- Feedback: `MessageService` toasts for success/error; `ConfirmationService` for deletes.  
- Layout: header with brand and hamburger for mobile; aside with `p-panelMenu`; main content flex grows; footer bar simple.  
- Styling: Tailwind utilities for spacing, grid, rounded corners; PrimeNG Aura theme.  
- Accessibility: keyboard focus via PrimeNG; add `aria-label` for icon-only buttons as needed.

---

## 20. Data Flow Examples

### 20.1 Create Appointment Flow
1) User opens `/appointments/create`.  
2) Component builds reactive form; may preload pets/vets options (future enhancement).  
3) On submit, `appointmentService.createAppointment(payload)` triggers `POST /api/appointments/`.  
4) Backend viewset validates and saves; returns created object.  
5) Service calls `refreshAppointments()` → `GET /api/appointments/` to update BehaviorSubject.  
6) Component subscribes to `appointments$`; toast shows success; router navigates to `/appointments`.

### 20.2 Delete Owner Flow
1) In `/owners`, user clicks delete → ConfirmDialog.  
2) On accept, `ownerService.deleteOwner(id)` → `DELETE /api/owners/{id}/`.  
3) Backend cascades to pets and dependent records (per FK).  
4) Service refreshes list; toast success; table updates via subscription.

### 20.3 Payment Recording Flow
1) User navigates to `/payments/create`.  
2) Fills owner and amount; submit triggers `POST /api/payments/`.  
3) Backend creates payment with auto timestamp; returns entity.  
4) Service refreshes; table shows new payment and updated totals can be read from `statistics`.

---

## 21. Suggested Roadmap (Expanded)

- Implement `environment.ts` usage and base URL injection on frontend.  
- Add auth guards and route protection; surface `my_permissions` to disable UI actions.  
- Add search/filtering and server-side pagination controls (page size options).  
- Introduce dashboards: upcoming appointments, payments per month, active vs inactive counts with charts.  
- Include attachments/uploads for lab results or vaccination proofs (requires storage).  
- Add notification system (email/SMS) for appointment reminders.  
- Improve validation messages and backend serializer error mapping to UI.  
- Add CI pipeline (lint, test, build) and containerization (Docker).  
- Performance: consider `select_related/prefetch_related` in viewsets for relational efficiency.  
- Observability: integrate Sentry or similar; custom `GlobalErrorHandler` for frontend.

---

## 22. Example ERD ASCII (Simplified)

```
Owners (1) ------< Pets (1) -----< Appointments >----- (0..1) Veterinarians
   |                  |              |
   |                  |              +----< Procedures
   |                  |              +----< Laboratories
   |                  |              +----< VaccineApplications >---- Vac-catalog
   |                  |              +----< Prescriptions >----< PrescriptionMedications >---- Medications
   |                  |
   |                  +--(1:1)-- MedicalRecords
   |
   +----< Payments
```

---

## 23. Backend Serializer Notes

- DRF ModelSerializers mirror models; default create/update used.  
- Pagination set globally with `PAGE_SIZE=20`; adjust per viewset if needed.  
- Status choices enforced by model field choices.  
- Unique constraint on `Veterinarian.license_id`; DB enforces.  
- Date fields with `auto_now_add` ensure server-side timestamps (appointments use provided datetime).

---

## 24. Permission & Group Behavior (if enabled)

- BaseModelViewSet filters active records for users in group `Empleados` (requires auth setup).  
- Managers (`Gerentes`) blocked from creating records with status inactive or changing status to inactive.  
- Admin stats endpoint intended for admin roles.  
- `my_permissions` returns CRUD capability flags per model and user groups.  
- Currently authentication classes are empty to allow public CRUD during development; enable JWT to enforce the above.

---

## 25. Configuration Checklist (Production Hardening)

- [ ] Set `DEBUG=False`.  
- [ ] Configure `ALLOWED_HOSTS`.  
- [ ] Enable JWT auth classes in DRF settings.  
- [ ] Restrict CORS origins.  
- [ ] Configure database with credentials and SSL if required.  
- [ ] Set `TIME_ZONE` and `USE_TZ=True` if consistent UTC handling is needed.  
- [ ] Migrate database; run `createsuperuser`.  
- [ ] Add logging handlers (file/JSON) and monitoring.  
- [ ] Backup/restore strategy defined.  
- [ ] HTTPS termination configured.

---

## 26. Data Seeding Idea (Optional)

- Use Faker to create sample owners, pets, appointments, and payments.  
- Management command sketch:
  - Create N owners with random phones.  
  - For each owner, create pets and random appointments in the last 30 days.  
  - Generate payments tied to owners.  
- Useful for demo and frontend pagination during development.

---

## 27. REST Client Snippets (VS Code `.http`)

- `GET http://localhost:8000/api/owners/`  
- `POST http://localhost:8000/api/owners/`
```
{
  "first_name": "Test",
  "last_name": "User",
  "phone": "3000000000",
  "status": "active"
}
```
- `DELETE http://localhost:8000/api/procedures/1/`  
- Auth:
```
POST http://localhost:8000/api/auth/login/
Content-Type: application/json

{
  "username": "admin",
  "password": "admin"
}
```

---

## 28. Versioning & Formatting

- Backend: use Black (`black .`) for formatting.  
- Frontend: Prettier (width 100, singleQuote true); Angular 20 defaults.  
- Git hooks not configured; CI/CD suggested for consistency.  
- No explicit semantic versioning; recommend tagging releases when deployed.

---

## 29. Logging & Monitoring

- Current: console prints on create/update/delete in viewsets.  
- Future: integrate Django logging config; send to file/ELK/Sentry.  
- Frontend: add global error handler to capture HTTP errors and show contextual toasts; optional telemetry.

---

## 30. Accessibility Quick Wins

- Add `aria-label` to action buttons in tables.  
- Ensure focus ring visibility; PrimeNG provides defaults.  
- Check color contrast in header/sidebar (pink palette) for AA compliance.  
- Support keyboard navigation in forms; avoid key-trap components.  
- Provide toast role `status` or `alert` if necessary.

---

## 31. Performance Considerations

- Backend: add `select_related` on FK-heavy viewsets (appointments include pet/vet) and `prefetch_related` for nested data if returned.  
- Frontend: consider OnPush change detection and `trackBy` on tables; add server-side filters to limit payload size.  
- Database: indexes on FK and frequently filtered fields (`status`, `date`).  
- Caching: not implemented; could add per-endpoint caching or CDN for static assets.

---

## 32. Future Integrations

- Authentication/Authorization: JWT enforced, role-based UI.  
- Payments: integrate payment gateways or invoicing; current model is simple record.  
- Telemedicine: add online appointment links.  
- Inventory: track medication stock levels.  
- Reporting: CSV/PDF export for appointments and payments.  
- Notifications: email/SMS for reminders (use SendGrid already in requirements).  
- PWA/offline mode with caching strategies.

---

## 33. Glossary

- SPA: Single Page Application (Angular).  
- DRF: Django REST Framework.  
- JWT: JSON Web Token.  
- CRUD: Create, Read, Update, Delete.  
- BehaviorSubject: RxJS subject holding the latest value for subscribers.  
- PanelMenu: PrimeNG component for side navigation.

---

## 34. Changelog (v1.0)

- Added full CRUD for Core, Clinical, Treatments, Billing.  
- Implemented DRF viewsets with statistics and permission info endpoints.  
- Configured PrimeNG Aura theme and Tailwind v4.  
- Added pagination handling on frontend services.  
- Provided JWT endpoints (dev mode permissive).  
- Documented entity relationships and payloads.

---

## 35. Quick Start Recap

1) Backend: set env vars → `pip install -r requirements.txt` → `python manage.py migrate` → `python manage.py runserver`.  
2) Frontend: `cd vet_kyms_front && npm install && npm start`.  
3) Open `http://localhost:4200/`; verify lists load from `http://localhost:8000/api/`.  
4) Use CRUD screens to manage clinic data; watch terminal logs for backend operations.

---

## 36. Database Connectivity Notes

- Default uses MySQL with utf8mb4 charsets to support emoji (e.g., 🐾).  
- Alternative connectors in requirements: `psycopg2-binary` for PostgreSQL, `mysqlclient` for MySQL, `pyodbc` for SQL Server (via `mssql-django`).  
- Update `DATABASE_ENGINE` and corresponding env vars; run migrations after switching.  
- For SQLite dev, adjust settings to use `django.db.backends.sqlite3` (not preconfigured but easy to add).

---

## 37. Timezone & Localization

- Backend `TIME_ZONE = 'America/Bogota'`, `USE_TZ = False`; adjust to UTC best practice if multi-region.  
- Frontend uses ISO strings/dates; ensure consistent offset handling if enabling `USE_TZ=True`.  
- Language currently Spanish on backend (`LANGUAGE_CODE = 'es'`); UI strings in Spanish; documentation now in English.

---

## 38. Data Protection & Privacy

- Medical notes may contain sensitive data; enforce access controls before production.  
- Ensure backups are encrypted; control DB access via least privilege.  
- Audit logs recommended for create/update/delete actions beyond current console logs.  
- If deploying in regulated regions, add privacy policy and consent flows.

---

## 39. Error Handling Pattern (Frontend)

- Services use `catchError` + `throwError`; components show toasts.  
- Recommended: central HTTP interceptor to map status codes to friendly messages (e.g., 400 validation, 401 login required, 403 forbidden, 500 server error).  
- Add loading indicators during async operations; disable submit buttons while pending to prevent duplicates.

---

## 40. Continuous Improvement Ideas

- Add breadcrumbs per route.  
- Show status chips in tables (active/inactive).  
- Include quick filters (status, date range for appointments/payments).  
- Add export to CSV for owners/pets/appointments.  
- Introduce dashboard landing page with KPIs (appointments today, payments today).

---

## 41. Maintenance Tips

- Regularly update dependencies (`npm update`, `pip list --outdated`).  
- Run migrations on staging before production.  
- Keep secrets out of version control; use `.env` (already present).  
- Validate `.env` sample for new contributors; document required variables.  
- Monitor DB growth; archive old records if necessary.

---

## 42. DB Table Relationships (Verbose)

- `Owner` → `Pet` (CASCADE).  
- `Pet` → `MedicalRecord` (OneToOne, CASCADE).  
- `Pet` → `Appointment` (CASCADE).  
- `Appointment` → `Procedure`/`Laboratory`/`VaccineApplication`/`Prescription` (CASCADE).  
- `Prescription` → `PrescriptionMedication` (CASCADE); `Medication` referenced via FK.  
- `Appointment` → `VaccineApplication` → `Vaccine`.  
- `Owner` → `Payment` (CASCADE).

---

## 43. PrimeNG Components in Use

- `p-table`, `p-button`, `p-panelMenu`, `p-toast`, `p-confirmDialog`, `p-select`, `p-tooltip`.  
- Animations provided by `provideAnimationsAsync()`.  
- Icons via PrimeIcons import in `styles.css`.

---

## 44. Tailwind Usage

- Using Tailwind v4 `@import "tailwindcss";` with utility classes in templates.  
- Palettes lean to soft pinks for branding; adjust via custom CSS or Tailwind config if needed.  
- Can add `tailwind.config.ts` to extend theme and generate design tokens.

---

## 45. Build & Serve Details

- Angular dev server supports HMR-like reloads.  
- Production build `npm run build` produces optimized bundles; deploy `dist/vet-kyms`.  
- Django serves API only; static handling not configured for Angular (handled separately).

---

## 46. Backup & Restore (Database)

- MySQL: `mysqldump -u user -p db > backup.sql`; restore with `mysql db < backup.sql`.  
- Postgres (if used): `pg_dump` / `psql` restore.  
- Ensure downtime or locks according to size; schedule regular backups for production.

---

## 47. KPIs to Track (Future)

- Daily appointments count and completion rate.  
- Revenue per day/week/month (payments).  
- Vaccine application counts per period.  
- Active vs inactive records by domain.  
- Average appointment duration (if start/end added later).

---

## 48. Observability Hooks (Future)

- Add middleware for request/response timing.  
- Add database query logging on slow queries.  
- Frontend: wrap service calls with timing metrics; send to monitoring endpoint.  
- Error context: include user id and route on captured errors (respect privacy).

---

## 49. Data Export Ideas

- CSV exports for owners/pets/appointments/payments.  
- PDF summaries for prescriptions or lab results (requires rendering library).  
- API endpoints could support `?format=csv` with DRF renderers.

---

## 50. Support & Contact

- Issues during setup: verify env vars, DB connectivity, and API base URL.  
- For role-based or auth questions: enable JWT and group permissions as described.  
- For UI/UX feedback: adjust Tailwind/PrimeNG theme and navigation structure.  
- For data model changes: update models, migrations, serializers, services, and components consistently.
