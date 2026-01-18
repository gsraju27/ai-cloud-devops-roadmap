# Complete Docker & Container Interview Guide

Master Docker with 12 real-world interview questions covering Dockerfile optimization, multi-stage builds, networking, security, debugging, and production best practices. Practice scenarios that mirror actual senior DevOps/SRE engineer challenges.

**Companies that ask these questions:** Google | Amazon | Microsoft | Netflix | Spotify | Uber

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | This Docker image is 2GB. Optimize it to under 100MB using m... | Practical | Dockerfile Optimization |
| 2 | A container keeps exiting immediately after starting. How do... | Debugging | Debugging & Troubleshooting |
| 3 | Two containers can't communicate with each other. Diagnose a... | Debugging | Networking |
| 4 | This container is running as root with full capabilities. Se... | Practical | Security |
| 5 | A container can't write to its mounted volume. Fix the permi... | Debugging | Volumes & Storage |
| 6 | Docker builds are taking 10 minutes. Optimize the Dockerfile... | Practical | Build Optimization |
| 7 | Design a Docker Compose configuration for a microservices ap... | Architecture | Docker Compose |
| 8 | A container's memory usage keeps growing until it gets OOM k... | Debugging | Resource Management |
| 9 | This container doesn't shut down gracefully and loses in-fli... | Practical | Container Lifecycle |
| 10 | Security scan found critical vulnerabilities in your base im... | Practical | Security |
| 11 | Container DNS resolution is failing intermittently. Troubles... | Debugging | Networking |
| 12 | Design a CI/CD pipeline for building, testing, and deploying... | Architecture | CI/CD & Registry |

---

## What You'll Learn

- Optimize Dockerfiles using multi-stage builds to reduce image size by 90%+
- Debug container crashes, networking issues, and resource problems
- Implement Docker security best practices and vulnerability scanning
- Design efficient Docker networking for microservices architectures
- Manage persistent data with volumes and bind mounts
- Write production-ready Docker Compose configurations
- Troubleshoot common container runtime issues
- Implement health checks and graceful shutdown patterns
- Optimize build caching and layer efficiency
- Handle secrets and environment configuration securely

---

## Interview Questions

### Question 1: This Docker image is 2GB. Optimize it to under 100MB using multi-stage builds.

**Type:** Practical | **Category:** Dockerfile Optimization

## The Scenario

Your team inherited a Node.js microservice with this Dockerfile:

```dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

The resulting image is **2.1GB**. Your DevOps lead wants it under 100MB for faster deployments and reduced storage costs. The image takes 3 minutes to pull on each deployment, causing slow rollouts.

## The Challenge

Optimize this Dockerfile to reduce the image size by 95%+ while maintaining functionality. Explain each optimization technique and why it works.


### Step 1: Understand Why the Image is Large

```bash
# Analyze image layers
docker history your-app:latest

# Check what's taking space
docker run --rm -it your-app:latest du -sh /*
```

**Common culprits:**
- Full Node.js image with npm, yarn, build tools (~900MB)
- Development dependencies in node_modules (~500MB+)
- Source files, tests, documentation (~100MB+)
- Build artifacts and cache files

### Step 2: Implement Multi-Stage Build

```dockerfile
# ============================================
# Stage 1: Build
# ============================================
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files first (better layer caching)
COPY package*.json ./

# Install ALL dependencies (including devDependencies)
RUN npm ci

# Copy source code
COPY . .

# Build the application
RUN npm run build

# ============================================
# Stage 2: Production
# ============================================
FROM node:18-alpine AS production

# Add non-root user for security
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy only production dependencies
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Copy built application from builder stage
COPY --from=builder /app/dist ./dist

# Use non-root user
USER nodejs

EXPOSE 3000

CMD ["node", "dist/server.js"]
```

### Step 3: Add .dockerignore

```plaintext
# .dockerignore
node_modules
npm-debug.log
Dockerfile*
.dockerignore
.git
.gitignore
README.md
.env*
coverage
.nyc_output
*.test.js
*.spec.js
__tests__
docs
```

### Step 4: Verify the Optimization

```bash
# Build and check size
docker build -t your-app:optimized .
docker images | grep your-app

# Before: 2.1GB
# After:  ~80MB (96% reduction!)
```


## Size Comparison

| Approach | Image Size | Reduction |
|----------|-----------|-----------|
| Original (node:18) | 2.1 GB | - |
| node:18-slim | 450 MB | 78% |
| node:18-alpine | 180 MB | 91% |
| Multi-stage + alpine | 80 MB | 96% |
| Distroless | 50 MB | 98% |

## Advanced: Using Distroless Images

For maximum security and minimum size:

```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage with distroless
FROM gcr.io/distroless/nodejs18-debian11

WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

EXPOSE 3000
CMD ["dist/server.js"]
```

**Distroless benefits:**
- No shell, package manager, or unnecessary binaries
- Smaller attack surface
- ~50MB final image size


---

### Quick Check

**What is the primary benefit of using multi-stage Docker builds?**

   A. They make the build process faster by parallelizing stages
-> B. **They allow you to use different base images for building and running, keeping only necessary artifacts in the final image**
   C. They automatically remove all development dependencies
   D. They compress the final image using advanced algorithms

<details>
<summary>See Answer</summary>

Multi-stage builds let you use a full-featured image (with compilers, build tools, dev dependencies) for building, then copy only the compiled artifacts to a minimal runtime image. This separates build-time requirements from runtime requirements, dramatically reducing the final image size. The other stages and their contents are discarded, not included in the final image.

</details>

---

### Question 2: A container keeps exiting immediately after starting. How do you debug it?

**Type:** Debugging | **Category:** Debugging & Troubleshooting

## The Scenario

You deploy a new version of your API service and the container keeps crashing:

```bash
$ docker ps -a
CONTAINER ID   IMAGE          STATUS                     NAMES
a1b2c3d4e5f6   api:v2.1.0     Exited (1) 2 seconds ago   api-service
```

Every time you start it, it exits within seconds. The previous version (v2.0.0) works fine. Production is partially down and you need to fix this quickly.

## The Challenge

Walk through your systematic debugging process. What commands would you run, in what order, and why? How do you identify whether this is a code issue, configuration problem, or missing dependency?


### Step 1: Check Exit Code and Logs (First 30 seconds)

```bash
# Check the exit code
docker inspect api-service --format='{{.State.ExitCode}}'
# Exit code 1 = application error
# Exit code 137 = OOM killed (128 + 9 SIGKILL)
# Exit code 139 = Segmentation fault
# Exit code 143 = SIGTERM (graceful shutdown)

# Get container logs
docker logs api-service

# If container restarts too fast, get logs from last run
docker logs api-service 2>&1 | tail -100
```

**Common log patterns:**
- `Error: Cannot find module 'express'` → Missing dependency
- `EADDRINUSE` → Port already in use
- `permission denied` → File/directory access issue
- `ECONNREFUSED` → Can't reach database/dependency

### Step 2: Inspect Container Configuration

```bash
# Check full container details
docker inspect api-service

# Key things to look for:
docker inspect api-service --format='
CMD: {{.Config.Cmd}}
Entrypoint: {{.Config.Entrypoint}}
Env: {{range .Config.Env}}{{.}} {{end}}
WorkingDir: {{.Config.WorkingDir}}
'

# Compare with working version
docker inspect api:v2.0.0 --format='{{.Config.Cmd}}'
docker inspect api:v2.1.0 --format='{{.Config.Cmd}}'
```

### Step 3: Get Shell Access for Live Debugging

```bash
# Override entrypoint to get shell access
docker run -it --entrypoint /bin/sh api:v2.1.0

# Inside the container, manually run the command
$ node server.js
# Now you'll see the actual error in real-time

# Check if files exist
$ ls -la /app
$ cat /app/package.json

# Check environment variables
$ env | grep -i database
```

### Step 4: Check Resource Constraints

```bash
# Was it killed due to memory limits?
docker inspect api-service --format='{{.State.OOMKilled}}'

# Check memory limit vs usage
docker stats api-service --no-stream

# Check if there are resource limits
docker inspect api-service --format='
Memory Limit: {{.HostConfig.Memory}}
CPU Shares: {{.HostConfig.CpuShares}}
'
```

### Step 5: Compare Image Layers

```bash
# Check what changed between versions
docker history api:v2.0.0
docker history api:v2.1.0

# Use dive tool for detailed analysis
dive api:v2.1.0
```


## Common Root Causes and Fixes

| Exit Code | Meaning | Common Causes | Fix |
|-----------|---------|---------------|-----|
| 0 | Success | CMD finished (not a daemon) | Ensure process runs in foreground |
| 1 | General error | Application crash, missing config | Check logs for specific error |
| 126 | Permission denied | Script not executable | `chmod +x script.sh` |
| 127 | Command not found | Wrong CMD/ENTRYPOINT path | Verify binary exists in image |
| 137 | SIGKILL (OOM) | Memory limit exceeded | Increase memory or fix leak |
| 139 | SIGSEGV | Segmentation fault | Debug application code |
| 143 | SIGTERM | Graceful shutdown | Check why container was stopped |

## Real-World Example: Missing Environment Variable

**Logs show:**
```
Error: DATABASE_URL environment variable is required
    at validateConfig (/app/dist/config.js:15:11)
    at Object.<anonymous> (/app/dist/server.js:3:1)
```

**Root Cause:** New version added database URL validation, but the environment variable wasn't set in the docker run command.

**Fix:**
```bash
# Add the missing environment variable
docker run -d \
  -e DATABASE_URL=postgres://user:pass@db:5432/myapp \
  --name api-service \
  api:v2.1.0
```

## Debugging Quick Reference

```bash
# Full debugging workflow
docker logs <container>                          # Check logs
docker inspect <container> --format='{{.State}}' # Check state
docker diff <container>                          # Check filesystem changes
docker top <container>                           # Check running processes
docker exec -it <container> /bin/sh              # Get shell (if running)
docker run -it --entrypoint /bin/sh <image>      # Override entrypoint
```


---

### Quick Check

**A container exits with code 137. What is the most likely cause?**

   A. The application encountered an unhandled exception
-> B. **The container was killed due to exceeding its memory limit (OOM)**
   C. The entrypoint script could not be found
   D. The container received a SIGTERM signal for graceful shutdown

<details>
<summary>See Answer</summary>

Exit code 137 equals 128 + 9, where 9 is SIGKILL. In containerized environments, this almost always indicates the container exceeded its memory limit and was killed by the OOM (Out of Memory) killer. Exit code 1 indicates application errors, 127 means command not found, and 143 (128 + 15) indicates SIGTERM for graceful shutdown.

</details>

---

### Question 3: Two containers can't communicate with each other. Diagnose and fix the networking issue.

**Type:** Debugging | **Category:** Networking

## The Scenario

You have a Node.js API container trying to connect to a Redis container:

```bash
$ docker ps
CONTAINER ID   IMAGE      PORTS                    NAMES
a1b2c3d4e5f6   api:v1     0.0.0.0:3000->3000/tcp   api
b2c3d4e5f6g7   redis:7    0.0.0.0:6379->6379/tcp   redis

$ docker logs api
Error: connect ECONNREFUSED 127.0.0.1:6379
    at TCPConnectWrap.afterConnect
```

The API can't reach Redis, but you can connect to Redis from your host machine. What's wrong?

## The Challenge

Diagnose the networking issue and fix it. Explain the different Docker networking modes and when to use each.


### Step 1: Understand the Problem

```bash
# Check which networks each container is on
docker inspect api --format='{{range .NetworkSettings.Networks}}{{.NetworkID}}{{end}}'
docker inspect redis --format='{{range .NetworkSettings.Networks}}{{.NetworkID}}{{end}}'

# List all networks
docker network ls

# Inspect the default bridge network
docker network inspect bridge
```

**The issue:** By default, containers on the default bridge network can't resolve each other by name. They need to be on a user-defined network for DNS resolution.

### Step 2: Create a User-Defined Network

```bash
# Create a custom bridge network
docker network create app-network

# Verify it was created
docker network ls
```

### Step 3: Connect Containers to the Network

```bash
# Option 1: Connect running containers
docker network connect app-network api
docker network connect app-network redis

# Option 2: Start containers on the network
docker run -d --name redis --network app-network redis:7
docker run -d --name api --network app-network -p 3000:3000 api:v1
```

### Step 4: Update Application Configuration

```javascript
// Before (wrong)
const redis = new Redis({
  host: '127.0.0.1',  // This is the container itself!
  port: 6379
});

// After (correct)
const redis = new Redis({
  host: 'redis',  // Container name as hostname
  port: 6379
});
```

### Step 5: Verify Connectivity

```bash
# Test DNS resolution from API container
docker exec api ping redis

# Test TCP connection
docker exec api nc -zv redis 6379

# Check network configuration
docker inspect api --format='{{json .NetworkSettings.Networks}}' | jq
```


## Docker Network Types

| Network Type | Use Case | Container-to-Container | Host Access |
|--------------|----------|------------------------|-------------|
| **bridge** (default) | Default for standalone containers | By IP only (no DNS) | Via port mapping |
| **user-defined bridge** | Multi-container apps | By name (DNS) | Via port mapping |
| **host** | Performance-critical apps | Via localhost | Direct (no port mapping) |
| **none** | Maximum isolation | No networking | None |
| **overlay** | Multi-host (Swarm/K8s) | By name across hosts | Via routing mesh |

## Docker Compose Solution

The best way to handle multi-container networking:

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build: ./api
    ports:
      - "3000:3000"
    environment:
      - REDIS_HOST=redis
    depends_on:
      - redis
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  redis-data:
```

**Docker Compose automatically:**
- Creates a network named `<project>_app-network`
- Connects all services to it
- Enables DNS resolution by service name

## Debugging Network Issues

```bash
# Check container's network configuration
docker inspect <container> --format='{{json .NetworkSettings}}' | jq

# List containers on a network
docker network inspect app-network --format='{{range .Containers}}{{.Name}} {{end}}'

# Test connectivity from inside container
docker exec -it api sh
$ ping redis
$ nc -zv redis 6379
$ curl http://other-service:8080/health

# Check DNS resolution
docker exec api nslookup redis

# Check iptables rules (on host)
sudo iptables -L -n | grep DOCKER
```

---

### Question 4: This container is running as root with full capabilities. Secure it for production.

**Type:** Practical | **Category:** Security

## The Scenario

Your security team flagged this Dockerfile during a compliance audit:

```dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["node", "server.js"]
```

**Security findings:**
- Container runs as root (UID 0)
- All Linux capabilities enabled
- Writable filesystem
- No resource limits
- Base image has known vulnerabilities

You need to harden this container for PCI-DSS compliance before the next deployment.

## The Challenge

Implement security best practices to harden this container. Explain each security measure and the attack vectors it prevents.


### Step 1: Create Non-Root User with Proper Ownership

```dockerfile
FROM node:18-alpine

# Create non-root user BEFORE copying files
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 -G nodejs

WORKDIR /app

# Copy with correct ownership
COPY --chown=nodejs:nodejs package*.json ./
RUN npm ci --only=production

COPY --chown=nodejs:nodejs . .

# Switch to non-root user
USER nodejs

EXPOSE 3000
CMD ["node", "server.js"]
```

### Step 2: Drop Capabilities and Add Security Options

```bash
# Run with minimal capabilities
docker run -d \
  --name api \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --security-opt=no-new-privileges:true \
  --read-only \
  --tmpfs /tmp:rw,noexec,nosuid \
  api:secure
```

**Security flags explained:**
- `--cap-drop=ALL` - Remove all Linux capabilities
- `--cap-add=NET_BIND_SERVICE` - Only add what's needed
- `--no-new-privileges` - Prevent privilege escalation via setuid
- `--read-only` - Immutable filesystem
- `--tmpfs /tmp` - Writable temp directory without persistence

### Step 3: Complete Secure Dockerfile

```dockerfile
# Use specific version, not 'latest'
FROM node:18.19.0-alpine3.19

# Security: Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 -G nodejs

# Security: Set restrictive permissions on workdir
WORKDIR /app
RUN chown nodejs:nodejs /app

# Install dependencies as root, then fix ownership
COPY package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force && \
    chown -R nodejs:nodejs /app

# Copy application code with correct ownership
COPY --chown=nodejs:nodejs . .

# Security: Switch to non-root user
USER nodejs

# Security: Don't run as PID 1 (use tini or dumb-init)
# Alpine includes tini
ENTRYPOINT ["/sbin/tini", "--"]

EXPOSE 3000

# Use exec form to ensure signals are handled properly
CMD ["node", "server.js"]

# Security labels
LABEL security.privileged="false" \
      security.allowPrivilegeEscalation="false"
```

### Step 4: Docker Compose with Security Options

```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "3000:3000"
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid,size=64m
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          memory: 256M
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### Step 5: Scan for Vulnerabilities

```bash
# Scan image for vulnerabilities
docker scout cves api:secure

# Or use trivy
trivy image api:secure

# Or use grype
grype api:secure

# Fix vulnerabilities by updating base image
# Check for newer Alpine/Node versions with fewer CVEs
```


## Security Checklist

| Security Measure | Attack Vector Prevented |
|------------------|------------------------|
| Non-root user | Privilege escalation, host filesystem access |
| Drop capabilities | Kernel exploits, container escapes |
| Read-only filesystem | Malware persistence, config tampering |
| No-new-privileges | Setuid/setgid exploits |
| Resource limits | DoS attacks, resource exhaustion |
| Minimal base image | Reduced attack surface, fewer CVEs |
| Specific image tags | Supply chain attacks, unexpected changes |
| Vulnerability scanning | Known CVE exploitation |

## Runtime Security with Docker

```bash
# Full security hardened run command
docker run -d \
  --name api \
  --user 1001:1001 \
  --cap-drop=ALL \
  --security-opt=no-new-privileges:true \
  --security-opt=apparmor=docker-default \
  --read-only \
  --tmpfs /tmp:rw,noexec,nosuid,size=64m \
  --memory=512m \
  --memory-swap=512m \
  --cpus=1 \
  --pids-limit=100 \
  --restart=on-failure:3 \
  --health-cmd="wget -q --spider http://localhost:3000/health" \
  --health-interval=30s \
  -p 3000:3000 \
  api:secure
```

---

### Question 5: A container can't write to its mounted volume. Fix the permission issue.

**Type:** Debugging | **Category:** Volumes & Storage

## The Scenario

You're running a Node.js application that needs to write logs to a mounted volume:

```bash
$ docker run -d \
    --name api \
    --user 1001:1001 \
    -v /data/logs:/app/logs \
    api:latest

$ docker logs api
Error: EACCES: permission denied, open '/app/logs/app.log'
```

The container runs as a non-root user (security best practice), but now it can't write to the mounted volume.

## The Challenge

Fix the permission issue while maintaining security best practices. Explain the relationship between container user IDs and host filesystem permissions.


### Understanding the Problem

```bash
# Check who owns the host directory
ls -la /data/logs
# drwxr-xr-x 2 root root 4096 Jan 15 10:00 /data/logs

# Container runs as UID 1001, but directory is owned by root
# UID 1001 has no write permission
```

**Key Insight:** Docker doesn't map usernames between host and container. It uses UIDs directly. A user named `nodejs` with UID 1001 in the container is treated as UID 1001 on the host, regardless of what that UID is called on the host.

### Solution 1: Fix Host Directory Ownership

```bash
# Create directory with correct ownership
sudo mkdir -p /data/logs
sudo chown 1001:1001 /data/logs

# Verify
ls -la /data/
# drwxr-xr-x 2 1001 1001 4096 Jan 15 10:00 logs

# Now the container can write
docker run -d \
    --name api \
    --user 1001:1001 \
    -v /data/logs:/app/logs \
    api:latest
```

### Solution 2: Use Named Volumes (Recommended)

```bash
# Create a named volume
docker volume create app-logs

# Run container with named volume
docker run -d \
    --name api \
    --user 1001:1001 \
    -v app-logs:/app/logs \
    api:latest
```

**Why named volumes are better:**
- Docker manages permissions automatically
- Portable across hosts
- Easier backup and migration
- Better for production workloads

### Solution 3: Init Container Pattern

In Docker Compose, use an init container to fix permissions:

```yaml
version: '3.8'

services:
  # Init container fixes permissions
  init-permissions:
    image: busybox
    command: chown -R 1001:1001 /data
    volumes:
      - app-data:/data
    user: root

  # Application container
  api:
    image: api:latest
    user: "1001:1001"
    volumes:
      - app-data:/app/logs
    depends_on:
      init-permissions:
        condition: service_completed_successfully

volumes:
  app-data:
```

### Solution 4: Match UIDs in Dockerfile

```dockerfile
FROM node:18-alpine

# Use a UID that matches your host system
ARG UID=1001
ARG GID=1001

RUN addgroup -g ${GID} -S nodejs && \
    adduser -S nodejs -u ${UID} -G nodejs

WORKDIR /app
COPY --chown=nodejs:nodejs . .

USER nodejs
CMD ["node", "server.js"]
```

Build with matching UID:

```bash
# Build with UID that matches host directory owner
docker build --build-arg UID=$(id -u) --build-arg GID=$(id -g) -t api:latest .
```


## Permission Debugging Commands

```bash
# Check container's user
docker exec api id
# uid=1001(nodejs) gid=1001(nodejs)

# Check mounted directory permissions inside container
docker exec api ls -la /app/logs

# Check host directory permissions
ls -la /data/logs

# Test write access
docker exec api touch /app/logs/test.txt

# Check volume mount details
docker inspect api --format='{{json .Mounts}}' | jq
```

## Common Permission Scenarios

| Scenario | Host Owner | Container User | Result | Fix |
|----------|------------|----------------|--------|-----|
| Default | root (0) | root (0) | Works | - |
| Secure container | root (0) | 1001 | Fails | chown 1001 on host |
| Named volume | Docker | 1001 | Works | - |
| SELinux enabled | root (0) | 1001 | Fails | Add :z or :Z suffix |

## SELinux/AppArmor Considerations

On systems with SELinux (RHEL/CentOS) or AppArmor:

```bash
# Add :z for shared volume (multiple containers)
docker run -v /data/logs:/app/logs:z api:latest

# Add :Z for private volume (single container)
docker run -v /data/logs:/app/logs:Z api:latest
```

**Warning:** `:Z` relabels the host directory. Don't use it on system directories!

---

### Question 6: Docker builds are taking 10 minutes. Optimize the Dockerfile to use layer caching effectively.

**Type:** Practical | **Category:** Build Optimization

## The Scenario

Your CI/CD pipeline builds this Docker image on every commit:

```dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
CMD ["node", "dist/server.js"]
```

**Current build metrics:**
- Average build time: 10 minutes
- `npm install` runs every build (5 minutes)
- Any file change invalidates all layers
- CI costs are increasing rapidly

## The Challenge

Optimize this Dockerfile to take advantage of Docker's layer caching. Explain how layer caching works and why the order of instructions matters.


### Understanding Docker Layer Caching

```
Each instruction creates a layer:
FROM node:18           → Layer 1 (base image)
WORKDIR /app           → Layer 2 (cached unless base changes)
COPY . .               → Layer 3 (invalidated on ANY file change!)
RUN npm install        → Layer 4 (rebuilt because layer 3 changed)
RUN npm run build      → Layer 5 (rebuilt because layer 4 changed)
```

**The problem:** `COPY . .` invalidates the cache whenever ANY file changes, forcing npm install to run every time.

### Step 1: Separate Dependencies from Source Code

```dockerfile
FROM node:18-alpine
WORKDIR /app

# Step 1: Copy ONLY package files (changes rarely)
COPY package.json package-lock.json ./

# Step 2: Install dependencies (cached if package*.json unchanged)
RUN npm ci

# Step 3: Copy source code (changes frequently)
COPY . .

# Step 4: Build (only runs if source changed)
RUN npm run build

CMD ["node", "dist/server.js"]
```

**Result:** If only source code changes, Docker uses cached npm install layer!

### Step 2: Add .dockerignore

```plaintext
# .dockerignore - prevent unnecessary cache invalidation
node_modules
npm-debug.log
.git
.gitignore
*.md
Dockerfile*
.dockerignore
coverage
.env*
dist
.nyc_output
```

### Step 3: Use BuildKit for Parallel Builds

```dockerfile
# syntax=docker/dockerfile:1.4
FROM node:18-alpine AS base
WORKDIR /app

# Dependencies stage
FROM base AS deps
COPY package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci

# Build stage
FROM base AS builder
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Production stage
FROM base AS runner
COPY --from=builder /app/dist ./dist
COPY --from=deps /app/node_modules ./node_modules
CMD ["node", "dist/server.js"]
```

Enable BuildKit:

```bash
DOCKER_BUILDKIT=1 docker build -t myapp .
```

### Step 4: Cache npm Downloads

```dockerfile
# syntax=docker/dockerfile:1.4
FROM node:18-alpine
WORKDIR /app

COPY package.json package-lock.json ./

# Mount cache for npm packages
RUN --mount=type=cache,target=/root/.npm \
    npm ci --prefer-offline

COPY . .
RUN npm run build

CMD ["node", "dist/server.js"]
```

**BuildKit cache mounts:**
- `--mount=type=cache` persists directory across builds
- npm/yarn don't re-download unchanged packages
- Significant speedup for dependency installation


## Layer Caching Best Practices

| Instruction | Cache Invalidation | Optimization |
|-------------|-------------------|--------------|
| `COPY . .` | Any file change | Split into multiple COPYs |
| `RUN npm install` | package.json change | Copy package*.json first |
| `RUN apt-get update` | Always re-run | Combine with install in one layer |
| `ARG VERSION` | Value change | Put after static layers |
| `ENV` | Value change | Put late if dynamic |

## Optimized Build Order

```dockerfile
# syntax=docker/dockerfile:1.4
FROM node:18-alpine AS builder

# 1. Install system dependencies (rarely changes)
RUN apk add --no-cache python3 make g++

WORKDIR /app

# 2. Copy dependency manifests (changes weekly)
COPY package.json package-lock.json ./

# 3. Install dependencies with cache mount
RUN --mount=type=cache,target=/root/.npm \
    npm ci

# 4. Copy configuration files (changes monthly)
COPY tsconfig.json ./

# 5. Copy source code (changes on every commit)
COPY src ./src

# 6. Build application
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER node
CMD ["node", "dist/server.js"]
```

## CI/CD Cache Configuration

```yaml
# GitHub Actions example
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3

- name: Build and push
  uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: myapp:latest
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

## Measuring Build Performance

```bash
# Time the build
time docker build -t myapp .

# Show layer sizes
docker history myapp

# Analyze with dive
dive myapp
```


---

### Quick Check

**Why should you COPY package.json before COPY . . in a Node.js Dockerfile?**

   A. To make the image smaller by excluding node_modules
   B. To ensure npm install runs before the source code is available
-> C. **So that npm install layer is cached when only source code changes, not package.json**
   D. To prevent security vulnerabilities in the source code from affecting dependencies

<details>
<summary>See Answer</summary>

Docker caches layers sequentially. If you COPY package.json first, then run npm install, that layer is cached. When only source code changes (not package.json), Docker reuses the cached npm install layer instead of re-running it. This can save 5+ minutes per build since dependency installation is typically the slowest step. COPY . . would invalidate the cache on any file change.

</details>

---

### Question 7: Design a Docker Compose configuration for a microservices application with proper networking, health checks, and dependencies.

**Type:** Architecture | **Category:** Docker Compose

## The Scenario

You're containerizing a microservices application with:
- **API Gateway** (Node.js) - receives all traffic, routes to services
- **User Service** (Python) - handles authentication
- **Order Service** (Go) - processes orders
- **PostgreSQL** - user data
- **Redis** - session cache
- **RabbitMQ** - async messaging

Current issues with the basic setup:
- Services start before databases are ready
- No health monitoring
- All services on same network (security concern)
- No resource limits (one service can starve others)

## The Challenge

Design a production-ready Docker Compose configuration that handles service dependencies, network isolation, health checks, and resource management.


> **Common Mistake:** A junior engineer might use depends_on without health checks, put all services on the default network, skip resource limits, hardcode credentials in the compose file, and not consider graceful startup order. This leads to race conditions, security vulnerabilities, resource exhaustion, and leaked secrets.

> **Senior Engineer Approach:** A senior engineer implements defense in depth: use depends_on with service_healthy condition, separate networks for different tiers, proper health checks for all services, resource limits, externalized secrets, and proper logging configuration. Each layer addresses specific production concerns.

### Step 1: Basic Structure with Health Checks

```yaml
version: '3.8'

services:
  # API Gateway - Entry point
  api-gateway:
    build: ./api-gateway
    ports:
      - "8080:8080"
    environment:
      - USER_SERVICE_URL=http://user-service:3000
      - ORDER_SERVICE_URL=http://order-service:4000
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    depends_on:
      user-service:
        condition: service_healthy
      order-service:
        condition: service_healthy
```

### Step 2: Network Isolation

```yaml
networks:
  # Public network - only API gateway exposed
  frontend:
    driver: bridge

  # Internal network for services
  backend:
    driver: bridge
    internal: true  # No external access

  # Database network - maximum isolation
  database:
    driver: bridge
    internal: true

services:
  api-gateway:
    networks:
      - frontend
      - backend

  user-service:
    networks:
      - backend
      - database

  order-service:
    networks:
      - backend
      - database

  postgres:
    networks:
      - database  # Only accessible from backend services

  redis:
    networks:
      - backend  # Cache accessible from backend only
```

### Step 3: Complete Production Configuration

```yaml
version: '3.8'

services:
  # ============================================
  # API Gateway
  # ============================================
  api-gateway:
    build:
      context: ./api-gateway
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - NODE_ENV=production
      - USER_SERVICE_URL=http://user-service:3000
      - ORDER_SERVICE_URL=http://order-service:4000
      - REDIS_URL=redis://redis:6379
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    depends_on:
      user-service:
        condition: service_healthy
      order-service:
        condition: service_healthy
      redis:
        condition: service_healthy
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
        reservations:
          cpus: '0.25'
          memory: 128M
    networks:
      - frontend
      - backend
    restart: unless-stopped
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

  # ============================================
  # User Service
  # ============================================
  user-service:
    build: ./user-service
    environment:
      - DATABASE_URL=postgres://user:${DB_PASSWORD}@postgres:5432/users
      - REDIS_URL=redis://redis:6379
      - RABBITMQ_URL=amqp://rabbitmq:5672
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
    networks:
      - backend
      - database
    restart: unless-stopped

  # ============================================
  # Order Service
  # ============================================
  order-service:
    build: ./order-service
    environment:
      - DATABASE_URL=postgres://order:${DB_PASSWORD}@postgres:5432/orders
      - RABBITMQ_URL=amqp://rabbitmq:5672
    healthcheck:
      test: ["CMD", "/app/healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 3
    depends_on:
      postgres:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 256M
    networks:
      - backend
      - database
    restart: unless-stopped

  # ============================================
  # PostgreSQL
  # ============================================
  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_MULTIPLE_DATABASES=users,orders
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init-db.sh:/docker-entrypoint-initdb.d/init-db.sh
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
    networks:
      - database
    restart: unless-stopped

  # ============================================
  # Redis
  # ============================================
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
    networks:
      - backend
    restart: unless-stopped

  # ============================================
  # RabbitMQ
  # ============================================
  rabbitmq:
    image: rabbitmq:3-management-alpine
    environment:
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=${RABBITMQ_PASSWORD}
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "-q", "ping"]
      interval: 30s
      timeout: 10s
      retries: 5
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
    networks:
      - backend
    restart: unless-stopped

# ============================================
# Networks
# ============================================
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true
  database:
    driver: bridge
    internal: true

# ============================================
# Volumes
# ============================================
volumes:
  postgres-data:
  redis-data:
  rabbitmq-data:
```


## Dependency Management

```yaml
# Wrong - just waits for container to start
depends_on:
  - postgres

# Right - waits for health check to pass
depends_on:
  postgres:
    condition: service_healthy
```

## Health Check Patterns

| Service Type | Health Check Command |
|--------------|---------------------|
| HTTP API | `wget -q --spider http://localhost:PORT/health` |
| PostgreSQL | `pg_isready -U postgres` |
| Redis | `redis-cli ping` |
| RabbitMQ | `rabbitmq-diagnostics -q ping` |
| MongoDB | `mongosh --eval "db.adminCommand('ping')"` |
| Custom binary | `/app/healthcheck` |

---

### Question 8: A container's memory usage keeps growing until it gets OOM killed. How do you diagnose and fix it?

**Type:** Debugging | **Category:** Resource Management

## The Scenario

Your production API container keeps getting killed after running for a few hours:

```bash
$ docker ps -a
CONTAINER ID   IMAGE      STATUS                       NAMES
a1b2c3d4e5f6   api:v2.0   Exited (137) 5 minutes ago   api-service

$ docker inspect api-service --format='{{.State.OOMKilled}}'
true

$ docker stats --no-stream api-service
CONTAINER ID   NAME          CPU %     MEM USAGE / LIMIT     MEM %
a1b2c3d4e5f6   api-service   2.5%      512MiB / 512MiB       100%
```

The container starts at 150MB memory usage but grows steadily until it hits the 512MB limit and gets killed.

## The Challenge

Diagnose the memory leak and implement fixes. Explain how to monitor container memory, identify the source of leaks, and prevent OOM kills in production.


### Step 1: Confirm It's Actually a Leak

```bash
# Monitor memory over time
docker stats api-service

# Export metrics to analyze trend
docker stats --format "{{.MemUsage}}" api-service >> memory.log

# Check if memory grows continuously or reaches a plateau
watch -n 5 'docker stats --no-stream api-service'
```

**Memory growth patterns:**
- **Leak:** Continuous unbounded growth until OOM
- **Cache:** Growth to a plateau, then stable
- **Normal:** Growth during load, decrease after

### Step 2: Profile Memory Inside Container

```bash
# Get shell access
docker exec -it api-service sh

# For Node.js - check heap usage
$ node -e "console.log(process.memoryUsage())"

# For Python - check memory
$ python -c "import resource; print(resource.getrusage(resource.RUSAGE_SELF).ru_maxrss)"

# For any process - use top
$ top -b -n 1 | head -20

# Check process memory details
$ cat /proc/1/status | grep -i mem
VmRSS:	  524288 kB  # Resident memory
VmSize:	 1048576 kB  # Virtual memory
```

### Step 3: Identify Leak Source (Node.js Example)

```dockerfile
# Add debugging flags to CMD
CMD ["node", "--expose-gc", "--max-old-space-size=400", "server.js"]
```

```javascript
// Add memory monitoring endpoint
app.get('/debug/memory', (req, res) => {
  const used = process.memoryUsage();
  res.json({
    heapTotal: `${Math.round(used.heapTotal / 1024 / 1024)} MB`,
    heapUsed: `${Math.round(used.heapUsed / 1024 / 1024)} MB`,
    external: `${Math.round(used.external / 1024 / 1024)} MB`,
    rss: `${Math.round(used.rss / 1024 / 1024)} MB`
  });
});

// Force garbage collection endpoint (dev only)
app.post('/debug/gc', (req, res) => {
  if (global.gc) {
    global.gc();
    res.json({ status: 'GC triggered' });
  } else {
    res.status(400).json({ error: 'GC not exposed. Run with --expose-gc' });
  }
});
```

### Step 4: Common Leak Patterns and Fixes

**Event Listeners Not Removed:**
```javascript
// LEAK: Adding listener on every request
app.get('/data', (req, res) => {
  eventEmitter.on('data', handler); // Never removed!
});

// FIX: Use once or remove listener
app.get('/data', (req, res) => {
  eventEmitter.once('data', handler);
  // OR
  const handler = (data) => { /* ... */ };
  eventEmitter.on('data', handler);
  req.on('close', () => eventEmitter.off('data', handler));
});
```

**Unclosed Database Connections:**
```javascript
// LEAK: Connections never returned to pool
async function query(sql) {
  const conn = await pool.getConnection();
  const result = await conn.query(sql);
  // Connection never released!
  return result;
}

// FIX: Always release connections
async function query(sql) {
  const conn = await pool.getConnection();
  try {
    return await conn.query(sql);
  } finally {
    conn.release();
  }
}
```

**Growing Caches:**
```javascript
// LEAK: Unbounded cache
const cache = {};
function getCached(key) {
  if (!cache[key]) {
    cache[key] = expensiveComputation(key);
  }
  return cache[key];
}

// FIX: Use LRU cache with max size
const LRU = require('lru-cache');
const cache = new LRU({ max: 500 });
```


## Container Memory Limits

```yaml
# docker-compose.yml
services:
  api:
    image: api:v2.0
    deploy:
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M
    # Swap disabled for predictable behavior
    memswap_limit: 512M
```

## Memory Monitoring Stack

```yaml
# Add cAdvisor for container metrics
services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8081:8080"

  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
```

## OOM Prevention Strategies

| Strategy | Implementation |
|----------|---------------|
| Set appropriate limits | Match actual app needs, not arbitrary numbers |
| Application-level limits | `--max-old-space-size` for Node.js |
| Memory monitoring | Prometheus + Grafana alerts |
| Graceful degradation | Shed load before OOM |
| Restart policies | `restart: on-failure:3` |

---

### Question 9: This container doesn't shut down gracefully and loses in-flight requests. Fix it.

**Type:** Practical | **Category:** Container Lifecycle

## The Scenario

During deployments, your API container is losing requests:

```bash
$ docker stop api-service
# Takes 10 seconds (Docker's default timeout), then force kills

$ docker logs api-service
# Last lines:
[2024-01-15T10:30:45] Request received: POST /orders
[2024-01-15T10:30:45] Processing order...
# No completion log - request was killed mid-processing!
```

Users are seeing failed requests during every deployment. The engineering team has resorted to deploying only during low-traffic periods.

## The Challenge

Implement graceful shutdown so the container finishes processing in-flight requests before exiting. Explain the SIGTERM/SIGKILL lifecycle and how to handle it properly.


### Understanding the Shutdown Sequence

```
1. docker stop <container>
2. Docker sends SIGTERM to PID 1 in container
3. Container has 10 seconds (default) to exit gracefully
4. If still running, Docker sends SIGKILL (immediate termination)
```

**Common Problem:** If your app runs via a shell script, the shell (PID 1) receives SIGTERM but doesn't forward it to your app!

### Step 1: Fix the Dockerfile

```dockerfile
# WRONG: Shell form - runs through /bin/sh
CMD npm start

# WRONG: Shell wrapper traps signals
CMD ["./start.sh"]

# RIGHT: Exec form - app is PID 1, receives signals directly
CMD ["node", "server.js"]
```

### Step 2: Use a Proper Init System

```dockerfile
FROM node:18-alpine

# Install tini (included in Alpine)
RUN apk add --no-cache tini

WORKDIR /app
COPY . .
RUN npm ci --only=production

# Tini handles signal forwarding and zombie reaping
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "server.js"]
```

**Why tini?**
- Forwards signals to child processes
- Reaps zombie processes
- Lightweight (< 1MB)
- Handles edge cases your app shouldn't worry about

### Step 3: Implement Signal Handler in Application

```javascript
// server.js
const express = require('express');
const app = express();

let isShuttingDown = false;
let server;

// Middleware to reject requests during shutdown
app.use((req, res, next) => {
  if (isShuttingDown) {
    res.status(503).json({ error: 'Server is shutting down' });
    return;
  }
  next();
});

// Your routes
app.post('/orders', async (req, res) => {
  // Long-running request simulation
  await processOrder(req.body);
  res.json({ status: 'completed' });
});

// Start server
server = app.listen(3000, () => {
  console.log('Server started on port 3000');
});

// Graceful shutdown handler
function gracefulShutdown(signal) {
  console.log(`Received ${signal}. Starting graceful shutdown...`);
  isShuttingDown = true;

  // Stop accepting new connections
  server.close((err) => {
    if (err) {
      console.error('Error during shutdown:', err);
      process.exit(1);
    }

    console.log('All connections closed. Exiting.');
    process.exit(0);
  });

  // Force exit after timeout (safety net)
  setTimeout(() => {
    console.error('Shutdown timeout. Forcing exit.');
    process.exit(1);
  }, 25000); // Leave buffer before Docker's SIGKILL
}

// Handle shutdown signals
process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
process.on('SIGINT', () => gracefulShutdown('SIGINT'));
```

### Step 4: Configure Docker Stop Timeout

```yaml
# docker-compose.yml
services:
  api:
    image: api:v2.0
    stop_grace_period: 30s  # Give app 30 seconds to shutdown
```

```bash
# Command line
docker stop --time 30 api-service
```

### Step 5: Handle Database Connections

```javascript
const pool = require('./db');

async function gracefulShutdown(signal) {
  console.log(`Received ${signal}. Starting graceful shutdown...`);
  isShuttingDown = true;

  // 1. Stop accepting new requests
  server.close(async () => {
    try {
      // 2. Close database connections
      await pool.end();
      console.log('Database pool closed');

      // 3. Close other resources (Redis, MQ, etc.)
      await redis.quit();
      await mqConnection.close();

      console.log('All resources closed. Exiting.');
      process.exit(0);
    } catch (err) {
      console.error('Error during cleanup:', err);
      process.exit(1);
    }
  });
}
```


## Graceful Shutdown Checklist

| Step | Action |
|------|--------|
| 1 | Receive SIGTERM signal |
| 2 | Stop accepting new connections |
| 3 | Return 503 for new requests |
| 4 | Wait for in-flight requests to complete |
| 5 | Close database connections |
| 6 | Close message queue connections |
| 7 | Flush logs and metrics |
| 8 | Exit with code 0 |

## Testing Graceful Shutdown

```bash
# Terminal 1: Start container
docker run --name api -p 3000:3000 api:v2.0

# Terminal 2: Send long-running request
curl -X POST http://localhost:3000/orders -d '{"item":"test"}' &

# Terminal 3: Stop container while request is in progress
docker stop api

# Check logs - request should complete before shutdown
docker logs api
```

## Python Example (Flask)

```python


from flask import Flask

app = Flask(__name__)
is_shutting_down = False

@app.before_request
def check_shutdown():
    if is_shutting_down:
        return {'error': 'Server shutting down'}, 503

def graceful_shutdown(signum, frame):
    global is_shutting_down
    print(f'Received signal {signum}. Shutting down...')
    is_shutting_down = True
    # In production, use a proper WSGI server that handles this
    sys.exit(0)

signal.signal(signal.SIGTERM, graceful_shutdown)
signal.signal(signal.SIGINT, graceful_shutdown)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=3000)
```

<InterviewQuiz
  question="Why does using 'CMD npm start' in a Dockerfile often prevent graceful shutdown?"
  options={[
    "npm start is slower than directly running node",
    "The shell form runs through /bin/sh which becomes PID 1 and doesn't forward SIGTERM to node",
    "npm doesn't support the SIGTERM signal",
    "The CMD instruction doesn't allow signal handling"
  ]}
  correctAnswer={1}
  explanation="When you use shell form (CMD npm start), Docker runs '/bin/sh -c npm start'. The shell becomes PID 1 and receives SIGTERM, but it doesn't forward this signal to the child process (node). Your application never knows it should shut down. Using exec form (CMD ['node', 'server.js']) makes your app PID 1, so it receives signals directly. Alternatively, use an init system like tini that properly forwards signals."
/>

---

### Question 10: Security scan found critical vulnerabilities in your base image. How do you fix them?

**Type:** Practical | **Category:** Security

## The Scenario

Your CI/CD pipeline's security scan failed:

```bash
$ trivy image api:latest

api:latest (debian 11.6)
Total: 127 (CRITICAL: 5, HIGH: 23, MEDIUM: 67, LOW: 32)

┌────────────────┬──────────────────┬──────────┬───────────────────┐
│    Library     │  Vulnerability   │ Severity │  Fixed Version    │
├────────────────┼──────────────────┼──────────┼───────────────────┤
│ openssl        │ CVE-2023-0286    │ CRITICAL │ 1.1.1t-1          │
│ curl           │ CVE-2023-23916   │ CRITICAL │ 7.88.1-1          │
│ libc6          │ CVE-2023-0687    │ HIGH     │ 2.36-9            │
│ ...            │ ...              │ ...      │ ...               │
└────────────────┴──────────────────┴──────────┴───────────────────┘
```

The security team is blocking the release. You need to remediate these vulnerabilities before the deployment deadline.

## The Challenge

Fix the vulnerabilities and implement a process to prevent them in the future. Explain the trade-offs between different remediation strategies.


### Step 1: Update Base Image

```dockerfile
# BEFORE: Old image with vulnerabilities
FROM node:18

# AFTER: Specific patched version
FROM node:18.19.0-bookworm-slim

# Or even better - Alpine (fewer packages = fewer vulns)
FROM node:18.19.0-alpine3.19
```

Check for updates:
```bash
# Find latest tags
docker pull node:18-alpine
trivy image node:18-alpine

# Compare vulnerability counts
trivy image node:18          # ~150 vulns
trivy image node:18-slim     # ~50 vulns
trivy image node:18-alpine   # ~5 vulns
```

### Step 2: Use Multi-Stage Build to Minimize Attack Surface

```dockerfile
# Build stage - has build tools with potential vulns
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage - minimal runtime only
FROM node:18.19.0-alpine3.19 AS production

# Don't include npm, yarn, or build tools
RUN apk add --no-cache tini

WORKDIR /app

# Copy only production dependencies
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./

USER node
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "dist/server.js"]
```

### Step 3: Use Distroless for Maximum Security

```dockerfile
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm ci && npm run build

# Distroless has no shell, no package manager, minimal OS
FROM gcr.io/distroless/nodejs18-debian12

COPY --from=builder /app/dist /app/dist
COPY --from=builder /app/node_modules /app/node_modules

WORKDIR /app
USER nonroot
CMD ["dist/server.js"]
```

**Distroless scan:**
```bash
$ trivy image gcr.io/distroless/nodejs18-debian12
Total: 0 (CRITICAL: 0, HIGH: 0, MEDIUM: 0, LOW: 0)
```

### Step 4: CI/CD Integration

```yaml
# GitHub Actions - fail on critical vulnerabilities
name: Security Scan

on: [push, pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myapp:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'  # Fail pipeline on vulnerabilities

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
```

### Step 5: Automated Base Image Updates

```yaml
# Renovate config - auto-update Dockerfile base images
# renovate.json
{
  "extends": ["config:base"],
  "dockerfile": {
    "enabled": true,
    "fileMatch": ["Dockerfile$", "Dockerfile\\.[a-z]+$"]
  },
  "packageRules": [
    {
      "matchDatasources": ["docker"],
      "matchUpdateTypes": ["patch", "minor"],
      "automerge": true
    }
  ]
}
```


## Vulnerability Remediation Strategies

| Strategy | Effort | Reduction | Trade-offs |
|----------|--------|-----------|------------|
| Update base image tag | Low | 20-50% | May introduce breaking changes |
| Switch to slim variant | Low | 50-70% | Some packages missing |
| Switch to Alpine | Medium | 80-90% | musl libc compatibility issues |
| Use distroless | High | 95%+ | No shell for debugging |
| Multi-stage builds | Medium | 60-80% | Longer Dockerfile |

## Scanning Tools Comparison

```bash
# Trivy (recommended - fast, accurate)
trivy image myapp:latest

# Docker Scout (integrated with Docker Desktop)
docker scout cves myapp:latest

# Grype (Anchore)
grype myapp:latest

# Snyk
snyk container test myapp:latest
```

## Setting Vulnerability Thresholds

```yaml
# .trivyignore - accept known risks
# CVE-2023-XXXX  # Documented: not exploitable in our context
CVE-2023-12345  # False positive: feature not used

# trivy.yaml - policy configuration
severity:
  - CRITICAL
  - HIGH
ignore-unfixed: true
```

<InterviewQuiz
  question="Which base image would typically have the fewest vulnerabilities for a Node.js application?"
  options={[
    "node:18 (Debian-based full image)",
    "node:18-slim (Debian-based minimal)",
    "node:18-alpine (Alpine Linux-based)",
    "gcr.io/distroless/nodejs18 (Google Distroless)"
  ]}
  correctAnswer={3}
  explanation="Distroless images contain only your application and its runtime dependencies - no shell, package manager, or other OS utilities. This dramatically reduces the attack surface and vulnerability count (often to zero OS-level CVEs). Alpine is second-best with ~5-10 vulns, slim has ~50, and full Debian has ~150+. The trade-off with distroless is that you can't shell into the container for debugging, requiring alternative approaches like ephemeral debug containers."
/>

---

### Question 11: Container DNS resolution is failing intermittently. Troubleshoot and fix it.

**Type:** Debugging | **Category:** Networking

## The Scenario

Your microservices are experiencing intermittent failures:

```bash
$ docker logs api-service
Error: getaddrinfo EAI_AGAIN user-service
Error: getaddrinfo ENOTFOUND payment-service
Connection to redis failed: EHOSTUNREACH

# Sometimes it works, sometimes it fails
$ docker exec api-service ping user-service
PING user-service (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.123 ms

# A minute later...
$ docker exec api-service ping user-service
ping: bad address 'user-service'
```

The failures are random - sometimes DNS works, sometimes it doesn't. This is causing cascading failures across your services.

## The Challenge

Diagnose the root cause of intermittent DNS resolution failures and implement a robust fix. Explain how Docker DNS works and common failure modes.


### Understanding Docker DNS

```
Container DNS Resolution Flow:
1. App requests "user-service"
2. Query goes to Docker DNS (127.0.0.11)
3. Docker checks container names in network
4. If not found, forwards to host DNS
5. Response cached (default: 600s)
```

### Step 1: Diagnose DNS Configuration

```bash
# Check container's DNS configuration
docker exec api-service cat /etc/resolv.conf

# Expected output:
nameserver 127.0.0.11  # Docker's embedded DNS
options ndots:0

# If you see external DNS servers, network might be misconfigured
```

### Step 2: Test DNS Resolution

```bash
# Install dig/nslookup if not available
docker exec api-service apk add --no-cache bind-tools

# Test internal service resolution
docker exec api-service nslookup user-service
docker exec api-service dig user-service

# Test external resolution
docker exec api-service nslookup google.com

# Check DNS response time
docker exec api-service time nslookup user-service
```

### Step 3: Common Issues and Fixes

**Issue 1: ndots Causing Slow Resolution**

```bash
# Check ndots setting
docker exec api-service cat /etc/resolv.conf
# options ndots:5  <- This is problematic!
```

With `ndots:5`, queries for "user-service" try:
1. user-service.default.svc.cluster.local (timeout)
2. user-service.svc.cluster.local (timeout)
3. user-service.cluster.local (timeout)
4. user-service.localdomain (timeout)
5. user-service (finally works!)

**Fix:**
```yaml
# docker-compose.yml
services:
  api:
    dns_opt:
      - ndots:1
```

**Issue 2: Docker DNS Server Overwhelmed**

```bash
# Check Docker daemon logs
sudo journalctl -u docker.service | grep -i dns

# Check for DNS-related errors
docker events --filter 'type=network'
```

**Fix:** Increase DNS cache or add local DNS cache:
```yaml
services:
  dnsmasq:
    image: andyshinn/dnsmasq
    cap_add:
      - NET_ADMIN
    command: --cache-size=10000 --log-facility=-

  api:
    dns:
      - dnsmasq  # Use local cache first
```

**Issue 3: Network Driver Issues**

```bash
# Check network details
docker network inspect app-network

# Look for driver errors
docker network ls
docker network inspect bridge
```

**Fix:** Recreate network with explicit configuration:
```bash
# Remove and recreate
docker network rm app-network
docker network create \
  --driver bridge \
  --opt com.docker.network.driver.mtu=1450 \
  app-network
```

### Step 4: Implement Application-Level Resilience

```javascript
// DNS-aware retry logic
const dns = require('dns');
const { promisify } = require('util');
const resolve4 = promisify(dns.resolve4);

async function resolveWithRetry(hostname, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const addresses = await resolve4(hostname);
      return addresses[0];
    } catch (err) {
      if (i === maxRetries - 1) throw err;
      // Exponential backoff
      await new Promise(r => setTimeout(r, 100 * Math.pow(2, i)));
    }
  }
}

// Connection pool with DNS refresh
const pool = mysql.createPool({
  host: 'database-service',
  // Refresh DNS periodically
  dns: {
    ttl: 30,  // Cache DNS for 30 seconds
    refreshInterval: 10000  // Refresh every 10 seconds
  }
});
```

### Step 5: Docker Compose Best Practices

```yaml
version: '3.8'

services:
  api:
    image: api:latest
    networks:
      - backend
    dns_opt:
      - ndots:1
      - timeout:2
      - attempts:3
    depends_on:
      user-service:
        condition: service_healthy

  user-service:
    image: user-service:latest
    networks:
      - backend
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 3

networks:
  backend:
    driver: bridge
    driver_opts:
      com.docker.network.driver.mtu: 1450
```


## DNS Debugging Commands

```bash
# Check DNS server status
docker exec api-service cat /etc/resolv.conf

# Test resolution
docker exec api-service nslookup user-service

# Check network connectivity
docker exec api-service ping -c 3 user-service

# Trace DNS queries
docker exec api-service tcpdump -i eth0 port 53

# Check container's network
docker inspect api-service --format='{{json .NetworkSettings.Networks}}'
```

## Common DNS Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| EAI_AGAIN | DNS server timeout | Increase timeout, add retries |
| ENOTFOUND | Service not on same network | Check network membership |
| Slow resolution | High ndots value | Set ndots:1 |
| Intermittent failures | DNS cache issues | Restart Docker daemon |
| External DNS fails | Network isolation | Check NAT and firewall |

---

### Question 12: Design a CI/CD pipeline for building, testing, and deploying Docker images with proper tagging strategy.

**Type:** Architecture | **Category:** CI/CD & Registry

## The Scenario

Your team needs to set up a CI/CD pipeline for a containerized application. Current problems:

- Developers use `docker build` locally with inconsistent results
- Images are tagged as `latest`, making rollbacks impossible
- No automated testing before deployment
- No vulnerability scanning
- Manual deployments to production
- No way to trace deployed images back to source commits

## The Challenge

Design a complete CI/CD pipeline that builds, tests, scans, and deploys Docker images with a proper tagging strategy. Explain how to ensure reproducible builds and enable reliable rollbacks.


### Step 1: Image Tagging Strategy

```bash
# BAD: Mutable tags
myapp:latest          # Which version? Nobody knows
myapp:dev             # Changes constantly
myapp:stable          # When was this built?

# GOOD: Immutable, traceable tags
myapp:1.2.3                           # Semantic version
myapp:1.2.3-abc1234                   # Version + short SHA
myapp:sha-abc1234def5678              # Full commit SHA
myapp:pr-123                          # Pull request preview
myapp:1.2.3-abc1234-20240115-142030   # Version + SHA + timestamp
```

### Step 2: Complete GitHub Actions Pipeline

```yaml
name: Build, Test, and Deploy

on:
  push:
    branches: [main, develop]
    tags: ['v*']
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ============================================
  # Build and Test
  # ============================================
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      security-events: write

    outputs:
      image_tags: ${{ steps.meta.outputs.tags }}
      image_digest: ${{ steps.build.outputs.digest }}

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata for Docker
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            # Branch name
            type=ref,event=branch
            # PR number
            type=ref,event=pr
            # Semantic version from tag
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            # Git SHA
            type=sha,prefix=sha-
            # Latest for main branch
            type=raw,value=latest,enable=${{ github.ref == 'refs/heads/main' }}

      - name: Build and push
        id: build
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            BUILD_DATE=${{ github.event.head_commit.timestamp }}
            VCS_REF=${{ github.sha }}
            VERSION=${{ steps.meta.outputs.version }}

  # ============================================
  # Security Scan
  # ============================================
  scan:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:sha-${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'

      - name: Upload scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

      - name: Fail on critical vulnerabilities
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:sha-${{ github.sha }}
          format: 'table'
          exit-code: '1'
          severity: 'CRITICAL'

  # ============================================
  # Integration Tests
  # ============================================
  test:
    needs: build
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run integration tests
        run: |
          docker run --network host \
            -e DATABASE_URL=postgres://postgres:test@localhost:5432/test \
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:sha-${{ github.sha }} \
            npm run test:integration

  # ============================================
  # Deploy to Staging
  # ============================================
  deploy-staging:
    needs: [build, scan, test]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: staging

    steps:
      - name: Deploy to Kubernetes
        uses: azure/k8s-deploy@v4
        with:
          manifests: k8s/staging/
          images: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:sha-${{ github.sha }}

  # ============================================
  # Deploy to Production
  # ============================================
  deploy-production:
    needs: deploy-staging
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: Deploy to Kubernetes
        uses: azure/k8s-deploy@v4
        with:
          manifests: k8s/production/
          images: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.ref_name }}
```

### Step 3: Dockerfile with Build Metadata

```dockerfile
# syntax=docker/dockerfile:1.4
FROM node:18-alpine AS builder

ARG BUILD_DATE
ARG VCS_REF
ARG VERSION

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine AS production

# OCI Image Labels (standard metadata)
LABEL org.opencontainers.image.created="${BUILD_DATE}" \
      org.opencontainers.image.revision="${VCS_REF}" \
      org.opencontainers.image.version="${VERSION}" \
      org.opencontainers.image.source="https://github.com/myorg/myapp"

WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

USER node
CMD ["node", "dist/server.js"]
```


## Tagging Strategy Summary

| Trigger | Tag Pattern | Use Case |
|---------|-------------|----------|
| Push to main | `sha-abc1234`, `latest` | Development builds |
| Pull request | `pr-123` | Preview environments |
| Git tag v1.2.3 | `1.2.3`, `1.2`, `1` | Production releases |
| Scheduled | `nightly-20240115` | Nightly builds |

## Rollback Strategy

```bash
# List available versions
docker images myapp --format "{{.Tag}}" | head -10

# Rollback to specific version
kubectl set image deployment/api api=myapp:1.2.2

# Or using Helm
helm rollback api-release 3
```

## Image Signing (Cosign)

```yaml
- name: Sign image with Cosign
  uses: sigstore/cosign-installer@v3

- name: Sign the image
  run: |
    cosign sign --yes \
      ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}@${{ steps.build.outputs.digest }}
```

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*
