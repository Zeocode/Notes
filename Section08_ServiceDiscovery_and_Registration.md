# Section 08: Service Discovery & Service Registration in Microservices

---

## Table of Contents

| Lec # | Topic |
|-------|-------|
| 1 | [Brief Introduction About Microservices Traffic](#lecture-1--brief-introduction-about-microservices-traffic) |
| 2 | [Introduction to Service Discovery & Registration](#lecture-2--introduction-to-service-discovery--registration) |
| 3 | [Why Not Traditional Load Balancers for Microservices](#lecture-3--why-not-traditional-load-balancers-for-microservices) |
| 4 | [Service Discovery & Registration Pattern](#lecture-4--service-discovery--registration-pattern) |
| 5 | [How Client-Side Service Discovery & Load Balancing Works](#lecture-5--how-client-side-service-discovery--load-balancing-works) |
| 6 | [Spring Cloud Support for Service Discovery & Registration](#lecture-6--spring-cloud-support-for-service-discovery--registration) |
| 7 | [Setup Service Discovery Agent Using Eureka Server](#lecture-7--setup-service-discovery-agent-using-eureka-server) |
| 8 | [Make Code Changes in Accounts Microservice to Connect Eureka Server](#lecture-8--make-code-changes-in-accounts-microservice-to-connect-eureka-server) |
| 9 | [Make Code Changes in Loans & Cards Microservice to Connect Eureka Server](#lecture-9--make-code-changes-in-loans--cards-microservice-to-connect-eureka-server) |
| 10 | [De-registration from Eureka Server When Microservices Shutdown](#lecture-10--de-registration-from-eureka-server-when-microservices-shutdown) |
| 11 | [Demo of Heartbeats Mechanism to Eureka Server from Clients](#lecture-11--demo-of-heartbeats-mechanism-to-eureka-server-from-clients) |
| 12 | [Feign Client Code Changes to Invoke Other Microservices – Part 1](#lecture-12--feign-client-code-changes-to-invoke-other-microservices--part-1) |
| 13 | [Feign Client Code Changes to Invoke Other Microservices – Part 2](#lecture-13--feign-client-code-changes-to-invoke-other-microservices--part-2) |
| 14 | [Eureka Self-Preservation Mode to Avoid Network Trap Issues](#lecture-14--eureka-self-preservation-mode-to-avoid-network-trap-issues) |
| 15 | [Generating Docker Images with Service Discovery Changes & Push to Docker Hub](#lecture-15--generating-docker-images-with-service-discovery-changes--push-to-docker-hub) |
| 16 | [Updating Docker Compose File to Adapt Service Discovery Changes](#lecture-16--updating-docker-compose-file-to-adapt-service-discovery-changes) |
| 17 | [Starting All Microservices Using Docker Compose](#lecture-17--starting-all-microservices-using-docker-compose) |
| 18 | [Demo of Client-Side Service Discovery & Load Balancing](#lecture-18--demo-of-client-side-service-discovery--load-balancing) |
| — | [Summary](#summary--section-08) |

---

## Lecture 1 — Brief Introduction About Microservices Traffic

- Microservices are deployed inside a **microservice network** (not on separate public servers).
- External clients communicate with the microservice network through a **single entry point** called the **API Gateway**.
  - The API Gateway handles security, auditing, and logging (non-functional requirements).
  - External traffic = traffic coming through the API Gateway from outside.
- **Internal communication** = communication between microservices inside the network.
  - e.g., `accounts` microservice calling `loans` and `cards` microservices.
- This section focuses on **internal communication** challenges; API Gateway will be covered in a later section.

---

## Lecture 2 — Introduction to Service Discovery & Registration

> **Challenge #5** in the microservices journey.

### Key Questions / Problems:
1. **How does a microservice locate other microservices?**
   - Hardcoding IPs or DNS names works for monolithic apps, but microservices scale up/down dynamically — IPs change constantly.
2. **How does a new service instance enter the network?**
   - During auto-scaling or container replacement, new instances get new IPs. Other services won't be aware automatically.
3. **How is load balancing achieved across multiple instances?**
   - Without awareness of all instances, traffic will concentrate on a single instance.

### Solution:
- **Service Discovery**, **Service Registration**, and **Load Balancing** patterns.

---

## Lecture 3 — Why Not Traditional Load Balancers for Microservices

### Traditional Approach (IP/DNS based):
- **Upstream service**: service that depends on another (e.g., `accounts`).
- **Downstream/backing service**: service being depended on (e.g., `loans`).
- Options: hardcode IP or use DNS names mapped to IP → works for static/monolithic environments.

### Problems with Traditional Load Balancers in Microservices:
| Disadvantage | Explanation |
|---|---|
| **Limited horizontal scalability** | Routing tables must be manually updated with every IP change. |
| **License cost** | Traditional LBs from cloud providers have licensing costs. |
| **Single point of failure** | Even with primary + secondary LB setup, both can fail; not easily clusterable. |
| **Manual IP management** | Impossible to keep routing tables up-to-date in a dynamic container environment. |
| **Complex & container-unfriendly** | Not designed for ephemeral (short-lived) containers. |

> **Key term**: *Ephemeral* — containers are short-lived and can be disposed of at any time.

---

## Lecture 4 — Service Discovery & Registration Pattern

### Solution: Service Discovery Pattern
- A **Service Registry** (central server) maintains information about all running service instances.
- **Registration**: On startup, a microservice registers its IP, hostname, and port with the registry.
- **De-registration**: On shutdown, the microservice removes itself from the registry.
- **Heartbeats**: Running instances send regular heartbeat signals to confirm health.
- **Discovery**: When a service needs to communicate with another, it queries the registry.
- **Load Balancing**: If multiple instances exist, the registry returns all IPs and a load-balancing strategy is applied.

### Two Approaches:
| Approach | Description |
|---|---|
| **Client-Side Service Discovery** | Client microservice queries registry and decides which instance to call. |
| **Server-Side Service Discovery** | Kubernetes environment handles discovery automatically. |

> This section focuses on **Client-Side Service Discovery**.

---

## Lecture 5 — How Client-Side Service Discovery & Load Balancing Works

### Flow:
1. **Service Registry** starts first.
2. Microservice instances (e.g., loans) start and **register** themselves (IP, hostname, port) with the registry.
3. Instances send **heartbeat** signals every ~30 seconds.
4. When `accounts` needs to call `loans`, it **queries the registry** for `loans` instances.
5. Registry returns a **list of IP addresses**.
6. `accounts` applies a **load-balancing strategy** (client-side) and picks an instance.
7. `accounts` **caches** the IP list locally (refreshed every ~10–20 seconds) to reduce load on the registry.
8. On exception (e.g., an instance is unreachable), the cache is **invalidated** and re-fetched from the registry.

### Load Balancing Strategies (Client-Side):
- Round Robin
- Weighted Round Robin
- Least Connections
- Custom algorithms

### Advantages:
- Flexible load balancing strategy choices.
- IP configurations update dynamically — no manual work.

### Drawbacks:
- Developers must make code changes in each microservice.
- Requires a centralized service registry server.

### Client-Side vs Server-Side:
- **Client-Side**: suitable for projects that can't afford Kubernetes.
- **Server-Side** (Kubernetes): suitable for projects with dedicated ops/platform teams and cloud budget.

---

## Lecture 6 — Spring Cloud Support for Service Discovery & Registration

### Spring Cloud Components Used:

| Component | Role |
|---|---|
| **Spring Cloud Netflix Eureka Server** | Central service registry & discovery agent |
| **Spring Cloud Load Balancer** | Client-side load balancing (replaces deprecated Netflix Ribbon) |
| **Spring Cloud OpenFeign (Netflix Feign Client)** | REST client for microservice-to-microservice communication |

### Why Netflix?
- Netflix built internal libraries (Ribbon, Eureka, Hystrix, Governator) around 2007 for cloud-native development.
- In 2012, Netflix **open-sourced** these libraries to the Spring community.
- In 2015, **Spring Cloud Netflix** project was created.
- Netflix itself now uses Spring Boot + Spring Cloud Load Balancer + Resilience4j (replacing their own older stack).

### Modern Stack (what we use):
- ~~Netflix Ribbon~~ → **Spring Cloud Load Balancer**
- **Spring Cloud Netflix Eureka** (still used for service discovery)
- ~~Hystrix~~ → **Resilience4j** (for fault tolerance)
- **Spring Cloud OpenFeign** (for inter-service REST calls)

---

## Lecture 7 — Setup Service Discovery Agent Using Eureka Server

### Steps to Create a Eureka Server:

1. **Go to `start.spring.io`** and create a new project:
   - Group: `com.eazybytes`
   - Artifact: `eurekaserver`
   - Dependencies: `Eureka Server`, `Config Client`, `Spring Boot Actuator`

2. **Add `@EnableEurekaServer`** annotation to the main class → converts the Spring Boot app into a service discovery agent.

3. **Rename `application.properties` → `application.yml`** and add:
   ```yaml
   spring:
     application:
       name: eurekaserver
     config:
       import: "optional:configserver:http://localhost:8071/"
   management:
     endpoints:
       web:
         exposure:
           include: "*"
     health:
       readinessstate:
         enabled: true
       livenessstate:
         enabled: true
   ```

4. **In Config Server GitHub repo**, create `eurekaserver.yml` (or `.properties`) with:
   ```yaml
   server:
     port: 8070
   eureka:
     instance:
       hostname: localhost
     client:
       fetchRegistry: false          # Eureka server doesn't need to fetch other services
       registerWithEureka: false     # Eureka server doesn't register itself
       serviceUrl:
         defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
   ```

5. **Start order**: Config Server → Eureka Server.

6. **Validate**: Access `http://localhost:8070` → Eureka Dashboard should appear (empty at first).

> **Key**: The Eureka file name in the config repo must match `spring.application.name`.

---

## Lecture 8 — Make Code Changes in Accounts Microservice to Connect Eureka Server

### Steps for Accounts Microservice:

1. **Add dependency** in `pom.xml`:
   ```xml
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   ```

2. **Update `application.yml`** with Eureka client properties:
   ```yaml
   eureka:
     instance:
       preferIpAddress: true   # Register with IP address (not hostname)
     client:
       fetchRegistry: true
       registerWithEureka: true
       serviceUrl:
         defaultZone: http://localhost:8070/eureka/
   info:
     app:
       name: "accounts"
       description: "Eazy Bank Accounts Application"
       version: "1.0.0"
   management:
     info:
       env:
         enabled: true
     endpoint:
       shutdown:
         enabled: true
   endpoints:
     shutdown:
       enabled: true
   ```

3. **`preferIpAddress: true`** — registers using IP address instead of hostname (important for local environments without DNS).

4. **`info.*` properties** — displayed on the Eureka Dashboard when clicking on a service instance link (exposed via `/actuator/info`).

5. **Shutdown endpoint** — enables graceful shutdown via `POST /actuator/shutdown` → triggers de-registration from Eureka.

6. **Start order**: Config Server → Eureka Server → Accounts Microservice.

7. **Validate**: Accounts microservice appears in Eureka Dashboard with status `UP`. Default heartbeat interval = **30 seconds**.

---

## Lecture 9 — Make Code Changes in Loans & Cards Microservice to Connect Eureka Server

- Apply the **same changes** from Lecture 8 to both `loans` and `cards` microservices.
- Same dependencies in `pom.xml`.
- Same `application.yml` properties (update `info.app.name` and `description` accordingly).
  - Cards: `name: "cards"`, `description: "Eazy Bank Cards Application"`
  - Loans: `name: "loans"`, `description: "Eazy Bank Loans Application"`
- After startup, all three microservices appear in the Eureka Dashboard.

### Useful Eureka Endpoint:
- `GET http://localhost:8070/eureka/apps` → returns all registered instances in **XML** (default).
- For **JSON** format: add header `Accept: application/json`.
- Filter by service: `http://localhost:8070/eureka/apps/accounts` → only accounts info.

Each registered instance includes:
- Instance ID, hostname, app name, IP address, status, port
- Renewal interval, registration time, last renewal timestamp, health check URL, etc.

---

## Lecture 10 — De-registration from Eureka Server When Microservices Shutdown

- When a microservice is **gracefully shut down**, it automatically **de-registers** from Eureka.
- **Do NOT** stop the app via IDE (kills the process immediately without graceful shutdown).
- Use the **actuator shutdown endpoint**:
  ```
  POST http://localhost:8080/actuator/shutdown   (accounts)
  POST http://localhost:9000/actuator/shutdown   (cards)
  POST http://localhost:8090/actuator/shutdown   (loans)
  ```
- After shutdown, the service disappears from the Eureka Dashboard.
- Log evidence: `Unregistering application ACCOUNTS with eureka with status 200`.

---

## Lecture 11 — Demo of Heartbeats Mechanism to Eureka Server from Clients

- Microservices send **heartbeat signals every 30 seconds** (default) to Eureka Server.
- **Demo**: Stop the Eureka Server → after ~30 seconds, microservices log exceptions:
  > `Was unable to send Heartbeat to Eureka Server!`
- The heartbeat request is a `PUT` to the Eureka Server URL.

### Summary of Lifecycle:
| Event | Action |
|---|---|
| **Startup** | Microservice registers with Eureka |
| **Running** | Heartbeat every 30s |
| **Shutdown** | Microservice de-registers from Eureka |

All of this happens **automatically** — no manual work required.

---

## Lecture 12 — Feign Client Code Changes to Invoke Other Microservices – Part 1

### Goal:
Build a new REST API in `accounts` microservice that consolidates data from `accounts`, `loans`, and `cards` microservices for a given mobile number.

### Steps:

1. **Add OpenFeign dependency** in `accounts/pom.xml`:
   ```xml
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```

2. **Add `@EnableFeignClients`** annotation to the `AccountsApplication` main class.

3. **Create `CardsFeignClient` interface** under `com.eazybytes.accounts.service.client`:
   ```java
   @FeignClient("cards")   // logical name registered with Eureka
   public interface CardsFeignClient {
       @GetMapping(value = "/api/fetch", consumes = "application/json")
       public ResponseEntity<CardsDto> fetchCardDetails(
           @RequestParam String mobileNumber);
   }
   ```

4. **Create `LoansFeignClient` interface** (same pattern):
   ```java
   @FeignClient("loans")
   public interface LoansFeignClient {
       @GetMapping(value = "/api/fetch", consumes = "application/json")
       public ResponseEntity<LoansDto> fetchLoanDetails(
           @RequestParam String mobileNumber);
   }
   ```

5. **Copy DTOs** from the respective microservices into `accounts`:
   - `CardsDto` from `cards` microservice
   - `LoansDto` from `loans` microservice

### Key Rules for FeignClient:
- `@FeignClient("name")` value must match the **logical service name** registered with Eureka.
- The **method signature** (return type, parameters) must match the actual REST API in the target microservice.
- No implementation code needed — OpenFeign generates it at runtime.
- Similar concept to Spring Data JPA (declarative, no implementation).

---

## Lecture 13 — Feign Client Code Changes to Invoke Other Microservices – Part 2

### Steps (continued):

1. **Create `CustomerDetailsDto`** in `accounts/dto`:
   - Extends/combines `CustomerDto` with `AccountsDto`, `CardsDto`, and `LoansDto`.

2. **Create `ICustomerService` interface** in the service package:
   ```java
   CustomerDetailsDto fetchCustomerDetails(String mobileNumber);
   ```

3. **Create `CustomerServiceImpl`** class:
   - Inject: `AccountsRepository`, `CustomerRepository`, `CardsFeignClient`, `LoansFeignClient`.
   - Annotate with `@Service` and `@AllArgsConstructor` (Lombok).
   - Business logic:
     1. Fetch customer from DB by mobile number.
     2. Fetch account from DB by customerId.
     3. Call `cardsFeignClient.fetchCardDetails(mobileNumber)` → `CardsDto`.
     4. Call `loansFeignClient.fetchLoanDetails(mobileNumber)` → `LoansDto`.
     5. Map everything into `CustomerDetailsDto` and return.

4. **Create `CustomerController`**:
   - `GET /api/fetchCustomerDetails?mobileNumber=...`
   - Returns `ResponseEntity<CustomerDetailsDto>`.

### Behind the Scenes (OpenFeign + Eureka + Load Balancer):
1. FeignClient connects to Eureka to get all instances of e.g. `cards`.
2. Caches the IP list locally for ~30 seconds.
3. Spring Cloud Load Balancer picks one instance (round-robin by default).
4. Makes the actual HTTP call.

---

## Lecture 14 — Eureka Self-Preservation Mode to Avoid Network Trap Issues

### The Warning on Eureka Dashboard:
> *"EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY ARE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE."*

### What is Self-Preservation Mode?
- Normally, if Eureka doesn't receive heartbeats from an instance within the expiry window, it **removes** it from the registry.
- Problem: **Temporary network glitches** can cause heartbeats to be missed → false eviction of healthy instances.
- **Self-Preservation Mode**: When Eureka detects that the majority of instances have stopped sending heartbeats (likely a network issue), it **stops evicting instances** and preserves the current registry.

### Self-Preservation Trigger:
- Activated when heartbeats received fall below **85%** of the expected threshold (default `renewal-percent-threshold = 0.85`).

### Key Properties:

| Property | Default | Description |
|---|---|---|
| `eureka.instance.lease-renewal-interval-in-seconds` | `30s` | How often clients send heartbeats |
| `eureka.instance.lease-expiration-duration-in-seconds` | `90s` | Max wait time before evicting an instance |
| `eureka.server.eviction-interval-timer-milliseconds` | `60000ms` (60s) | How often the eviction task runs |
| `eureka.server.renewal-percent-threshold` | `0.85` (85%) | Threshold for activating self-preservation |
| `eureka.server.renewal-threshold-update-interval-ms` | `15 min` | How often expected heartbeat count is recalculated |
| `eureka.server.enable-self-preservation` | `true` | Disable self-preservation by setting to `false` |

> ⚠️ Disabling self-preservation can cause instability during temporary network issues.

---

## Lecture 15 — Generating Docker Images with Service Discovery Changes & Push to Docker Hub

- Build Docker images for all microservices including the new Eureka Server and updated microservices (accounts, loans, cards).
- Push the images to Docker Hub.
- Each microservice now includes Eureka client dependency and configuration.
- The new **Eureka Server** also gets its own Docker image.

---

## Lecture 16 — Updating Docker Compose File to Adapt Service Discovery Changes

- Add **Eureka Server** service to `docker-compose.yml`.
- Configure **health checks** for Eureka Server (using `/actuator/health/readiness`).
- Use `depends_on` with `condition: service_healthy` so that `accounts`, `loans`, and `cards` only start **after Eureka Server is healthy**.
- Update service URLs for Eureka clients: use **Docker container name** instead of `localhost` (e.g., `http://eurekaserver:8070/eureka/`).

---

## Lecture 17 — Starting All Microservices Using Docker Compose

- Command: `docker compose up -d`
- Startup order: Config Server → Eureka Server → Accounts, Loans, Cards.
- Validate all services are registered on Eureka Dashboard at `http://localhost:8070`.

---

## Lecture 18 — Demo of Client-Side Service Discovery & Load Balancing

### Setup:
- Add a **second instance of `loans` microservice** to `docker-compose.yml`:
  - New service name: `loans1` (distinct container name: `loans-ms1`)
  - Use a different external port (e.g., `8091:8090`).

### Observation:
- Both loans instances register with Eureka → `LOANS (2)` shown on dashboard.
- `GET http://localhost:8070/eureka/apps` → two instance entries under LOANS.

### Load Balancing Demo:
- Create loan data via instance 1 (`localhost:8090`) but NOT via instance 2.
- Call `GET /api/fetchCustomerDetails?mobileNumber=...` on accounts microservice multiple times:
  - Sometimes success (routed to instance with data).
  - Sometimes error: `loan not found` (routed to instance without data).
- This **proves round-robin client-side load balancing** is working via Spring Cloud Load Balancer.

### Key Takeaway:
- No manual configuration needed — Eureka + OpenFeign + Spring Cloud Load Balancer handle everything automatically.
- In production: all instances share a single database (MySQL), so load balancing is seamless.

---

## Summary — Section 08

| Concept | Tool/Library |
|---|---|
| Service Registry & Discovery Agent | Spring Cloud Netflix Eureka Server |
| Service Registration (client-side) | `spring-cloud-starter-netflix-eureka-client` |
| Client-Side Load Balancing | Spring Cloud Load Balancer (built into OpenFeign) |
| Inter-service REST communication | Spring Cloud OpenFeign |
| Graceful shutdown / de-registration | Spring Boot Actuator shutdown endpoint |
| Self-preservation (network fault tolerance) | Eureka built-in |

### Architecture Flow:
```
[External Client]
       |
   [API Gateway]         ← covered in a later section
       |
  [Accounts MS]  ──── queries ────►  [Eureka Server]
       |                                    ▲
       ├─ [Loans FeignClient]  ◄── caches ──┘
       └─ [Cards FeignClient]
              |
    [Load Balancer picks instance]
              |
    [Loans Instance 1 / Instance 2]
    [Cards Instance 1]
```

