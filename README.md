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

## 🧩 PHASE 5: Pagination, Filtering & Sorting

---

### 🔹 Objective

Implement **paginated, filterable, and sortable** list retrieval for `Customer`, `Account`, and `Transaction` endpoints.
Provide clients with **clear, predictable** metadata for navigation via **one of three supported approaches**.

---

### 🔸 Problem Statement

Currently, list endpoints return **entire datasets** with no limits, making them inefficient for large banking datasets and risky for performance.

We must:

* Implement **pagination** (page/size params) with default limits and caps.
* Implement **sorting** (multi-field, ASC/DESC) and **filtering** (per-entity relevant fields).
* Provide **metadata** about the dataset and navigation options in **one of three ways**:

  1. **(Option A)** Embed metadata inside the JSON **response body** (recommended standard).
  2. **(Option B)** Provide metadata only via **HTTP response headers** (`Link`, `X-Total-Count`, etc.).
  3. **(Option C)** Use a **wrapper object** that contains both the `content` list and a `page` object with metadata.

---

### ✅ Acceptance Criteria (Detailed & Verifiable)

#### 📥 Query Parameters (Standardized for all list endpoints)

* `page` → 0-based page index (default `0`)
* `size` → number of records per page (default `20`, max `100`)
* `sort` → comma-separated field + direction (`name,asc` or `createdAt,desc`)
* Filtering params per entity:

  * **Customers**: `name`, `email`, `createdAfter`, `createdBefore`
  * **Accounts**: `customerId`, `accountType`, `status`
  * **Transactions**: `accountId`, `type`, `dateFrom`, `dateTo`

---

#### 📤 Response Format Options

**Option A — Metadata inside Response Body** *(Standard Approach)*

```json
{
  "content": [
    {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 52,
  "totalPages": 3,
  "sort": [
    { "field": "name", "direction": "ASC" }
  ]
}
```

**Option B — Metadata in Response Headers** *(Lightweight)*
**Headers:**

```
X-Total-Count: 52
X-Total-Pages: 3
X-Page-Number: 0
X-Page-Size: 20
Link: <https://api.bank.com/customers?page=1&size=20>; rel="next",
      <https://api.bank.com/customers?page=2&size=20>; rel="last"
```

**Body:**

```json
[
  { "id": 1, "name": "John Doe", "email": "john@example.com" }
]
```

**Option C — Wrapper with `content` + `page` object** *(Clean & Extensible)*

```json
{
  "content": [
    {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com"
    }
  ],
  "page": {
    "page": 0,
    "size": 20,
    "totalElements": 52,
    "totalPages": 3,
    "last": false
  }
}
```

---

#### 🛠 Implementation Details

* Use **Spring Data JPA `Pageable` and `Page<T>`** for pagination mechanics.
* Support **multi-field sorting** (`sort=name,asc&sort=createdAt,desc`).
* Validate `size` limit to prevent abuse.
* Add DB **indexes** for frequently filtered fields.
* For filtering, use:

  * **Dynamic JPQL** with `@Query`, or
  * **Criteria API / Specifications** for flexibility.

---

#### 🔌 Error Handling

* Invalid page/size → HTTP 400 with `"Page index must be non-negative"`.
* Invalid sort field → HTTP 400 with `"Invalid sort field: xyz"`.

---

#### 📚 References

* [Spring Data JPA Pagination & Sorting](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.paging-and-sorting)
* [RFC 8288 - Web Linking (Link Header)](https://datatracker.ietf.org/doc/html/rfc8288)
* [REST API Pagination Best Practices](https://developer.okta.com/blog/2021/02/09/pagination-in-rest-api)
* [Spring Data JPA Specifications](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#specifications)

---

### 🔍 Diagram — Query Flow for Paginated Endpoint

```plaintext
Client Request (page, size, sort, filters)
       |
       v
Controller (parse & validate params)
       |
       v
Service (build Pageable + filter criteria)
       |
       v
Repository (Spring Data JPA query)
       |
       v
Database (apply LIMIT/OFFSET + WHERE)
       |
       v
Page<T> Result
       |
       v
Controller maps to chosen response format (A, B, or C)
```



---

## 🧩 PHASE 6: Logging with SLF4J + Log4j 1.x

---

### 🔹 Objective

Implement **centralized, structured, and environment-aware logging** across all layers of the BankEase API using **SLF4J** as the API and **Log4j 1.x** as the backend via the `slf4j-log4j12` bridge.
Logs should be **consistent, traceable via correlation IDs, secure against sensitive data leaks, and configurable per environment**.

---

### 🔸 Problem Statement

Currently, the application either lacks logging or uses ad-hoc print statements, which is unacceptable for enterprise-grade banking APIs because:

* Debugging production issues is difficult without consistent, rich logs.
* Auditors require sensitive operations to be logged (without exposing PII).
* Developers need **correlation IDs** to trace requests across services.
* Different environments require different logging verbosity.

We must:

1. **Remove all `System.out.println()`** and replace with SLF4J logging.
2. Configure **Log4j 1.x** for structured patterns, rolling logs, and async logging in production.
3. Define **log levels per environment** (`DEBUG` in dev, `INFO` in prod).
4. Enforce **PII masking** for sensitive fields.
5. Ensure **MDC correlation IDs** are added for all requests.

---

### ✅ Acceptance Criteria (Detailed & Verifiable)

#### 📦 Dependencies

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.36</version>
</dependency>
```

*(SLF4J routes logs to Log4j 1.x under the hood.)*

---

#### 📝 Environment-Specific Log4j Configurations

We will maintain separate configuration files:

* `log4j-dev.xml` → console + file logging, `DEBUG` level.
* `log4j-prod.xml` → async rolling file logging, `INFO` level.
* `log4j-test.xml` → minimal logging for test runs.

Use Spring profiles to load the right one:

```java
System.setProperty("log4j.configuration", "classpath:log4j-" + activeProfile + ".xml");
```

---

#### 📄 Sample `log4j-dev.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">

<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">

    <!-- Console Appender -->
    <appender name="console" class="org.apache.log4j.ConsoleAppender">
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d{yyyy-MM-dd'T'HH:mm:ss.SSSZ} [%t] %-5p %c{36} [correlationId=%X{correlationId}] - %m%n"/>
        </layout>
    </appender>

    <!-- Rolling File Appender -->
    <appender name="file" class="org.apache.log4j.RollingFileAppender">
        <param name="File" value="logs/bankease.log"/>
        <param name="MaxFileSize" value="10MB"/>
        <param name="MaxBackupIndex" value="30"/>
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d{yyyy-MM-dd'T'HH:mm:ss.SSSZ} [%t] %-5p %c{36} [correlationId=%X{correlationId}] - %m%n"/>
        </layout>
    </appender>

    <!-- Root Logger -->
    <root>
        <priority value="DEBUG" />
        <appender-ref ref="console" />
        <appender-ref ref="file" />
    </root>

</log4j:configuration>
```

---

#### ⚡ Async Logging in Prod

For production, wrap appenders inside `AsyncAppender`:

```xml
<appender name="asyncFile" class="org.apache.log4j.AsyncAppender">
    <appender-ref ref="file" />
    <bufferSize>5000</bufferSize>
</appender>
```

This reduces request latency during high traffic.

---

#### 🔑 Correlation ID Handling (MDC)

Add a filter to generate a UUID per incoming request:

```java
@Component
public class CorrelationIdFilter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        String correlationId = UUID.randomUUID().toString();
        MDC.put("correlationId", correlationId);
        try {
            chain.doFilter(req, res);
        } finally {
            MDC.remove("correlationId");
        }
    }
}
```

**Pattern Note** → `%X{correlationId}` in Log4j pattern automatically prints it.

---

#### 🛡 PII Masking

Use a utility method to sanitize sensitive fields before logging:

```java
public static String mask(String value) {
    return value == null ? null : "****";
}
```

For JSON payloads, consider a custom serializer that masks sensitive keys.

---

#### 📌 Logging Guidelines

* **Controller Layer**: Entry + exit logs, request IDs, but exclude raw PII.
* **Service Layer**: Log decisions, warnings, and important state changes.
* **Repository Layer**: Only log query exceptions or anomalies.
* **Global Exception Handler**: Log stack traces with `ERROR`.

---

#### 📤 Example Log Output

```
2025-08-13T15:44:23.127+0000 [http-nio-8080-exec-5] INFO  c.b.s.CustomerService [correlationId=aa9c58f5-7f43-44c5-9c84-4b4ad2b81c9e] - Creating customer with email=masked@example.com
2025-08-13T15:44:23.142+0000 [http-nio-8080-exec-5] ERROR c.b.s.TransactionService [correlationId=aa9c58f5-7f43-44c5-9c84-4b4ad2b81c9e] - Insufficient balance for accountId=1001
```

---

#### 📚 References

* [SLF4J Manual](http://www.slf4j.org/manual.html)
* [Log4j 1.x PatternLayout](https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/PatternLayout.html)
* [MDC in SLF4J](https://www.baeldung.com/mdc-in-log4j-2-logback) *(works for Log4j 1.x as well)*

---

#### 🔍 Diagram — Logging Flow

```plaintext
Incoming HTTP Request
       |
Correlation ID Filter (UUID + MDC)
       |
Controller (entry + exit logs)
       |
Service (business logic logs)
       |
Repository (DB error logs)
       |
Controller Advice (exception logs)
       |
SLF4J API  -->  Log4j Appenders (Console, Rolling File, Async)
```



---

# 🧩 PHASE 7 (EXPANDED): Bean Validation + Custom Annotations

---

## 🔹 Objective (short)

Define a complete **validation contract** for every incoming API payload and method parameter: field-level rules, POJO-level (cross-field) rules, validation groups for create/update flows, message interpolation & i18n rules, and the exact error shapes to return on failure. *No implementation instructions here — only rules, messages, alternatives, and acceptance criteria.*

---

## 🔸 Problem statement (concise)

Requests are currently accepted with inconsistent checks. We must prevent invalid data reaching business logic by defining a single, authoritative validation specification for all DTOs and parameters. This specification must cover:

* precise rules per field (type, min/max, pattern, enumerations, database referential checks),
* custom bank-specific constraints (account-number format, idempotency-key rules),
* cross-field rules (e.g., `from != to` in transfers),
* localized, interpolated messages,
* versioned validation behaviors (create vs update),
* and expected HTTP responses for validation failures.

---

## ✅ Requirements — DTO by DTO (fields, required validation types, message keys)

> For each field below I give: Field name → Requirement → **Type of validation** → Suggested message key (for interpolation/i18n).

### Customer DTOs

* **CustomerCreateDTO**

  * `name` → required, non-blank, length 2–100.

    * **Type:** not-blank, size(min=2,max=100)
    * **Message key:** `customer.name.required` / `customer.name.size`
  * `email` → required, valid email format, **unique** (DB-level logical check).

    * **Type:** email pattern + DB-uniqueness (logical)
    * **Message key:** `customer.email.invalid` / `customer.email.duplicate`
  * `phone` → optional; if present must match `^[+0-9\-\s]{7,20}$` (international friendly).

    * **Type:** regex pattern
    * **Message key:** `customer.phone.invalid`
* **CustomerUpdateDTO**

  * `name` → optional for update but if present same length rule.

    * **Type:** conditional size
    * **Message key:** same as create
  * `email` → **immutable** once created; attempts to update email → rejection.

    * **Type:** business rule (reject), HTTP 400 (see error section)
    * **Message key:** `customer.email.immutable`
* **CustomerResponseDTO** → no validations (response only).

---

### Account DTOs

* **AccountCreateDTO**

  * `customerId` → required, must reference existing customer (DB existence check).

    * **Type:** not-null + DB-existence logical validation
    * **Message key:** `account.customer.notfound`
  * `accountNumber` → required, unique, pattern `[A-Za-z0-9]{10,20}`.

    * **Type:** custom pattern + DB-uniqueness
    * **Message key:** `account.number.invalid` / `account.number.duplicate`
  * `accountType` → required, allowed enum values: `SAVINGS`, `CURRENT`.

    * **Type:** enum validation
    * **Message key:** `account.type.invalid`
  * `currency` → optional; if present must be 3-letter ISO code; default `"INR"` if omitted.

    * **Type:** pattern/enum + defaulting (business contract)
    * **Message key:** `account.currency.invalid`
* **AccountUpdateDTO**

  * `accountType` → optional, enum only
  * `status` → optional, enum: `ACTIVE`, `SUSPENDED`, `CLOSED`

    * **Message keys:** `account.type.invalid`, `account.status.invalid`
* **AccountResponseDTO** → no validations.

---

### Transaction DTOs

* **TransactionCreateDTO**

  * `accountId` → required, must exist.

    * **Type:** not-null + DB existence check
    * **Message key:** `transaction.account.notfound`
  * `type` → required, enum: `DEBIT`, `CREDIT`
  * `amount` → required, decimal > 0.00, scale ≤ 2 (DECIMAL(18,2)).

    * **Type:** numeric min, scale check
    * **Message key:** `transaction.amount.invalid`
  * `remarks` → optional, max length 255

    * **Message key:** `transaction.remarks.toolong`
* **TransferRequestDTO**

  * `fromAccountNumber` → required, account-number pattern.

    * **Message key:** `transfer.from.invalid`
  * `toAccountNumber` → required, account-number pattern; **must not equal** `fromAccountNumber`.

    * **Type:** field-level pattern + class-level cross-field check
    * **Message key:** `transfer.to.invalid` / `transfer.sameaccount`
  * `amount` → required, > 0.00, scale ≤ 2.

    * **Message key:** `transfer.amount.invalid`
  * `idempotencyKey` → optional; if present must be 1–50 alphanumeric/hyphen/underscore.

    * **Message key:** `transfer.idempotency.invalid`

---

## 🔧 Types of validation (summary, no implementation)

* **Standard field-level:** not-null, not-blank, size, pattern, enum membership, numeric min/max, decimal scale.
* **Custom field-level:** account number format, idempotency key format.
* **Class-level (POJO/cross-field):** e.g., `fromAccountNumber != toAccountNumber`.
* **DB-referential/logical:** existence of foreign keys, uniqueness constraints.
* **Validation groups (see below):** different rules for Create vs Update vs Partial Update.

---

## 🔁 Validation Groups (what to define) — which fields belong to which group

**Groups (recommended names)** — these are *contracts*, not implementation:

* `OnCreate` (used when creating new resource)
* `OnUpdate` (used for full updates; `id` required)
* `OnPatch` / `OnPartialUpdate` (used for partial updates where optional fields allowed)

**Which field -> which group (OR'd style)**

* `Customer.id` → required for `OnUpdate` OR absent for `OnCreate`.
* `Customer.email` → validated on `OnCreate` (uniqueness check) OR **immutable** on `OnUpdate` (reject if changed).
* `Account.customerId` → required on `OnCreate` OR ignored on `OnUpdate`.
* `Transaction.amount` → required on `OnCreate` OR present & numeric on `OnPatch`.

(You may implement group behavior either via **validation groups** OR via **separate DTOs for create/update** — both are valid choices.)

---

## 🌐 Message interpolation & i18n — what to specify (OR'd options)

**Goal:** every validation message must be localizable and able to interpolate constraint attributes (e.g., `{min}`, `{max}`, `{validatedValue}`).

**Message format contract**

* Each validation rule must have a message key, for example:

  * `customer.name.required = Customer name is required.`
  * `account.number.invalid = Account number must be 10–20 alphanumeric characters.`
  * `transaction.amount.invalid = Amount must be greater than {value} and have at most {scale} decimals.`
* Messages may include interpolation tokens (`{min}`, `{max}`, `{validatedValue}`, custom tokens).

**i18n alternatives (choose one OR more):**

1. Use **properties files** per locale (`ValidationMessages.properties`, `ValidationMessages_fr.properties`, ...).
   OR
2. Use a **database-backed message store** (keys → localized values) to allow non-developers to change translations.
   OR
3. Use an **external translation service** at runtime (rare; adds latency and complexity).

**Locale resolution alternatives (OR'd):**

* `Accept-Language` HTTP header → server picks best available locale.
  OR
* Explicit request parameter or URL prefix (`/v1/en/...`) → client chooses locale.
  OR
* Cookie/session-based locale set by authentication/profile.

**Interpolation behavior (requirements):**

* Messages must support interpolation of constraint attributes. Example message:
  `account.name.size = Name must be between {min} and {max} characters.`
* If no locale is resolvable, default to `en` (fallback).
* For DB-referenced violations (e.g., email duplicate), use a dedicated message key: `customer.email.duplicate` with optional `{value}` interpolation.

---

## 📤 Expected HTTP status codes & message contract (validation errors)

**Primary behavior**

* On any payload validation failure:

  * **HTTP 400 Bad Request**
  * **JSON body** following this contract (single canonical form or RFC-7807 alternative — both allowed, pick one):

**Canonical error format (recommended)**

```json
{
  "timestamp": "2025-08-13T10:15:30Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Validation failed",
  "details": [
    { "field": "email", "messageKey": "customer.email.invalid", "message": "must be a well-formed email address", "rejectedValue": "bob@" },
    { "field": "toAccountNumber", "messageKey": "transfer.sameaccount", "message": "destination must differ from source" }
  ],
  "locale": "en"
}
```

**OR (RFC 7807-style Problem Details alternative)**

* `Content-Type: application/problem+json`
* Top-level `type`, `title`, `status`, `detail`, and `instance`, plus an extension `invalidParams` array with the same entries as `details`.

  * *Choose either canonical JSON body OR RFC 7807 formatting. Both must include message keys for i18n.*

**Message preference**

* Always include both `messageKey` (machine-friendly) and `message` (localized text) in the response.
* Include `rejectedValue` only if it’s not sensitive (do not echo sensitive PII like full card numbers).

---

## ✅ Acceptance Criteria (testable)

1. **Per-field** rules: for each DTO field listed above, sending invalid value yields HTTP 400 with entry in `details` matching the field and messageKey.
2. **Class-level** checks: e.g., transfer from==to → HTTP 400 + `transfer.sameaccount`.
3. **Validation groups**: `OnCreate` rules triggered when creating, `OnUpdate` rules triggered on update. Choosing groups OR separate DTOs behavior must be consistent across APIs.
4. **i18n**: When client requests `Accept-Language: fr`, messages are returned in French if translations exist; otherwise fallback to `en`.
5. **Message interpolation**: `{min}`, `{max}`, `{validatedValue}` tokens are replaced in localized messages.
6. **DB-referenced checks**: uniqueness/existence checks return specific message keys (`*.duplicate`, `*.notfound`) and 400/409 as per business rule (see note below).
7. **No sensitive echoing**: `rejectedValue` must not include sensitive data for any field marked sensitive.
8. **Validation occurs before service business logic**: invalid requests do not reach core service transactions.

**Business-rule mapping note:**

* Format/structure validation → **400 Bad Request**.
* Uniqueness conflict (resource already exists) → either **400** or **409 Conflict** depending on team decision (specify one). *Choose one: use 409 for uniqueness conflicts OR 400 consistently for all validation failures.* (Record the chosen policy).

---

## ⚠ Disallowed / Pitfalls (explicit)

* **Heavy DB calls inside validators**: avoid validators that perform expensive queries per request (scan over indexes is ok; but heavy joins/aggregation is not).
* **Echoing PII in `rejectedValue`** (SSN, full account numbers, card PANs) — mask or omit.
* **Inconsistent message keys / duplicate messages** across modules (always centralize `ValidationMessages`).
* **Mixing implementation-specific messages** (stack traces) into user-facing validation messages.
* **Relying on client locale** exclusively — server must provide fallback.
* **Using the same DTO for both public API and internal persistence** (exposes internal fields/validation mismatch).
* **Overly generic messages** (e.g., “Invalid input”) — always provide field-level detail and message keys for automation.
* **Conflicting HTTP statuses** for the same logical error across endpoints (pick 400 vs 409 policy and stick to it).

---

## 🧪 Functional Verification Checklist (concrete)

* Send invalid `CustomerCreateDTO` (missing name/email) → 400; `details` includes `customer.name.required`, `customer.email.invalid`.
* Send `TransferRequestDTO` with same `from` & `to` → 400; `details` contains `transfer.sameaccount`.
* Create customer with existing email → 409 *OR* 400 depending on team choice; response includes `customer.email.duplicate`.
* Request with `Accept-Language: es` → messages in Spanish when translations present.
* Partial update (`PATCH`) triggers `OnPatch` rules only; full update (`PUT`) triggers `OnUpdate` rules.

---

## 🖼 Diagrams

### 1) Entity/DTO relationship (ASCII)

```
[Customer Entity] ←─< 1..* ── [Account Entity] ──< 1..* ── [Transaction Entity]
     |                          |                          |
     v                          v                          v
CustomerCreateDTO           AccountCreateDTO          TransactionCreateDTO
CustomerUpdateDTO           AccountUpdateDTO         TransactionResponseDTO
CustomerResponseDTO         AccountResponseDTO       TransferRequestDTO
```

### 2) Validation flow (with OR'd places where validation may be triggered)

```
[HTTP Request]
     |
     v
(Option 1) Pre-controller shallow JSON schema check  OR
(Option 2) Controller @Valid on @RequestBody (primary)  OR
(Option 3) Controller accepts raw and Service layer validates via @Validated
     |
     v
Field-level checks  +  Class-level checks  +  DB-referential checks
     |
     |- if failure -> Formatted error body (messageKey + localized message) -> HTTP 400/409
     |
     v
Service layer execution (only if validation passed)
```

*(Choose one OR combination of 1/2/3 for your enforcement architecture; the contract above applies regardless.)*

---

## 📚 References (expanded)

* Bean Validation (JSR-380): [https://beanvalidation.org/2.0-jsr380/](https://beanvalidation.org/2.0-jsr380/)
* Hibernate Validator (user guide, 6.x): [https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html\_single/](https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/)
* Spring Framework — Validation integration: [https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation)
* Baeldung — Custom Constraint & Validation groups:

  * Custom constraint: [https://www.baeldung.com/spring-mvc-custom-validator](https://www.baeldung.com/spring-mvc-custom-validator)
  * Validation groups: [https://www.baeldung.com/spring-validation-groups](https://www.baeldung.com/spring-validation-groups)
  * i18n messages: [https://www.baeldung.com/spring-message-source](https://www.baeldung.com/spring-message-source)
* RFC 7807 — Problem Details for HTTP APIs: [https://datatracker.ietf.org/doc/html/rfc7807](https://datatracker.ietf.org/doc/html/rfc7807)
* Example discussion on class-level constraints: [https://stackoverflow.com/questions/32881129/custom-class-level-bean-validation-constraint](https://stackoverflow.com/questions/32881129/custom-class-level-bean-validation-constraint)
* Practical guide on message interpolation: [https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html\_single/#section-message-interpolation](https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#section-message-interpolation)

---



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
