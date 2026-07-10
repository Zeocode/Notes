# MySQL Database Integration with Microservices
## Study Notes — Section 7
### 5 Lectures | 1 Section

---

## Table of Contents

- [Section 7 — MySQL Database Integration](#section-7--mysql-database-integration)
  - [Lec 1 — Create MySQL DB Containers for Microservices](#lec-1--create-mysql-db-containers-for-microservices)
  - [Lec 2 — Update Microservices Code to Replace H2 DB with MySQL](#lec-2--update-microservices-code-to-replace-h2-db-with-mysql-db)
  - [Lec 3 — Update Docker Compose File to Create & Use MySQL DB](#lec-3--update-docker-compose-file-to-create--use-mysql-db)
  - [Lec 4 — Running Microservices & MySQL DB Containers via Docker Compose](#lec-4--running-microservices--mysql-db-containers-using-docker-compose)
  - [Lec 5 — Demo of Docker Network Concept](#lec-5--demo-of-docker-network-concept)

---

## Section 7 — MySQL Database Integration

> **Context**: This section takes a break from microservice pattern challenges to migrate all three microservices (accounts, loans, cards) from the embedded **H2 database** to a real **MySQL database**. Spring Cloud Bus / RabbitMQ dependencies are also removed to reduce container overhead during development.

---

### Lec 1 — Create MySQL DB Containers for Microservices

#### Why MySQL Instead of H2?

| Database | Use Case |
|---|---|
| H2 (in-memory) | Quick local development — not for production |
| MySQL | Real relational database — production-ready |

H2 is embedded and auto-configured by Spring Boot, but it resets on every restart and is not recommended for real projects.

#### Why Docker Instead of Manual MySQL Installation?

With Docker you can:
- Spin up a MySQL database **in seconds** with a single command.
- Avoid installing software on your local machine.
- Stop or delete the container when not needed.
- Run **multiple isolated MySQL instances** on the same machine.

#### Cleanup: Remove Spring Cloud Bus Dependencies First

Before adding MySQL, remove RabbitMQ-related dependencies from all services to reduce container count and system load:

**In `configserver/pom.xml`** — remove:
```xml
<dependency>spring-cloud-starter-bus-amqp</dependency>
<dependency>spring-cloud-config-monitor</dependency>
```

**In `accounts/`, `loans/`, `cards/` `pom.xml`** — remove:
```xml
<dependency>spring-cloud-starter-bus-amqp</dependency>
```

**In each microservice `application.yml`** — remove:
```yaml
spring:
  rabbitmq:       # ← delete this entire block
    host: localhost
    port: 5672
    username: guest
    password: guest
```

> Keep the actuator management endpoints and health probe properties — those are still needed.

---

#### Step-by-Step: Create MySQL Containers with Docker

##### Accounts Microservice Database

```bash
docker run \
  -p 3306:3306 \
  --name accountsDB \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=accountsdb \
  -d \
  mysql
```

| Flag | Meaning |
|---|---|
| `-p 3306:3306` | Expose container port 3306 to local port 3306 |
| `--name accountsDB` | Container name |
| `-e MYSQL_ROOT_PASSWORD=root` | Root user password |
| `-e MYSQL_DATABASE=accountsdb` | Auto-create this schema on startup |
| `-d` | Run in detached (background) mode |
| `mysql` | Docker image name |

##### Loans Microservice Database

```bash
docker run \
  -p 3307:3306 \
  --name loansDB \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=loansdb \
  -d \
  mysql
```

> `3307:3306` — maps host port `3307` to container's internal port `3306`  
> Inside the container, MySQL always listens on `3306` regardless of the host port.

##### Cards Microservice Database

```bash
docker run \
  -p 3308:3306 \
  --name cardsDB \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=cardsdb \
  -d \
  mysql
```

#### Why Different Host Ports?

Each container runs MySQL on port `3306` internally (their own isolated network), but they must be exposed on **different host ports** so your local system can reach each one:

```
Host 3306 → accountsDB container (internal: 3306)
Host 3307 → loansDB container    (internal: 3306)
Host 3308 → cardsDB container    (internal: 3306)
```

#### Connecting with SQL Electron (GUI Client)

**SqlElectron** is a lightweight, free DB GUI client. Download from `sqlelectron.github.io`.

| Field | accountsDB | loansDB | cardsDB |
|---|---|---|---|
| Server | localhost | localhost | localhost |
| Port | 3306 | 3307 | 3308 |
| Username | root | root | root |
| Password | root | root | root |
| Database | accountsdb | loansdb | cardsdb |

> The databases are empty at this point — no tables yet. Tables are created during microservice startup via `schema.sql`.

#### One Best Practice Per Microservice

Each microservice has its own dedicated database — this is the **microservice database isolation** best practice. No two services share a database.

---

### Lec 2 — Update Microservices Code to Replace H2 DB with MySQL DB

#### Step 1 — Update `pom.xml` in All Three Microservices

Remove H2 dependency and add MySQL connector:

```xml
<!-- REMOVE: H2 -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- ADD: MySQL -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
```

Apply to: `accounts/pom.xml`, `loans/pom.xml`, `cards/pom.xml`  
Config server has no database — skip it.

---

#### Step 2 — Update `application.yml` in Accounts Microservice

Replace H2 datasource config with MySQL config:

```yaml
# accounts/src/main/resources/application.yml

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/accountsdb
    username: root
    password: root
    sql:
      init:
        mode: always    # ← always run schema.sql on startup
  jpa:
    show-sql: true
```

> **Key note**: The `sql.init.mode: always` property must be under `spring.datasource`, **not** under `spring.jpa`. Common mistake!

**Property key:** `spring.datasource.sql.init.mode=always`

#### Step 3 — Update `application.yml` in Loans Microservice

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3307/loansdb
    username: root
    password: root
    sql:
      init:
        mode: always
  jpa:
    show-sql: true
```

#### Step 4 — Update `application.yml` in Cards Microservice

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3308/cardsdb
    username: root
    password: root
    sql:
      init:
        mode: always
  jpa:
    show-sql: true
```

#### Why `sql.init.mode: always`?

With **H2**, Spring Boot automatically runs `schema.sql` at startup.  
With **MySQL** (or any real DB), it does NOT — unless you tell it to with `mode: always`.

The `schema.sql` scripts use `CREATE TABLE IF NOT EXISTS`, so they are safe to re-run:

```sql
-- Example from accounts schema.sql
CREATE TABLE IF NOT EXISTS `customer` (
  `customer_id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(100) NOT NULL,
  `email` varchar(100) NOT NULL,
  `mobile_number` varchar(20) NOT NULL,
  PRIMARY KEY (`customer_id`)
);
```

The `IF NOT EXISTS` clause prevents errors when the table already exists from a previous run.

#### Important: Do NOT Hardcode Production Credentials

For local development, hardcoding `root/root` in `application.yml` is acceptable. In real environments:

| Environment | How to provide DB credentials |
|---|---|
| Docker Compose | Environment variables in `docker-compose.yml` |
| Kubernetes | ConfigMaps and Secrets |
| CI/CD | CLI arguments / JVM system properties |

Never hardcode production passwords in source code.

#### Step 5 — Start and Verify

Start services in order:
1. Config Server
2. Accounts → Loans → Cards microservices

Verify in SqlElectron:
- `accountsdb` → tables: `customer`, `accounts`
- `loansdb` → table: `loans`
- `cardsdb` → table: `cards`

#### Data Persistence Behaviour with Docker Containers

| Action | Data Status |
|---|---|
| Stop container (not deleted) | Data **preserved** |
| Start stopped container | Data **available** again |
| Delete container | Data **permanently lost** |

> If you delete a MySQL container, you must recreate it with `docker run` and all tables will be empty.

---

### Lec 3 — Update Docker Compose File to Create & Use MySQL DB

#### Why `localhost` Doesn't Work in Docker Compose

Inside Docker, each container runs in its own isolated network. When the accounts microservice container tries to connect to `localhost:3306`, it tries to find MySQL **inside its own container** — which doesn't exist.

```
❌ WRONG (inside Docker):
  jdbc:mysql://localhost:3306/accountsdb

✅ CORRECT (inside Docker — use service name):
  jdbc:mysql://accountsdb:3306/accountsdb
```

The service name (`accountsdb`) resolves to the correct container IP automatically when all containers are on the same Docker network.

#### Step 1 — Rebuild Docker Images with Tag S7

Update the image tag in all microservice `pom.xml` files:

```xml
<!-- Change S6 → S7 in all pom.xml Jib plugin configs -->
<to>
    <image>eazybytes/${project.artifactId}:S7</image>
</to>
```

Rebuild all images:

```bash
# From each project folder:
cd accounts  && mvn compile jib:dockerBuild
cd ../loans  && mvn compile jib:dockerBuild
cd ../cards  && mvn compile jib:dockerBuild
cd ../configserver && mvn compile jib:dockerBuild
```

#### Step 2 — Add Database Service Entries to `common-config.yml`

Create a new reusable service for all MySQL containers:

```yaml
# docker-compose/default/common-config.yml

services:

  # Existing services...
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

  # NEW: shared config for all MySQL DB containers
  microservice-db-config:
    extends:
      service: network-deploy-service
    image: mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      timeout: 10s
      retries: 10
      interval: 10s
      start_period: 10s
    environment:
      MYSQL_ROOT_PASSWORD: "root"

  microservice-configserver-config:
    extends:
      service: microservice-base-config
    environment:
      SPRING_PROFILES_ACTIVE: default
      SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071/"
    depends_on:
      configserver:
        condition: service_healthy

```

#### Step 3 — Add Database Services to `docker-compose.yml`

```yaml
# docker-compose/default/docker-compose.yml

services:

  accountsdb:
    container_name: accountsdb
    ports:
      - "3306:3306"
    environment:
      MYSQL_DATABASE: "accountsdb"
    extends:
      file: common-config.yml
      service: microservice-db-config

  loansdb:
    container_name: loansdb
    ports:
      - "3307:3306"
    environment:
      MYSQL_DATABASE: "loansdb"
    extends:
      file: common-config.yml
      service: microservice-db-config

  cardsdb:
    container_name: cardsdb
    ports:
      - "3308:3306"
    environment:
      MYSQL_DATABASE: "cardsdb"
    extends:
      file: common-config.yml
      service: microservice-db-config

  configserver:
    image: "eazybytes/configserver:S7"
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
      service: microservice-base-config

  accounts:
    image: "eazybytes/accounts:S7"
    container_name: accounts-ms
    ports:
      - "8080:8080"
    environment:
      SPRING_APPLICATION_NAME: "accounts"
      SPRING_DATASOURCE_URL: "jdbc:mysql://accountsdb:3306/accountsdb"
    depends_on:
      accountsdb:
        condition: service_healthy
    extends:
      file: common-config.yml
      service: microservice-configserver-config

  loans:
    image: "eazybytes/loans:S7"
    container_name: loans-ms
    ports:
      - "8090:8090"
    environment:
      SPRING_APPLICATION_NAME: "loans"
      SPRING_DATASOURCE_URL: "jdbc:mysql://loansdb:3306/loansdb"
    depends_on:
      loansdb:
        condition: service_healthy
    extends:
      file: common-config.yml
      service: microservice-configserver-config

  cards:
    image: "eazybytes/cards:S7"
    container_name: cards-ms
    ports:
      - "9000:9000"
    environment:
      SPRING_APPLICATION_NAME: "cards"
      SPRING_DATASOURCE_URL: "jdbc:mysql://cardsdb:3306/cardsdb"
    depends_on:
      cardsdb:
        condition: service_healthy
    extends:
      file: common-config.yml
      service: microservice-configserver-config

networks:
  eazybank:
    driver: bridge
```

#### Step 4 — Add Shared DB Credentials to `common-config.yml`

Since `root/root` is the same for all microservices, put it in the shared config:

```yaml
# In microservice-configserver-config under common-config.yml
microservice-configserver-config:
  extends:
    service: microservice-base-config
  environment:
    SPRING_PROFILES_ACTIVE: default
    SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071/"
    SPRING_DATASOURCE_USERNAME: "root"      # ← shared for all microservices
    SPRING_DATASOURCE_PASSWORD: "root"      # ← shared for all microservices
  depends_on:
    configserver:
      condition: service_healthy
```

> Only `SPRING_DATASOURCE_URL` is unique per microservice (different service name + port + DB name), so it stays in `docker-compose.yml` per-service.

#### Port Mapping Rule Clarification

```
For containers communicating WITH EACH OTHER (inside Docker network):
  Use internal port: 3306  ← MySQL's actual port inside the container

For tools OUTSIDE Docker (SqlElectron on your laptop):
  Use the exposed host port: 3307, 3308, etc.
```

```
# WRONG — microservice container connecting to loansdb:
SPRING_DATASOURCE_URL: jdbc:mysql://loansdb:3307/loansdb  ← 3307 is the HOST port

# CORRECT — microservice container connecting to loansdb:
SPRING_DATASOURCE_URL: jdbc:mysql://loansdb:3306/loansdb  ← 3306 is the INTERNAL port
```

#### Remove RabbitMQ `depends_on` from Config Server

Since Spring Cloud Bus was removed, delete the RabbitMQ dependency from config server in `docker-compose.yml`:

```yaml
# REMOVE this from configserver:
depends_on:
  rabbit:            # ← rabbit service no longer exists
    condition: service_healthy
```

#### Startup Order with MySQL

```
accountsdb  (healthcheck: mysqladmin ping)
    ↓ (service_healthy)
loansdb     (healthcheck: mysqladmin ping)
    ↓ (service_healthy)
cardsdb     (healthcheck: mysqladmin ping)
    ↓ (service_healthy)
configserver (healthcheck: actuator/health/readiness)
    ↓ (service_healthy)
accounts + loans + cards (start in parallel, each depends on its own DB + configserver)
```

---

### Lec 4 — Running Microservices & MySQL DB Containers Using Docker Compose

#### Run Docker Compose

```bash
cd docker-compose/default
docker compose up -d
```

#### Expected Running Containers (7 total)

| Container | Image | Port | Role |
|---|---|---|---|
| accountsdb | mysql | 3306 | Accounts DB |
| loansdb | mysql | 3307 | Loans DB |
| cardsdb | mysql | 3308 | Cards DB |
| configserver-ms | eazybytes/configserver:S7 | 8071 | Config Server |
| accounts-ms | eazybytes/accounts:S7 | 8080 | Accounts API |
| loans-ms | eazybytes/loans:S7 | 8090 | Loans API |
| cards-ms | eazybytes/cards:S7 | 9000 | Cards API |

#### Common Error: Wrong Port in `SPRING_DATASOURCE_URL`

```
Error: Unable to connect to database
```

**Cause**: Using the host-exposed port (`3307`) instead of the internal Docker port (`3306`) in the datasource URL.

```yaml
# WRONG:
SPRING_DATASOURCE_URL: "jdbc:mysql://loansdb:3307/loansdb"

# CORRECT:
SPRING_DATASOURCE_URL: "jdbc:mysql://loansdb:3306/loansdb"
```

**Why?** Inside the Docker network, containers talk to each other using their **internal ports**. The host port (`3307`) only matters for tools running outside Docker (like SqlElectron on your laptop).

#### Testing the APIs via Postman

```
POST http://localhost:8080/api/create  → Create Account ✓
POST http://localhost:9000/api/create  → Create Card ✓
POST http://localhost:8090/api/create  → Create Loan ✓
```

#### Verifying Data in SqlElectron

| Connection | Port | Table | Has Data? |
|---|---|---|---|
| LocalAccountsDB | 3306 | accounts, customer | ✓ |
| LocalLoansDB | 3307 | loans | ✓ |
| LocalCardsDB | 3308 | cards | ✓ |

#### Setup for QA and Prod Docker Compose Files

Copy `docker-compose.yml` and `common-config.yml` from `default/` into `qa/` and `prod/` folders.

**Only one change needed in each `common-config.yml`:**

```yaml
# qa/common-config.yml
SPRING_PROFILES_ACTIVE: "qa"      # was "default"

# prod/common-config.yml
SPRING_PROFILES_ACTIVE: "prod"    # was "default"
```

All other content (DB services, credentials, URLs) is identical across environments.

#### External Database (Not in Docker)

If your team uses a cloud-hosted or dev-server database instead of Docker containers:

1. Remove the `accountsdb`, `loansdb`, `cardsdb` services from `docker-compose.yml`.
2. Update `SPRING_DATASOURCE_URL` per microservice to point to the actual hostname/IP of the external DB.

```yaml
accounts:
  environment:
    SPRING_DATASOURCE_URL: "jdbc:mysql://dev-db.company.com:3306/accountsdb"
    SPRING_DATASOURCE_USERNAME: "dev_user"
    SPRING_DATASOURCE_PASSWORD: "dev_secret"
```

---

### Lec 5 — Demo of Docker Network Concept

#### What This Lecture Demonstrates

A live demo showing what happens when database containers are **removed from the Docker network** — microservices can no longer connect to them. This proves that Docker network membership is required for inter-container communication.

#### The Experiment

**Setup**: Run with QA profile (`docker-compose/qa/`)

**Test**: Remove `extends: network-deploy-service` from the DB services in `common-config.yml` so they start in Docker's default network instead of the `eazybank` network.

**Result**: Accounts/Loans/Cards microservices enter an **exited** state because they cannot reach the database containers.

#### Viewing Docker Networks

```bash
# List all networks
docker network ls

# Output example:
NETWORK ID     NAME              DRIVER    SCOPE
abc123         eazybank_qa       bridge    local
def456         bridge            bridge    local
ghi789         host              host      local
```

#### Inspecting Which Network a Container Belongs To

```bash
docker inspect <container_id>
# → Look for "Networks" section in JSON output
```

Example output shows `qa_default` network (the default Docker Compose network) instead of `eazybank_qa` — confirming the DB containers are on the wrong network when `extends` is removed.

#### Key Docker Network Rules

| Rule | Detail |
|---|---|
| **Same network = can communicate** | All containers on `eazybank` can reach each other by service name |
| **Different network = no communication** | Containers in different networks are completely isolated |
| **No network defined = default network** | Docker assigns a default per-compose-file network |
| **Port mapping is for external access only** | `-p 3307:3306` allows your laptop to reach the container, but containers talk directly via the internal network |

#### The Fix

Restore the `extends` block in `common-config.yml` for DB services:

```yaml
microservice-db-config:
  extends:
    service: network-deploy-service    # ← this puts DB containers in the eazybank network
  image: mysql
  healthcheck:
    ...
```

#### Why Future Sections Revert to H2

After demonstrating MySQL integration, the course reverts to H2 for subsequent sections because:

- Running 7+ containers (3 DBs + config server + 3 microservices) strains low-RAM laptops (8 GB).
- More components will be added (Eureka Server, Gateway Server, Grafana, etc.).
- Each additional container increases memory pressure and risk of slowness.

> **For powerful machines (16+ GB RAM)**: You can continue using MySQL throughout the course. Just keep the DB containers running alongside future additions.

---

## Quick Reference

### Docker Run Commands for MySQL Containers

```bash
# Accounts DB
docker run -p 3306:3306 --name accountsDB \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=accountsdb \
  -d mysql

# Loans DB
docker run -p 3307:3306 --name loansDB \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=loansdb \
  -d mysql

# Cards DB
docker run -p 3308:3306 --name cardsDB \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=cardsdb \
  -d mysql
```

### `application.yml` DataSource Config Per Microservice

```yaml
# accounts
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/accountsdb
    username: root
    password: root
    sql:
      init:
        mode: always

# loans
spring:
  datasource:
    url: jdbc:mysql://localhost:3307/loansdb
    username: root
    password: root
    sql:
      init:
        mode: always

# cards
spring:
  datasource:
    url: jdbc:mysql://localhost:3308/cardsdb
    username: root
    password: root
    sql:
      init:
        mode: always
```

### Docker Compose Environment Variables for MySQL

```yaml
# In docker-compose.yml per microservice
environment:
  SPRING_DATASOURCE_URL: "jdbc:mysql://<service-name>:3306/<dbname>"

# In common-config.yml (shared)
environment:
  SPRING_DATASOURCE_USERNAME: "root"
  SPRING_DATASOURCE_PASSWORD: "root"
```

### Docker Compose DB Service Template

```yaml
accountsdb:
  container_name: accountsdb
  ports:
    - "3306:3306"       # host:container
  environment:
    MYSQL_DATABASE: "accountsdb"
  extends:
    file: common-config.yml
    service: microservice-db-config
```

### Critical Port Rule

```
Inside Docker network  → always use internal port 3306
Outside Docker network → use host-exposed port (3306/3307/3308)
```

### Startup Order (with MySQL)

```
MySQL DBs (all 3, parallel, healthcheck: mysqladmin ping)
    ↓ service_healthy
Config Server (healthcheck: actuator/health/readiness)
    ↓ service_healthy
Microservices (accounts + loans + cards, parallel)
    ↓ each depends_on its own DB + configserver
```

### Dependency Chain Summary

```
accounts-ms  depends_on → accountsdb (service_healthy) + configserver (service_healthy)
loans-ms     depends_on → loansdb    (service_healthy) + configserver (service_healthy)
cards-ms     depends_on → cardsdb    (service_healthy) + configserver (service_healthy)
configserver depends_on → (no DB dependency)
```

---

*Notes compiled from Section 7 Lectures 1–5: MySQL Database Integration with Microservices.*
