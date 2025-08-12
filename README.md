---

## 🧩 PHASE 1: Vanilla Spring MVC Project Setup with Open Liberty and Java 8

---

### 🔹 Objective

Set up the foundational structure for an enterprise banking API using **Vanilla Spring Web MVC** with **Java 8** and **Open Liberty** as the runtime. 
This setup should enforce best practices around **layered architecture**, **Java-based configuration**, and **annotation-driven** development — without using 
Spring Boot or XML configuration unless absolutely necessary.

---

### 🔸 Problem Statement

> You are tasked with initializing a Java 8-based backend application for the “BankEase” project — a secure, maintainable, and scalable banking REST API.
>
> This phase focuses solely on scaffolding the initial project skeleton using the **Spring Framework (not Spring Boot)** and deploying it on **Open Liberty** as a WAR file.
>
> The architecture must use:
>
> * **Java-based configuration (@Configuration, @EnableWebMvc, etc.)**
> * **Annotation-driven component scanning and servlet initialization**
> * **Strict folder layering for separation of concerns**
> * **Maven** as the build tool
> * **WAR packaging**
>
> The final deliverable should be a working project deployed on Open Liberty, accessible via a `/health` endpoint that returns a static health message (`{ "status": "UP" }`). 
> No business logic, database, or security is needed in this phase.

---

### ✅ Acceptance Criteria (Exhaustive & Verifiable)

#### 📁 Project Structure

* Maven project created using:

    * `groupId = com.bankease`
    * `artifactId = bankease-api`
    * `packaging = war`
    * `java.version = 1.8`
* Folder hierarchy follows:

  ```
  com.bankease
  ├── config
  ├── controller
  ├── service
  ├── repository
  ├── model
  ├── dto
  ├── exception
  └── util
  ```

#### 🧾 Maven Configuration (`pom.xml`)

* Dependencies added:

    * `spring-core`
    * `spring-webmvc`
    * `jackson-databind`
    * `servlet-api` (scope: provided)
    * `junit`
    * `slf4j-api` + `logback-classic`
* Build plugins:

    * `maven-compiler-plugin` with source/target `1.8`
    * `maven-war-plugin` for packaging WAR
* No Spring Boot starters allowed

#### 🛠️ Spring Configuration

* Java-based configuration only:

    * Create `AppConfig.java`:

        * `@Configuration`, `@ComponentScan("com.bankease")`, `@EnableWebMvc`
    * Create `WebAppInitializer.java`:

        * Extends `AbstractAnnotationConfigDispatcherServletInitializer`
        * Registers servlet, context config

#### 🌐 Controller

* Create `HealthController.java` in `controller` package:

    * Annotated with `@RestController`
    * Exposes `GET /health`
    * Returns:

      ```json
      {
        "status": "UP",
        "timestamp": "2025-06-18T10:00:00Z"
      }
      ```

#### 🧪 Testing

* Application deploys on Open Liberty (localhost:9080/bankease-api)
* `/health` returns static JSON response
* Logs show successful startup and request handling

#### 🧾 Open Liberty Configuration

* Create `src/main/liberty/config/server.xml`

    * Configure:

        * `httpPort=9080`
        * WAR deployment for `bankease-api.war`
* Ensure deployment via Liberty Maven plugin or manual WAR drop in `dropins` folder

#### ❌ Disallowed

* Spring Boot (`@SpringBootApplication`, auto-config, embedded Tomcat)
* XML-based Spring configs (except for Open Liberty `server.xml`)
* Packaging as JAR

#### 📚 Reference Links

* Spring MVC (Java config): [https://www.baeldung.com/spring-java-config](https://www.baeldung.com/spring-java-config)
* WebInitializer setup: [https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/WebApplicationInitializer.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/WebApplicationInitializer.html)
* Liberty guide: [https://openliberty.io/guides/getting-started.html](https://openliberty.io/guides/getting-started.html)
* WAR deployment: [https://openliberty.io/guides/deploying-war-file.html](https://openliberty.io/guides/deploying-war-file.html)



---

## 🧩 PHASE 2: MySQL Integration + JPA + Entity Mapping

---

### 🔹 Objective

Establish MySQL as the primary **persistent storage** for your banking API using **Spring Data JPA** with a **Java-based configuration**. Create foundational entities (`Customer`, `Account`, `Transaction`) and set up the data source, transaction manager, and JPA environment without XML. This will serve as the backbone for all future business operations.

---

### 🔸 Problem Statement

> You now need to persist and manage banking-related data in a relational database. Your goal is to integrate **MySQL** into the `bankease-api` project, configure the ORM layer using **Spring Data JPA**, and define the **domain model** through annotated entity classes.
>
> This includes:
>
> * Setting up a **local MySQL database**
> * Creating **Java-based configuration** for JPA + DataSource
> * Defining **3 entities**: `Customer`, `Account`, and `Transaction`
> * Ensuring **proper schema design**, with relationships and indexing
> * Enabling **connection pooling** via HikariCP for production-like behavior

This step will **not** implement business logic or endpoints. It only sets up the database connectivity and domain layer cleanly and correctly.

---

### ✅ Acceptance Criteria (Detailed & Verifiable)

#### 🛢️ MySQL Setup

* MySQL server is installed locally and running.
* Database created: `bankease_db`
* Credentials (can be default):

  * Username: `root`
  * Password: `password`
* Tables **must not be manually created** — they will be generated via JPA on app startup.

#### 📦 Dependencies in `pom.xml`

* Add:

  * `spring-data-jpa`
  * `hibernate-core`
  * `mysql-connector-java`
  * `HikariCP` (optional, recommended for real-world)
* Set MySQL connector version to 8.x.

#### 🧾 Java Configuration

* Create `DatabaseConfig.java` in `com.bankease.config`:

  * Annotated with `@Configuration` and `@EnableTransactionManagement`
  * Defines:

    * `DataSource` bean using HikariCP
    * `LocalContainerEntityManagerFactoryBean` bean
    * `JpaTransactionManager` bean
    * JPA properties (dialect, ddl-auto, etc.)

#### 🧑‍💼 Domain Entities

##### ✅ Entity 1: Customer

* Fields:

  * `id: Long` (PK)
  * `name: String`
  * `email: String (unique)`
  * `createdAt: Timestamp`
* Annotations: `@Entity`, `@Table`, `@Id`, `@GeneratedValue`, `@Column(unique = true)`

##### ✅ Entity 2: Account

* Fields:

  * `id: Long` (PK)
  * `accountNumber: String` (unique)
  * `accountType: Enum (SAVINGS, CURRENT)`
  * `balance: BigDecimal`
  * `customer: Customer` (ManyToOne)
  * `createdAt`, `updatedAt: Timestamp`
  * `version: int` (`@Version`)
* Relationships:

  * `@ManyToOne(fetch = FetchType.LAZY)` → Customer
  * `@JoinColumn(name = "customer_id")`

##### ✅ Entity 3: Transaction

* Fields:

  * `id: Long` (PK)
  * `account: Account` (ManyToOne)
  * `amount: BigDecimal`
  * `type: Enum (DEBIT, CREDIT)`
  * `remarks: String`
  * `timestamp: Instant`
* Relationships:

  * `@ManyToOne(fetch = FetchType.LAZY)` → Account
  * `@JoinColumn(name = "account_id")`

##### General Annotations:

* All timestamps use `@CreationTimestamp`, `@UpdateTimestamp`
* Use of `@Enumerated(EnumType.STRING)` for enums
* Lazy loading for `@ManyToOne` by default
* Lombok is **not allowed** for now — you must write getters/setters manually

#### 🛠️ Repository Layer

* Create `CustomerRepository`, `AccountRepository`, and `TransactionRepository`:

  * Extend `JpaRepository<Entity, Long>`
  * Use generics appropriately
* Annotate each interface with `@Repository`

#### 🔌 Application Startup Validation

* On starting the application:

  * No errors in logs
  * Console shows Hibernate schema generation (`hibernate.hbm2ddl.auto = update`)
  * MySQL `bankease_db` contains 3 tables
  * Foreign key constraints are present
  * Tables have auto-generated PKs

#### ✅ Connection Pooling

* HikariCP is configured with:

  * Minimum idle: 5
  * Maximum pool size: 10–20
  * Idle timeout: 600000ms
  * Connection timeout: 30000ms
* Pool settings included in configuration class using `HikariConfig`

#### 🧪 Verification Checklist

* Confirm via MySQL Workbench:

  * Tables are created automatically
  * Account table has FK → customer
  * Transaction table has FK → account
  * Email and account number are unique
* Logs show:

  * Successful database connection
  * Hibernate dialect selected for MySQL

#### ❌ Disallowed

* XML-based Hibernate or JPA configuration
* Hardcoding credentials inside classes (use `application.properties` or system env vars)
* Disabling FK constraints
* Spring Boot auto-config (still vanilla Spring)

---

### 📚 References

* [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)
* [Hibernate ORM 5.6+](https://hibernate.org/orm/documentation/)
* [MySQL Connector/J Docs](https://dev.mysql.com/doc/connector-j/8.0/en/)
* [Spring Transaction Management](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction)



---

## 🧩 PHASE 3: Health Check Endpoint

---

### 🔹 Objective

Implement a `/api/health` **REST endpoint** that exposes the runtime status of your application and verifies **connectivity with MySQL**. This endpoint is critical for monitoring, deployment pipelines, and alerting systems in production.

---

### 🔸 Problem Statement

> Your banking application now needs an operational health check endpoint to confirm that:
>
> 1. The API server is up and responding.
> 2. The MySQL database is reachable and functioning.
>
> This will help future deployment pipelines, load balancers, and monitoring agents (like AWS ELB, Kubernetes probes, etc.) determine whether the application is **healthy** or needs to be restarted.
>
> You must create a dedicated **controller** with a `GET /api/health` endpoint that performs:
>
> * A **static check** for API availability (always UP unless internal crash).
> * A **dynamic check** to test database connectivity via a lightweight query.
>
> Return a clear and well-structured **JSON response** with all necessary metadata and map the HTTP status code properly (200 for healthy, 503 for unhealthy).

---

### ✅ Acceptance Criteria (Thorough and Measurable)

#### 🧾 Controller Definition

* Create class: `HealthCheckController.java` under `com.bankease.controller`
* Annotated with `@RestController`
* Maps to `/api/health` via `@GetMapping("/api/health")`

#### ✅ Response Format

* Returns JSON object with:

```json
{
  "status": "UP",
  "database": "CONNECTED",
  "timestamp": "2025-07-25T10:30:45Z"
}
```

* If DB is disconnected or query fails:

```json
{
  "status": "DOWN",
  "database": "DISCONNECTED",
  "timestamp": "2025-07-25T10:30:45Z"
}
```

#### ⚙️ Technical Behavior

* Executes a DB check query (e.g., `SELECT 1`) via `JdbcTemplate`, `EntityManager`, or repository.
* Wrap DB call in try-catch and log exceptions.
* Dynamically set:

  * `"status"` → `"UP"` or `"DOWN"`
  * `"database"` → `"CONNECTED"` or `"DISCONNECTED"`
  * `"timestamp"` → current UTC time in ISO 8601 format (use `Instant.now()` + `DateTimeFormatter.ISO_INSTANT`)
* Uses proper HTTP response codes:

  * `200 OK` if healthy
  * `503 Service Unavailable` if database or app context is down

#### 🔒 Security Considerations

* Endpoint is **public** in this phase, but:

  * Add a TODO for securing it later using header-based authentication.
  * Mention that only internal systems will eventually access it.

#### 🧪 Logging Requirements

* Log each health check call at INFO level:

  ```
  [INFO] Health check invoked – App: UP, DB: CONNECTED, Time: 2025-07-25T10:30:45Z
  ```
* Log DB failure scenarios with full exception stack (at ERROR level).

#### 🧪 Test Scenarios

| Scenario                     | Expected Result                           |
| ---------------------------- | ----------------------------------------- |
| API called with DB connected | JSON `status: UP`, HTTP 200               |
| API called with DB shutdown  | JSON `status: DOWN`, HTTP 503             |
| API crashes                  | Liberty returns HTTP 500                  |
| Timestamp format             | ISO 8601 UTC (`Z` suffix)                 |
| Logs                         | Entry in app logs for each health request |

#### 📋 Tools to Test

* Postman or curl:

  ```
  curl http://localhost:9080/bankease-api/api/health
  ```
* MySQL server ON and OFF test cases
* Log file should reflect status

#### ❌ Disallowed

* Hardcoded status
* Returning HTML or XML
* Embedding database credentials in controller
* Using Spring Boot’s actuator (still on vanilla Spring)

---

### 📦 Deliverables

* Fully working `/api/health` endpoint with live DB ping
* JSON responses with correct structure
* Log entries for each call
* Verified behavior for both healthy and unhealthy DB status

---

### 📚 References

* [Spring REST Controllers](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-restcontroller)
* [Using JdbcTemplate for Raw Queries](https://www.baeldung.com/spring-jdbc-jdbctemplate)
* [DateTimeFormatter ISO\_INSTANT](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_INSTANT)



---

## 🧩 PHASE 4: CRUD Operations for Customer, Account, and Transaction

---

### 🔹 Objective

Implement complete, RESTful CRUD operations for **Customer**, **Account**, and **Transaction** entities using **vanilla Spring MVC + JPA**.
Ensure **layered architecture**, **DTO separation**, **validation**, **business rules**, **database constraints**, and **predictable response formats** — without yet implementing global exception handling (that will come in Phase 8).

---

### 🔸 Problem Statement

You now need to provide Create, Read, Update, and Delete operations for the three core banking entities in the API.
This phase introduces **business rules enforcement** and **input validation**, but **not centralized error handling** — each controller or service may temporarily handle exceptions inline.

You must:

* Build distinct **Controller**, **Service**, and **Repository** layers for each entity.
* Use **DTOs** for requests and responses (no direct JPA entity exposure).
* Apply **JSR-380 validation** to incoming requests.
* Enforce **business rules** (e.g., no deleting accounts with non-zero balance).
* Ensure **database constraints** are respected.
* Keep **API versioning** under `/api/v1`.
* Maintain **consistent JSON response shapes** for success and error cases (error format is defined now, but global handler comes in Phase 8).
* Implement **logging** for key actions at INFO and ERROR levels.

---

### ✅ Acceptance Criteria (Detailed & Verifiable)

---

#### 🧑‍💼 Domain DTOs & Validation

* **Customer DTOs**

  * `CustomerCreateDTO`: name (required, min/max length), email (required, unique, valid), phone (optional, regex if present)
  * `CustomerUpdateDTO`: only mutable fields
  * `CustomerResponseDTO`: id, name, email, phone, createdAt, updatedAt
* **Account DTOs**

  * `AccountCreateDTO`: customerId (required, must exist), accountNumber (required, unique, pattern `^[A-Za-z0-9]{10,20}$`), accountType (enum: SAVINGS, CURRENT), currency (default “INR”)
  * `AccountUpdateDTO`: accountType, status (ACTIVE, SUSPENDED, CLOSED)
  * `AccountResponseDTO`: id, customerId, accountNumber, accountType, status, currency, balance, createdAt, updatedAt
* **Transaction DTOs**

  * `TransactionCreateDTO`: accountId, type (DEBIT, CREDIT), amount (> 0), remarks
  * `TransactionResponseDTO`: id, txnReference, accountId, type, amount, balanceBefore, balanceAfter, createdAt
  * `TransferRequestDTO`: fromAccountNumber, toAccountNumber, amount (> 0), optional idempotencyKey
* **Validation Rules**

  * Standard JSR-380 annotations
  * Custom validator for account number pattern
  * Invalid inputs return:

    ```json
    {
      "timestamp": "2025-08-09T10:15:30Z",
      "status": 400,
      "error": "Bad Request",
      "message": "Validation failed",
      "details": [
        { "field": "email", "message": "must be a well-formed email address" }
      ]
    }
    ```

---

#### 🛠️ Repository Layer

* `CustomerRepository` with `findByEmail`
* `AccountRepository` with `findByAccountNumber`, `findByCustomerId`
* `TransactionRepository` with `findByTxnReference`, `findByAccountId`

---

#### 📦 Service Layer & Business Rules

* **CustomerService**

  * create → fail if email exists
  * delete → fail if customer has accounts
* **AccountService**

  * create → fail if customer not found or accountNumber exists
  * delete → fail if balance > 0 or pending transactions
* **TransactionService**

  * debit → fail if insufficient funds
  * credit → increase balance
  * transfer → atomic debit + credit; fail if same account; fail if insufficient funds
* Logging:

  * INFO: successful operations
  * ERROR: failures, with masked sensitive data

---

#### 🧾 Controller Layer

* Annotate with `@RestController` + `@RequestMapping("/api/v1")`
* Endpoints:

**Customers**

* POST `/customers` → 201 Created + Location header
* GET `/customers/{id}` → 200 or 404
* GET `/customers` → 200 list
* PUT `/customers/{id}` → 200 updated or 404
* DELETE `/customers/{id}` → 204 or 409

**Accounts**

* POST `/accounts` → validates customer exists
* GET `/accounts/{id}` → 200 or 404
* GET `/accounts?customerId=` → list
* PUT `/accounts/{id}` → update type/status only
* DELETE `/accounts/{id}` → 204 or 409 if balance > 0

**Transactions**

* POST `/transactions` → debit/credit
* GET `/transactions/{id}` → 200 or 404
* GET `/transactions?accountId=` → list
* POST `/transactions/transfer` → atomic debit+credit

---

#### 🛢️ Database Constraints

* `customers.email` UNIQUE
* `accounts.account_number` UNIQUE
* FK: `accounts.customer_id → customers.id`
* FK: `transactions.account_id → accounts.id`
* Monetary fields: `DECIMAL(18,2)`
* `@Version` in Account for optimistic locking

---

#### 🔌 Error Handling (Phase 4 scope)

* No global exception handler yet
* Each controller/method must:

  * Catch known business rule violations and return correct status code + JSON
  * Allow Spring’s default error handling for unexpected errors
* Example temporary inline error return:

  ```java
  if (balance.compareTo(amount) < 0) {
      return ResponseEntity.status(HttpStatus.BAD_REQUEST)
          .body(Map.of("error", "Insufficient funds"));
  }
  ```

---

#### 📤 Response Format Consistency

* All timestamps in ISO 8601 UTC
* Monetary values with 2 decimal places
* Example (GET account):

  ```json
  {
    "id": 101,
    "customerId": 1,
    "accountNumber": "ABC1234567",
    "accountType": "SAVINGS",
    "status": "ACTIVE",
    "currency": "INR",
    "balance": 12000.50,
    "createdAt": "2025-08-09T10:15:30Z",
    "updatedAt": "2025-08-09T10:15:30Z"
  }
  ```

---

#### 🧪 Functional Verification Points

* ✅ Cannot create duplicate customer email
* ✅ Cannot create account for non-existing customer
* ✅ Debit fails if insufficient funds
* ✅ Transfer rolls back fully if debit fails
* ✅ Cannot delete customer with linked accounts
* ✅ Cannot delete account with balance > 0
* ✅ GET endpoints return correct DTOs and codes

---

### ❌ Disallowed

* Direct entity exposure in responses
* Lombok usage
* Balance updates via Account PUT
* Skipping validation rules
* Inconsistent JSON response shapes

---

### 📚 References

* [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)
* [Bean Validation 2.0 (JSR-380)](https://beanvalidation.org/2.0/)
* [RESTful API Design — RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231)
* [Spring MVC Controller Best Practices](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-controller)
* [Spring Transaction Management](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction)




---
# 🧩 PHASE 9: Transaction Management, Locking, and Atomicity — Specification

---

## 🔹 Objective

Define precise **transaction boundaries**, **isolation levels**, **locking strategies**, **atomicity rules**, and **retry/rollback policies** for all database-related operations in the **BankEase API**.  
Goal: guarantee **data consistency**, **correctness**, and **reliability** under concurrent workloads while maintaining optimal performance.

This phase is **specification-only** — implementation of `@Transactional`, lock hints, retry handling, and exception mappings will be done after **Phase 8** (Global Exception Handling).

---

## 🔸 Problem Statement

Without strict transaction and locking specifications:

- Concurrent writes can cause **lost updates**.
- Concurrent reads can cause **dirty** or **non-repeatable reads**.
- Multi-step operations can leave **partial commits**.
- Phantom records may appear mid-transaction.
- Deadlocks may occur from inconsistent locking.
- Performance may degrade from excessive isolation levels or long-held locks.

---

## ✅ Requirements

### 1️⃣ Transaction Boundaries

- **Service layer** is the only place transactions are started.
- **One HTTP request = one transaction scope** (commit or rollback).
- No long-running transactions — avoid holding DB locks during:
  - File uploads
  - External API calls
  - Waiting for user input

---

### 2️⃣ Service-layer Transaction Policy (per operation mapping)

| Service Method                                   | Propagation   | Isolation Level                                              | readOnly | Timeout | Retryable? | Notes                                                                 |
| ------------------------------------------------ | ------------- | ------------------------------------------------------------ | -------- | ------- | ---------- | --------------------------------------------------------------------- |
| CustomerService.create()                         | REQUIRED      | READ_COMMITTED                                               | false    | 30s     | no         | Create with unique email check                                       |
| CustomerService.update()                         | REQUIRED      | READ_COMMITTED                                               | false    | 30s     | no         | Immutable fields rejected                                             |
| AccountService.create()                          | REQUIRED      | READ_COMMITTED                                               | false    | 30s     | no         |                                                                       |
| AccountService.getById()                         | SUPPORTS      | READ_COMMITTED                                               | true     | 5s      | no         |                                                                       |
| AccountService.updateStatus()                    | REQUIRED      | READ_COMMITTED                                               | false    | 30s     | no         |                                                                       |
| TransactionService.create() (credit/debit)       | REQUIRED      | READ_COMMITTED OR REPEATABLE_READ (for balance check)        | false    | 30s     | maybe      | Use REPEATABLE_READ if reading balance before update                  |
| TransactionService.transfer() (debit + credit)   | REQUIRED      | REPEATABLE_READ OR SERIALIZABLE (absolute safety)            | false    | 60s     | yes        | Must be atomic; REPEATABLE_READ recommended for performance           |
| ReportingService.generateReport()                | NOT_SUPPORTED | READ_COMMITTED                                               | true     | 300s    | no         | Read-only; prefer snapshot DB or replica                              |

**Notes:**

- `REPEATABLE_READ` prevents non-repeatable reads for transfers.
- `SERIALIZABLE` is safest but may cause contention — only for critical ledger ops.
- Always set `readOnly=true` for read-only methods.
- Avoid long timeouts unless report generation is heavy.

---

### 3️⃣ Isolation Level Rules

| Operation Type                            | Isolation Level    | Reason                                          |
| ----------------------------------------- | ------------------ | ----------------------------------------------- |
| Simple CRUD                               | `READ_COMMITTED`   | Prevent dirty reads; default in most RDBMS      |
| Fund transfer / multi-step ledger ops     | `REPEATABLE_READ`  | Prevents non-repeatable reads on balance checks |
| Month-end ledger closing                  | `SERIALIZABLE`     | Prevents phantom reads                          |
| Reports / dashboards                      | `READ_COMMITTED` or `READ_ONLY` | Performance-focused                             |
| Batch processing                          | `READ_COMMITTED`   | Minimizes contention                            |

**Disallowed:** `READ_UNCOMMITTED`

---

### 4️⃣ Locking Strategy

- **Primary:** Optimistic locking (`@Version` column in entity) for most cases.
- **Fallback:** Pessimistic locking (`SELECT ... FOR UPDATE`) for high-contention accounts.

#### **Optimistic Locking (default)**

- Add `@Version` field to entities (`long` or `int`).
- Ensure Hibernate maps the version column.
- Handle `OptimisticLockException` → map to HTTP **409 Conflict** in Phase 8.
- Retry policy:
  - **Policy A (recommended):** For idempotent operations (e.g., transfer with idempotency key), retry up to **3 times** with exponential backoff (100ms → 200ms → 400ms).
  - **Policy B:** For non-idempotent ops, no retry — return **409** to client.
  - **Policy C:** Use Spring Retry library OR manual retry loop.

#### **Pessimistic Locking (targeted)**

- Use `@Lock(LockModeType.PESSIMISTIC_WRITE)` or native `FOR UPDATE` queries.
- Apply only to:
  - Very high concurrency accounts (hot wallets).
  - Operations where retries are unacceptable.
- Acquire → update → commit quickly.
- Avoid for large table scans or read-heavy queries.

---

### 5️⃣ Lock Ordering Policy

- Always lock multiple accounts in **ascending account ID** order.
- Apply same policy across microservices in Phase 14.
- Test: simulate two opposite transfers between same accounts to ensure no deadlock.

Example (ASCII):

```
transfer(from=A, to=B):
  if A.id < B.id:
     lock A
     lock B
  else:
     lock B
     lock A
  perform debit/credit
```

---

### 6️⃣ Atomicity Rules

- Multi-step operations succeed **entirely** or fail **entirely**.
- If any step fails → rollback everything.
- Retryable failures → rollback and retry if idempotent.

---

### 7️⃣ Rollback Policy

Rollback **on**:

- Runtime exceptions
- Declared checked business exceptions

Do **not rollback on**:

- Validation failures before DB access
- No-op idempotent retries

---

### 8️⃣ Propagation Rules

| Scenario                                          | Propagation Type |
| ------------------------------------------------- | ---------------- |
| Service A calls Service B, must share transaction | `REQUIRED`       |
| Service A calls Service B, must run independently | `REQUIRES_NEW`   |
| Service A calls Service B, transaction optional   | `SUPPORTS`       |
| Read-only method in transaction                   | `NOT_SUPPORTED`  |

---

## 📤 Response Requirements

- Rollback due to business violation → **409 Conflict**
- Rollback due to system error → **500 Internal Server Error**
- Rollback due to missing resource → **404 Not Found**

Example error:

```json
{
  "timestamp": "2025-08-13T09:15:30Z",
  "status": 409,
  "error": "Conflict",
  "message": "Insufficient funds for debit operation",
  "transactionId": "0f3a9a6d-87f2-4c9d-9321-2141a90a06ef"
}
```

---

## 🧪 Acceptance Criteria

1. Two concurrent withdrawals never cause negative balance.
2. Fund transfer rolls back entirely if credit fails after debit.
3. Pessimistic locks prevent concurrent writes to the same account.
4. `REPEATABLE_READ` prevents mid-transaction balance change.
5. Read-only reports don’t block writers.

---

## ⚠ Disallowed

- `READ_UNCOMMITTED` isolation.
- Long-lived locks during external calls.
- Transactions starting in controller layer.
- Locking whole tables without explicit approval.
- Nesting `REQUIRES_NEW` calls recklessly.

---

## 🔄 Alternatives (OR’d for choice later)

- Isolation: default to `READ_COMMITTED` **OR** increase to `REPEATABLE_READ`.
- Locking: optimistic everywhere **OR** mix with pessimistic for hot accounts.
- Rollback: all exceptions **OR** only business-critical exceptions.

---

## 🖼 Diagrams

**Transaction Lifecycle**

```
[API Request]
    |
[Controller]
    |
[Service Layer] <-- Transaction starts
    |
[Repository calls]
    |
[Commit or Rollback]
    |
[Response to Client]
```

**Lock Ordering**

```
Account IDs: 101, 202
Lock smaller ID first → then larger
Perform business logic
Commit and release locks
```

---

## 📚 References

- [Spring Transaction Management](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction)
- [JPA Locking](https://jakarta.ee/specifications/persistence/3.0/jakarta-persistence-spec-3.0.html#locking)
- [Isolation Level Patterns](https://martinfowler.com/articles/patterns-of-distributed-systems/isolation-levels.html)
- [Deadlock Prevention](https://docs.oracle.com/cd/B28359_01/server.111/b28318/consist.htm)

---



# 🧩 PHASE 5: Pagination, Filtering & Sorting

(Finalized detailed content from our earlier discussion with 3 OR'd options)


# 🧩 PHASE 6: Logging with SLF4J + Log4j

(Finalized detailed content with profile-based config, log4j.xml usage, and disallowed items)


# 🧩 PHASE 7: Bean Validation + Custom Annotations

(Finalized detailed, thorough spec including validation rules, groups, i18n, pitfalls, and diagrams)


# 🧩 PHASE 8: Global Exception Handling

(Finalized detailed spec covering exception-to-HTTP mapping, standard error response JSON, and disallowed practices)


# 🧩 PHASE 9: Transaction Management, Locking, and Atomicity

(Finalized detailed Phase 9 text from our last response)
