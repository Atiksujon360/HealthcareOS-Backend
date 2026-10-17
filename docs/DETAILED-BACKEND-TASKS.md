# HealthcareOS Backend — Detailed Task Specifications

This document is the source of truth for detailed backend task descriptions that will later be copied into the matching ClickUp tasks. These are **planned tasks**, not proof that the work is already implemented.

## Task 1 — Scaffold the Spring Boot modular monolith

### Goal
Create a clean, buildable Java/Spring Boot backend without disrupting the existing HealthcareOS frontend.

### Learn first
JDK vs JVM, Maven, Spring Boot, dependency injection, annotations, packages, controller/service/repository/DTO responsibilities.

### Steps
1. Inspect the repository, branch, existing files, and frontend/backend integration expectations before editing.
2. Create the backend project with a supported Java LTS release and compatible Spring Boot version.
3. Add only the required dependencies initially: Spring Web, Validation, Spring Data JPA, PostgreSQL driver, Flyway, Spring Security, Actuator, and test dependencies.
4. Establish domain packages for config, common, identity, tenant, patient, appointment, clinical, pharmacy, and audit.
5. Add safe local/test configuration; do not commit credentials or API keys.
6. Add a simple health endpoint and explain how to run the app from IntelliJ and PowerShell.
7. Add README instructions for prerequisites, build, run, tests, environment variables, and troubleshooting.
8. Keep business rules out of controllers and avoid exposing persistence entities directly.

### Acceptance criteria
- Clean checkout builds successfully.
- Application starts with documented local configuration.
- Health endpoint returns a successful response.
- Tests can be run from the command line.
- Frontend remains unaffected.
- No secrets or real patient information are committed.

### Deliverables
Build file, source layout, minimal endpoint, tests, README and architecture sketch.

### Learning resources
- Bangla: https://www.youtube.com/results?search_query=Java+Spring+Boot+Bangla+beginner+project
- Official: https://spring.io/guides

---

## Task 2 — Implement authentication and session lifecycle

### Goal
Establish real server-side identity and remove frontend role selection as a security authority.

### Steps
1. Agree the login identifier, account status, session strategy, expiry, logout, revocation and account-disable behavior.
2. Define identity and session persistence models and migrations.
3. Use a well-supported adaptive password hash; never store plaintext passwords.
4. Configure Spring Security with explicit public routes and deny-by-default behavior for protected routes.
5. Implement validated login/logout and session expiry/revocation.
6. For browser sessions, choose secure HttpOnly cookies and explicitly configure Secure, SameSite and CSRF based on the deployment topology.
7. Configure explicit trusted CORS origins and rate limiting at the suitable application/edge layer.
8. Return consistent errors without disclosing whether an account exists.
9. Add safe security audit events without recording passwords, tokens, cookies or unnecessary health data.
10. Test invalid credentials, disabled accounts, expiry, logout, revocation, cookie flags and abuse controls.

### Acceptance criteria
- Protected endpoints reject unauthenticated requests with 401.
- Logout/revocation invalidates the session server-side.
- Client-supplied role values cannot grant permissions.
- Secrets are not stored in source control or logs.
- Automated security tests pass.

### Deliverables
Identity/session schema and migrations, security configuration, endpoint contract, threat notes and tests.

### Learning resources
- Bangla: https://www.youtube.com/results?search_query=Spring+Security+authentication+Bangla+beginner
- Official: https://spring.io/guides/gs/securing-web/

---

## Task 3 — Implement tenant, organization, branch and membership APIs

### Goal
Ensure each healthcare organization and its branches can manage authorized staff while tenant data remains isolated.

### Steps
1. Define organization, branch and tenant-membership lifecycle states.
2. Model membership explicitly; do not assume every user belongs to only one organization unless that is a confirmed product rule.
3. Document the role/permission matrix for clinic administrators, clinicians, pharmacy staff and platform administrators.
4. Implement authorized create/read/update and membership-management APIs.
5. Validate that each branch belongs to the selected organization and membership is active.
6. Derive tenant scope from authenticated identity and server-side membership; never trust a browser-provided tenant ID alone.
7. Apply tenant-scoped repository/query patterns and tenant-aware database constraints.
8. Audit membership and permission changes.
9. Test cross-tenant ID guessing, inactive memberships, unknown branches and privilege escalation attempts.

### Acceptance criteria
- A user accesses only organizations and branches they are authorized for.
- Tenant A cannot read, change, enumerate or infer Tenant B records.
- Membership deactivation takes effect according to documented policy.
- No permission is enforced only in the frontend.

### Deliverables
Entities, migrations, repositories, services, controllers, OpenAPI examples, permission matrix and authorization tests.

### Learning resources
- Bangla: https://www.youtube.com/results?search_query=Spring+Boot+multi+tenant+application+Bangla
- Official: https://spring.io/guides/gs/securing-web/

---

## Task 4 — Implement patient, appointment and queue APIs

### Goal
Support the first end-to-end clinic operations workflow using synthetic records.

### Steps
1. Confirm which staff roles can register, search, view and update patient records.
2. Implement patient create, detail, update and paginated search APIs with minimum necessary fields.
3. Define a reviewed patient matching policy; do not automatically merge patients based on name alone.
4. Define appointment states and valid transitions with the product owner.
5. Implement booking, list/filter, reschedule and cancellation with branch and clinician checks.
6. Define timezone handling and prevent conflicting bookings under concurrent requests.
7. Implement queue check-in, call-next and status updates with role checks and safe state transitions.
8. Limit public queue displays to approved minimal information.
9. Use DTOs, validation, consistent errors and OpenAPI documentation.
10. Add PostgreSQL integration tests for tenant scope, uniqueness, pagination, conflicts and concurrent operations.

### Acceptance criteria
- Authorized staff can create/find a synthetic patient and book/check in an appointment.
- Invalid state transitions and unauthorized access are rejected.
- Cross-tenant records are inaccessible.
- Duplicate check-ins and booking conflicts are handled predictably.
- List/search endpoints are paginated.

### Deliverables
Patient/appointment/queue APIs, migrations, state diagram, OpenAPI examples and tests.

### Learning resources
- Bangla: https://www.youtube.com/results?search_query=Spring+Boot+REST+API+PostgreSQL+Bangla+CRUD
- Official: https://spring.io/guides/gs/rest-service/

---

## Task 5 — Implement encounters, consultation notes and prescriptions

### Goal
Support traceable clinical workflows without silent overwrites or unauthorized signing.

### Steps
1. Confirm required fields and workflow states with an appropriate clinical/product reviewer.
2. Link each encounter to the correct patient, tenant/branch and authorized clinician.
3. Define valid encounter transitions and enforce them server-side.
4. Implement draft and finalized consultation notes with author and timestamps.
5. Define post-finalization correction as a documented amendment/revision where required; preserve history.
6. Model prescription headers and items separately, with clinically approved fields and validation.
7. Implement prescription draft, review, finalize/sign, cancel and amend actions according to approved workflow.
8. Ensure only authorized clinicians can finalize prescriptions and cannot sign for another clinician.
9. Use transactions for multi-record clinical changes and audit actor/action/time/revision metadata.
10. Test permissions, invalid transitions, amendment history, rollback and duplicate requests.

### Acceptance criteria
- Clinical actions are authorized by the backend.
- Finalized records remain traceable and cannot be silently overwritten.
- Unauthorized signing is rejected.
- Failed multi-record operations leave no partial state.
- Workflow is tested with synthetic patients only.

### Deliverables
Encounter, note and prescription APIs; migrations; state diagrams; permission matrix; audit/revision design; tests.

### Learning resources
- Bangla: https://www.youtube.com/results?search_query=Spring+Boot+JPA+REST+API+Bangla
- Official: https://spring.io/guides/gs/rest-service/

---

## Task 6 — Implement pharmacy dispensing, stock movement and audit

### Goal
Make inventory changes accurate, atomic, traceable and safe under concurrent requests.

### Steps
1. Separate medicine catalog records from branch-specific stock and inventory lots.
2. Define lot/batch, expiry, branch, source and quantity fields as needed by the approved workflow.
3. Record receipts, adjustments, dispensing, returns, expiry and write-offs as stock movements.
4. Implement authorized receipt and adjustment APIs with reasons and actor attribution.
5. Validate prescription, branch, medicine matching, lot expiry and available quantity before dispensing.
6. Perform stock validation, dispense record creation, stock movement and balance updates in one transaction.
7. Prevent overselling using appropriate locking or atomic conditional updates and database constraints.
8. Protect against duplicate dispense requests using idempotency or a unique request key.
9. Define correction and return flows without deleting historical movements.
10. Keep medicine source/provenance explicit; never call mock data verified official data without evidence.
11. Test insufficient/expired stock, concurrent dispensing, retries, rollback, tenant access and reconciliation.

### Acceptance criteria
- Stock never falls below zero in tested concurrent scenarios.
- Every stock change has a traceable actor, reason and source.
- Failed dispensing leaves no partial state.
- Duplicate retries do not double-dispense.
- Stock balance reconciles with movement history.

### Deliverables
Medicine/inventory migrations, services and APIs, transaction design, reconciliation procedure and tests.

### Learning resources
- Bangla: https://www.youtube.com/results?search_query=Spring+Boot+inventory+management+PostgreSQL+Bangla
- Official: https://docs.spring.io/spring-framework/reference/data-access/transaction.html

---

## Task 7 — Publish OpenAPI contracts and standard errors

### Goal
Allow the frontend and backend to integrate through a stable, reviewed API contract.

### Steps
1. Version routes under /api/v1.
2. Add OpenAPI/Swagger UI for development and test environments.
3. Define request/response DTOs for all endpoints; do not serialize JPA entities directly.
4. Standardize error responses with safe error code, message, field errors where relevant, request/correlation ID and timestamp.
5. Document required authentication and permissions per operation.
6. Document pagination, sorting, filtering, date/time formats, nullability, state values and conflict behavior.
7. Add success, validation, unauthorized, forbidden, not-found and conflict examples.
8. Add contract tests and update frontend types deliberately when contracts change.

### Acceptance criteria
- Frontend implementation can follow the API contract without inspecting persistence classes.
- Error responses are predictable and contain no stack traces, SQL or secrets.
- API documentation matches actual tested behavior.
- Breaking changes are identified and versioned deliberately.

### Deliverables
OpenAPI document, API conventions, error schema, permission matrix and contract tests.

### Learning resources
- Bangla: https://www.youtube.com/results?search_query=Swagger+OpenAPI+Spring+Boot+Bangla
- Official: https://springdoc.org/

---

## Task 8 — Create ERD and data dictionary

### Goal
Agree the MVP data model before production migrations grow.

### Steps
1. Review frontend types/mock data as requirements clues, not as the final schema.
2. Model identity, tenant, branch, membership, patient, appointment, queue, encounter, notes/revisions, prescription/items, medicine, inventory lots, stock movements, dispense and audit events.
3. Document each field's meaning, SQL type, nullability, validation, sensitivity, source and retention considerations.
4. Define keys, relationships, tenant-aware uniqueness, check constraints and foreign keys.
5. Mark which tables are tenant-owned and how tenant scope is enforced.
6. Design clinical revision and inventory movement history.
7. Add indexes based on expected query paths and document why each is needed.
8. Review retention and deletion policies with qualified local reviewers.

### Acceptance criteria
- Every MVP workflow maps to a coherent model.
- Database constraints protect integrity independently of application code.
- Sensitive fields and retention questions are documented.
- ERD is reviewed before production schema is finalized.

### Deliverables
ERD source/render, data dictionary, tenant ownership matrix, index rationale and decision log.

### Learning resources
- Bangla: https://www.youtube.com/results?search_query=PostgreSQL+database+design+ERD+Bangla
- Official: https://www.postgresql.org/docs/

---

## Task 9 — Set up PostgreSQL and Flyway

### Goal
Make database setup and schema evolution repeatable for local development, CI and staging.

### Steps
1. Create separate development and test databases.
2. Create a least-privilege application database user; do not run the app as a PostgreSQL superuser.
3. Supply connection settings through environment variables or ignored local configuration.
4. Add ordered Flyway migrations (for example, V1__create_identity_tables.sql).
5. Treat applied shared-environment migrations as immutable; add a new migration for later changes.
6. Use Hibernate schema validation rather than production auto-create/update behavior.
7. Keep synthetic fixture loading separate from schema migrations and disable it in production by default.
8. Test clean database creation, restart, migration history and failure behavior.
9. Add a disposable PostgreSQL migration check to CI.
10. Document backup/restore and forward-fix procedures.

### Acceptance criteria
- A clean database can be built from version-controlled migrations alone.
- Restart does not duplicate schema changes.
- Migration failures block release.
- Credentials are not committed or logged.
- Application credentials are least privilege.

### Deliverables
Flyway config/migrations, setup guide, fixture approach, CI checks and troubleshooting notes.

### Learning resources
- Bangla: https://www.youtube.com/results?search_query=PostgreSQL+Flyway+Spring+Boot+Bangla
- Official: https://documentation.red-gate.com/flyway

---

## Task 10 — Enforce tenant-safe constraints, indexes and queries

### Goal
Protect data integrity and make the main MVP query paths safe and predictable.

### Steps
1. List query patterns: patient search, appointments by date/status, queue display, encounter history, prescription lookup and inventory by branch/expiry.
2. Add indexes for measured filter/sort patterns; avoid indexing every column.
3. Add foreign keys, check constraints and tenant-aware unique constraints.
4. Require authorized tenant context for tenant-owned repository operations.
5. Use parameterized queries and safe search patterns.
6. Apply pagination, stable sorting and maximum page sizes.
7. Use EXPLAIN (ANALYZE, BUFFERS) with representative synthetic data for important queries.
8. Test guessed IDs, missing tenant context and cross-tenant access.

### Acceptance criteria
- Cross-tenant reads and writes fail.
- Constraints reject invalid relationships and duplicates.
- List endpoints are paginated and stable.
- Query-plan evidence exists for important query paths.
- Sensitive patient information is not written to query logs by default.

### Deliverables
Constraint/index migrations, tenant-safe repository tests, query-plan notes and synthetic benchmark data.

### Learning resources
- Bangla: https://www.youtube.com/results?search_query=PostgreSQL+index+foreign+key+constraint+Bangla
- Official: https://www.postgresql.org/docs/current/performance-tips.html

---

## Task 11 — Design clinical revisions, audit and inventory transactions

### Goal
Make high-impact clinical and pharmacy operations traceable and consistent after failures or retries.

### Steps
1. Define draft versus finalized clinical records and the amendment policy.
2. Record actor, action, timestamp, target record and revision linkage for critical changes.
3. Restrict access to audit/revision data and document retention questions for review.
4. Define dispensing's atomic boundary: authorization, stock check/reservation, dispense record and stock movement.
5. Choose and document concurrency control and transaction isolation for stock invariants.
6. Prevent retries from creating duplicate dispensing or stock movements.
7. Define rollback and reconciliation behavior for failures and anomalies.
8. Add PostgreSQL integration tests for concurrent requests, duplicate submissions, forced rollback and unauthorized attempts.
9. Ensure application logs do not become an uncontrolled copy of clinical data.

### Acceptance criteria
- Clinical amendments preserve traceable history.
- Inventory cannot silently drift under tested retries/concurrency.
- Failed operations leave no partial state.
- Reviewers approve the design before real patient data is used.

### Deliverables
Revision/audit design, transaction sequence diagram, migrations/constraints, rollback/concurrency tests and reconciliation procedure.

### Learning resources
- Bangla: https://www.youtube.com/results?search_query=Spring+Boot+transaction+management+Bangla
- Official: https://docs.spring.io/spring-framework/reference/data-access/transaction.html
