# Section 17: Server-Side Service Discovery and Load Balancing in Kubernetes

---

## Table of Contents

| Lec # | Topic |
|-------|-------|
| 1 | [Introduction to Server-Side Service Discovery and Load Balancing](#lecture-1--introduction-to-server-side-service-discovery-and-load-balancing) |
| 2 | [How to Set Up Discovery Server in K8s Cluster Using Spring Cloud Kubernetes](#lecture-2--how-to-set-up-discovery-server-in-k8s-cluster-using-spring-cloud-kubernetes) |
| 3 | [Install Spring Cloud Kubernetes Discovery Server in K8s Cluster](#lecture-3--install-spring-cloud-kubernetes-discovery-server-in-k8s-cluster) |
| 4 | [Making Kubernetes Discovery Client Changes in Microservices (Part 1)](#lecture-4--making-kubernetes-discovery-client-changes-in-microservices-part-1) |
| 5 | [Making Kubernetes Discovery Client Changes in Microservices (Part 2)](#lecture-5--making-kubernetes-discovery-client-changes-in-microservices-part-2) |
| 6 | [Updating Helm Charts for Kubernetes Discovery Server Changes](#lecture-6--updating-helm-charts-for-kubernetes-discovery-server-changes) |
| 7 | [Demo of Server-Side Service Discovery and Load Balancing](#lecture-7--demo-of-server-side-service-discovery-and-load-balancing) |
| — | [Startup Order](#startup-order) |
| — | [Quick Reference: Key Properties](#quick-reference-key-properties) |
| — | [Quick Reference: Dependency](#quick-reference-dependency) |

---

## Lecture 1 — Introduction to Server-Side Service Discovery and Load Balancing

### Client-Side Discovery and Load Balancing (Recap)

In the Eureka-based approach (client-side), all microservices:
- Register themselves with the Eureka Server on startup
- Unregister on shutdown
- Send periodic **heartbeats** to prove healthy status

**Flow:**
1. `accounts` microservice asks Eureka Server for details of `loans` microservice
2. Eureka returns all running instance IP addresses (e.g., two instances)
3. `accounts` uses **Spring Cloud Load Balancer** to pick one instance
4. Request is forwarded to the selected instance

This is called **Client-Side Load Balancing** — the client itself decides which instance to call.

### Disadvantages of Client-Side Discovery

- Developers must manually maintain the Eureka Server (create a Spring Boot app, configure it as Eureka Server)
- Every microservice must be configured to register with Eureka
- Extra configuration burden across all services

### Server-Side Service Discovery and Load Balancing

This approach is only available when deploying to a **Kubernetes cluster**.

**How it works:**
1. A **Kubernetes Discovery Server** monitors all running pods/instances — it does NOT require apps to self-register
2. Instead, the Discovery Server uses **Kubernetes APIs** to fetch service/endpoint details automatically
3. When `accounts` wants to call `loans`, it simply sends a request to the Kubernetes service using its **DNS/service name**
4. The Kubernetes service (ClusterIP or LoadBalancer type) + Discovery Server handle load balancing transparently

**Advantages:**
- No need to maintain Eureka Server manually
- Microservices don't register themselves or send heartbeats
- Kubernetes handles all of this automatically

**Disadvantages:**
- No developer control over load balancing algorithm/strategy — Kubernetes decides

---

## Lecture 2 — How to Set Up Discovery Server in K8s Cluster Using Spring Cloud Kubernetes

### Spring Cloud Kubernetes Project

- Use the **Spring Cloud Kubernetes** project to set up server-side service discovery
- The Spring Cloud Kubernetes team provides a ready-made Docker image for the Discovery Server — no need to build your own like Eureka

### Kubernetes Manifest File: `kubernetes-discoveryserver.yaml`

Create the file at: `section_17/kubernetes/kubernetes-discoveryserver.yaml`

```yaml
apiVersion: v1
kind: List
items:

  # 1. Service
  - apiVersion: v1
    kind: Service
    metadata:
      labels:
        app: spring-cloud-kubernetes-discoveryserver
      name: spring-cloud-kubernetes-discoveryserver
    spec:
      ports:
        - port: 80
          targetPort: 8761
      selector:
        app: spring-cloud-kubernetes-discoveryserver
      type: ClusterIP

  # 2. ServiceAccount
  - apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: spring-cloud-kubernetes-discoveryserver

  # 3. ClusterRoleBinding
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: spring-cloud-kubernetes-discoveryserver
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: Role
      name: namespace-reader
    subjects:
      - kind: ServiceAccount
        name: spring-cloud-kubernetes-discoveryserver
        namespace: default

  # 4. Role (with pods access — required fix vs. official blog)
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: namespace-reader
    rules:
      - apiGroups: [""]
        resources: ["services", "endpoints", "pods"]   # ← pods added vs. official blog
        verbs: ["get", "list", "watch"]

  # 5. Deployment
  - apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: spring-cloud-kubernetes-discoveryserver
    spec:
      selector:
        matchLabels:
          app: spring-cloud-kubernetes-discoveryserver
      template:
        metadata:
          labels:
            app: spring-cloud-kubernetes-discoveryserver
        spec:
          serviceAccountName: spring-cloud-kubernetes-discoveryserver
          containers:
            - name: spring-cloud-kubernetes-discoveryserver
              image: springcloud/spring-cloud-kubernetes-discoveryserver:3.0.4  # ← use latest stable tag
              imagePullPolicy: IfNotPresent
              readinessProbe:
                httpGet:
                  path: /actuator/health/readiness
                  port: 8761
                initialDelaySeconds: 100   # ← important: give enough time to start
                periodSeconds: 30
              livenessProbe:
                httpGet:
                  path: /actuator/health/liveness
                  port: 8761
                initialDelaySeconds: 100
                periodSeconds: 30
              ports:
                - containerPort: 8761
```

### Key Notes

- The official blog only included `services` and `endpoints` in the Role resources — you must also add **`pods`**, otherwise the Discovery Server won't work
- Use Docker Hub tag `3.0.4` (or latest stable) instead of the outdated `2.1.0-M3` from the blog
- `initialDelaySeconds: 100` is needed to give the Discovery Server time to start before Kubernetes checks health probes (default is ~10–15s which causes crash-loop)
- `periodSeconds: 30` means Kubernetes re-checks the probe every 30 seconds after initial delay

---

## Lecture 3 — Install Spring Cloud Kubernetes Discovery Server in K8s Cluster

### Why Not Use Helm Chart?

1. Discovery Server is a **one-time setup** — no frequent changes needed
2. No Helm chart exists in the community (including Bitnami) for this Discovery Server

### Install the Discovery Server

Run the following command:

```bash
kubectl apply -f kubernetes-discoveryserver.yaml
```

This creates:
- Service
- ServiceAccount
- RoleBinding
- Role
- Deployment

Verify via Kubernetes Dashboard → Pods → Check logs to confirm Spring Boot started successfully.

### Prepare Microservices Code

Copy microservices from `section_14` (latest code) into a new folder `section_17`:

```
section_17/
├── accounts/
├── cards/
├── config-server/
├── gateway-server/
├── loans/
├── message/
├── kubernetes/
│   └── kubernetes-discoveryserver.yaml
└── helm/          ← copied from section_16
```

> **Note:** Do NOT copy the Eureka Server or Docker Compose files — they are no longer needed.

Open `section_17` in IntelliJ IDEA, load as Maven projects, enable annotation processing, and build.

---

## Lecture 4 — Making Kubernetes Discovery Client Changes in Microservices (Part 1)

### Changes Required in Each Microservice

Remove Eureka → Add Spring Cloud Kubernetes Discovery Client

---

### Accounts Microservice

**`pom.xml`** — Replace Eureka dependency:

```xml
<!-- REMOVE -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

<!-- ADD -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-kubernetes-discoveryclient</artifactId>
</dependency>
```

Also update image tag from `s14` → `s17`.

**`AccountsApplication.java`** — Add annotation:

```java
@SpringBootApplication
@EnableDiscoveryClient   // ← Add this
public class AccountsApplication {
    public static void main(String[] args) {
        SpringApplication.run(AccountsApplication.class, args);
    }
}
```

**`application.yml`** — Remove all Eureka config, add Kubernetes discovery config:

```yaml
spring:
  cloud:
    kubernetes:
      discovery:
        all-namespaces: true   # ← discover services across all namespaces
```

**`CardsFeignClient.java`** — Add explicit URL (no Eureka to resolve service names):

```java
@FeignClient(name = "cards", url = "http://cards:9000")
public interface CardsFeignClient {
    // ...
}
```

**`LoansFeignClient.java`**:

```java
@FeignClient(name = "loans", url = "http://loans:8090")
public interface LoansFeignClient {
    // ...
}
```

> **Note:** The `url` uses the Kubernetes service name as the hostname. This only works inside the cluster. For local testing without Kubernetes, replace with `http://localhost:<port>`.

---

### Cards Microservice

**`pom.xml`** — Same dependency swap as Accounts. Update tag to `s17`.

**`CardsApplication.java`**:

```java
@SpringBootApplication
@EnableDiscoveryClient
public class CardsApplication { ... }
```

**`application.yml`**:

```yaml
spring:
  cloud:
    kubernetes:
      discovery:
        all-namespaces: true
```

No Feign client changes needed — Cards does not call other microservices.

---

### Loans Microservice

Same changes as Cards:

**`pom.xml`** — Swap dependency, update tag to `s17`.

**`LoansApplication.java`**:

```java
@SpringBootApplication
@EnableDiscoveryClient
public class LoansApplication { ... }
```

**`application.yml`**:

```yaml
spring:
  cloud:
    kubernetes:
      discovery:
        all-namespaces: true
```

---

### Config Server

No Eureka integration → only update tag from `s14` → `s17` in `pom.xml`.

### Message Microservice

No Eureka integration → only update tag from `s14` → `s17` in `pom.xml`.

---

## Lecture 5 — Making Kubernetes Discovery Client Changes in Microservices (Part 2)

### Gateway Server

**`pom.xml`** — Swap Eureka for Kubernetes Discovery Client. Update tag to `s17`.

**`GatewayserverApplication.java`**:

```java
@SpringBootApplication
@EnableDiscoveryClient
public class GatewayserverApplication {

    @Bean
    public RouteLocator easyBankRouteConfig(RouteLocatorBuilder routeLocatorBuilder) {
        return routeLocatorBuilder.routes()
            .route(p -> p.path("/easybank/accounts/**")
                .filters(f -> f.rewritePath("/easybank/accounts/(?<segment>.*)", "/${segment}")
                    .addResponseHeader("X-Response-Time", LocalDateTime.now().toString()))
                .uri("http://accounts:8080"))    // ← Replace lb://ACCOUNTS with direct URL

            .route(p -> p.path("/easybank/loans/**")
                .filters(f -> f.rewritePath("/easybank/loans/(?<segment>.*)", "/${segment}")
                    .addResponseHeader("X-Response-Time", LocalDateTime.now().toString()))
                .uri("http://loans:8090"))       // ← Replace lb://LOANS

            .route(p -> p.path("/easybank/cards/**")
                .filters(f -> f.rewritePath("/easybank/cards/(?<segment>.*)", "/${segment}")
                    .addResponseHeader("X-Response-Time", LocalDateTime.now().toString()))
                .uri("http://cards:9000"))       // ← Replace lb://CARDS
            .build();
    }
}
```

> **Why remove `lb://`?** The `lb://` prefix tells Spring Cloud Gateway to use client-side load balancing via Spring Cloud Load Balancer. Since we're switching to server-side load balancing (Kubernetes handles it), we use direct service DNS URLs instead.

**`application.yml`** — Remove Eureka discovery locator, add Kubernetes discovery config:

```yaml
spring:
  cloud:
    gateway:
      # REMOVE these Eureka-related properties:
      # discovery:
      #   locator:
      #     enabled: true
      #     lower-case-service-id: true

    kubernetes:
      discovery:
        enabled: true           # ← tell Gateway to use K8s discovery
        all-namespaces: true

    discovery:
      client:
        health-indicator:
          enabled: false        # ← workaround for known GitHub issue with this library
```

> **Why `health-indicator.enabled: false`?** There is a known bug in the Spring Cloud Kubernetes discovery client where the default health indicator bean fails to initialize. This is a temporary workaround confirmed by the Spring Cloud Kubernetes team.

### Generate Docker Images

After all changes, build and push Docker images with tag `s17` for all microservices:

```bash
# Example for accounts
mvn spring-boot:build-image -Dspring-boot.build-image.imageName=<dockerhub-user>/accounts:s17

# Repeat for: cards, loans, gateway-server, config-server, message
```

---

## Lecture 6 — Updating Helm Charts for Kubernetes Discovery Server Changes

### Setup (Behind the Scenes)
- Docker images with tag `s17` are built and pushed
- Kubernetes cluster components installed: Keycloak, Kafka, Grafana, Prometheus

### Helm Chart Updates

Navigate to `section_17/helm/easybank-services/`

**Step 1 — Delete Eureka Server Helm chart folder** from `easybank-services/`

**Step 2 — Update `values.yaml` for each microservice** (change tag + replica count):

```yaml
# accounts/values.yaml
image:
  tag: s17            # ← update from s14

replicaCount: 2       # ← set to 2 for load balancing demo
```

For all other microservices (cards, config-server, gateway-server, loans, message):

```yaml
image:
  tag: s17
```

**Step 3 — Update `chart.yaml` in each environment** (`dev-env`, `prod-env`, `qa-env`):

Remove the Eureka Server dependency:

```yaml
dependencies:
  # DELETE this block:
  # - name: eurekaserver
  #   repository: ...
  #   version: ...

  - name: accounts
    ...
  - name: cards
    ...
  # etc.
```

**Step 4 — Delete `chart.lock` files** (stale lock files cause issues after removing dependencies):

```bash
rm helm/environments/dev-env/chart.lock
rm helm/environments/prod-env/chart.lock
rm helm/environments/qa-env/chart.lock
```

**Step 5 — Fix missing Discovery Server URL property**

The microservices need to know where the Discovery Server is running. Add this to the common Helm config map template.

**`easybank-common/templates/configmap.yaml`** — Replace Eureka URL with Discovery Server URL:

```yaml
# REPLACE:
# EUREKA.CLIENT.SERVICEURL.DEFAULTZONE: {{ .Values.global.eurekaServerURL }}

# WITH:
SPRING.CLOUD.KUBERNETES.DISCOVERY.DISCOVERY-SERVER-URL: {{ .Values.global.discoveryServerURL }}
```

**`easybank-common/templates/deployment.yaml`** — Update environment variable injection condition:

```yaml
# REPLACE:
# {{- if .Values.global.eurekaEnabled }}

# WITH:
{{- if .Values.global.discoveryEnabled }}
  - name: SPRING_CLOUD_KUBERNETES_DISCOVERY_DISCOVERYSERVERURL
    valueFrom:
      configMapKeyRef:
        name: {{ .Values.global.configMapName }}
        key: SPRING.CLOUD.KUBERNETES.DISCOVERY.DISCOVERY-SERVER-URL
{{- end }}
```

**`easybank-services/<each-service>/values.yaml`** — Rename variable:

```yaml
# REPLACE in all service values.yaml:
# eurekaEnabled: true

# WITH:
discoveryEnabled: true
```

**`environments/prod-env/values.yaml`** (and dev-env, qa-env):

```yaml
global:
  # REPLACE:
  # eurekaServerURL: "http://eurekaserver/eureka/"

  # WITH:
  discoveryServerURL: "http://spring-cloud-kubernetes-discoveryserver:80"
```

> The service name `spring-cloud-kubernetes-discoveryserver` comes from the `metadata.name` defined in the Kubernetes manifest file created in Lecture 2.

**Step 6 — Rebuild Helm dependencies:**

```bash
cd helm/easybank-services/accounts && helm dependencies build
cd helm/easybank-services/cards && helm dependencies build
# ... repeat for all services

cd helm/environments/dev-env && helm dependencies build
cd helm/environments/prod-env && helm dependencies build
cd helm/environments/qa-env && helm dependencies build
```

**Step 7 — Validate with helm template:**

```bash
cd helm/environments/prod-env
helm template .
```

If output is generated without errors, the charts are compiled correctly.

**Step 8 — Deploy all microservices:**

```bash
cd helm/environments
helm install easybank prod-env/
```

---

## Lecture 7 — Demo of Server-Side Service Discovery and Load Balancing

### Setup

- All microservices started successfully in Kubernetes cluster
- Accounts microservice deployed with **2 replicas** (each has its own in-memory H2 database)
- Keycloak client `easybank-callcenter-cc` created with required roles

### Testing Steps

**Step 1 — Get access token from Keycloak (via Postman)**

Use the OAuth2 flow to obtain a bearer token for authorization.

**Step 2 — Create an account**

```
POST http://localhost:8072/easybank/accounts/api/create
Authorization: Bearer <token>
Body: { "mobileNumber": "...", ... }
```

Response: `Account created successfully`

**Step 3 — Fetch account (tests load balancing)**

```
GET http://localhost:8072/easybank/accounts/api/fetch?mobileNumber=<number>
```

Keep sending the request multiple times. At some point, the request will be routed to the **other pod** (which has a different H2 database and won't find the account), returning a `404 Not Found` — confirming load balancing is working.

### Sticky Sessions Behavior

Kubernetes uses **sticky sessions** — requests from the same client IP are consistently routed to the same pod as long as requests come frequently. To observe load balancing between pods:
- Wait 1–2 minutes between requests, OR
- Use a different client (e.g., browser incognito mode vs Postman)

### Summary: Server-Side Discovery Advantages

| Feature | Eureka (Client-Side) | Kubernetes Discovery Server (Server-Side) |
|---|---|---|
| Service registration | Manual (each app registers) | Automatic (K8s API) |
| Heartbeat | Required | Not required |
| Load balancing control | Full (Spring Cloud LB strategies) | None (K8s decides) |
| Maintenance overhead | High | Low |
| Works without K8s | Yes | No |

---

## Startup Order

The components must be started in the following order to avoid dependency failures:

| Step | Component | How Deployed | Reason |
|------|-----------|--------------|--------|
| 1 | **Keycloak** | Helm / manual | Identity provider — must be ready before Gateway can validate tokens |
| 2 | **Kafka** | Helm / manual | Message broker — must be ready before `message` microservice starts |
| 3 | **Prometheus / Grafana** | Helm / manual | Monitoring stack — independent, but best started early |
| 4 | **Spring Cloud Kubernetes Discovery Server** | `kubectl apply -f kubernetes-discoveryserver.yaml` | All microservices query this server for service URLs; must exist before they start |
| 5 | **Config Server** | Helm (`easybank-services`) | Provides external configuration to every microservice on startup |
| 6 | **Accounts, Cards, Loans, Message** | Helm (`easybank-services`) | Core microservices — depend on Config Server and Discovery Server being healthy |
| 7 | **Gateway Server** | Helm (`easybank-services`) | Entry point — depends on all core microservices being discoverable and healthy |

### Notes

- **Steps 1–3** are typically pre-installed in the cluster and do not change between deployments.
- **Step 4** is a one-time manual step (no Helm chart available for the Discovery Server).
- **Steps 5–7** are deployed together via a single Helm install command:
  ```bash
  helm install easybank prod-env/
  ```
  Kubernetes **readiness probes** enforce the effective ordering — the Config Server pod must pass its readiness check before dependent microservices are considered ready, and those must be ready before the Gateway routes traffic to them.
- You do **not** need to manually sequence Steps 5–7 — the readiness probes defined in each Helm chart handle this automatically.

---

## Quick Reference: Key Properties

| Property | Value | Purpose |
|---|---|---|
| `spring.cloud.kubernetes.discovery.all-namespaces` | `true` | Discover services across all K8s namespaces |
| `spring.cloud.kubernetes.discovery.enabled` | `true` | Enable K8s discovery (required for Gateway Server) |
| `spring.cloud.kubernetes.discovery.discovery-server-url` | `http://spring-cloud-kubernetes-discoveryserver:80` | URL of the Discovery Server pod |
| `spring.cloud.discovery.client.health-indicator.enabled` | `false` | Workaround for known bean initialization bug |

## Quick Reference: Dependency

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-kubernetes-discoveryclient</artifactId>
</dependency>
```
