# The Twelve-Factor App

Methodology for building modern, scalable, maintainable SaaS applications. From [12factor.net](https://12factor.net).

> **License:** The Twelve-Factor App methodology is by Adam Wiggins, published under [CC BY 2.0](https://creativecommons.org/licenses/by/2.0/). This deck contains original educational summaries and original code examples inspired by the twelve factors.

---

## 1. Codebase — One Codebase, Many Deploys
**Advice:** One codebase tracked in version control, many deploys (dev, staging, production). If you have multiple codebases, it's a distributed system, not an app.
**Bad:** Copy-pasting code between repos for different environments. Different branches for dev vs prod with diverging code.
**Good:** One repo, environment differences handled through configuration. Same artifact deployed everywhere.
**Commit check:** Is all code for this app in one repository? Am I using branches for environments instead of configuration?

## 2. Dependencies — Explicitly Declare and Isolate
**Advice:** Never rely on system-wide packages. Declare all dependencies in a manifest (build.gradle, pom.xml, package.json) with exact versions.
**Bad:** "It works on my machine" because a system library is installed locally but not declared.
**Good:** `build.gradle` with all dependencies pinned. Gradle wrapper (`gradlew`) checked in so everyone uses the same build tool version.
**Commit check:** Are all dependencies explicitly declared? Would a fresh checkout build successfully?

## 3. Config — Store Config in the Environment
**Advice:** Configuration that varies between deploys (DB URLs, API keys, feature flags) must be stored in environment variables, not in code.
**Bad:**
```java
private static final String DB_URL = "jdbc:postgresql://prod-server:5432/mydb";
```
**Good:**
```java
@Value("${DATABASE_URL}") private String dbUrl;
// or: application.yml with ${DB_URL} references
```
**Commit check:** Am I hardcoding any environment-specific values? Could this config differ between deploys?

## 4. Backing Services — Treat as Attached Resources
**Advice:** Databases, message queues, SMTP servers, and caches should be treated as attached resources, accessed via URL/credentials in config. Swapping a local PostgreSQL for a managed one should require only a config change.
**Bad:** Code that assumes a specific database host or queue provider is always available at a hardcoded address.
**Good:** All backing services configured via URLs in environment. Local MySQL and Amazon RDS are interchangeable without code changes.
**Commit check:** Can I swap a backing service (database, cache, queue) by changing only configuration?

## 5. Build, Release, Run — Strict Separation
**Advice:** Strictly separate build (compile + dependencies), release (build + config), and run (launch process) stages. Releases are immutable and append-only — every release has a unique ID.
**Bad:** SSH into production and `git pull && gradle build` to deploy.
**Good:** CI/CD pipeline: build artifact once, combine with environment config, deploy immutable release with version tag.
**Commit check:** Can my app be built, released, and run as separate, repeatable steps?

## 6. Processes — Stateless and Share-Nothing
**Advice:** App processes should be stateless. Any data that needs to persist must be stored in a backing service (database, cache). Never rely on in-memory state or local filesystem between requests.
**Bad:**
```java
// In-memory session storage — lost on restart, breaks with multiple instances
private static Map<String, Session> sessions = new HashMap<>();
```
**Good:** Store sessions in Redis/database. Uploaded files go to object storage (S3), not local disk.
**Commit check:** Am I storing state in memory or local filesystem that should be in a backing service?

## 7. Port Binding — Export Services via Port
**Advice:** The app is self-contained and exports HTTP (or other protocols) by binding to a port. It does not rely on a running web server being injected at runtime.
**Good:** Spring Boot's embedded Tomcat — `java -jar app.jar` starts the app on port 8080. No external container needed.
**Commit check:** Can my app run standalone, or does it require an external server container?

## 8. Concurrency — Scale Out via the Process Model
**Advice:** Scale by running more instances of your app, not by making one instance bigger. Design for horizontal scaling — no shared memory between processes.
**Bad:** Single monolithic process handling everything with internal thread pool for scaling.
**Good:** Stateless app instances behind a load balancer. Background work handled by separate worker processes.
**Commit check:** Would my app work correctly if 5 instances ran simultaneously?

## 9. Disposability — Fast Startup, Graceful Shutdown
**Advice:** Processes should start fast and shut down gracefully. Handle SIGTERM, finish current requests, release resources, then exit.
**Bad:** 5-minute startup time. Abrupt kill on shutdown drops in-flight requests.
**Good:** Spring Boot graceful shutdown (`server.shutdown=graceful`). Startup in seconds. Connection draining on shutdown.
**Commit check:** Does my app handle shutdown signals gracefully? Does startup take more than a few seconds?

## 10. Dev/Prod Parity — Keep Environments Similar
**Advice:** Keep development, staging, and production as similar as possible. Same backing services, same OS, same versions. "Works on my machine" is a symptom of environment divergence.
**Bad:** Development on H2, production on PostgreSQL. Docker in prod, bare metal locally.
**Good:** Docker Compose for local development with the same PostgreSQL version as production.
**Commit check:** Am I introducing a development-only shortcut that doesn't exist in production?

## 11. Logs — Treat as Event Streams
**Advice:** An app should never concern itself with routing or storage of logs. Write to stdout and let the execution environment handle collection, routing, and archival.
**Bad:**
```java
FileWriter fw = new FileWriter("/var/log/app.log"); // app manages its own log files
```
**Good:**
```java
log.info("Order {} processed successfully", orderId); // writes to stdout
// Log aggregation (ELK, CloudWatch, Datadog) configured externally
```
**Commit check:** Am I writing logs to files, or to stdout where the platform can collect them?

## 12. Admin Processes — Run as One-Off Tasks
**Advice:** Administrative tasks (database migrations, console sessions, one-time scripts) should run in the same environment as the app, using the same codebase and config.
**Bad:** Running database migrations manually via SQL client connected to production.
**Good:** `./gradlew flywayMigrate` or Liquibase running as part of app startup. One-off scripts packaged with the app.
**Commit check:** Are my admin/maintenance tasks part of the codebase and runnable in any environment?