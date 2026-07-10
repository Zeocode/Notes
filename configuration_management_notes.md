# Configuration Management in Microservices
## Complete Study Notes — Spring Boot & Spring Cloud Config
### 20 Lectures | 6 Sections

---

## Table of Contents

- [Section 1 — Introduction](#section-1--introduction-to-configuration-management)
  - [Lec 1 — Configuration Management Challenges](#lec-1--configuration-management-challenges-in-microservices)
- [Section 2 — Spring Boot Configuration Approaches](#section-2--spring-boot-configuration-approaches)
  - [Lec 2 — How Configurations Work in Spring Boot](#lec-2--how-configurations-work-in-spring-boot)
  - [Lec 3 — Reading Configurations Using `@Value`](#lec-3--reading-configurations-using-value-annotation)
  - [Lec 4 — Reading Configurations Using `Environment` Interface](#lec-4--reading-configurations-using-environment-interface)
  - [Lec 5 — Reading Configurations Using `@ConfigurationProperties`](#lec-5--reading-configurations-using-configurationproperties)
- [Section 3 — Spring Boot Profiles](#section-3--spring-boot-profiles)
  - [Lec 6 — Introduction to Spring Boot Profiles](#lec-6--introduction-to-spring-boot-profiles)
  - [Lec 7 — Demo: Profiles in Accounts Microservice](#lec-7--demo-spring-boot-profiles-in-accounts-microservice)
  - [Lec 8 — Externalizing Configurations via CLI / JVM / Env](#lec-8--externalizing-configurations-via-cli--jvm--environment-options)
  - [Lec 9 — Activating Profiles via External Options](#lec-9--activating-profiles-via-cli--jvm--environment-options-demo)
  - [Lec 10 — Assignment: Profile Changes in Loans & Cards](#lec-10--assignment-profile-changes-in-loans--cards-microservices)
  - [Lec 11 — Demo: Profile Changes in Loans & Cards](#lec-11--demo-profile-changes-in-loans--cards-microservices)
- [Section 4 — Drawbacks & Intro to Spring Cloud Config](#section-4--drawbacks--introduction-to-spring-cloud-config)
  - [Lec 12 — Drawbacks of Spring Boot Alone](#lec-12--drawbacks-of-externalized-configurations-using-spring-boot-alone)
  - [Lec 13 — Introduction to Spring Cloud Config](#lec-13--introduction-to-spring-cloud-config)
- [Section 5 — Building & Integrating the Config Server](#section-5--building--integrating-the-config-server)
  - [Lec 14 — Building the Config Server](#lec-14--building-config-server-using-spring-cloud-config)
  - [Lec 15 — Reading Configs from Classpath Location](#lec-15--reading-configurations-from-classpath-location-of-config-server)
  - [Lec 16 — Connecting Accounts Microservice to Config Server](#lec-16--updating-accounts-microservice-to-read-from-config-server)
  - [Lec 17 — Connecting Loans & Cards to Config Server](#lec-17--updating-loans--cards-microservices-to-read-from-config-server)
- [Section 6 — Advanced Config Server Backends](#section-6--advanced-config-server-backends)
  - [Lec 18 — Reading from File System Location](#lec-18--reading-configurations-from-a-file-system-location)
  - [Lec 19 — Reading from GitHub Repository](#lec-19--reading-configurations-from-a-github-repository)
  - [Lec 20 — Encryption & Decryption of Properties](#lec-20--encryption--decryption-of-properties-in-config-server)

---

## Section 1 — Introduction to Configuration Management

---

### Lec 1 — Configuration Management Challenges in Microservices

#### The Core Problem

In a microservices architecture, configurations (database URLs, credentials, feature flags, environment-specific values) must be **separated from business logic**. Without this separation:

- You cannot reuse the same Docker image across environments.
- You must rebuild and redeploy for every config change.
- Secrets get embedded into source code or images.

#### Key Questions This Section Answers

1. How do we **separate** configurations from business logic?
2. How do we **inject** configurations at runtime?
3. How do we **centrally manage** thousands of properties across hundreds of microservices?

#### The 15-Factor App Methodology

> Store all config that varies between deployments (credentials, URLs, resource handles) **outside** the build artifact.

The artifact (Docker image) must be **immutable** — the same image is deployed to dev, QA, and prod, with environment-specific configs injected at startup.

```
GitHub Repo → Build (Docker Image) ──┬──► DEV   + dev configs
                                      ├──► QA    + qa configs
                                      └──► PROD  + prod configs
```

#### Three Solutions Available (Covered in This Course)

| Approach | Description |
|---|---|
| **1. Spring Boot Properties + Profiles** | Basic — property files + profile activation |
| **2. External Configuration (CLI/JVM/Env)** | Inject config from outside at runtime |
| **3. Spring Cloud Config Server** | Production-grade centralized config management |

---

## Section 2 — Spring Boot Configuration Approaches

---

### Lec 2 — How Configurations Work in Spring Boot

#### Configuration Sources (Lowest → Highest Priority)

```
application.properties / application.yml       ← lowest priority
OS Environment Variables
Java System Properties  (-D flags)
JNDI / Servlet attributes
Command-line arguments  (--key=value)           ← highest priority
```

> If the same property is defined in multiple places, the source with **higher priority wins**.

#### Ways to Read Properties in Java Code

| Approach | Best For |
|---|---|
| `@Value` annotation | 1–2 individual properties |
| `Environment` interface | OS environment variables |
| `@ConfigurationProperties` | Many grouped properties (recommended) |

---

### Lec 3 — Reading Configurations Using `@Value` Annotation

#### Project Setup

```
workspace/
  section6/
    v1-springboot/
      accounts/
      cards/
      loans/
```

All microservices use **Google Jib** Maven plugin to build Docker images.

#### Step 1 — Define a property in `application.yml`

```yaml
# accounts/src/main/resources/application.yml
build:
  version: "1.0"
```

#### Step 2 — Read it in the controller using `@Value`

```java
// AccountsController.java
import org.springframework.beans.factory.annotation.Value;

@RestController
@RequestMapping("/api")
public class AccountsController {

    // ✅ Inject via @Value — Spring reads from application.yml
    @Value("${build.version}")
    private String buildVersion;

    // Manual constructor (do NOT use @AllArgsConstructor — Spring can't inject String beans)
    private final IAccountService iAccountService;

    public AccountsController(IAccountService iAccountService) {
        this.iAccountService = iAccountService;
    }

    @Operation(summary = "Get build information",
               description = "Get the build information deployed into accounts microservice")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "HttpStatus.OK"),
        @ApiResponse(responseCode = "500", description = "Internal Server Error",
                     content = @Content(schema = @Schema(implementation = ErrorResponseDto.class)))
    })
    @GetMapping("/build-info")
    public ResponseEntity<String> getBuildInfo() {
        return ResponseEntity.status(HttpStatus.OK).body(buildVersion);
    }
}
```

> ⚠️ **Common mistake**: Using `@AllArgsConstructor` on the controller includes `buildVersion` (a String) in the constructor, which Spring cannot autowire. Always create a manual constructor for only your service dependencies.

#### Test in Postman

```
GET http://localhost:8080/api/build-info
→ Response: 1.0
```

#### Limitations of `@Value`

- Hardcodes property key names inside Java code.
- Can only read **one property at a time** per field.
- Not scalable for many properties.

---

### Lec 4 — Reading Configurations Using `Environment` Interface

#### Why Use Environment Variables?

Sensitive data (passwords, secrets) should **never** be in `application.yml` — they should be OS-level environment variables, accessible only to server admins.

#### Step 1 — Autowire the `Environment` interface

```java
// AccountsController.java
import org.springframework.core.env.Environment;  // ← from Spring Core, NOT Hibernate!

@RestController
@RequestMapping("/api")
public class AccountsController {

    @Autowired
    private Environment environment;

    @GetMapping("/java-version")
    public ResponseEntity<String> getJavaVersion() {
        return ResponseEntity
            .status(HttpStatus.OK)
            .body(environment.getProperty("JAVA_HOME"));
    }
}
```

> ⚠️ Make sure you import `org.springframework.core.env.Environment`, **not** the Hibernate one.

#### Test in Postman

```
GET http://localhost:8080/api/java-version
→ Response: /Users/username/.sdkman/candidates/java/current  (or wherever Java is installed)
```

#### Limitations of `Environment` Interface

- Still reads **one property at a time**.
- Property key is still hardcoded as a String.
- Not recommended for many properties.

---

### Lec 5 — Reading Configurations Using `@ConfigurationProperties`

#### Why `@ConfigurationProperties`?

- Reads **multiple grouped properties** at once.
- **No hardcoded keys** inside Java code.
- Spring team's **recommended** approach.

#### Step 1 — Define grouped properties in `application.yml`

```yaml
# accounts/src/main/resources/application.yml
accounts:
  message: "Welcome to EazyBank accounts related local APIs"
  contactDetails:
    name: "John Doe - Developer"
    email: "john@eazybank.com"
  onCallSupport:
    - "+91 1234567890"
    - "+91 9876543210"
```

#### Step 2 — Create a Java Record (DTO) mapped to these properties

```java
// dto/AccountsContactInfoDto.java
import org.springframework.boot.context.properties.ConfigurationProperties;
import java.util.List;
import java.util.Map;

@ConfigurationProperties(prefix = "accounts")
public record AccountsContactInfoDto(
    String message,
    Map<String, String> contactDetails,
    List<String> onCallSupport
) {}
```

> **Java Record**: Introduced in Java 17. Fields are final, immutable, auto-generates getters and constructor. No setters.
>
> Field names and data types **must exactly match** the YAML property names and structure:
> - `message` → `String`
> - `contactDetails` → `Map<String, String>` (key-value pairs)
> - `onCallSupport` → `List<String>` (list of phone numbers)

#### Step 3 — Enable in the main application class

```java
// AccountsApplication.java
import org.springframework.boot.context.properties.EnableConfigurationProperties;

@SpringBootApplication
@EnableConfigurationProperties(value = {AccountsContactInfoDto.class})
public class AccountsApplication {
    public static void main(String[] args) {
        SpringApplication.run(AccountsApplication.class, args);
    }
}
```

#### Step 4 — Inject and use in controller

```java
// AccountsController.java
@RestController
@RequestMapping("/api")
public class AccountsController {

    @Autowired
    private AccountsContactInfoDto accountsContactInfoDto;

    @GetMapping("/contact-info")
    public ResponseEntity<AccountsContactInfoDto> getContactInfo() {
        return ResponseEntity
            .status(HttpStatus.OK)
            .body(accountsContactInfoDto);
    }
}
```

#### Test in Postman

```
GET http://localhost:8080/api/contact-info
→ Response:
{
  "message": "Welcome to EazyBank accounts related local APIs",
  "contactDetails": { "name": "John Doe - Developer", "email": "john@eazybank.com" },
  "onCallSupport": ["+91 1234567890", "+91 9876543210"]
}
```

---

## Section 3 — Spring Boot Profiles

---

### Lec 6 — Introduction to Spring Boot Profiles

#### The Problem

All environments (dev, QA, prod) currently use the same property values. Real projects need **different values per environment** (e.g. different DB credentials, different contact details).

#### What Are Profiles?

Spring Boot **profiles** group configurations that are activated based on the running environment.

```
Default Profile → application.yml          (always active, lowest priority)
QA Profile      → application_qa.yml       (activated when spring.profiles.active=qa)
Prod Profile    → application_prod.yml     (activated when spring.profiles.active=prod)
```

#### Activating a Profile in `application.yml`

```yaml
spring:
  profiles:
    active: "prod"   # Activates the prod profile
```

You can also activate **multiple profiles** with comma-separated values:

```yaml
spring:
  profiles:
    active: "prod,feature-flags"
```

#### How Profile Override Works

When a profile is activated, Spring **loads both** the default profile AND the active profile. Properties in the active profile **override** matching properties in the default profile.

#### Profile Benefits

- No code change or Docker image rebuild needed to switch environments.
- Beans can also be **conditionally created** based on active profile using `@Profile("prod")`.

---

### Lec 7 — Demo: Spring Boot Profiles in Accounts Microservice

#### Step 1 — Create profile-specific YAML files

```
accounts/src/main/resources/
  application.yml          ← default profile (local dev)
  application_qa.yml       ← QA environment
  application_prod.yml     ← Production environment
```

#### Step 2 — `application.yml` (default profile)

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:h2:mem:testdb
    # ... h2 config
  config:
    import:
      - "application_qa.yml"
      - "application_prod.yml"

build:
  version: "3.0"

accounts:
  message: "Welcome to EazyBank accounts related local APIs"
  contactDetails:
    name: "John Doe - Developer"
    email: "john@eazybank.com"
  onCallSupport:
    - "+91 1111111111"
    - "+91 2222222222"
```

#### Step 3 — `application_qa.yml` (QA profile)

```yaml
spring:
  config:
    activate:
      on-profile: "qa"

build:
  version: "2.0"

accounts:
  message: "Welcome to EazyBank accounts related QA APIs"
  contactDetails:
    name: "Jane Smith - QA Lead"
    email: "jane@eazybank.com"
  onCallSupport:
    - "+91 3333333333"
    - "+91 4444444444"
```

#### Step 4 — `application_prod.yml` (Production profile)

```yaml
spring:
  config:
    activate:
      on-profile: "prod"

build:
  version: "1.0"

accounts:
  message: "Welcome to EazyBank accounts related prod APIs"
  contactDetails:
    name: "Product Owner - Prod"
    email: "owner@eazybank.com"
  onCallSupport:
    - "+91 5555555555"
    - "+91 6666666666"
```

#### Step 5 — Activate QA profile in `application.yml`

```yaml
spring:
  profiles:
    active: "qa"
```

#### Test Results

```
GET /api/build-info   → "2.0"     (from qa profile)
GET /api/contact-info → qa lead details
```

> ⚠️ **YAML indentation is critical!** If `profiles` is accidentally indented under `config`, the profile will never activate. Always verify the YAML hierarchy.

---

### Lec 8 — Externalizing Configurations via CLI / JVM / Environment Options

#### Problem with Hardcoding the Active Profile

If `spring.profiles.active=prod` is hardcoded in `application.yml`, you must change that file and rebuild the Docker image every time you switch environments — violating the immutability principle.

#### Solution: Pass Config from Outside

##### 1. Command-Line Arguments (Highest Priority)

```bash
# Start with fat jar
java -jar accounts-0.0.1-SNAPSHOT.jar \
  --spring.profiles.active=prod \
  --build.version=1.1
```

Format: `--propertyKey=value` (two hyphens prefix)

##### 2. JVM System Properties

```bash
java -Dspring.profiles.active=prod \
     -Dbuild.version=1.2 \
     -jar accounts-0.0.1-SNAPSHOT.jar
```

Format: `-DpropertyKey=value` (single `-D` prefix)

##### 3. OS Environment Variables

```bash
# Linux / macOS
SPRING_PROFILES_ACTIVE=prod BUILD_VERSION=1.3 java -jar accounts.jar

# Windows
$env:SPRING_PROFILES_ACTIVE="prod"; java -jar accounts.jar
```

Format rules for environment variables:
- Convert all letters to **UPPERCASE**
- Replace all `.` (dots) with `_` (underscores)
- Separate multiple variables with `;` (semicolon)

```
spring.profiles.active  →  SPRING_PROFILES_ACTIVE
build.version           →  BUILD_VERSION
```

##### Priority Order Summary

```
application.yml (lowest)
  < OS Environment Variables
  < JVM System Properties (-D flags)
  < Command-Line Arguments (--) (highest)
```

---

### Lec 9 — Activating Profiles via CLI / JVM / Environment Options (Demo)

#### Using IntelliJ Run Configurations

Instead of building a fat jar, developers can pass these options directly via IntelliJ:

**Right-click on main class → Modify Run Configuration**

##### Command-Line Arguments (Program Arguments field)
```
--spring.profiles.active=prod --build.version=1.1
```

##### JVM System Variables (VM Options field — click "Modify Options → Add VM Options")
```
-Dspring.profiles.active=prod -Dbuild.version=1.3
```

##### Environment Variables (Environment Variables field)
```
SPRING_PROFILES_ACTIVE=prod;BUILD_VERSION=1.8
```

#### Precedence Demo Results

| Config Source Active | `build-info` API Returns |
|---|---|
| All three active (CLI + JVM + Env) | `1.3` (CLI wins) |
| JVM + Env active | `1.1` (JVM wins) |
| Env only | `1.8` (Env used) |
| Nothing external | `1.0` (from prod profile file) |

---

### Lec 10 — Assignment: Profile Changes in Loans & Cards Microservices

#### Task

Apply the same changes done in the accounts microservice to both **loans** and **cards** microservices:

1. Add `build.version`, `message`, `contactDetails`, `onCallSupport` properties in each microservice's `application.yml`.
2. Create `application_qa.yml` and `application_prod.yml` for each.
3. Create `LoansContactInfoDto` and `CardsContactInfoDto` record classes with `@ConfigurationProperties`.
4. Register them in `LoansApplication` and `CardsApplication` with `@EnableConfigurationProperties`.
5. Add three new REST APIs: `/build-info`, `/java-version`, `/contact-info`.

> Reference code: GitHub repo `eazybytes/microservice` → section6 → v1-springboot

---

### Lec 11 — Demo: Profile Changes in Loans & Cards Microservices

#### Loans Microservice — Key Changes

**`LoansContactInfoDto.java`**
```java
@ConfigurationProperties(prefix = "loans")   // ← prefix must be "loans"
public record LoansContactInfoDto(
    String message,
    Map<String, String> contactDetails,
    List<String> onCallSupport
) {}
```

**`LoansApplication.java`**
```java
@SpringBootApplication
@EnableConfigurationProperties(value = {LoansContactInfoDto.class})
public class LoansApplication { ... }
```

**`LoansController.java`** — Three new APIs
```java
@RestController
@RequestMapping("/api")
public class LoansController {

    @Value("${build.version}")
    private String buildVersion;

    @Autowired
    private Environment environment;

    @Autowired
    private LoansContactInfoDto loansContactInfoDto;

    public LoansController(ILoansService iLoansService) {
        this.iLoansService = iLoansService;
    }

    @GetMapping("/build-info")
    public ResponseEntity<String> getBuildInfo() {
        return ResponseEntity.ok(buildVersion);
    }

    @GetMapping("/java-version")
    public ResponseEntity<String> getJavaVersion() {
        return ResponseEntity.ok(environment.getProperty("JAVA_HOME"));
    }

    @GetMapping("/contact-info")
    public ResponseEntity<LoansContactInfoDto> getContactInfo() {
        return ResponseEntity.ok(loansContactInfoDto);
    }
}
```

**`application_prod.yml`** (loans)
```yaml
spring:
  config:
    activate:
      on-profile: "prod"

build:
  version: "1.0"

loans:
  message: "Welcome to EazyBank loans related prod APIs"
  contactDetails:
    name: "Loans Manager - Prod"
    email: "loans@eazybank.com"
  onCallSupport:
    - "+91 7777777777"
    - "+91 8888888888"
```

Apply the same pattern for the **cards** microservice, using the prefix `cards`.

---

## Section 4 — Drawbacks & Introduction to Spring Cloud Config

---

### Lec 12 — Drawbacks of Externalized Configurations Using Spring Boot Alone

#### Limitation 1 — Manual Injection Risk

Injecting CLI/JVM/env variables requires manual setup during deployment (in CI/CD pipelines). This is error-prone and doesn't scale for hundreds of microservices.

#### Limitation 2 — No Versioning or Auditing

- Configurations embedded in source code can't be independently versioned.
- No audit trail for who changed a property and when.
- Anyone with source code access sees all configurations.

#### Limitation 3 — No Granular Access Control

Environment variables are visible to all server admins — you can't restrict who sees sensitive values.

#### Limitation 4 — Scaling Problem

With 100 microservices × 5 instances each = **500 running instances**. Manually providing configs to all of them (even via CI/CD) is extremely challenging.

#### Limitation 5 — No Encryption Support

CLI args, JVM properties, and env variables all expose **plain text** secrets. There is no built-in encryption.

#### Limitation 6 — Restart Required for Config Changes

Any change to a property requires a **full application restart**. There's no way to hot-reload configs.

---

### Lec 13 — Introduction to Spring Cloud Config

#### What is Spring Cloud Config?

A dedicated **Spring Cloud** project that provides a centralized configuration server for distributed systems (microservices).

```
┌─────────────────────────────────────────────┐
│           Config Repository                  │
│  (GitHub / File System / Classpath / DB)     │
└──────────────────┬──────────────────────────┘
                   │ stores
                   ▼
┌─────────────────────────────────────────────┐
│         Spring Cloud Config Server           │
│         (port 8071)                          │
└──────────────────┬──────────────────────────┘
                   │ serves properties
       ┌───────────┼──────────────┐
       ▼           ▼              ▼
  Accounts      Loans          Cards
 Microservice Microservice  Microservice
  (client)      (client)      (client)
```

#### Two Core Elements

1. **Config Repository** — Where properties are stored (GitHub, file system, classpath, DB).
2. **Config Server** — Reads from the repository and distributes to all microservices.

#### Benefits Over Spring Boot Alone

| Feature | Spring Boot Alone | Spring Cloud Config |
|---|---|---|
| Centralized management | ✗ | ✓ |
| Versioning & audit trail | ✗ | ✓ (via Git) |
| Encryption support | ✗ | ✓ |
| Hot-reload (no restart) | ✗ | ✓ |
| Granular access control | ✗ | ✓ |
| Scales to 100+ services | ✗ | ✓ |

#### Spring Cloud vs Spring Boot

- **Spring Boot** — Framework for building standalone applications.
- **Spring Cloud** — A collection of frameworks/projects to solve distributed-systems patterns.
- Spring Cloud Config is one project within Spring Cloud.
- Spring Boot and Spring Cloud have **separate version numbers** (mapped via a compatibility table on `spring.io`).

> 💡 **Fun fact**: Spring Cloud version names (Dalston, Edgware, Finchley, Greenwich, Hoxton…) are all **London Underground station names**, following alphabetical order.

---

## Section 5 — Building & Integrating the Config Server

---

### Lec 14 — Building Config Server Using Spring Cloud Config

#### Project Setup

Create folder `v2-spring-cloud-config` (copied from v1, starting fresh for this approach).

#### Step 1 — Generate the project at `start.spring.io`

| Field | Value |
|---|---|
| Language | Java |
| Project | Maven |
| Group | `com.eazybytes` |
| Artifact | `configserver` |
| Name | `configserver` |
| Description | Config Server for EazyBank Microservices |
| Packaging | JAR |
| Dependencies | **Config Server**, **Spring Boot Actuator** |

Generated `pom.xml` will include:

```xml
<properties>
    <spring-cloud.version>2022.0.3</spring-cloud.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

#### Step 2 — Enable Config Server in main class

```java
// ConfigserverApplication.java
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer       // ← This annotation activates the config server
public class ConfigserverApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigserverApplication.class, args);
    }
}
```

#### Step 3 — Configure `application.yml`

```yaml
server:
  port: 8071   # Config server always runs on 8071 in this course
```

---

### Lec 15 — Reading Configurations from Classpath Location of Config Server

#### Step 1 — Set a name for the config server

```yaml
# configserver/src/main/resources/application.yml
server:
  port: 8071

spring:
  application:
    name: "configserver"
  profiles:
    active: native          # "native" = use classpath or file system
  cloud:
    config:
      server:
        native:
          search-locations: "classpath:/config"
```

> **Why `native` profile?** The `native` profile tells Spring Cloud Config Server to read configs from the local file system or classpath (instead of a Git repo).

#### Step 2 — Create the `config/` folder and copy property files

```
configserver/src/main/resources/
  config/
    accounts.yml          ← default profile for accounts
    accounts-prod.yml     ← prod profile for accounts
    accounts-qa.yml       ← qa profile for accounts
    loans.yml
    loans-prod.yml
    loans-qa.yml
    cards.yml
    cards-prod.yml
    cards-qa.yml
```

> ⚠️ **Critical naming rule**: Files must be named `{microservice-name}.yml` and `{microservice-name}-{profile}.yml` using a **hyphen** (NOT underscore). The microservice name must match its `spring.application.name`.

#### File content example — `accounts-prod.yml`

```yaml
build:
  version: "1.0"

accounts:
  message: "Welcome to EazyBank accounts related prod APIs"
  contactDetails:
    name: "Product Owner - Prod"
    email: "owner@eazybank.com"
  onCallSupport:
    - "+91 5555555555"
    - "+91 6666666666"
```

> Remove the `spring.config.activate.on-profile` section from these files — the config server handles profile routing by file name convention.

#### Step 3 — Validate the Config Server

After starting the config server on port 8071, test via browser or Postman:

```
GET http://localhost:8071/accounts/prod
→ Returns merged properties (prod profile + default profile)

GET http://localhost:8071/accounts/qa
→ Returns merged properties (qa profile + default profile)

GET http://localhost:8071/loans/default
→ Returns only default profile properties
```

---

### Lec 16 — Updating Accounts Microservice to Read from Config Server

#### Step 1 — Clean up `application.yml` in accounts microservice

Remove:
- `application_qa.yml` and `application_prod.yml` files (no longer needed)
- `spring.config.import` for local profile files
- `spring.profiles.active` hardcoded value
- `build.version` and `accounts.*` properties (now stored in config server)

Keep:
- `server.port`
- `spring.datasource` (H2 database config)

#### Step 2 — Add `spring.application.name` and connect to config server

```yaml
# accounts/src/main/resources/application.yml
server:
  port: 8080

spring:
  application:
    name: "accounts"        # ← Must match file name in config server's config/ folder
  profiles:
    active: "prod"          # ← Default active profile (can be overridden externally)
  config:
    import: "optional:configserver:http://localhost:8071/"
  datasource:
    url: jdbc:h2:mem:testdb
    # ... h2 config
```

> **`optional:configserver:`** prefix tells Spring Boot this is a config server endpoint. The `optional:` prefix means the app will still start even if the config server is unavailable (shows a warning instead of an error).

#### Step 3 — Add Spring Cloud Config Client dependency to `pom.xml`

```xml
<properties>
    <spring-cloud.version>2022.0.3</spring-cloud.version>
</properties>

<dependencies>
    <!-- Existing dependencies... -->

    <!-- Config Client — connects this microservice to the config server -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

#### How It Works at Startup

```
Accounts starts
  → Reads spring.application.name = "accounts"
  → Reads spring.profiles.active = "prod"
  → Calls config server: GET http://localhost:8071/accounts/prod
  → Config server returns merged properties from accounts-prod.yml + accounts.yml
  → Accounts injects those properties into @Value / @ConfigurationProperties fields
```

#### Test via Postman

```
GET http://localhost:8080/api/contact-info  → prod profile properties
GET http://localhost:8080/api/build-info    → "1.0"
```

To switch to QA profile externally:
```
--spring.profiles.active=qa
```
Then:
```
GET http://localhost:8080/api/build-info    → "2.0"
GET http://localhost:8080/api/contact-info  → qa profile properties
```

---

### Lec 17 — Updating Loans & Cards Microservices to Read from Config Server

Apply the same changes as Accounts to both microservices.

#### Loans Microservice `application.yml`

```yaml
server:
  port: 8090

spring:
  application:
    name: "loans"           # ← Must match loans.yml files in config server
  profiles:
    active: "prod"
  config:
    import: "optional:configserver:http://localhost:8071/"
  datasource:
    url: jdbc:h2:mem:testdb
```

#### Cards Microservice `application.yml`

```yaml
server:
  port: 9000

spring:
  application:
    name: "cards"           # ← Must match cards.yml files in config server
  profiles:
    active: "prod"
  config:
    import: "optional:configserver:http://localhost:8071/"
  datasource:
    url: jdbc:h2:mem:testdb
```

Both also need the same `spring-cloud-starter-config` dependency and Spring Cloud BOM in their `pom.xml` files.

#### Microservice Ports Summary

| Microservice | Port |
|---|---|
| Config Server | 8071 |
| Accounts | 8080 |
| Loans | 8090 |
| Cards | 9000 |

---

## Section 6 — Advanced Config Server Backends

---

### Lec 18 — Reading Configurations from a File System Location

#### Why File System?

- Server admins can apply OS-level folder access restrictions.
- Configs are separate from both the application code and the config server code.
- Only the config server process can read the folder.

#### Step 1 — Copy config files to a local folder

Example path (macOS):
```
/Users/eazybytes/Documents/config/
  accounts.yml
  accounts-prod.yml
  accounts-qa.yml
  loans.yml
  loans-prod.yml
  loans-qa.yml
  cards.yml
  cards-prod.yml
  cards-qa.yml
```

#### Step 2 — Update Config Server `application.yml`

```yaml
spring:
  application:
    name: "configserver"
  profiles:
    active: native        # still native — file system is also "native"
  cloud:
    config:
      server:
        native:
          search-locations: "file:///Users/eazybytes/Documents/config"
          # Windows example: "file:///C://config"
```

> **Path syntax:**
> - `file:///` = 3 forward slashes (file: + // + /path)
> - macOS/Linux: `file:///Users/username/Documents/config`
> - Windows: `file:///C://config`

#### Verification

```
GET http://localhost:8071/loans/prod
→ Response shows location: file:///.../loans-prod.yml ✓
```

---

### Lec 19 — Reading Configurations from a GitHub Repository

#### Why GitHub is the Most Recommended Approach

| Benefit | Description |
|---|---|
| **Versioning** | Full Git history — see what properties were used 2 years ago |
| **Auditing** | Commit log shows who changed what and when |
| **Security** | Repo can be private with access control |
| **Branching** | Separate branches per environment or release |
| **Widely Used** | Standard practice in production organizations |

#### Step 1 — Create/Use a GitHub Repo

Example: `https://github.com/eazybytes/eazybytes-config`

Upload all config files:
```
eazybytes-config/
  accounts.yml
  accounts-prod.yml
  accounts-qa.yml
  loans.yml
  loans-prod.yml
  loans-qa.yml
  cards.yml
  cards-prod.yml
  cards-qa.yml
```

#### Step 2 — Update Config Server `application.yml` to use GitHub

```yaml
spring:
  application:
    name: "configserver"
  profiles:
    active: "git"          # ← Switch from "native" to "git"
  cloud:
    config:
      server:
        git:
          uri: "https://github.com/eazybytes/eazybytes-config"
          default-label: main         # ← Branch name to use
          timeout: 5                  # ← Fail after 5 seconds if can't connect
          clone-on-start: true        # ← Clone repo at startup, not on first request
          force-pull: true            # ← Always pull from GitHub, override local changes
        # native config is now commented out:
        # native:
        #   search-locations: "classpath:/config"
```

#### Private GitHub Repo Authentication

If the repository is private, add credentials:

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: "https://github.com/your-org/private-config-repo"
          username: "your-github-username"
          password: "your-personal-access-token"
          default-label: main
```

Or use SSH keys — see Spring Cloud Config official documentation for details.

#### `clone-on-start` vs Default Behavior

| Setting | Behavior |
|---|---|
| `clone-on-start: false` (default) | Clones GitHub repo on first client request — may cause startup failures |
| `clone-on-start: true` | Clones GitHub repo when config server starts — recommended |

#### `force-pull: true`

Forces config server to always pull fresh from GitHub on startup, discarding any locally cached/modified copies.

#### Startup Order

Always start services in this order:
1. **Config Server** (must be ready before microservices)
2. Accounts Microservice
3. Loans Microservice
4. Cards Microservice

#### Verification

```
GET http://localhost:8071/accounts/prod
→ Source shows GitHub URL ✓

GET http://localhost:8080/api/contact-info
→ Returns prod profile data from GitHub repo ✓
```

#### Other Supported Backends (from official docs)

| Backend | Use Case |
|---|---|
| Classpath | Development / testing |
| File System | Server with strict folder security |
| **GitHub/Git** | **Recommended for production** |
| HashiCorp Vault | Secrets management |
| AWS S3 | Cloud storage |
| AWS CodeCommit | AWS-native Git |
| CredHub | Cloud Foundry |
| AWS Secret Manager | AWS secrets |
| JDBC | Store configs in a database |

Official docs: `spring.io → Projects → Spring Cloud → Spring Cloud Config → Reference Doc`

---

### Lec 20 — Encryption & Decryption of Properties in Config Server

#### The Problem

Even with a private GitHub repo, properties like email addresses, passwords, and API keys are stored as **plain text**. Anyone with repo access can read them.

#### Solution: Symmetric Encryption via Config Server

Spring Cloud Config Server can encrypt values before they're stored and decrypt them before sending to microservices.

#### Step 1 — Add `encrypt.key` to Config Server `application.yml`

```yaml
# configserver/src/main/resources/application.yml
encrypt:
  key: "5j8T!mN2#vR6@kL9pQ3wX7uA4hB1yC0d"   # ← Use a complex, random key!
```

> ⚠️ **Never use simple keys** like `password`, `12345`, or `abcdef`. Use a strong random key. Online generators can help.

> In production, store this key as an environment variable instead of in the config file:
> ```bash
> ENCRYPT_KEY=5j8T!mN2#vR6@kL9pQ3wX7uA4hB1yC0d
> ```

#### Step 2 — Encrypt a value using the `/encrypt` API

Once config server is running, it exposes two new endpoints:

```
POST http://localhost:8071/encrypt
Body (raw text): john@eazybank.com

→ Response: a7f8b3c2d1e0f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9
```

#### Step 3 — Store the encrypted value in GitHub with `{cipher}` prefix

In `accounts-prod.yml` on GitHub:

```yaml
# BEFORE (plain text)
accounts:
  contactDetails:
    name: "Product Owner - Prod"
    email: "owner@eazybank.com"    # ← plain text

# AFTER (encrypted)
accounts:
  contactDetails:
    name: "Product Owner - Prod"
    email: "{cipher}a7f8b3c2d1e0f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9"
```

> The `{cipher}` prefix tells Spring Cloud Config Server: "this value is encrypted — decrypt it before returning to clients."

#### How It Works End-to-End

```
GitHub stores:   email = "{cipher}encrypted_value"
                           ↓
Config Server reads it, sees {cipher} prefix
                           ↓
Config Server decrypts using encrypt.key
                           ↓
Microservice receives:   email = "owner@eazybank.com"   (plain text)
```

#### Verify the Config Server Decrypts Correctly

```
GET http://localhost:8071/accounts/prod
→ email field shows decrypted value ✓
```

#### Verify the Microservice Receives Plain Text

```
GET http://localhost:8080/api/contact-info
→ { "contactDetails": { "email": "owner@eazybank.com" } }  ✓ (plain text)
```

#### Decrypt a Value Manually (Testing)

```
POST http://localhost:8071/decrypt
Body (raw text): a7f8b3c2d1e0f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9

→ Response: owner@eazybank.com
```

> **Why is the decrypt endpoint not a security risk in production?**  
> In production, config server is deployed **behind a firewall**. Only internal services can communicate with it. Additionally, you can add Spring Security to the config server to require authentication for all endpoints.

---

## Quick Reference Summary

### Configuration Reading Approaches

```java
// Approach 1 — @Value (1-2 properties)
@Value("${build.version}")
private String buildVersion;

// Approach 2 — Environment (OS env vars)
@Autowired
private Environment environment;  // org.springframework.core.env.Environment
String javaHome = environment.getProperty("JAVA_HOME");

// Approach 3 — @ConfigurationProperties (many grouped properties) ← RECOMMENDED
@ConfigurationProperties(prefix = "accounts")
public record AccountsContactInfoDto(
    String message,
    Map<String, String> contactDetails,
    List<String> onCallSupport
) {}
```

### Profile File Naming Convention

```
application.yml            ← default profile
application_{name}.yml     ← Spring Boot v1 naming  (v1-springboot)
{appname}.yml              ← Config Server naming   (v2-spring-cloud-config)
{appname}-{profile}.yml    ← Config Server profile  (v2-spring-cloud-config)
```

### Startup Order with Config Server

```
1. Config Server (8071)
2. Accounts (8080)
3. Loans (8090)
4. Cards (9000)
```

### Config Server URL Pattern

```
http://localhost:8071/{application-name}/{profile}

Examples:
  http://localhost:8071/accounts/prod
  http://localhost:8071/loans/qa
  http://localhost:8071/cards/default
```

### Full Config Evolution Path

```
application.yml (hardcoded)
  → @Value / Environment / @ConfigurationProperties
  → Spring Boot Profiles (application_prod.yml)
  → Externalizing via CLI/JVM/Env
  → Spring Cloud Config Server
    → Classpath backend
    → File System backend
    → GitHub backend  ← Production recommended
      → With Encryption (encrypt.key + {cipher} prefix)
```

---

*Notes compiled from course lectures 1–20 on Configuration Management in Microservices.*

---

## Section 7 — Runtime Config Refresh

---

### Lec 21 — Refresh Configurations at Runtime Using `/actuator/refresh`

#### The New Problem

Microservices load properties from the config server **only during startup**. If a property changes in GitHub, microservices don't see the new value until they are restarted — which is expensive and disruptive across hundreds of instances.

#### Goal

Refresh properties in a running microservice **without restarting** it.

#### Step 1 — Switch DTO classes from `record` to regular `class`

`record` fields are `final` and cannot be updated after object creation. For runtime refresh to work, Spring needs to call setter methods on the DTO.

**Before (won't work with refresh):**
```java
@ConfigurationProperties(prefix = "accounts")
public record AccountsContactInfoDto(
    String message,
    Map<String, String> contactDetails,
    List<String> onCallSupport
) {}
```

**After (supports runtime refresh):**
```java
// AccountsContactInfoDto.java
import lombok.Getter;
import lombok.Setter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import java.util.List;
import java.util.Map;

@ConfigurationProperties(prefix = "accounts")
@Getter
@Setter    // ← Setter required so Spring can update fields at runtime
public class AccountsContactInfoDto {
    private String message;
    private Map<String, String> contactDetails;
    private List<String> onCallSupport;
}
```

Apply the same change to `LoansContactInfoDto` and `CardsContactInfoDto` (with their respective prefixes `loans` and `cards`).

#### Step 2 — Expose the actuator `refresh` endpoint in `application.yml`

Add to `application.yml` of **all three microservices**:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"    # Expose ALL actuator endpoints (includes /refresh, /bus-refresh, /health, etc.)
```

> Using `"*"` is convenient during development. In production, restrict to only the endpoints you need.

#### Step 3 — Invoke the refresh endpoint

After changing a property in GitHub, call the refresh endpoint on each microservice:

```
POST http://localhost:8080/actuator/refresh    (accounts)
POST http://localhost:8090/actuator/refresh    (loans)
POST http://localhost:9000/actuator/refresh    (cards)
```

- No request body needed.
- Response lists the property keys that were refreshed:
  ```json
  ["accounts.message", "config.client.version"]
  ```

#### How It Works

```
1. Developer pushes change to GitHub repo
2. Config server always reads latest from GitHub (no local cache)
3. Developer POSTs to microservice /actuator/refresh
4. Microservice calls config server to fetch changed properties
5. Config server pulls latest from GitHub
6. Microservice updates in-memory bean values without restart
```

#### Limitation

With 500 running instances (100 microservices × 5 each), you would need to invoke `/actuator/refresh` **500 times**. That's still manual and not scalable.

---

### Lec 22 — Refresh Configurations at Runtime Using Spring Cloud Bus

#### Solution: Spring Cloud Bus + RabbitMQ

Spring Cloud Bus links all microservice nodes via a **lightweight message broker** (RabbitMQ or Kafka). Invoking `/actuator/busrefresh` on **any one instance** propagates the refresh to **all registered instances** automatically.

```
                     ┌─────────────┐
                     │   RabbitMQ  │ ← Message Broker
                     └──────┬──────┘
               ┌────────────┼────────────┐
               ▼            ▼            ▼
           Accounts       Loans        Cards
           Instance 1   Instance 2  Instance 3
           (POST /busrefresh)
                 └── broadcasts → all others refresh automatically
```

#### Step 1 — Start RabbitMQ via Docker

```bash
docker run -it --rm --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  rabbitmq:3.13-management
```

- Port `5672` — core messaging
- Port `15672` — management UI (`http://localhost:15672`, guest/guest)

#### Step 2 — Add Spring Cloud Bus AMQP dependency to ALL services (including config server)

```xml
<!-- pom.xml of accounts, loans, cards, AND configserver -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

> `amqp` = Advanced Message Queuing Protocol (used by RabbitMQ)

#### Step 3 — Add RabbitMQ connection details to `application.yml` of all microservices

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

> These are RabbitMQ's default values. Spring Boot uses them automatically even without explicit config, but it's good practice to define them explicitly.

#### Step 4 — Invoke bus-refresh on ANY ONE instance

```
POST http://localhost:8080/actuator/busrefresh
```

- Response: `204 No Content` (success, no body)
- Behind the scenes: Spring Cloud Bus broadcasts a config-change event to the message broker, which notifies ALL registered instances to refresh their properties from the config server.

#### Verification

```
# Before bus-refresh:
GET http://localhost:9000/api/contact-info  → "prod APIs"

# After invoking bus-refresh on accounts only:
GET http://localhost:9000/api/contact-info  → "prod APIs" updated automatically ✓
GET http://localhost:8090/api/contact-info  → updated automatically ✓
```

#### Spring Cloud Bus Flow

```
1. Push property change to GitHub
2. POST /actuator/busrefresh to any ONE microservice instance
3. That instance notifies config server of the change event via RabbitMQ
4. RabbitMQ broadcasts the event to ALL registered microservice instances
5. All instances call config server and reload changed properties (no restart)
```

#### Remaining Limitation

Someone still has to invoke `/actuator/busrefresh` at least once. This can be automated via CI/CD, but it's still a manual trigger.

---

### Lec 23 — Fully Automated Refresh Using Spring Cloud Bus + Config Monitor + GitHub Webhook

#### The Goal: Zero Manual Steps

Whenever a property is pushed to GitHub, all microservices automatically refresh — with no human action required.

#### How It Works

```
Developer pushes to GitHub
       ↓
GitHub Webhook fires a POST to /monitor on Config Server
       ↓
Config Server triggers a bus-refresh event via RabbitMQ
       ↓
All microservice instances automatically refresh their properties
```

#### Step 1 — Add `spring-cloud-config-monitor` to config server pom.xml ONLY

```xml
<!-- configserver/pom.xml ONLY — not in microservices -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-monitor</artifactId>
</dependency>
```

This exposes a new REST endpoint `/monitor` on the config server (not under `/actuator`).

#### Step 2 — Enable actuator endpoints in config server `application.yml`

```yaml
# configserver/src/main/resources/application.yml
management:
  endpoints:
    web:
      exposure:
        include: "*"

spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

#### Step 3 — Create a GitHub Webhook

In your GitHub config repo → **Settings → Webhooks → Add webhook**:

| Field | Value |
|---|---|
| Payload URL | `https://<public-url>/monitor` |
| Content type | `application/json` |
| Secret | (empty) |
| Events | **Just the push event** |

> ⚠️ GitHub cannot call `localhost:8071/monitor` directly. You need a public-facing URL. Use **hookdeck.com** for local development:

```bash
# Install hookdeck CLI
brew install hookdeck/hookdeck/hookdeck

# Login
hookdeck login

# Create a tunnel to your local port 8071 at path /monitor
hookdeck listen 8071 /monitor
# → Provides a public URL like: https://events.hookdeck.com/e/src_xxx
# Use this URL as the GitHub webhook payload URL
```

#### Complete Automated Flow

```
1. Developer edits a property file in GitHub repo and commits
2. GitHub detects the push event and sends a POST to webhook URL
3. Hookdeck (or production reverse proxy) forwards request to localhost:8071/monitor
4. Config server /monitor endpoint receives the push notification
5. Config server triggers Spring Cloud Bus refresh event via RabbitMQ
6. RabbitMQ broadcasts to all microservice instances
7. All microservices fetch latest properties from config server automatically
```

No manual API call is needed.

#### Steps Summary

| Step | Action | Where |
|---|---|---|
| 1 | Add `spring-boot-starter-actuator` | All services |
| 2 | Enable `management.endpoints.web.exposure.include=*` | All services |
| 3 | Add `spring-cloud-starter-bus-amqp` | All services (incl. config server) |
| 4 | Add `spring-cloud-config-monitor` | Config server only |
| 5 | Start RabbitMQ | Docker / local |
| 6 | Create GitHub webhook → POST to `/monitor` | GitHub repo settings |

---

## Section 8 — Docker Compose Integration

---

### Lec 24 — Updating Docker Compose File for Config Server — Part 1

#### Project Structure for Docker Compose

```
v2-spring-cloud-config/
  docker-compose/
    default/
      docker-compose.yml
      common-config.yml
    qa/
      docker-compose.yml
      common-config.yml
    prod/
      docker-compose.yml
      common-config.yml
```

One Docker Compose file per environment, each activating a different Spring profile.

#### Initial `docker-compose.yml` (default folder)

```yaml
services:
  configserver:
    image: "eazybytes/configserver:S6"
    container_name: configserver-ms
    ports:
      - "8071:8071"
    deploy:
      resources:
        limits:
          memory: 700m
    networks:
      - eazybank

  accounts:
    image: "eazybytes/accounts:S6"
    container_name: accounts-ms
    ports:
      - "8080:8080"
    environment:
      SPRING_APPLICATION_NAME: "accounts"
      SPRING_PROFILES_ACTIVE: default
      SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071/"
    deploy:
      resources:
        limits:
          memory: 700m
    networks:
      - eazybank

  loans:
    image: "eazybytes/loans:S6"
    container_name: loans-ms
    ports:
      - "8090:8090"
    environment:
      SPRING_APPLICATION_NAME: "loans"
      SPRING_PROFILES_ACTIVE: default
      SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071/"
    deploy:
      resources:
        limits:
          memory: 700m
    networks:
      - eazybank

  cards:
    image: "eazybytes/cards:S6"
    container_name: cards-ms
    ports:
      - "9000:9000"
    environment:
      SPRING_APPLICATION_NAME: "cards"
      SPRING_PROFILES_ACTIVE: default
      SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071/"
    deploy:
      resources:
        limits:
          memory: 700m
    networks:
      - eazybank

networks:
  eazybank:
    driver: bridge
```

#### Key Points

**Why `configserver:http://configserver:8071/` (not `localhost`)?**

Inside Docker, each container runs in its own isolated network. `localhost` inside the accounts container refers to accounts itself, not the config server. Use the **Docker service name** (`configserver`) as the hostname — all containers on the same Docker network can resolve each other by service name.

**Why `SPRING_APPLICATION_NAME` in Docker Compose despite being in `application.yml`?**

A known bug in Spring Cloud Config Server at time of recording requires the application name to also be provided via environment variable for correct property resolution in Docker.

**Why remove `optional:` from `SPRING_CONFIG_IMPORT`?**

In Docker/production environments, the app **must** connect to the config server. Remove `optional:` so the container fails fast and clearly if the config server is unreachable.

---

### Lec 25 — Introduction to Liveness and Readiness Probes

#### Why Probes Are Needed

Before accounts/loans/cards containers start, the config server must be fully ready. Docker needs a way to determine "is the config server ready?"

#### Liveness Probe

> "Is this container alive?" — TRUE/FALSE

- `UP`: Container is alive, no action needed.
- `DOWN`: Container is dead → Kubernetes/Docker restarts or replaces it.

Analogy: A boxer sitting in the corner — alive but maybe not yet ready to fight.

#### Readiness Probe

> "Is this container ready to accept traffic?" — TRUE/FALSE

- `UP`: Container is ready to receive requests.
- `DOWN`: Container is alive but still starting up — don't send traffic yet.

Analogy: The boxer standing up in the ring — both alive AND ready to fight.

#### Spring Boot Actuator Endpoints

```
GET /actuator/health           → overall health (includes liveness + readiness + other indicators)
GET /actuator/health/liveness  → liveness state only
GET /actuator/health/readiness → readiness state only
```

#### Enable Liveness & Readiness in Config Server `application.yml`

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      probes:
        enabled: true       # ← enable liveness/readiness endpoints
  health:
    readiness-state:
      enabled: true
    liveness-state:
      enabled: true
```

#### Test the Probes

```
GET http://localhost:8071/actuator/health/liveness
→ { "status": "UP" }

GET http://localhost:8071/actuator/health/readiness
→ { "status": "UP" }
```

> If RabbitMQ is not running, `/actuator/health` returns `"status": "DOWN"` but `/actuator/health/readiness` may still return `"UP"` (RabbitMQ is optional for config server startup).

---

### Lec 26 — Updating Docker Compose File for Config Server — Part 2

#### Adding Health Check for Config Server

```yaml
configserver:
  image: "eazybytes/configserver:S6"
  container_name: configserver-ms
  ports:
    - "8071:8071"
  healthcheck:
    test: "curl --fail --silent localhost:8071/actuator/health/readiness | grep UP || exit 1"
    interval: 10s      # check every 10 seconds
    timeout: 5s        # each check must respond within 5 seconds
    retries: 10        # retry up to 10 times before marking unhealthy
    start_period: 10s  # wait 10 seconds before starting health checks
  networks:
    - eazybank
```

#### Adding `depends_on` with `condition: service_healthy`

```yaml
accounts:
  image: "eazybytes/accounts:S6"
  depends_on:
    configserver:
      condition: service_healthy   # ← wait until configserver health check passes
  environment:
    SPRING_APPLICATION_NAME: "accounts"
    SPRING_PROFILES_ACTIVE: default
    SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071/"
  networks:
    - eazybank
```

Apply `depends_on: configserver: condition: service_healthy` to all of accounts, loans, and cards.

| `condition` value | Behavior |
|---|---|
| `service_started` | Waits only until container process starts (not recommended here) |
| `service_healthy` | Waits until health check passes ✓ |
| `service_completed_successfully` | Waits until service exits with code 0 (for one-shot tasks) |

#### Adding RabbitMQ Service

```yaml
rabbit:
  image: rabbitmq:3.13-management
  hostname: rabbitmq
  ports:
    - "5672:5672"
    - "15672:15672"
  healthcheck:
    test: rabbitmq-diagnostics check_port_connectivity
    interval: 10s
    timeout: 5s
    retries: 10
    start_period: 5s
  networks:
    - eazybank
```

#### Config Server `depends_on` RabbitMQ

```yaml
configserver:
  depends_on:
    rabbit:
      condition: service_healthy
```

#### Startup Order

```
1. RabbitMQ starts → health check passes
2. Config Server starts → health check passes
3. Accounts, Loans, Cards start in parallel (all depend on configserver being healthy)
```

> Accounts/Loans/Cards also implicitly depend on RabbitMQ because Config Server depends on it — no need to list RabbitMQ again under their `depends_on`.

---

### Lec 27 — Optimizing Docker Compose File

#### Problem: Repetitive Configuration

The `docker-compose.yml` has duplicated `deploy`, `networks`, `SPRING_PROFILES_ACTIVE`, `SPRING_CONFIG_IMPORT`, and `depends_on` blocks across all services.

#### Solution: Extract to `common-config.yml`

Create `common-config.yml` alongside `docker-compose.yml`:

```yaml
# common-config.yml
services:

  network-deploy-service:
    networks:
      - eazybank

  microservice-base-config:
    extends:
      service: network-deploy-service
    deploy:
      resources:
        limits:
          memory: 700m

  microservice-configserver-config:
    extends:
      service: microservice-base-config
    environment:
      SPRING_PROFILES_ACTIVE: default
      SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071/"
      SPRING_RABBITMQ_HOST: "rabbit"        # ← override localhost with Docker service name
    depends_on:
      configserver:
        condition: service_healthy
```

#### Cleaned-up `docker-compose.yml`

```yaml
services:

  rabbit:
    image: rabbitmq:3.13-management
    hostname: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    healthcheck:
      test: rabbitmq-diagnostics check_port_connectivity
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 5s
    extends:
      file: common-config.yml
      service: network-deploy-service      # ← only needs network, not Spring config

  configserver:
    image: "eazybytes/configserver:S6"
    container_name: configserver-ms
    ports:
      - "8071:8071"
    healthcheck:
      test: "curl --fail --silent localhost:8071/actuator/health/readiness | grep UP || exit 1"
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 10s
    extends:
      file: common-config.yml
      service: microservice-base-config    # ← network + deploy, no Spring profile needed
    depends_on:
      rabbit:
        condition: service_healthy

  accounts:
    image: "eazybytes/accounts:S6"
    container_name: accounts-ms
    ports:
      - "8080:8080"
    environment:
      SPRING_APPLICATION_NAME: "accounts"
    extends:
      file: common-config.yml
      service: microservice-configserver-config  # ← network + deploy + profiles + config import + rabbitmq + depends_on

  loans:
    image: "eazybytes/loans:S6"
    container_name: loans-ms
    ports:
      - "8090:8090"
    environment:
      SPRING_APPLICATION_NAME: "loans"
    extends:
      file: common-config.yml
      service: microservice-configserver-config

  cards:
    image: "eazybytes/cards:S6"
    container_name: cards-ms
    ports:
      - "9000:9000"
    environment:
      SPRING_APPLICATION_NAME: "cards"
    extends:
      file: common-config.yml
      service: microservice-configserver-config

networks:
  eazybank:
    driver: bridge
```

#### Benefits of This Structure

- Change network name in **one place** in `common-config.yml`.
- Change memory limits in **one place**.
- Change config server URL in **one place**.
- Each service definition is short and focused only on what's unique to it.

---

### Lec 28 — Generating Docker Images and Pushing to Docker Hub

#### Add Google Jib Plugin to Config Server `pom.xml`

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
    <!-- Add Jib plugin -->
    <plugin>
      <groupId>com.google.cloud.tools</groupId>
      <artifactId>jib-maven-plugin</artifactId>
      <configuration>
        <to>
          <image>eazybytes/${project.artifactId}:S6</image>
        </to>
      </configuration>
    </plugin>
  </plugins>
</build>
```

#### Build Docker Images with Jib

Run from the root of each project folder:

```bash
# From accounts/ folder
mvn compile jib:dockerBuild

# From cards/ folder
cd ../cards && mvn compile jib:dockerBuild

# From loans/ folder
cd ../loans && mvn compile jib:dockerBuild

# From configserver/ folder
cd ../configserver && mvn compile jib:dockerBuild
```

This creates local Docker images:
- `eazybytes/accounts:S6`
- `eazybytes/cards:S6`
- `eazybytes/loans:S6`
- `eazybytes/configserver:S6`

#### Push Images to Docker Hub

```bash
docker image push docker.io/eazybytes/accounts:S6
docker image push docker.io/eazybytes/loans:S6
docker image push docker.io/eazybytes/cards:S6
docker image push docker.io/eazybytes/configserver:S6
```

> Make sure you are logged in: `docker login`

---

### Lec 29 — End-to-End Testing with Docker Compose (Default Profile)

#### Start All Containers

```bash
cd docker-compose/default
docker compose up -d
```

#### Verify Containers are Running

```bash
docker ps
```

Expected output:

| Container | Status |
|---|---|
| rabbitmq | healthy |
| configserver-ms | healthy |
| accounts-ms | started |
| loans-ms | started |
| cards-ms | started |

#### Test the APIs

```
GET http://localhost:8080/api/build-info    → "3.0"  (default profile)
GET http://localhost:8080/api/contact-info  → default profile properties
GET http://localhost:8090/api/contact-info  → loans default properties
GET http://localhost:9000/api/contact-info  → cards default properties
```

#### Fix: RabbitMQ Host Must Be `rabbit` Inside Docker

A common problem when first running with Docker Compose is that the config server health shows `DOWN` while readiness shows `UP`. This happens because the config server is trying to connect to `rabbitmq` at `localhost` instead of the Docker service name.

**Fix in `common-config.yml`:**

```yaml
microservice-base-config:
  environment:
    SPRING_RABBITMQ_HOST: "rabbit"   # ← service name, not localhost
```

Then restart:

```bash
docker compose down
docker compose up -d
```

#### Test Automatic Config Refresh End-to-End

1. Start hook deck tunnel:
   ```bash
   hookdeck listen 8071 /monitor
   # → copy the provided public URL
   ```

2. Update GitHub webhook payload URL to the hookdeck URL.

3. Edit a property in `accounts.yml` on GitHub (e.g., change `local` → `docker`).

4. Commit the change.

5. GitHub sends a webhook → hookdeck forwards to `localhost:8071/monitor` → config server triggers bus-refresh via RabbitMQ → all containers update.

6. Test without restarting:
   ```
   GET http://localhost:8080/api/contact-info
   → message now shows "docker" ✓
   ```

#### Stop Containers

```bash
docker compose down
```

---

### Lec 30 — Preparing Docker Compose Files for QA & Prod Profiles

#### How Easy It Is

Simply copy the two files from `default/` into `qa/` and `prod/` folders:

```
docker-compose/
  default/
    docker-compose.yml     ← original
    common-config.yml      ← original
  qa/
    docker-compose.yml     ← copied from default (unchanged)
    common-config.yml      ← one change only
  prod/
    docker-compose.yml     ← copied from default (unchanged)
    common-config.yml      ← one change only
```

#### Change in `qa/common-config.yml`

```yaml
# Only one line changes:
SPRING_PROFILES_ACTIVE: "qa"    # was "default"
```

#### Change in `prod/common-config.yml`

```yaml
# Only one line changes:
SPRING_PROFILES_ACTIVE: "prod"  # was "default"
```

#### Run in Prod Profile

```bash
cd docker-compose/prod
docker compose up -d
```

All containers start with `spring.profiles.active=prod`, loading properties from `accounts-prod.yml`, `loans-prod.yml`, `cards-prod.yml` in the GitHub config repo.

#### Test in Prod

```
GET http://localhost:8080/api/build-info    → "1.0"   (prod profile)
GET http://localhost:8080/api/contact-info  → prod profile properties
```

Edit `cards-prod.yml` in GitHub → webhook fires → all containers refresh automatically within seconds.

#### The Power of Immutable Docker Images

```
Same Docker image:  eazybytes/accounts:S6
  + default/common-config.yml  → runs as dev
  + qa/common-config.yml       → runs as QA
  + prod/common-config.yml     → runs as prod
```

One build artifact, deployed to all environments with no rebuilding.

---

## Updated Quick Reference

### All Refresh Approaches Compared

| Approach | Steps | Manual Work | Scales? |
|---|---|---|---|
| Restart microservice | Rebuild Docker image | Full restart | ✗ |
| `/actuator/refresh` | Add actuator, expose endpoint | POST to each instance | ✗ (500 calls) |
| `/actuator/busrefresh` | + RabbitMQ + spring-cloud-bus | POST to ONE instance | ✓ (one call) |
| GitHub Webhook + /monitor | + spring-cloud-config-monitor + webhook | Push to GitHub only | ✓✓ (fully automatic) |

### Docker Compose Service Startup Order

```
rabbit (health check)
  ↓ (service_healthy)
configserver (health check: /actuator/health/readiness)
  ↓ (service_healthy)
accounts + loans + cards (start in parallel)
```

### Environment Variable Naming in Docker Compose

```
Spring property         → Docker environment variable
spring.profiles.active  → SPRING_PROFILES_ACTIVE
spring.config.import    → SPRING_CONFIG_IMPORT
spring.application.name → SPRING_APPLICATION_NAME
spring.rabbitmq.host    → SPRING_RABBITMQ_HOST
```

Rules: UPPERCASE + replace `.` with `_`

### Docker Commands Summary

```bash
# Build Docker image with Jib
mvn compile jib:dockerBuild

# Push to Docker Hub
docker image push docker.io/eazybytes/<service>:S6

# Start all containers (detached)
docker compose up -d

# View running containers
docker ps

# Stop and remove all containers
docker compose down

# View logs of a specific container
docker logs configserver-ms
```

---

*Notes compiled from course lectures 1–30 on Configuration Management in Microservices.*
