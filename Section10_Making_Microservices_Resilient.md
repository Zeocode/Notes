# Section 10: Making Microservices Resilient

> **Challenge 7** — Making microservices fault-tolerant and resilient using **Resilience4j**

---

## Table of Contents

| Lec # | Topic |
|-------|-------|
| 1 | [Introduction to the Need of Resiliency](#lec-1--introduction-to-the-need-of-resiliency) |
| 2 | [Typical Use Case / Scenario for Resiliency](#lec-2--typical-use-case--scenario-for-resiliency) |
| 3 | [Deep Dive on Circuit Breaker Pattern](#lec-3--deep-dive-on-circuit-breaker-pattern) |
| 4 | [Three States of Circuit Breaker Pattern](#lec-4--three-states-of-circuit-breaker-pattern) |
| 5 | [Implementing Circuit Breaker in Gateway – Part 1](#lec-5--implementing-circuit-breaker-in-gateway--part-1) |
| 6 | [Implementing Circuit Breaker in Gateway – Part 2 (Fallback)](#lec-6--implementing-circuit-breaker-in-gateway--part-2-fallback) |
| 7 | [Implementing Circuit Breaker with Feign Client – Part 1](#lec-7--implementing-circuit-breaker-with-feign-client--part-1) |
| 8 | [Implementing Circuit Breaker with Feign Client – Part 2 (Demo)](#lec-8--implementing-circuit-breaker-with-feign-client--part-2-demo) |
| 9 | [HTTP Timeout Configurations](#lec-9--http-timeout-configurations) |
| 10 | [Introduction to Retry Pattern](#lec-10--introduction-to-retry-pattern) |
| 11 | [Implementing Retry Pattern in Gateway](#lec-11--implementing-retry-pattern-in-gateway) |
| 12 | [Implementing Retry Pattern in Accounts – Part 1](#lec-12--implementing-retry-pattern-in-accounts--part-1) |
| 13 | [Implementing Retry Pattern in Accounts – Part 2](#lec-13--implementing-retry-pattern-in-accounts--part-2) |
| 14 | [Introduction to Rate Limiter Pattern](#lec-14--introduction-to-rate-limiter-pattern) |
| 15 | [Introduction to Redis RateLimiter in Gateway](#lec-15--introduction-to-redis-ratelimiter-in-gateway) |
| 16 | [Implementing Redis RateLimiter in Gateway](#lec-16--implementing-redis-ratelimiter-in-gateway) |
| 17 | [Implementing RateLimiter Pattern in Accounts](#lec-17--implementing-ratelimiter-pattern-in-accounts) |
| 18 | [Introduction to Bulkhead Pattern](#lec-18--introduction-to-bulkhead-pattern) |
| 19 | [Aspect Order of Resiliency Patterns](#lec-19--aspect-order-of-resiliency-patterns) |
| 20 | [Demo with Docker Containers & Docker Compose](#lec-20--demo-with-docker-containers--docker-compose) |

---

## Lec 1 – Introduction to the Need of Resiliency

### What is Resiliency?
- Resiliency = the ability to **withstand tough times and bounce back**.
- Microservices face real-world challenges: network problems, performance issues, slow responses, temporary failures.

### Key Problems to Solve

| Problem | Description |
|---------|-------------|
| **Cascading Failures** | If one microservice fails, the ripple effect can bring down the entire chain |
| **Graceful Fallback** | When one service fails, return partial data instead of a hard exception |
| **Self-Healing** | Use timeouts and retries to give failed services time to recover |

### Solution: Resilience4j Library

- **Hystrix** (Netflix) was the old library — entered maintenance mode in **2018**.
- **Resilience4j** replaced it — lightweight, designed for **functional programming**.

#### Core Patterns in Resilience4j

| Pattern | Purpose |
|---------|---------|
| **Circuit Breaker** | Stop cascading failures by cutting off traffic to a failing service |
| **Fallback** | Return default/cached response when a service fails |
| **Retry** | Retry failed calls automatically |
| **Rate Limiter** | Limit the number of requests per time window |
| **Bulkhead** | Isolate resource pools for different services |
| **Time Limiter** | Set a maximum wait time for an operation |

> 🔗 Official docs: [https://resilience4j.readme.io](https://resilience4j.readme.io)

---

## Lec 2 – Typical Use Case / Scenario for Resiliency

### Scenario: `fetchCustomerDetails` API

```
Client App
    → Gateway Server (Edge Server)
        → Accounts Microservice
            → Loans Microservice  (cards details)
            → Cards Microservice  ⚠️ (failing/slow)
```

### What Happens When Cards Microservice is Slow?

1. **Accounts MS** waits 10+ seconds for Cards response → thread is blocked.
2. **Gateway Server** also waits for Accounts → its threads get blocked too.
3. **All other APIs** via the Gateway are now slow (ripple effect).
4. One bad microservice has degraded the **entire network**.

### Solution → **Circuit Breaker Pattern**

---

## Lec 3 – Deep Dive on Circuit Breaker Pattern

### Analogy: Electrical Circuit Breaker
- An electrical circuit breaker **trips** (opens) when it detects excess current → protects components.
- Similarly, software circuit breaker **opens** when too many calls fail → protects upstream services.

### How Software Circuit Breaker Works

1. **Monitors** all remote calls to a microservice.
2. If a service is **slow or unresponsive**, it kills the call.
3. If **majority of calls** are failing, the circuit breaker **opens** → immediately fails all future requests.
4. Downstream services (Accounts, Gateway) are **not blocked** waiting for a dead service.
5. **Periodically** sends partial traffic to check if the service has recovered.
6. Once recovery is confirmed → allows all traffic again.

### Primary Advantages

| Advantage | Explanation |
|-----------|-------------|
| **Fail Fast** | Instead of waiting 10s, fail within milliseconds |
| **Graceful Degradation** | Fallback mechanism sends partial/default response |
| **Self-Healing** | Gives the failing service 30–90s rest to recover |

---

## Lec 4 – Three States of Circuit Breaker Pattern

```
                ┌──────────────────────────────────────────┐
                │              CLOSED STATE                │
                │  (initial state, all requests allowed)   │
                │  Monitors success/failure rate           │
                └──────────────────┬───────────────────────┘
                                   │ failure rate >= threshold (e.g. 50%)
                                   ▼
                ┌──────────────────────────────────────────┐
                │               OPEN STATE                 │
                │  All requests FAIL IMMEDIATELY           │
                │  No calls sent to actual microservice    │
                │  Waits for configured duration (e.g. 90s)│
                └──────────────────┬───────────────────────┘
                                   │ after wait duration
                                   ▼
                ┌──────────────────────────────────────────┐
                │            HALF-OPEN STATE               │
                │  Allows limited requests (e.g. 10)       │
                │  If > 50% succeed → CLOSED               │
                │  If > 50% fail   → OPEN again            │
                └──────────────────────────────────────────┘
```

| State | Description |
|-------|-------------|
| **CLOSED** | All traffic flows normally; failures are monitored |
| **OPEN** | All requests fail immediately; service gets rest time |
| **HALF-OPEN** | Limited test requests; decides to close or re-open |

---

## Lec 5 – Implementing Circuit Breaker in Gateway – Part 1

### Step 1: Add Maven Dependency (Gateway Server `pom.xml`)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>
```
> ⚠️ Use the **reactor** variant because Gateway Server is built on **Spring WebFlux** (reactive).

### Step 2: Configure Circuit Breaker Filter in `GatewayServerApplication.java`

```java
@Bean
public RouteLocator eazyBankRouteConfig(RouteLocatorBuilder routeLocatorBuilder) {
    return routeLocatorBuilder.routes()
        .route(p -> p
            .path("/eazybank/accounts/**")
            .filters(f -> f
                .rewritePath("/eazybank/accounts/(?<segment>.*)", "/${segment}")
                .addResponseHeader("X-Response-Time", new Date().toString())
                // ✅ Add Circuit Breaker filter
                .circuitBreaker(config -> config
                    .setName("accountCircuitBreaker")
                    .setFallbackUri("forward:/contactSupport")
                )
            )
            .uri("lb://ACCOUNTS")
        )
        .build();
}
```

### Step 3: Configure Circuit Breaker Properties in `application.yml`

```yaml
resilience4j:
  circuitbreaker:
    configs:
      default:
        slidingWindowSize: 10               # Monitor 10 requests before deciding state
        permittedNumberOfCallsInHalfOpenState: 2  # Allow 2 calls in HALF-OPEN
        failureRateThreshold: 50            # Open if >= 50% of calls fail
        waitDurationInOpenState: 10000      # Wait 10s in OPEN before going HALF-OPEN
```

### Monitoring via Actuator

```
GET http://localhost:8072/actuator/circuitbreakers
GET http://localhost:8072/actuator/circuitbreakerevents?name=accountCircuitBreaker
```

#### Circuit Breaker States in Actuator Output

| Field | Meaning |
|-------|---------|
| `state` | Current state: `CLOSED`, `OPEN`, `HALF_OPEN` |
| `failureRate` | Percentage of failed calls |
| `bufferedCalls` | Total calls monitored in the sliding window |
| `failedCalls` | Number of failed calls |
| `notPermittedCalls` | Calls rejected while circuit is OPEN |

---

## Lec 6 – Implementing Circuit Breaker in Gateway – Part 2 (Fallback)

### Problem Without Fallback
- Without fallback: clients receive raw `504 Gateway Timeout` or `503 Service Unavailable`.
- Not acceptable for production — clients should get a meaningful message.

### Step 1: Create a `FallbackController` in Gateway Server

```java
package com.eazybank.gatewayserver.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Mono;

@RestController
public class FallbackController {

    @RequestMapping("/contactSupport")
    public Mono<String> contactSupport() {
        return Mono.just("An error occurred. Please try after some time or contact support team.");
    }
}
```

> ⚠️ Uses `Mono<String>` because the Gateway is built on **Spring Reactive (WebFlux)**.

### Step 2: Wire Fallback URI to Circuit Breaker

```java
.circuitBreaker(config -> config
    .setName("accountCircuitBreaker")
    .setFallbackUri("forward:/contactSupport")  // ✅ Fallback endpoint
)
```

### Summary: Steps to Implement Circuit Breaker in Gateway

```
1. Add dependency: spring-cloud-starter-circuitbreaker-reactor-resilience4j
2. In routing config: add .circuitBreaker(config -> config.setName(...).setFallbackUri(...))
3. Create FallbackController with a REST API at /contactSupport
4. Add properties in application.yml:
   - slidingWindowSize
   - permittedNumberOfCallsInHalfOpenState
   - failureRateThreshold
   - waitDurationInOpenState
```

---

## Lec 7 – Implementing Circuit Breaker with Feign Client – Part 1

### Context
- `fetchCustomerDetails` in **Accounts MS** calls **Loans MS** and **Cards MS** via **Feign Client**.
- If Loans or Cards is down → RuntimeException cascades up to Gateway.
- Solution: Enable **Circuit Breaker for OpenFeign**.

### Step 1: Add Maven Dependency (Accounts `pom.xml`)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
```
> ⚠️ Use the **non-reactor** variant for standard Spring MVC microservices.

### Step 2: Enable Circuit Breaker for OpenFeign in `application.yml`

```yaml
spring:
  cloud:
    openfeign:
      circuitbreaker:
        enabled: true
```

### Step 3: Create Fallback Classes

#### `LoansFallback.java`

```java
package com.eazybank.accounts.service.client;

import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Component;
import com.eazybank.accounts.dto.LoansDto;

@Component
public class LoansFallback implements LoansFeignClient {

    @Override
    public ResponseEntity<LoansDto> fetchLoanDetails(String correlationId, String mobileNumber) {
        return null;  // Return null to indicate loans data is unavailable
    }
}
```

#### `CardsFallback.java`

```java
package com.eazybank.accounts.service.client;

import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Component;
import com.eazybank.accounts.dto.CardsDto;

@Component
public class CardsFallback implements CardsFeignClient {

    @Override
    public ResponseEntity<CardsDto> fetchCardDetails(String correlationId, String mobileNumber) {
        return null;  // Return null to indicate cards data is unavailable
    }
}
```

### Step 4: Add `fallback` to `@FeignClient` Annotations

#### `LoansFeignClient.java`

```java
@FeignClient(name = "loans", fallback = LoansFallback.class)
public interface LoansFeignClient {
    @GetMapping(value = "/api/fetch", consumes = "application/json")
    ResponseEntity<LoansDto> fetchLoanDetails(
        @RequestHeader("eazybank-correlation-id") String correlationId,
        @RequestParam String mobileNumber
    );
}
```

#### `CardsFeignClient.java`

```java
@FeignClient(name = "cards", fallback = CardsFallback.class)
public interface CardsFeignClient {
    @GetMapping(value = "/api/fetch", consumes = "application/json")
    ResponseEntity<CardsDto> fetchCardDetails(
        @RequestHeader("eazybank-correlation-id") String correlationId,
        @RequestParam String mobileNumber
    );
}
```

### Step 5: Add Null Checks in `CustomerServiceImpl.java`

```java
// Loans
ResponseEntity<LoansDto> loansResponseEntity = loansFeignClient.fetchLoanDetails(correlationId, mobileNumber);
if (loansResponseEntity != null) {
    customerDetailsDto.setLoansDto(loansResponseEntity.getBody());
}

// Cards
ResponseEntity<CardsDto> cardsResponseEntity = cardsFeignClient.fetchCardDetails(correlationId, mobileNumber);
if (cardsResponseEntity != null) {
    customerDetailsDto.setCardsDto(cardsResponseEntity.getBody());
}
```

### Step 6: Add Circuit Breaker Properties in Accounts `application.yml`

```yaml
resilience4j:
  circuitbreaker:
    configs:
      default:
        slidingWindowSize: 10
        permittedNumberOfCallsInHalfOpenState: 2
        failureRateThreshold: 50
        waitDurationInOpenState: 10000
```

### Summary: Steps to Implement Circuit Breaker with Feign Client

```
1. Add dependency: spring-cloud-starter-circuitbreaker-resilience4j
2. Enable in application.yml: spring.cloud.openfeign.circuitbreaker.enabled=true
3. Create fallback classes implementing the FeignClient interfaces (@Component)
4. Add fallback = XxxFallback.class to @FeignClient annotations
5. Add null checks in service layer
6. Add resilience4j circuit breaker properties in application.yml
```

---

## Lec 8 – Implementing Circuit Breaker with Feign Client – Part 2 (Demo)

### Circuit Breaker Names in Actuator

When Feign Client circuit breakers activate, they appear in actuator with names like:
```
CardsFeignClient#fetchCardDetails(String,String)
LoansFeignClient#fetchLoanDetails(String,String)
```

### Demo Behavior

| Scenario | Result |
|----------|--------|
| All services running | Full `customerDetails` response (accounts + loans + cards) |
| Loans MS stopped | `loans: null`, accounts + cards returned (fallback active) |
| Both Loans + Cards stopped | `loans: null`, `cards: null`, only accounts returned |
| Multiple failures | Circuit opens → `notPermittedCalls` increase |

### Real-World Analogy (Amazon)
> If a recommendation microservice fails, Amazon still shows products, banners, and discounts. Only the recommendations section is hidden. The customer is **happy with partial data** rather than seeing a broken page.

---

## Lec 9 – HTTP Timeout Configurations

### Problem
- Without timeouts, a slow microservice causes Gateway threads to **wait indefinitely**.
- Gateway resources are wasted; all other APIs suffer.

### Why Accounts MS Behaves Differently from Loans MS

- Accounts MS has a **Circuit Breaker** (which has its own internal 1-second timeout).
- Loans MS has **no Circuit Breaker** → waits indefinitely.

### Solution: Global Timeout in Gateway `application.yml`

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        connect-timeout: 1000    # 1 second to establish connection
        response-timeout: 2s     # 2 seconds max to receive response
```

| Property | Description |
|----------|-------------|
| `connect-timeout` | Max time (ms) to get a connection to the target microservice |
| `response-timeout` | Max time to receive a response after connecting |

### Per-Route Timeout Override (Java DSL)

```java
.route(p -> p
    .path("/eazybank/cards/**")
    .filters(f -> f
        .rewritePath(...)
        .metadata(CONNECT_TIMEOUT_ATTR, 1000)        // ms
        .metadata(RESPONSE_TIMEOUT_ATTR, Duration.ofSeconds(5))
    )
    .uri("lb://CARDS")
)
```

### Disable Timeout for a Specific Route

```java
.metadata(RESPONSE_TIMEOUT_ATTR, -1)  // -1 = no timeout for this route
```

### Overriding Circuit Breaker's Internal Timeout

Circuit Breaker by default waits **1 second** (TimeLimiter). To change it in Gateway Server:

```java
@Bean
public Customizer<ReactiveResilience4JCircuitBreakerFactory> defaultCustomizer() {
    return factory -> factory.configureDefault(id -> new Resilience4JConfigBuilder(id)
        .circuitBreakerConfig(CircuitBreakerConfig.ofDefaults())
        .timeLimiterConfig(TimeLimiterConfig.custom()
            .timeoutDuration(Duration.ofSeconds(4))  // Increase to 4 seconds
            .build()
        )
        .build()
    );
}
```

---

## Lec 10 – Introduction to Retry Pattern

### What is the Retry Pattern?
- Automatically **retry a failed operation** multiple times before giving up.
- Especially useful for **transient network failures** where a retry might succeed.

### Key Concepts

#### Number of Retries
- Configure how many times to retry: 3, 5, 10 etc.
- Can be conditional: only retry on specific error codes / exceptions / response status.

#### Backoff Strategy

| Strategy | Description |
|----------|-------------|
| **Fixed** | Same delay between retries (e.g., every 2s) |
| **Exponential Backoff** | Delay doubles each time (2s → 4s → 8s → ...) — reduces system overload |

#### Exponential Backoff Example
```
Attempt 1 (initial)    → wait 100ms
Retry 1                → wait 200ms
Retry 2                → wait 400ms
Retry 3                → wait 800ms (capped by maxBackoff)
```

### ⚠️ Important Rule: Only Use Retry on Idempotent Operations

| HTTP Method | Safe for Retry? | Reason |
|-------------|-----------------|--------|
| GET | ✅ Yes | No side effects |
| DELETE | ⚠️ Careful | May delete twice |
| POST | ❌ No | Creates duplicate records |
| PUT / PATCH | ❌ No | Updates multiple times |

### Integration with Circuit Breaker
- Retry can be **combined with Circuit Breaker**.
- After N consecutive retry failures → Circuit Breaker opens.

---

## Lec 11 – Implementing Retry Pattern in Gateway

### Step: Add Retry Filter in `GatewayServerApplication.java`

```java
.route(p -> p
    .path("/eazybank/loans/**")
    .filters(f -> f
        .rewritePath("/eazybank/loans/(?<segment>.*)", "/${segment}")
        .addResponseHeader("X-Response-Time", new Date().toString())
        // ✅ Retry Filter
        .retry(retryConfig -> retryConfig
            .setRetries(3)                           // Retry 3 times after initial attempt
            .setMethods(HttpMethod.GET)              // Only retry GET requests
            .setBackoff(
                Duration.ofMillis(100),   // firstBackoff: wait 100ms before retry 1
                Duration.ofMillis(1000),  // maxBackoff: never wait more than 1000ms
                2,                        // factor: multiply previous backoff by 2
                true                      // basedOnPreviousValue: apply factor on previous backoff
            )
        )
    )
    .uri("lb://LOANS")
)
```

### How Retries Work in Gateway

```
Initial Request → FAIL (timeout)
    Retry 1 (after 100ms) → FAIL
    Retry 2 (after 200ms) → FAIL
    Retry 3 (after 400ms) → FAIL
→ Return final error to client

Total time ≈ 4 × response-timeout (e.g. 4 × 2s = 8s)
```

### Fix: Duplicate Correlation ID Header Bug

When retries happen, the `ResponseTraceFilter` adds the correlation ID header multiple times.

**Fix in `ResponseTraceFilter.java`:**

```java
@Override
public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
    return chain.filter(exchange).then(Mono.fromRunnable(() -> {
        HttpHeaders requestHeaders = exchange.getRequest().getHeaders();
        String correlationId = requestHeaders.getFirst(EazyBankConstants.CORRELATION_ID);

        // ✅ Only add if not already present in response
        if (!exchange.getResponse().getHeaders().containsKey(EazyBankConstants.CORRELATION_ID)) {
            log.debug("Updated the correlation id to the outbound headers: {}", correlationId);
            exchange.getResponse().getHeaders().add(EazyBankConstants.CORRELATION_ID, correlationId);
        }
    }));
}
```

---

## Lec 12 – Implementing Retry Pattern in Accounts – Part 1

### Step 1: Add `@Retry` Annotation on the Method

```java
// AccountsController.java

private static final Logger logger = LoggerFactory.getLogger(AccountsController.class);

@Retry(name = "getBuildInfo", fallbackMethod = "getBuildInfoFallback")
@GetMapping("/build-info")
public ResponseEntity<String> getBuildInfo() {
    logger.debug("getBuildInfo() method invoked");
    return ResponseEntity.ok(buildVersion);  // Original logic
}

// ✅ Fallback method — must have SAME signature + extra Throwable parameter
public ResponseEntity<String> getBuildInfoFallback(Throwable throwable) {
    logger.debug("getBuildInfoFallback() method invoked");
    return ResponseEntity.ok("0.9");         // Default fallback version
}
```

#### Rules for Fallback Method

| Rule | Detail |
|------|--------|
| Same return type | `ResponseEntity<String>` in both original and fallback |
| Same parameters | Original params + one extra `Throwable` param at the end |
| Different name | Cannot have the same method name — suffix with `Fallback` |

### Step 2: Add Retry Properties in Accounts `application.yml`

```yaml
resilience4j:
  retry:
    configs:
      default:
        maxRetryAttempts: 3           # Retry up to 3 times (includes initial call)
        waitDuration: 100             # Wait 100ms between retries
        enableExponentialBackoff: true
        exponentialBackoffMultiplier: 2  # Each retry waits 2x longer
```

### Retry Count Difference: Gateway vs Individual Microservice

| Approach | Initial Call | Retries | Total |
|----------|-------------|---------|-------|
| **Gateway Server** | 1 | 3 (additional) | **4** |
| **Individual MS** | counted as 1st retry | 3 (includes initial) | **3** |

### Step 3: Overriding Circuit Breaker Time Limiter

When `waitDuration` is 500ms and 3 retries → total wait ~1.5s.
This **exceeds** the default circuit breaker TimeLimiter (1s) → circuit breaker fallback triggers first.

**Fix:** Increase TimeLimiter timeout (see Lec 9 code — set to 4s).

---

## Lec 13 – Implementing Retry Pattern in Accounts – Part 2

### Configure `ignoreExceptions` (Do NOT retry for specific exceptions)

```yaml
resilience4j:
  retry:
    configs:
      default:
        maxRetryAttempts: 3
        waitDuration: 100
        enableExponentialBackoff: true
        exponentialBackoffMultiplier: 2
        ignoreExceptions:
          - java.lang.NullPointerException    # Don't retry on NullPointerException
```

> 💡 **Why?** If a `NullPointerException` occurs with specific input, retrying with the same input always gives the same result — retrying is pointless.

### Configure `retryExceptions` (ONLY retry for specific exceptions)

```yaml
resilience4j:
  retry:
    configs:
      default:
        maxRetryAttempts: 3
        waitDuration: 100
        enableExponentialBackoff: true
        exponentialBackoffMultiplier: 2
        retryExceptions:
          - java.util.concurrent.TimeoutException  # Only retry on TimeoutException
```

> ⚠️ When `retryExceptions` is defined, you **don't need** `ignoreExceptions` — all other exceptions are automatically ignored.

### Configuring Retry Exceptions in Gateway Server

```java
.retry(retryConfig -> retryConfig
    .setRetries(3)
    .setMethods(HttpMethod.GET)
    .setExceptions(ConnectException.class, TimeoutException.class)  // Only retry these
    // OR
    .setStatuses(HttpStatus.INTERNAL_SERVER_ERROR, HttpStatus.BAD_GATEWAY)
)
```

### Summary: Steps to Configure Retry in Individual Microservice

```
1. Add @Retry(name = "xxx", fallbackMethod = "xxxFallback") on the method
2. Create fallback method: same signature + Throwable parameter at the end
3. Configure properties in application.yml:
   - maxRetryAttempts
   - waitDuration
   - enableExponentialBackoff
   - exponentialBackoffMultiplier
   - ignoreExceptions (optional)
   - retryExceptions (optional)
```

---

## Lec 14 – Introduction to Rate Limiter Pattern

### What is Rate Limiter?
- **Controls and limits the rate of incoming requests** to a specific API or microservice.
- Protects the system from being overwhelmed by excessive or malicious requests.

### Why Use Rate Limiter?

| Use Case | Problem Solved |
|----------|----------------|
| One user sends millions of requests | Prevents DoS attacks |
| Misbehaving client script | Stops accidental traffic spikes |
| Low-priority API eating resources | Ensures high-priority APIs have resources |
| Subscription tiers (Basic/Premier/Enterprise) | Enforce different limits per tier |

### How It Works

- Returns **HTTP 429 Too Many Requests** when limit is exceeded.
- Can be enforced based on: **User**, **IP Address**, **Session**, **Server**, **Tenant**.

### Rate Limiter vs Other Patterns

| Pattern | Purpose |
|---------|---------|
| Circuit Breaker | Protect from downstream failures |
| Retry | Handle temporary failures |
| **Rate Limiter** | Protect from excessive incoming traffic |

---

## Lec 15 – Introduction to Redis RateLimiter in Gateway

### Token Bucket Algorithm (used by Spring Cloud Gateway)

```
 ┌─────────────────────────────────────────────────┐
 │                  TOKEN BUCKET                   │
 │                                                 │
 │   replenishRate = 100 tokens/second             │
 │   burstCapacity = 200 tokens max                │
 │   requestedTokens = 1 token per request         │
 │                                                 │
 │   Each request "consumes" tokens from bucket    │
 │   Bucket refills at replenishRate per second    │
 │   Bucket max capacity = burstCapacity           │
 └─────────────────────────────────────────────────┘
```

### Key Properties

| Property | Description | Example |
|----------|-------------|---------|
| `replenishRate` | Tokens added per second | `1` = 1 request/sec |
| `burstCapacity` | Max tokens bucket can hold | `200` = allow burst of 200 |
| `requestedTokens` | Tokens consumed per request | `1` = default, `60` = 1/minute |

### Configuration Examples

#### Allow 1 Request Per Minute
```
replenishRate:   1   (1 token per second added)
burstCapacity:  60   (max 60 tokens can accumulate)
requestedTokens: 60  (each request costs 60 tokens)
```

#### Allow Steady Rate of 100 req/sec
```
replenishRate:  100
burstCapacity:  100
requestedTokens:  1
```

#### Allow 100 req/sec with Temporary Burst up to 200
```
replenishRate:  100
burstCapacity:  200  (unused tokens from prev second carry over)
requestedTokens:  1
```

### KeyResolver
- Determines **who** the limit applies to.
- Default: `PrincipalNameKeyResolver` (Spring Security logged-in user).
- Custom: Based on header, IP, query param, etc.

---

## Lec 16 – Implementing Redis RateLimiter in Gateway

### Step 1: Add Redis Dependency (Gateway Server `pom.xml`)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

### Step 2: Create Beans in `GatewayServerApplication.java`

```java
// Bean 1: KeyResolver — defines the key for rate limiting (by "user" header)
@Bean
public KeyResolver userKeyResolver() {
    return exchange -> Mono.justOrEmpty(
        exchange.getRequest().getHeaders().getFirst("user")
    ).defaultIfEmpty("anonymous");
}

// Bean 2: RedisRateLimiter — token bucket config
@Bean
public RedisRateLimiter redisRateLimiter() {
    return new RedisRateLimiter(1, 1, 1);
    // replenishRate=1, burstCapacity=1, requestedTokens=1
    // = 1 request per second per user
}
```

### Step 3: Add RateLimiter Filter to Card Routes

```java
.route(p -> p
    .path("/eazybank/cards/**")
    .filters(f -> f
        .rewritePath("/eazybank/cards/(?<segment>.*)", "/${segment}")
        .addResponseHeader("X-Response-Time", new Date().toString())
        // ✅ RateLimiter Filter
        .requestRateLimiter(config -> config
            .setRateLimiter(redisRateLimiter())
            .setKeyResolver(userKeyResolver())
        )
    )
    .uri("lb://CARDS")
)
```

### Step 4: Configure Redis in `application.yml` (Gateway Server)

```yaml
spring:
  data:
    redis:
      connect-timeout: 2s
      host: localhost
      port: 6379
      timeout: 1s
```

### Step 5: Start Redis Container

```bash
docker run -p 6379:6379 --name eazyredis -d redis
```

### Demo Result with Apache Benchmark

```bash
ab -n 10 -c 2 -v 3 http://localhost:8072/eazybank/cards/api/contact-info
```

```
Total requests: 10
Failed requests: 9
→ First request returns HTTP 200
→ Remaining 9 return HTTP 429 Too Many Requests
```

---

## Lec 17 – Implementing RateLimiter Pattern in Accounts

### Step 1: Add `@RateLimiter` Annotation on Method

```java
// AccountsController.java

@RateLimiter(name = "getJavaVersion", fallbackMethod = "getJavaVersionFallback")
@GetMapping("/java-version")
public ResponseEntity<String> getJavaVersion() {
    return ResponseEntity.ok(environment.getProperty("JAVA_HOME"));
}

// Fallback method
public ResponseEntity<String> getJavaVersionFallback(Throwable throwable) {
    return ResponseEntity.ok("Java 17");  // Default fallback value
}
```

### Step 2: Add RateLimiter Properties in `application.yml`

```yaml
resilience4j:
  ratelimiter:
    configs:
      default:
        limitRefreshPeriod: 5000   # Refresh quota every 5 seconds (ms)
        limitForPeriod: 1          # Allow only 1 request per 5 seconds
        timeoutDuration: 1000      # Wait max 1s for a quota slot before failing (ms)
```

### Property Explanations

| Property | Description |
|----------|-------------|
| `limitRefreshPeriod` | Time window for quota renewal (ms) |
| `limitForPeriod` | Number of requests allowed per window |
| `timeoutDuration` | Max time a thread waits for quota before giving up |

### Key Difference: Gateway vs Individual MS Rate Limiter

| Approach | Criteria | Use Case |
|----------|----------|---------|
| **Gateway + Redis** | Per user/IP/session (KeyResolver) | Multi-tenant, per-user limits |
| **Individual MS @RateLimiter** | Global (all requests to that API) | Protect low-priority APIs, resource limits |

---

## Lec 18 – Introduction to Bulkhead Pattern

### Inspiration: Ship Bulkheads
- Ships have physical **compartments (bulkheads)** to prevent one flooded section from sinking the whole ship.
- Similarly, Bulkhead pattern **isolates resources** per API to prevent one heavy API from starving others.

### Problem Without Bulkhead

```
Accounts Microservice (shared thread pool)
│
├── /myAccount          → Simple, fast (needs few threads)
└── /myCustomerDetails  → Complex, invokes Loans + Cards

If too many /myCustomerDetails requests arrive:
→ All threads consumed
→ /myAccount cannot get threads
→ /myAccount becomes slow/unavailable
```

### Solution With Bulkhead

```
Accounts Microservice
│
├── /myAccount          → [Thread Pool A: max 10 threads]
└── /myCustomerDetails  → [Thread Pool B: max 20 threads]

Each API has its OWN resource boundary.
/myAccount always has threads, regardless of /myCustomerDetails load.
```

### Implementation (Resilience4j — no Gateway support)

#### Annotation

```java
@Bulkhead(name = "getBuildInfo", type = Bulkhead.Type.THREADPOOL)
@GetMapping("/build-info")
public CompletableFuture<ResponseEntity<String>> getBuildInfo() {
    return CompletableFuture.supplyAsync(() ->
        ResponseEntity.ok(buildVersion)
    );
}
```

### Configuration Properties

```yaml
resilience4j:
  bulkhead:
    configs:
      default:
        maxConcurrentCalls: 10     # Max concurrent requests
        maxWaitDuration: 10ms      # Max wait for a slot

  thread-pool-bulkhead:
    configs:
      default:
        maxThreadPoolSize: 10      # Max threads in pool
        coreThreadPoolSize: 2      # Core threads always alive
        queueCapacity: 20          # Waiting queue size
```

> ⚠️ Bulkhead demo requires **commercial tools** like JMeter or LoadRunner to visualize thread usage.

---

## Lec 19 – Aspect Order of Resiliency Patterns

### Default Execution Order (when multiple patterns on same method)

```
Bulkhead → TimeLimiter → RateLimiter → CircuitBreaker → Retry
(highest priority)                                 (lowest priority / outermost wrapper)
```

In other words, the **outermost** wrapper is **Retry**, meaning:
- Retry wraps CircuitBreaker wraps RateLimiter wraps TimeLimiter wraps Bulkhead

### Customizing the Order in `application.yml`

```yaml
# Give Retry higher priority (executed first/outermost):
resilience4j:
  retry:
    retryAspectOrder: 2
  circuitbreaker:
    circuitBreakerAspectOrder: 1
```

> Higher number = higher priority = executes first (outermost wrapper).

### ⚠️ Best Practice Advice

> Do **NOT** combine all patterns on every API. That's **over-engineering**.
> - Pick the pattern(s) that solve your specific problem.
> - Test thoroughly before going to production.
> - Multiple patterns combined without testing can cause **unexpected surprises in production**.

---

## Lec 20 – Demo with Docker Containers & Docker Compose

### Changes to `docker-compose.yml`

#### 1. Add Redis Service

```yaml
redis:
  image: redis
  ports:
    - "6379:6379"
  healthcheck:
    test: [ "CMD-SHELL", "redis-cli ping | grep PONG" ]
    timeout: 10s
    retries: 10
  extends:
    file: common-config.yml
    service: network-deploy-service
```

#### 2. Update Gateway Server Service

```yaml
gatewayserver:
  image: eazybytes/gatewayserver:s10
  depends_on:
    redis:
      condition: service_healthy
  environment:
    # ... existing env vars ...
    SPRING_DATA_REDIS_CONNECT-TIMEOUT: 2s
    SPRING_DATA_REDIS_HOST: redis
    SPRING_DATA_REDIS_PORT: 6379
    SPRING_DATA_REDIS_TIMEOUT: 1s
```

#### 3. Update All Service Tags (s9 → s10)

```yaml
accounts:
  image: eazybytes/accounts:s10

loans:
  image: eazybytes/loans:s10

cards:
  image: eazybytes/cards:s10

configserver:
  image: eazybytes/configserver:s10

eurekaserver:
  image: eazybytes/eurekaserver:s10

gatewayserver:
  image: eazybytes/gatewayserver:s10
```

### Commands

```bash
# Start all containers
docker compose up -d

# Stop all containers
docker compose down

# Push new Docker images (via Google Jib)
mvn compile jib:build
```

### Docker Build Tag Change in `pom.xml`

```xml
<to>
    <image>docker.io/eazybytes/accounts:s10</image>
</to>
```

---

## Complete Code Summary

### Files Changed in Section 10

| File | Change |
|------|--------|
| `gatewayserver/pom.xml` | Add `reactor-resilience4j` + `redis-reactive` dependencies |
| `GatewayServerApplication.java` | Add CircuitBreaker, Retry, RateLimiter filters; KeyResolver + RedisRateLimiter beans; TimeLimiter customizer |
| `FallbackController.java` | New controller with `/contactSupport` endpoint |
| `ResponseTraceFilter.java` | Fix duplicate correlation ID header on retries |
| `gatewayserver/application.yml` | Add circuit breaker, http timeout, redis properties |
| `accounts/pom.xml` | Add `resilience4j` dependency |
| `accounts/application.yml` | Add circuit breaker, retry, rate limiter properties; feign circuit breaker enabled |
| `LoansFeignClient.java` | Add `fallback = LoansFallback.class` |
| `CardsFeignClient.java` | Add `fallback = CardsFallback.class` |
| `LoansFallback.java` | New class implementing `LoansFeignClient` |
| `CardsFallback.java` | New class implementing `CardsFeignClient` |
| `CustomerServiceImpl.java` | Add null checks for loans/cards response |
| `AccountsController.java` | Add `@Retry`, `@RateLimiter` annotations + fallback methods |
| `docker-compose.yml` | Add Redis service, update tags to s10, add Redis env vars to gateway |

---

## Quick Reference: Pattern Comparison

| Pattern | Where Implemented | Annotation / Filter | Purpose |
|---------|------------------|---------------------|---------|
| **Circuit Breaker** | Gateway Server | `.circuitBreaker(...)` | Stop cascading failures |
| **Circuit Breaker** | Accounts MS (Feign) | `spring.cloud.openfeign.circuitbreaker.enabled=true` | Feign call protection |
| **HTTP Timeout** | Gateway Server | `spring.cloud.gateway.httpclient.*` | Global response timeout |
| **Retry** | Gateway Server | `.retry(...)` | Retry on network failure |
| **Retry** | Accounts MS | `@Retry(name, fallbackMethod)` | Retry with fallback |
| **Rate Limiter** | Gateway + Redis | `.requestRateLimiter(...)` | Per-user request limit |
| **Rate Limiter** | Accounts MS | `@RateLimiter(name, fallbackMethod)` | Global API rate limit |
| **Bulkhead** | Accounts MS (concept) | `@Bulkhead(name, type)` | Thread pool isolation |

---

## Resilience4j Dependencies Quick Reference

```xml
<!-- For Gateway Server (Spring WebFlux / Reactive) -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>

<!-- For Normal Microservices (Spring MVC) -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>

<!-- Redis for Gateway RateLimiter -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

