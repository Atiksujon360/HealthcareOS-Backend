# HealthcareOS Backend

A beginner-friendly backend engineering project for **HealthcareOS**, a Bangladesh-first, multi-tenant healthcare operations SaaS.

> **Project status:** Initial repository / planning stage. This README describes the intended architecture and development plan; it does not claim that APIs, authentication, database migrations, or production controls have already been implemented.

## 1. Purpose

The backend will gradually replace frontend-only state and mock data with a real, secure API and persistent database. Development starts with synthetic data and a small, verifiable vertical slice before expanding into clinical and pharmacy workflows.

### MVP outcomes

- Secure user authentication and server-side authorization
- Organization, branch, and tenant membership management
- Patient directory, appointments, and clinic queue
- Clinical encounters, consultation notes, and prescription lifecycle
- Pharmacy inventory, dispensing, stock movements, and audit history
- Documented REST APIs, automated tests, and repeatable deployment

**Out of scope until validated:** premature microservices, autonomous AI diagnosis or prescribing, live GPS tracking, and integrations that are not essential to the pilot.

## 2. Intended technology stack

| Area | Planned choice | Why |
|---|---|---|
| Language | Java (supported LTS release) | Strong typing and mature backend ecosystem |
| Framework | Spring Boot | Application configuration, REST APIs, dependency injection |
| Build | Maven Wrapper (mvnw) | Reproducible builds across developer machines and CI |
| Database | PostgreSQL | Relational data, transactions, constraints, and integrity |
| Migrations | Flyway | Version-controlled, repeatable schema evolution |
| Persistence | Spring Data JPA / Hibernate | Data access and object-relational mapping |
| Security | Spring Security | Authentication and authorization |
| Validation | Jakarta Bean Validation | Validate API inputs at the boundary |
| API documentation | OpenAPI / Swagger UI | Shared frontend/backend API contract |
| Testing | JUnit, Spring Boot Test, Testcontainers where appropriate | Unit and real-PostgreSQL integration testing |
| Delivery | Docker and CI workflow | Consistent local, test, and deployment processes |

Versions should be selected together at project creation and recorded in the build file. Avoid upgrading dependencies independently without running the full test suite.

## 3. Architecture approach

Start with a **modular monolith**, not microservices. It keeps the first version easier to understand, run, test, and deploy while maintaining clear domain boundaries.

Initial package direction (final names may change after reviewing the generated project):

~~~text
src/main/java/.../
  config/         # application configuration
  common/         # shared error responses and utilities
  identity/       # users, credentials, sessions
  tenant/         # organizations, branches, memberships
  patient/        # patient directory and identifiers
  appointment/    # bookings and clinic queue
  clinical/       # encounters, notes, prescriptions, revisions
  pharmacy/       # medicine catalog, stock, dispensing
  audit/          # security and business audit events
~~~

Keep responsibilities clear:
- **Controller:** HTTP request/response boundary; no large business logic.
- **DTO:** explicit API request and response shapes.
- **Service:** business rules and transaction boundaries.
- **Repository:** database access.
- **Entity:** persistence model; do not expose JPA entities directly as public API responses.
- **Migration:** versioned database schema change.
- **Test:** proves behavior, including failures and authorization boundaries.

## 4. Beginner learning path

Complete the steps in order. Each step should include a short learning note explaining what was built and why.

1. **Verify the environment** — check Java, Git, IntelliJ IDEA, Maven Wrapper, and PostgreSQL.
2. **Learn Java essentials** — variables, types, conditions, loops, methods, classes, constructors, access modifiers, interfaces, collections, exceptions, and packages.
3. **Learn OOP through small exercises** — encapsulation, composition, inheritance where appropriate, interfaces, and dependency inversion.
4. **Create the Spring Boot application** — understand the project layout, dependencies, annotations, configuration, and application startup.
5. **Build a tiny REST API** — controller, DTO, service, validation, HTTP status codes, and error responses; initially use in-memory data only to learn the request flow.
6. **Connect PostgreSQL** — database/user setup, connection properties, SQL basics, primary/foreign keys, constraints, and transactions.
7. **Add Flyway and JPA** — versioned migrations, entity mapping, repositories, and integration tests against PostgreSQL.
8. **Establish API conventions** — /api/v1, DTOs, validation, pagination, OpenAPI, and standardized errors.
9. **Implement identity and tenant security** — authentication, sessions, permissions, tenant membership, and cross-tenant access tests.
10. **Build the MVP workflows** — patient directory, appointments, queue, encounters, prescriptions, pharmacy inventory, and dispensing.
11. **Harden and deploy** — automated tests, secrets management, logs/metrics, CI, backups, restoration rehearsal, staging, and release checks.

Do not skip environment verification or start with complex clinical endpoints before the fundamentals are working.

## 5. Data and security principles

Healthcare data is sensitive. Use **synthetic test data only** until privacy, security, clinical, and legal reviews approve any real-world pilot.

- Enforce authorization in the backend on every protected resource; frontend route guards are not security controls.
- Derive tenant access from authenticated identity and active membership; never trust a browser-supplied role or tenant ID by itself.
- Use least-privilege database credentials and keep secrets out of Git.
- Validate request bodies and constrain pagination and search inputs.
- Use database constraints and transactions for critical state changes.
- Keep finalized clinical records traceable through amendments/revisions instead of silently overwriting history.
- Record stock movements and audit high-impact actions; do not silently edit inventory balances.
- Prevent duplicate operations on retries where needed (for example, dispensing).
- Do not log passwords, session tokens, secrets, or unnecessary patient details.
- Do not describe mock medicine records as verified official data without reliable source evidence.
- Do not claim regulatory compliance without a qualified review and supporting evidence.

## 6. Local setup (to be completed as implementation progresses)

### Prerequisites

- Supported Java LTS JDK
- Git
- IntelliJ IDEA
- PostgreSQL
- A generated Spring Boot project with Maven Wrapper

### Verify Java

Run in PowerShell:

~~~powershell
java -version
javac -version
git --version
~~~

These commands only verify that the tools are available; they do not start the backend.

### Build and test

Once the Spring Boot project and Maven Wrapper have been committed, run from the directory containing mvnw.cmd:

~~~powershell
.\mvnw.cmd test
~~~

To start the app after database configuration and the first application code are in place:

~~~powershell
.\mvnw.cmd spring-boot:run
~~~

The exact startup instructions, required environment variables, database creation steps, and health endpoint will be updated after the initial project scaffold is committed. Do not add real passwords or API keys to this README.

## 7. Definition of a production-minded MVP

A feature is not complete just because it works in the UI. Before calling an MVP workflow complete, verify:

- Acceptance criteria and permissions are documented.
- Request validation and safe error handling are implemented.
- PostgreSQL schema changes are version-controlled with Flyway.
- Automated tests cover success, invalid input, unauthorized access, tenant isolation, and important failure cases.
- Critical multi-record updates are transactional and safe under retries/concurrency.
- API documentation matches actual behavior.
- Logs and audit records avoid exposing secrets and unnecessary health data.
- Staging smoke tests, backup/restore procedures, and operational ownership are documented.
- Clinical, privacy, security, and Bangladesh-specific legal/regulatory review is completed before any real-patient pilot.

## 8. Documentation to add as the project grows

- docs/architecture.md — module boundaries and architectural decisions
- docs/data-model.md — ERD, data dictionary, ownership and retention notes
- docs/security.md — threat model, authentication, authorization and tenant isolation
- docs/api-conventions.md — routes, DTOs, errors, pagination and versioning
- docs/runbooks/ — local setup, deployment, incident response and restore rehearsal

Create each document when its corresponding design has been reviewed; do not document planned behavior as already implemented.

## 9. Learning resources

- Java tutorials in Bangla: https://www.youtube.com/results?search_query=Java+programming+Bangla+beginner
- Spring Boot beginner tutorials in Bangla: https://www.youtube.com/results?search_query=Spring+Boot+Bangla+beginner
- PostgreSQL tutorials in Bangla: https://www.youtube.com/results?search_query=PostgreSQL+Bangla+tutorial
- Official Spring guides: https://spring.io/guides
- Official Spring Security guide: https://spring.io/guides/gs/securing-web/
- PostgreSQL documentation: https://www.postgresql.org/docs/
- Flyway documentation: https://documentation.red-gate.com/flyway
- Maven Wrapper documentation: https://maven.apache.org/wrapper/

## 10. Project links

- Backend repository: https://github.com/Atiksujon360/HealthcareOS-Backend
- Frontend repository: https://github.com/Atiksujon360/HealthcareOS

---

**Working rule:** understand the code before expanding it. Make small changes, run the relevant tests, inspect the diff, and commit working milestones. Use AI as a reviewer and tutor, not as a substitute for understanding security-critical behavior.
