# Section 09: Gateway, Routing & Cross-Cutting Concerns in Microservices

---

## Table of Contents

| Lec # | Topic |
|-------|-------|
| 1 | [Challenges While Dealing with External Communication in Microservices](#lecture-1--challenges-while-dealing-with-external-communication-in-microservices) |
| 2 | [Why We Need an Edge Server or API Gateway Inside Microservices](#lecture-2--why-we-need-an-edge-server-or-api-gateway-inside-microservices) |
| 3 | [Introduction to Spring Cloud Gateway](#lecture-3--introduction-to-spring-cloud-gateway) |
| 4 | [Deep Dive on Spring Cloud Gateway Internal Architecture](#lecture-4--deep-dive-on-spring-cloud-gateway-internal-architecture) |
| 5 | [Building Edge Server Using Spring Cloud Gateway](#lecture-5--building-edge-server-using-spring-cloud-gateway) |
| 6 | [Demo of Edge Server with Default Routing Configs](#lecture-6--demo-of-edge-server-with-default-routing-configs) |
| 7 | [Make Changes Inside Gateway Server to Accept Service Names with Lowercase](#lecture-7--make-changes-inside-gateway-server-to-accept-service-names-with-lowercase) |
| 8 | [Implementing Custom Routing Using Spring Cloud Gateway](#lecture-8--implementing-custom-routing-using-spring-cloud-gateway) |
| 9 | [Demo of `addResponseHeader` Filter](#lecture-9--demo-of-addresponseheader-filter) |
| 10 | [Implementing Cross-Cutting Concerns: Tracing & Logging (Part 1)](#lecture-10--implementing-cross-cutting-concerns-tracing--logging-part-1) |
| 11 | [Implementing Cross-Cutting Concerns: Tracing & Logging (Part 2)](#lecture-11--implementing-cross-cutting-concerns-tracing--logging-part-2) |
| 12 | [Design Patterns Around API Gateway](#lecture-12--design-patterns-around-api-gateway) |
| 13 | [Generating and Pushing Docker Images with Spring Cloud Gateway Changes](#lecture-13--generating-and-pushing-docker-images-with-spring-cloud-gateway-changes) |
| 14 | [Updating Docker Compose File to Adapt Spring Cloud Gateway Changes](#lecture-14--updating-docker-compose-file-to-adapt-spring-cloud-gateway-changes) |
| — | [Section Summary](#section-summary) |

---

## Lecture 1 — Challenges While Dealing with External Communication in Microservices

> **Challenge #6** in the microservices journey.

### Key Questions / Problems:
1. **How to maintain a single entry point into the microservice network?**
   - Without a single entry point, external clients must track URLs and ports of every individual microservice.
   - Exposing all microservices directly to external clients is insecure and unmanageable.

2. **How to handle cross-cutting concerns (logging, auditing, tracing, security)?**
   - Implementing these in every microservice leads to duplicate code and inconsistent behaviour.
   - A centralised solution is needed.

3. **How to perform dynamic routing based on custom requirements?**
   - e.g., route based on URL path, HTTP headers (API version), or other attributes.

### Solution:
- Use an **Edge Server** (also called **API Gateway**) — a server that sits on the edge of the microservice network and monitors all incoming/outgoing requests.

---

## Lecture 2 — Why We Need an Edge Server or API Gateway Inside Microservices

### Problems Without an API Gateway:
| Problem | Explanation |
|---|---|
| **Duplicate cross-cutting logic** | Security, logging, auditing must be repeated in every microservice. |
| **Inconsistency** | Different developers implement cross-cutting concerns differently. |
| **Tight coupling via shared libraries** | Common libraries require regression testing across all services for every change. |

### How the API Gateway / Edge Server Helps:
- Acts as a **mediator** between client applications and individual microservices.
- Executes all cross-cutting concern logic **before** forwarding the request.

### Capabilities of an API Gateway:
| Capability | Description |
|---|---|
| **Validation** | Include/exclude lists, blacklist/whitelist of IP addresses. |
| **Authentication & Authorization** | Enforce security on all incoming requests. |
| **Rate Limiting** | Limit the number of requests from a specific client. |
| **Dynamic Routing** | Route requests to appropriate microservices. |
| **Service Discovery** | Integrate with Eureka to locate microservice instances. |
| **Request/Response Modification** | Modify headers, bodies before forwarding or returning. |
| **Protocol Conversion** | e.g., HTTPS → HTTP for internal traffic. |
| **Circuit Breaker** | Prevent cascading failures; make downstream services resilient. |
| **Retries & Timeouts** | Handle transient failures gracefully. |
| **Quota Policies** | Enforce usage limits per subscription plan (standard, premium, advance). |
| **Logging & Monitoring** | Send data to tools like Grafana for dashboards. |
| **Redis Caching** | Cache responses for performance improvement. |

> **Note:** Eureka Server handles only service discovery/registration. API Gateway handles all cross-cutting concerns.

---

## Lecture 3 — Introduction to Spring Cloud Gateway

### What is Spring Cloud Gateway?
- A project under **Spring Cloud** used to build a production-ready edge server / API Gateway.
- Built on the **Spring Reactive Framework** (Project Reactor) — non-blocking, efficient with low memory and threads.
- Acts as the **gatekeeper** for all inbound traffic.

### Key Features:
- Dynamically route requests based on any request attribute (URL, headers, method, cookies, etc.).
- Use **predicates** and **filters** per route.
- Integration with **Circuit Breaker** and **Spring Cloud Discovery Client** (Eureka).
- Supports **path rewriting**, **request rate limiting**, **sticky sessions**.
- Capable of acting as a **central policy enforcement point**.

### Spring Cloud Gateway vs Zuul:
| Feature | Spring Cloud Gateway | Zuul |
|---|---|---|
| Architecture | Reactive (non-blocking) | Blocking (Servlet-based) |
| Circuit Breaker Integration | ✅ Built-in | Limited |
| Service Discovery | ✅ Eureka integration | Limited |
| Performance | Higher (less memory/threads) | Lower |
> **Spring Cloud Gateway is the preferred choice** for modern Java microservices.

---

## Lecture 4 — Deep Dive on Spring Cloud Gateway Internal Architecture

### Internal Architecture Flow:

```
Client App
   │
   ▼
[Gateway Handler Mapping]   ← Developer configures routing rules
   │
   ▼
[Predicates]                ← Evaluate conditions (return true/false)
   │  (if predicates pass)
   ▼
[Pre-Filters]               ← Execute before forwarding request (logging, security, etc.)
   │
   ▼
[Actual Microservice]       ← accounts / loans / cards
   │
   ▼
[Post-Filters]              ← Execute after receiving response (modify response, add headers)
   │
   ▼
[Gateway Handler Mapping]
   │
   ▼
Client App (Response)
```

### Key Components:

| Component | Role |
|---|---|
| **Gateway Handler Mapping** | Identifies request path and determines which microservice to route to. |
| **Predicates** | Conditions that must be true for a route to match. Returns boolean. |
| **Pre-Filters** | Logic executed before forwarding to microservice (validation, auditing, logging, security). |
| **Post-Filters** | Logic executed on the response before sending back to client. |

### Predefined Route Predicate Factories:
Route matching can be based on: **header value, cookie, host, method, path, query parameter, remote address, weight**.

### Predefined Gateway Filter Factories (selection):
- `AddRequestHeader` / `AddResponseHeader`
- `RewritePath`
- `CircuitBreaker`
- `CacheRequestBody`
- `ModifyRequestBody` / `ModifyResponseBody`
- `RetryGatewayFilter`
- `TokenRelayGatewayFilter`
- `GatewayMetricsFilter`
- `JsonToGrpc`

> **Official Docs:** [spring.io → Spring Cloud Gateway](https://spring.io/projects/spring-cloud-gateway)

---

## Lecture 5 — Building Edge Server Using Spring Cloud Gateway

### Project Setup (Spring Initializr):
| Field | Value |
|---|---|
| **Build tool** | Maven |
| **Language** | Java |
| **Group** | `com.eazybytes` |
| **Artifact / Name** | `gatewayserver` |
| **Packaging** | Jar |
| **Java Version** | 17 |

### Required Dependencies:
- `Spring Cloud Gateway` — core gateway functionality
- `Eureka Discovery Client` — connect to Eureka to fetch service details
- `Config Client` — load configurations from Config Server at startup
- `Actuator` — expose management endpoints
- `Spring Boot DevTools` — faster restarts during development

### `pom.xml` — Add Jib Plugin:
```xml
<!-- Copy jib plugin from other microservices, update tag to s9 -->
```

### `application.yml` Configuration (gatewayserver):
```yaml
spring:
  application:
    name: gatewayserver
  config:
    import: "optional:configserver:http://localhost:8071/"
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
          lower-case-service-id: true  # added in Lecture 7

management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    gateway:
      enabled: true

info:
  app:
    name: "gatewayserver"
    description: "Eazy Bank Gateway Server Application"

logging:
  level:
    com:
      eazybytes:
        gatewayserver: DEBUG
```

### `gatewayserver.yml` (in Config Server repo):
```yaml
server:
  port: 8072

eureka:
  instance:
    preferIpAddress: true
  client:
    fetchRegistry: true
    registerWithEureka: true
    serviceUrl:
      defaultZone: "http://localhost:8070/eureka/"
```

### Key Configuration Property:
```yaml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
```
> Tells the gateway to connect with Eureka and use discovered service details to forward traffic.

### Startup Order:
1. Config Server
2. Eureka Server
3. accounts, loans, cards microservices (any order)
4. **Gateway Server** (last — needs all others registered in Eureka)

---

## Lecture 6 — Demo of Edge Server with Default Routing Configs

### How Default Routing Works:
- Gateway server fetches routing info from Eureka automatically when `discovery.locator.enabled: true`.
- Each microservice gets a default route entry accessible via `actuator/gateway/routes`.

### Default Route Example (for `loans`):
```json
{
  "predicate": "Path: /loans/**",
  "uri": "lb://LOANS",
  "filters": ["RewritePath: /loans/(?<remaining>.*) -> /${remaining}"]
}
```
- `lb://LOANS` — load-balanced URI using Eureka's registered app name `LOANS`.
- **RewritePath filter** strips the service-name prefix before forwarding.

### Example Postman Calls Through Gateway:
| Request | URL |
|---|---|
| Create account | `POST http://localhost:8072/ACCOUNTS/api/create` |
| Fetch account | `GET http://localhost:8072/ACCOUNTS/api/fetch?mobileNumber=...` |
| Fetch loans | `GET http://localhost:8072/LOANS/api/fetch?mobileNumber=...` |

> Service names in the URL must be in **ALL CAPS** by default (matches Eureka registration).

---

## Lecture 7 — Make Changes Inside Gateway Server to Accept Service Names with Lowercase

### Problem:
- By default, Eureka registers app names in **UPPERCASE** → clients must use uppercase in URLs (e.g., `/ACCOUNTS/...`), which is unprofessional.

### Solution:
Add the `lower-case-service-id` property in `application.yml`:

```yaml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
          lower-case-service-id: true
```

### Result:
- Clients can now use lowercase paths: `/accounts/api/fetch`, `/loans/api/fetch`, etc.
- Capital letter paths (e.g., `/ACCOUNTS/`) will return `404 Not Found`.

---

## Lecture 8 — Implementing Custom Routing Using Spring Cloud Gateway

### Why Custom Routing?
- Default routing exposes service names directly in the URL (e.g., `/accounts/...`).
- A more professional approach uses a prefix like `/eazybank/accounts/**`.

### Implementation — Java-Based Configuration (Preferred):
Create a `@Bean` of type `RouteLocator` in the main application class:

```java
@Bean
public RouteLocator eazybankRouteConfig(RouteLocatorBuilder routeLocatorBuilder) {
    return routeLocatorBuilder.routes()
        .route(p -> p
            .path("/eazybank/accounts/**")
            .filters(f -> f
                .rewritePath("/eazybank/accounts/(?<segment>.*)", "/${segment}")
                .addResponseHeader("X-Response-Time", LocalDateTime.now().toString()))
            .uri("lb://ACCOUNTS"))
        .route(p -> p
            .path("/eazybank/loans/**")
            .filters(f -> f
                .rewritePath("/eazybank/loans/(?<segment>.*)", "/${segment}")
                .addResponseHeader("X-Response-Time", LocalDateTime.now().toString()))
            .uri("lb://LOANS"))
        .route(p -> p
            .path("/eazybank/cards/**")
            .filters(f -> f
                .rewritePath("/eazybank/cards/(?<segment>.*)", "/${segment}")
                .addResponseHeader("X-Response-Time", LocalDateTime.now().toString()))
            .uri("lb://CARDS"))
        .build();
}
```

> **Note:** `lb://ACCOUNTS` uses uppercase because that's how Eureka registers app names.

### Disable Default Auto-Discovery Routing:
Once custom routing is defined, disable the default Eureka-based routing to avoid confusion:
```yaml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: false
```
> This keeps only your 3 custom routes — no duplicates.

### YAML-Based Routing (Alternative — Less Flexible):
Spring Cloud Gateway also supports routing config in `application.yml`, but **Java-based config is recommended** for complex routing logic and multiple filters.

---

## Lecture 9 — Demo of `addResponseHeader` Filter

### Built-in Filter: `addResponseHeader`
Used to add a custom header to every response going back to the client.

### Example Usage:
```java
.filters(f -> f
    .rewritePath("/eazybank/accounts/(?<segment>.*)", "/${segment}")
    .addResponseHeader("X-Response-Time", LocalDateTime.now().toString()))
```

### Result in Postman Response Headers:
```
X-Response-Time: 2024-07-15T14:23:01.123456
```

### How to Add Multiple Filters:
Chain multiple filter calls using dot notation:
```java
.filters(f -> f
    .rewritePath(...)
    .addResponseHeader("Header1", "Value1")
    .addResponseHeader("Header2", "Value2"))
```

### Available Predefined Filters (Reference):
| Filter | Purpose |
|---|---|
| `rewritePath` | Rewrite the request path |
| `addResponseHeader` | Add header to response |
| `addRequestHeader` | Add header to request |
| `addRequestParameter` | Add query param to request |
| `circuitBreaker` | Implement circuit breaker |
| `retry` | Retry on failure |
| `modifyRequestBody` | Modify incoming request body |
| `modifyResponseBody` | Modify outgoing response body |
| `prefixPath` | Add prefix to path |
| `redirectTo` | Redirect to another URL |

---

## Lecture 10 — Implementing Cross-Cutting Concerns: Tracing & Logging (Part 1)

### Business Scenario:
When a request arrives at the gateway, generate a **Correlation ID** (trace ID) and:
1. Pass it to all downstream microservices via request headers.
2. Include it in the final response header sent back to the client.
3. Log the correlation ID at each microservice for debugging.

### Custom Filters Created:

#### Package: `com.eazybytes.gatewayserver.filters`

---

#### `FilterUtility.java`
Utility class providing reusable header operations:
```java
@Component
public class FilterUtility {
    public static final String CORRELATION_ID = "eazybank-correlation-id";

    public String getCorrelationId(HttpHeaders requestHeaders) {
        if (requestHeaders.get(CORRELATION_ID) != null) {
            return requestHeaders.get(CORRELATION_ID).stream().findFirst().get();
        }
        return null;
    }

    public ServerWebExchange setCorrelationId(ServerWebExchange exchange, String correlationId) {
        return exchange.mutate()
            .request(exchange.getRequest().mutate()
                .header(CORRELATION_ID, correlationId).build())
            .build();
    }
}
```

---

#### `RequestTraceFilter.java` (Pre-Filter / GlobalFilter)
Generates or detects a correlation ID on every incoming request:
```java
@Component
@Order(1)
public class RequestTraceFilter implements GlobalFilter {
    private static final Logger logger = LoggerFactory.getLogger(RequestTraceFilter.class);

    @Autowired FilterUtility filterUtility;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        HttpHeaders requestHeaders = exchange.getRequest().getHeaders();
        if (isCorrelationIdPresent(requestHeaders)) {
            logger.debug("eazybank-correlation-id found in RequestTraceFilter: {}",
                filterUtility.getCorrelationId(requestHeaders));
        } else {
            String correlationId = generateCorrelationId();
            exchange = filterUtility.setCorrelationId(exchange, correlationId);
            logger.debug("eazybank-correlation-id generated in RequestTraceFilter: {}", correlationId);
        }
        return chain.filter(exchange);
    }

    private boolean isCorrelationIdPresent(HttpHeaders headers) {
        return filterUtility.getCorrelationId(headers) != null;
    }

    private String generateCorrelationId() {
        return java.util.UUID.randomUUID().toString();
    }
}
```

> **`@Order(1)`** — ensures this filter runs first among all global filters.

> **`GlobalFilter`** — applies to all traffic received by the gateway.

> **Reactive types used:**
> - `ServerWebExchange` — gives access to request + response in reactive context.
> - `Mono<Void>` — returns nothing, just proceeds to next filter.
> - `GatewayFilterChain` — chain of filters; call `chain.filter(exchange)` to pass to next.

---

#### `ResponseTraceFilter.java` (Post-Filter via `@Bean`)
Adds the correlation ID to the outgoing response header:
```java
@Configuration
public class ResponseTraceFilter {
    private static final Logger logger = LoggerFactory.getLogger(ResponseTraceFilter.class);

    @Autowired FilterUtility filterUtility;

    @Bean
    public GlobalFilter postGlobalFilter() {
        return (exchange, chain) -> chain.filter(exchange).then(Mono.fromRunnable(() -> {
            HttpHeaders requestHeaders = exchange.getRequest().getHeaders();
            String correlationId = filterUtility.getCorrelationId(requestHeaders);
            exchange.getResponse().getHeaders().add(FilterUtility.CORRELATION_ID, correlationId);
            logger.debug("Updated the correlation id to the outbound headers: {}", correlationId);
        }));
    }
}
```

> **`.then()`** — ensures post-filter logic runs **after** the microservice responds.

### Enable Debug Logging in `application.yml` (Gateway Server):
```yaml
logging:
  level:
    com:
      eazybytes:
        gatewayserver: DEBUG
```

---

## Lecture 11 — Implementing Cross-Cutting Concerns: Tracing & Logging (Part 2)

### Changes in Individual Microservices:

#### 1. `CustomerController.java` (accounts microservice):
Accept correlation ID from request header and log it:
```java
private static final Logger logger = LoggerFactory.getLogger(CustomerController.class);

@GetMapping("/fetchCustomerDetails")
public ResponseEntity<CustomerDetailsDto> fetchCustomerDetails(
        @RequestHeader("eazybank-correlation-id") String correlationId,
        @RequestParam String mobileNumber) {
    logger.debug("eazybank-correlationId found: {}", correlationId);
    CustomerDetailsDto customerDetails = iCustomerService.fetchCustomerDetails(mobileNumber, correlationId);
    return ResponseEntity.ok(customerDetails);
}
```

#### 2. `ICustomerService` and `CustomerServiceImpl`:
Pass `correlationId` as parameter to service, then forward it to feign clients.

#### 3. Feign Client Interfaces (loans & cards):
```java
// LoansFeignClient.java
@GetMapping("/api/fetch")
ResponseEntity<LoansDto> fetchLoanDetails(
    @RequestHeader("eazybank-correlation-id") String correlationId,
    @RequestParam String mobileNumber);

// CardsFeignClient.java
@GetMapping("/api/fetch")
ResponseEntity<CardsDto> fetchCardDetails(
    @RequestHeader("eazybank-correlation-id") String correlationId,
    @RequestParam String mobileNumber);
```

#### 4. `LoansController.java` and `CardsController.java`:
Accept and log the correlation ID in their fetch APIs:
```java
private static final Logger logger = LoggerFactory.getLogger(LoansController.class);

@GetMapping("/api/fetch")
public ResponseEntity<LoansDto> fetchLoanDetails(
        @RequestHeader("eazybank-correlation-id") String correlationId,
        @RequestParam String mobileNumber) {
    logger.debug("eazybank-correlationId found: {}", correlationId);
    // ...
}
```

#### 5. Enable Debug Logging in each microservice's `application.yml`:
```yaml
# accounts
logging.level.com.eazybytes.accounts: DEBUG

# loans
logging.level.com.eazybytes.loans: DEBUG

# cards
logging.level.com.eazybytes.cards: DEBUG
```

### End-to-End Tracing Flow:
```
Client → Gateway Server
  → generates correlationId → adds to request header
  → forwards to accounts microservice (logs correlationId)
    → accounts calls loans (passes correlationId) → loans logs correlationId
    → accounts calls cards (passes correlationId) → cards logs correlationId
  → Gateway adds correlationId to response header
Client ← receives correlationId in response header
```

> Clients can use this correlationId when raising support tickets — developers trace logs using the same ID.

### Summary of Steps:
1. Create Spring Boot gateway project with required dependencies.
2. Add properties in `application.yml` (discovery locator, actuator, logging).
3. Define routing configurations (custom `RouteLocator` bean).
4. Create custom `GlobalFilter` classes (`RequestTraceFilter`, `ResponseTraceFilter`).
5. Update microservices to accept & log the correlation ID header.
6. Test via Postman — invoke `fetchCustomerDetails` through gateway at port `8072`.

---

## Lecture 12 — Design Patterns Around API Gateway

### Five Key Patterns:

#### 1. API Gateway Pattern
- A single component acts as the **entry point** into the microservices network.
- Handles routing, security, and communication streamlining.
- ✅ Implemented: Spring Cloud Gateway at port 8072.

#### 2. Gateway Routing Pattern
- The entry point can **route requests** to appropriate backend microservices based on URL, headers, or request params.
- ✅ Implemented: custom `RouteLocator` routing `/eazybank/accounts/**` → `lb://ACCOUNTS`.

#### 3. Gateway Offloading Pattern
- The edge server **offloads cross-cutting concerns** (security, caching, auditing, load balancing, SSL termination) from individual microservices.
- ✅ Implemented: correlation ID logic, response headers, future security via gateway.
- **Rule:** Only offload non-functional/cross-cutting concerns — never business logic.

#### 4. Backend For Frontend (BFF) Pattern
- Multiple edge servers, each **tailored for a specific client type** (web, mobile, tablet).
- Each gateway filters/transforms responses differently based on client needs.

| Client | Gateway | Response |
|---|---|---|
| Web App | Web Gateway | Full data (images, transactions, etc.) |
| Mobile App | Mobile Gateway | Compressed, minimal data |
| Tablet App | Tablet Gateway | Moderate data |

> Microservices always return the same response — the BFF layer filters it per client.

#### 5. Gateway Aggregator / Gateway Composition Pattern
- The API Gateway **invokes multiple microservices** and combines their responses into a single response for the client.
- Client makes one request; gateway internally calls accounts + loans + cards and returns combined result.
- Reduces chattiness between client and backend.

```
Client → Gateway (fetchSummary)
          ├─→ accounts microservice
          ├─→ loans microservice
          └─→ cards microservice
         ← combines all data
Client ← single combined response
```

> **Alternatives:**  
> a) Client calls each microservice separately (3 calls — not recommended).  
> b) One microservice calls others internally (e.g., accounts calls loans + cards — what we implemented).

---

## Lecture 13 — Generating and Pushing Docker Images with Spring Cloud Gateway Changes

### Pre-requisites:
1. Enable health probes in `accounts`, `loans`, and `cards` `application.yml`:
```yaml
management:
  endpoint:
    health:
      probes:
        enabled: true
  health:
    livenessState:
      enabled: true
    readinessState:
      enabled: true
```

2. Update all `pom.xml` tag names from `s8` → `s9`.

### Generate Docker Images (Jib):
Run from the terminal inside each service folder:
```bash
# accounts
mvn compile jib:dockerBuild

# loans
mvn compile jib:dockerBuild

# cards
mvn compile jib:dockerBuild

# configserver
mvn compile jib:dockerBuild

# eurekaserver
mvn compile jib:dockerBuild

# gatewayserver
mvn compile jib:dockerBuild
```

### Verify Generated Images:
```bash
docker images
# Should show 6 images all tagged with s9
```

### Push to Docker Hub:
```bash
docker image push docker.io/<your-username>/accounts:s9
docker image push docker.io/<your-username>/loans:s9
docker image push docker.io/<your-username>/cards:s9
docker image push docker.io/<your-username>/configserver:s9
docker image push docker.io/<your-username>/eurekaserver:s9
docker image push docker.io/<your-username>/gatewayserver:s9
```

---

## Lecture 14 — Updating Docker Compose File to Adapt Spring Cloud Gateway Changes

### Changes to `docker-compose.yml`:

#### 1. Update all image tags from `s8` → `s9` (find & replace all).

#### 2. Add Health Checks for accounts, loans, cards:
```yaml
accounts:
  # ...existing config...
  healthcheck:
    test: "curl --fail --silent localhost:8080/actuator/health/readiness | grep UP || exit 1"
    interval: 10s
    timeout: 5s
    retries: 10
    start_period: 10s

loans:
  # ...existing config...
  healthcheck:
    test: "curl --fail --silent localhost:8090/actuator/health/readiness | grep UP || exit 1"
    interval: 10s
    timeout: 5s
    retries: 10
    start_period: 10s

cards:
  # ...existing config...
  healthcheck:
    test: "curl --fail --silent localhost:9000/actuator/health/readiness | grep UP || exit 1"
    interval: 10s
    timeout: 5s
    retries: 10
    start_period: 10s
```

#### 3. Add Gateway Server Service:
```yaml
gatewayserver:
  image: eazybytes/gatewayserver:s9
  container_name: gatewayserver-ms
  ports:
    - "8072:8072"
  environment:
    SPRING_APPLICATION_NAME: "gatewayserver"
  extends:
    file: common-config.yml
    service: microservice-eureka-config
  depends_on:
    accounts:
      condition: service_healthy
    loans:
      condition: service_healthy
    cards:
      condition: service_healthy
```

> **`depends_on` with `service_healthy`** — gateway starts only after accounts, loans, and cards are healthy.

### Start All Containers:
```bash
docker compose up -d
```

### Startup Sequence (Automatic):
1. Config Server
2. Eureka Server
3. accounts, loans, cards (in parallel)
4. gatewayserver (after all above are healthy)

### Validation:
1. Check Eureka dashboard — all 4 services registered.
2. Invoke `fetchCustomerDetails` via Postman through gateway (port 8072).
3. Check response headers for `eazybank-correlationId`.
4. Verify logs in each container show matching correlation ID.

### Stop All Containers:
```bash
docker compose down
```

### Copy `docker-compose.yml` to other environments:
- `prod/docker-compose.yml`
- `qa/docker-compose.yml`
> Ensure each has the correct `spring.profiles.active` value (`prod` or `qa`).

---

## Section Summary

| What We Built | How |
|---|---|
| Edge Server / API Gateway | Spring Cloud Gateway at port 8072 |
| Default auto-routing | `spring.cloud.gateway.discovery.locator.enabled: true` |
| Lowercase service IDs in URL | `lower-case-service-id: true` |
| Custom routing with prefix `/eazybank` | Java `RouteLocator` bean |
| Response timestamp header | `addResponseHeader("X-Response-Time", ...)` |
| Correlation ID tracing | Custom `RequestTraceFilter` (pre) + `ResponseTraceFilter` (post) |
| Docker containerisation | Jib plugin, 6 Docker images tagged `s9` |
| Docker Compose orchestration | Health checks + `depends_on: service_healthy` |

### Design Patterns Implemented:
1. ✅ **API Gateway Pattern** — single entry point
2. ✅ **Gateway Routing Pattern** — URL-based routing
3. ✅ **Gateway Offloading Pattern** — correlation ID, headers
4. 📖 **Backend For Frontend (BFF)** — theory only
5. 📖 **Gateway Aggregator / Composition Pattern** — theory only

