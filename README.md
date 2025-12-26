Here is a **clean, distilled list of Spring Boot best practices** that **senior/pro developers actually follow** in real production projects.
No bullshit, only what matters.

---

# 🧱 **1. Project Structure (Modular, Not Dumped Together)**

```
com.yourapp
 ├── config
 ├── controller
 ├── service
 ├── repository
 ├── entity
 ├── dto
 ├── exception
 ├── util
 └── validation
```

❌ Don’t mix controller/service/entity in the same package.
✔️ Keep modules small & clean.

---

# 📦 **2. Always Use DTOs**

Never expose JPA entities directly.

✔️ RequestDTO
✔️ ResponseDTO
✔️ Entity

**Mapper:** MapStruct (professional devs use it)

---

# 🧪 **3. Avoid Business Logic in Controller**

Controller = Validate + Delegate
Service = Business logic
Repository = DB logic

---

# 🪝 **4. Use GlobalExceptionHandler (ControllerAdvice)**

Custom exceptions:

```
ResourceNotFoundException  
BadRequestException  
BusinessException  
UnauthorizedException  
```

Return consistent error format:

```json
{
  "status": 404,
  "error": "Not Found",
  "message": "Brand not found"
}
```

---

# 🧵 **5. Avoid @Autowired Field Injection**

❌ `@Autowired private UserService service;`
✔️ Constructor-based DI:

```java
@RequiredArgsConstructor
public class UserController {
    private final UserService service;
}
```

---

# ⚙️ **6. Use @ConfigurationProperties for External Config**

Don't scatter `@Value` everywhere.

```java
@ConfigurationProperties(prefix = "app.storage")
public class StorageProperties {
    private String baseUrl;
}
```

---

# 🗄️ **7. Use Database Migrations (Mandatory)**

Professional devs never manually create DB tables.

✔️ Flyway
or
✔️ Liquibase

---

# 🔐 **8. Externalize Secrets Properly**

❌ NEVER put credentials in application.yml
✔️ Use environment variables
✔️ Use Vault / AWS Secrets Manager in production
✔️ Use `.env` only in local dev

---

# 📃 **9. Follow Logging Best Practices**

* Use `Slf4j`
* No Sysout
* Log meaningful context
* Use structured JSON logging in production

---

# 🏎️ **10. Use Caching Where Needed**

Spring Cache with Redis for heavy read endpoints.

```java
@Cacheable("brands")
public Brand getBrand(UUID id) {}
```

---

# 🔄 **11. Avoid N+1 Queries**

Use:

✔️ `@EntityGraph`
✔️ `JOIN FETCH`
✔️ Lazy loading smartly

---

# 🔥 **12. Use ResponseEntity Everywhere**

Return:

```java
return ResponseEntity.ok(dto);
```

Not raw objects.

---

# 🎯 **13. Keep “Fat Services, Thin Controllers”**

Business logic always lives in service, not controller, not repository.

---

# ⚡ **14. Use Records for Immutable DTOs (Java 17+)**

```java
public record BrandResponse(UUID id, String name) {}
```

---

# 🧩 **15. Write Integration Tests (Not Just Unit Tests)**

Use:

✔️ SpringBootTest
✔️ Testcontainers (pro devs use this for DB testing)

---

# 🔍 **16. Validate Inputs Using @Valid + Custom Validators**

Never trust client data.

Use:

* `@Valid`
* Custom annotation validators
* Central validation error handler

---

# 🔄 **17. REST API Standards**

* Use nouns, not verbs (`/brands`, not `/getBrand`)
* Return proper HTTP status codes
* Use pagination (`limit`, `offset`)
* Use `Location` header on POST create

---

# 🪪 **18. Security: Always Use Spring Security**

Even for internal services:

* JWT
* OAuth2 resource server
* RBAC

---

# 🌐 **19. Use WebClient Not RestTemplate (RestTemplate is deprecated)**

---

# 🧱 **20. Use Layered Configuration**

* `application.yml`
* `application-dev.yml`
* `application-prod.yml`

Active profile decides config.

---

Here’s a **clean, structured, senior-level guide** to testing, API versioning, and multi-environment deployments in Spring Boot — the way **pro backend developers** usually work.

---

# ✅ **1. Testing Best Practices in a Spring Boot Project**

## **A. Unit Tests (JUnit + Mockito) – *Test logic***

✔ Purpose: Test **business logic**, NOT HTTP endpoints or databases.
✔ Fastest tests → run on every commit.
✔ No Spring context unless needed.

### **What to test**

* Service methods
* Utility classes
* Helper functions
* Edge cases
* Exception flows

### **What not to test**

* Database
* API controllers
* External calls
* JSON serialization

### **Structure**

```
src/test/java/com/myapp/service/UserServiceTest.java
```

### **Pro Tip**

Use:

* **Mockito** → for mocking dependencies
* **AssertJ** → readable assertions
* **Testcontainers** only in integration tests, not unit tests

---

## **B. Integration Tests – *Test full application wiring***

✔ Purpose: Validate **Spring wiring**, controllers, DB, external services.
✔ Load Spring context once.
✔ Use **Testcontainers** for real DB.

### **Used for**

* Controllers + @WebMvcTest
* Services using real DB
* Repositories using real DB
* External API integrations

### **Example**

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerIT {
   @Autowired MockMvc mockMvc;
}
```

---

## **C. API Contract Testing – *Swagger → Validator*

Use to ensure your API always matches schema:

* Springdoc OpenAPI
* OpenAPI schema validation tests
* MockMvc + JSON schema validator

Used when working with FE or external clients.

---

## **D. Postman Collections – *Manual validation + smoke tests***

✔ Best for **manual test cycles**, QA, dev handoff.
✔ Useful for **integration testing** before PR merge.
✔ Use **Postman Environment Variables**:

* local
* dev
* stage
* prod

### **Automate Postman**

Use **Newman** (CLI runner):

```
newman run postman_collection.json -e environment.json
```

Integrate with:

* GitHub Actions
* GitLab CI
* Jenkins

---

## **E. End-to-End (E2E) Tests – *Full system test***

✔ Run only before release.
✔ Requires all services running.
✔ Usually done by QA or automation engineers.

---

# ✅ **2. When to Use What**

| Test Type             | Purpose                 | When to Run     | Who Uses It |
| --------------------- | ----------------------- | --------------- | ----------- |
| **Unit Tests**        | Validate business logic | On every commit | Devs        |
| **Integration Tests** | Validate wiring & DB    | On PR merge     | Devs        |
| **Postman Tests**     | API functional check    | Before merge    | Dev + QA    |
| **Newman CI Tests**   | Automated API tests     | CI pipeline     | CI/CD       |
| **E2E Tests**         | Validate whole system   | Before release  | QA          |
| **Smoke Tests**       | Quick sanity check      | After deploy    | DevOps      |

---

# ✅ **3. API Versioning Best Practices**

⭐ Standard: **URI versioning**

```
/api/v1/users
/api/v2/users
```

Why?
✔ Clean
✔ Easy FE migration
✔ Allows breaking changes

**Other ways** (less common):

* Header-based versioning
* Content negotiation (MIME versioning)

---

# ✅ **4. Multi-Environment Deployment Best Practices**

## **Environments**

Every professional setup has:

* **local**
* **dev**
* **qa**
* **stage**
* **prod**

## **Config Management**

Use:

### **application-*.yaml**

```
application.yaml
application-dev.yaml
application-prod.yaml
```

### **NEVER commit secrets**

Use:

* AWS Secrets Manager
* Hashicorp Vault
* Kubernetes Secrets
* Docker secrets

---

# ✅ **5. CI/CD Testing Strategy**

## **Pipeline Stages**

```
1. Build + Unit Tests
2. Integration Tests (Testcontainers)
3. API Tests using Newman
4. Build Docker image
5. Deploy to dev
6. Smoke tests
7. Promote to qa / stage / prod
```

---

# ✅ **6. Folder Structure for Tests (Pro Level)**

```
src/
 ├── main/java
 └── test/java
        ├── unit
        ├── integration
        ├── contract
        └── e2e
```

---

# 🔥 **7. What Pro Developers Actually Do**

✔ Write mandatory unit tests
✔ Write integration tests for critical endpoints
✔ Use Testcontainers for DB tests
✔ Use Postman + Newman for API test automation
✔ Maintain API versioning from Day 1
✔ Maintain proper multi-environment configs
✔ Fail PRs automatically if tests fail
✔ Automatically deploy to dev & run smoke tests

---

If you want, I can also generate:

✅ A complete **example test setup**
✅ A sample **GitHub Actions CI pipeline**
✅ A **Testcontainers example**
✅ A **Newman setup**
✅ A complete folder structure

Just tell me what you want next.


Here are **fast development practices used by top-tier backend engineers** to build high-quality Spring Boot systems *quickly*, without compromising stability.

These are based on what senior developers actually do in real teams.

---

# 🚀 **1. Start With a Clean Project Template (Don’t reinvent boilerplate)**

Every pro dev keeps a **starter template** with:

✔ Preconfigured logging
✔ Global exception handler
✔ Swagger/OpenAPI
✔ Common filters (tenant, correlation-id, auth)
✔ Basic structure (controller/service/repo)
✔ Security disabled for local
✔ Test setup (JUnit + Mockito + Testcontainers)
✔ Dockerfile + docker-compose
✔ CI pipeline sample

This saves **hours** every new project.

If you want, I can generate a ready-to-use template for you.

---

# 🚀 **2. Use Code Generators (Build endpoints in seconds)**

Stop writing DTOs, mappers, and controllers manually.

Use:

### ✔ Lombok

```
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
```

### ✔ MapStruct

Auto-generate mapper code.

### ✔ SpringDoc + OpenAPI

Write **OpenAPI first**, generate:

* DTOs
* Controllers
* Interfaces
* Postman collections

This is how teams build APIs super fast.

---

# 🚀 **3. Use Dev Profiles + Hot Reload**

Speed increases massively when local environment is optimized.

### ✔ Spring Boot DevTools

Auto-reload changes.

### ✔ Disable security on local

No need to authenticate every request.

### ✔ Use test DB locally (H2 or Postgres via Docker)

Run:

```
docker compose up -d
```

### ✔ Make "local" environment extremely fast

```
spring.profiles.active=local
```

---

# 🚀 **4. Use Minimal Controller → Service → Repository Structure**

Pro developers keep controllers extremely thin:

```java
return userService.create(request);
```

All logic is in service.

---

# 🚀 **5. Keep Feature Branches Short-Lived**

Best practice for speed:

✔ branch per feature
✔ PR within 1–3 days
✔ small commits
✔ automatic tests run on PR

*Small code reviews = fast development.*

---

# 🚀 **6. Use a Good API Testing Workflow**

Pro teams never test APIs manually every time.

### Use:

* Postman environment + collection
* Newman CLI
* Swagger UI during dev
* Integration tests for key endpoints

This makes validation **instant**.

---

# 🚀 **7. Run the Full Stack Locally (Fast integration)**

Use Docker Compose for services:

```
backend
db
cache
queue
file-storage
```

Run everything with:

```
docker compose up
```

Developers test features end-to-end in seconds.

---

# 🚀 **8. Use Code Snippets / Templates / Live Templates**

VSCode / IntelliJ live templates:

* controller skeleton
* exception template
* service template
* repository template
* standard API responses

Pro devs rarely “type” the same patterns repeatedly.

---

# 🚀 **9. Use Smart Commits**

Senior devs write short meaningful commits:

```
feat: add tenant filter
fix: null pointer on user update
refactor: split payment logic
```

Also use:

```
git stash
git rebase
git cherry-pick
```

This keeps repo clean → easier to move faster.

---

# 🚀 **10. Automate Everything (CI/CD)**

A fast team has:

✔ Linting
✔ Formatting (Spotless plugin)
✔ Unit tests
✔ Integration tests
✔ Newman API tests

Triggered **on every PR**.

Small mistakes are caught early → development becomes smooth.

---

# 🚀 **11. Keep Documentation Synchronized Automatically**

✔ Swagger auto-generation
✔ Postman auto-generation
✔ ADR (Architecture Decision Records) for changes
✔ README updated automatically

No manual docs → always up to date.

---

# 🚀 **12. Use Feature Toggles**

Allows releasing incomplete features safely.

* LaunchDarkly
* Unleash
* Simple DB toggle

You can deploy 80% of the feature while still working on 20%.

---

# 🚀 **13. Write Only Necessary Tests**

Don't over-test. For fast development:

✔ Unit tests for logic
✔ Integration tests for core flows
✔ Postman/Newman for endpoints

That's it.

Skip unnecessary mocks / edge-case tests in early stages.

---

# 🚀 **14. Follow the “90/10 rule” for exceptions**

Write a global exception handler once.

Then don't write try-catch everywhere — allow it to bubble up.

Cleaner + faster development.

---

# 🚀 **15. Reuse Existing Libraries**

Don’t create custom:

* Pagination
* DTO structures
* Response wrappers
* Tenant filters
* JWT utilities
* JSON utilities

Good senior developers **reuse** proven code.

---

# 🚀 **16. Use API First Approach**

This is the fastest workflow:

1. Write OpenAPI spec
2. Generate code skeleton
3. Implement service logic
4. Run tests
5. Merge PR

This eliminates rework.

---

# 🚀 **17. Use Tracing & Logging Early**

Add:

* Correlation-Id filter
* Tenant-Id filter
* Request logging
* Error logging

With logs in place, debugging is fast.

---

# 🚀 **18. Choose the Right Tools**

Pro teams standardize:

| Feature   | Best Tools                       |
| --------- | -------------------------------- |
| API Docs  | SpringDoc                        |
| Mapping   | MapStruct                        |
| Logging   | SLF4J + Logback                  |
| Build     | Maven                            |
| Tests     | JUnit + Mockito + Testcontainers |
| API Tests | Newman                           |
| Dev DB    | PostgreSQL / H2                  |
| CI/CD     | GitHub Actions                   |

---

# 🚀 **19. Invest in Clean Code Structure**

Fast development always comes from **predictable structure**:

```
controller/
service/
repository/
model/
dto/
config/
exception/
mapper/
```

Predictable structure = fast onboarding + fast coding.

---

# 🚀 **20. Follow “Working Code > Perfect Code” Rule**

Senior devs optimize **later**, not during early feature work.

Focus:
👉 Working logic
👉 Passing tests
👉 Stable API
👉 Clean enough

Refactor after merging the feature.

---
Below is the **proper and professional way** to handle errors when **creating indexes** in a Spring Boot application — whether your indexes are created:

✔ Automatically by Hibernate
✔ Manually via SQL
✔ Via Flyway/Liquibase migrations
✔ Programmatically in runtime

I’ll show **all cases**, so use the one that matches your setup.

---

# ✅ 1. **If you use Flyway (recommended by pro devs)**

Flyway automatically stops the migration if index creation fails.
But you can handle it gracefully with:

### **Option A: Use `IF NOT EXISTS` (best practice)**

PostgreSQL example:

```sql
CREATE INDEX IF NOT EXISTS idx_user_devices_user_id
    ON user_devices (user_id);
```

This avoids errors like:

* `duplicate key value`
* `index already exists`
* migration failure

### **Option B: Wrap in a transaction**

```sql
BEGIN;

CREATE INDEX IF NOT EXISTS idx_user_devices_token
    ON user_devices (device_token);

COMMIT;
```

---

# ✅ 2. **If you use Liquibase**

Liquibase lets you handle errors with `preConditions`:

```xml
<changeSet id="create-index" author="kunal">

    <preConditions onFail="MARK_RAN">
        <not>
            <indexExists indexName="idx_user_devices_token"/>
        </not>
    </preConditions>

    <createIndex indexName="idx_user_devices_token"
                 tableName="user_devices">
        <column name="device_token"/>
    </createIndex>

</changeSet>
```

👉 If index already exists → Liquibase **skips creation**, doesn’t fail deployment.

---

# ✅ 3. **If you use JPA/Hibernate indexes**

You define them in your entity:

```java
@Table(
    name = "user_devices",
    indexes = {
        @Index(name = "idx_user_devices_user", columnList = "user_id"),
        @Index(name = "idx_user_devices_token", columnList = "device_token")
    }
)
public class UserDevice { ... }
```

### How to handle errors?

Hibernate applies indexes at **schema creation time** (DDL mode).

If you see errors like:

```
ERROR: index "idx_user_devices_token" already exists
```

Then set the following:

### **Option A: Let Hibernate ignore failures**

In `application.properties`:

```
spring.jpa.properties.hibernate.hbm2ddl.halt_on_error=false
```

### **Option B: Disable Hibernate index management (best)**

Remove indexes from entity → create via Flyway/Liquibase instead.

**Pro devs NEVER rely on Hibernate for index creation in production**.

---

# ✅ 4. **If you want to create indexes programmatically at runtime**

For example, run on startup with error handling.

```java
@Component
public class IndexCreator {

    private final JdbcTemplate jdbc;

    public IndexCreator(JdbcTemplate jdbc) {
        this.jdbc = jdbc;
    }

    @PostConstruct
    public void createIndexes() {
        try {
            jdbc.execute("CREATE INDEX IF NOT EXISTS idx_user_devices_token ON user_devices(device_token)");
        } catch (Exception e) {
            // Log properly but don't crash the app
            log.error("Failed to create index idx_user_devices_token", e);
        }
    }
}
```

✔ App won’t crash
✔ Logs the cause
✔ Index created only if missing

---

# 🔥 Recommended Strategy (Used by Senior Devs)

### **DO NOT let Hibernate create indexes**

Reason:

* Non-idempotent
* Breaks on prod
* Hard to track changes

### **Use Flyway or Liquibase for index creation**

* Works in all environments
* Supports rollbacks
* Allows graceful error handling
* Cleaner version control

---

## Relationship modeling at code level
| # | Approach                            | How it’s Stored   | Ordering Support | Performance       | Complexity | When to Use                                                 | When NOT to Use               |
| - | ----------------------------------- | ----------------- | ---------------- | ----------------- | ---------- | ----------------------------------------------------------- | ----------------------------- |
| 1 | **IDs only (No JPA relation)**      | `UUID pageId`     | ✅ Excellent      | ✅ Excellent       | 🟢 Low     | CMS, Navigation, Menus, Multi-tenant systems, Microservices | If you need full object graph |
| 2 | **`@ManyToOne` (Uni)**              | FK + entity ref   | ⚠️ Limited       | ⚠️ Medium (lazy!) | 🟡 Medium  | Strong ownership (Invoice → Customer)                       | Trees, menus, deep graphs     |
| 3 | **Bidirectional `@OneToMany`**      | FK + collections  | ❌ Painful        | ❌ Poor            | 🔴 High    | Small aggregates only                                       | Navigation, ordering, APIs    |
| 4 | **Join Entity (Association table)** | Separate table    | ✅ Best           | ✅ Best            | 🟡 Medium  | Ordered relations, metadata, trees                          | Simple 1-1 ownership          |
| 5 | **`@ManyToMany`**                   | Hidden join table | ❌ None           | ❌ Poor            | 🔴 High    | Rare edge cases                                             | Ordered or metadata relations |
| 6 | **`@ElementCollection`**            | Child table       | ❌ No             | ⚠️ Medium         | 🟢 Low     | Tags, value lists                                           | Entities, navigation          |
| 7 | **Embedded (`@Embedded`)**          | Same table        | ❌ No             | ✅ Excellent       | 🟢 Low     | Grouping fields                                             | Relations                     |
| 8 | **DTO-only relations**              | API layer only    | ✅ Flexible       | ✅ Best            | 🟡 Medium  | API-first systems                                           | Heavy domain logic            |

