# AI Website Testing Platform — Requirements

## 1. Project Overview

The AI Website Testing Platform is a web application that helps developers and QA testers test websites and web applications using artificial intelligence (AI) and browser automation.

Users create projects by providing a website URL and testing instructions, uploading requirements documents, or both.

AI analyzes the provided information and generates test cases containing ordered test steps. Users can review, edit, enable, or disable these test cases before execution.

The backend manages test dependencies and schedules ready-to-run test jobs through a Redis-backed queue. Playwright workers execute the tests in real browsers and return their results.

The platform stores test execution history, screenshots, logs, and browser traces to help users understand test failures.

The platform also supports team collaboration, allowing multiple users to work on the same project with different permissions.

## 2. Project Goals

The platform aims to:

- Reduce the need to write automated browser tests manually.
- Generate test cases from written instructions and uploaded requirements documents.
- Allow users to review and improve AI-generated tests.
- Execute tests automatically using Playwright.
- Support test dependencies and correct execution ordering.
- Process test jobs asynchronously using Redis and background workers.
- Provide clear results, screenshots, logs, and execution history.
- Support collaboration between developers and QA testers.
- Protect user accounts, project data, and testing credentials.
- Provide a consistent development environment using Docker.
- Automate builds, tests, and deployment workflows using GitHub Actions.

## 3. Technology Stack

### 3.1 Frontend

- React
- JavaScript
- Vite

The frontend provides interfaces for authentication, projects, team collaboration, test management, execution, and reports.

### 3.2 Backend

- Java
- Spring Boot
- Spring Security
- JWT authentication using HS256

The backend manages authentication, authorization, business logic, project data, test dependencies, and job scheduling.

### 3.3 AI Service

- Python
- FastAPI

The AI service processes testing instructions and requirements documents to generate structured test cases.

### 3.4 Browser Testing Worker

- Node.js
- Playwright

Workers execute browser tests and collect execution evidence.

### 3.5 Database, Queue, and Storage

- MySQL — relational application data.
- Redis — reliable background test job queue.
- MinIO — uploaded requirements documents and test artifacts.

### 3.6 Development and CI/CD

- Docker
- Docker Compose
- GitHub
- GitHub Actions

## 4. Users and Project Roles

### 4.1 Registered Users

Users can create accounts, log in, and create testing projects.

Each user can belong to multiple projects.

Project access is determined by project membership rather than a single global user role.

### 4.2 Project Roles

The platform supports four project-level roles defined by the `ProjectRole` enum.

**OWNER**
- Manages project settings.
- Invites and removes project members.
- Assigns and changes member roles.
- Manages requirements documents and test accounts.
- Generates, edits, and executes tests.
- Views test results and artifacts.
- Can delete the project.

**DEVELOPER**
- Views project information and requirements.
- Uploads and manages requirements documents.
- Configures test accounts.
- Generates, edits, and executes tests.
- Reviews results and artifacts.
- Cannot manage project members or delete the project.

**QA_TESTER**
- Views project information and requirements.
- Uploads and manages requirements documents.
- Configures test accounts.
- Generates, reviews, edits, and executes tests.
- Reviews results and artifacts.
- Cannot manage project members or delete the project.

**VIEWER**
- Views project information.
- Views requirements documents, test cases, execution history, and reports.
- Cannot create, edit, or execute tests.
- Cannot manage test accounts, project settings, or members.

### 4.3 Role Rules

- Roles are assigned per project.
- A user can have different roles in different projects.
- The project creator automatically becomes an OWNER.
- Every project must have at least one OWNER.
- The last OWNER cannot be removed or demoted.
- Project permissions must be enforced by the backend.
- A user must be an authorized project member to access private project resources.

## 5. Functional Requirements

### FR-01: User Registration and Authentication

- Users can register using an email address and password.
- Email addresses must be unique.
- Users can log in and log out.
- Passwords must be securely hashed using BCrypt or Argon2.
- Authentication must use Spring Security and JWT.
- JWT access tokens must be signed using HS256.
- JWT signing must use one securely generated secret key of at least 256 bits.
- JWT access tokens must have an expiration time.
- The backend must validate the JWT signature and expiration.
- Protected endpoints must require authentication.
- Invalid or expired tokens must be rejected.
- Logout must remove the client's active authentication credentials.

### FR-02: Project Management

- Authenticated users can create testing projects.
- Every project must have a name and website base URL.
- A project can contain a written description with testing instructions.
- A project can contain uploaded requirements documents.
- At least one description or requirements document must be available before AI test generation.
- Project owners can update project settings and delete projects.
- Authorized project members can view and work with projects according to their roles.
- The platform must record which user created each project.
- The project creator must automatically become an OWNER.

### FR-03: Team Collaboration

- Projects can have multiple members.
- Users can belong to multiple projects.
- Project owners can invite existing registered users to join a project.
- Project owners can assign one of the four project roles.
- Project owners can change member roles and remove members.
- Users can view projects in which they are members.
- Duplicate project memberships must be prevented.
- Every project must retain at least one OWNER.
- Project membership must be checked before granting access.
- Users must not access another project's private data without authorization.

### FR-04: Requirements Documents

- Authorized users can upload requirements documents.
- Supported formats are PDF, TXT, and Markdown.
- A project can contain multiple requirements documents.
- Uploaded files must be stored in MinIO.
- Document metadata must be stored in MySQL.
- The AI service can use uploaded document contents for test generation.
- Authorized users can view and remove documents.
- Uploaded files must be validated before processing.

### FR-05: Test Accounts

- Authorized users can configure optional accounts for the target website.
- A project can have multiple test accounts.
- Each test account includes a username and secure credential reference.
- Target website accounts are separate from platform user accounts.
- Test credentials must not be stored as plain text.
- Credentials must not be exposed in API responses or application logs.
- Tests requiring unavailable credentials must be marked SKIPPED or INCONCLUSIVE rather than PASSED.

### FR-06: AI Test Generation

- AI generates test cases from written testing instructions and uploaded documents.
- Users must not be required to write Playwright scripts.
- Each generated test case must include a title and description.
- Each test case must contain ordered test steps before execution.
- Generated steps must use supported browser actions.
- Authorized users can review and edit generated test cases.
- Test cases can be marked DRAFT, READY, or DISABLED.
- Test cases must support versioning.
- AI may suggest dependencies between test cases.
- The backend must validate generated test cases and dependencies before execution.

### FR-07: Supported Browser Actions

Version 1 supports the following `ActionType` values:

- `NAVIGATE` — open a webpage.
- `CLICK` — click an element.
- `FILL` — enter text into an input field.
- `ASSERT_VISIBLE` — verify that an element is visible.
- `ASSERT_TEXT` — verify an element's text.
- `ASSERT_URL` — verify the current page URL.

Test steps must execute in ascending sequence order.

Additional browser actions may be introduced in later versions.

### FR-08: Test Dependencies

- A test case can have zero or one prerequisite test case.
- A test case can be a prerequisite for multiple other test cases.
- A dependent test must wait until its prerequisite passes.
- If a prerequisite fails, dependent tests must be marked SKIPPED.
- A test case cannot depend on itself.
- Circular dependencies are not allowed.
- Dependencies must reference test cases within the same project.
- The backend must validate dependencies before execution.
- The backend must determine which test cases are ready to execute.

### FR-09: Test Execution

- Authorized users can start test runs.
- A test run represents the execution of selected test cases within a project.
- The backend coordinates test execution.
- Tests execute asynchronously in Playwright workers.
- Test steps within each test case execute sequentially.
- Failed tests must not prevent unrelated tests from running.
- Independent tests may run concurrently when isolation is safe.
- Execution concurrency must be configurable.
- Each test run must record its status and timestamps.
- The platform must support cancellation of test runs where practical.

### FR-10: Redis Job Queue

- Redis must be used for the background test execution queue.
- The backend must enqueue ready-to-run test jobs.
- Playwright workers must consume jobs from Redis.
- The queue must support reliable acknowledgement.
- Jobs must be recoverable after worker failures.
- Failed or interrupted jobs must follow a defined retry policy.
- Retries must not create duplicate test results.
- Tests with unfinished prerequisites must not be scheduled.
- The platform must track job progress and execution outcomes.

### FR-11: Playwright Workers

- A dedicated Node.js worker service must execute browser tests.
- Workers must retrieve jobs from Redis.
- Workers must execute test steps in order.
- Workers must handle browser errors and timeouts.
- Workers must report execution results to the backend.
- Workers must collect screenshots, logs, and traces when available.
- Workers must run with appropriate isolation and resource limits.
- The architecture must support adding more workers in the future.

### FR-12: Test Results and Reports

- Every test run must record its status.
- Each executed test case must produce a result.
- Test results must use the `ResultStatus` enum.
- Failed tests may include error messages.
- Users can view historical test runs.
- Users can inspect individual test results.
- Historical results must preserve the executed test case version or snapshot.
- Screenshots, traces, and logs must be stored in MinIO.
- Artifact metadata must be stored in MySQL.
- Artifacts must be accessible only to authorized project members.

### FR-13: Docker Environment

- Docker and Docker Compose are included in Version 1.
- The local environment must include MySQL, Redis, and MinIO.
- The frontend, backend, AI service, and worker must be runnable in containers.
- Persistent data must use Docker volumes where appropriate.
- Environment-specific configuration must be supported.
- Secrets must not be committed to GitHub.
- Developers must be able to start the required local services consistently.

### FR-14: GitHub Actions and CI/CD

- GitHub Actions must be used for CI/CD.
- CI workflows must run on pushes and pull requests.
- CI must build the frontend, backend, AI service, and worker.
- CI must execute available automated tests.
- CI should validate configuration where practical.
- Failed builds and tests must be visible in GitHub.
- CD workflows must support deployment to a configured environment.
- Deployment secrets must be stored securely.
- CI/CD workflows must expand as the application is implemented.

### FR-15: Authorization and Access Control

- Spring Security must protect backend API endpoints.
- The backend must authenticate requests using JWT access tokens.
- The backend must verify JWT signatures using the configured HS256 secret.
- The backend must identify the authenticated user from the validated JWT.
- Project access must be checked using `ProjectMember`.
- Project permissions must be determined by the `ProjectRole` enum.
- The backend must not trust roles or user IDs supplied by the frontend.
- Users must not access another project's resources by changing IDs in requests.
- OWNER permissions are required for member management and project deletion.
- DEVELOPER and QA_TESTER permissions allow test creation, editing, and execution.
- VIEWER permissions provide read-only access.
- Permission checks must apply to documents, test accounts, test cases, results, artifacts, and execution endpoints.

## 6. Domain Model and Enums

### 6.1 Domain Entities

Version 1 contains ten core domain entities:

1. `User`
2. `Project`
3. `ProjectMember`
4. `RequirementDocument`
5. `TestAccount`
6. `TestCase`
7. `TestStep`
8. `TestRun`
9. `TestResult`
10. `TestArtifact`

### 6.2 Enumerations

**ProjectRole**
- `OWNER`
- `DEVELOPER`
- `QA_TESTER`
- `VIEWER`

**TestCaseStatus**
- `DRAFT`
- `READY`
- `DISABLED`

**ActionType**
- `NAVIGATE`
- `CLICK`
- `FILL`
- `ASSERT_VISIBLE`
- `ASSERT_TEXT`
- `ASSERT_URL`

**RunStatus**
- `PENDING`
- `RUNNING`
- `COMPLETED`
- `FAILED`
- `CANCELLED`

**ResultStatus**
- `PASSED`
- `FAILED`
- `SKIPPED`
- `INCONCLUSIVE`

**ArtifactType**
- `SCREENSHOT`
- `TRACE`
- `LOG`

### 6.3 Enum Storage

- Java enums must be stored as string values in MySQL.
- Enum columns use VARCHAR with appropriate validation constraints.
- Unsupported enum values must be rejected.
- Separate database tables are not required for enums.

## 7. Non-Functional Requirements

### NFR-01: Authentication Security

- Spring Security must manage authentication and endpoint protection.
- User passwords must be hashed using BCrypt or Argon2.
- JWT access tokens must be signed using HS256.
- The HS256 secret key must contain at least 256 bits of cryptographically secure randomness.
- The signing secret must be stored securely and must never be committed to GitHub.
- Only the authentication backend should have access to the JWT signing secret.
- JWT access tokens must be short-lived, with an initial target lifetime of 15 minutes.
- The backend must validate token signatures, expiration, and configured issuer and audience claims.
- The backend must explicitly allow only the expected JWT signing algorithm.
- JWTs must be transmitted over HTTPS in deployed environments.
- Browser token storage must minimize exposure to cross-site scripting attacks.
- Secure, HttpOnly, SameSite cookies are preferred where suitable, with CSRF protection when cookie-based authentication is used.
- Authentication failures must not reveal sensitive implementation details.

### NFR-02: Authorization Security

- Project access must be based on membership and roles.
- Authorization must be enforced on every protected backend operation.
- Project permissions must be checked against trusted backend data.
- Users must not access resources belonging to unauthorized projects.
- The last project owner must not be removed or demoted.
- Authorization must protect files, results, and execution endpoints.
- Frontend permission controls must not replace backend authorization checks.

### NFR-03: Credential Protection

- Target website credentials must not be stored as plain text.
- Credentials must be encrypted or managed through a secure secret-storage mechanism.
- Database records must store secure references to credentials.
- Secrets must not appear in logs, reports, or API responses.
- Only authorized execution components may retrieve credentials when required.
- Credentials should be removed from browser execution environments after use.

### NFR-04: File Security

- Uploaded documents must be checked for supported file types.
- Upload size limits must be enforced.
- File names and paths must be handled safely.
- MinIO buckets must not be publicly accessible by default.
- Document and artifact access must require authorization.
- Uploaded documents must be treated as untrusted content.
- Document content must not override application security rules or AI service instructions.

### NFR-05: Browser Execution Security

- Playwright workers must execute tests in isolated environments.
- Browser sessions must have execution timeouts.
- Workers must have resource limits.
- User-provided website URLs must be validated.
- Workers must be prevented from accessing sensitive internal network services and cloud metadata endpoints.
- Browser execution must not expose backend or infrastructure credentials.
- Test artifacts must be handled as potentially sensitive data.

### NFR-06: Application Security

- Backend inputs must be validated.
- Database access must use parameterized queries or safe ORM mechanisms.
- The application must protect against common web vulnerabilities.
- CORS must be configured for trusted frontend origins.
- HTTPS must be used in deployed environments.
- Security-sensitive configuration must use environment variables or a secure secret manager.
- Secrets must never be committed to source control.
- Dependencies should be checked for known vulnerabilities during CI.

### NFR-07: Reliability

- Worker failures must not silently lose jobs.
- Failed jobs must follow a defined retry policy.
- Duplicate processing must not create inconsistent results.
- Test dependencies must always be respected.
- A failed test must not stop unrelated tests.
- Errors must be recorded when execution cannot complete.
- Historical results must remain available after test case changes.

### NFR-08: Performance

- The frontend must remain responsive during test execution.
- Test jobs must run asynchronously.
- Worker concurrency must be configurable.
- Parallel execution must not compromise correctness.
- Database queries should efficiently support projects, tests, and results.

### NFR-09: Maintainability

- Frontend, backend, AI service, and worker responsibilities must remain separated.
- Code must use understandable modules and naming conventions.
- The architecture and database must be documented.
- New test actions should be easy to introduce.
- Automated tests must be included in CI.
- Docker must support reproducible local development.

### NFR-10: Usability

- Users must be able to generate tests without writing Playwright scripts.
- Project roles and permissions must be understandable.
- Generated tests must be reviewable before execution.
- Test statuses must be clearly displayed.
- Failure messages should help users identify problems.
- Reports and artifacts must be easy to access.

### NFR-11: Scalability

- The architecture must support multiple workers.
- The queue must support independent job processing.
- The system must support increasing execution concurrency safely.
- MySQL and MinIO must support multiple users, projects, and historical runs.

### NFR-12: Logging and Monitoring

- Application errors must be logged.
- Failed login attempts and security-relevant permission changes should be recorded.
- Logs must not expose passwords, JWTs, signing secrets, or test account credentials.
- Worker errors and job failures must be traceable.
- CI must report build and test failures.

## 8. Version 1 Scope

### 8.1 Included Features

Version 1 includes:

- User registration and login.
- Spring Security authentication.
- JWT access tokens signed using HS256 with one secret key.
- Project-based authorization.
- Project creation and management.
- Team collaboration.
- Project roles and membership management.
- Written testing instructions.
- Requirements document uploads.
- Optional target website test accounts.
- AI-generated test cases and steps.
- Test case review, editing, and versioning.
- Test dependencies.
- Playwright browser execution.
- Redis-backed job queues.
- Dedicated Playwright workers.
- Controlled concurrency.
- Test results and execution history.
- Screenshots, logs, and traces.
- MySQL database.
- MinIO object storage.
- Docker and Docker Compose.
- GitHub Actions CI/CD.
- Core application, file, and worker security.

### 8.2 Features Planned for Later Versions

- Scheduled and recurring test runs.
- Advanced test analytics dashboards.
- Email notifications.
- Testing across multiple browser engines.
- Large-scale distributed execution with advanced test data isolation.
- Automatic creation of accounts on target websites.
- Advanced AI test maintenance and self-healing selectors.

Native mobile application testing and dedicated API testing are outside the project's current scope.

## 9. MVP Success Criteria

Version 1 is successful when:

1. Users can register and log in securely.
2. Spring Boot can issue and validate HS256-signed JWT access tokens.
3. A user can create a project and automatically become its OWNER.
4. An owner can add project members and assign roles.
5. Project permissions are enforced by the backend.
6. An authorized user can provide a website URL and testing instructions or documents.
7. AI can generate test cases with ordered steps.
8. Authorized users can review, edit, and enable test cases.
9. The backend can validate dependencies and determine execution order.
10. Redis can reliably queue ready-to-run test jobs.
11. Playwright workers can execute jobs and report results.
12. Users can review execution history and available artifacts.
13. Sensitive credentials and files are protected.
14. Docker Compose can start the required local services.
15. GitHub Actions can build and test the application automatically.
16. A configured CD workflow can deploy the application.

The complete testing workflow must operate without requiring users to manually write Playwright scripts.

## 10. Out of Scope

Version 1 does not include:

- Native Android or iOS application testing.
- Dedicated API testing as a separate product feature.
- Automatic website development or code generation.
- Full replacement of human QA review.
- Guaranteed detection of every website defect.

The platform focuses on AI-assisted browser testing for websites and web applications.