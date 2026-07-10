# Docker Microservices Practice Guide
### Master Microservices with Spring Boot, Docker & Kubernetes
**EazyBytes | Udemy Course**

> This guide summarizes key concepts from the course and provides structured practice exercises. Work through each section in order, as each builds on the previous.

---

## Table of Contents
1. [Microservices Challenges](#section-1-microservices-challenges)
2. [Containers vs Virtual Machines](#section-2-containers-vs-virtual-machines)
3. [Containers & Docker Definitions](#section-3-containers--docker-definitions)
4. [Docker Architecture](#section-4-docker-architecture)
5. [Docker Installation & Hub](#section-5-docker-installation--hub)
6. [Three Approaches to Docker Images](#section-6-three-approaches-to-docker-images)
7. [Dockerfile Approach — Accounts](#section-7-dockerfile-approach--accounts-lec-46-48)
8. [Building & Running the Accounts Container](#section-8-building--running-the-accounts-container-lec-48-49)
9. [Challenges with Dockerfile & Buildpacks](#section-9-challenges-with-dockerfile--buildpacks-lec-50-51)
10. [Google Jib — Cards](#section-10-google-jib--cards-lec-52)
11. [Comparing All Three Approaches](#section-11-comparing-dockerfile-buildpacks--jib-lec-53)
12. [Pushing to Docker Hub](#section-12-pushing-docker-images-to-docker-hub-lec-54)
13. [Introduction to Docker Compose](#section-13-introduction-to-docker-compose-lec-55)
14. [Dockerfile Instructions Cheat Sheet](#dockerfile-instructions-cheat-sheet)
15. [Common Docker Commands](#common-docker-commands-quick-reference)

---

## Section 1: Microservices Challenges

### Challenge 3: Deployment, Portability & Scalability

Before containerization, deploying hundreds of microservices across different environments was a major pain point.

| Challenge | Problem Statement |
|-----------|-------------------|
| **Deployment** | How do we deploy all the tiny hundreds of microservices with less effort & cost? |
| **Portability** | How do we move our 100s of microservices across environments with less effort, configurations & cost? |
| **Scalability** | How do we scale our applications based on demand on the fly with minimum effort & cost? |

### The Solution: Containerization

Containers offer a self-contained and isolated environment for applications, including all necessary dependencies. By containerizing an application:

- It becomes portable and can run seamlessly in any cloud environment
- Enables unified management of applications regardless of the language or framework used
- Simplifies deployment by packaging code + dependencies together

---

## Section 2: Containers vs Virtual Machines

| Aspect | Description |
|--------|-------------|
| **Virtual Machines** | Each VM has its own Guest OS, Bins/Libs, and App. Managed via a Hypervisor on physical hardware. Heavy and resource-intensive. |
| **Containers** | Containers share the Host OS kernel. Each container has only Bins/Libs + App. Managed by Container Engine (Docker). Lightweight and fast. |
| **Key Advantage** | Containers eliminate redundant Guest OS layers, making them faster to start, smaller in size, and cheaper to run at scale. |

---

## Section 3: Containers & Docker Definitions

### Key Definitions

| Term | Definition |
|------|------------|
| **Container** | A loosely isolated environment that allows us to build and run software packages. These packages include the code and all dependencies to run applications quickly and reliably on any computing environment. |
| **Software Containerization** | An OS virtualization method used to deploy and run containers without using a VM. Containers can run on physical hardware, in the cloud, VMs, and across multiple OSes. |
| **Docker** | An open-source platform that enables developers to automate the deployment, scaling, and management of applications using containerization. |

### How Containerization Works Internally

| Concept | Purpose |
|---------|---------|
| **Namespaces** | Allow creation of isolated environments within the OS. Each container has its own set of namespaces (process, network, mount, IPC, user) — ensuring processes within a container can only interact with resources in their namespace. |
| **Control Groups (cgroups)** | Provide resource management and allocation for containers. Allow administrators to control and limit CPU, memory, disk I/O, and network bandwidth per container. |

> **Note:** Since containerization relies on Linux kernel features, Docker on macOS or Windows installs a lightweight Linux VM in the background. The Docker client runs natively, but the Docker server (daemon) runs inside that Linux VM.

---

## Section 4: Docker Architecture

### Three-Component Architecture

| Component | Responsibilities |
|-----------|-----------------|
| **Docker Client** | Issues commands to Docker Daemon via Docker CLI or Docker Remote API. The interface you interact with (e.g., `docker run`, `docker build`). |
| **Docker Host / Server** | Runs the Docker Daemon which creates and manages images and containers. Maintains local image cache. |
| **Docker Registry** | Stores Docker images. Docker Hub is the public registry. The daemon pulls images from here if not found locally. |

### How a Container Runs (4-Step Flow)

1. Docker Client sends instruction to the Docker Server to run a container
2. Docker Server checks if the image exists locally; if not, searches the registry
3. Docker Server pulls the image from Docker Hub or private registry into local cache
4. Docker Server creates a running container from the local image

---

## Section 5: Docker Installation & Hub

### Windows System Requirements (WSL 2 Backend)

- WSL version 1.1.3.0 or above
- Windows 10/11 64-bit (Home or Pro, version 21H2 or higher)
- Enable WSL 2 feature in Windows settings
- 64-bit processor with SLAT support
- 4GB system RAM minimum
- BIOS-level hardware virtualization enabled

> ⚠️ **Important:** Commercial use of Docker Desktop in larger enterprises (>250 employees OR >$10M annual revenue) requires a paid subscription.

### Verifying Installation

```bash
docker version
```

---

## Section 6: Three Approaches to Docker Images

| Approach | Used For | Description |
|----------|----------|-------------|
| **01 — Dockerfile** | `accounts` microservice | Write a Dockerfile with explicit build instructions. Gives maximum control but requires more manual configuration. |
| **02 — Buildpacks** | `loans` microservice | Tool from Heroku & Pivotal, now hosted by CNCF. No need to write a low-level Dockerfile — it auto-detects your app type. |
| **03 — Google Jib** | `cards` microservice | Open-source Java tool by Google. Builds Docker images for Java applications without requiring Docker to be running. |

---

## Section 7: Dockerfile Approach — Accounts (Lec. 46–48)

### Step 1: Set Up the Project

Open the `section4` folder in IntelliJ IDEA. It contains three Spring Boot microservices: `accounts`, `loans`, and `cards`. The accounts service uses Spring Boot 3.1.1 with Java 17.

**Key `pom.xml` settings for accounts:**

```xml
<groupId>com.eazybytes</groupId>
<artifactId>accounts</artifactId>
<version>0.0.1-SNAPSHOT</version>
<packaging>jar</packaging>
<java.version>17</java.version>
```

The build section uses the `spring-boot-maven-plugin` and excludes Lombok from the final JAR:

```xml
<artifactId>spring-boot-maven-plugin</artifactId>
<excludes>
  <exclude>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
  </exclude>
</excludes>
```

### Step 2: Build the JAR File

```bash
cd section4/accounts
mvn -version            # Verify Maven & Java versions
mvn clean install       # Compiles, tests, and packages the application
```

After a successful build, the `target/` folder is created containing:
- `accounts-0.0.1-SNAPSHOT.jar` — the executable Spring Boot fat JAR
- `accounts-0.0.1-SNAPSHOT.jar.original` — the original pre-repackaged JAR
- `classes/`, `generated-sources/`, `test-classes/`, `maven-archiver/` etc.

Verify the app runs correctly before containerizing:

```bash
mvn spring-boot:run                              # Run via Maven
java -jar target/accounts-0.0.1-SNAPSHOT.jar    # Run the JAR directly
```

> 💡 **Tip:** Always verify the Spring Boot app starts and responds correctly before writing a Dockerfile. This ensures any issues during containerization are Docker-related, not code-related.

### Step 3: Choose a Base Image

> ⚠️ **Important:** The `openjdk` Docker image on Docker Hub is officially **deprecated**. Use alternatives such as `amazoncorretto`, `eclipse-temurin`, `ibm-semeru-runtimes`, `ibmjava`, or `sapmachine` instead.

For this course, the instructor uses `openjdk:17-jdk-slim`. When choosing a base image, consider:

- **Version match** — base image Java version must match your application Java version
- **Size** — slim and alpine variants are smaller; bullseye/bookworm refer to Debian releases
- **Architecture** — check if the image supports your CPU architecture (amd64, arm64)
- **Update frequency** — prefer actively maintained images

**Available openjdk tag variants:**
```
22-slim-bullseye    # Java 22, slim Debian Bullseye
22-slim-bookworm    # Java 22, slim Debian Bookworm
22-slim            # Java 22, slim (latest Debian)
17-jdk-slim        # Java 17, slim JDK (used in this course)
```

### Step 4: Write the Dockerfile

Create a new file named exactly `Dockerfile` (no extension) in the root of the accounts project folder (same level as `pom.xml`).

**Complete Dockerfile for the Accounts microservice:**

```dockerfile
# Start with a base image containing Java runtime
FROM openjdk:17-jdk-slim

# Information around who maintains the image
MAINTAINER eazybytes.com

# Add the application's jar to the image
COPY target/accounts-0.0.1-SNAPSHOT.jar accounts-0.0.1-SNAPSHOT.jar

# Execute the application
ENTRYPOINT ["java", "-jar", "accounts-0.0.1-SNAPSHOT.jar"]
```

**Dockerfile Instruction Reference:**

| Instruction | Purpose |
|-------------|---------|
| `FROM <image>` | Sets the base image. Must be the first instruction. |
| `MAINTAINER <email>` | Metadata — specifies the maintainer (deprecated; prefer `LABEL`). |
| `COPY <src> <dest>` | Copies files from build context into the image filesystem. |
| `ENTRYPOINT ["cmd", "args"]` | Configures the container to run as an executable (exec form preferred). |

> **Why exec form for ENTRYPOINT?** Using `["java", "-jar", "app.jar"]` (exec form) instead of `java -jar app.jar` (shell form) ensures the Java process receives OS signals directly, allowing graceful shutdown.

### Practice Exercises — Section 7

**Exercise 5 — Build the Accounts JAR**
1. Clone the course repository and navigate to `section4/accounts`
2. Run `mvn -version` to confirm Java 17 and Maven 3.x are installed
3. Run `mvn clean install` and verify the BUILD SUCCESS message
4. Open the `target/` folder and identify the `.jar` and `.jar.original` files
5. Run the JAR: `java -jar target/accounts-0.0.1-SNAPSHOT.jar`
6. Confirm the app starts on port 8080

**Exercise 6 — Write and Analyze the Dockerfile**
1. Create a new Dockerfile in the accounts root folder
2. Write the four instructions: FROM, MAINTAINER, COPY, ENTRYPOINT
3. For each instruction, write a one-sentence explanation of what it does
4. Research `eclipse-temurin:17-jdk-jammy` as an alternative base image
5. Answer: Why is exec form of ENTRYPOINT preferred over shell form?

---

## Section 8: Building & Running the Accounts Container (Lec. 48–49)

### Step 5: Build the Docker Image

```bash
docker build . -t eazybytes/accounts:s4
```

| Part | Meaning |
|------|---------|
| `docker build` | Instructs Docker daemon to build an image |
| `.` | Build context — the current directory |
| `-t eazybytes/accounts:s4` | Tags the image: repo=eazybytes, name=accounts, tag=s4 |

> 💡 **Tag naming convention:** Use meaningful tags to track versions — `:s4` for section 4, `:latest` for most recent, `:1.0.0` for semantic versioning.

### Step 6: Verify the Image

```bash
docker images                    # List all local images
docker inspect image <IMAGE_ID>  # Detailed JSON metadata
```

The inspect output shows: `Author: eazybytes.com`, `JAVA_HOME`, `JAVA_VERSION`, and `Entrypoint: [java, -jar, accounts-0.0.1-SNAPSHOT.jar]`.

### Step 7: Port Mapping

By default, containers are connected to an isolated Docker network. Use `-p` to expose ports:

> **Syntax:** `-p <external_port>:<container_port>`  
> Example: `-p 8081:8080` → access via `localhost:8081`, traffic forwarded to port `8080` inside container.

| Scenario | Command |
|----------|---------|
| Same port | `docker run -p 8080:8080 eazybytes/accounts:s4` |
| Different external port | `docker run -p 8081:8080 eazybytes/accounts:s4` |
| Detached (background) mode | `docker run -d -p 8080:8080 eazybytes/accounts:s4` |

### Step 8: Container Lifecycle

```bash
# Foreground mode
docker run -p 8080:8080 eazybytes/accounts:s4

# Detached/background mode (returns container ID)
docker run -d -p 8080:8080 eazybytes/accounts:s4
```

**Key lifecycle commands:**

| Command | What It Does |
|---------|-------------|
| `docker ps` | Lists RUNNING containers |
| `docker ps -a` | Lists ALL containers including stopped |
| `docker stop <id>` | Gracefully stops a running container |
| `docker start <id>` | Restarts a stopped container (reuses same container) |
| `docker run ... (again)` | Creates a BRAND NEW container |
| `docker rm <id>` | Permanently deletes a stopped container |
| `docker logs <id>` | View stdout/stderr logs |

> **Key insight:** `docker stop` then `docker start` reuses the same container. `docker run` always creates a new container with a new random name (e.g., `strange_mcclintock`, `exciting_benz`).

**Running multiple instances (demonstrating scalability):**

```bash
docker run -d -p 8080:8080 eazybytes/accounts:s4   # Instance 1
docker run -d -p 8081:8080 eazybytes/accounts:s4   # Instance 2
```

### Practice Exercises — Section 8

**Exercise 8 — Build, Inspect & Run**
1. Build: `docker build . -t eazybytes/accounts:s4`
2. Run `docker images` and note the image size
3. Run `docker inspect image <ID>` and find: Author, Entrypoint, and `JAVA_VERSION`
4. Run the container in foreground mode and observe Spring Boot startup logs
5. Re-run in detached mode and confirm it starts without blocking the terminal

**Exercise 9 — Port Mapping**
1. Run the container on port 8080 and test: `curl http://localhost:8080/actuator/health`
2. Run a SECOND instance on port 8081 and test it independently
3. Run `docker ps` and observe both containers running simultaneously
4. Stop one instance and verify with `docker ps`

---

## Section 9: Challenges with Dockerfile & Buildpacks (Lec. 50–51)

### Lecture 50: Challenges with the Dockerfile Approach

| Challenge | Details |
|-----------|---------|
| **No layer caching optimization** | The naive approach copies the entire fat JAR as a single layer. Any code change forces a full re-upload of the JAR layer, even if only one class changed. |
| **MAINTAINER is deprecated** | Modern Dockerfiles should use `LABEL maintainer=` instead. |
| **Security concerns** | Running as root user inside container unless explicitly configured otherwise. |
| **Manual version management** | Must manually update the JAR filename in the Dockerfile whenever the version changes. |

### Better Dockerfile Pattern: Layered JAR

```dockerfile
# Multi-stage Dockerfile with layered JAR
FROM openjdk:17-jdk-slim as build
WORKDIR /workspace/app
COPY target/*.jar app.jar
RUN java -Djarmode=layertools -jar app.jar extract

FROM openjdk:17-jdk-slim
WORKDIR /application
COPY --from=build /workspace/app/dependencies/ ./
COPY --from=build /workspace/app/spring-boot-loader/ ./
COPY --from=build /workspace/app/snapshot-dependencies/ ./
COPY --from=build /workspace/app/application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

> The layered JAR approach splits your app into: **dependencies** (rarely change), **spring-boot-loader** (rarely changes), **snapshot-dependencies** (occasionally change), and **application** (your code, changes often). Docker caches unchanged layers.

### Lecture 51: Buildpacks for Loans

Spring Boot 2.3+ has native Buildpacks support built into the `spring-boot-maven-plugin`.

**Configure `pom.xml` for Buildpacks — Step 1 (simple):**

```xml
<configuration>
  <image>
    <name>eazybytes/loans</name>
  </image>
  <excludes>...</excludes>
</configuration>
```

**Step 2: Use Maven variable (recommended):**

```xml
<image>
  <name>eazybytes/${project.artifactId}:s4</name>
</image>
```

**Build command:**

```bash
cd section4/loans
mvn spring-boot:build-image
```

| Aspect | Dockerfile vs Buildpacks |
|--------|--------------------------|
| **Setup** | Manual Dockerfile required | Zero configuration |
| **Security** | Configure manually | Automatic best practices |
| **Layer optimization** | Manual | Automatic by default |
| **Command** | `docker build . -t <tag>` | `mvn spring-boot:build-image` |

**Image size results:**

| Image | Size | Notes |
|-------|------|-------|
| eazybytes/accounts (Dockerfile) | 456MB | openjdk:17-jdk-slim base |
| eazybytes/loans (Buildpacks) | 311MB | Buildpacks-optimized JRE runtime |
| paketobuildpacks/run | 87.1MB | Buildpacks run image |
| paketobuildpacks/builder | 1.31GB | Builder used during build only |

> **Key observation:** Buildpacks-generated loans image (311MB) is smaller than Dockerfile accounts (456MB). Buildpacks use a JRE-only runtime rather than the full JDK.

**Running the Loans container:**

```bash
docker run -d -p 8090:8090 eazybytes/loans:s4
```

> **Apple Silicon / ARM warning:** You may see: `WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8)`. The container still runs via Rosetta emulation.

### Practice Exercises — Section 9

**Exercise 10 — Configure and Build the Loans Image**
1. Open `loans/pom.xml` and add `<image><name>` configuration
2. Try hardcoded: `eazybytes/loans`, then update to `eazybytes/${project.artifactId}:s4`
3. Run `mvn spring-boot:build-image` and observe the build phases
4. Run `docker images` and note the size difference vs accounts

**Exercise 11 — Run and Compare**
1. Start: `docker run -d -p 8090:8090 eazybytes/loans:s4`
2. Run `docker ps` to confirm both accounts (8080) and loans (8090) running
3. Open Docker Desktop and explore: Logs, Inspect, Terminal tabs
4. Answer: Why is the Buildpacks image smaller? (Hint: JDK vs JRE)

---

## Section 10: Google Jib — Cards (Lec. 52)

Google Jib builds optimized Docker images without requiring Docker to be installed or running locally, and without a Dockerfile. It builds directly from Maven/Gradle.

> ⚠️ **Bug Notice:** Jib version 3.4.3 has a known bug preventing image generation on some systems. Use version **3.3.2** as specified in the course GitHub repo.

### Configure `pom.xml` for Jib

Add the `jib-maven-plugin` as a second plugin in `cards/pom.xml`:

```xml
<plugin>
  <groupId>com.google.cloud.tools</groupId>
  <artifactId>jib-maven-plugin</artifactId>
  <version>3.3.2</version>
  <configuration>
    <to>
      <image>eazybytes/${project.artifactId}:s4</image>
    </to>
  </configuration>
</plugin>
```

### Build Commands

| Command | Description |
|---------|-------------|
| `mvn compile jib:build` | Builds and pushes directly to a remote registry. Does NOT require Docker locally. |
| `mvn compile jib:dockerBuild` | Builds image into local Docker daemon. Requires Docker running. Used in this course. |

```bash
cd section4/cards
mvn compile jib:dockerBuild
```

**What Jib does:**
- Pulls base image `eclipse-temurin:17-jre` from Docker Hub
- Sets entrypoint to `[java, -cp, @/app/jib-classpath-file, com.eazybytes.cards.CardsApplication]`
- Builds image as `eazybytes/cards:s4`
- Completes in ~11 seconds (much faster than Buildpacks!)

> 💡 **Why is Jib fast?** Jib builds in separate layers — dependencies (rarely change), resources, and classes (your code). Only changed layers are rebuilt and uploaded on subsequent builds.

### Final Size Comparison — All Three Images

| Image (Method) | Size | Notes |
|----------------|------|-------|
| eazybytes/accounts (Dockerfile) | 456MB | openjdk:17-jdk-slim, full JDK |
| eazybytes/loans (Buildpacks) | 311MB | Buildpacks-optimized JRE |
| eazybytes/cards (Google Jib) | 322MB | eclipse-temurin:17-jre |

### Practice Exercises — Section 10

**Exercise 12 — Build the Cards Image with Jib**
1. Open `cards/pom.xml` and add the `jib-maven-plugin` with version 3.3.2
2. Configure `<to><image>` with the Maven variable pattern
3. Run: `mvn compile jib:dockerBuild`
4. Observe: note the base image, entrypoint, and build time in the output
5. Run `docker images` and compare all three microservice image sizes
6. Answer: Which approach produced the smallest image? Why?

---

## Section 11: Comparing Dockerfile, Buildpacks & Jib (Lec. 53)

Official feature matrix from the Cloud Native Buildpacks website:

| Feature | Buildpacks | Dockerfile | Jib |
|---------|-----------|------------|-----|
| Advanced Caching | ✅ Yes | ❌ No | ❌ No |
| Auto-detection of app type | ✅ Yes | ❌ No | ✅ Yes |
| Bill-of-Materials (SBOM) | ✅ Yes | ❌ No | ❌ No |
| Modular / Pluggable | ✅ Yes | ❌ No | N/A |
| Multi-language support | ✅ Yes | ✅ Yes | ❌ No (Java only) |
| Multi-process containers | ✅ Yes | ❌ No | ❌ No |
| Minimal app image | ✅ Yes | ✅ Yes* | ✅ Yes |
| Rebasing (fast security patches) | ✅ Yes | ❌ No | ❌ No |
| Reproducibility | ✅ Yes | ❌ No | ✅ Yes |
| Reusability | ✅ Yes | ❌ No | N/A |

> **Key takeaway:** Buildpacks has the most enterprise-grade features. Jib is excellent for Java-only projects needing reproducibility and speed. Dockerfile gives maximum flexibility but requires the most manual work.

### When to Use Each Approach

- **Dockerfile** — Full control, multi-stage builds, non-Java services, or custom base images
- **Buildpacks** (`mvn spring-boot:build-image`) — Zero-config, enterprise security features, SBOM generation
- **Google Jib** (`mvn compile jib:dockerBuild`) — Fast Java-specific builds without Docker running, or direct push to registry from CI/CD

---

## Section 12: Pushing Docker Images to Docker Hub (Lec. 54)

### Pre-requisites: Login

```bash
docker login    # Login via CLI (if not using Docker Desktop)
```

### Push Command

```bash
# Push the accounts image
docker image push docker.io/eazybytes/accounts:s4

# Or without registry prefix (Docker Hub is default)
docker image push eazybytes/accounts:s4
```

**What happens during a push:** Docker pushes each layer separately. Layers already on Docker Hub (like the base openjdk layers) are reported as `Mounted from library/openjdk` — not re-uploaded. Only your custom application layer (~53.9MB) is actually pushed. A digest confirms success:

```
2b20dda18447: Pushing [===========] 53.9MB
c82e5bf37b8a: Mounted from library/openjdk
2f263e87cb11: Mounted from library/openjdk
s4: digest: sha256:c731e292e2307... size: 1165
```

### Push All Three Images

```bash
docker image push eazybytes/accounts:s4
docker image push eazybytes/loans:s4
docker image push eazybytes/cards:s4
```

### Verify on Docker Hub

After pushing, navigate to `hub.docker.com`. Under your account you'll see:
- `eazybytes/cards` — Last pushed: 2 minutes ago
- `eazybytes/loans` — Last pushed: 5 minutes ago
- `eazybytes/accounts` — Last pushed: 9 minutes ago

**Docker Hub repository tabs:**

| Tab | Contents |
|-----|---------|
| **General** | Image description, Docker push command, tag list with OS/arch and compressed size |
| **Tags** | All tags with digest, OS/ARCH (linux/amd64), compressed size (e.g., cards:s4 = 138MB compressed) |
| **Settings** | Docker Scout vulnerability scanning, visibility (Public/Private). Free accounts get 1 private repo. |

> **Compressed vs uncompressed:** eazybytes/cards:s4 is 322MB locally but only 138MB compressed on Docker Hub. Docker compresses layers during push.

### Pull from Docker Hub

```bash
docker pull eazybytes/cards:s4
```

### Practice Exercises — Section 12

**Exercise 13 — Push All Images to Docker Hub**
1. Create a Docker Hub account at `hub.docker.com` if you don't have one
2. Log in via Docker Desktop
3. Push all three images: accounts, loans, and cards
4. Verify all three repositories appear on Docker Hub
5. Click Tags tab — note compressed size vs local size
6. On a different machine, pull and run: `docker pull eazybytes/cards:s4`

---

## Section 13: Introduction to Docker Compose (Lec. 55)

### What is Docker Compose?

Docker Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a **YAML file** to configure all your application services. Then, with a single command, you create and start all services from that configuration.

> **Version note:** From July 2023, Compose V1 stopped receiving updates and is no longer available in new Docker Desktop releases. **Compose V2** is included with all currently supported Docker Desktop. The command is now `docker compose` (no hyphen, was `docker-compose`).

### Why Docker Compose?

Without Compose, starting all three microservices requires three separate `docker run` commands with the correct ports, networks, and environment variables. With Compose, you define everything once in `docker-compose.yml` and start everything with one command.

### Key Features

- **Multiple isolated environments on a single host** — different projects don't interfere
- **Preserves volume data when containers are created** — persistent data across restarts
- **Only recreate containers that have changed** — efficient incremental updates
- **Supports variables and moving between environments** — dev, staging, prod configs

### Docker Compose Lifecycle Commands

| Command | What it Does |
|---------|-------------|
| `docker compose up` | Start, build, and attach to containers |
| `docker compose up -d` | Start all services in detached mode |
| `docker compose down` | Stop and remove containers and networks |
| `docker compose ps` | List containers managed by this Compose file |
| `docker compose logs` | View output from all containers |
| `docker compose start` | Start existing containers |
| `docker compose stop` | Stop running containers without removing them |
| `docker compose build` | Build or rebuild service images |

### Practice Exercises — Section 13

**Exercise 14 — Explore Docker Compose**
1. Run: `docker compose version` to confirm Compose V2 is installed
2. Read the official docs at `docs.docker.com/compose`
3. Identify: What are the 4 key features that make Compose effective?
4. Think through: What would a `docker-compose.yml` look like for accounts (8080), loans (8090), and cards (9000)?

---

## Dockerfile Instructions Cheat Sheet

| Instruction | Syntax & Notes |
|-------------|----------------|
| `FROM` | `FROM <image>[:<tag>]` — Must be first instruction. Sets base image. |
| `MAINTAINER` | `MAINTAINER <name/email>` — Deprecated; use `LABEL` instead. |
| `LABEL` | `LABEL key=value` — Adds metadata as key-value pairs. |
| `RUN` | `RUN <command>` — Executes a command in a new layer during build. |
| `COPY` | `COPY <src> <dest>` — Copies files from build context into image. |
| `ADD` | `ADD <src> <dest>` — Like COPY but also supports URLs and tar auto-extraction. |
| `WORKDIR` | `WORKDIR /path` — Sets working directory for subsequent instructions. |
| `ENV` | `ENV KEY=value` — Sets environment variables in the image. |
| `EXPOSE` | `EXPOSE <port>` — Documents which port the container listens on. |
| `ENTRYPOINT` | `ENTRYPOINT ["cmd", "arg"]` — Sets the container startup executable. |
| `CMD` | `CMD ["arg"]` — Default arguments for ENTRYPOINT, or default command. |
| `VOLUME` | `VOLUME /path` — Creates a mount point for external volumes. |
| `ARG` | `ARG name[=default]` — Build-time variable (not persisted in image). |

---

## Common Docker Commands Quick Reference

| Command | Purpose |
|---------|---------|
| `docker version` | Show client and server version info |
| `docker pull <image>` | Download an image from Docker Hub |
| `docker images` | List all locally available images |
| `docker run -d -p <host>:<container> <image>` | Run a container in detached mode with port mapping |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (including stopped) |
| `docker stop <name/id>` | Stop a running container |
| `docker start <name/id>` | Start a stopped container |
| `docker rm <name/id>` | Remove a stopped container |
| `docker rmi <image>` | Remove a local image |
| `docker logs <name/id>` | View container logs |
| `docker exec -it <name/id> bash` | Open interactive terminal inside container |
| `docker build -t <tag> .` | Build image from Dockerfile in current directory |
| `docker image push <tag>` | Push image to Docker Hub |
| `docker inspect image <id>` | Show detailed image metadata as JSON |
| `docker compose up -d` | Start all Compose services in background |
| `docker compose down` | Stop and remove all Compose containers |

---

## Notes

_Use this space to record your own observations, errors encountered, and solutions found during the hands-on exercises._

---
*Course: Master Microservices with Spring Boot, Docker & Kubernetes | Platform: Udemy | Instructor: EazyBytes*

---

## Section 14: Writing the docker-compose.yml File (Lec. 55–56)

### The Problem It Solves

With three microservices, you need three separate `docker run` commands — and that's just three. With 100 microservices, issuing `docker run` manually 100 times is not viable. Docker Compose solves this by letting you define everything in a single YAML file and start everything with **one command**.

### Creating the docker-compose.yml

Create a file named `docker-compose.yml` inside the `accounts/` project folder (can be placed anywhere, but committing it alongside a microservice is a common pattern).

> ⚠️ **YAML indentation is critical.** Every space matters. Tab characters are not allowed — use spaces only. Incorrect indentation will cause errors.

### Complete docker-compose.yml

```yaml
services:
  accounts:
    image: "eazybytes/accounts:s4"
    container_name: accounts-ms
    ports:
      - "8080:8080"
    deploy:
      resources:
        limits:
          memory: 700m
    networks:
      - eazybank

  loans:
    image: "eazybytes/loans:s4"
    container_name: loans-ms
    ports:
      - "8090:8090"
    deploy:
      resources:
        limits:
          memory: 700m
    networks:
      - eazybank

  cards:
    image: "eazybytes/cards:s4"
    container_name: cards-ms
    ports:
      - "9000:9000"
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

### YAML Structure Explained

| Key | Purpose |
|-----|---------|
| `services:` | Root element — all service definitions live here |
| `accounts:` / `loans:` / `cards:` | Service names (used internally by Compose) |
| `image:` | Docker image to use for this service |
| `container_name:` | Explicit name for the container (avoids random names like `angry_cannon`) |
| `ports:` | Port mapping list. Hyphen `-` indicates an array element |
| `deploy.resources.limits.memory:` | Max memory this container can use (e.g., `700m` = 700 MB) |
| `networks:` | Tags the service to a named network |
| `networks: eazybank: driver: bridge` | Root-level network definition — creates the shared bridge network |

### Why Define a Network?

By default, each container gets its own isolated network and **cannot communicate** with other containers. By assigning all three services to the same `eazybank` bridge network, Docker creates a bridge between them, enabling inter-service communication using container names as hostnames.

> 💡 **Tip:** Give every container an explicit `container_name`. Without it, Docker generates random names (e.g., `strange_mcclintock`) making it impossible to identify which service a container belongs to at a glance.

---

## Section 15: Running All Containers with Docker Compose (Lec. 56)

### Starting All Services

Run the following command **from the directory where `docker-compose.yml` is located** (i.e., the `accounts/` folder):

```bash
# Must be run from the folder containing docker-compose.yml
docker compose up -d
```

If you run `docker compose up` from a folder that does not contain a `docker-compose.yml`, you will get: `no configuration file provided: not found`.

**What happens:**
- Docker creates all three containers from their images
- Each container is assigned to the `eazybank` bridge network
- All three start in detached (background) mode — your terminal stays free
- Verify with `docker ps` — all three containers running at ports 8080, 8090, and 9000

### In Docker Desktop

After running `docker compose up`, Docker Desktop shows a parent group named after the folder (e.g., `accounts`) containing all three containers: `accounts-ms`, `loans-ms`, `cards-ms`.

### Stopping All Services

```bash
docker compose down    # Stops AND removes all containers (recommended)
docker compose stop    # Stops containers but keeps them (does NOT remove)
```

> **Best practice:** Use `docker compose down` rather than `docker compose stop`. Removing containers when you're done frees memory and disk space on your local system.

---

## Section 16: Docker Compose Commands Deep Dive (Lec. 57)

### The Four Core Commands Compared

| Command | Creates containers? | Removes containers? | Use when... |
|---------|--------------------|--------------------|-------------|
| `docker compose up -d` | ✅ Yes (if not exist) | ❌ No | Starting from scratch, or resuming stopped containers |
| `docker compose down` | ❌ No | ✅ Yes | Done for the day, want clean state |
| `docker compose stop` | ❌ No | ❌ No | Temporarily pausing, want to resume later |
| `docker compose start` | ❌ No | ❌ No | Resuming previously stopped (not removed) containers |

### Key Behavioral Differences

**`docker compose up` is smart:** If containers with the correct names already exist (e.g., from a previous `docker compose stop`), it reuses them instead of creating new ones from scratch.

**`docker compose start` requires existing containers:** If you ran `docker compose down` (which removes containers), then `docker compose start` will fail with `no service with name cards/loans/accounts`.

### Typical Workflow

```bash
# Day 1 — First time
docker compose up -d         # Creates containers from scratch, starts them

# End of day
docker compose down          # Stops and removes all containers

# Day 2 — Starting again
docker compose up -d         # Creates fresh containers again
```

### Practice Exercises — Sections 14–16

**Exercise 15 — Write and Run docker-compose.yml**
1. Create `docker-compose.yml` in your accounts folder with all three services
2. Run `docker compose up -d` from that folder
3. Verify with `docker ps` — confirm all three containers running
4. Test each service: accounts on 8080, loans on 8090, cards on 9000
5. Run `docker compose down` and confirm containers are removed

**Exercise 16 — Compose Commands Comparison**
1. Run `docker compose up -d`
2. Run `docker compose stop` — confirm containers exist but are stopped (`docker ps -a`)
3. Run `docker compose start` — confirm containers resume
4. Run `docker compose down` — confirm containers are fully removed
5. Try `docker compose start` again — observe the failure message
6. Answer: What is the difference between `docker compose up` and `docker compose start`?

---

## Section 17: Deep Dive on Docker Commands (Lec. 58)

### Complete Docker Command Reference

| # | Command | Purpose |
|---|---------|---------|
| 1 | `docker images` | List all local Docker images |
| 2 | `docker image inspect <id>` | Show detailed metadata of a specific image (pass first 3–4 chars of ID) |
| 3 | `docker image rm <id>` | Remove one or more images (separate multiple IDs with space) |
| 4 | `docker build -t <tag> .` | Build a Docker image from Dockerfile in current directory |
| 5 | `docker run -d -p <host>:<container> <image>` | Create and start a container with port mapping |
| 6 | `docker ps` | List running containers |
| 7 | `docker ps -a` | List all containers including stopped |
| 8 | `docker container start <id>` | Start a stopped container (can pass multiple IDs separated by space) |
| 9 | `docker container pause <id>` | Pause a container — stops accepting traffic temporarily |
| 10 | `docker container unpause <id>` | Resume a paused container |
| 11 | `docker container stop <id>` | Gracefully stop — gives ~5 seconds to close resources (DB connections, file systems) |
| 12 | `docker container kill <id>` | Instantly kill a container — no graceful shutdown |
| 13 | `docker container restart <id>` | Restart a running or stopped container |
| 14 | `docker container inspect <id>` | Show detailed JSON metadata of a container |
| 15 | `docker container logs <id>` | View all logs of a container |
| 16 | `docker container logs -f <id>` | Follow/stream logs in real time (`-f` = follow) |
| 17 | `docker rm <id>` | Remove a stopped container (multiple IDs separated by space) |
| 18 | `docker container prune` | Remove ALL stopped containers at once (no need to specify IDs) |
| 19 | `docker image push <tag>` | Push image to Docker Hub |
| 20 | `docker pull <image>` | Pull image from Docker Hub |
| 21 | `docker image prune` | Remove all **unused** images (images with no associated containers) |
| 22 | `docker container stats` | Live CPU, memory, and I/O usage stats for all containers |
| 23 | `docker system prune` | Remove stopped containers + unused images + unused networks + volumes + cache |
| 24 | `docker rmi <image-id>` | Alternative way to remove an image (`rmi` = remove image) |
| 25 | `docker login -u <username>` | Log in to Docker Hub from CLI (prompts for password) |
| 26 | `docker logout` | Log out from Docker Hub |
| 27 | `docker history <image>` | Show all intermediate layers and commands used to build an image — useful for debugging |
| 28 | `docker exec -it <id> sh` | Open an interactive shell inside a running container |
| 29 | `docker compose up -d` | Create and start all Compose services in background |
| 30 | `docker compose down` | Stop and remove all Compose containers |
| 31 | `docker compose stop` | Stop containers without removing them |
| 32 | `docker compose start` | Start existing (not removed) containers |

### Key Distinctions to Remember

**`stop` vs `kill`:**
- `docker container stop` → sends SIGTERM, waits ~5 seconds for graceful shutdown (closes DB connections, file handles), then sends SIGKILL
- `docker container kill` → sends SIGKILL immediately — no cleanup time

**`prune` commands:**
- `docker container prune` — removes all **stopped** containers
- `docker image prune` — removes all **unused** images (no containers, running or stopped, reference them)
- `docker system prune` — removes everything unused in one shot (containers + images + networks + volumes + build cache)

**Short IDs:** You never need to type the full 64-character container or image ID. Docker accepts the first 3–4 characters and resolves the full ID automatically.

**`docker history <image>`:** Reveals all the layers Buildpacks or Jib generated behind the scenes. Extremely useful when debugging why an image is larger than expected.

---

## Section 18: Docker Extensions & Logs Explorer (Lec. 59)

### Docker Extensions Overview

Docker Desktop has a marketplace of **thousands of extensions** that extend its functionality. Whenever you encounter a problem or a repetitive task, always check if an extension already solves it.

**How to access:** Docker Desktop → Extensions → Add Extensions → search for what you need.

### Logs Explorer Extension

Published officially by Docker. Provides a unified, color-coded log view across **all running and stopped containers** in one panel.

**Installing:**
1. Open Docker Desktop → Add Extensions
2. Search for "Logs Explorer"
3. Click Install

**Features:**

| Feature | Details |
|---------|---------|
| **Color-coded logs** | Each container gets its own color (e.g., accounts = blue, loans = red, cards = green) |
| **Container filter** | Select a specific container to see only its logs |
| **Status filter** | Filter by running containers only, or include stopped containers |
| **Log type filter** | Filter by stdout or stderr |
| **Search** | Full-text search across all container logs |

> **Recommendation:** Install Logs Explorer if you're running multiple microservices. Instead of clicking into each container individually in Docker Desktop, you can see all logs in one view with clear color separation.

---

## Section 19: Section 4 Summary & Key Takeaways (Lec. 60)

### What We Built

By the end of Section 4, all three microservices are containerized and pushed to Docker Hub:

| Microservice | Image | Method | Port | Size |
|---|---|---|---|---|
| accounts | `eazybytes/accounts:s4` | Dockerfile | 8080 | 456MB |
| loans | `eazybytes/loans:s4` | Buildpacks | 8090 | 311MB |
| cards | `eazybytes/cards:s4` | Google Jib | 9000 | 322MB |

All three are started with a single command: `docker compose up -d`

### The "It Works on My Machine" Problem — Solved

Before Docker, developers would say "it works on my machine" when code worked locally but failed in dev/QA/production. Docker solves this because:

- The Docker image packages **code + all dependencies** together
- The same image runs identically across **all environments** (local, dev, QA, production)
- No more "please install Java version X, set this env var, configure this server..."

### Core Concepts Mastered

- **Containers vs VMs** — shared kernel, lightweight, faster startup
- **Docker Architecture** — Client → Host/Daemon → Registry
- **Three image-building approaches** — Dockerfile, Buildpacks, Google Jib — with trade-offs
- **Port mapping** — `-p <external>:<container>` for network access
- **Container lifecycle** — run, stop, start, kill, remove, prune
- **Docker Compose** — single YAML file, one command to rule them all
- **Docker Hub** — push local images, pull from anywhere in the world
- **Docker Extensions** — Logs Explorer for unified multi-container log monitoring

---

## Updated Common Docker Commands Quick Reference

| Command | Purpose |
|---------|---------|
| `docker version` | Show client and server version info |
| `docker pull <image>` | Download an image from Docker Hub |
| `docker images` | List all locally available images |
| `docker image inspect <id>` | Detailed image metadata as JSON |
| `docker history <image>` | Show build layers and commands used |
| `docker build -t <tag> .` | Build image from Dockerfile in current directory |
| `docker run -d -p <host>:<container> <image>` | Run container in detached mode with port mapping |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers including stopped |
| `docker container stop <id>` | Graceful stop (5s cleanup window) |
| `docker container kill <id>` | Instant stop (no cleanup) |
| `docker container pause/unpause <id>` | Pause/resume container traffic |
| `docker container restart <id>` | Restart a container |
| `docker container start <id>` | Start a stopped container |
| `docker container logs <id>` | View container logs |
| `docker container logs -f <id>` | Follow/stream logs in real time |
| `docker container inspect <id>` | Detailed container metadata as JSON |
| `docker container stats` | Live CPU/memory/IO usage for all containers |
| `docker rm <id>` | Remove a stopped container |
| `docker container prune` | Remove ALL stopped containers |
| `docker rmi <id>` / `docker image rm <id>` | Remove a local image |
| `docker image prune` | Remove all unused images |
| `docker system prune` | Remove stopped containers + unused images + networks + cache |
| `docker image push <tag>` | Push image to Docker Hub |
| `docker exec -it <id> sh` | Open interactive shell inside running container |
| `docker login -u <username>` | Log in to Docker Hub from CLI |
| `docker logout` | Log out from Docker Hub |
| `docker compose up -d` | Create and start all Compose services (background) |
| `docker compose down` | Stop and remove all Compose containers |
| `docker compose stop` | Stop containers without removing |
| `docker compose start` | Start existing (non-removed) containers |
| `docker compose ps` | List Compose-managed containers |
| `docker compose logs` | View logs from all Compose services |
| `docker compose version` | Confirm Compose V2 is installed |
| `mvn spring-boot:build-image` | Build Docker image via Buildpacks (no Dockerfile) |
| `mvn compile jib:dockerBuild` | Build Docker image via Google Jib (local daemon) |

---

*Course: Master Microservices with Spring Boot, Docker & Kubernetes | Platform: Udemy | Instructor: EazyBytes*
