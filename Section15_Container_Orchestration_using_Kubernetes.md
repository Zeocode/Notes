# Section 15 — Container Orchestration using Kubernetes

---

## Table of Contents

| Lec # | Topic |
|-------|-------|
| 1 | [Introduction to the Challenges Related to Container Orchestration](#lec-1--introduction-to-the-challenges-related-to-container-orchestration) |
| 2 | [Introduction to Kubernetes](#lec-2--introduction-to-kubernetes) |
| 3 | [Deep Dive on Kubernetes Internal Architecture](#lec-3--deep-dive-on-kubernetes-internal-architecture) |
| 4 | [Setup a Local Kubernetes Cluster using Docker Desktop](#lec-4--setup-a-local-kubernetes-cluster-using-docker-desktop) |
| 5 | [Deploying the Kubernetes Dashboard UI](#lec-5--deploying-the-kubernetes-dashboard-ui) |
| 6 | [Deep Dive on Kubernetes YAML Configurations to Deploy a Microservice](#lec-6--deep-dive-on-kubernetes-yaml-configurations-to-deploy-a-microservice) |
| 7 | [Deploying ConfigServer into Kubernetes Cluster](#lec-7--deploying-configserver-into-kubernetes-cluster) |
| 8 | [Create Environment Variables Using ConfigMap](#lec-8--create-environment-variables-using-configmap) |
| 9 | [Preparing Kubernetes Manifest Files for Remaining Microservices](#lec-9--preparing-kubernetes-manifest-files-for-remaining-microservices) |
| 10 | [Deploying Remaining Microservices into Kubernetes Cluster](#lec-10--deploying-remaining-microservices-into-kubernetes-cluster) |
| 11 | [Automatic Self-Healing Inside Kubernetes Cluster](#lec-11--automatic-self-healing-inside-kubernetes-cluster) |
| 12 | [Automatic Rollout & Rollback Inside Kubernetes Cluster](#lec-12--automatic-rollout--rollback-inside-kubernetes-cluster) |
| 13 | [Introduction to Kubernetes Service Types](#lec-13--introduction-to-kubernetes-service-types) |
| 14 | [Demo of Kubernetes Service Types](#lec-14--demo-of-kubernetes-service-types) |
| 15 | [Problems with Manually Created Kubernetes Manifest Files](#lec-15--problems-with-manually-created-kubernetes-manifest-files) |
| — | [Quick Reference: Essential kubectl Commands](#quick-reference-essential-kubectl-commands) |
| — | [Summary: Section 15 Concepts Map](#summary-section-15-concepts-map) |

---

## Lec 1 — Introduction to the Challenges Related to Container Orchestration

### What is Orchestration?
- **Analogy**: In a music orchestra, a **conductor** stands in the center and tells musicians when to play, stop, or pass. Similarly, in microservices, we need a "conductor" that manages all containers.
- **Container Orchestration** = managing, controlling, and automating the operation of many Docker containers across multiple servers.

### Why Do We Need Container Orchestration?
Real production apps can have **100+ microservices** → 100+ running containers. Managing them manually is impossible.

### Three Key Challenges Solved by Container Orchestration:

| Challenge | Description |
|-----------|-------------|
| **Automated Deployments / Rollouts / Rollbacks** | Deploy new versions without downtime. Roll back immediately if a bug is found in production. |
| **Self-Healing** | Automatically detect unhealthy containers, kill them, and replace with new ones. |
| **Auto Scaling** | Automatically increase/decrease container instances based on CPU/traffic (e.g., Netflix on Friday nights). |

### The Solution → **Kubernetes**
> Kubernetes is an open-source container orchestration platform that automates deployments, rollouts, scaling, and managing all containerized applications.

- Originally developed by **Google** (used internally for 15+ years for YouTube, Gmail, Google Photos).
- Open-sourced around **2015**.
- Now maintained by the **Cloud Native Computing Foundation (CNCF)**.

---

## Lec 2 — Introduction to Kubernetes

### Key Facts About Kubernetes
- Open-source system for automating **deployment, scaling, and management** of containerized applications.
- Most widely used container orchestration platform in the market.
- **Cloud-neutral** → Kubernetes concepts are the same whether running locally, on AWS, GCP, or Azure.

### What Kubernetes Provides
- ✅ Automated scaling and fail-over handling
- ✅ Zero-downtime deployment patterns
- ✅ Service discovery & **server-side load balancing** (can replace Eureka Server)
- ✅ Container and storage orchestration
- ✅ Automated rollouts & rollbacks
- ✅ Self-healing
- ✅ Configuration and secret management

### Name Origins
- **Kubernetes** = Greek word for *helmsman* or *pilot* who controls a ship.
- The logo is a ship's wheel (helm).
- Short form: **K8s** — (K + 8 letters between K and s + s)

---

## Lec 3 — Deep Dive on Kubernetes Internal Architecture

### Cluster Overview
A **Kubernetes Cluster** is a set of servers/VMs working together. It has two types of nodes:

| Node Type | Purpose |
|-----------|---------|
| **Master Node** (Control Plane) | Controls and maintains the entire cluster |
| **Worker Nodes** | Handle the actual traffic; containers are deployed here |

---

### Master Node (Control Plane) Components

```
┌──────────────────────────────────────────┐
│             MASTER NODE                  │
│  ┌────────────┐   ┌─────────────────┐    │
│  │ Kube API   │   │   Scheduler     │    │
│  │  Server    │   │                 │    │
│  └────────────┘   └─────────────────┘    │
│  ┌────────────────┐  ┌─────────────┐     │
│  │Controller Mgr  │  │   etcd      │     │
│  │                │  │  (Brain/DB) │     │
│  └────────────────┘  └─────────────┘     │
└──────────────────────────────────────────┘
```

| Component | Description |
|-----------|-------------|
| **Kube API Server** | Entry point for all communication — exposes REST APIs. Receives instructions from `kubectl` CLI or Admin UI. |
| **Scheduler** | Decides **which worker node** to deploy a container on based on available CPU/memory bandwidth. |
| **Controller Manager** | Continuously monitors the cluster and ensures the **desired state** matches the **actual state** (e.g., always keep 3 replicas of accounts service running). |
| **etcd** | Distributed key-value store — the **brain/database** of the cluster. Stores all cluster state and configuration data. |

---

### Worker Node Components

```
┌──────────────────────────────────────────────────┐
│                 WORKER NODE                       │
│  ┌─────────┐  ┌───────────────┐  ┌────────────┐  │
│  │ Kubelet │  │  Container    │  │  Kube      │  │
│  │ (Agent) │  │  Runtime      │  │  Proxy     │  │
│  └─────────┘  │  (Docker)     │  └────────────┘  │
│               └───────────────┘                   │
│  ┌────────────────────────────────────────────┐   │
│  │   POD                                      │   │
│  │  ┌───────────────┐  ┌──────────────────┐   │   │
│  │  │ Container 1   │  │ Container 2      │   │   │
│  │  │ (Microservice)│  │ (Helper/Sidecar) │   │   │
│  │  └───────────────┘  └──────────────────┘   │   │
│  └────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────┘
```

| Component | Description |
|-----------|-------------|
| **Kubelet** | Agent that runs on every worker node. Receives deployment instructions from the master node via Kube API Server. |
| **Container Runtime** | Usually **Docker** — actually runs the containers inside pods. |
| **Pod** | The **smallest deployable unit** in Kubernetes. Containers run inside pods (not directly in worker nodes). Usually 1 container per pod. |
| **Kube Proxy** | Exposes containers to the outside world or restricts them to internal cluster communication. |

### Pod Rules
- 1 pod → 1 microservice (usually 1 container).
- A pod can have an extra **sidecar/helper container** alongside the main microservice container.
- Different microservices (accounts, loans, cards) are **never** deployed in the same pod.

### Interaction Methods
Two ways to send instructions to the Kubernetes cluster:
1. **Kubernetes Admin UI** (Dashboard)
2. **`kubectl` CLI** — command-line tool

---

## Lec 4 — Setup a Local Kubernetes Cluster using Docker Desktop

### Why Docker Desktop (not Minikube)?
- **Minikube** has some commands that differ from production Kubernetes → not ideal for learning.
- **Docker Desktop** creates a local cluster that behaves identically to production.

### Steps to Enable Kubernetes in Docker Desktop

1. Open **Docker Desktop**
2. Click **Settings** (top right)
3. Click **Kubernetes** (left sidebar)
4. Check **Enable Kubernetes**
5. **Do NOT** check "Show system containers" (to avoid Kubernetes internal containers appearing in `docker ps`)
6. Click **Apply & Restart**
7. Wait 2–3 minutes for the cluster to start.

> ✅ When done, you'll see two green icons at the bottom of Docker Desktop:
> - Docker engine is running
> - Kubernetes is running

### Verify `kubectl` Installation
Docker Desktop automatically installs `kubectl`. Verify it:

```bash
# Check available contexts (isolated environments)
kubectl config get-contexts

# Check available clusters
kubectl config get-clusters

# Set docker-desktop as the active context (if you have multiple contexts)
kubectl config use-context docker-desktop

# Verify nodes in your cluster
kubectl get nodes
```

> ℹ️ Local cluster = **single node** (same node acts as both master and worker). Production clusters have multiple nodes.

---

## Lec 5 — Deploying the Kubernetes Dashboard UI

### What is Kubernetes Dashboard?
An admin UI that lets you visually manage and monitor your Kubernetes cluster.

### Step 1 — Install Helm
Helm = **package manager for Kubernetes** (like npm for Node.js or Homebrew for Mac).

**Mac:**
```bash
brew install helm
```

**Windows (via Chocolatey):**
```powershell
# First install Chocolatey, then:
choco install kubernetes-helm
```

**Verify installation:**
```bash
helm version
```

### Step 2 — Install Kubernetes Dashboard via Helm

```bash
# Add the kubernetes-dashboard Helm repo
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/

# Install the dashboard
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard \
  --create-namespace --namespace kubernetes-dashboard
```

### Step 3 — Access the Dashboard

Run the port-forward command (keep this terminal open!):
```bash
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443
```

Access at: **https://localhost:8443**
> ⚠️ Browser will warn about certificate — click "Advanced" → "Visit this website" (safe for local testing).

### Step 4 — Create Admin User & Get Token

**Create service account** (`dashboard-adminuser.yaml`):
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

**Create cluster role binding** (`dashboard-rolebinding.yaml`):
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: kubernetes-dashboard
```

**Create long-lived token secret** (`secret.yaml`):
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: "admin-user"
type: kubernetes.io/service-account-token
```

**Apply all files:**
```bash
kubectl apply -f dashboard-adminuser.yaml
kubectl apply -f dashboard-rolebinding.yaml
kubectl apply -f secret.yaml
```

**Get the long-lived token:**
```bash
kubectl get secret admin-user -n kubernetes-dashboard \
  -o jsonpath={".data.token"} | base64 -d
```
> ⚠️ Don't copy the trailing `%` character at the end.

**Or get a short-lived token:**
```bash
kubectl -n kubernetes-dashboard create token admin-user
```

Paste the token on the dashboard login page and click **Sign In**.

---

## Lec 6 — Deep Dive on Kubernetes YAML Configurations to Deploy a Microservice

### Kubernetes Manifest Files
Kubernetes uses **YAML manifest files** to define deployments and services (different format from Docker Compose).

### Full Example — `configserver.yaml`

```yaml
# ===== DEPLOYMENT =====
apiVersion: apps/v1
kind: Deployment
metadata:
  name: configserver-deployment
  labels:
    app: configserver
spec:
  replicas: 1
  selector:
    matchLabels:
      app: configserver
  template:
    metadata:
      labels:
        app: configserver
    spec:
      containers:
        - name: configserver
          image: eazybytes/configserver:s14
          ports:
            - containerPort: 8071
---
# ===== SERVICE =====
apiVersion: v1
kind: Service
metadata:
  name: configserver         # ← Used as HOSTNAME for inter-service communication
spec:
  selector:
    app: configserver        # ← Links service to the deployment with same label
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 8071             # ← External port exposed to outside world
      targetPort: 8071       # ← Port inside the container (must match containerPort)
```

### Key Concepts

| Concept | Details |
|---------|---------|
| `---` separator | Separates multiple YAML configs in one file (treated as separate files by Kubernetes) |
| `kind: Deployment` | Tells Kubernetes to create a deployment object |
| `kind: Service` | Tells Kubernetes how to expose the container |
| `replicas: 1` | Number of pod instances to maintain |
| `selector.matchLabels` | Links the spec to the correct deployment via matching labels |
| `template` | Blueprint for creating pods |
| `containerPort` | Port the app listens on inside the container |
| `port` | Port exposed to the service (external or internal) |
| `targetPort` | Internal container port the service forwards traffic to |
| `type: LoadBalancer` | Exposes the service to outside the cluster |
| `metadata.name` (service) | Acts as the **hostname** for other microservices inside the cluster |

### How Labels Link Deployment ↔ Service
```
Deployment metadata.labels.app = "configserver"
          ↕ (must match)
Service   spec.selector.app    = "configserver"
```

---

## Lec 7 — Deploying ConfigServer into Kubernetes Cluster

### Pre-Deployment Checks
```bash
# Check deployments (should be empty initially)
kubectl get deployments

# Check services (only default 'kubernetes' service exists)
kubectl get services

# Check replica sets
kubectl get replicaset

# Check pods
kubectl get pods
```

### Deploy ConfigServer
```bash
# Navigate to your kubernetes folder
cd path/to/section_15/kubernetes

# Apply the manifest file
kubectl apply -f configserver.yaml
```

### Post-Deployment Verification
```bash
# Check deployments — should show: configserver-deployment (1/1 ready)
kubectl get deployments

# Check services — should show: configserver (LoadBalancer, localhost:8071)
kubectl get services

# Check replica sets — desired=1, current=1, ready=1
kubectl get replicaset

# Check pods — STATUS: Running
kubectl get pods
```

### Test the Deployment
```
GET http://localhost:8071/accounts/prod
GET http://localhost:8071/loans/default
```

### What the Output Means
| Column | Meaning |
|--------|---------|
| `DESIRED` | Expected number of replicas |
| `CURRENT` | Actual running replicas |
| `READY` | Replicas ready to serve traffic |
| `EXTERNAL-IP` | `localhost` on local cluster; real public IP on cloud |

---

## Lec 8 — Create Environment Variables Using ConfigMap

### What is a ConfigMap?
> A Kubernetes object used to store **non-confidential** data in key-value pairs. Containers can consume them as environment variables.

**vs. Secrets:**
- `ConfigMap` = plain text (visible in dashboard)
- `Secret` = base64-encoded (hidden in dashboard)

> ⚠️ Kubernetes secrets are not truly secure. Use cloud-specific secret managers in production.

### Create `configmaps.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: easybank-configmap
data:
  # Profiles & Config Server
  SPRING_PROFILES_ACTIVE: "prod"
  SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071/"

  # Eureka
  EUREKA_CLIENT_SERVICEURL_DEFAULTZONE: "http://eurekaserver:8070/eureka/"

  # Application Names (separate keys per service)
  configserver_application_name: "configserver"
  eurekaserver_application_name: "eurekaserver"
  accounts_application_name: "accounts"
  loans_application_name: "loans"
  cards_application_name: "cards"
  gateway_application_name: "gatewayserver"

  # Keycloak
  KEYCLOAK_ADMIN: "admin"
  KEYCLOAK_ADMIN_PASSWORD: "admin"
  spring_security_oauth2_resourceserver_jwt_jwk-set-uri: "http://keycloak:7080/realms/master/protocol/openid-connect/certs"
```

### Apply the ConfigMap
```bash
kubectl apply -f configmaps.yaml
# Output: configmap/easybank-configmap created
```

### Verify in Dashboard
- Dashboard → Namespace: `default` → **Config Maps** → `easybank-configmap`

---

## Lec 9 — Preparing Kubernetes Manifest Files for Remaining Microservices

### File Order for Deployment
Name files with number prefixes to enforce deployment order:

```
1_keycloak.yaml
2_configmaps.yaml
3_configserver.yaml
4_eurekaserver.yaml
5_accounts.yaml
6_loans.yaml
7_cards.yaml
8_gateway.yaml
```

### Injecting ConfigMap Values into Containers (using `env` + `valueFrom`)

```yaml
spec:
  containers:
    - name: keycloak
      image: quay.io/keycloak/keycloak:latest
      args: ["start-dev"]
      env:
        - name: KEYCLOAK_ADMIN
          valueFrom:
            configMapKeyRef:
              name: easybank-configmap
              key: KEYCLOAK_ADMIN
        - name: KEYCLOAK_ADMIN_PASSWORD
          valueFrom:
            configMapKeyRef:
              name: easybank-configmap
              key: KEYCLOAK_ADMIN_PASSWORD
      ports:
        - containerPort: 8080
```

### `keycloak.yaml` Service Section (exposed at 7080 → 8080 container)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: keycloak
spec:
  selector:
    app: keycloak
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 7080
      targetPort: 8080
```

### `eurekaserver.yaml` — Injected ENV Variables
```yaml
env:
  - name: SPRING_APPLICATION_NAME
    valueFrom:
      configMapKeyRef:
        name: easybank-configmap
        key: eurekaserver_application_name
  - name: SPRING_CONFIG_IMPORT
    valueFrom:
      configMapKeyRef:
        name: easybank-configmap
        key: SPRING_CONFIG_IMPORT
```

### `accounts.yaml` — Extra ENV Variable
```yaml
env:
  # ... (same as eurekaserver + spring.config.import)
  - name: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
    valueFrom:
      configMapKeyRef:
        name: easybank-configmap
        key: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
```

### `gateway.yaml` — Extra ENV Variable (Keycloak JWT)
```yaml
env:
  # ... (all standard + eureka)
  - name: SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_JWK_SET_URI
    valueFrom:
      configMapKeyRef:
        name: easybank-configmap
        key: spring_security_oauth2_resourceserver_jwt_jwk-set-uri
```

> 💡 **Note on image tags**: This section uses `s12` tags (not `s13`/`s14`) to avoid requiring Kafka/RabbitMQ setup. Section 13/14 images need event-driven infrastructure.

---

## Lec 10 — Deploying Remaining Microservices into Kubernetes Cluster

### Deploy All Services in Order

```bash
# 1. Keycloak (must be first)
kubectl apply -f 1_keycloak.yaml

# 2. ConfigMap (Kubernetes detects no changes → "unchanged")
kubectl apply -f 2_configmaps.yaml

# 3. Config Server
kubectl apply -f 3_configserver.yaml

# 4. Wait for Config Server to start, then deploy Eureka
kubectl apply -f 4_eurekaserver.yaml

# 5. Wait for Eureka, then deploy business services
kubectl apply -f 5_accounts.yaml
kubectl apply -f 6_loans.yaml
kubectl apply -f 7_cards.yaml

# 6. Wait for accounts/loans/cards to register with Eureka, then deploy Gateway
kubectl apply -f 8_gateway.yaml
```

### Verify All Deployments
```bash
kubectl get pods
kubectl get deployments
kubectl get services
```

### Kubernetes "Unchanged" Feature
> 💡 If you re-apply a manifest with no changes, Kubernetes outputs `unchanged` and does nothing — idempotent behavior.

### Inter-Service Dependency
- Kubernetes **does not** have a built-in `depends_on` like Docker Compose.
- If you start services before their dependencies, containers will keep **restarting** until dependencies are available (eventually succeeds).
- Best practice: deploy in the order above and wait for each to be fully running.

### Keycloak Setup for Testing

1. Access Keycloak Admin Console: `http://localhost:7080`
2. Login: `admin` / `admin`
3. Go to **Clients** → **Create Client**
4. Client ID: `easybank-callcenter-cc`
5. Enable **Client Authentication**, disable Standard Flow & Direct Access Grants
6. Enable **Service Account Roles** → Next → Save
7. Copy the **Client Secret** from the **Credentials** tab
8. Create **Roles**: `accounts`, `cards`, `loans`
9. Go to **Service Account Roles** → **Assign Role** → assign all three roles

---

## Lec 11 — Automatic Self-Healing Inside Kubernetes Cluster

### What is Self-Healing?
> When a container crashes or becomes unhealthy, Kubernetes automatically kills it and creates a new one to match the desired state.

### Demo: Scale Accounts Service to 2 Replicas

**Option 1 — Update YAML file and re-apply:**
```yaml
# In accounts.yaml
spec:
  replicas: 2
```
```bash
kubectl apply -f 5_accounts.yaml
```

**Option 2 — Use kubectl scale command:**
```bash
kubectl scale deployment accounts-deployment --replicas=2
```

### Verify Scaling
```bash
kubectl get replicaset
# accounts-deployment: desired=2, current=2, ready=2

kubectl get pods
# Two accounts pods running
```

### Demo: Simulate Self-Healing by Manually Deleting a Pod

```bash
# 1. Get pod names
kubectl get pods

# 2. Delete one accounts pod
kubectl delete pod <pod-name>
# Example: kubectl delete pod accounts-deployment-abc123

# 3. Immediately check again — Kubernetes creates a new pod!
kubectl get pods
# New pod appears within seconds

# 4. Verify replica set still shows desired=2
kubectl get replicaset
```

### View Events Behind the Scenes
```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```
> You'll see: `Killing` → immediately followed by → `Created pod` (self-healing in action)

### Key Insight
```
Desired State (in YAML/etcd): replicas=2
      ↕ Controller Manager monitors
Actual State (running pods):   replicas=2 ✅
         ↕ if pod dies
Actual State:                  replicas=1 ❌
         ↓ Controller Manager detects mismatch
         ↓ Creates new pod
Actual State:                  replicas=2 ✅
```

---

## Lec 12 — Automatic Rollout & Rollback Inside Kubernetes Cluster

### Rolling Deployment (Zero Downtime)
When you push new changes, Kubernetes:
1. Creates a new pod with the new image.
2. Waits for it to start successfully.
3. **Only then** terminates the old pod.
4. Repeats for each replica incrementally.

This means **no downtime** even with 100 instances!

### Update Image via `kubectl set image`

```bash
# Syntax:
kubectl set image deployment/<deployment-name> <container-name>=<image>:<tag> --record

# Example: update gateway server from s12 to s11
kubectl set image deployment/gatewayserver-deployment \
  gatewayserver=eazybytes/gatewayserver:s11 --record
```

> ⚠️ `--record` flag is deprecated and may be removed in newer versions. Remove it if it causes errors.

### What Happens with an Invalid Image?

```bash
# Intentionally invalid image tag
kubectl set image deployment/gatewayserver-deployment \
  gatewayserver=eazybytes/gatewayserver:s111

# Check pods — new pod will show: ErrImagePull
kubectl get pods
```
> ✅ Kubernetes keeps the **old running pod alive** while attempting the new one. No downtime.

### View Rollout History

```bash
kubectl rollout history deployment/gatewayserver-deployment
# REVISION  CHANGE-CAUSE
# 1         <none>
# 2         kubectl set image ... s111   (invalid)
# 3         kubectl set image ... s11    (current)
```

### Rollback to a Previous Revision

```bash
# Rollback to specific revision
kubectl rollout undo deployment/gatewayserver-deployment --to-revision=1

# Verify rollback — check pod image
kubectl describe pod <new-pod-name>
# Image: eazybytes/gatewayserver:s12  ← back to revision 1
```

### Scaling Replicas (CLI vs YAML)

```bash
# Via command (immediate, but doesn't update YAML file)
kubectl scale deployment accounts-deployment --replicas=1

# Via YAML (recommended — always keep YAML in sync)
# Update replicas in accounts.yaml then:
kubectl apply -f 5_accounts.yaml
```

### Kubernetes Object Relationships
```
Deployment  →  creates  →  ReplicaSet  →  creates  →  Pod  →  runs  →  Container
(specs)                    (count)                    (unit)            (app)
```

### Auto Scaling (Horizontal Pod Autoscaler)
> Kubernetes can automatically scale pods based on CPU/memory utilization using `HorizontalPodAutoscaler` objects. This is an advanced topic handled by Kubernetes admins.
```bash
# Conceptual — handled by DevOps/platform team
kubectl autoscale deployment accounts-deployment --min=1 --max=10 --cpu-percent=70
```

---

## Lec 13 — Introduction to Kubernetes Service Types

### Three Main Service Types

| Service Type | Exposure | Use Case |
|-------------|----------|----------|
| **ClusterIP** | Internal only (within cluster) | Default type; for microservice-to-microservice communication |
| **NodePort** | External via Worker Node IP + Port | Testing; exposes on port 30000–32767 on each node |
| **LoadBalancer** | External via stable public IP | Production external exposure; cloud provides a real public IP |

---

### 1. ClusterIP (Default)

```yaml
spec:
  type: ClusterIP    # Can omit — this is the default
  ports:
    - port: 80           # Port used inside cluster
      targetPort: 8080   # Container port
```

- Other services in cluster → communicate via **service name** or **cluster IP** at `port: 80`
- External traffic → ❌ **Blocked**
- Kubernetes auto load-balances between pods of the same service

**Example:** Gateway Server calls Accounts at `http://accounts:80`

---

### 2. NodePort

```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 32593     # Optional; auto-assigned in range 30000-32767 if omitted
```

- Exposed at: `<WorkerNodeIP>:<nodePort>`
- **Drawback**: client apps must know the worker node IP, which can change.

**Traffic Flow:**
```
External Client → Worker Node IP:32593
                → ClusterIP:80
                → Pod:8080
```

---

### 3. LoadBalancer

```yaml
spec:
  type: LoadBalancer
  ports:
    - port: 8071
      targetPort: 8071
```

- Kubernetes creates an external **LoadBalancer** with a **stable public IP** (or `localhost` locally).
- The public IP **never changes** — can be mapped to a domain name.
- On cloud: each LoadBalancer service incurs **extra billing** for the public IP.

**Traffic Flow:**
```
External Client → LoadBalancer (stable public IP)
               → Worker Node:NodePort
               → ClusterIP
               → Pod:containerPort
```

---

### Best Practice Architecture

```
Internet → Gateway Server (LoadBalancer: port 8080)
                ↓ (internal ClusterIP)
         ┌──────┴───────────────────┐
         ↓                          ↓
  Accounts (ClusterIP)     Loans (ClusterIP)
         ↓
  Cards (ClusterIP)
```

> 🔐 Only the **Gateway Server** should use `LoadBalancer`. All other microservices use `ClusterIP` for security.

---

## Lec 14 — Demo of Kubernetes Service Types

### Check Current Service Types
```bash
kubectl get services
# NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
# accounts     LoadBalancer   10.108.x.x     localhost     8080:30175/TCP
# kubernetes   ClusterIP      10.96.0.1      <none>        443/TCP
```

### Switch Accounts Service to ClusterIP

```yaml
# In accounts.yaml — change:
spec:
  type: ClusterIP   # was LoadBalancer
```

```bash
kubectl apply -f 5_accounts.yaml
kubectl get services
# accounts     ClusterIP   10.108.x.x   <none>   8080/TCP

# Test — should FAIL (no external access)
# http://localhost:8080/api/contact-info  → ERR_CONNECTION_REFUSED ✅
```

### Switch to NodePort (for Testing)
```yaml
spec:
  type: NodePort
  # (no nodePort value → randomly assigned)
```

```bash
kubectl apply -f 5_accounts.yaml
kubectl get services
# accounts  NodePort  10.108.x.x  <none>  8080:31182/TCP

# Access via NodePort
# http://localhost:31182/api/contact-info  → ✅ Success
# http://localhost:8080/api/contact-info   → ❌ Fails
```

### Why Prefer LoadBalancer over NodePort
| | LoadBalancer | NodePort |
|--|--|--|
| External IP stability | ✅ Stable (won't change) | ❌ Changes with worker nodes |
| Cloud billing | ❌ Costs per IP | ✅ No extra cost |
| Domain mapping | ✅ Easy | ❌ Complex |

> For the coming sections with Helm, all microservices (except Gateway) will be changed to **ClusterIP**.

---

## Lec 15 — Problems with Manually Created Kubernetes Manifest Files

### Problems with the Manual Approach

| Problem | Description |
|---------|-------------|
| **Too Many Files** | 100 microservices = 100+ manifest files to write and maintain |
| **Manual Apply** | Must run `kubectl apply -f` for each file individually |
| **Multi-Environment** | Dev/QA/Prod need different configs (replicas, resources). Files multiply: `100 services × 3 envs = 300+ files` |
| **Manual Delete** | Must run `kubectl delete -f` for each file to uninstall |

### Demo: Manual Delete (Shows the Pain)
```bash
kubectl delete -f 8_gateway.yaml
kubectl delete -f 7_cards.yaml
kubectl delete -f 6_loans.yaml
kubectl delete -f 5_accounts.yaml
kubectl delete -f 4_eurekaserver.yaml
kubectl delete -f 3_configserver.yaml
kubectl delete -f 2_configmaps.yaml
kubectl delete -f 1_keycloak.yaml

# Verify everything is gone
kubectl get deployments
kubectl get pods
kubectl get services
```

### The Solution → **Helm**
> **Helm** is a **package manager for Kubernetes** (like npm for Node.js).

With Helm:
- Define templates once, parameterize for different environments.
- Install/uninstall entire application stacks with one command.
- Manage 3rd-party components (Kafka, Grafana, Redis) easily via pre-built Helm charts.

> 📌 Helm will be covered in detail in the **next section**.

---

## Quick Reference: Essential `kubectl` Commands

### Cluster Info
```bash
kubectl config get-contexts
kubectl config get-clusters
kubectl config use-context docker-desktop
kubectl get nodes
```

### Deployments
```bash
kubectl get deployments
kubectl describe deployment <name>
kubectl apply -f <file.yaml>
kubectl delete -f <file.yaml>
```

### Pods
```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl delete pod <pod-name>
```

### Services
```bash
kubectl get services
kubectl describe service <name>
```

### Replica Sets
```bash
kubectl get replicaset
```

### Scaling
```bash
kubectl scale deployment <deployment-name> --replicas=<count>
```

### Rolling Updates
```bash
# Update image
kubectl set image deployment/<deployment> <container>=<image>:<tag>

# View history
kubectl rollout history deployment/<deployment>

# Rollback
kubectl rollout undo deployment/<deployment> --to-revision=<n>
```

### Events & Debugging
```bash
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

### ConfigMaps & Secrets
```bash
kubectl get configmap
kubectl describe configmap <name>
kubectl get secrets
```

---

## Summary: Section 15 Concepts Map

```
Container Orchestration
└── Kubernetes (K8s)
    ├── Architecture
    │   ├── Master Node (Control Plane)
    │   │   ├── Kube API Server
    │   │   ├── Scheduler
    │   │   ├── Controller Manager
    │   │   └── etcd (key-value store)
    │   └── Worker Node
    │       ├── Kubelet (agent)
    │       ├── Container Runtime (Docker)
    │       ├── Kube Proxy
    │       └── Pods → Containers
    │
    ├── Manifest Files (YAML)
    │   ├── Deployment (how to deploy)
    │   └── Service (how to expose)
    │       ├── ClusterIP (internal only)
    │       ├── NodePort (external via node port)
    │       └── LoadBalancer (external via stable IP)
    │
    ├── ConfigMap (non-sensitive env vars)
    ├── Secret (sensitive env vars)
    │
    ├── Features
    │   ├── Self-Healing (auto pod replacement)
    │   ├── Rolling Updates (zero downtime)
    │   ├── Rollback (to previous revision)
    │   └── Auto Scaling (HPA - advanced)
    │
    └── Problems with Manual Manifests → Solution: Helm (next section)
```

