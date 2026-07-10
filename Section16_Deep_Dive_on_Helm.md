# Section 16 — Deep Dive on Helm

---

## Table of Contents

| Lec # | Topic |
|-------|-------|
| 1 | [Introduction to Helm & the Problems that it Solves](#lec-1--introduction-to-helm--the-problems-that-it-solves) |
| 2 | [Installing Helm](#lec-2--installing-helm) |
| 3 | [Installing a Sample Helm Chart](#lec-3--installing-a-sample-helm-chart) |
| 4 | [Understanding Helm Chart Structure](#lec-4--understanding-helm-chart-structure) |
| 5 | [Creating Our Own Helm Chart & Template Files](#lec-5--creating-our-own-helm-chart--template-files) |
| 6 | [Creating Helm Chart for Accounts Microservice](#lec-6--creating-helm-chart-for-accounts-microservice) |
| 7 | [Creating Helm Charts for Other Microservices](#lec-7--creating-helm-charts-for-other-microservices) |
| 8 | [Creating Helm Charts for Dev, QA, and Prod Environments](#lec-8--creating-helm-charts-for-dev-qa-and-prod-environments) |
| 9 | [Demo of `helm template` Command](#lec-9--demo-of-helm-template-command) |
| 10 | [Install KeyCloak in Kubernetes Cluster Using Helm Chart](#lec-10--install-keycloak-in-kubernetes-cluster-using-helm-chart) |
| 11 | [Install Kafka in Kubernetes Cluster Using Helm Chart](#lec-11--install-kafka-in-kubernetes-cluster-using-helm-chart) |
| 12 | [Install Prometheus in Kubernetes Cluster Using Helm Chart](#lec-12--install-prometheus-in-kubernetes-cluster-using-helm-chart) |
| 13 | [Install Grafana Loki & Tempo in Kubernetes Cluster Using Helm Chart](#lec-13--install-grafana-loki--tempo-in-kubernetes-cluster-using-helm-chart) |
| 14 | [Install Grafana in Kubernetes Cluster Using Helm Chart](#lec-14--install-grafana-in-kubernetes-cluster-using-helm-chart) |
| 15 | [Install EazyBank Microservices in Kubernetes Cluster Using Helm Chart](#lec-15--install-eazybank-microservices-in-kubernetes-cluster-using-helm-chart) |
| 16 | [Demo of `helm upgrade` Command](#lec-16--demo-of-helm-upgrade-command) |
| 17 | [Demo of `helm history` and `rollback` Commands](#lec-17--demo-of-helm-history-and-rollback-commands) |
| 18 | [Demo of `helm uninstall` Command](#lec-18--demo-of-helm-uninstall-command) |
| 19 | [Quick Revision of Important Helm Commands](#lec-19--quick-revision-of-important-helm-commands) |
| — | [Quick Reference](#section-16--quick-reference) |

---

## Lec 1 — Introduction to Helm & the Problems that it Solves

### What is Helm?
> **Helm is a package manager for Kubernetes.**

- Helm helps developers and DevOps team members **manage Kubernetes projects and deployments** by offering a more efficient approach in handling Kubernetes manifest files.
- Just like **pip** (Python) or **npm** (JavaScript), Helm is the best way to find, share, and use software built for Kubernetes.

### Problems Without Helm

Without Helm, you need to maintain **separate Kubernetes manifest files** for each microservice (deployment, service, configmap, etc.) and manually apply or delete them with `kubectl` commands.

**Example:** For `accounts`, `loans`, and `cards` microservices — you need 3 different Service manifest files. Their skeleton is almost **identical**, except for a few dynamic values:

| Field | Static/Dynamic |
|-------|---------------|
| `apiVersion: v1` | ✅ Static |
| `kind: Service` | ✅ Static |
| `metadata.name` | ⚡ Dynamic |
| `spec.selector.app` | ⚡ Dynamic |
| `spec.type` | ⚡ Dynamic |
| `spec.ports.protocol: TCP` | ✅ Static |
| `spec.ports.port` | ⚡ Dynamic |
| `spec.ports.targetPort` | ⚡ Dynamic |

### How Helm Solves This — Templates + Values

Helm approach:
1. Create **one single template YAML file** (e.g., `service.yaml`) for all microservices.
2. Dynamic values are injected at runtime using **double curly brace syntax** `{{ .Values.key }}`.
3. Provide a **`values.yaml`** file per microservice with the dynamic values.
4. Helm generates the actual Kubernetes manifest files automatically at runtime.

**Example — Helm service template (single file for all microservices):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.deploymentLabel }}
spec:
  selector:
    app: {{ .Values.appLabel }}
  type: {{ .Values.service.type }}
  ports:
    - name: http
      protocol: TCP
      port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
```

**Example — `values.yaml` for Accounts microservice:**
```yaml
deploymentLabel: accounts
appLabel: accounts
service:
  type: ClusterIP
  port: 8080
  targetPort: 8080
```

### Three Key Advantages of Helm

| Advantage | Description |
|-----------|-------------|
| **Packaging** | All Kubernetes manifest files bundled into a single Helm **Chart**, shareable via public or private repositories |
| **Easier Installation** | Deploy, upgrade, rollback, or uninstall entire microservice networks with a **single command** |
| **Release/Version Management** | Roll back the **entire Kubernetes cluster** to a previous working state with a single command (unlike `kubectl rollout`, which only rolls back individual microservices) |

### Helm Charts
- A **Chart** is a collection of files describing a related set of Kubernetes resources.
- Charts can have **child charts / dependent charts** (like parent-child classes in Java).
- Install your whole dependency tree — your own microservices + third-party components — with one command.

---

## Lec 2 — Installing Helm

### Prerequisites
1. A running Kubernetes cluster (local or cloud).
2. Security configuration (optional).
3. Helm installed on local system.

### Installation Options

#### macOS (using Homebrew)
```bash
brew install helm
```

#### Windows (using Chocolatey)

**Step 1 — Install Chocolatey (if not already installed):**
1. Open **PowerShell as Administrator** (right-click → Run as administrator).
2. Check execution policy:
```powershell
Get-ExecutionPolicy
```
3. If output is `Restricted`, change it:
```powershell
Set-ExecutionPolicy AllSigned
```
4. Run the Chocolatey install command (from [chocolatey.org/install](https://chocolatey.org/install)):
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```
5. Verify Chocolatey installation:
```powershell
choco
```

**Step 2 — Install Helm using Chocolatey:**
```powershell
choco install kubernetes-helm
```

#### Linux/Ubuntu
Refer to [helm.sh/docs/intro/install](https://helm.sh/docs/intro/install/) for script-based or binary installation.

### Verify Helm Installation
```bash
helm version
```
Example output: `version.BuildInfo{Version:"v3.2.3", ...}`

---

## Lec 3 — Installing a Sample Helm Chart

### Step 1 — Verify Local Kubernetes Cluster is Running
```bash
kubectl get services
```

### Step 2 — List Existing Helm Releases
```bash
helm ls
```
> Helm reads Kubernetes cluster connection details from `~/.kube/config` (same file used by `kubectl`).

### Step 3 — Search for a Helm Chart

**Search on Artifact Hub (public repositories):**
```bash
helm search hub wordpress
```

### Step 4 — Add a Helm Repository

Before installing a chart from a remote repository, add the repo locally:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```
> **Bitnami** is a famous repository that maintains **production-ready Helm charts** for many popular software products.

### Step 5 — Install a Helm Chart
```bash
helm install happy-panda bitnami/wordpress
```
- `happy-panda` → the **release name** (can be any name you choose).
- `bitnami/wordpress` → `<repo-name>/<chart-name>`.

### Step 6 — Access the Installed App
```bash
# Get the WordPress URL
export SERVICE_IP=$(kubectl get svc --namespace default happy-panda-wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
echo "WordPress URL: http://$SERVICE_IP/"

# Get the admin username
echo Username: user

# Get the admin password
echo Password: $(kubectl get secret --namespace default happy-panda-wordpress -o jsonpath="{.data.wordpress-password}" | base64 -d)
```
> Wait **1–2 minutes** for the LoadBalancer IP to become available.

### What Helm Creates Behind the Scenes
Behind a single `helm install` command, Helm automatically created:
- ✅ **Deployments** (WordPress + MariaDB)
- ✅ **Pods** (WordPress pod + MariaDB pod)
- ✅ **ReplicaSets**
- ✅ **Services** (WordPress as `LoadBalancer`, MariaDB as `ClusterIP`)
- ✅ **ConfigMaps**
- ✅ **Secrets** (admin credentials stored securely)

### Check Helm Environment Variables
```bash
helm env
```
Helm charts are cached locally under `HELM_CACHE_HOME` (e.g., `~/Library/Caches/helm/repository/`).

### Uninstall a Helm Chart
```bash
helm uninstall happy-panda
```
> A single `helm uninstall` removes **all** Kubernetes objects (deployments, services, secrets, configmaps, pods) created by that chart.

---

## Lec 4 — Understanding Helm Chart Structure

### Standard Helm Chart Structure
```
<chart-name>/              ← Parent folder (name = chart name)
├── Chart.yaml             ← Metadata about the chart
├── values.yaml            ← Default dynamic values for templates
├── charts/                ← Dependent charts folder
│   └── <dependency>.tgz   ← Packaged dependent helm charts
└── templates/             ← Kubernetes manifest template files
    ├── deployment.yaml
    ├── service.yaml
    ├── configmap.yaml
    └── ...
```

> Additional files like `.helmignore`, `Chart.lock`, `values.schema.json` are managed by Helm itself and don't need manual creation.

### `Chart.yaml` — Chart Metadata
```yaml
apiVersion: v2
name: wordpress
description: WordPress is the world's most popular blogging and content management platform
type: application
version: 17.1.4          # Version of the Helm chart itself
appVersion: "6.3.1"      # Version of the application (WordPress) being deployed
maintainers:
  - name: Bitnami
dependencies:
  - name: memcached
    version: "6.x.x"
    repository: https://charts.bitnami.com/bitnami
  - name: mariadb
    version: "12.x.x"
    repository: https://charts.bitnami.com/bitnami
  - name: common
    version: "2.x.x"
    repository: https://charts.bitnami.com/bitnami
```

### `values.yaml` — Dynamic Values
Contains all key-value pairs that get injected into templates at runtime.
```yaml
image:
  repository: bitnami/wordpress
  tag: latest
replicaCount: 1
service:
  type: LoadBalancer
  port: 80
```

### `charts/` — Dependent Charts
Contains other Helm charts that the current chart depends on (e.g., WordPress depends on MariaDB, Memcached).

### `templates/` — Kubernetes Manifest Templates
Contains template YAML files with dynamic placeholders. Example `deployment.yaml` template:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.deploymentName }}
spec:
  replicas: {{ .Values.replicaCount }}
  ...
  containers:
    - image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

### Uninstall Chart (Demo Cleanup)
```bash
helm ls
# Shows: NAME=happy-panda, CHART=wordpress-17.1.4, APP VERSION=6.3.1

helm uninstall happy-panda
# Removes all WordPress + MariaDB objects from the cluster
```

---

## Lec 5 — Creating Our Own Helm Chart & Template Files

### Overview
In real enterprise projects, you must build your **own Helm charts** based on your microservices. The goal is to create a **common Helm chart** (`eazybank-common`) that holds reusable template files for all microservices.

### Step 1 — Create Folder Structure
```
Section_16/
└── Helm/
    ├── eazybank-common/       ← Common chart with reusable templates
    └── eazybank-services/     ← Individual microservice charts
```

### Step 2 — Create the Common Helm Chart
```bash
cd Section_16/Helm
helm create eazybank-common
```
- Delete all auto-generated template files inside `templates/`.
- Clear all content in `values.yaml` (leave it empty).
- `charts/` folder remains empty (no dependencies for common chart).

### Step 3 — Update `Chart.yaml` for Common Chart
```yaml
apiVersion: v2
name: eazybank-common
description: A common Helm chart for EazyBank microservices
type: application
version: 0.1.0
appVersion: "1.0.0"
```

### Step 4 — Create Template Files

> **Important:** Helm uses **Go template language** (and sometimes Spring template language). Templates are defined with `{{- define "name" -}}` ... `{{- end }}`.
> - `{{- define "name" -}}` → gives the template a reusable name
> - `{{- end }}` → closes the template definition
> - The hyphens (`-`) trim whitespace before/after the statement.

---

#### `templates/service.yaml` — Service Template
```yaml
{{- define "common.service" -}}
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.serviceName }}
spec:
  selector:
    app: {{ .Values.appLabel }}
  type: {{ .Values.service.type }}
  ports:
    - name: http
      protocol: TCP
      port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
{{- end }}
```

---

#### `templates/deployment.yaml` — Deployment Template
```yaml
{{- define "common.deployment" -}}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.deploymentName }}
  labels:
    app: {{ .Values.appLabel }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.appLabel }}
  template:
    metadata:
      labels:
        app: {{ .Values.appLabel }}
    spec:
      containers:
        - name: {{ .Values.appLabel }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: {{ .Values.containerPort }}
              protocol: TCP
          env:
            {{- if .Values.appName_enabled }}
            - name: SPRING_APPLICATION_NAME
              value: {{ .Values.appName }}
            {{- end }}
            {{- if .Values.profile_enabled }}
            - name: SPRING_PROFILES_ACTIVE
              valueFrom:
                configMapKeyRef:
                  name: {{ .Values.global.configMapName }}
                  key: SPRING_PROFILES_ACTIVE
            {{- end }}
            {{- if .Values.config_enabled }}
            - name: SPRING_CONFIG_IMPORT
              valueFrom:
                configMapKeyRef:
                  name: {{ .Values.global.configMapName }}
                  key: SPRING_CONFIG_IMPORT
            {{- end }}
            {{- if .Values.eureka_enabled }}
            - name: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
              valueFrom:
                configMapKeyRef:
                  name: {{ .Values.global.configMapName }}
                  key: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
            {{- end }}
            {{- if .Values.resourceserver_enabled }}
            - name: SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_JWK_SET_URI
              valueFrom:
                configMapKeyRef:
                  name: {{ .Values.global.configMapName }}
                  key: SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_JWK_SET_URI
            {{- end }}
            {{- if .Values.otel_enabled }}
            - name: JAVA_TOOL_OPTIONS
              valueFrom:
                configMapKeyRef:
                  name: {{ .Values.global.configMapName }}
                  key: JAVA_TOOL_OPTIONS
            - name: OTEL_EXPORTER_OTLP_ENDPOINT
              valueFrom:
                configMapKeyRef:
                  name: {{ .Values.global.configMapName }}
                  key: OTEL_EXPORTER_OTLP_ENDPOINT
            - name: OTEL_METRICS_EXPORTER
              valueFrom:
                configMapKeyRef:
                  name: {{ .Values.global.configMapName }}
                  key: OTEL_METRICS_EXPORTER
            - name: OTEL_SERVICE_NAME
              value: {{ .Values.appName }}
            {{- end }}
            {{- if .Values.kafka_enabled }}
            - name: SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS
              valueFrom:
                configMapKeyRef:
                  name: {{ .Values.global.configMapName }}
                  key: SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS
            {{- end }}
{{- end }}
```

**Explanation of `if` conditions in deployment template:**

| Variable | Purpose |
|----------|---------|
| `appName_enabled` | Inject `SPRING_APPLICATION_NAME` env var |
| `profile_enabled` | Inject `SPRING_PROFILES_ACTIVE` from ConfigMap |
| `config_enabled` | Inject `SPRING_CONFIG_IMPORT` (Config Server URL) |
| `eureka_enabled` | Inject Eureka server URL |
| `resourceserver_enabled` | Inject KeyCloak JWT URL (only for Gateway Server) |
| `otel_enabled` | Inject OpenTelemetry + Grafana Tempo variables |
| `kafka_enabled` | Inject Kafka broker URL (only for accounts & message microservices) |

---

#### `templates/configmap.yaml` — ConfigMap Template
```yaml
{{- define "common.configmap" -}}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Values.global.configMapName }}
data:
  SPRING_PROFILES_ACTIVE: {{ .Values.global.activeProfile }}
  SPRING_CONFIG_IMPORT: {{ .Values.global.configServerUrl }}
  EUREKA_CLIENT_SERVICEURL_DEFAULTZONE: {{ .Values.global.eurekaServiceUrl }}
  SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_JWK_SET_URI: {{ .Values.global.keycloakUrl }}
  JAVA_TOOL_OPTIONS: {{ .Values.global.javaToolOptions }}
  OTEL_EXPORTER_OTLP_ENDPOINT: {{ .Values.global.otelExporterEndpoint }}
  OTEL_METRICS_EXPORTER: {{ .Values.global.otelMetricsExporter }}
  SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS: {{ .Values.global.kafkaBrokerUrl }}
{{- end }}
```

> **Note:** The `global` prefix in property names is a convention to indicate these values are **shared across all microservices**.

### Step 5 — Leave `values.yaml` Empty for Common Chart
The `eazybank-common` chart is a **library/template provider** — its consumers (microservice-specific charts) provide their own `values.yaml`.

---

## Lec 6 — Creating Helm Chart for Accounts Microservice

### Overview
The `accounts` Helm chart will:
1. **Depend on** `eazybank-common` (to use its templates).
2. Provide its own `values.yaml` with accounts-specific values.
3. Contain template files that **reference** the named templates from `eazybank-common`.

### Step 1 — Create the Accounts Helm Chart
```bash
cd Section_16/Helm/eazybank-services
helm create accounts
```
- Delete all auto-generated template files inside `templates/`.
- Clear `values.yaml`.

### Step 2 — Update `Chart.yaml` — Add Dependency on `eazybank-common`
```yaml
apiVersion: v2
name: accounts
description: A Helm chart for Accounts Microservice
type: application
version: 0.1.0
appVersion: "1.0.0"
dependencies:
  - name: eazybank-common
    version: "0.1.0"
    repository: "file://../../../eazybank-common"
    # file:// → chart is on local filesystem
    # ../../../eazybank-common → navigate 3 folders up from Chart.yaml location
```

> **Note:** `file://` prefix with relative paths points to a chart on the local filesystem. For remote charts, use `https://` URLs.

### Step 3 — Create Template Files (Reference Common Templates)

**`templates/deployment.yaml`:**
```yaml
{{ include "common.deployment" . }}
```

**`templates/service.yaml`:**
```yaml
{{ include "common.service" . }}
```

> The `.` (dot) passes the current context (all values) to the referenced template.

### Step 4 — Populate `values.yaml` for Accounts
```yaml
deploymentName: accounts-deployment
serviceName: accounts
appLabel: accounts
appName: accounts
replicaCount: 1

image:
  repository: eazybytes/accounts
  tag: "s14"

containerPort: 8080

service:
  type: ClusterIP       # Not exposed externally; only gateway communicates with it
  port: 8080
  targetPort: 8080

# Feature flags — control which env vars get injected
appName_enabled: true
profile_enabled: true
config_enabled: true       # Needs to connect to Config Server
eureka_enabled: true
resourceserver_enabled: false   # Only gateway server acts as resource server
otel_enabled: true         # Send traces to Grafana Tempo
kafka_enabled: true        # Publishes events to Kafka for message microservice
```

### Step 5 — Compile (Build Dependencies)
```bash
cd accounts
helm dependency build
```
- Downloads and packages `eazybank-common` chart into the `charts/` folder (as a `.tgz` file).
- Run this command **whenever your chart has dependencies on other charts**.

```
accounts/
└── charts/
    └── eazybank-common-0.1.0.tgz   ← auto-generated after dependency build
```

---

## Lec 7 — Creating Helm Charts for Other Microservices

### Approach
Copy the `accounts` Helm chart folder and adjust `Chart.yaml` name and `values.yaml` values for each microservice.

### `values.yaml` Differences Per Microservice

#### Cards Microservice
```yaml
deploymentName: cards-deployment
serviceName: cards
appLabel: cards
appName: cards
image:
  repository: eazybytes/cards
  tag: "s14"
containerPort: 9000
service:
  type: ClusterIP
  port: 9000
  targetPort: 9000
appName_enabled: true
profile_enabled: true
config_enabled: true
eureka_enabled: true
resourceserver_enabled: false
otel_enabled: true
kafka_enabled: false   # Cards does NOT use Kafka
```

#### Loans Microservice
```yaml
deploymentName: loans-deployment
serviceName: loans
appLabel: loans
appName: loans
image:
  repository: eazybytes/loans
  tag: "s14"
containerPort: 8090
service:
  type: ClusterIP
  port: 8090
  targetPort: 8090
appName_enabled: true
profile_enabled: true
config_enabled: true
eureka_enabled: true
resourceserver_enabled: false
otel_enabled: true
kafka_enabled: false   # Loans does NOT use Kafka
```

#### Config Server
```yaml
deploymentName: configserver-deployment
serviceName: configserver
appLabel: configserver
appName: configserver
image:
  repository: eazybytes/configserver
  tag: "s14"
containerPort: 8071
service:
  type: ClusterIP
  port: 8071
  targetPort: 8071
appName_enabled: true
profile_enabled: false  # Config server loads all profiles, no need to specify one
config_enabled: false   # Config server IS the config server, doesn't connect to another
eureka_enabled: false
resourceserver_enabled: false
otel_enabled: true
kafka_enabled: false
```

#### Eureka Server
```yaml
deploymentName: eurekaserver-deployment
serviceName: eurekaserver
appLabel: eurekaserver
appName: eurekaserver
image:
  repository: eazybytes/eurekaserver
  tag: "s14"
containerPort: 8070
service:
  type: ClusterIP
  port: 8070
  targetPort: 8070
appName_enabled: true
profile_enabled: false
config_enabled: true    # Fetches config from Config Server
eureka_enabled: false   # Doesn't register with itself
resourceserver_enabled: false
otel_enabled: true
kafka_enabled: false
```

#### Gateway Server
```yaml
deploymentName: gatewayserver-deployment
serviceName: gatewayserver
appLabel: gatewayserver
appName: gatewayserver
image:
  repository: eazybytes/gatewayserver
  tag: "s14"
containerPort: 8072
service:
  type: LoadBalancer   # Exposed externally — clients connect through gateway
  port: 8072
  targetPort: 8072
appName_enabled: true
profile_enabled: true
config_enabled: true
eureka_enabled: true
resourceserver_enabled: true  # ONLY Gateway Server acts as OAuth2 Resource Server
otel_enabled: true
kafka_enabled: false
```

#### Message Microservice
```yaml
deploymentName: message-deployment
serviceName: message
appLabel: message
appName: message
image:
  repository: eazybytes/message
  tag: "s14"
containerPort: 9010
service:
  type: ClusterIP
  port: 9010
  targetPort: 9010
appName_enabled: true
profile_enabled: false  # Spring Cloud Function/Stream-based; no profile needed
config_enabled: false
eureka_enabled: false
resourceserver_enabled: false
otel_enabled: false
kafka_enabled: true    # Consumes events from Kafka
```

### Recompile After Changes to Common Chart
```bash
# Run for each microservice after modifying eazybank-common
cd accounts
helm dependency build

cd ../cards
helm dependency build

# ...repeat for all microservices
```

### Microservice Chart Feature Flag Summary

| Microservice | profile | config | eureka | resourceserver | otel | kafka |
|--------------|---------|--------|--------|----------------|------|-------|
| accounts     | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ |
| cards        | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| loans        | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| configserver | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| eurekaserver | ❌ | ✅ | ❌ | ❌ | ✅ | ❌ |
| gatewayserver| ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| message      | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

---

## Lec 8 — Creating Helm Charts for Dev, QA, and Prod Environments

### Concept
Create **environment-specific Helm charts** that:
- Depend on **all microservice charts**.
- Contain only a **ConfigMap template** (deployment/service templates are already in each microservice chart).
- Have a `values.yaml` with all **global/shared values** (URLs, profiles, etc.) for that environment.
- Allow deploying the **entire microservice network** with a **single `helm install` command**.

### Folder Structure
```
Section_16/Helm/
├── eazybank-common/
├── eazybank-services/
│   ├── accounts/
│   ├── cards/
│   ├── loans/
│   ├── configserver/
│   ├── eurekaserver/
│   ├── gatewayserver/
│   └── message/
└── environments/
    ├── dev-env/
    ├── qa-env/
    └── prod-env/
```

### Step 1 — Create Environment Chart
```bash
cd Section_16/Helm/environments
helm create dev-env
```
- Delete all auto-generated template files.
- Clear `values.yaml`.

### Step 2 — Update `Chart.yaml` — Depends on ALL Microservice Charts
```yaml
apiVersion: v2
name: dev-env
description: Helm chart for Dev Environment
type: application
version: 0.1.0
appVersion: "1.0.0"
dependencies:
  - name: eazybank-common
    version: "0.1.0"
    repository: "file://../../eazybank-common"
  - name: accounts
    version: "0.1.0"
    repository: "file://../../eazybank-services/accounts"
  - name: cards
    version: "0.1.0"
    repository: "file://../../eazybank-services/cards"
  - name: loans
    version: "0.1.0"
    repository: "file://../../eazybank-services/loans"
  - name: configserver
    version: "0.1.0"
    repository: "file://../../eazybank-services/configserver"
  - name: eurekaserver
    version: "0.1.0"
    repository: "file://../../eazybank-services/eurekaserver"
  - name: gatewayserver
    version: "0.1.0"
    repository: "file://../../eazybank-services/gatewayserver"
  - name: message
    version: "0.1.0"
    repository: "file://../../eazybank-services/message"
```

### Step 3 — Add ConfigMap Template to `templates/`

**`templates/configmap.yaml`:**
```yaml
{{ include "common.configmap" . }}
```

### Step 4 — Populate `values.yaml` for Dev Environment
```yaml
global:
  configMapName: eazybank-dev-configmap
  activeProfile: default      # Dev uses Spring Boot default profile

  configServerUrl: "optional:configserver:http://configserver:8071/"
  eurekaServiceUrl: "http://eurekaserver:8070/eureka/"

  # Keycloak service DNS within K8s cluster (installed via Helm)
  keycloakUrl: "http://keycloak.default.svc.cluster.local:80/realms/master/protocol/openid-connect/certs"

  # OpenTelemetry agent path (inside container)
  javaToolOptions: "-javaagent:/app/libs/opentelemetry-javaagent.jar"

  # Grafana Tempo endpoint for distributed tracing
  otelExporterEndpoint: "http://tempo.default.svc.cluster.local:4317"
  otelMetricsExporter: "none"

  # Kafka broker DNS within K8s cluster (installed via Helm)
  kafkaBrokerUrl: "kafka.default.svc.cluster.local:9092"
```

> **`global` prefix** is a naming convention — it signals that these values are **shared/common across all microservices** in the chart.

### Step 5 — Compile the Dev Environment Chart
```bash
cd dev-env
helm dependency build
```
After build, `charts/` folder will contain all dependent charts in `.tgz` format.

### QA Environment — `values.yaml` Changes
```yaml
global:
  configMapName: eazybank-qa-configmap
  activeProfile: qa           # Activates Spring Boot QA profile
  # All URLs remain the same unless different hostnames in QA
```

### Prod Environment — `values.yaml` Changes
```yaml
global:
  configMapName: eazybank-prod-configmap
  activeProfile: prod         # Activates Spring Boot prod profile
```

### Compile All Environments
```bash
cd qa-env && helm dependency build
cd ../prod-env && helm dependency build
```

---

## Lec 9 — Demo of `helm template` Command

### What is `helm template`?
> The `helm template` command **renders Kubernetes manifest files locally** without deploying them to the cluster. It's useful to **validate** the generated YAML before actual installation.

### Run `helm template`
```bash
cd Section_16/Helm/environments/dev-env
helm template .
```
> The `.` (dot) means: chart is available in the current folder.

**Sample output — Message microservice deployment:**
```yaml
---
# Source: dev-env/charts/message/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: message-deployment
spec:
  replicas: 1
  ...
  containers:
    - name: message
      image: "eazybytes/message:s14"
      env:
        - name: SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS
          valueFrom:
            configMapKeyRef:
              name: eazybank-dev-configmap
              key: SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS
```

### Workflow After Fixing Errors
If you make a change to `eazybank-common` templates, you must **recompile all dependent charts**:
```bash
# 1. Recompile all microservice charts
cd eazybank-services/accounts && helm dependency build
cd ../cards && helm dependency build
cd ../loans && helm dependency build
# ...repeat for all microservices

# 2. Recompile all environment charts
cd environments/dev-env && helm dependency build
cd ../qa-env && helm dependency build
cd ../prod-env && helm dependency build
```

### Validate for Different Environments
```bash
# Dev environment
cd environments/dev-env
helm template .
# ConfigMap name in output: eazybank-dev-configmap

# Prod environment
cd environments/prod-env
helm template .
# ConfigMap name in output: eazybank-prod-configmap
```

### Next Steps After Validation
Before deploying microservices, set up third-party components inside Kubernetes:
- ✅ **Keycloak** (OAuth2 / Identity Provider)
- ✅ **Kafka** (Event Broker)
- ✅ **Grafana** stack (Loki, Tempo, Prometheus)

These are **not** built manually — use **public Helm charts** (e.g., from Bitnami).

---

## Lec 10 — Install KeyCloak in Kubernetes Cluster Using Helm Chart

### Bitnami — The Go-To Source for Production-Ready Helm Charts
- Bitnami (supported by VMware) provides production-ready Helm charts for many popular products.
- GitHub: [github.com/bitnami/charts](https://github.com/bitnami/charts)
- Available charts include: Kafka, Keycloak, Grafana, Loki, Tempo, Prometheus, MySQL, PostgreSQL, Redis, MongoDB, Elasticsearch, Nginx, ArgoCD, and more.

### Step 1 — Get the Keycloak Helm Chart
Download or copy the `keycloak` Helm chart from Bitnami to your `Helm/` directory.

### Step 2 — Customize `values.yaml` for Keycloak

Open `keycloak/values.yaml` and make two changes:

**Change 1 — Service type (expose Keycloak externally):**
```yaml
# Find: type: ClusterIP
# Change to:
service:
  type: LoadBalancer
```

**Change 2 — Set admin password:**
```yaml
# Find: auth.adminPassword: ""
# Change to:
auth:
  adminUser: user
  adminPassword: "password"
```
> If `adminPassword` is left empty, Helm generates a random password stored in a Kubernetes Secret.

### Step 3 — Build Dependencies
```bash
cd keycloak
helm dependency build
```

### Step 4 — Install Keycloak
```bash
cd ..     # Go to Helm/ parent directory
helm install keycloak keycloak
# Syntax: helm install <release-name> <chart-folder>
```

### Step 5 — Access Keycloak
After installation, Helm prints access instructions:
```bash
# Get the Keycloak URL
echo "http://localhost:80"
# Username: user
# Password: password
```
> Wait **1–2 minutes** for the LoadBalancer to be ready.

### Step 6 — Configure Keycloak
1. Go to `http://localhost:80` → click **Administration Console**.
2. Login: `user` / `password`.
3. Create a new **Client**:
   - Client ID: `easybank-callcenter-cc`
   - Enable: `Client Authentication`
   - Disable: `Standard Flow`, `Direct Access Grants`
   - Enable: `Service Account Roles`
4. Go to **Credentials** → copy the `client_secret`.
5. Create **Roles**: `accounts`, `cards`, `loans`.
6. Go to **Clients → easybank-callcenter-cc → Service Account Roles** → Assign all created roles.

### How to Derive the Keycloak Internal DNS Name
After installation, Helm prints:
```
Keycloak can be accessed through the following DNS name from within your cluster:
keycloak.default.svc.cluster.local
```
This is the value used in `values.yaml` for the environment charts:
```yaml
keycloakUrl: "http://keycloak.default.svc.cluster.local:80/realms/master/protocol/openid-connect/certs"
```

### Keycloak Helm Chart — What it Creates Behind the Scenes
- Pod: `keycloak`
- Pod: `keycloak-postgresql` (Keycloak uses PostgreSQL as its database)
- Services, ConfigMaps, Secrets, PVCs, Deployments...

> Demonstrates the **power of Helm** — complex production setups with a single command. Without Helm, you'd need to manually create dozens of Kubernetes manifest files.

### Increase Docker Desktop Resources (for Local Dev)
For running all components locally, increase Docker Desktop resources:
- Go to **Docker Desktop → Settings → Resources**.
- Increase **CPUs**: 4 → 6
- Increase **Memory**: 8 GB → 12 GB

---

## Lec 11 — Install Kafka in Kubernetes Cluster Using Helm Chart

### Step 1 — Handle Persistent Volume Claims (PVC) Issue

> ⚠️ **Known Helm Limitation**: `helm uninstall` does **not** delete PVCs. If you uninstall and reinstall a chart, orphan PVCs can cause failures.

**Check existing PVCs:**
```bash
kubectl get pvc
```

**Delete orphan PVCs manually:**
```bash
kubectl delete pvc <pvc-name>
```
Or via Kubernetes Dashboard → Storage → Persistent Volume Claims → Delete.

### Step 2 — Get the Kafka Helm Chart
Copy the `kafka` chart from Bitnami to your `Helm/` directory.

### Step 3 — Customize Kafka `values.yaml`

**Change 1 — Reduce replica count for local testing:**
```yaml
# Default is 3 brokers (production standard)
# Change to 1 for local development
replicaCount: 1
```

**Change 2 — Disable SASL security (for local testing):**
```yaml
# Default (production): SASL_PLAINTEXT
# Change to: PLAINTEXT (no authentication)

# Find all occurrences of "SASL_PLAINTEXT" in values.yaml
# Change each to "PLAINTEXT"
# Allowed values: PLAINTEXT, SSL, SASL_PLAINTEXT, SASL_SSL
```

> In production, Kafka admins control replica counts and security settings. For local testing, we simplify.

### Step 4 — Build Dependencies
```bash
cd kafka
helm dependency build
```

### Step 5 — Install Kafka
```bash
cd ..     # Go to parent Helm/ directory
helm install kafka kafka
# Syntax: helm install <release-name> <chart-folder>
```

### Step 6 — Verify Kafka Installation
```bash
# Check pods
kubectl get pods
# Expected: kafka-0 pod

# Check services
kubectl get services
# Expected: kafka service (ClusterIP)
```
Via Kubernetes Dashboard → Workloads → Pods → `kafka-0`

### Kafka Internal DNS for Microservices
After installation, Helm output shows:
```
Each Kafka broker can be accessed by producers via port 9092 using the DNS name:
kafka.default.svc.cluster.local
```
This DNS name is used in the environment chart `values.yaml`:
```yaml
kafkaBrokerUrl: "kafka.default.svc.cluster.local:9092"
```
The **accounts** and **message** microservices use this URL to connect to Kafka.

### Helm Uninstall Commands Reference
```bash
# Check all releases
helm ls

# Uninstall a specific release
helm uninstall keycloak
helm uninstall kafka

# After uninstall, remember to delete leftover PVCs:
kubectl get pvc
kubectl delete pvc <pvc-name>
```

---

## Lec 12 — Install Prometheus in Kubernetes Cluster Using Helm Chart

### Goal
Set up **Prometheus** inside the Kubernetes cluster so it can scrape metrics from all individual microservices via the Spring Boot Actuator Prometheus endpoint.

### Chart Source
Use the `kube-prometheus` chart from Bitnami (a production-ready Prometheus stack). Copy the `kube-prometheus` folder from the Bitnami downloaded repository into your `Helm/` directory.

### Step 1 — Customize `values.yaml` for kube-prometheus

**Change 1 — Enable additional scrape configs:**
```yaml
# Find: additionalScrapeConfigs: {}
# Change to:
additionalScrapeConfigs:
  enabled: true
  type: internal   # only scrape inside the Kubernetes cluster (not external)
```

**Change 2 — Define scrape job list (in JSON format under `internal`):**
```yaml
additionalScrapeConfigs:
  enabled: true
  type: internal
  internal:
    jobList:
      - job_name: configserver
        metrics_path: /actuator/prometheus
        static_configs:
          - targets: ["configserver:8071"]
      - job_name: eurekaserver
        metrics_path: /actuator/prometheus
        static_configs:
          - targets: ["eurekaserver:8070"]
      - job_name: accounts
        metrics_path: /actuator/prometheus
        static_configs:
          - targets: ["accounts:8080"]
      - job_name: loans
        metrics_path: /actuator/prometheus
        static_configs:
          - targets: ["loans:8090"]
      - job_name: cards
        metrics_path: /actuator/prometheus
        static_configs:
          - targets: ["cards:9000"]
      - job_name: gatewayserver
        metrics_path: /actuator/prometheus
        static_configs:
          - targets: ["gatewayserver:8072"]
```
> Each job entry provides the Kubernetes **service name** (as DNS) and the **port** to connect to. Prometheus uses these to pull metrics from each microservice.

### Step 2 — Build Dependencies
```bash
cd kube-prometheus
helm dependency build
```

### Step 3 — Install Prometheus
```bash
cd ..     # Go back to Helm/ parent directory
helm install prometheus kube-prometheus
```

### Step 4 — Access Prometheus (Optional Debug)
By default, Prometheus is exposed as a **ClusterIP** (internal only). To temporarily access it:
```bash
kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090
```
Then open: `http://localhost:9090`

- Go to **Status → Targets** to see all scrape targets.
- Targets for accounts, cards, loans etc. will show **red (down)** until microservices are deployed.
- After `Ctrl+C`, Prometheus is no longer accessible from local machine.

### What Prometheus Monitors
Besides your microservices, the `kube-prometheus` chart also monitors many internal Kubernetes components (node-exporter, kube-state-metrics, etc.) — hence many targets appear in the Targets page.

---

## Lec 13 — Install Grafana Loki & Tempo in Kubernetes Cluster Using Helm Chart

### Overview
Two components need to be set up before Grafana:
1. **Grafana Loki** — aggregates and stores logs from all microservices.
2. **Grafana Tempo** — stores distributed traces sent by OpenTelemetry agents.

### Part A — Install Grafana Loki

#### Step 1 — No `values.yaml` Changes Needed
Loki works fine with **all default values** from the Bitnami chart. No customization is required.

#### Step 2 — Build Dependencies
```bash
cd grafana-loki
helm dependency build
```
> Loki has internal dependencies (e.g., Memcached) that are automatically downloaded.

#### Step 3 — Install Loki
```bash
cd ..    # Go back to parent Helm/ directory
helm install loki grafana-loki
# release name = loki, chart folder = grafana-loki
```

**Components installed by Loki:**
- Ingester, Distributor, Querier, Promtail, Compactor, Gateway

> Setting up all these manually (without Helm) could take **months of effort** for a Kubernetes admin.

---

### Part B — Install Grafana Tempo

#### Step 1 — Enable OTLP in `values.yaml`
By default, OpenTelemetry (OTLP) communication is **disabled** in the Tempo Helm chart. Both HTTP and gRPC protocols must be enabled so that the OpenTelemetry Java agent inside each microservice can send trace data to Tempo.

```yaml
# Search for "otlp" in values.yaml
# Find the section for OTLP receivers — both http and grpc are false by default
# Change both to true:

tempo:
  receivers:
    otlp:
      protocols:
        http:
          enabled: true    # was: false
        grpc:
          enabled: true    # was: false
```

#### Step 2 — Build Dependencies
```bash
cd grafana-tempo
helm dependency build
```

#### Step 3 — Install Tempo
```bash
cd ..
helm install tempo grafana-tempo
# release name = tempo, chart folder = grafana-tempo
```

**Components installed by Tempo:**
- Ingester, Distributor, Querier, Query-frontend, Compactor, Vulture

#### Step 4 — Find the Tempo Distributor Service (for OTEL endpoint)
After installation, run:
```bash
kubectl get services
```
Look for Tempo-related services. The service your microservices' OpenTelemetry agent should connect to is the **distributor**:

| Service (example) | Port | Used by |
|-------------------|------|---------|
| `tempo-distributor` | `4317` | OpenTelemetry Java agents in microservices |
| `tempo-query-frontend` | `3200` | Grafana (for tracing queries) |

This is why in the environment chart `values.yaml`:
```yaml
otelExporterEndpoint: "http://tempo.default.svc.cluster.local:4317"
```

> **How to know which service to use?** Based on official Tempo documentation: OpenTelemetry sources should send traces to the **distributor** component at port **4317** (gRPC).

---

## Lec 14 — Install Grafana in Kubernetes Cluster Using Helm Chart

### Goal
Install Grafana and configure it to automatically connect to:
- **Prometheus** (metrics)
- **Loki** (logs)
- **Tempo** (distributed traces)

### Step 1 — Get the Grafana Helm Chart
Copy the `grafana` folder from the Bitnami charts repository into your `Helm/` directory.

> **Tip:** Use the helm charts committed to the GitHub repo for the course — all `values.yaml` changes are already pre-applied.

### Step 2 — Configure Data Sources in `values.yaml`

Inside the Grafana `values.yaml`, locate the `secretDefinition` section. Data sources are configured here using Grafana's provisioning YAML format:

```yaml
# Find: secretDefinition: {}
# Replace with:
secretDefinition:
  apiVersion: 1
  deleteDatasources:
    - name: Prometheus
    - name: Loki
    - name: Tempo
  datasources:
    # ---- Prometheus ----
    - name: Prometheus
      type: prometheus
      url: http://<prometheus-service-dns>:9090
      isDefault: true

    # ---- Tempo ----
    - name: Tempo
      type: tempo
      url: http://tempo-grafana-tempo-query-frontend.default.svc.cluster.local:3200

    # ---- Loki ----
    - name: Loki
      type: loki
      url: http://<loki-gateway-service>.default.svc.cluster.local:80
      jsonData:
        derivedFields:
          # Link from Loki log to Tempo trace (via traceID field)
          - name: traceID
            matcherRegex: "traceId=(\\w+)"
            url: "${__value.raw}"
            datasourceName: Tempo
```

> - `deleteDatasources` → ensures old data source entries are cleaned up before re-adding.
> - `derivedFields` inside Loki config → enables **clickable trace links** directly from Loki log lines to Tempo.
> - Tempo Grafana integration uses the **query-frontend** service at port **3200** (not the distributor).
> - Loki Grafana integration uses the **gateway** service at port **80**.

### Step 3 — Build Dependencies
```bash
cd grafana
helm dependency build
```

### Step 4 — Install Grafana
```bash
cd ..
helm install grafana grafana
# release name = grafana, chart folder = grafana
```
By default, Grafana is exposed as a **ClusterIP** service.

### Step 5 — Access Grafana (Port Forward)
```bash
# Expose Grafana at local port 3000
kubectl port-forward svc/grafana 3000:3000
```
Then open: `http://localhost:3000`

> **Note:** Avoid using port `8080` for port-forward because the `accounts` microservice (LoadBalancer) also uses port `8080`, which would cause conflicts.

### Step 6 — Login to Grafana
- **Username:** `admin`
- **Password:** Retrieve from the Kubernetes Secret (see instructions printed by Helm install output):
  ```bash
  kubectl get secret grafana -o jsonpath="{.data.admin-password}" | base64 -d
  ```

### Step 7 — Validate Data Sources
Go to **Explore** (left sidebar) → click the data source dropdown.
You should see: **Loki**, **Prometheus**, **Tempo** — all auto-configured ✅

### Final State — All Releases Installed
After all installations, `helm ls` shows:

| Release | Chart |
|---------|-------|
| grafana | grafana |
| kafka | kafka |
| keycloak | keycloak |
| loki | grafana-loki |
| prometheus | kube-prometheus |
| tempo | grafana-tempo |

---

## Lec 15 — Install EazyBank Microservices in Kubernetes Cluster Using Helm Chart

### Deploy All Microservices with a Single Command

Navigate to the `environments` folder and choose the target environment:

```bash
cd Section_16/Helm/environments

# Deploy all microservices using the production environment chart
helm install easybank prod-env
# release name = easybank, chart folder = prod-env
```

### What Happens After `helm install`
All microservices **start simultaneously** — this means:
- Services that depend on Config Server (Eureka, Accounts, Cards, Loans, Gateway) will **initially fail to start**
- Kubernetes detects failures and **auto-restarts** pods with **exponential backoff** delays:
  - First retry: ~2 seconds
  - Subsequent retries: progressively longer (4s, 8s, 16s...)
- Eventually, once Config Server is up → Eureka starts → all other services start

### Startup Order & Expected Restart Counts

| Microservice | Depends On | Expected Restarts |
|---|---|---|
| configserver | Nothing | 0 |
| eurekaserver | configserver | 2+ (until config server is ready) |
| accounts | configserver + eurekaserver | 2+ |
| cards | configserver + eurekaserver | 2+ |
| loans | configserver + eurekaserver | 2+ |
| gatewayserver | configserver + eurekaserver | 2+ |
| message | Kafka only (no config/Eureka) | **0** (starts immediately) |

### Monitoring via Kubernetes Dashboard
Go to: **Dashboard → Workloads → Pods**
- Click on a pod → **View Logs**
- Enable **Auto Refresh every 5 seconds** to watch startup progress

### How to Confirm Each Service Started
Look for log statements like:
- Config Server: `Started ConfigServerApplication in X seconds`
- Eureka Server: `Started EurekaServerApplication in X seconds`
- Accounts, Cards, Loans, Gateway: `Started [Name]Application in X seconds`
- Message: connected to Kafka broker (logged on startup)

### Testing After All Services Start

**1. Confirm active profile (Production):**
```
GET http://<gateway-ip>:8072/eazybank/accounts/contact-info
```
Response should contain properties from the **prod** Spring profile.

**2. Get OAuth2 access token from Keycloak (port 80):**
```
POST http://localhost:80/realms/master/protocol/openid-connect/token
  client_id = easybank-callcenter-cc
  client_secret = <your-secret>
  grant_type = client_credentials
```

**3. Create Account, Card, Loan (using access token):**
```
POST /eazybank/accounts/api/create      → HTTP 201 Created ✅
POST /eazybank/cards/api/create         → HTTP 201 Created ✅
POST /eazybank/loans/api/create         → HTTP 201 Created ✅
```

**4. Fetch combined customer details:**
```
GET /eazybank/accounts/api/fetchCustomerDetails?mobileNumber=<number>
```
Response contains: `accounts` + `loans` + `cards` data ✅

### Validate Grafana (with port-forward active on 3000)

**Logs (Loki):**
- Explore → select **Loki** → label: `container` → value: `gatewayserver` → Run query
- Logs appear → click any log line → click trace ID link → opens in Tempo ✅

**Distributed Traces (Tempo):**
- Navigated from Loki log line → full distributed trace shown across microservices ✅

**Metrics (Prometheus):**
- Explore → select **Prometheus** → Metric: `up` → label: `container` → Run query
- Change time range to **Last 15 minutes** → graph style: **Stack lines**
- Graph shows uptime for all running containers ✅

---

## Lec 16 — Demo of `helm upgrade` Command

### Use Case
After deploying microservices with `helm install`, you may need to:
- Deploy a **new Docker image** (new version of a microservice)
- Change **replica count** (scale up/down)
- Update **configuration values**

### How `helm upgrade` Works
- Helm compares the current state with the new chart/values
- Deploys **only the changed components** (not the entire stack)
- Increments the **revision number** automatically

### Step-by-Step Demo

**Scenario:** Change the Gateway Server image from `s14` → `s11` (no OAuth2 security in s11)

**Step 1 — Modify `values.yaml` in the gateway server chart:**
```yaml
# In eazybank-services/gatewayserver/values.yaml
image:
  repository: eazybytes/gatewayserver
  tag: "s11"    # was: s14
```

**Step 2 — Recompile the environment chart (because prod-env depends on gatewayserver):**
```bash
cd environments/prod-env
helm dependency build
```

**Step 3 — Run `helm upgrade`:**
```bash
cd environments    # go to parent of prod-env
helm upgrade easybank prod-env
# Syntax: helm upgrade <release-name> <chart-folder>
```

**Output:**
- Revision increments (e.g., `REVISION: 2`)
- Helm deploys only the changed Gateway Server pod

**Step 4 — Verify in Kubernetes Dashboard:**
- Go to Pods → Gateway Server → Logs → confirm new pod started with new image

### Key Points

| Feature | `kubectl rollout restart` | `helm upgrade` |
|---------|--------------------------|---------------|
| Scope | One deployment at a time | All changed components at once |
| History | No automatic versioning | Tracks revisions automatically |
| Rollback | Manual per deployment | Single command for all |

> **Note from demo:** The lecturer initially typed `s12` instead of `s11` — section 12 still has OAuth2. The correct non-auth tag is `s11`. This shows that even mistakes are manageable with Helm's revision tracking.

---

## Lec 17 — Demo of `helm history` and `rollback` Commands

### `helm history` — View Deployment History

```bash
helm history easybank
```

**Example output** (after 3 installs/upgrades):

| REVISION | UPDATED | STATUS | CHART | APP VERSION | DESCRIPTION |
|---|---|---|---|---|---|
| 1 | ... | superseded | prod-env-0.1.0 | 1.0.0 | Install complete |
| 2 | ... | superseded | prod-env-0.1.0 | 1.0.0 | Upgrade complete (s12 tag) |
| 3 | ... | deployed | prod-env-0.1.0 | 1.0.0 | Upgrade complete (s11 tag) |

- **`superseded`** = older revision, replaced by a newer one
- **`deployed`** = currently active revision

### `helm rollback` — Roll Back to a Specific Revision

```bash
helm rollback easybank 1
# Syntax: helm rollback <release-name> <revision-number>
```

- Instantly rolls back the **entire Kubernetes cluster** state to revision 1
- A **single command** for rolling back all changed microservices simultaneously

**After rollback, `helm history easybank` shows:**

| REVISION | STATUS | DESCRIPTION |
|---|---|---|
| 1 | superseded | Install complete |
| 2 | superseded | Upgrade complete |
| 3 | superseded | Upgrade complete |
| 4 | **deployed** | **Rollback to 1** |

> Rollback creates a **new revision** (revision 4) rather than deleting old ones — preserving full audit trail.

### Roll Back to Immediate Previous Revision (Optional Syntax)
```bash
helm rollback easybank
# No revision number = rolls back to the immediately previous revision
```

### Comparison: Helm vs kubectl Rollback

| | `helm rollback` | `kubectl rollout undo` |
|---|---|---|
| Scope | Entire release (all microservices) | One Deployment at a time |
| Command count | **1 command** | N commands (one per microservice) |
| History tracking | Full revision history | Basic rollout history only |

### Validate Rollback
After rollback to revision 1 (which uses `s14` tag with security enabled):
```
POST /eazybank/accounts/api/create  (without auth token)
→ HTTP 401 Unauthorized ✅  (confirms rollback to secured version)
```

---

## Lec 18 — Demo of `helm uninstall` Command

### Uninstall All Releases

```bash
# List all installed releases
helm ls

# Uninstall all releases one by one:
helm uninstall easybank      # removes all EazyBank microservices (accounts, cards, loans, gateway, eureka, config, message)
helm uninstall grafana
helm uninstall tempo
helm uninstall loki
helm uninstall prometheus
helm uninstall kafka
helm uninstall keycloak

# Verify all removed
helm ls    # returns empty result
```

### What Gets Deleted
`helm uninstall` removes **all** Kubernetes resources created by that release:
- ✅ Pods
- ✅ Deployments
- ✅ Services
- ✅ ConfigMaps
- ✅ Secrets
- ❌ **Persistent Volume Claims (PVCs) — NOT deleted automatically**

### ⚠️ Critical: Manually Delete PVCs After Uninstall

```bash
# Check for leftover PVCs
kubectl get pvc

# Delete each one manually
kubectl delete pvc <pvc-name>
```
Or via **Kubernetes Dashboard → Storage → Persistent Volume Claims → Delete**

> **Why it matters:** Leftover PVCs from a previous installation will **cause errors** when reinstalling charts like Keycloak or Kafka. Always clean them up before reinstalling.

### Helm vs Kustomize

| | Helm | Kustomize |
|---|---|---|
| Learning curve | Steeper | Easier |
| Community charts | ✅ Huge (Bitnami, ArtifactHub, etc.) | ❌ Limited |
| Capabilities | More powerful | Simpler |
| Use case | Deployment + package management | Configuration management only |

- **Kustomize** (`kustomize.io`) is a popular alternative to Helm
- Some teams start with Kustomize and later adopt Helm for its ecosystem
- **Recommendation:** Start with Helm — richer ecosystem, more capabilities, and many community-maintained charts for third-party products (Kafka, Keycloak, Prometheus, etc.)

---

## Lec 19 — Quick Revision of Important Helm Commands

### All Key Helm Commands Summary

| Command | Syntax | Description |
|---------|--------|-------------|
| `helm create` | `helm create <chart-name>` | Creates a new Helm chart scaffold: `Chart.yaml`, `values.yaml`, `charts/`, `templates/` |
| `helm dependency build` | `helm dependency build` | Compiles the chart and downloads all dependent charts into the `charts/` folder as `.tgz` files |
| `helm install` | `helm install <release-name> <chart-folder>` | Installs a Helm chart into the Kubernetes cluster; creates all K8s resources |
| `helm upgrade` | `helm upgrade <release-name> <chart-folder>` | Deploys updated chart/values to an existing release; increments revision number |
| `helm history` | `helm history <release-name>` | Shows the full history of all revisions (installs, upgrades, rollbacks) for a release |
| `helm rollback` | `helm rollback <release-name> [revision]` | Rolls back to a specific revision; omit revision to go to the immediate previous |
| `helm uninstall` | `helm uninstall <release-name>` | Removes all K8s resources of a release (except PVCs — delete manually) |
| `helm template` | `helm template .` | Renders manifest YAML locally without deploying — useful for validation/debugging |
| `helm ls` | `helm ls` | Lists all currently installed Helm releases in the cluster |

### Typical Helm Lifecycle Workflow

```
helm create <chart>          ← Scaffold the chart
     ↓
Edit Chart.yaml + values.yaml + templates/
     ↓
helm dependency build        ← Compile & download dependencies
     ↓
helm template .              ← Validate rendered YAML (optional)
     ↓
helm install <release> <chart>   ← First deployment
     ↓
(make changes)
     ↓
helm dependency build        ← Recompile after changes
helm upgrade <release> <chart>   ← Roll out changes (new revision)
     ↓
(something goes wrong)
     ↓
helm history <release>       ← Identify which revision to rollback to
helm rollback <release> <n>  ← Roll back entire cluster in one command
     ↓
(cleanup)
     ↓
helm uninstall <release>     ← Remove everything
kubectl delete pvc --all     ← Also clean up PVCs manually
```

### All Helm Charts from This Course
All charts are committed to GitHub under the **Section 16 → Helm** folder:
```
Section_16/
└── Helm/
    ├── eazybank-common/       ← Common templates
    ├── eazybank-services/     ← Individual microservice charts
    │   ├── accounts/
    │   ├── cards/
    │   ├── loans/
    │   ├── configserver/
    │   ├── eurekaserver/
    │   ├── gatewayserver/
    │   └── message/
    ├── environments/          ← Environment-specific charts
    │   ├── dev-env/
    │   ├── qa-env/
    │   └── prod-env/
    ├── keycloak/              ← Bitnami chart (customized)
    ├── kafka/                 ← Bitnami chart (customized)
    ├── kube-prometheus/       ← Bitnami chart (customized)
    ├── grafana-loki/          ← Bitnami chart (defaults)
    ├── grafana-tempo/         ← Bitnami chart (OTLP enabled)
    └── grafana/               ← Bitnami chart (datasources configured)
```

> These charts can also be deployed to **cloud Kubernetes clusters** (covered in upcoming sections). The process is essentially the same — the main difference is more cluster capacity in the cloud.

---

## Section 16 — Quick Reference

### Core Helm Commands

| Command | Description |
|---------|-------------|
| `helm version` | Check Helm version |
| `helm ls` | List all installed releases |
| `helm env` | Show Helm environment variables |
| `helm search hub <keyword>` | Search public Artifact Hub |
| `helm repo add <name> <url>` | Add a Helm repository |
| `helm create <name>` | Scaffold a new Helm chart |
| `helm dependency build` | Compile chart and download dependencies |
| `helm template .` | Render templates locally (no cluster needed) |
| `helm install <release> <chart>` | Install a Helm chart |
| `helm upgrade <release> <chart>` | Deploy new changes to an existing release |
| `helm history <release>` | View full revision history of a release |
| `helm rollback <release> [n]` | Roll back to revision `n` (or previous if omitted) |
| `helm uninstall <release>` | Uninstall a Helm chart (PVCs not deleted!) |

### Helm Chart Hierarchy for EazyBank

```
dev-env / qa-env / prod-env  (environment chart)
├── eazybank-common (common templates)
├── accounts         ──→ uses eazybank-common
├── cards            ──→ uses eazybank-common
├── loans            ──→ uses eazybank-common
├── configserver     ──→ uses eazybank-common
├── eurekaserver     ──→ uses eazybank-common
├── gatewayserver    ──→ uses eazybank-common
└── message          ──→ uses eazybank-common
```

### Third-Party Helm Charts Used (Bitnami)

| Release Name | Chart Name | Purpose | Key Customization |
|---|---|---|---|
| `keycloak` | `keycloak` | OAuth2 / Identity Provider | Service type → LoadBalancer; admin password |
| `kafka` | `kafka` | Event/Message Broker | replicaCount → 1; SASL_PLAINTEXT → PLAINTEXT |
| `prometheus` | `kube-prometheus` | Metrics scraping | Enable additionalScrapeConfigs; job list |
| `loki` | `grafana-loki` | Log aggregation | None (use defaults) |
| `tempo` | `grafana-tempo` | Distributed tracing | Enable OTLP HTTP + gRPC receivers |
| `grafana` | `grafana` | Dashboards & visualization | Configure datasources (Prometheus, Loki, Tempo) |

### Deploy / Uninstall Entire Microservice Network
```bash
# Deploy all microservices + ConfigMap for prod
helm install easybank environments/prod-env

# Upgrade (roll out changes)
helm upgrade easybank environments/prod-env

# View history
helm history easybank

# Rollback to revision 1
helm rollback easybank 1

# Uninstall everything at once
helm uninstall easybank

# Also uninstall third-party components
helm uninstall grafana
helm uninstall tempo
helm uninstall loki
helm uninstall prometheus
helm uninstall kafka
helm uninstall keycloak

# ⚠️ Always manually delete PVCs after uninstall
kubectl get pvc
kubectl delete pvc <pvc-name>
```

### Helm Template Language Cheat Sheet

| Syntax | Description |
|--------|-------------|
| `{{ .Values.key }}` | Read a value from `values.yaml` |
| `{{ .Values.nested.key }}` | Read a nested value |
| `{{- define "name" -}}` | Define a named template |
| `{{ include "name" . }}` | Include/call a named template |
| `{{- if .Values.flag }}` | Conditional block |
| `{{- end }}` | End a define/if block |
| `{{- ... -}}` | Trim surrounding whitespace |

