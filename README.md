# ShareHub

ShareHub is a community resource-sharing web application. Donors can publish useful items, and recipients can discover available donations and submit or cancel requests. The project is built with Java 17 and Spring Boot and includes both server-rendered web pages and a session-based JSON API.

Explore the repository visually with [GitDiagram](https://gitdiagram.com/nahul100/sharehub).

## The problem this project addresses

Useful items can be available in a community while people nearby still struggle to find them. ShareHub models a simple way to connect those sides: donors list available food, clothing, books, household goods, or other resources; recipients browse the listings and request items. Donation ownership, availability, and request status are represented in the application rather than coordinated through disconnected posts or messages.

This describes the problem the project is designed to solve; it is not a claim that the application is currently deployed for a live community.

## Features

- Donor and recipient accounts, registration, login, and logout.
- BCrypt password hashing and role-based access controls.
- Session-authenticated Thymeleaf pages and JSON API endpoints.
- Separate dashboard and community donation listings: dashboard-created donations appear on the creator's dashboard, while `/donations/new` listings appear on the community Donations page.
- Donor listing editing, image upload, availability management, and dashboard listing history.
- Recipient donation discovery, request submission, request history, and cancellation of pending requests. Dashboard discovery and the general Donations page show their respective listing origins.
- Optional Gemini-assisted donation drafts and natural-language donation search with local relevance ranking.
- PostgreSQL persistence for users, donations, donation images, and resource requests.
- Flyway-managed SQL migrations and Hibernate schema validation in the default application configuration.
- OpenAPI documentation and Docker Compose development environment.

## Technology

| Area | Technology |
|---|---|
| Language | Java 17 |
| Framework | Spring Boot 3.4.7 |
| Web | Spring MVC, Thymeleaf |
| Security | Spring Security 6, BCrypt, HTTP session cookies |
| Persistence | Spring Data JPA, Hibernate |
| Database | PostgreSQL 15 in Docker Compose |
| Schema changes | Flyway |
| API documentation | springdoc OpenAPI 2.2.0 |
| AI assistance | GitHub Copilot |
| Build and tests | Maven Wrapper, JUnit and Spring Boot Test |
| Containerization | Multi-stage Docker build, Docker Compose |


## AI features

The donation form can use Google Gemini to draft a title, description, and category from donor notes and suggest listing details that may be missing. Recipients can search in natural language from the dashboard or Donations page. Gemini interprets the search into keywords and optional category/location hints; ShareHub matches and ranks listings locally using text and category comparisons, not embeddings or a vector database. Donation listings, account data, and recipient profile data are not sent to the AI provider. AI suggestions are editable and are never published automatically.

| Variable | Default | Purpose |
|---|---|---|
| `GEMINI_API_KEY` | unset | Google AI Studio API key; enables Gemini features |
| `GEMINI_MODEL` | `gemini-3.8-flash` | Gemini model ID |
| `GEMINI_FALLBACK_MODEL` | `gemini-3.5-flash-lite` | Secondary Gemini model used when the primary model exhausts retries with HTTP 503; set blank to disable |
| `GEMINI_API_ENDPOINT` | Google Gemini OpenAI-compatible chat completions URL | Optional API endpoint override |

Transient Gemini failures (HTTP 429/5xx and network errors) are retried up to three total attempts. Normal backoff waits at least 2 seconds before attempt 2 and 4–5 seconds before attempt 3. An HTTP 503 immediately routes the next attempt to the configured fallback model without waiting; if that attempt also gets 503, the final retry stays on the fallback model and uses normal backoff.


## Database migrations

Flyway migrations are in `src/main/resources/db/migration`:

- `V1`: users
- `V2`: donations
- `V3`: resource requests
- `V4`: donation images
- `V5`: donation origin, defaulting existing records to the dashboard

When running with the default application configuration, Flyway manages schema changes and Hibernate validates the resulting schema at startup.

## Build and tests

Build the application without running tests:

```bash
./mvnw package -DskipTests
```

Windows:

```powershell
.\mvnw.cmd package -DskipTests
```

Run the test suite:

```bash
./mvnw test
```

Windows:

```powershell
.\mvnw.cmd test
```

The repository includes service tests for donation and resource-request behavior. The Spring application-context test connects to PostgreSQL and runs Flyway, so a reachable database with matching configuration is required for the full test suite.

## Production engineering lessons from this project

Building ShareHub has given me practical experience with production-oriented concerns:

- **Configuration by environment:** database connection values and the server port can be supplied by the deployment environment instead of being tied to one machine.
- **Database change management:** versioned Flyway migrations provide an auditable schema history; Hibernate schema validation can catch a mismatch at startup.
- **Authentication and authorization:** BCrypt password hashing, session-cookie settings, and donor/recipient permissions protect account and listing workflows.
- **Containerized delivery:** a multi-stage Dockerfile separates the Java build image from the smaller runtime image; Compose coordinates the app and database and waits for PostgreSQL's health check.
- **Operational data and startup behavior:** a named database volume preserves local data across container restarts, while database availability and credentials are essential to application startup.
- **Testing behavior, not just compilation:** service tests cover core donation and request rules, and the database-backed context test exercises Flyway and application wiring.
- **Deployment trade-offs:** secure cookies need HTTPS, local Compose credentials must be replaced, and running multiple app instances requires shared session storage and operational setup such as monitoring and database backups.

This project is a learning and portfolio project, not a claim of a fully production-hardened service. The lessons above reflect concrete implementation choices and deployment gaps identified while building it.

## Project layout

Explore the repository structure [Gitingest](https://gitingest.com/nahul100/ShareHub)

```
