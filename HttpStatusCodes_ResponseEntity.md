# HTTP Status Codes & ResponseEntity — Spring Boot

---

## 📦 What is ResponseEntity?

`ResponseEntity` represents the **entire HTTP response** — status code, headers, and body.
It gives you full control over what you return from a controller method.

```java
ResponseEntity<T>  // T = type of the response body
```

---

## 🟢 2xx — Success

| Code | Name | Meaning |
|------|------|---------|
| `200` | OK | Request succeeded. Standard response for GET, PUT, PATCH |
| `201` | Created | Resource successfully created. Used after POST |
| `202` | Accepted | Request accepted but processing not yet complete (async) |
| `204` | No Content | Request succeeded but no body to return. Used after DELETE |

### Examples

```java
// 200 OK
@GetMapping("/users/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    User user = userService.findById(id);
    return ResponseEntity.ok(user);
    // OR: return ResponseEntity.status(HttpStatus.OK).body(user);
}

// 201 Created
@PostMapping("/users")
public ResponseEntity<User> createUser(@RequestBody User user) {
    User saved = userService.save(user);
    return ResponseEntity.status(HttpStatus.CREATED).body(saved);
}

// 201 Created with Location header
@PostMapping("/users")
public ResponseEntity<User> createUser(@RequestBody User user, UriComponentsBuilder uriBuilder) {
    User saved = userService.save(user);
    URI location = uriBuilder.path("/users/{id}").buildAndExpand(saved.getId()).toUri();
    return ResponseEntity.created(location).body(saved);
}

// 202 Accepted
@PostMapping("/reports/generate")
public ResponseEntity<String> generateReport() {
    reportService.triggerAsync();
    return ResponseEntity.accepted().body("Report generation started");
}

// 204 No Content
@DeleteMapping("/users/{id}")
public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
    userService.delete(id);
    return ResponseEntity.noContent().build();
}
```

---

## 🔵 3xx — Redirection

| Code | Name | Meaning |
|------|------|---------|
| `301` | Moved Permanently | Resource has permanently moved to a new URL. Client should update bookmarks |
| `302` | Found | Resource temporarily at a different URL. Client should keep using original URL |
| `303` | See Other | Redirect to another URL using GET (common after POST/PUT/DELETE) |
| `304` | Not Modified | Cached version is still valid — no need to resend the resource |


### Examples

```java
// 301 Moved Permanently
@GetMapping("/old-endpoint")
public ResponseEntity<Void> oldEndpoint() {
    URI newLocation = URI.create("/new-endpoint");
    return ResponseEntity.status(HttpStatus.MOVED_PERMANENTLY)
                         .location(newLocation)
                         .build();
}

// 302 Found (Temporary Redirect)
@GetMapping("/dashboard")
public ResponseEntity<Void> dashboard() {
    URI tempLocation = URI.create("/maintenance");
    return ResponseEntity.status(HttpStatus.FOUND)
                         .location(tempLocation)
                         .build();
}

// 303 See Other — redirect after form submit (POST → GET)
@PostMapping("/users")
public ResponseEntity<Void> createUser(@RequestBody User user) {
    User saved = userService.save(user);
    URI location = URI.create("/users/" + saved.getId());
    return ResponseEntity.status(HttpStatus.SEE_OTHER)
                         .location(location)
                         .build();
}

// 304 Not Modified — used with caching (ETag)
@GetMapping("/users/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id,
                                    @RequestHeader(value = "If-None-Match", required = false) String etag) {
    User user = userService.findById(id);
    String currentEtag = "\"" + user.getVersion() + "\"";
    if (currentEtag.equals(etag)) {
        return ResponseEntity.status(HttpStatus.NOT_MODIFIED).build();
    }
    return ResponseEntity.ok()
                         .eTag(currentEtag)
                         .body(user);
}

```

---

## 🔴 4xx — Client Errors

| Code | Name | Meaning |
|------|------|---------|
| `400` | Bad Request | Invalid input / malformed request body |
| `401` | Unauthorized | Not authenticated — login required |
| `402` | Payment Required | Access denied due to unpaid subscription / quota exceeded |
| `403` | Forbidden | Authenticated but not authorized to access resource |
| `404` | Not Found | Resource does not exist |
| `405` | Method Not Allowed | HTTP method not supported on this endpoint |
| `409` | Conflict | Resource conflict (e.g., duplicate entry) |
| `422` | Unprocessable Entity | Validation failed on well-formed request |
| `429` | Too Many Requests | Rate limit exceeded |

### Examples

```java
// 400 Bad Request
@PostMapping("/users")
public ResponseEntity<String> createUser(@RequestBody User user) {
    if (user.getName() == null || user.getName().isBlank()) {
        return ResponseEntity.badRequest().body("Name cannot be blank");
    }
    userService.save(user);
    return ResponseEntity.status(HttpStatus.CREATED).build();
}

// 401 Unauthorized
@GetMapping("/profile")
public ResponseEntity<String> getProfile() {
    return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                         .body("Please login to access this resource");
}

// 402 Payment Required
@GetMapping("/reports/export")
public ResponseEntity<String> exportReport() {
    if (!subscriptionService.isPremiumUser()) {
        return ResponseEntity.status(HttpStatus.PAYMENT_REQUIRED)
                             .body("Upgrade to a paid plan to access this feature");
    }
    return ResponseEntity.ok(reportService.export());
}

// 403 Forbidden
@DeleteMapping("/admin/users/{id}")
public ResponseEntity<String> deleteUser(@PathVariable Long id) {
    return ResponseEntity.status(HttpStatus.FORBIDDEN)
                         .body("You do not have permission to perform this action");
}

// 404 Not Found
@GetMapping("/users/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    return userService.findById(id)
            .map(user -> ResponseEntity.ok(user))
            .orElse(ResponseEntity.notFound().build());
}

// 409 Conflict
@PostMapping("/users")
public ResponseEntity<String> registerUser(@RequestBody User user) {
    if (userService.existsByEmail(user.getEmail())) {
        return ResponseEntity.status(HttpStatus.CONFLICT)
                             .body("Email already in use");
    }
    userService.save(user);
    return ResponseEntity.status(HttpStatus.CREATED).build();
}

// 422 Unprocessable Entity
@PostMapping("/orders")
public ResponseEntity<String> placeOrder(@RequestBody Order order) {
    if (order.getQuantity() <= 0) {
        return ResponseEntity.unprocessableEntity()
                             .body("Quantity must be greater than zero");
    }
    orderService.place(order);
    return ResponseEntity.status(HttpStatus.CREATED).build();
}
```

---

## 🔴 5xx — Server Errors

| Code | Name | Meaning |
|------|------|---------|
| `500` | Internal Server Error | Unexpected server-side error |
| `501` | Not Implemented | Feature/endpoint not yet implemented |
| `502` | Bad Gateway | Invalid response from upstream server |
| `503` | Service Unavailable | Server is down or overloaded |
| `504` | Gateway Timeout | Upstream server timed out |

> **When does 502 happen?**
> Your server is acting as a **gateway/proxy** (e.g., API Gateway, Nginx, Load Balancer) and the upstream service it called returned an **invalid or corrupt response** — or crashed mid-response.
> Common causes:
> - Microservice behind the gateway crashed or returned garbage
> - Upstream app server restarted while processing a request
> - Incompatible protocol between proxy and upstream
>
> **When does 504 happen?**
> Your server called an upstream service but **did not get a response within the allowed time**. The upstream is alive but too slow.
> Common causes:
> - Downstream microservice is overloaded and taking too long
> - Database query is running too long
> - External third-party API is unresponsive
> - Network latency between services is too high

### Examples

```java
// 500 Internal Server Error
@GetMapping("/data")
public ResponseEntity<String> getData() {
    try {
        String result = dataService.fetch();
        return ResponseEntity.ok(result);
    } catch (Exception e) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                             .body("Something went wrong: " + e.getMessage());
    }
}

// 503 Service Unavailable
@GetMapping("/health")
public ResponseEntity<String> healthCheck() {
    if (!dbService.isConnected()) {
        return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE)
                             .body("Database is currently unavailable");
    }
    return ResponseEntity.ok("Service is healthy");
}

// 502 Bad Gateway — upstream service returned an invalid response
@GetMapping("/orders/summary")
public ResponseEntity<String> getOrderSummary() {
    try {
        String result = inventoryServiceClient.getSummary(); // calling another microservice
        return ResponseEntity.ok(result);
    } catch (InvalidResponseException e) {
        // upstream service returned something unexpected
        return ResponseEntity.status(HttpStatus.BAD_GATEWAY)
                             .body("Upstream service returned an invalid response");
    }
}

// 504 Gateway Timeout — upstream service took too long to respond
@GetMapping("/payments/status")
public ResponseEntity<String> getPaymentStatus() {
    try {
        String status = paymentServiceClient.fetchStatus(); // slow external API
        return ResponseEntity.ok(status);
    } catch (ResourceAccessException | TimeoutException e) {
        // upstream did not respond in time
        return ResponseEntity.status(HttpStatus.GATEWAY_TIMEOUT)
                             .body("Payment service did not respond in time — please retry");
    }
}
```

---

## 🏗️ ResponseEntity — Builder Methods Cheatsheet

| Method | Status Code | Usage |
|--------|------------|-------|
| `ResponseEntity.ok(body)` | 200 | Return body with OK |
| `ResponseEntity.ok().build()` | 200 | No body |
| `ResponseEntity.created(uri)` | 201 | With Location URI |
| `ResponseEntity.accepted()` | 202 | Async accepted |
| `ResponseEntity.noContent()` | 204 | No body (DELETE) |
| `ResponseEntity.badRequest()` | 400 | Bad input |
| `ResponseEntity.notFound()` | 404 | Resource missing |
| `ResponseEntity.unprocessableEntity()` | 422 | Validation failed |
| `ResponseEntity.status(HttpStatus.XXX)` | Any | Custom status |
| `.body(object)` | — | Attach a response body |
| `.build()` | — | No body |
| `.header("Key", "Value")` | — | Add custom header |

---

## 🌍 Global Exception Handling with @ControllerAdvice

Instead of handling errors in every controller, use `@ControllerAdvice` to centralize:

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    // 404 - Resource Not Found
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                             .body(ex.getMessage());
    }

    // 400 - Bad Request / Validation Error
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors()
          .forEach(err -> errors.put(err.getField(), err.getDefaultMessage()));
        return ResponseEntity.badRequest().body(errors);
    }

    // 409 - Conflict
    @ExceptionHandler(DuplicateResourceException.class)
    public ResponseEntity<String> handleConflict(DuplicateResourceException ex) {
        return ResponseEntity.status(HttpStatus.CONFLICT)
                             .body(ex.getMessage());
    }

    // 500 - Generic fallback
    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGeneric(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                             .body("An unexpected error occurred");
    }
}
```

---

## 📐 Standard Error Response Body (Best Practice)

Instead of returning plain strings, return a structured error object:

```java
public class ErrorResponse {
    private int status;
    private String error;
    private String message;
    private LocalDateTime timestamp;

    // Constructors, Getters, Setters
}
```

```java
@ExceptionHandler(ResourceNotFoundException.class)
public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
    ErrorResponse error = new ErrorResponse(
        404,
        "Not Found",
        ex.getMessage(),
        LocalDateTime.now()
    );
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
}
```

**Response JSON:**
```json
{
  "status": 404,
  "error": "Not Found",
  "message": "User with ID 5 not found",
  "timestamp": "2026-04-02T10:30:00"
}
```

---

## 🔁 Quick Revision — Status Code Table

| Code | Category | When to use |
|------|----------|-------------|
| 200 | ✅ Success | GET / PUT / PATCH succeeded |
| 201 | ✅ Success | POST — resource created |
| 202 | ✅ Success | Async operation started |
| 204 | ✅ Success | DELETE — nothing to return |
| 301 | 🔵 Redirect | Resource permanently moved to new URL |
| 302 | 🔵 Redirect | Resource temporarily at different URL |
| 303 | 🔵 Redirect | Redirect to GET after POST/PUT/DELETE |
| 304 | 🔵 Redirect | Cached version still valid — skip resend |
| 400 | ❌ Client | Invalid request / bad input |
| 401 | ❌ Client | Not logged in |
| 402 | ❌ Client | Payment or subscription required |
| 403 | ❌ Client | Logged in but no permission |
| 404 | ❌ Client | Resource doesn't exist |
| 409 | ❌ Client | Duplicate / conflict |
| 422 | ❌ Client | Validation failed |
| 429 | ❌ Client | Too many requests |
| 500 | 🔥 Server | Unexpected server error |
| 502 | 🔥 Server | Upstream service crashed or returned invalid response |
| 503 | 🔥 Server | Server down / unavailable |
| 504 | 🔥 Server | Upstream service alive but too slow / timed out |

