# Complete Kubernetes & Container Orchestration Interview Guide

Master Kubernetes with 10 real-world interview questions covering debugging, architecture, security, scaling, and production operations. Practice scenarios that mirror actual senior DevOps/SRE engineer challenges at Fortune 500 companies.

**Companies that ask these questions:** Google | Amazon | Microsoft | Netflix | Spotify | Uber

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | A critical production pod is stuck in CrashLoopBackOff. How ... | Debugging | Pod Lifecycle & Debugging |
| 2 | Design a highly available, multi-region Kubernetes architect... | Architecture | High Availability & Architecture |
| 3 | Implement a zero-downtime deployment strategy for a critical... | Practical | Deployment Strategies |
| 4 | Services can't communicate across namespaces. Debug this net... | Debugging | Networking & Service Discovery |
| 5 | Pods are being OOMKilled in production. How do you diagnose ... | Debugging | Resource Management |
| 6 | Implement proper RBAC for a multi-tenant cluster where teams... | Practical | Security & RBAC |
| 7 | Your application experiences unpredictable traffic spikes. D... | Architecture | Autoscaling |
| 8 | A StatefulSet's pods can't mount their persistent volumes. T... | Debugging | Storage & StatefulSets |
| 9 | Implement a GitOps workflow for deploying applications to mu... | Practical | GitOps & CI/CD |
| 10 | Your production cluster went down. Walk through your disaste... | Architecture | Disaster Recovery |

---

## What You'll Learn

- Debug crashing pods and CrashLoopBackOff errors in production
- Design highly available, multi-region Kubernetes architectures
- Implement zero-downtime rolling updates and rollback strategies
- Troubleshoot networking issues: Service discovery, DNS, Ingress
- Optimize resource requests/limits and prevent OOMKilled errors
- Implement proper RBAC and security policies for multi-tenant clusters
- Design autoscaling strategies: HPA, VPA, Cluster Autoscaler
- Debug persistent volume and storage class issues
- Implement GitOps workflows and CI/CD pipelines for Kubernetes
- Handle disaster recovery and backup/restore strategies

---

## Interview Questions

### Question 1: A critical production pod is stuck in CrashLoopBackOff. How do you diagnose and fix it?

**Type:** Debugging | **Category:** Pod Lifecycle & Debugging

## The Scenario

It's 3 AM and you get paged. Your company's payment processing service—a critical microservice handling thousands of transactions per minute—has been down for 5 minutes. The on-call engineer tried restarting the deployment, but the pods keep crashing.

When you check the cluster, you see:

```bash
kubectl get pods -n payments
NAME                           READY   STATUS             RESTARTS   AGE
payment-processor-6d4f7b-abc   0/1     CrashLoopBackOff   5          3m
payment-processor-6d4f7b-def   0/1     CrashLoopBackOff   5          3m
payment-processor-6d4f7b-ghi   0/1     CrashLoopBackOff   5          3m
```

Every transaction is failing. Revenue is being lost. Your VP of Engineering is awake and watching Slack. **You have 10 minutes to diagnose and fix this.**

## The Challenge

Walk me through your systematic debugging process. What commands would you run, in what order, and why? How would you quickly isolate whether this is an application issue, configuration problem, or infrastructure failure?


### Step 1: Check Recent Changes (First 30 seconds)

Before diving into logs, check what changed:

```bash
# Check recent deployments
kubectl rollout history deployment/payment-processor -n payments

# Check recent events
kubectl get events -n payments --sort-by='.lastTimestamp' | tail -20
```

Why: 80% of production incidents are caused by recent changes. If you see a deployment 5 minutes ago, that's your smoking gun.

**Quick Fix:** If a recent deployment caused this:
```bash
# Immediate rollback
kubectl rollout undo deployment/payment-processor -n payments

# Verify pods are recovering
kubectl get pods -n payments -w
```

### Step 2: Examine Pod Logs (Next 2 minutes)

If rollback doesn't help or there was no recent deployment:

```bash
# Get logs from the crashing pod
kubectl logs payment-processor-6d4f7b-abc -n payments

# If the container crashed before logging anything, check previous instance
kubectl logs payment-processor-6d4f7b-abc -n payments --previous

# Check all container logs if it's a multi-container pod
kubectl logs payment-processor-6d4f7b-abc -n payments --all-containers=true
```

**What to Look For:**
- Application errors: Stack traces, null pointer exceptions, connection errors
- Configuration errors: "Cannot read config file", "Environment variable X not set"
- Dependency failures: "Cannot connect to database", "Redis timeout"
- OOM kills: "Out of memory" or sudden termination with exit code 137

### Step 3: Inspect Pod Description (Next 2 minutes)

```bash
kubectl describe pod payment-processor-6d4f7b-abc -n payments
```

**Critical Sections to Check:**

1. Events Section showing why the pod failed
2. Last State showing the exit code

**Common Exit Codes:**
- 0: Successful exit (shouldn't cause crash)
- 1: Application error (check logs)
- 137: OOMKilled (out of memory)
- 139: Segmentation fault
- 143: Terminated by SIGTERM

### Step 4: Check Dependencies and ConfigMaps/Secrets (Next 2 minutes)

```bash
# Verify ConfigMap exists and has correct data
kubectl get configmap payment-config -n payments -o yaml

# Verify Secrets exist
kubectl get secrets -n payments

# Check if the database/Redis are accessible
kubectl run debug-pod --rm -it --image=busybox -n payments -- sh
# Inside the pod:
nslookup payment-db.payments.svc.cluster.local
wget -O- payment-db.payments.svc.cluster.local:5432
```

### Step 5: Check Resource Limits (Next 1 minute)

```bash
# Check if pods are being OOMKilled
kubectl describe pod payment-processor-6d4f7b-abc -n payments | grep -A 5 "Last State"

# Check current resource usage vs limits
kubectl top pods -n payments
kubectl describe deployment payment-processor -n payments | grep -A 3 "Limits"
```


### Common Root Causes and Fixes

| Symptom | Root Cause | Fix |
|---------|-----------|-----|
| `ErrImagePull` / `ImagePullBackOff` | Wrong image tag or registry auth failure | Check image name, verify image exists, check imagePullSecrets |
| `CrashLoopBackOff` + Exit Code 1 | Application error on startup | Check logs, verify env vars, config files |
| `CrashLoopBackOff` + Exit Code 137 | Pod using more memory than limit | Increase memory limits or fix memory leak |
| `CreateContainerConfigError` | Missing ConfigMap/Secret | Verify ConfigMap/Secret exists and is mounted correctly |
| Pods start but immediately crash | Failed health check or dependency unreachable | Check liveness/readiness probes, verify database connectivity |

### Real-World Example: Database Connection Failure

**Logs show:**
```
Error: connect ECONNREFUSED payment-db:5432
Application failed to start
```

**Root Cause:** Database service name changed from `payment-db` to `payment-database` in a recent update, but the app's environment variable still points to the old name.

**Fix:**
```bash
# Update the deployment's environment variable
kubectl set env deployment/payment-processor -n payments \
  DATABASE_HOST=payment-database.payments.svc.cluster.local

# Verify pods are running
kubectl get pods -n payments -w
```

---

### Question 2: Design a highly available, multi-region Kubernetes architecture for a financial services application.

**Type:** Architecture | **Category:** High Availability & Architecture

## The Scenario

You're the Lead Cloud Architect at a Fortune 500 financial services company. The business is launching a new real-time trading platform that must meet these requirements:

- **99.99% uptime SLA** (less than 53 minutes of downtime per year)
- **Handles 50,000 transactions per second** during market hours
- **Regulatory requirement:** Data must remain in specific regions (US data in US, EU data in EU)
- **RTO (Recovery Time Objective):** 5 minutes
- **RPO (Recovery Point Objective):** Zero data loss
- **Global user base:** Users from North America, Europe, and Asia-Pacific

The CTO asks you: "Design our Kubernetes infrastructure to meet these requirements. We have budget for AWS, GCP, or Azure—your choice."

## The Challenge

Design a comprehensive, production-ready multi-region Kubernetes architecture. Your design must address:

1. **Cluster topology:** How many clusters? Where? Why?
2. **High availability:** How do you ensure 99.99% uptime?
3. **Data residency:** How do you meet regulatory requirements?
4. **Disaster recovery:** What happens if an entire region fails?
5. **Traffic routing:** How do users reach the right region?

Draw the architecture and explain your design decisions.

<JuniorVsSenior
  juniorAnswer="A junior architect might propose using one large Kubernetes cluster spanning multiple regions with a simple load balancer distributing traffic globally and replicated databases without considering data residency. This fails because cross-region latency kills performance, it violates data residency regulations like GDPR, network costs are enormous, there's a single point of failure for the control plane, and it creates a compliance nightmare."
  seniorAnswer="A Principal Engineer designs a multi-cluster, multi-region architecture with active-active setup. The architecture includes three separate regional clusters (US-East, EU-West, AP-Southeast) for data residency and fault isolation, multi-AZ within each region for high availability, active-active traffic routing using AWS Global Accelerator for fast failover, and RDS with cross-region async replication for disaster recovery. This achieves 99.99% uptime with under 2-minute RTO.">

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Global Load Balancer                       │
│          (AWS Global Accelerator / GCP Cloud CDN)            │
│        Routes users to nearest healthy region                │
└────────────┬────────────────────┬────────────────────────────┘
             │                    │
   ┌─────────▼─────────┐  ┌──────▼──────────┐  ┌─────────────┐
   │  US-EAST (Primary)│  │  EU-WEST        │  │  AP-SOUTHEAST│
   │  ┌──────────────┐ │  │  ┌────────────┐ │  │  ┌─────────┐│
   │  │  EKS Cluster │ │  │  │ EKS Cluster│ │  │  │EKS Cluster││
   │  │  Multi-AZ    │ │  │  │  Multi-AZ  │ │  │  │ Multi-AZ ││
   │  │  3 Nodes/AZ  │ │  │  │ 3 Nodes/AZ │ │  │  │3 Nodes/AZ││
   │  └──────────────┘ │  │  └────────────┘ │  │  └─────────┘│
   │                    │  │                 │  │             │
   │  RDS Multi-AZ     │  │  RDS Multi-AZ   │  │ RDS Multi-AZ│
   └───────────────────┘  └─────────────────┘  └─────────────┘
             │                    │                    │
             └────────────────────┴────────────────────┘
                           Cross-Region
                        Async Replication
```

### Key Design Decisions

**1. Three Separate Regional Clusters (Not One Global Cluster)**

Why:
- Data Residency: GDPR requires EU customer data to stay in EU
- Fault Isolation: If one region fails, others continue operating
- Latency: Users connect to geographically closest cluster (50ms vs 200ms)
- Compliance: Financial regulations require data sovereignty

**2. Multi-AZ Within Each Region**

Each cluster (e.g., us-east-1) spans 3 availability zones with 3 worker nodes and 1 control plane per AZ. This provides high availability where if one AZ fails, pods reschedule to healthy AZs, achieves 99.99% SLA (single AZ gives 99.9%, multi-AZ gives 99.99%), and enables zero-downtime deployments by draining one AZ while others handle traffic.

**3. Active-Active Traffic Routing**

Users are routed to their nearest region: New York users go to US-EAST, London users to EU-WEST, and Tokyo users to AP-SOUTHEAST. AWS Global Accelerator provides 10x faster failover than Route53, detects failures in seconds not minutes, uses static anycast IPs with no DNS propagation delays, and routes traffic to healthy regions within 30 seconds automatically.

**4. Database Strategy**

US-EAST has RDS Multi-AZ as the primary with sync replica and async replication to EU-WEST. EU-WEST has an RDS Read Replica that can be promoted to primary with async replication to AP-SOUTHEAST. The automated failover process detects failures within 10 seconds, shifts traffic to EU-WEST in 20 seconds, promotes the EU-WEST replica to primary in 1 minute, for a total RTO under 2 minutes (well under the 5-minute requirement).

</JuniorVsSenior>

### Complete Deployment with Multi-AZ

```yaml
# Deployment with pod anti-affinity to spread across AZs
apiVersion: apps/v1
kind: Deployment
metadata:
  name: trading-api
spec:
  replicas: 9  # 3 per AZ
  template:
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: trading-api
            topologyKey: topology.kubernetes.io/zone
      containers:
      - name: api
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
```

### Cost Optimization

**Estimated Monthly Cost (AWS):**
- EKS Clusters (3): $219/month
- EC2 Nodes (27 instances, m5.2xlarge): ~$7,000/month
- RDS Multi-AZ (3 db.r5.2xlarge): ~$3,000/month
- Global Accelerator: ~$500/month
- Data Transfer: ~$2,000/month
- **Total: ~$12,700/month** (handles 50K TPS with 99.99% uptime)

---

### Question 3: Implement a zero-downtime deployment strategy for a critical microservice handling 10K requests/second.

**Type:** Practical | **Category:** Deployment Strategies

## The Scenario

You're the DevOps Lead at a major e-commerce company. Your checkout microservice handles 10,000 requests per second during peak hours (Black Friday, holiday season). Each request represents real revenue—even 1 second of downtime costs thousands of dollars.

Your team needs to deploy a critical security patch **today**. The deployment includes:
- Updated container image with security fixes
- New environment variables for enhanced monitoring
- Updated database schema (backward-compatible migration)

**The business requirement is clear:** Zero downtime. Not even 1 failed request.

Your current deployment manifest:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: checkout-service
  namespace: production
spec:
  replicas: 20
  selector:
    matchLabels:
      app: checkout-service
  template:
    metadata:
      labels:
        app: checkout-service
    spec:
      containers:
      - name: checkout
        image: company/checkout:v1.2.3
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
```

## The Challenge

Design and implement a **zero-downtime deployment strategy**. Your solution must:

1. **Guarantee zero failed requests** during the deployment
2. **Handle the database migration** safely
3. **Provide instant rollback capability** if something goes wrong
4. **Be automated** (no manual kubectl commands during deployment)

Show the complete deployment configuration and explain each decision.

<JuniorVsSenior
  juniorAnswer="Basic rolling update without proper configuration - just update the image tag and let Kubernetes handle it. No readiness probes, no preStop hooks, default maxUnavailable allows downtime, no PodDisruptionBudget, no database migration strategy, no graceful shutdown period. Result: Failed requests during deployment."
  seniorAnswer="Production-grade zero-downtime deployment using Rolling Update strategy with Readiness Probes, PreStop Hooks, PodDisruptionBudget, and proper resource management. This approach ensures traffic only goes to ready pods, gracefully finishes in-flight requests, prevents too many pods from being down simultaneously, and handles database migrations safely."
>

### Junior Approach: Basic Rolling Update

The junior developer just updates the image tag without proper configuration:

```yaml
spec:
  replicas: 20
  template:
    spec:
      containers:
      - name: checkout
        image: company/checkout:v1.2.4  # Updated
```

Problems with this approach:
- No readiness probes (traffic sent to pods before they're ready)
- No preStop hooks (abrupt connection closures)
- Default maxUnavailable allows downtime
- No PodDisruptionBudget (too many pods can be down)
- No database migration strategy
- No graceful shutdown period
- Result: Failed requests during deployment

### Senior Approach: Production-Ready Zero-Downtime Deployment

This is exactly what senior engineers at Netflix, Amazon, and Uber implement daily. Here's the complete solution:

**Strategy: Rolling Update with Readiness Probes + PreStop Hooks + PodDisruptionBudget**

#### Complete Production-Ready Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: checkout-service
  namespace: production
  annotations:
    # Enable Prometheus monitoring for deployment metrics
    prometheus.io/scrape: "true"
spec:
  replicas: 20

  # Rolling update strategy - this is KEY for zero downtime
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 5           # Create up to 5 extra pods during update
      maxUnavailable: 0     # NEVER allow any pods to be unavailable

  selector:
    matchLabels:
      app: checkout-service

  template:
    metadata:
      labels:
        app: checkout-service
        version: v1.2.4    # New version label for monitoring
      annotations:
        # Force pod restart on configmap/secret change
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}

    spec:
      # Graceful termination - critical for zero downtime
      terminationGracePeriodSeconds: 60

      containers:
      - name: checkout
        image: company/checkout:v1.2.4  # New version
        imagePullPolicy: Always

        ports:
        - name: http
          containerPort: 8080
          protocol: TCP

        # Environment variables
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
        - name: NEW_MONITORING_ENDPOINT
          value: "https://metrics.company.com"

        # Resource limits - prevent OOMKilled during deployment
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"

        # CRITICAL: Readiness probe ensures traffic only goes to ready pods
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          successThreshold: 1
          failureThreshold: 3

        # Liveness probe restarts crashed pods
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3

        # Startup probe for slow-starting applications
        startupProbe:
          httpGet:
            path: /health/startup
            port: 8080
          initialDelaySeconds: 0
          periodSeconds: 5
          timeoutSeconds: 3
          successThreshold: 1
          failureThreshold: 30  # 30 * 5s = 150s max startup time

        # PreStop hook - gracefully finish in-flight requests
        lifecycle:
          preStop:
            exec:
              command:
              - sh
              - -c
              - |
                # Stop accepting new connections
                kill -TERM 1
                # Wait for existing requests to complete (up to 45s)
                sleep 45

---
# PodDisruptionBudget - prevents too many pods from being down
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: checkout-service-pdb
  namespace: production
spec:
  minAvailable: 15  # Always keep at least 15 pods running (out of 20)
  selector:
    matchLabels:
      app: checkout-service

---
# Service - ensures traffic routing
apiVersion: v1
kind: Service
metadata:
  name: checkout-service
  namespace: production
spec:
  type: ClusterIP
  selector:
    app: checkout-service
  ports:
  - name: http
    port: 80
    targetPort: 8080
    protocol: TCP
  # Session affinity for stateful connections (optional)
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 3600

---
# HorizontalPodAutoscaler - handle traffic spikes during deployment
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: checkout-service-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: checkout-service
  minReplicas: 20
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300  # Don't scale down too quickly
```

#### How This Achieves Zero Downtime

**1. maxUnavailable: 0**

This is the most critical setting. Kubernetes will create new pods FIRST (up to maxSurge: 5), wait for them to become ready, and only THEN terminate old pods.

Timeline:
- t=0s: 20 old pods running
- t=10s: 25 pods running (20 old + 5 new starting)
- t=30s: 25 pods running (20 old + 5 new READY)
- t=31s: 20 pods running (15 old + 5 new) - first 5 old pods terminated
- t=60s: 20 pods running (all new) - deployment complete

**2. Readiness Probe - The Traffic Controller**

Kubernetes Service only sends traffic to pods that pass readiness checks. During deployment, new pods start but DON'T receive traffic yet. They perform database migrations and warm up caches. Once /health/ready returns 200 OK, traffic flows to them. Old pods continue handling requests until removed.

Application code for /health/ready:

```javascript
app.get('/health/ready', async (req, res) => {
  try {
    // Check database connectivity
    await db.query('SELECT 1');

    // Check cache is warmed up
    if (!cache.isReady()) {
      return res.status(503).send('Cache not ready');
    }

    // Check critical dependencies
    const redisOk = await redis.ping();
    if (!redisOk) {
      return res.status(503).send('Redis unavailable');
    }

    res.status(200).send('OK');
  } catch (error) {
    res.status(503).send('Not ready');
  }
});
```

**3. PreStop Hook - Graceful Shutdown**

When Kubernetes terminates a pod:
1. PreStop hook runs - tells app to stop accepting new requests
2. 45-second grace period - app finishes in-flight requests
3. Pod removed from Service - no new requests routed to it
4. SIGTERM sent - app shuts down cleanly
5. After terminationGracePeriodSeconds (60s), SIGKILL if still running

Why 45 seconds? Most requests complete in less than 30s, database transactions need time to commit, and this prevents abrupt connection closures.

**4. PodDisruptionBudget - Prevent Mass Termination**

PDB prevents Kubernetes from draining too many nodes at once, cluster autoscaler from removing too many nodes, and admin from accidentally deleting too many pods. If someone runs kubectl drain node-1 (which has 10 pods), PDB will allow draining only if 15 pods remain healthy on other nodes. Otherwise, it blocks the drain operation.

#### Database Migration Strategy

Problem: New version requires database schema changes.

Solution: Backward-compatible migrations with separate job

```yaml
# Run BEFORE deploying new version
apiVersion: batch/v1
kind: Job
metadata:
  name: checkout-db-migration-v124
  namespace: production
spec:
  template:
    spec:
      restartPolicy: OnFailure
      containers:
      - name: migrate
        image: company/checkout:v1.2.4
        command:
        - sh
        - -c
        - |
          # Run backward-compatible migration
          # Old version can still work with new schema
          npm run db:migrate
      initContainers:
      - name: wait-for-db
        image: postgres:15
        command:
        - sh
        - -c
        - until pg_isready -h $DB_HOST; do sleep 2; done
```

Migration best practices:
1. Additive only: Add new columns/tables, don't drop anything
2. Default values: New columns have sensible defaults
3. Two-phase deploy:
   - Phase 1: Deploy schema changes (old app still works)
   - Phase 2: Deploy new app version (uses new columns)

#### Deployment Commands (Automated in CI/CD)

```bash
#!/bin/bash
set -e

echo "1. Running database migration..."
kubectl apply -f db-migration-job.yaml
kubectl wait --for=condition=complete --timeout=300s job/checkout-db-migration-v124 -n production

echo "2. Deploying new version..."
kubectl apply -f deployment.yaml

echo "3. Watching rollout..."
kubectl rollout status deployment/checkout-service -n production --timeout=600s

echo "4. Verifying health..."
kubectl get pods -n production -l app=checkout-service
kubectl get deployment checkout-service -n production

echo "5. Checking metrics..."
curl -s http://checkout-service.production.svc.cluster.local/metrics | grep http_requests_total

echo "✅ Deployment complete!"
```

#### Instant Rollback Strategy

If anything goes wrong during deployment:

```bash
# Immediate rollback to previous version
kubectl rollout undo deployment/checkout-service -n production

# Or rollback to specific revision
kubectl rollout history deployment/checkout-service -n production
kubectl rollout undo deployment/checkout-service --to-revision=42 -n production
```

Why rollback is instant: Old ReplicaSet is still there (not deleted), Kubernetes just scales old ReplicaSet back up, and it takes approximately 30 seconds to restore service.

#### Monitoring During Deployment

```bash
# Watch pod status in real-time
watch kubectl get pods -n production -l app=checkout-service

# Monitor metrics during rollout
kubectl top pods -n production -l app=checkout-service

# Check for errors in logs
kubectl logs -n production -l app=checkout-service --tail=100 -f
```

Key metrics to watch:
- Error rate: Should stay at less than 0.01% during rollout
- Response time: Should not increase
- Pod count: Should maintain minAvailable from PDB
- Request success rate: Should stay at 100%

</JuniorVsSenior>

---

### Question 4: Services can't communicate across namespaces. Debug this networking issue.

**Type:** Debugging | **Category:** Networking & Service Discovery

## The Scenario

You're the Platform Engineer at a SaaS company with a microservices architecture. Your cluster has multiple namespaces for isolation:

- `frontend` namespace: Web application
- `api` namespace: Backend API services
- `data` namespace: Data processing services
- `shared` namespace: Shared services (auth, logging, monitoring)

Everything was working fine until this morning. Now developers are reporting:

```
Error: Failed to connect to auth-service
ECONNREFUSED: Connection refused at auth-service:8080
```

**What's happening:**

1. The `frontend-app` (in `frontend` namespace) cannot connect to `auth-service` (in `shared` namespace)
2. The API service (in `api` namespace) also cannot connect to `auth-service`
3. Services **within** the same namespace can communicate fine
4. No recent code deployments

The `auth-service` is running and healthy:

```bash
$ kubectl get pods -n shared
NAME                            READY   STATUS    RESTARTS   AGE
auth-service-7d9f8c-xyz         1/1     Running   0          2h

$ kubectl get svc -n shared
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
auth-service   ClusterIP   10.100.20.50    <none>        8080/TCP   30d
```

## The Challenge

Debug this cross-namespace networking issue. Walk through your debugging process:

1. What's the first command you run?
2. How do you verify DNS resolution is working?
3. What are the potential causes?
4. How do you fix it?


### Step 1: Verify Service DNS Format

```bash
# WRONG - only works within same namespace
curl http://auth-service:8080

# CORRECT - fully qualified domain name (FQDN)
curl http://auth-service.shared.svc.cluster.local:8080
```

**Kubernetes DNS Format:**
```
<service-name>.<namespace>.svc.cluster.local
```

**Test from frontend pod:**
```bash
kubectl exec -it frontend-app-xyz -n frontend -- sh

# This will FAIL (different namespace)
$ curl http://auth-service:8080
curl: (6) Could not resolve host: auth-service

# This will WORK (FQDN)
$ curl http://auth-service.shared.svc.cluster.local:8080
{"status":"ok"}
```

### Step 2: Test DNS Resolution

```bash
# Exec into a pod in the frontend namespace
kubectl exec -it frontend-app-xyz -n frontend -- sh

# Test DNS resolution
$ nslookup auth-service
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

nslookup: can't resolve 'auth-service'

# Test FQDN resolution
$ nslookup auth-service.shared.svc.cluster.local
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      auth-service.shared.svc.cluster.local
Address 1: 10.100.20.50 auth-service.shared.svc.cluster.local
```

This confirms DNS works, but the application is using the wrong service name.

### Step 3: Check Network Policies

Even with correct DNS, traffic might be blocked by NetworkPolicies:

```bash
# Check if NetworkPolicies exist
kubectl get networkpolicies -n shared
kubectl get networkpolicies -n frontend

# Describe the policy
kubectl describe networkpolicy -n shared
```


### Root Causes and Solutions

**Root Cause #1: Incorrect Service DNS Name (Most Common)**

**Problem:** Application uses short name `auth-service` instead of FQDN.

```javascript
// ❌ WRONG - Only works in same namespace
const authUrl = 'http://auth-service:8080';

// ✅ CORRECT - Works cross-namespace
const authUrl = 'http://auth-service.shared.svc.cluster.local:8080';

// ✅ ALSO CORRECT - Shorter form (svc.cluster.local is default)
const authUrl = 'http://auth-service.shared:8080';
```

**Fix: Update application configuration**

```yaml
# ConfigMap for frontend app
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-config
  namespace: frontend
data:
  AUTH_SERVICE_URL: "http://auth-service.shared.svc.cluster.local:8080"
```

**Root Cause #2: NetworkPolicy Blocking Traffic**

**Problem:** NetworkPolicy blocks cross-namespace traffic.

**Solution: Allow ingress from specific namespaces**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-frontend-and-api
  namespace: shared
spec:
  podSelector:
    matchLabels:
      app: auth-service
  policyTypes:
  - Ingress
  ingress:
  # Allow from frontend namespace
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend
    ports:
    - protocol: TCP
      port: 8080
  # Allow from api namespace
  - from:
    - namespaceSelector:
        matchLabels:
          name: api
    ports:
    - protocol: TCP
      port: 8080
  # Allow from within same namespace
  - from:
    - podSelector: {}
```

**Important:** Namespaces must have labels for namespaceSelector to work:

```bash
# Add labels to namespaces
kubectl label namespace frontend name=frontend
kubectl label namespace api name=api
kubectl label namespace shared name=shared
```

### Complete Working Example

**Correct setup for cross-namespace communication:**

```yaml
# 1. Shared namespace with labels
apiVersion: v1
kind: Namespace
metadata:
  name: shared
  labels:
    name: shared

---
# 2. Auth service in shared namespace
apiVersion: v1
kind: Service
metadata:
  name: auth-service
  namespace: shared
spec:
  selector:
    app: auth-service
  ports:
  - name: http
    port: 8080
    targetPort: 8080

---
# 3. Auth service deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-service
  namespace: shared
spec:
  replicas: 3
  selector:
    matchLabels:
      app: auth-service
  template:
    metadata:
      labels:
        app: auth-service
    spec:
      containers:
      - name: auth
        image: company/auth-service:v1.0
        ports:
        - containerPort: 8080

---
# 4. NetworkPolicy allowing cross-namespace traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-app-namespaces
  namespace: shared
spec:
  podSelector:
    matchLabels:
      app: auth-service
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend
    - namespaceSelector:
        matchLabels:
          name: api

---
# 5. Frontend app using FQDN
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-config
  namespace: frontend
data:
  # Use FQDN for cross-namespace communication
  AUTH_URL: "http://auth-service.shared.svc.cluster.local:8080"
```

### Testing Cross-Namespace Communication

```bash
# Test DNS resolution from frontend namespace
kubectl run test-pod --rm -it --image=busybox -n frontend -- sh

# Test short name (will fail)
$ nslookup auth-service
** server can't find auth-service: NXDOMAIN

# Test FQDN (will succeed)
$ nslookup auth-service.shared.svc.cluster.local
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local
Name:      auth-service.shared.svc.cluster.local
Address 1: 10.100.20.50

# Test actual HTTP connection
$ wget -O- http://auth-service.shared.svc.cluster.local:8080/health
{"status":"healthy"}
```


---

### Quick Check

**You have a service database in namespace data that needs to be accessed by a pod in namespace backend. Which service URL will work correctly?**

   A. http://database:5432
-> B. **http://database.data:5432**
   C. http://database.backend:5432
   D. http://data.database:5432

<details>
<summary>See Answer</summary>

The correct Kubernetes DNS format for cross-namespace service access is service-name.namespace with optional svc.cluster.local suffix. Option A (http://database:5432) only works if the service is in the same namespace as the pod, and since the pod is in backend namespace, this would try to find database.backend.svc.cluster.local. Option B (http://database.data:5432) is correct as it expands to database.data.svc.cluster.local where the service database is in namespace data. Option C would look for a service named database in the backend namespace, not data. Option D has the service name and namespace reversed since Kubernetes DNS format is service.namespace, not namespace.service.

</details>

---

### Question 5: Pods are being OOMKilled in production. How do you diagnose and prevent this?

**Type:** Debugging | **Category:** Resource Management

## The Scenario

It's Monday morning. Your analytics service—which processes customer data for reporting—keeps crashing. The on-call logs show pods restarting every 10-15 minutes during peak hours.

When you check the cluster:

```bash
$ kubectl get pods -n analytics
NAME                           READY   STATUS      RESTARTS   AGE
analytics-worker-7d9f8c-abc    0/1     OOMKilled   12         45m
analytics-worker-7d9f8c-def    1/1     Running     8          45m
analytics-worker-7d9f8c-ghi    0/1     OOMKilled   10         45m
```

The logs don't show application errors—pods just suddenly restart. Users are complaining that reports are incomplete or missing data. Your VP of Product is asking for an ETA on the fix.

Current deployment configuration:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: analytics-worker
  namespace: analytics
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: worker
        image: company/analytics-worker:v2.1
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "512Mi"  # Same as request
            cpu: "1000m"
```

## The Challenge

1. **Diagnose:** Prove that OOMKilled is actually the issue and identify the memory consumption pattern
2. **Immediate fix:** Get the service stable ASAP
3. **Root cause:** Why is memory usage increasing?
4. **Long-term solution:** Prevent this from happening again

Walk through your complete debugging and remediation process.


### Wrong Approach: Just Increase Memory

The wrong approach is to blindly increase the memory limit:

```yaml
resources:
  limits:
    memory: "4Gi"  # Just make it bigger
```

Problems with this approach:
- Doesn't address root cause (memory leak still exists)
- Wastes cluster resources
- Problem will return when memory reaches 4Gi
- No monitoring or alerting setup
- Doesn't understand WHY pods are OOMKilled

### Right Approach: Systematic Debugging and Comprehensive Solution

This is one of the most common production issues in Kubernetes. Here's how senior SREs handle it:

#### Phase 1: Confirm OOMKilled (30 seconds)

```bash
# Check pod status
kubectl get pods -n analytics

# Describe pod to see exit code
kubectl describe pod analytics-worker-7d9f8c-abc -n analytics

# Look for this in the output:
Last State:     Terminated
  Reason:       OOMKilled
  Exit Code:    137
  Started:      Mon, 15 Jan 2024 09:00:00 +0000
  Finished:     Mon, 15 Jan 2024 09:12:34 +0000
```

Exit Code 137 = 128 + 9 (SIGKILL) - This is the definitive proof of OOMKilled.

#### Phase 2: Check Resource Metrics (Next 2 minutes)

```bash
# Check current memory usage across all pods
kubectl top pods -n analytics

NAME                           CPU(cores)   MEMORY(bytes)
analytics-worker-7d9f8c-abc    450m         498Mi
analytics-worker-7d9f8c-def    380m         512Mi  # At limit!
analytics-worker-7d9f8c-ghi    520m         501Mi

# Check node capacity
kubectl top nodes

# Get detailed resource info
kubectl describe node <node-name> | grep -A 5 "Allocated resources"
```

#### Phase 3: Analyze Historical Memory Usage

If you have Prometheus + Grafana, run these queries:

```promql
# Memory usage over time
container_memory_usage_bytes{
  namespace="analytics",
  pod=~"analytics-worker.*"
}

# Memory usage as % of limit
(container_memory_usage_bytes / container_spec_memory_limit_bytes) * 100
```

What to look for:
- Gradual increase: Memory leak in application
- Sudden spikes: Processing large datasets
- Cyclic pattern: Batch jobs running periodically

#### Phase 4: Examine Application Logs

```bash
# Check logs from the crashed pod
kubectl logs analytics-worker-7d9f8c-abc -n analytics --previous

# Look for memory-related errors before crash
# Common patterns:
# - "heap out of memory"
# - "cannot allocate memory"
# - Processing large files: "loading 10GB CSV file"
```

#### Immediate Fix: Increase Memory Limits

Short-term solution (deploy immediately):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: analytics-worker
  namespace: analytics
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: worker
        image: company/analytics-worker:v2.1
        resources:
          requests:
            memory: "512Mi"   # Request stays same (guaranteed memory)
            cpu: "500m"
          limits:
            memory: "2Gi"     # Increased 4x (max burst capacity)
            cpu: "2000m"
```

Why this works:
- Gives pods more headroom during memory spikes
- Prevents OOMKilled during peak processing
- Pods can burst above request but stay within limit

Deploy the fix:

```bash
kubectl apply -f deployment.yaml
kubectl rollout status deployment/analytics-worker -n analytics
kubectl get pods -n analytics -w
```

#### Understanding Kubernetes QoS Classes

Kubernetes assigns pods to QoS (Quality of Service) classes based on resources:

**1. Guaranteed (Highest Priority)**

```yaml
resources:
  requests:
    memory: "1Gi"
    cpu: "1000m"
  limits:
    memory: "1Gi"  # Same as request
    cpu: "1000m"   # Same as request
```

- Gets evicted last
- Best for critical workloads
- But: No burst capacity!

**2. Burstable (Medium Priority)**

```yaml
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "2Gi"   # Higher than request
    cpu: "2000m"
```

- Can burst above request
- Good for variable workloads
- Gets evicted after BestEffort

**3. BestEffort (Lowest Priority - AVOID)**

```yaml
resources: {}  # No requests or limits
```

- Gets evicted first
- Never use in production!

Our fix uses Burstable QoS:
- Guaranteed 512Mi (request)
- Can burst to 2Gi (limit) during processing

#### Root Cause Analysis: Memory Leak Investigation

Check application code for memory leaks:

```javascript
// ❌ MEMORY LEAK - Array grows forever
const processedRecords = [];

async function processData() {
  while (true) {
    const batch = await fetchNextBatch();
    processedRecords.push(...batch);  // Never cleared!
    await generateReport(processedRecords);
  }
}

// ✅ FIX - Clear array after processing
const processedRecords = [];

async function processData() {
  while (true) {
    const batch = await fetchNextBatch();
    processedRecords.push(...batch);
    await generateReport(processedRecords);
    processedRecords.length = 0;  // Clear memory
  }
}
```

Common memory leak patterns:
1. Event listeners not removed
2. Caching without eviction policy
3. Large objects kept in memory
4. Database connections not closed
5. Timers/intervals not cleared

Node.js specific debugging:

```bash
# Get heap snapshot from running pod
kubectl exec -it analytics-worker-7d9f8c-abc -n analytics -- \
  node --expose-gc --inspect=0.0.0.0:9229 app.js

# Port-forward to access debugger
kubectl port-forward analytics-worker-7d9f8c-abc 9229:9229 -n analytics

# Open Chrome DevTools → Memory tab → Take heap snapshot
```

#### Long-Term Solutions

**1. Implement Memory Monitoring with Alerts**

```yaml
# Prometheus alert rule
groups:
- name: kubernetes-memory
  rules:
  - alert: PodHighMemoryUsage
    expr: |
      (container_memory_usage_bytes / container_spec_memory_limit_bytes) > 0.9
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Pod {{ $labels.pod }} is using > 90% memory"
      description: "Memory usage: {{ $value | humanizePercentage }}"
```

**2. Set Appropriate Resource Requests and Limits**

Best practice formula:

```
Request = Average usage + 20% buffer
Limit = Peak usage + 30% buffer
```

Example calculation:

```
Average memory usage: 400Mi (from metrics)
Request: 400Mi * 1.2 = 480Mi → Round to 512Mi

Peak usage during processing: 1.5Gi (from metrics)
Limit: 1.5Gi * 1.3 = 1.95Gi → Round to 2Gi
```

**3. Implement Horizontal Pod Autoscaling**

Instead of making pods bigger, make more pods:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: analytics-worker-hpa
  namespace: analytics
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: analytics-worker
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70  # Scale when avg pod uses >70% memory
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 100  # Double pods at once if needed
        periodSeconds: 60
```

**4. Optimize Application Memory Usage**

```javascript
// Process data in smaller chunks instead of all at once
async function processDataOptimized() {
  const CHUNK_SIZE = 1000;

  while (true) {
    // Fetch small batch
    const batch = await fetchNextBatch(CHUNK_SIZE);
    if (batch.length === 0) break;

    // Process batch
    await processBatch(batch);

    // Batch is garbage collected after loop iteration
  }
}

// Use streams for large files
const fs = require('fs');
const readline = require('readline');

async function processLargeFile(filePath) {
  const fileStream = fs.createReadStream(filePath);
  const rl = readline.createInterface({
    input: fileStream,
    crlfDelay: Infinity
  });

  for await (const line of rl) {
    // Process one line at a time
    await processLine(line);
    // Memory is released after each line
  }
}
```

**5. Enable Vertical Pod Autoscaler (VPA) for Automatic Tuning**

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: analytics-worker-vpa
  namespace: analytics
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: analytics-worker
  updatePolicy:
    updateMode: "Auto"  # Automatically adjust requests/limits
  resourcePolicy:
    containerPolicies:
    - containerName: worker
      minAllowed:
        memory: "512Mi"
      maxAllowed:
        memory: "4Gi"
      controlledResources: ["memory"]
```

VPA will:
- Monitor actual memory usage
- Automatically adjust requests/limits
- Prevent OOMKills by increasing limits proactively

#### Production Checklist: Preventing OOMKilled

- Set memory requests based on average usage + buffer
- Set memory limits based on peak usage + buffer
- Use Burstable QoS (requests &lt; limits) for variable workloads
- Implement memory usage alerts (> 80% for warning, > 90% for critical)
- Enable HPA to scale out instead of up
- Consider VPA for automatic resource tuning
- Profile application for memory leaks
- Process large datasets in chunks/streams
- Monitor with Prometheus + Grafana
- Test under production-like load before deploying

---

### Question 6: Implement proper RBAC for a multi-tenant cluster where teams should not access each other's resources.

**Type:** Practical | **Category:** Security & RBAC

## The Scenario

You're the Platform Security Lead at a fast-growing tech company. Your company uses a shared Kubernetes cluster for multiple product teams:

- **Team Alpha:** E-commerce platform (namespace: `alpha-prod`, `alpha-dev`)
- **Team Beta:** Analytics dashboard (namespace: `beta-prod`, `beta-dev`)
- **Team Gamma:** Internal tools (namespace: `gamma-prod`, `gamma-dev`)

**Current problem:** All developers have cluster-admin access. Last week:
- A Team Alpha developer accidentally deleted Team Beta's production database
- A Team Gamma engineer viewed Team Alpha's secrets containing API keys
- Your security audit failed due to insufficient access controls

**Your CISO mandates:**

1. **Principle of Least Privilege:** Teams can only access their own namespaces
2. **Role Separation:** Developers can deploy, but cannot view secrets in production
3. **Admin Access:** Only platform team has cluster-wide admin rights
4. **Compliance:** All access must be auditable (who did what, when)

## The Challenge

Design and implement a complete RBAC (Role-Based Access Control) strategy that:

1. Isolates teams to their own namespaces
2. Differentiates between dev and prod permissions
3. Provides read-only access for on-call engineers
4. Allows platform team to manage everything

<JuniorVsSenior
  juniorAnswer="I'll create a single role for all developers and bind it to everyone using ClusterRoleBinding with the built-in 'edit' role. This gives everyone access but violates least privilege."
  seniorAnswer="Enterprise-grade RBAC requires a hierarchy: Cluster Admins → Namespace Admins → Developers → Viewers. Use ClusterRoles for reusability, bind with RoleBindings for namespace-specific access.">

**RBAC Hierarchy Design:**

```
Cluster Admins (Platform Team)
    ↓
Namespace Admins (Team Leads)
    ↓
Developers (Read/Write in dev, Deploy in prod)
    ↓
Viewers (Read-only for on-call)
```

### Step 1: Create Namespaces with Labels

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: alpha-prod
  labels:
    team: alpha
    environment: production
---
apiVersion: v1
kind: Namespace
metadata:
  name: alpha-dev
  labels:
    team: alpha
    environment: development
```

### Step 2: Define ClusterRoles (Reusable Across Namespaces)

```yaml
# Role for developers in dev environments (full access)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: developer-dev-role
rules:
- apiGroups: ["", "apps", "batch"]
  resources: [pods, deployments, services, configmaps, jobs]
  verbs: ["*"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]  # Can view, not edit
- apiGroups: [""]
  resources: ["pods/log", "pods/exec"]
  verbs: ["get", "create"]  # For debugging
---
# Role for developers in production (limited access)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: developer-prod-role
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "update", "patch"]  # No delete!
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
# CANNOT view secrets or exec in production
```

### Step 3: Create RoleBindings (Namespace-Specific)

```yaml
# Team Alpha: Developers in dev environment
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: alpha-developers-dev
  namespace: alpha-dev
subjects:
- kind: Group
  name: alpha-developers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: developer-dev-role
  apiGroup: rbac.authorization.k8s.io
---
# Team Alpha: Developers in production (limited)
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: alpha-developers-prod
  namespace: alpha-prod
subjects:
- kind: Group
  name: alpha-developers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: developer-prod-role
  apiGroup: rbac.authorization.k8s.io
```

### Step 4: Platform Team Cluster-Wide Access

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: platform-team-admin
subjects:
- kind: Group
  name: platform-team
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

### Testing RBAC Permissions

```bash
# Test as a developer from Team Alpha
kubectl auth can-i get pods -n alpha-dev --as bob@company.com
# yes

kubectl auth can-i delete deployments -n alpha-prod --as bob@company.com
# no

kubectl auth can-i get pods -n beta-dev --as bob@company.com
# no (Cannot access Team Beta's namespace)

kubectl auth can-i get secrets -n alpha-prod --as bob@company.com
# no (Developers cannot view production secrets)
```

### RBAC Best Practices

- Use Groups instead of individual users
- Define ClusterRoles (reusable), bind with RoleBindings (namespace-specific)
- Principle of Least Privilege (start minimal, add as needed)
- Separate dev and prod permissions
- Never give direct Secret access in production
- Enable audit logging for compliance
- Regular access reviews (quarterly)
</JuniorVsSenior>


---

### Quick Check

**A developer is bound to a Role that allows get, list, watch on pods. Can they run kubectl exec -it my-pod -- /bin/sh?**

   A. Yes, because exec is covered by the get verb
   B. Yes, because they have watch permission which includes exec
-> C. **No, they would need explicit permission on the pods/exec resource**
   D. Yes, all developers should be able to exec into pods by default

<details>
<summary>See Answer</summary>

In Kubernetes RBAC, pods/exec is a separate subresource that requires explicit permission. The verbs get, list, and watch on pods only allow viewing pod details. To allow kubectl exec, you need: resources: pods/exec with verbs: create. This is a security feature - exec gives shell access to containers.

</details>

---

### Question 7: Your application experiences unpredictable traffic spikes. Design a comprehensive autoscaling strategy.

**Type:** Architecture | **Category:** Autoscaling

## The Scenario

You're the Cloud Infrastructure Architect at a news media company. Your application has highly unpredictable traffic:

- **Normal traffic:** 1,000 requests/second (10 pods sufficient)
- **Breaking news events:** 50,000+ requests/second (need 200+ pods)
- **Daily pattern:** Traffic spikes at 8 AM, 12 PM, 6 PM
- **Unpredictable spikes:** Major news events can happen anytime

**Current problems:**
- Manual scaling is too slow—by the time engineers add pods, the spike is over
- Over-provisioning wastes money—paying for 200 pods 24/7 costs $50K/month
- Under-provisioning causes crashes—site went down during last major event

**Your CEO's requirements:**
1. Handle traffic spikes within 60 seconds
2. Scale down to save costs during low traffic
3. Maintain 99.9% uptime
4. Keep infrastructure costs under $15K/month

## The Challenge

Design a comprehensive autoscaling strategy using:
1. **Horizontal Pod Autoscaler (HPA)** - Scale pods based on metrics
2. **Vertical Pod Autoscaler (VPA)** - Right-size pod resources
3. **Cluster Autoscaler** - Add/remove nodes as needed

Explain when to use each, how they work together, and provide complete configurations.

<JuniorVsSenior
  juniorAnswer="A junior engineer might use basic CPU-based HPA with defaults, set aggressive scaling without understanding behavior, ignore cluster capacity leading to pending pods, and not configure PodDisruptionBudgets causing downtime. This fails because CPU alone doesn't reflect application load, aggressive scaling causes pod thrashing, pending pods mean requests fail during spikes, and there's no protection during node maintenance."
  seniorAnswer="A senior architect implements a comprehensive three-layer autoscaling strategy: Layer 1 is HPA scaling pods based on CPU, memory, and custom metrics like requests per second; Layer 2 is Cluster Autoscaler adding nodes when pods can't be scheduled; Layer 3 is VPA right-sizing pod resource requests. The HPA uses multiple metrics with behavior policies controlling scale-up (immediate, 100% increase) and scale-down (gradual, 5-minute stabilization). This achieves 89% cost reduction while maintaining 99.9% uptime.">

### Complete HPA with Multiple Metrics

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: news-app-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: news-app
  minReplicas: 10        # Always keep at least 10 pods (handle baseline)
  maxReplicas: 200       # Never exceed 200 pods (cost control)

  # Multiple metrics - scale based on whichever hits threshold first
  metrics:
  # Scale based on CPU utilization
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # Scale when avg CPU > 70%

  # Scale based on memory utilization
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80  # Scale when avg memory > 80%

  # Scale based on custom metric (requests per second)
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "100"  # Scale when pod handles > 100 RPS

  # Scaling behavior - control how fast to scale up/down
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0  # Scale up immediately (no delay)
      policies:
      - type: Percent
        value: 100  # Double the pods at once if needed (10 → 20 → 40 → 80)
        periodSeconds: 15
      - type: Pods
        value: 20   # Or add 20 pods at once, whichever is higher
        periodSeconds: 15
      selectPolicy: Max  # Use the policy that adds more pods

    scaleDown:
      stabilizationWindowSeconds: 300  # Wait 5 min before scaling down
      policies:
      - type: Percent
        value: 50  # Remove max 50% of pods at once (slow scale-down)
        periodSeconds: 60
      - type: Pods
        value: 5   # Or remove 5 pods, whichever is lower
        periodSeconds: 60
      selectPolicy: Min  # Use the policy that removes fewer pods
```

### How HPA Works

The HPA calculation works as follows: it checks current state (10 pods at 90% CPU), calculates target replicas as current multiplied by current utilization divided by desired utilization (10 times 90 divided by 70 equals 13 pods rounded), scales to 13 pods, then recalculates every 15 seconds based on latest metrics.

### Cluster Autoscaler Configuration

```yaml
# AWS EKS Cluster Autoscaler
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  replicas: 1
  template:
    spec:
      serviceAccountName: cluster-autoscaler
      containers:
      - name: cluster-autoscaler
        image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.27.0
        command:
        - ./cluster-autoscaler
        - --cloud-provider=aws
        - --namespace=kube-system
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled
        - --balance-similar-node-groups
        - --skip-nodes-with-system-pods=false
        - --scale-down-enabled=true
        - --scale-down-delay-after-add=10m
        - --scale-down-unneeded-time=10m
        - --scale-down-utilization-threshold=0.5
```

### VPA for Right-Sizing

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: news-app-vpa
  namespace: production
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: news-app
  updatePolicy:
    updateMode: "Recommender"  # Only recommend, don't auto-apply
  resourcePolicy:
    containerPolicies:
    - containerName: news-app
      minAllowed:
        cpu: "100m"
        memory: "128Mi"
      maxAllowed:
        cpu: "4000m"
        memory: "8Gi"
```

</JuniorVsSenior>

### Cost Optimization Strategy

**Current costs vs optimized:**

```
Without autoscaling (static 200 pods):
- Nodes: 40 m5.2xlarge on-demand = $0.38 * 40 * 730 hours = $11,096/month
- Over-provisioned 23 hours/day = $10,000 wasted

With autoscaling:
- Baseline: 5 nodes * $0.38 * 730 = $1,387/month
- Spike hours: +45 nodes * 1 hour/day * 30 days = +$513/month
- Spot savings (80% spot): Additional savings of 70% = Total ~$1,200/month

Savings: $11,096 - $1,200 = $9,896/month (89% cost reduction!)
```

### Monitoring and Alerting

```yaml
# Prometheus alert for autoscaling issues
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: autoscaling-alerts
  namespace: monitoring
spec:
  groups:
  - name: autoscaling
    rules:
    - alert: HPAMaxedOut
      expr: |
        kube_horizontalpodautoscaler_status_current_replicas >=
        kube_horizontalpodautoscaler_spec_max_replicas
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "HPA {{ $labels.horizontalpodautoscaler }} reached max replicas"
        description: "Consider increasing maxReplicas or adding more node capacity"

    - alert: ClusterAutoscalerFailing
      expr: cluster_autoscaler_failed_scale_ups_total > 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Cluster Autoscaler cannot add nodes"
        description: "Check ASG limits and AWS quotas"
```

---

### Question 8: A StatefulSet's pods can't mount their persistent volumes. Troubleshoot and fix the issue.

**Type:** Debugging | **Category:** Storage & StatefulSets

## The Scenario

You're the Site Reliability Engineer at a fintech company running a PostgreSQL database cluster using StatefulSets. It's 9 AM Monday and you receive alerts:

```
CRITICAL: Database pods failing to start
3 pods stuck in "Pending" state
Users cannot access account data
```

When you check the cluster:

```bash
$ kubectl get pods -n database
NAME          READY   STATUS    RESTARTS   AGE
postgres-0    1/1     Running   0          5d
postgres-1    1/1     Running   0          5d
postgres-2    0/1     Pending   0          15m
postgres-3    0/1     Pending   0          15m
postgres-4    0/1     Pending   0          15m
```

You were trying to scale from 2 to 5 replicas for increased capacity. Pods 2-4 won't start.

```bash
$ kubectl describe pod postgres-2 -n database
Events:
  Warning  FailedScheduling  5m  persistentvolumeclaim "data-postgres-2" not found
  Warning  FailedMount       3m  MountVolume.SetUp failed for volume "pvc-xyz" :
           rpc error: code = DeadlineExceeded desc = context deadline exceeded
```

Your VP of Engineering needs the database scaled up **today** for a product launch tomorrow.

## The Challenge

Debug and fix this persistent volume issue. Walk through:

1. How do you diagnose PVC/PV binding problems?
2. What are the common causes of volume mount failures?
3. How do you verify the storage provisioner is working?
4. What's your complete fix and validation process?


### Phase 1: Check PVC Status

```bash
# List all PVCs in the namespace
kubectl get pvc -n database

NAME              STATUS    VOLUME    CAPACITY   STORAGECLASS   AGE
data-postgres-0   Bound     pv-001    100Gi      fast-ssd       5d
data-postgres-1   Bound     pv-002    100Gi      fast-ssd       5d
data-postgres-2   Pending   -         -          fast-ssd       15m
data-postgres-3   Pending   -         -          fast-ssd       15m
data-postgres-4   Pending   -         -          fast-ssd       15m

# Describe the pending PVC to see events
kubectl describe pvc data-postgres-2 -n database

Events:
  Warning  ProvisioningFailed  1m  failed to provision volume:
           StorageQuota exceeded for StorageClass fast-ssd
```

This reveals the problem: Storage quota exceeded!

### Phase 2: Check StorageClass Configuration

```bash
# Get StorageClass details
kubectl get storageclass fast-ssd -o yaml
```

### Phase 3: Check Persistent Volumes

```bash
# List all PVs
kubectl get pv

NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM
pv-001   100Gi      RWO            Retain           Bound       database/data-postgres-0
pv-002   100Gi      RWO            Retain           Bound       database/data-postgres-1
pv-003   100Gi      RWO            Retain           Released    database/old-postgres-2
pv-004   100Gi      RWO            Retain           Released    database/old-postgres-3
```

Key observation: PVs exist but are in "Released" state from previous deletions!

### Phase 4: Check Storage Provisioner Logs

```bash
# For AWS EBS CSI driver
kubectl logs -n kube-system -l app=ebs-csi-controller

ERROR: VolumeQuotaExceeded: Maximum number of volumes (20) reached for instance type m5.xlarge
ERROR: Failed to create volume: RequestLimitExceeded

# Check AWS quotas
aws service-quotas get-service-quota \
  --service-code ec2 \
  --quota-code L-D18FCD1D  # EBS volume quota

Quota: 20 volumes per instance
Current usage: 20 volumes
```


### Root Causes and Solutions

**Root Cause #1: Released PVs Not Reclaimed**

**Problem:** Old PVs are stuck in "Released" state after StatefulSet pods were deleted.

```bash
# PVs in Released state still hold data and can't be reused
kubectl get pv pv-003 -o yaml

status:
  phase: Released  # Not Available!
spec:
  claimRef:
    name: data-postgres-2  # Still references old claim
    namespace: database
```

**Solution: Manually reclaim the volumes**

```bash
# Option 1: Patch the PV to remove claimRef (if data not needed)
kubectl patch pv pv-003 -p '{"spec":{"claimRef": null}}'
kubectl patch pv pv-004 -p '{"spec":{"claimRef": null}}'

# Verify PVs are now Available
kubectl get pv
NAME     STATUS      CLAIM
pv-003   Available   -
pv-004   Available   -

# Option 2: Delete and recreate PV (if using dynamic provisioning)
kubectl delete pv pv-003 pv-004
# Dynamic provisioner will create new volumes
```

**Root Cause #2: AWS Volume Quota Exceeded**

**Problem:** Instance has reached max EBS volumes (20 for m5.xlarge).

**Solution: Use larger instance types or increase quota**

```bash
# Increase AWS Service Quota
aws service-quotas request-service-quota-increase \
  --service-code ec2 \
  --quota-code L-D18FCD1D \
  --desired-value 50

# OR: Use nodes with higher volume limits
# m5.xlarge: 20 volumes
# m5.2xlarge: 27 volumes
# m5.4xlarge: 27 volumes
# c5.9xlarge: 50 volumes
```

### Complete Working StatefulSet with Persistent Storage

```yaml
---
# StorageClass for database workloads
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: database-storage
provisioner: ebs.csi.aws.com  # AWS EBS CSI driver
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  encrypted: "true"
  kmsKeyId: "arn:aws:kms:us-east-1:123456789:key/abc-123"
volumeBindingMode: Immediate
allowVolumeExpansion: true
reclaimPolicy: Retain  # Don't delete PV when PVC is deleted

---
# StatefulSet for PostgreSQL
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: database
spec:
  serviceName: postgres-headless
  replicas: 5
  selector:
    matchLabels:
      app: postgres

  # Volume claim templates - creates PVC for each pod
  volumeClaimTemplates:
  - metadata:
      name: data
      labels:
        app: postgres
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: database-storage
      resources:
        requests:
          storage: 100Gi

  template:
    metadata:
      labels:
        app: postgres
    spec:
      # Ensure pods spread across availability zones
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: postgres
            topologyKey: topology.kubernetes.io/zone

      containers:
      - name: postgres
        image: postgres:15-alpine
        ports:
        - containerPort: 5432
          name: postgres

        # Mount the persistent volume
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
          subPath: pgdata  # Use subdirectory to avoid permission issues

        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata

        resources:
          requests:
            memory: "4Gi"
            cpu: "2000m"
          limits:
            memory: "8Gi"
            cpu: "4000m"

        # Probes for health checking
        livenessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - postgres
          initialDelaySeconds: 30
          periodSeconds: 10

        readinessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - postgres
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Storage Troubleshooting Checklist

```
✓ Check PVC status: kubectl get pvc -n <namespace>
✓ Describe PVC for events: kubectl describe pvc <name>
✓ Check PV availability: kubectl get pv
✓ Verify StorageClass exists: kubectl get storageclass
✓ Check storage provisioner pods running
✓ Review provisioner logs for errors
✓ Verify cloud provider quotas (EBS volume limits, IOPS, etc.)
✓ Check IAM permissions for CSI driver
✓ Ensure nodes have capacity for new volumes
✓ Test with simple PVC/Pod before StatefulSet
✓ Check for Released PVs that need reclaiming
```


---

### Quick Check

**You delete a StatefulSet but keep the PVCs. Later, you recreate the StatefulSet with the same name. What happens to the data?**

   A. Data is lost because StatefulSet creates new PVCs
-> B. **Data is preserved and pods mount the existing PVCs**
   C. Pods fail to start because PVCs have wrong owner references
   D. You get an error because PVC names conflict

<details>
<summary>See Answer</summary>

StatefulSets have a unique and powerful feature for persistent data: When you delete a StatefulSet, pods are deleted but PVCs are NOT deleted and remain in the cluster with data intact. When you recreate the StatefulSet with the same name, it tries to create PVC data-postgres-0 but the PVC already exists, so the StatefulSet uses the existing one. The pod mounts the existing PVC and all data is preserved. This is intentional design since StatefulSets are for stateful applications like databases and caches where data should survive StatefulSet deletion and recreation. This allows safe updates, migrations, and disaster recovery.

</details>

---

### Question 9: Implement a GitOps workflow for deploying applications to multiple environments (dev/staging/prod).

**Type:** Practical | **Category:** GitOps & CI/CD

## The Scenario

You're the Platform Engineering Lead at a rapidly growing SaaS company. Your team manages 50+ microservices deployed across three environments:

- **Development:** 5 microservices, rapid deployments (50+ per day)
- **Staging:** 20 microservices, QA testing environment
- **Production:** 50 microservices, serving 1M+ users

**Current problems with manual kubectl deployments:**

1. **No audit trail** - Can't track who deployed what, when
2. **Configuration drift** - Production configs don't match Git
3. **Manual errors** - Typos in kubectl commands caused 2 outages last month
4. **No rollback mechanism** - Reverting bad deploys takes 30+ minutes
5. **Security risk** - 15 developers have direct kubectl access to production

**Your CTO's mandate:**

> "Implement GitOps. All deployments must go through Git. No direct kubectl access to production. Full audit trail. Automatic rollback on failures."

## The Challenge

Design and implement a complete GitOps pipeline using ArgoCD or Flux that:

1. **Single source of truth:** All configs in Git repositories
2. **Automated deployments:** Git commit → automatic deployment
3. **Environment promotion:** Tested changes flow from dev → staging → prod
4. **Rollback capability:** Instant rollback via Git revert
5. **Security:** Remove direct kubectl access, use RBAC for Git-based approvals

Show the complete implementation with repository structure, ArgoCD configs, and workflows.

<JuniorVsSenior
  juniorAnswer="Basic CI/CD without GitOps principles - set up a Jenkins job that runs kubectl apply when code is pushed. No single source of truth (deployments happen from CI, not Git), configuration drift (cluster state may differ from Git), no automatic healing (manual changes not reverted), no clear audit trail, still requires kubectl access from CI, no environment-specific configs, and manual rollback process."
  seniorAnswer="Production GitOps architecture using ArgoCD with complete environment separation, automated sync policies, Kustomize overlays for environment-specific configs, AppProjects for RBAC, sync windows for production control, self-healing to prevent drift, and integrated CI/CD pipeline that updates Git repos to trigger deployments."
>

### Junior Approach: Basic CI/CD Without GitOps

The junior developer sets up a simple Jenkins pipeline:

```bash
# Jenkins pipeline
stage('Deploy') {
  sh 'kubectl apply -f deployment.yaml'
}
```

Problems with this approach:
- No single source of truth (deployments happen from CI, not Git)
- Configuration drift (cluster state may differ from Git)
- No automatic healing (manual changes not reverted)
- No clear audit trail
- Still requires kubectl access from CI
- No environment-specific configs
- Manual rollback process

### Senior Approach: Production GitOps Architecture

This mirrors what companies like Weaveworks, Intuit, and Adobe use. Here's the complete solution:

#### GitOps Architecture Overview

```
┌─────────────────┐
│  Git Repository │  (Source of Truth)
│  - Helm charts  │
│  - Manifests    │
│  - Kustomize    │
└────────┬────────┘
         │
         │ Git Commit
         ↓
┌─────────────────┐
│     ArgoCD      │  (Continuous Delivery)
│  - Monitors Git │
│  - Syncs K8s    │
│  - Auto-heal    │
└────────┬────────┘
         │
         │ kubectl apply
         ↓
┌─────────────────┐
│  Kubernetes     │  (Runtime)
│  - Dev cluster  │
│  - Staging      │
│  - Production   │
└─────────────────┘
```

#### Repository Structure (GitOps Best Practice)

```
company-gitops/
├── apps/                          # Application definitions
│   ├── frontend/
│   │   ├── base/                  # Common configs
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   └── kustomization.yaml
│   │   ├── overlays/
│   │   │   ├── dev/
│   │   │   │   ├── kustomization.yaml
│   │   │   │   └── config.yaml
│   │   │   ├── staging/
│   │   │   │   ├── kustomization.yaml
│   │   │   │   └── config.yaml
│   │   │   └── production/
│   │   │       ├── kustomization.yaml
│   │   │       └── config.yaml
│   ├── api-service/
│   └── payment-service/
│
├── infrastructure/                # Cluster configs
│   ├── namespaces/
│   ├── ingress/
│   ├── monitoring/
│   └── storage/
│
├── argocd/                        # ArgoCD application definitions
│   ├── apps/
│   │   ├── frontend-dev.yaml
│   │   ├── frontend-staging.yaml
│   │   └── frontend-prod.yaml
│   └── projects/
│       ├── dev-project.yaml
│       ├── staging-project.yaml
│       └── prod-project.yaml
│
└── README.md
```

#### Complete ArgoCD Installation

```bash
# 1. Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 2. Expose ArgoCD UI
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

# 3. Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d

# 4. Login to ArgoCD
argocd login <ARGOCD_SERVER>
argocd account update-password
```

#### ArgoCD Project Configuration (Environment Isolation)

```yaml
---
# Production project - strict controls
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  description: Production environment

  # Which Git repos are allowed
  sourceRepos:
  - 'https://github.com/company/gitops.git'

  # Where apps can be deployed
  destinations:
  - namespace: 'prod-*'
    server: 'https://kubernetes.default.svc'

  # What resources can be managed
  clusterResourceWhitelist:
  - group: '*'
    kind: '*'

  # Require manual approval for prod
  syncWindows:
  - kind: allow
    schedule: '0 10 * * 1-5'  # Mon-Fri 10 AM only
    duration: 8h
    applications:
    - '*'
    manualSync: true  # Require manual approval

  # Orphaned resource protection
  orphanedResources:
    warn: true

---
# Development project - more permissive
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: development
  namespace: argocd
spec:
  description: Development environment

  sourceRepos:
  - 'https://github.com/company/gitops.git'

  destinations:
  - namespace: 'dev-*'
    server: 'https://kubernetes.default.svc'

  clusterResourceWhitelist:
  - group: '*'
    kind: '*'

  # Auto-sync enabled for dev
  syncWindows:
  - kind: allow
    schedule: '* * * * *'  # Always allowed
    duration: 24h
    applications:
    - '*'
```

#### Application Configuration (Frontend Service Example)

```yaml
---
# ArgoCD Application for Production
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: frontend-production
  namespace: argocd
  # Finalizer ensures cascade delete
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  # Project for RBAC
  project: production

  # Git source
  source:
    repoURL: 'https://github.com/company/gitops.git'
    targetRevision: main
    path: apps/frontend/overlays/production

    # Use Kustomize for config management
    kustomize:
      namePrefix: prod-
      commonLabels:
        environment: production
      images:
      - name: company/frontend
        newTag: v1.2.4  # Managed by CI/CD

  # Destination cluster and namespace
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: prod-frontend

  # Sync policy
  syncPolicy:
    automated:
      prune: true      # Delete resources removed from Git
      selfHeal: true   # Auto-correct manual kubectl changes
      allowEmpty: false

    # Sync options
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    - PruneLast=true

    # Retry strategy
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m

  # Health checks
  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas  # Ignore HPA-controlled replicas

---
# ArgoCD Application for Staging (auto-sync enabled)
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: frontend-staging
  namespace: argocd
spec:
  project: staging

  source:
    repoURL: 'https://github.com/company/gitops.git'
    targetRevision: main
    path: apps/frontend/overlays/staging

  destination:
    server: 'https://kubernetes.default.svc'
    namespace: staging-frontend

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true

---
# ArgoCD Application for Development (fastest sync)
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: frontend-dev
  namespace: argocd
spec:
  project: development

  source:
    repoURL: 'https://github.com/company/gitops.git'
    targetRevision: main  # Or 'dev' branch for dev environment
    path: apps/frontend/overlays/dev

  destination:
    server: 'https://kubernetes.default.svc'
    namespace: dev-frontend

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

#### Application Manifests (Kustomize Structure)

Base deployment (apps/frontend/base/deployment.yaml):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3  # Overridden by overlays
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: company/frontend:latest  # Overridden by ArgoCD
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        env:
        - name: ENVIRONMENT
          value: "production"  # Overridden by overlays
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
```

Production overlay (apps/frontend/overlays/production/kustomization.yaml):

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# Base resources
resources:
- ../../base

# Production-specific namespace
namespace: prod-frontend

# Labels for all resources
commonLabels:
  environment: production
  managed-by: argocd

# Production-specific patches
patchesStrategicMerge:
- deployment-patch.yaml

# Production replicas
replicas:
- name: frontend
  count: 10  # Higher replicas for production

# Production image
images:
- name: company/frontend
  newTag: v1.2.4  # Specific stable version
```

Production deployment patch:

```yaml
# apps/frontend/overlays/production/deployment-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  template:
    spec:
      containers:
      - name: frontend
        env:
        - name: ENVIRONMENT
          value: "production"
        - name: API_URL
          value: "https://api.company.com"
        - name: LOG_LEVEL
          value: "info"
        resources:
          requests:
            memory: "512Mi"  # Higher for production
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
```

#### GitOps Workflow: CI/CD Pipeline Integration

```yaml
# .github/workflows/deploy.yml
name: Deploy to Kubernetes

on:
  push:
    branches:
      - main
    paths:
      - 'src/**'
      - 'Dockerfile'

jobs:
  build-and-update:
    runs-on: ubuntu-latest
    steps:
    # Build Docker image
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Build and push Docker image
      run: |
        docker build -t company/frontend:${{ github.sha }} .
        docker tag company/frontend:${{ github.sha }} company/frontend:latest
        docker push company/frontend:${{ github.sha }}
        docker push company/frontend:latest

    # Update GitOps repository
    - name: Update image tag in GitOps repo
      run: |
        git clone https://github.com/company/gitops.git
        cd gitops

        # Update development environment (auto-deploy)
        cd apps/frontend/overlays/dev
        kustomize edit set image company/frontend:${{ github.sha }}

        # Commit and push
        git config user.name "GitHub Actions"
        git config user.email "actions@github.com"
        git add .
        git commit -m "Update frontend image to ${{ github.sha }}"
        git push

    # ArgoCD will automatically sync the change to dev cluster

  promote-to-staging:
    needs: build-and-update
    runs-on: ubuntu-latest
    # Only run if tests pass
    if: github.ref == 'refs/heads/main'
    steps:
    - name: Run integration tests
      run: |
        # Wait for dev deployment
        sleep 60
        # Run tests against dev environment
        npm run test:integration

    - name: Promote to staging
      if: success()
      run: |
        git clone https://github.com/company/gitops.git
        cd gitops/apps/frontend/overlays/staging
        kustomize edit set image company/frontend:${{ github.sha }}
        git commit -am "Promote frontend to staging: ${{ github.sha }}"
        git push

  promote-to-production:
    needs: promote-to-staging
    runs-on: ubuntu-latest
    # Require manual approval for production
    environment: production
    steps:
    - name: Promote to production
      run: |
        git clone https://github.com/company/gitops.git
        cd gitops/apps/frontend/overlays/production
        kustomize edit set image company/frontend:${{ github.sha }}
        git commit -am "Deploy frontend to production: ${{ github.sha }}"
        git push

    # ArgoCD syncs during production sync window (Mon-Fri 10 AM)
```

#### Rollback Strategy

Instant rollback via Git revert:

```bash
# Check Git history
cd gitops
git log --oneline apps/frontend/overlays/production/kustomization.yaml

abc123 Deploy frontend to production: sha256abc
def456 Deploy frontend to production: sha256def  # Last good version
ghi789 Deploy frontend to production: sha256ghi

# Rollback to previous version
git revert abc123
git push

# ArgoCD automatically syncs the rollback within 3 minutes
# Or trigger immediate sync:
argocd app sync frontend-production
```

#### Monitoring GitOps Deployments

```yaml
# Prometheus alert for sync failures
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: argocd-alerts
  namespace: monitoring
spec:
  groups:
  - name: argocd
    rules:
    - alert: ArgocdAppOutOfSync
      expr: |
        argocd_app_info{sync_status="OutOfSync"} > 0
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: "ArgoCD app {{ $labels.name }} is out of sync"
        description: "Application has been out of sync for 10+ minutes"

    - alert: ArgocdAppSyncFailed
      expr: |
        argocd_app_sync_total{phase="Failed"} > 0
      labels:
        severity: critical
      annotations:
        summary: "ArgoCD sync failed for {{ $labels.name }}"
```

#### Security and RBAC for GitOps

```yaml
# ArgoCD RBAC policy (restrict production access)
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.csv: |
    # Developers: read-only on production
    p, role:developer, applications, get, production/*, allow
    p, role:developer, applications, sync, development/*, allow
    p, role:developer, applications, sync, staging/*, allow

    # Platform team: full access
    p, role:platform-admin, applications, *, */*, allow
    p, role:platform-admin, clusters, *, *, allow
    p, role:platform-admin, repositories, *, *, allow

    # Bind users to roles
    g, alice@company.com, role:platform-admin
    g, dev-team, role:developer
```

#### Complete Deployment Workflow Example

```
┌─────────────────────────────────────────────────────────┐
│ 1. Developer commits code to application repository     │
└────────────────────┬────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 2. CI builds Docker image: company/frontend:sha256abc   │
└────────────────────┬────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 3. CI updates GitOps repo: overlays/dev/kustomization   │
│    sets image: company/frontend:sha256abc               │
└────────────────────┬────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 4. ArgoCD detects Git change (polls every 3 minutes)    │
└────────────────────┬────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 5. ArgoCD syncs to dev cluster (automatic)              │
└────────────────────┬────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 6. Integration tests run against dev                    │
└────────────────────┬────────────────────────────────────┘
                     │
                     ↓ (tests pass)
┌─────────────────────────────────────────────────────────┐
│ 7. CI updates staging overlay with same image tag       │
└────────────────────┬────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 8. ArgoCD syncs to staging (automatic)                  │
└────────────────────┬────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 9. QA team approves staging deployment                  │
└────────────────────┬────────────────────────────────────┘
                     │
                     ↓ (manual approval)
┌─────────────────────────────────────────────────────────┐
│ 10. CI updates production overlay                       │
└────────────────────┬────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 11. ArgoCD syncs during sync window (manual trigger)    │
│     Production deployment complete!                     │
└─────────────────────────────────────────────────────────┘
```

#### GitOps Best Practices Checklist

- Separate Git repos: application code vs GitOps configs
- Use Kustomize or Helm for environment-specific configs
- Implement environment promotion (dev → staging → prod)
- Require manual approval for production deployments
- Use sync windows to control when prod deploys happen
- Enable auto-heal to prevent configuration drift
- Enable auto-prune to remove deleted resources
- Set up RBAC to restrict production access
- Monitor ArgoCD sync status with Prometheus alerts
- Use Git tags/branches for production stability
- Document rollback procedures
- Test GitOps workflow in dev before applying to prod

</JuniorVsSenior>


---

### Quick Check

**Your ArgoCD Application has selfHeal: true enabled. A developer runs kubectl scale deployment frontend --replicas=20 directly on the production cluster to handle a traffic spike. What happens?**

   A. The deployment stays at 20 replicas because kubectl has priority
-> B. **ArgoCD detects the drift and scales back to Git-defined replicas (3) within minutes**
   C. ArgoCD creates a conflict error and stops syncing
   D. Both Git and kubectl changes merge, resulting in 23 replicas

<details>
<summary>See Answer</summary>

With selfHeal: true enabled, ArgoCD actively monitors for configuration drift and automatically corrects it. When the developer manually scales the deployment to 20 replicas, ArgoCD detects the drift within its polling interval (every 3 minutes by default). It sees that Git defines replicas: 3 while the cluster has replicas: 20, marking the application as OutOfSync. ArgoCD then auto-heals by running kubectl apply internally to restore the deployment to 3 replicas. This is the POWER of GitOps - Git is the single source of truth, manual kubectl changes are considered drift, cluster state automatically converges to Git state, and this prevents configuration drift and snowflake clusters. If the developer truly needed 20 replicas, they should update Git by setting replicas: 20 in deployment-patch.yaml, commit and push the change, and let ArgoCD sync it. Now 20 replicas becomes the correct state. To temporarily disable self-heal, set selfHeal: false in the syncPolicy. An important exception is using ignoreDifferences to exclude specific fields like replica count from drift detection, which is common when using HPA (Horizontal Pod Autoscaler).

</details>

---

### Question 10: Your production cluster went down. Walk through your disaster recovery and backup/restore strategy.

**Type:** Architecture | **Category:** Disaster Recovery

## The Scenario

You're the Infrastructure Architect at a healthcare SaaS company. Your production Kubernetes cluster hosts critical patient data and healthcare provider applications serving 500+ hospitals.

**Business requirements from the CEO:**

- **RTO (Recovery Time Objective):** &lt; 1 hour - System must be back online within 1 hour of disaster
- **RPO (Recovery Point Objective):** &lt; 15 minutes - Maximum acceptable data loss is 15 minutes
- **Compliance:** HIPAA-compliant - All backups must be encrypted
- **Multi-region:** Failover to secondary region if primary region fails
- **Testing:** DR plan must be tested quarterly

**What counts as a "disaster":**

1. **Entire AWS region outage** (rare but happened: us-east-1 in 2017, 2021)
2. **Kubernetes cluster corruption** (etcd data loss, control plane failure)
3. **Ransomware attack** (malicious deletion of resources, data encryption)
4. **Accidental deletion** (developer runs `kubectl delete namespace production`)
5. **Data center fire/natural disaster** (hurricane, earthquake, flood)

Last week, during a routine upgrade, someone accidentally ran:

```bash
kubectl delete namespace production --force
```

**Everything was deleted:**
- 50 microservices
- 200 GB of persistent volume data
- ConfigMaps, Secrets, RBAC policies
- Ingress rules, Network Policies

Your CTO asks: **"How quickly can we recover?"**

Currently, you don't have a good answer. Your job is to design a complete disaster recovery plan.

## The Challenge

Design a comprehensive disaster recovery strategy that includes:

1. **Backup strategy:** What to back up and how often
2. **Storage location:** Where to store backups (encryption, geo-redundancy)
3. **Automated backup:** CI/CD integration and scheduling
4. **Recovery procedures:** Step-by-step restoration process
5. **Failover architecture:** Multi-region active-passive setup
6. **Testing plan:** Quarterly DR drills

Show complete configurations, tools (Velero, etcd backup), and runbooks.

<JuniorVsSenior
  juniorAnswer="Basic backups without comprehensive planning - set up weekly backups using kubectl get all -o yaml and store them on S3. Backup frequency too low (weekly equals up to 7 days data loss), kubectl get all doesn't capture everything (secrets, PVs, RBAC), no automation (manual backups are unreliable), no testing plan (backups might not work when needed), no multi-region failover, and no etcd backups (cluster state could be lost). This approach violates both RTO (less than 1 hour) and RPO (less than 15 minutes) requirements."
  seniorAnswer="Enterprise disaster recovery architecture with three-layer strategy: Layer 1 - etcd backup every 15 minutes for Kubernetes state, Layer 2 - Velero backup with hourly incremental and daily full backups for resources and volumes, Layer 3 - Multi-region replication with active-passive setup. All backups stored in S3 with cross-region replication, KMS encryption, and versioning enabled. Includes automated CronJobs, comprehensive restore procedures, monitoring with Prometheus alerts, quarterly testing plan, and detailed runbooks for various disaster scenarios."
>

### Junior Approach: Basic Backups Without Comprehensive Planning

The junior approach uses weekly kubectl backups:

```bash
kubectl get all -o yaml > backup.yaml
```

Problems with this approach:
- Backup frequency too low (weekly = up to 7 days data loss)
- kubectl get all doesn't capture everything (secrets, PVs, RBAC)
- No automation (manual backups are unreliable)
- No testing plan (backups might not work when needed)
- No multi-region failover
- No etcd backups (cluster state could be lost)

This approach violates both RTO (&lt; 1 hour) and RPO (&lt; 15 minutes) requirements.

### Senior Approach: Enterprise Disaster Recovery Architecture

This is exactly how financial institutions, healthcare companies, and Fortune 500 companies implement DR. Here's the complete solution:

#### Three-Layer DR Strategy

```
Layer 1: etcd Backup (Kubernetes state)
   ↓ Every 15 minutes
   ↓
Layer 2: Velero Backup (Resources + Volumes)
   ↓ Hourly incremental, Daily full
   ↓
Layer 3: Multi-Region Replication
   ↓ Active-Passive setup
   ↓
Storage: S3 with cross-region replication
```

#### Layer 1: etcd Backup (Control Plane State)

etcd stores the entire Kubernetes cluster state. If etcd is lost, the cluster is gone.

Automated etcd backup script:

```bash
#!/bin/bash
set -e

ETCD_ENDPOINTS="https://127.0.0.1:2379"
ETCD_CERT="/etc/kubernetes/pki/etcd/server.crt"
ETCD_KEY="/etc/kubernetes/pki/etcd/server.key"
ETCD_CA="/etc/kubernetes/pki/etcd/ca.crt"

BACKUP_DIR="/var/backups/etcd"
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/etcd-backup-${TIMESTAMP}.db"

# Create backup directory
mkdir -p ${BACKUP_DIR}

# Create etcd snapshot
ETCDCTL_API=3 etcdctl snapshot save ${BACKUP_FILE} \
  --endpoints=${ETCD_ENDPOINTS} \
  --cacert=${ETCD_CA} \
  --cert=${ETCD_CERT} \
  --key=${ETCD_KEY}

# Verify backup
ETCDCTL_API=3 etcdctl snapshot status ${BACKUP_FILE} -w table

# Upload to S3 with encryption
aws s3 cp ${BACKUP_FILE} \
  s3://company-k8s-backups/etcd/${TIMESTAMP}/ \
  --sse aws:kms \
  --sse-kms-key-id arn:aws:kms:us-east-1:123456789:key/abc-123

# Keep only last 7 days locally
find ${BACKUP_DIR} -type f -name "*.db" -mtime +7 -delete

echo "✅ etcd backup completed: ${BACKUP_FILE}"
```

Automated etcd backup CronJob:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: etcd-backup
  namespace: kube-system
spec:
  # Run every 15 minutes (RPO requirement)
  schedule: "*/15 * * * *"
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      template:
        spec:
          hostNetwork: true
          nodeName: master-node-1  # Run on control plane node
          containers:
          - name: etcd-backup
            image: company/etcd-backup:v1.0
            command: ["/scripts/backup-etcd.sh"]
            volumeMounts:
            - name: etcd-certs
              mountPath: /etc/kubernetes/pki/etcd
              readOnly: true
            - name: backup-dir
              mountPath: /var/backups/etcd
            env:
            - name: AWS_ACCESS_KEY_ID
              valueFrom:
                secretKeyRef:
                  name: aws-credentials
                  key: access-key-id
            - name: AWS_SECRET_ACCESS_KEY
              valueFrom:
                secretKeyRef:
                  name: aws-credentials
                  key: secret-access-key
          volumes:
          - name: etcd-certs
            hostPath:
              path: /etc/kubernetes/pki/etcd
          - name: backup-dir
            hostPath:
              path: /var/backups/etcd
          restartPolicy: OnFailure
```

etcd Restore Procedure:

```bash
#!/bin/bash
# Restore etcd from backup

BACKUP_FILE="/var/backups/etcd/etcd-backup-20250115-100000.db"
RESTORE_DIR="/var/lib/etcd-restore"

# Stop etcd
systemctl stop etcd

# Restore snapshot
ETCDCTL_API=3 etcdctl snapshot restore ${BACKUP_FILE} \
  --data-dir=${RESTORE_DIR} \
  --name=etcd-restore \
  --initial-cluster=etcd-restore=https://10.0.1.10:2380 \
  --initial-advertise-peer-urls=https://10.0.1.10:2380

# Update etcd data directory
rm -rf /var/lib/etcd
mv ${RESTORE_DIR} /var/lib/etcd

# Start etcd
systemctl start etcd

echo "✅ etcd restored from ${BACKUP_FILE}"
```

#### Layer 2: Velero Backup (Complete Cluster Backup)

Velero backs up all Kubernetes resources (Deployments, Services, ConfigMaps, Secrets, etc.), Persistent Volumes (using volume snapshots), and Namespaces, RBAC, Network Policies.

Install Velero:

```bash
# 1. Create S3 bucket for backups
aws s3 mb s3://company-velero-backups --region us-east-1

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket company-velero-backups \
  --versioning-configuration Status=Enabled

# Enable encryption
aws s3api put-bucket-encryption \
  --bucket company-velero-backups \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "arn:aws:kms:us-east-1:123456789:key/abc-123"
      }
    }]
  }'

# 2. Create IAM policy for Velero
cat > velero-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeVolumes",
        "ec2:DescribeSnapshots",
        "ec2:CreateTags",
        "ec2:CreateVolume",
        "ec2:CreateSnapshot",
        "ec2:DeleteSnapshot"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::company-velero-backups/*",
        "arn:aws:s3:::company-velero-backups"
      ]
    }
  ]
}
EOF

aws iam create-policy --policy-name VeleroPolicy --policy-document file://velero-policy.json

# 3. Install Velero
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.8.0 \
  --bucket company-velero-backups \
  --backup-location-config region=us-east-1 \
  --snapshot-location-config region=us-east-1 \
  --secret-file ./credentials-velero \
  --use-volume-snapshots=true \
  --use-node-agent
```

Velero Backup Schedules:

```yaml
---
# Hourly incremental backup
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: hourly-backup
  namespace: velero
spec:
  schedule: "0 * * * *"  # Every hour
  template:
    # Include all namespaces except system ones
    includedNamespaces:
    - production
    - staging
    excludedNamespaces:
    - kube-system
    - kube-public

    # Backup volumes
    defaultVolumesToRestic: true

    # Retention
    ttl: 72h  # Keep hourly backups for 3 days

    # Hooks for app-consistent backups
    hooks:
      resources:
      - name: postgres-backup
        includedNamespaces:
        - production
        labelSelector:
          matchLabels:
            app: postgres
        pre:
        - exec:
            container: postgres
            command:
            - /bin/bash
            - -c
            - pg_dump -U postgres > /tmp/backup.sql
        post:
        - exec:
            container: postgres
            command:
            - /bin/bash
            - -c
            - rm /tmp/backup.sql

---
# Daily full backup
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: daily-backup
  namespace: velero
spec:
  schedule: "0 2 * * *"  # 2 AM daily
  template:
    # Backup everything including cluster resources
    includedResources:
    - '*'
    includeClusterResources: true

    defaultVolumesToRestic: true
    ttl: 720h  # Keep daily backups for 30 days

---
# Weekly compliance backup (long-term retention)
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: weekly-backup
  namespace: velero
spec:
  schedule: "0 3 * * 0"  # 3 AM every Sunday
  template:
    includedResources:
    - '*'
    includeClusterResources: true
    defaultVolumesToRestic: true
    ttl: 8760h  # Keep weekly backups for 1 year

    # Store in separate long-term retention bucket
    storageLocation: long-term-storage
```

Velero Restore Procedure:

```bash
# 1. List available backups
velero backup get

NAME                STATUS      CREATED                         EXPIRES
hourly-backup-001   Completed   2025-01-15 10:00:00 +0000 UTC   3d
daily-backup-001    Completed   2025-01-15 02:00:00 +0000 UTC   30d

# 2. Restore from specific backup
velero restore create restore-prod-20250115 \
  --from-backup daily-backup-001 \
  --wait

# 3. Check restore status
velero restore describe restore-prod-20250115

# 4. Verify resources are restored
kubectl get all -n production
```

#### Layer 3: Multi-Region Active-Passive Architecture

```
┌─────────────────────────────────────────────────────────┐
│ Primary Region (us-east-1)                              │
│  ├── Production Cluster (Active)                        │
│  ├── RDS Multi-AZ (Primary)                             │
│  └── S3 Bucket (Velero backups)                         │
│       ↓ Cross-region replication                        │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ↓ Replication
┌─────────────────────────────────────────────────────────┐
│ Secondary Region (us-west-2)                            │
│  ├── Standby Cluster (Passive - Ready to activate)     │
│  ├── RDS Read Replica (Promoted to primary on failover)│
│  └── S3 Bucket (Replicated Velero backups)             │
└─────────────────────────────────────────────────────────┘
```

Cross-Region S3 Replication:

```bash
# Enable cross-region replication
aws s3api put-bucket-replication \
  --bucket company-velero-backups \
  --replication-configuration '{
    "Role": "arn:aws:iam::123456789:role/S3ReplicationRole",
    "Rules": [{
      "Status": "Enabled",
      "Priority": 1,
      "Filter": {},
      "Destination": {
        "Bucket": "arn:aws:s3:::company-velero-backups-dr",
        "ReplicationTime": {
          "Status": "Enabled",
          "Time": {
            "Minutes": 15
          }
        },
        "Metrics": {
          "Status": "Enabled",
          "EventThreshold": {
            "Minutes": 15
          }
        }
      }
    }]
  }'
```

Multi-Region Database Replication (using Terraform):

```hcl
# AWS RDS with cross-region read replica
resource "aws_db_instance" "primary" {
  identifier     = "production-db"
  engine         = "postgres"
  instance_class = "db.r5.2xlarge"

  # Multi-AZ for high availability
  multi_az = true

  # Enable automated backups
  backup_retention_period = 30
  backup_window          = "03:00-04:00"

  # Enable point-in-time recovery
  enabled_cloudwatch_logs_exports = ["postgresql"]

  # Encryption
  storage_encrypted = true
  kms_key_id       = "arn:aws:kms:us-east-1:123456789:key/abc-123"
}

# Cross-region read replica for DR
resource "aws_db_instance" "replica" {
  identifier             = "production-db-replica"
  replicate_source_db    = aws_db_instance.primary.arn
  instance_class         = "db.r5.2xlarge"

  # Different region
  provider = aws.us-west-2

  # Can be promoted to standalone on failover
  backup_retention_period = 30
  storage_encrypted       = true
}
```

#### Disaster Recovery Runbooks

**Scenario 1: Accidental Namespace Deletion**

```bash
# INCIDENT: Someone ran kubectl delete namespace production

# Step 1: Identify the backup to restore from
velero backup get | grep production
hourly-backup-20250115-1400  Completed  15m ago

# Step 2: Restore the namespace
velero restore create prod-restore-ns \
  --from-backup hourly-backup-20250115-1400 \
  --include-namespaces production \
  --wait

# Step 3: Verify restoration
kubectl get all -n production
kubectl get pvc -n production

# Step 4: Verify application health
kubectl get pods -n production
curl https://api.company.com/health

# Recovery Time: ~10 minutes
# Data Loss: ~15 minutes (last backup)
```

**Scenario 2: Complete Cluster Failure**

```bash
# INCIDENT: Control plane nodes failed, etcd corrupted

# Step 1: Provision new cluster
eksctl create cluster -f cluster-config.yaml

# Step 2: Install Velero on new cluster
velero install --provider aws --bucket company-velero-backups ...

# Step 3: Restore from latest backup
velero restore create full-cluster-restore \
  --from-backup daily-backup-20250115 \
  --wait

# Step 4: Update DNS to point to new cluster
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456 \
  --change-batch '{
    "Changes": [{
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "api.company.com",
        "Type": "A",
        "AliasTarget": {
          "HostedZoneId": "Z2FDTNDATAQYW2",
          "DNSName": "new-cluster-lb.us-east-1.elb.amazonaws.com",
          "EvaluateTargetHealth": false
        }
      }
    }]
  }'

# Recovery Time: ~45 minutes
# Data Loss: ~1 hour (last daily backup)
```

**Scenario 3: Region Failure (Failover to DR Region)**

```bash
# INCIDENT: Entire us-east-1 region is down

# Step 1: Promote RDS read replica in us-west-2 to primary
aws rds promote-read-replica \
  --db-instance-identifier production-db-replica \
  --region us-west-2

# Step 2: Scale up standby cluster in us-west-2
kubectl scale deployment --all --replicas=10 -n production

# Step 3: Update DNS to point to DR region
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456 \
  --change-batch '{
    "Changes": [{
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "api.company.com",
        "Type": "A",
        "AliasTarget": {
          "DNSName": "dr-cluster-lb.us-west-2.elb.amazonaws.com"
        }
      }
    }]
  }'

# Step 4: Verify failover
curl https://api.company.com/health

# Recovery Time: ~30 minutes
# Data Loss: ~15 minutes (RDS replication lag)
```

#### Quarterly DR Testing Plan

```yaml
# DR Test Checklist (Run every quarter)

Week 1: Test etcd Restore
  - Provision test cluster
  - Restore from etcd backup
  - Verify cluster state matches production
  - Document time taken

Week 2: Test Velero Namespace Restore
  - Delete test namespace
  - Restore from Velero backup
  - Verify all resources restored
  - Check PV data integrity

Week 3: Test Full Cluster Recovery
  - Provision new test cluster
  - Restore complete cluster from Velero
  - Run smoke tests
  - Measure RTO (should be &lt; 1 hour)

Week 4: Test Region Failover
  - Simulate region failure
  - Failover to DR region
  - Promote RDS replica
  - Update DNS
  - Verify application functionality
  - Measure RTO and RPO
```

#### Monitoring and Alerting

```yaml
# Prometheus alerts for backup failures
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: backup-alerts
  namespace: monitoring
spec:
  groups:
  - name: disaster-recovery
    rules:
    - alert: VeleroBackupFailed
      expr: |
        velero_backup_failure_total > 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Velero backup failed"
        description: "Backup {{ $labels.backup }} failed. Check Velero logs."

    - alert: EtcdBackupMissing
      expr: |
        time() - etcd_backup_last_success_timestamp > 1800
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "etcd backup not taken in 30+ minutes"
        description: "Last successful backup was > 30 minutes ago"

    - alert: S3ReplicationLag
      expr: |
        aws_s3_replication_lag_seconds > 900
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: "S3 cross-region replication lagging"
        description: "Replication lag is > 15 minutes (RPO violation)"
```

#### Cost Analysis

Monthly DR Costs:

```
etcd Backups:
- S3 storage (96 backups/day * 100 MB * 30 days): ~$7/month
- S3 cross-region replication: ~$3/month

Velero Backups:
- S3 storage (hourly 500 GB * 72 hours): ~$40/month
- S3 storage (daily 2 TB * 30 days): ~$120/month
- EBS snapshots (100 volumes * 100 GB * $0.05): ~$500/month

Multi-Region Setup:
- Standby cluster (10% capacity): ~$500/month
- RDS read replica: ~$800/month
- Cross-region data transfer: ~$200/month

Total DR Cost: ~$2,170/month

Cost of 1 hour outage (healthcare SaaS):
- Revenue loss: ~$50,000
- SLA penalties: ~$20,000
- Reputation damage: Priceless

ROI: DR costs $26K/year but prevents $70K+ losses per incident
```

</JuniorVsSenior>


---

### Quick Check

**Your Velero backup completed successfully at 2:00 PM. At 2:30 PM, a developer accidentally deletes the production namespace. You restore from the 2:00 PM backup. What is the RPO and RTO?**

   A. RPO = 0 minutes (no data loss), RTO = 10 minutes
-> B. **RPO = 30 minutes (data loss from 2:00-2:30 PM), RTO = 10 minutes**
   C. RPO = 15 minutes (backup policy), RTO = 30 minutes
   D. RPO = 30 minutes, RTO = 1 hour

<details>
<summary>See Answer</summary>

RPO (Recovery Point Objective) is the maximum acceptable data loss measured backwards from the incident. In this scenario, the last backup was at 2:00 PM and the incident occurred at 2:30 PM. Any data created or modified between 2:00-2:30 PM is LOST, so RPO equals 30 minutes. RTO (Recovery Time Objective) is the maximum acceptable downtime or time to restore service. In this scenario, the incident was detected at 2:30 PM, the Velero restore command was run at 2:32 PM, the restore completed at 2:40 PM, and the application was healthy at 2:40 PM, giving an RTO of 10 minutes. RPO is determined by backup frequency: hourly backups give RPO up to 60 minutes, backups every 15 minutes give RPO up to 15 minutes, and continuous replication gives RPO near 0 minutes. RTO is determined by restore speed: Velero namespace restore takes approximately 10 minutes, full cluster restore takes 30-60 minutes, and multi-region failover takes approximately 30 minutes. In this case, the backup was at 2:00 PM, incident at 2:30 PM, and any data created after 2:00 PM (last backup) is lost, therefore RPO equals 30 minutes (the gap between last backup and incident). To reduce RPO to less than 15 minutes, change the backup schedule from hourly to every 15 minutes. To meet strict RPO/RTO requirements: RPO less than 5 minutes requires database streaming replication, RTO less than 5 minutes requires multi-region active-active setup, and zero RPO requires synchronous replication (which is expensive).

</details>

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*
