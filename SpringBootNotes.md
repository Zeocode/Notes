# Spring – Bean, IOC, DI, AOP & Web Notes

---

## @Bean

- Tells Spring container to create and manage the Java object.
- Other Java objects **cannot** be seen by Spring unless annotated with `@Bean`.

### ProjectConfig.java

```java
@Configuration
@ComponentScan
class ProjectConfig {

    @Bean
    methodName() { }
}
```

### Application.java

```java
main() {
    // container = new AnnotationConfigApplicationContext(ProjectConfig.class);
    container.getBean("methodName", Vehicle.class);
}
```

---

## Spring IOC Container

> Manages objects, modules, and dependencies. Reduces manual object creation.

### Responsibilities

- Instantiate the application class
- Configure the application
- Assemble dependencies (wiring objects)

### Types of IOC

| Type | Description |
|------|-------------|
| `BeanFactory` | Basic container |
| `ApplicationContext` | Advanced container (most common) |

### Benefits of IOC

- Decoupling
- Less code
- Unit testing
- Reusability
- Independent development
- Easily replace modules

---

## NoUniqueBeanDefinitionException

Occurs when the container finds **more than one bean of the same type**.

```java
container.getBean(Vehicle.class); // has 3 beans → exception
```

---

## Different Ways to Name a Bean

### 1. Method Name (default)

Bean name defaults to the method name.

### 2. Using `@Bean` Annotation

```java
@Bean(name = "x")
@Bean(value = "x")
@Bean("x")
```

### 3. Multiple Beans of Same Class

```java
@Bean
vehicle1() { return new Vehicle(); }

@Bean
vehicle2() { return new Vehicle(); }

@Bean
vehicle3() { return new Vehicle(); }
```

---

## @Primary Annotation

Used when multiple beans of the same type exist — marks one as the default choice.

---

## @Bean vs @Component vs @Repository / @Service / @Controller

| Feature | `@Bean` | `@Component` |
|--------|---------|-------------|
| Instances | Multiple | Only one |
| Usage | Java config class | After component scan |
| Code | More code | Less code |
| Control | Full control | Less control |

> `@Repository`, `@Service`, `@Controller` are **stereotypes** of `@Component` used for specific layers.

---

## @PostConstruct & @PreDestroy

- `@PostConstruct` – Runs **after** bean construction
- `@PreDestroy` – Used for cleanup/initialization logic before destruction

---

## Add Bean Programmatically

```java
Supplier<Vehicle> volkswagenSupplier = () -> new Volkswagen();
context.registerBean(
    "volkswagen",
    Vehicle.class,
    volkswagenSupplier
);
```

---

## Bean with XML Configuration

```xml
<bean id="x" class="com.newXmlObject" />
```

```java
new ClassPathXmlApplicationContext("...");
context.close();
```

---

## Spring Wiring / Autowiring

Spring does **not** wire on its own. We wire via:

### 1. Method Call / Method Parameter

```java
@Bean
// Constructor injection (works with @Bean)
```

### 2. @Autowired

Simple change → more professional. Works with `@Bean`, `@Component` classes.

```java
@Autowired(required = false) // optional dependency
```

---

## Three Ways of Autowiring

| Method | Recommended? | Notes |
|--------|-------------|-------|
| Field injection | ❌ Not recommended | Can't use `final` fields |
| Setter injection | ⚠️ Not for PROD | Can't make field `final` |
| Constructor injection | ✅ Preferred | Supports `final`, most secure |

---

## Autowiring Multiple Beans

When multiple beans of same type exist, resolution order:

1. **Method name** – Container picks bean matching parameter name
2. **@Primary** – One bean marked primary is chosen by default
3. **@Qualifier** – Specify exact bean

```java
@Qualifier("name")
@Bean(name = "x")
@Component(value = "y")
constructor(@Qualifier("z") Vehicle v)
```

---

## Circular Dependency

```
Person ↔ Vehicle
```

Indicates a **cyclic dependency** issue — avoid or use `@Lazy`.

---

## @ComponentScan

```java
@ComponentScan(basePackages = "com.example")
@ComponentScan(basePackageClasses = MyClass.class)
```

Scans the base package and automatically detects Spring beans.

---

## getBeanNamesForType

```java
context.getBeanNamesForType(Person.class);
```

Returns all bean names of type `Person`. Useful when multiple beans exist.

---

## Bean Scopes

| Scope | Annotation | Description |
|-------|-----------|-------------|
| Singleton | `@Scope(BeanDefinition.SCOPE_SINGLETON)` | One instance (default) |
| Prototype | `@Scope(BeanDefinition.SCOPE_PROTOTYPE)` | New instance every time |
| Request | `@RequestScope` | Web – per HTTP request |
| Session | `@SessionScope` | Web – per HTTP session |
| Application | `@ApplicationScope` | Web – per app lifecycle |

### Eager vs Lazy

- **Default → Eager** (created at startup)
- **`@Lazy` → Lazy** (not created until first use)

---

## Singleton vs Prototype

| Feature | Singleton | Prototype |
|---------|-----------|-----------|
| Instances | One | New per request |
| Created | At startup | On demand |
| Mutability | Immutable (preferred) | Mutable |
| Usage | Pure logic, common | Mutable / multi-thread |

---

## Web Bean Scopes (HTTP Lifecycle)

### 1. @RequestScope

- New bean per HTTP request
- Destroyed after request completes
- Avoids storing unnecessary data

### 2. @SessionScope

- Lives for entire user session
- Persists across page refreshes
- Ideal for user-specific state
- ⚠️ Avoid storing too much (memory impact)

### 3. @ApplicationScope

- Lives for entire application lifetime
- Shared across **all users**
- Similar to Singleton but web-specific
- Good for config, counters, constants
- ⚠️ Never store per-user data here

---

## AOP (Aspect-Oriented Programming)

### Purpose

Used for **cross-cutting concerns**:
- Transaction management
- Logging
- Security
- Auditing

### Spring AOP

- Dynamically uses **proxies** (pluggable)
- Non-intrusive — doesn't modify original code

---

## Key AOP Terminology

| Term | Description |
|------|-------------|
| **Joinpoint** | Method inside the target object |
| **Target Object** | Actual object being advised (e.g., VehicleService bean) |
| **Pointcut** | Expression matching method execution |
| **Advice** | When to act — before/after/around |
| **Aspect** | The logic executed (before/after/around) |
| **Weaving** | Linking aspects with business logic |

> A **proxy object** is given which controls actual method invocation.

---

## Types of Advice

| Annotation | Description |
|------------|-------------|
| `@Before` | Runs before method |
| `@After` | Runs after method (always) |
| `@AfterReturning` | Runs after successful return |
| `@AfterThrowing` | Runs after exception |
| `@Around` | Wraps method (before + after) |

---

## Configuring AOP

```java
@Configuration
@EnableAspectJAutoProxy
class ProjectConfig { }
```

```java
@Aspect
@Component
public class LoggerAspect {

    @Around("execution(* com.example.services.*.*(..))")
    public void log(ProceedingJoinPoint joinPoint) throws Throwable {
        // Log info here
        joinPoint.proceed();
    }
}
```

### Ordering Advices

```java
@Order(1) // Controls priority when multiple advices apply
```

---

## Web Apps

### Before Spring (Servlets)

- One servlet per URL

### After Spring (DispatcherServlet)

- Single servlet for all URLs — `DispatcherServlet`

### Approaches

| Approach | Description |
|----------|-------------|
| Spring MVC (JSP) | With view/UI |
| Spring Boot | No view/UI (REST APIs) |

---

## Spring Boot Features

### Before Spring Boot (Traditional Spring)

- Configure manually (Maven/Gradle)
- Understand servlet internals
- Package WAR, deploy manually
- Manual application monitoring

### After Spring Boot

- Automatic configuration
- Embedded web server
- Production-ready features (Actuator, metrics, monitoring)

---

## @SpringBootApplication

```java
@SpringBootApplication
// Includes:
// @Configuration
// @EnableAutoConfiguration
// @ComponentScan
```

---

## Spring Boot Autoconfiguration

Enable debug output in `application.properties`:
```properties
debug=true
```

| Match Type | Description |
|-----------|-------------|
| Positive match | Dependency in `pom.xml` → bean autoconfigured |
| Negative match | No dependency OR custom bean defined → not autoconfigured |
| Exclusion | `@EnableAutoConfiguration(exclude = ...)` |
| Unconditional | Always configured regardless of dependencies |

---

## Default Port & Context Path

```properties
server.port=8080       # default
server.port=0          # random port (useful for testing)
```

---

## Lombok

| Annotation | Generates |
|-----------|-----------|
| `@Data` | Getters, Setters, Equals, HashCode, ToString |
| `@Slf4j` | Logger instance automatically |

---

## @RequestParam vs @PathVariable

| Annotation | Usage |
|-----------|-------|
| `@RequestParam` | Query parameters — filter/page params (`?key=value`) |
| `@PathVariable` | Path parameters (`/users/{id}`) |

---

## Validation

```java
@Valid       // validates request body
@Validated   // validates method params
```

- Used to secure and validate API input
- Prevents unwanted/malformed data
- Supports **custom annotations** for custom validation logic

---

## @ControllerAdvice

A global `@Component` that:

- Handles **global exceptions** across all controllers
- Shares common logic across multiple controllers
- Contains methods annotated with `@ExceptionHandler`

### @ExceptionHandler

- Intercepts and resolves exceptions at the global level
- Can handle **multiple exception types**
- Applies to web-layer (controller) exceptions

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleException(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                             .body(ex.getMessage());
    }
}
```

---

## RestTemplate & FeignClient

Used for making HTTP calls to external services / microservices.

| Tool | Description |
|------|-------------|
| `RestTemplate` | Traditional HTTP client (synchronous) |
| `FeignClient` | Declarative HTTP client (annotation-based) |

### RestTemplate

Synchronous HTTP client used to make REST API calls to external services or microservices.

```java
@Service
public class PaymentService {

    private final RestTemplate restTemplate;

    public PaymentService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    public String getPaymentStatus(int paymentId) {
        String url = "https://api.example.com/payments/" + paymentId;
        ResponseEntity<String> response = restTemplate.getForEntity(url, String.class);
        return response.getBody();
    }

    public Payment createPayment(Payment payment) {
        String url = "https://api.example.com/payments";
        ResponseEntity<Payment> response = restTemplate.postForEntity(url, payment, Payment.class);
        return response.getBody();
    }
}
```

> ⚙️ Register `RestTemplate` as a bean in your config:
```java
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

---

### FeignClient

Declarative HTTP client — define an interface with annotations and Spring generates the implementation automatically.

| Feature | RestTemplate | FeignClient |
|---------|-------------|------------|
| Style | Imperative | Declarative |
| Code | More boilerplate | Less code |
| Integration | Standard Spring | Spring Cloud |

**1️⃣ Add the dependency (`build.gradle`):**

```groovy
implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
```

**2️⃣ Enable Feign in your main/config class:**

```java
@SpringBootApplication
@EnableFeignClients
public class ExpenseManagerApplication { }
```

**3️⃣ Define the Feign client interface:**

```java
@FeignClient(name = "payment-service", url = "https://api.example.com")
public interface PaymentClient {

    @GetMapping("/payments/{id}")
    String getPaymentStatus(@PathVariable("id") int paymentId);

    @PostMapping("/payments")
    Payment createPayment(@RequestBody Payment payment);
}
```

**4️⃣ Inject and use it like any other Spring bean:**

```java
@Service
public class PaymentService {

    private final PaymentClient paymentClient;

    public PaymentService(PaymentClient paymentClient) {
        this.paymentClient = paymentClient;
    }

    public String getStatus(int id) {
        return paymentClient.getPaymentStatus(id);
    }
}
```

> 💡 No HTTP boilerplate needed — Spring generates the implementation at runtime.


---

## @Profile

Used to register beans **conditionally** based on the active environment (e.g., `dev`, `prod`, `test`).
Spring only creates the bean if the specified profile is currently active.

---

### 1️⃣ Define Profile-Specific Beans

```java
// Runs only in "dev" profile — uses a fake/mock payment gateway
@Component
@Profile("dev")
public class MockPaymentService implements PaymentService {

    @Override
    public String processPayment(double amount) {
        return "DEV: Simulated payment of $" + amount;
    }
}
```

```java
// Runs only in "prod" profile — uses the real payment gateway
@Component
@Profile("prod")
public class StripePaymentService implements PaymentService {

    @Override
    public String processPayment(double amount) {
        // Real Stripe API call here
        return "PROD: Payment of $" + amount + " processed via Stripe";
    }
}
```

---

### 2️⃣ Activate a Profile in `application.properties`

```properties
# application.properties
spring.profiles.active=dev
```

Or use **profile-specific property files**:

```
application.properties          ← shared config
application-dev.properties      ← loaded when profile = dev
application-prod.properties     ← loaded when profile = prod
```

```properties
# application-dev.properties
server.port=8080
db.url=jdbc:h2:mem:devdb

# application-prod.properties
server.port=443
db.url=jdbc:postgresql://prod-server/expensedb
```

---

### 3️⃣ @Profile on a @Configuration Class

```java
@Configuration
@Profile("dev")
public class DevDataSourceConfig {

    @Bean
    public DataSource dataSource() {
        // In-memory H2 database for development
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .build();
    }
}
```

```java
@Configuration
@Profile("prod")
public class ProdDataSourceConfig {

    @Bean
    public DataSource dataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:postgresql://prod-server/expensedb");
        ds.setUsername("prod_user");
        ds.setPassword("secret");
        return ds;
    }
}
```

---

### 4️⃣ Activate Profile in Tests with `@ActiveProfiles`

```java
@SpringBootTest
@ActiveProfiles("dev")          // forces the dev profile during this test
class PaymentServiceTest {

    @Autowired
    PaymentService paymentService;  // injects MockPaymentService

    @Test
    void testMockPayment() {
        String result = paymentService.processPayment(100.0);
        assertEquals("DEV: Simulated payment of $100.0", result);
    }
}
```

---

### 5️⃣ Multiple Profiles on One Bean

```java
@Component
@Profile({"dev", "test"})   // active in BOTH dev and test environments
public class SeedDataLoader implements CommandLineRunner {

    @Override
    public void run(String... args) {
        System.out.println("Loading seed data for dev/test...");
    }
}
```

---

### 🔑 Key Points

| Feature | Detail |
|---------|--------|
| `@Profile("dev")` | Bean only created when `dev` profile is active |
| `spring.profiles.active` | Set active profile in `application.properties` |
| `application-{profile}.properties` | Profile-specific config files |
| `@ActiveProfiles` | Use in tests to set the active profile |
| Multiple profiles | `@Profile({"dev", "test"})` |
| Default profile | `@Profile("default")` — used when no profile is set |
