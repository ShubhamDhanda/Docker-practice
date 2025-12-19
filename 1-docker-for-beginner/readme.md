# Docker Crash Course – Revision Notes

> Personal notes based on the **“Docker Crash Course for Absolute Beginners”** video. :contentReference[oaicite:0]{index=0}  
> Goal: Quickly refresh core Docker concepts, commands, and mental models.

---

## 1. What Is Docker?

- Docker is a **containerization / virtualization tool** that makes **developing and deploying applications easier**.
- It packages:
  - Application code
  - Libraries & dependencies
  - Runtime (Node, Java, etc.)
  - Environment/config (env vars, files, dirs)
- All of that lives inside a **container**, so it runs the same way on:
  - Your laptop
  - Staging server
  - Production

**Key benefit:** “It works on my machine” becomes less of a problem, because you package *the app + its environment* together.

---

## 2. Before Docker – What Was Painful?

### Local development without containers

- Each developer had to **manually install** every dependency:
  - Databases (PostgreSQL, MongoDB)
  - Redis, message brokers, etc.
- Problems:
  - Different OS → different install steps (macOS vs Windows vs Linux)
  - Lots of install steps → plenty of chances to mess up
  - Complex apps → 5–10+ services to install and configure
  - **Version conflicts**:
    - Need two different versions of the same DB/tool on one machine? Very hard.

### Deployment without containers

- Dev team shipped:
  - App artifact (e.g. `.jar`, Node build, etc.)
  - Long **textual instructions** for ops:
    - How to install runtime
    - Which libraries
    - How to configure DBs and services
- Ops team:
  - Installed everything directly on the server OS.
  - Dealt with:
    - Dependency conflicts
    - Missing/unclear steps
    - Back-and-forth with devs when something breaks

**Result:** Setup & deployment was error-prone, slow, and inconsistent.

---

## 3. How Docker Fixes This

### For developers

- No need to install services directly on your OS.
- You can run services as **containers**:
  - e.g. Postgres, Redis, MongoDB, Nginx, etc.
- One standard way to run *any* service:
  ```bash
  docker run ...
    ```

* Works the same on macOS, Windows, Linux (with Docker Desktop).
* Easy to:

  * Spin up many services
  * Run different versions of the same service side by side
  * Tear them down cleanly

### For operations / deployment

* Dev team ships an **image** that already contains:

  * The app
  * Runtime
  * Dependencies
  * Configuration
* Ops team doesn’t manually install those pieces on the server:

  * They just run:

    ```bash
    docker run <image:tag>
    ```
* Less configuration drift and fewer “works in dev but not on server” issues.

---

## 4. Docker vs Virtual Machines (VMs)

### OS internals refresher

* An OS has:

  * **Kernel (Main OS -Linux, Windows, Mac)** → talks to hardware (CPU, RAM, disk)
  * **User-space / application layer** → apps & tools running on top of kernel

### Virtual Machines

* A VM **virtualizes a full OS**:

  * Its **own kernel**
  * Its **own application layer**
* VM image size: often **GBs**
* Startup time: **seconds to minutes**
* Flexibility: can run **Linux VM on Windows host**, etc.

### Docker Containers

* Docker virtualizes only the **application layer**:

  * Container has:

    * Minimal OS user-space
    * App + deps
  * It **reuses the host kernel**.
* Image size: usually **MBs**, much smaller.
* Startup time: **milliseconds**.
* Tradeoff:

  * Linux containers expect a **Linux kernel**.
  * On Windows/macOS, Docker Desktop uses a **lightweight Linux VM (hypervisor)** under the hood to provide that kernel.

---

## 5. Installing Docker (Docker Desktop)

When you install **Docker Desktop** (Windows/macOS):

You get:

* **Docker Engine** – the core daemon/service that runs containers.
* **Docker CLI** – `docker` command line tool.
* **Docker Desktop GUI** – graphical client to:

  * See containers & images
  * Start/stop containers
  * Inspect logs and basic settings

On Linux, you typically install Docker Engine + CLI directly without “Docker Desktop” GUI (depending on distro).

---

## 6. Core Concepts: Images, Containers, Registries, Tags

### Image

* A **read-only package** (template) that contains:

  * App code
  * Runtime (e.g. Node, Java, Python)
  * Dependencies
  * Environment config
  * Base OS user-space (often a minimal Linux like Alpine)
* Think: “Installable artifact” / blueprint.

### Container

* A **running instance of an image**.
* You can run **multiple containers** from the same image.
* Container = process + isolated filesystem + networking etc.

### Registry (public & private)

* **Image storage**.
* Public:

  * **Docker Hub** – largest public registry.
  * Has many **official images** (Redis, Nginx, Postgres, Node, etc.).
* Private:

  * Cloud providers: AWS ECR, GCP Artifact Registry, Azure Container Registry.
  * Self-hosted: Nexus, private Docker Hub repos, etc.

### Repository & Tags

* **Registry** = service.
* **Repository** = logical collection of image versions for *one app*.

  * Example: `library/nginx`
* **Tags** = image versions:

  * e.g. `nginx:1.23`, `nginx:1.22-alpine`
  * Special tag: `latest` → the newest pushed version

    * Best practice: **pin a specific version**, don’t rely on `latest`.

---

## 7. Working with Public Images (Docker Hub)

### 7.1 Pulling images

Example: Nginx web server

1. Find image on Docker Hub (e.g. `nginx`), pick a version (tag), e.g. `1.23`.

2. Pull it:

   ```bash
   docker pull nginx:1.23
   ```

3. See local images:

   ```bash
   docker images
   ```

If you omit the tag:

```bash
docker pull nginx
```

* Docker will pull the `latest` tag.

### 7.2 Running containers

Basic `run`:

```bash
docker run nginx:1.23
```

* Starts container in the **foreground** – logs show in your terminal.
* `Ctrl+C` stops the container.

Start in background (**detached**):

```bash
docker run -d nginx:1.23
```

* `-d` = detached mode (doesn’t block terminal).

List running containers:

```bash
docker ps
```

List **all** containers (including stopped):

```bash
docker ps -a
```

View logs of a container:

```bash
docker logs <container-id-or-name>
```

Stop a running container:

```bash
docker stop <container-id-or-name>
```

Restart a stopped container:

```bash
docker start <container-id-or-name>
```

Give a container a friendly name:

```bash
docker run -d --name web-app nginx:1.23
docker logs web-app
docker stop web-app
```

---

## 8. Port Mapping (Accessing Containers from Host)

Containers live in an **internal Docker network**.
To access them from your machine (e.g. browser), you need **port binding**.

Example: Nginx

* Nginx listens on port `80` *inside* the container.

To expose it on your host’s port `9000`:

```bash
docker run -d -p 9000:80 nginx:1.23
```

* `-p HOST_PORT:CONTAINER_PORT`
* Now visit in browser:

  * `http://localhost:9000`
* You should see the Nginx welcome page.

Check which port a container is bound to:

```bash
docker ps
# Look at the "PORTS" column
```

Common pattern: use the **same port** on host & container if possible:

* e.g. `-p 3306:3306` for MySQL, `-p 6379:6379` for Redis, etc.

---

## 9. Creating Your Own Image with a Dockerfile (Node.js Example)

Goal: Package a simple Node.js app into a Docker image.

**Project structure:**

```text
.
├─ Dockerfile
├─ package.json      # has Express dependency
└─ src/
   └─ server.js      # starts app on port 3000, responds "Welcome..."
```

### 9.1 Dockerfile basics

Key Dockerfile instructions used:

* `FROM` – base image (usually a minimal OS + runtime).
* `COPY` – copy files from host into image.
* `WORKDIR` – set working directory inside image.
* `RUN` – run a command **during image build** (e.g. install deps).
* `CMD` – command to **start the app** when container runs.

Example Dockerfile:

```dockerfile
# 1. Base image: minimal Linux + Node + npm
FROM node:19-alpine

# 2. Copy dependency manifest first (for better layer caching)
COPY package.json /app/

# 3. Copy source files
COPY src /app/src/

# 4. Set working directory inside container
WORKDIR /app

# 5. Install dependencies
RUN npm install

# 6. Default command when container starts
CMD ["node", "src/server.js"]
```

### 9.2 Building the image

From the directory containing `Dockerfile`:

```bash
docker build -t node-app:1.0 .
```

* `-t node-app:1.0` → name `node-app`, tag `1.0`
* `.` → build context = current directory; Docker finds `Dockerfile` there.

After build:

```bash
docker images
# You should see node-app:1.0
```

### 9.3 Running your custom image

App listens on `3000` inside container (as in `server.js`), so:

```bash
docker run -d -p 3000:3000 node-app:1.0
```

Now go to:

* `http://localhost:3000`

You should see your app’s response (“Welcome to my awesome app” or similar).

Logs:

```bash
docker ps              # get container ID
docker logs <id-or-name>
```

---

## 10. Private Registries & Repositories (Company Setup)

Typical company setup:

* Use a **private registry** to store internal images:

  * Example: AWS ECR, private Docker Hub repo, Nexus, etc.
* Structure:

  * Registry (service)
  * Multiple **repositories**:

    * e.g. `company/frontend`, `company/payments-service`, etc.
  * Each repository holds multiple tags (versions).

Flow:

1. CI (e.g. Jenkins/GitHub Actions/GitLab CI) builds image from app source.
2. CI pushes image to **private registry**:

   ```bash
   docker push company/app:1.0.0
   ```
3. Servers pull and run that image:

   ```bash
   docker pull company/app:1.0.0
   docker run -d -p 80:3000 company/app:1.0.0
   ```

---

## 11. Docker in the Software Development Lifecycle (Big Picture)

### Local development

* You develop on laptop.
* Dependencies (DB, cache, etc.) run as containers:

  * `docker run -d -p 27017:27017 mongo:6`
  * `docker run -d -p 6379:6379 redis:7`
* Your app connects to those containerized services (usually via `localhost` and the mapped ports).

### CI / Build (e.g. Jenkins)

* On `git push` or PR:

  1. Build your app (tests, compile, etc.).
  2. Build a Docker image using `Dockerfile`.
  3. Tag and push image to **private registry**.

### Deployment

* Dev/staging/prod servers (or Kubernetes clusters) pull the image:

  ```bash
  docker pull company/app:1.0.0
  docker run -d -p 80:3000 company/app:1.0.0
  ```
* Other services like DB may:

  * Also run as containers (pulled from Docker Hub or internal registry), or
  * Be managed services (RDS, Cloud SQL, etc.) that your container connects to.

**Key idea:** Docker gives you a **standard, repeatable unit** (image) that moves through all stages: dev → CI → staging → prod.

---

## 12. Quick Command Reference

```bash
# List images
docker images

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Pull an image
docker pull nginx:1.23

# Run an image (foreground)
docker run nginx:1.23

# Run in background with name and port mapping
docker run -d --name web-app -p 9000:80 nginx:1.23

# View logs
docker logs web-app

# Stop / start / remove containers
docker stop web-app
docker start web-app
docker rm web-app

# Build custom image from Dockerfile
docker build -t node-app:1.0 .

# Run custom image
docker run -d -p 3000:3000 node-app:1.0
```

---

## 13. Next Topics to Revisit (Beyond This Crash Course)

When you’re comfortable with the basics above, next good Docker topics:

* **Docker Compose** – run multi-container setups (app + DB + cache) with one YAML file.
* **Volumes** – persist data outside the container (e.g. DB storage).
* **Networks** – custom networks for multi-service architecture.
* **Optimizing images** – multi-stage builds, smaller base images, caching.

---
