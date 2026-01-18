# Complete Google Cloud Platform (GCP) Interview Guide

Master GCP with 12 real-world interview questions covering IAM, networking, GKE, BigQuery, serverless, and cost optimization. Practice scenarios that mirror actual senior cloud engineer challenges.

**Companies that ask these questions:** Google | Spotify | Twitter | Snap | PayPal | Target

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | A Cloud Function can't access Cloud Storage. Debug the IAM p... | Debugging | IAM & Security |
| 2 | Design a secure VPC architecture with private GKE clusters a... | Architecture | Networking |
| 3 | Pods are stuck in Pending state and nodes keep getting OOMKi... | Debugging | GKE |
| 4 | Your Cloud Storage bill is $50K/month. Implement lifecycle p... | Practical | Storage |
| 5 | Cloud Run has 5-second cold starts causing timeouts. Optimiz... | Debugging | Serverless |
| 6 | A BigQuery query scans 10TB and costs $50 per run. Optimize ... | Practical | BigQuery |
| 7 | Cloud SQL connections are exhausted and failover takes too l... | Practical | Databases |
| 8 | Pub/Sub messages are processed out of order and some are los... | Practical | Messaging |
| 9 | Design a global load balancing architecture with Cloud CDN f... | Architecture | Load Balancing |
| 10 | Build a monitoring dashboard and alerting system that catche... | Practical | Observability |
| 11 | Implement a CI/CD pipeline with Cloud Build that deploys to ... | Practical | CI/CD |
| 12 | Your GCP bill increased 40% last month. Identify waste and i... | Practical | Cost Optimization |

---

## What You'll Learn

- Debug IAM permission issues and service account impersonation
- Design VPC networks with proper firewall rules and Private Google Access
- Troubleshoot GKE cluster and workload issues
- Optimize Cloud Storage costs with lifecycle policies
- Fix Cloud Run cold starts and scaling issues
- Optimize BigQuery queries to reduce costs by 90%
- Configure Cloud SQL for high availability and connection pooling
- Implement reliable Pub/Sub messaging with ordering and dead letters
- Design global load balancing with Cloud CDN
- Build effective monitoring and alerting with Cloud Operations
- Implement CI/CD pipelines with Cloud Build
- Optimize GCP costs with committed use and right-sizing

---

## Interview Questions

### Question 1: A Cloud Function can't access Cloud Storage. Debug the IAM permission denied error.

**Type:** Debugging | **Category:** IAM & Security

## The Scenario

Your Cloud Function needs to read files from a Cloud Storage bucket, but it's failing with:

```
Error: Permission denied on resource project my-project
googleapi: Error 403: my-function@my-project.iam.gserviceaccount.com does not have
storage.objects.get access to the Google Cloud Storage object., forbidden
```

The function worked in development but fails in production. You've already granted the Storage Object Viewer role to the service account.

## The Challenge

Debug the IAM permission issue and implement proper service account configuration. Explain the principle of least privilege and GCP's IAM model.


### Step 1: Identify Which Service Account the Function Uses

```bash
# Check the Cloud Function's service account
gcloud functions describe my-function \
  --region=us-central1 \
  --format="value(serviceAccountEmail)"

# Output might show:
# my-project@appspot.gserviceaccount.com  (App Engine default)
# 123456789-compute@developer.gserviceaccount.com  (Compute default)
# my-function@my-project.iam.gserviceaccount.com  (Custom SA)

# Common mistake: You granted permissions to the wrong service account!
```

### Step 2: Check Current IAM Permissions

```bash
# Check bucket-level permissions
gcloud storage buckets get-iam-policy gs://my-bucket

# Check if the service account has the expected role
gcloud projects get-iam-policy my-project \
  --flatten="bindings[].members" \
  --filter="bindings.members:my-function@my-project.iam.gserviceaccount.com" \
  --format="table(bindings.role)"

# Test the service account's access directly
gcloud auth activate-service-account \
  --key-file=service-account-key.json

gcloud storage ls gs://my-bucket/
```

### Step 3: Understand IAM Scope

```bash
# Project-level role (applies to ALL buckets in project)
gcloud projects add-iam-policy-binding my-project \
  --member="serviceAccount:my-function@my-project.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

# Bucket-level role (applies to specific bucket only - preferred!)
gcloud storage buckets add-iam-policy-binding gs://my-bucket \
  --member="serviceAccount:my-function@my-project.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

# Object-level (IAM conditions for specific prefixes)
gcloud storage buckets add-iam-policy-binding gs://my-bucket \
  --member="serviceAccount:my-function@my-project.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer" \
  --condition="expression=resource.name.startsWith('projects/_/buckets/my-bucket/objects/allowed-prefix/'),title=AllowedPrefixOnly"
```

### Step 4: Create Dedicated Service Account

```bash
# Create a dedicated service account (best practice)
gcloud iam service-accounts create cloud-function-storage \
  --display-name="Cloud Function Storage Access" \
  --description="Service account for Cloud Function to access specific GCS bucket"

# Grant minimal permissions at bucket level
gcloud storage buckets add-iam-policy-binding gs://my-bucket \
  --member="serviceAccount:cloud-function-storage@my-project.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

# Deploy function with the dedicated service account
gcloud functions deploy my-function \
  --region=us-central1 \
  --runtime=nodejs18 \
  --service-account=cloud-function-storage@my-project.iam.gserviceaccount.com \
  --source=.
```

### Step 5: Check for Blocking Policies

```bash
# Check VPC Service Controls (common in enterprise)
gcloud access-context-manager perimeters list

# Check Organization Policy constraints
gcloud resource-manager org-policies list --project=my-project

# Check IAM deny policies
gcloud iam policies list --attachment-point=projects/my-project

# If VPC-SC is blocking, you need to add the function to the perimeter
# or configure ingress/egress rules
```

### Step 6: Verify with IAM Policy Troubleshooter

```bash
# Use Policy Troubleshooter to test permissions
gcloud policy-troubleshoot iam \
  //storage.googleapis.com/projects/_/buckets/my-bucket/objects/test.txt \
  --principal-email=my-function@my-project.iam.gserviceaccount.com \
  --permission=storage.objects.get
```

### Terraform Implementation

```hcl
# Create dedicated service account
resource "google_service_account" "function_sa" {
  account_id   = "cloud-function-storage"
  display_name = "Cloud Function Storage Access"
  description  = "Dedicated SA for Cloud Function GCS access"
}

# Grant bucket-level access only
resource "google_storage_bucket_iam_member" "function_access" {
  bucket = google_storage_bucket.data.name
  role   = "roles/storage.objectViewer"
  member = "serviceAccount:${google_service_account.function_sa.email}"
}

# Deploy function with dedicated SA
resource "google_cloudfunctions_function" "my_function" {
  name        = "my-function"
  runtime     = "nodejs18"
  region      = "us-central1"

  service_account_email = google_service_account.function_sa.email

  # Other config...
}
```

### Common IAM Gotchas

| Issue | Cause | Fix |
|-------|-------|-----|
| Permission granted but still denied | Wrong service account | Verify function's actual SA |
| Works in dev, fails in prod | Different projects/SAs | Use dedicated SA per environment |
| Intermittent failures | IAM propagation delay | Wait 60s after granting roles |
| Access denied on new bucket | Uniform bucket-level access | Use IAM, not ACLs |
| Blocked by VPC-SC | Service perimeter | Add to perimeter or configure rules |

### Service Account Best Practices

```bash
# 1. One service account per function/service
# 2. Use bucket-level, not project-level permissions
# 3. Enable audit logging for service accounts

gcloud projects add-iam-policy-binding my-project \
  --member="serviceAccount:cloud-function-storage@my-project.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer" \
  --condition=None

# 4. Set up alerts for service account usage
# 5. Rotate keys regularly (or use Workload Identity)
```


## IAM Hierarchy in GCP

```
Organization
    │
    ├── Folder (departments)
    │       │
    │       └── Project
    │               │
    │               └── Resource (bucket, instance, etc.)
    │
    └── Permissions flow DOWN (inheritance)
        Restrictions flow UP (deny policies)
```

## Predefined vs Custom Roles

| Role Type | Example | Use Case |
|-----------|---------|----------|
| Basic | `roles/viewer` | Never use in production |
| Predefined | `roles/storage.objectViewer` | Most common, well-scoped |
| Custom | `projects/x/roles/myRole` | When predefined is too broad |


---

### Quick Check

**Why is granting roles at the bucket level preferred over project level for Cloud Storage access?**

   A. Bucket-level roles are faster to apply
   B. Project-level roles don't work with Cloud Functions
-> C. **Bucket-level follows least privilege - the SA can only access that specific bucket, not all buckets in the project**
   D. Bucket-level roles are free while project-level roles cost money

<details>
<summary>See Answer</summary>

Granting storage.objectViewer at the project level gives the service account access to ALL buckets in the project. Bucket-level permissions follow the principle of least privilege - the service account can only access the specific bucket it needs. If the SA is compromised, the blast radius is limited to that one bucket rather than all storage in the project.

</details>

---

### Question 2: Design a secure VPC architecture with private GKE clusters and controlled internet access.

**Type:** Architecture | **Category:** Networking

## The Scenario

You're designing the network architecture for a healthcare application that must:
- Keep all compute resources private (no public IPs)
- Allow GKE pods to pull images from gcr.io and access GCP APIs
- Enable controlled outbound internet access for specific services
- Support multiple environments (dev/staging/prod) with isolation
- Meet HIPAA compliance requirements

## The Challenge

Design a VPC architecture using GCP networking primitives: Private Google Access, Cloud NAT, Shared VPC, and firewall rules. Explain the tradeoffs.


> **Common Mistake:** A junior engineer might assign public IPs to all resources for simplicity, use default firewall rules, create separate VPCs without connectivity, or skip Private Google Access. This creates security vulnerabilities, compliance violations, and operational complexity.

> **Senior Engineer Approach:** A senior engineer designs a Shared VPC with private subnets, enables Private Google Access for GCP API calls, configures Cloud NAT for controlled outbound access, uses hierarchical firewall policies, and implements VPC Service Controls for data exfiltration prevention.

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         Host Project                             │
│                        (Shared VPC)                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                        VPC Network                          │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │ │
│  │  │ us-central1 │  │ us-east1    │  │ europe-west1│        │ │
│  │  │ 10.0.0.0/20 │  │ 10.1.0.0/20 │  │ 10.2.0.0/20 │        │ │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘        │ │
│  │         │                │                │                │ │
│  │         └────────────────┼────────────────┘                │ │
│  │                          │                                  │ │
│  │  ┌──────────────────────┴──────────────────────┐          │ │
│  │  │              Cloud Router                    │          │ │
│  │  │         + Cloud NAT (all regions)           │          │ │
│  │  └─────────────────────────────────────────────┘          │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Dev Project  │    │Staging Project│   │ Prod Project │
│(Service Proj)│    │(Service Proj) │   │(Service Proj)│
│   GKE, GCE   │    │   GKE, GCE   │    │   GKE, GCE   │
└──────────────┘    └──────────────┘    └──────────────┘
```

### Step 1: Create Shared VPC Host Project

```bash
# Enable Shared VPC in host project
gcloud compute shared-vpc enable host-project

# Associate service projects
gcloud compute shared-vpc associated-projects add dev-project \
  --host-project=host-project

gcloud compute shared-vpc associated-projects add prod-project \
  --host-project=host-project
```

### Step 2: Create VPC and Subnets

```hcl
# terraform/network/main.tf

resource "google_compute_network" "main" {
  name                    = "shared-vpc"
  project                 = var.host_project
  auto_create_subnetworks = false
  routing_mode            = "GLOBAL"
}

# Regional subnets with secondary ranges for GKE
resource "google_compute_subnetwork" "regional" {
  for_each = var.regions

  name                     = "subnet-${each.key}"
  project                  = var.host_project
  region                   = each.key
  network                  = google_compute_network.main.id
  ip_cidr_range            = each.value.primary_range
  private_ip_google_access = true  # Critical for private clusters!

  # Secondary ranges for GKE pods and services
  secondary_ip_range {
    range_name    = "pods"
    ip_cidr_range = each.value.pods_range
  }

  secondary_ip_range {
    range_name    = "services"
    ip_cidr_range = each.value.services_range
  }

  log_config {
    aggregation_interval = "INTERVAL_5_SEC"
    flow_sampling        = 0.5
    metadata             = "INCLUDE_ALL_METADATA"
  }
}

variable "regions" {
  default = {
    "us-central1" = {
      primary_range  = "10.0.0.0/20"
      pods_range     = "10.100.0.0/14"
      services_range = "10.104.0.0/20"
    }
    "us-east1" = {
      primary_range  = "10.1.0.0/20"
      pods_range     = "10.108.0.0/14"
      services_range = "10.112.0.0/20"
    }
  }
}
```

### Step 3: Configure Cloud NAT for Outbound Access

```hcl
# Cloud Router per region
resource "google_compute_router" "regional" {
  for_each = var.regions

  name    = "router-${each.key}"
  project = var.host_project
  region  = each.key
  network = google_compute_network.main.id
}

# Cloud NAT for outbound internet access
resource "google_compute_router_nat" "regional" {
  for_each = var.regions

  name                               = "nat-${each.key}"
  project                            = var.host_project
  router                             = google_compute_router.regional[each.key].name
  region                             = each.key
  nat_ip_allocate_option             = "AUTO_ONLY"
  source_subnetwork_ip_ranges_to_nat = "ALL_SUBNETWORKS_ALL_IP_RANGES"

  log_config {
    enable = true
    filter = "ERRORS_ONLY"
  }

  # Timeouts for connection tracking
  tcp_established_idle_timeout_sec = 1200
  tcp_transitory_idle_timeout_sec  = 30
  udp_idle_timeout_sec             = 30
}
```

### Step 4: Create Private GKE Cluster

```hcl
resource "google_container_cluster" "private" {
  name     = "private-cluster"
  project  = var.service_project
  location = "us-central1"

  # Use Shared VPC
  network    = "projects/${var.host_project}/global/networks/shared-vpc"
  subnetwork = "projects/${var.host_project}/regions/us-central1/subnetworks/subnet-us-central1"

  # Private cluster configuration
  private_cluster_config {
    enable_private_nodes    = true
    enable_private_endpoint = false  # Allow kubectl from authorized networks
    master_ipv4_cidr_block  = "172.16.0.0/28"
  }

  # Use secondary ranges for pods/services
  ip_allocation_policy {
    cluster_secondary_range_name  = "pods"
    services_secondary_range_name = "services"
  }

  # Authorized networks for master access
  master_authorized_networks_config {
    cidr_blocks {
      cidr_block   = "10.0.0.0/8"
      display_name = "Internal VPC"
    }
    cidr_blocks {
      cidr_block   = var.admin_cidr
      display_name = "Admin Access"
    }
  }

  # Workload Identity for pod service accounts
  workload_identity_config {
    workload_pool = "${var.service_project}.svc.id.goog"
  }

  # VPC-native cluster
  networking_mode = "VPC_NATIVE"
}
```

### Step 5: Implement Hierarchical Firewall Policies

```hcl
# Organization-level firewall policy
resource "google_compute_firewall_policy" "org_policy" {
  short_name = "org-security-policy"
  parent     = "organizations/${var.org_id}"
}

# Deny all ingress by default
resource "google_compute_firewall_policy_rule" "deny_ingress_default" {
  firewall_policy = google_compute_firewall_policy.org_policy.id
  priority        = 65534
  action          = "deny"
  direction       = "INGRESS"
  match {
    layer4_configs {
      ip_protocol = "all"
    }
  }
}

# Allow internal communication
resource "google_compute_firewall_policy_rule" "allow_internal" {
  firewall_policy = google_compute_firewall_policy.org_policy.id
  priority        = 1000
  action          = "allow"
  direction       = "INGRESS"
  match {
    src_ip_ranges = ["10.0.0.0/8"]
    layer4_configs {
      ip_protocol = "all"
    }
  }
}

# Allow GCP health checks
resource "google_compute_firewall_policy_rule" "allow_health_checks" {
  firewall_policy = google_compute_firewall_policy.org_policy.id
  priority        = 1001
  action          = "allow"
  direction       = "INGRESS"
  match {
    src_ip_ranges = [
      "35.191.0.0/16",    # Health check ranges
      "130.211.0.0/22"
    ]
    layer4_configs {
      ip_protocol = "tcp"
    }
  }
}

# VPC-level firewall rules for specific needs
resource "google_compute_firewall" "allow_iap" {
  name    = "allow-iap-ssh"
  project = var.host_project
  network = google_compute_network.main.name

  allow {
    protocol = "tcp"
    ports    = ["22"]
  }

  source_ranges = ["35.235.240.0/20"]  # IAP range
  target_tags   = ["allow-iap"]
}
```

### Step 6: VPC Service Controls (Data Exfiltration Prevention)

```hcl
# Service perimeter for sensitive data
resource "google_access_context_manager_service_perimeter" "healthcare" {
  parent = "accessPolicies/${var.access_policy_id}"
  name   = "accessPolicies/${var.access_policy_id}/servicePerimeters/healthcare"
  title  = "Healthcare Data Perimeter"

  status {
    resources = [
      "projects/${var.prod_project_number}"
    ]

    restricted_services = [
      "storage.googleapis.com",
      "bigquery.googleapis.com",
      "healthcare.googleapis.com"
    ]

    # Allow access from VPC
    vpc_accessible_services {
      enable_restriction = true
      allowed_services   = ["RESTRICTED-SERVICES"]
    }

    ingress_policies {
      ingress_from {
        sources {
          access_level = google_access_context_manager_access_level.corp_network.name
        }
      }
      ingress_to {
        resources = ["*"]
        operations {
          service_name = "storage.googleapis.com"
          method_selectors {
            method = "*"
          }
        }
      }
    }
  }
}
```


## Network Design Summary

| Component | Purpose | Configuration |
|-----------|---------|---------------|
| Shared VPC | Centralized network management | Host + service projects |
| Private Google Access | Access GCP APIs without public IPs | Enabled on subnets |
| Cloud NAT | Controlled outbound internet | Per-region with logging |
| Private GKE | No public IPs on nodes | Private nodes + authorized networks |
| Firewall Policies | Hierarchical security rules | Org → Folder → Project |
| VPC Service Controls | Data exfiltration prevention | Service perimeters |

---

### Question 3: Pods are stuck in Pending state and nodes keep getting OOMKilled. Fix this GKE cluster.

**Type:** Debugging | **Category:** GKE

## The Scenario

Your production GKE cluster is experiencing issues. Some pods are stuck in Pending:

```bash
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
api-server-7d9f8c-abc    0/1     Pending   0          45m
api-server-7d9f8c-def    1/1     Running   0          2h
worker-5c6b7d-ghi        0/1     Pending   0          30m

$ kubectl describe pod api-server-7d9f8c-abc
Events:
  Warning  FailedScheduling  2m  default-scheduler
    0/5 nodes are available: 2 Insufficient cpu, 3 Insufficient memory,
    5 node(s) had taint {node.kubernetes.io/memory-pressure: NoSchedule}
```

Meanwhile, nodes are being evicted due to memory pressure:

```bash
$ kubectl get nodes
NAME                                  STATUS                     ROLES    AGE
gke-cluster-default-pool-abc123-def   Ready,SchedulingDisabled   <none>   5h
gke-cluster-default-pool-abc123-ghi   NotReady                   <none>   3h
```

## The Challenge

Diagnose the root cause of pod scheduling failures and node instability. Implement fixes for resource management, autoscaling, and node pool configuration.


> **Common Mistake:** A junior engineer might manually delete pending pods, increase node size without understanding the cause, disable resource limits entirely, or restart nodes. These approaches mask symptoms without fixing root causes and often make things worse.

> **Senior Engineer Approach:** A senior engineer systematically investigates: resource requests vs actual usage, node allocatable resources, memory pressure causes, autoscaler configuration, and implements proper resource management with requests/limits, PodDisruptionBudgets, and priority classes.

### Step 1: Understand Why Pods Are Pending

```bash
# Check why pods can't be scheduled
kubectl describe pod api-server-7d9f8c-abc | grep -A 20 Events

# Check node resources
kubectl describe nodes | grep -A 10 "Allocated resources"

# Example output:
# Allocated resources:
#   (Total limits may be over 100 percent, i.e., overcommitted.)
#   Resource           Requests      Limits
#   --------           --------      ------
#   cpu                3800m (95%)   8000m (200%)
#   memory             14Gi (95%)    20Gi (133%)

# See what's consuming resources
kubectl top nodes
kubectl top pods --all-namespaces --sort-by=memory
```

### Step 2: Check Node Memory Pressure

```bash
# Check node conditions
kubectl get nodes -o custom-columns=\
NAME:.metadata.name,\
MEMORY_PRESSURE:.status.conditions[?(@.type=="MemoryPressure")].status,\
DISK_PRESSURE:.status.conditions[?(@.type=="DiskPressure")].status

# Check kubelet logs for evictions
gcloud logging read \
  'resource.type="k8s_node" AND
   textPayload:"eviction"' \
  --limit=50 \
  --format="table(timestamp, textPayload)"

# Check what's being evicted
kubectl get events --sort-by='.lastTimestamp' | grep -i evict
```

### Step 3: Analyze Pod Resource Configuration

```bash
# Find pods without resource limits (dangerous!)
kubectl get pods --all-namespaces -o json | \
  jq -r '.items[] | select(.spec.containers[].resources.limits == null) |
  "\(.metadata.namespace)/\(.metadata.name)"'

# Check current resource configuration
kubectl get pod api-server-7d9f8c-def -o yaml | \
  yq '.spec.containers[].resources'

# Typical problematic config:
# resources:
#   requests:
#     memory: "256Mi"    # Too low request
#     cpu: "100m"
#   limits:
#     memory: "8Gi"      # 32x the request - causes overcommit!
#     cpu: "2000m"
```

### Step 4: Fix Resource Requests and Limits

```yaml
# Proper resource configuration
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: api
        resources:
          requests:
            memory: "512Mi"    # Based on actual P95 usage
            cpu: "250m"
          limits:
            memory: "1Gi"      # 2x request is reasonable
            cpu: "500m"        # Limit CPU bursting
        # Add liveness/readiness for proper scheduling
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Step 5: Configure Cluster Autoscaler

```bash
# Check current autoscaler status
gcloud container clusters describe my-cluster \
  --zone=us-central1-a \
  --format="yaml(autoscaling)"

# Enable cluster autoscaler with proper limits
gcloud container clusters update my-cluster \
  --zone=us-central1-a \
  --enable-autoscaling \
  --min-nodes=2 \
  --max-nodes=20 \
  --node-pool=default-pool

# Check autoscaler events
kubectl get events -n kube-system | grep cluster-autoscaler
```

### Step 6: Create Properly Sized Node Pools

```hcl
# Terraform: Create node pool with appropriate sizing
resource "google_container_node_pool" "primary" {
  name       = "primary-pool"
  cluster    = google_container_cluster.main.name
  location   = "us-central1"

  # Autoscaling configuration
  autoscaling {
    min_node_count = 2
    max_node_count = 20
  }

  node_config {
    machine_type = "e2-standard-4"  # 4 vCPU, 16GB RAM

    # Reserve resources for system daemons
    # Allocatable = Total - Reserved
    # e2-standard-4: ~3.5 CPU, ~14GB allocatable

    labels = {
      workload = "general"
    }

    # Prevent scheduling on nodes during maintenance
    taint {
      key    = "dedicated"
      value  = "general"
      effect = "NO_SCHEDULE"
    }

    oauth_scopes = [
      "https://www.googleapis.com/auth/cloud-platform"
    ]
  }

  management {
    auto_repair  = true
    auto_upgrade = true
  }
}

# Separate pool for memory-intensive workloads
resource "google_container_node_pool" "memory_optimized" {
  name     = "memory-pool"
  cluster  = google_container_cluster.main.name
  location = "us-central1"

  autoscaling {
    min_node_count = 0
    max_node_count = 10
  }

  node_config {
    machine_type = "n2-highmem-4"  # 4 vCPU, 32GB RAM

    labels = {
      workload = "memory-intensive"
    }

    taint {
      key    = "workload"
      value  = "memory-intensive"
      effect = "NO_SCHEDULE"
    }
  }
}
```

### Step 7: Implement Resource Quotas and Limit Ranges

```yaml
# Namespace resource quota
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: production
spec:
  hard:
    requests.cpu: "20"
    requests.memory: "40Gi"
    limits.cpu: "40"
    limits.memory: "80Gi"
    pods: "50"

---
# Default limits for pods without explicit resources
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: production
spec:
  limits:
  - default:
      memory: "512Mi"
      cpu: "500m"
    defaultRequest:
      memory: "256Mi"
      cpu: "100m"
    max:
      memory: "4Gi"
      cpu: "2"
    min:
      memory: "64Mi"
      cpu: "50m"
    type: Container
```

### Step 8: Set Up Priority Classes

```yaml
# High priority for critical workloads
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "Critical production workloads"

---
# Default priority
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: default-priority
value: 0
globalDefault: true
description: "Default priority for all pods"

---
# Low priority for batch jobs (can be preempted)
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: -1000
globalDefault: false
preemptionPolicy: Never
description: "Batch jobs that can be preempted"
```

```yaml
# Use priority class in deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
spec:
  template:
    spec:
      priorityClassName: high-priority
      containers:
      - name: api
        # ...
```


## GKE Resource Debugging Cheatsheet

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| Pods Pending | Insufficient resources | Right-size requests or add nodes |
| Node MemoryPressure | Overcommitted memory | Reduce limits:requests ratio |
| OOMKilled pods | Memory limit too low | Increase limit based on actual usage |
| Slow scaling | Autoscaler configuration | Reduce scale-down delay |
| Uneven distribution | No PodAntiAffinity | Add topology spread constraints |

## Useful Debugging Commands

```bash
# Real-time resource monitoring
kubectl top pods --containers
kubectl top nodes

# Check why autoscaler isn't scaling
kubectl -n kube-system logs -l app=cluster-autoscaler --tail=100

# Find resource hogs
kubectl get pods -A -o json | jq -r '
  .items[] |
  "\(.metadata.namespace)/\(.metadata.name):
   CPU: \(.spec.containers[0].resources.requests.cpu // "none")
   MEM: \(.spec.containers[0].resources.requests.memory // "none")"'
```


---

### Quick Check

**Why does having memory limits much higher than requests (e.g., 256Mi request, 8Gi limit) cause node memory pressure issues?**

   A. The kubelet cannot track memory usage properly with such ratios
-> B. **Kubernetes schedules pods based on requests, so nodes get overcommitted when pods use memory up to their limits**
   C. GKE doesn't support limits higher than 4x requests
   D. Memory limits are ignored by the Linux kernel

<details>
<summary>See Answer</summary>

Kubernetes scheduler places pods based on resource requests, not limits. If 10 pods each request 256Mi memory (2.5Gi total) but can use up to 8Gi each (80Gi potential), they might all fit on a 16Gi node initially. When pods use memory up to their limits, the node becomes overcommitted and enters memory pressure, leading to evictions. Keeping limits close to requests (2x is reasonable) prevents severe overcommitment.

</details>

---

### Question 4: Your Cloud Storage bill is $50K/month. Implement lifecycle policies to reduce costs.

**Type:** Practical | **Category:** Storage

## The Scenario

Your company's Cloud Storage bill has grown to $50,000/month. Analysis shows:

```
Bucket: data-lake-prod (45TB)
├── raw/           25TB - Ingested daily, rarely accessed after 7 days
├── processed/     15TB - Analytics output, accessed weekly
├── archives/       5TB - Compliance data, never accessed
└── All in Standard storage class

Bucket: user-uploads (10TB)
├── avatars/        2TB - Frequently accessed
├── documents/      8TB - Accessed for 30 days, then rarely
└── All in Standard storage class

Operations: 500M Class A, 2B Class B per month
```

## The Challenge

Design a lifecycle management strategy to reduce costs by at least 60% while maintaining access patterns and compliance requirements.


> **Common Mistake:** A junior engineer might move everything to Coldline storage, delete old data without considering compliance, or implement complex policies that are hard to maintain. This breaks application access patterns, causes compliance violations, or creates operational nightmares.

> **Senior Engineer Approach:** A senior engineer analyzes access patterns, implements tiered storage classes based on data lifecycle, uses Autoclass for unpredictable access, sets up lifecycle rules for automatic transitions, and monitors to validate cost savings without access issues.

### Step 1: Analyze Current Usage

```bash
# Get bucket sizes by storage class
gsutil du -s gs://data-lake-prod/

# Analyze access patterns with Cloud Monitoring
gcloud logging read \
  'resource.type="gcs_bucket" AND
   protoPayload.methodName:"storage.objects.get"' \
  --format="csv(timestamp,protoPayload.resourceName)" \
  --limit=10000 > access_log.csv

# Check bucket metadata
gcloud storage buckets describe gs://data-lake-prod --format=yaml
```

### Step 2: Understand Storage Class Economics

| Storage Class | Storage Cost | Retrieval | Min Duration | Use Case |
|--------------|--------------|-----------|--------------|----------|
| Standard | $0.020/GB | Free | None | Frequent access |
| Nearline | $0.010/GB | $0.01/GB | 30 days | Monthly access |
| Coldline | $0.004/GB | $0.02/GB | 90 days | Quarterly access |
| Archive | $0.0012/GB | $0.05/GB | 365 days | Yearly/compliance |

### Step 3: Implement Lifecycle Policies

```json
// lifecycle-data-lake.json
{
  "lifecycle": {
    "rule": [
      {
        "action": {
          "type": "SetStorageClass",
          "storageClass": "NEARLINE"
        },
        "condition": {
          "age": 7,
          "matchesPrefix": ["raw/"]
        }
      },
      {
        "action": {
          "type": "SetStorageClass",
          "storageClass": "COLDLINE"
        },
        "condition": {
          "age": 30,
          "matchesPrefix": ["raw/"]
        }
      },
      {
        "action": {
          "type": "SetStorageClass",
          "storageClass": "ARCHIVE"
        },
        "condition": {
          "age": 90,
          "matchesPrefix": ["raw/"]
        }
      },
      {
        "action": {
          "type": "SetStorageClass",
          "storageClass": "NEARLINE"
        },
        "condition": {
          "age": 30,
          "matchesPrefix": ["processed/"]
        }
      },
      {
        "action": {
          "type": "SetStorageClass",
          "storageClass": "ARCHIVE"
        },
        "condition": {
          "age": 0,
          "matchesPrefix": ["archives/"]
        }
      },
      {
        "action": {
          "type": "Delete"
        },
        "condition": {
          "age": 2555,
          "matchesPrefix": ["raw/"]
        }
      }
    ]
  }
}
```

```bash
# Apply lifecycle policy
gcloud storage buckets update gs://data-lake-prod \
  --lifecycle-file=lifecycle-data-lake.json
```

### Step 4: Terraform Implementation

```hcl
resource "google_storage_bucket" "data_lake" {
  name     = "data-lake-prod"
  location = "US"

  # Enable uniform bucket-level access
  uniform_bucket_level_access = true

  # Lifecycle rules for cost optimization
  lifecycle_rule {
    condition {
      age                   = 7
      matches_prefix        = ["raw/"]
    }
    action {
      type          = "SetStorageClass"
      storage_class = "NEARLINE"
    }
  }

  lifecycle_rule {
    condition {
      age                   = 30
      matches_prefix        = ["raw/"]
    }
    action {
      type          = "SetStorageClass"
      storage_class = "COLDLINE"
    }
  }

  lifecycle_rule {
    condition {
      age                   = 90
      matches_prefix        = ["raw/"]
    }
    action {
      type          = "SetStorageClass"
      storage_class = "ARCHIVE"
    }
  }

  # Delete raw data after 7 years (compliance requirement)
  lifecycle_rule {
    condition {
      age                   = 2555
      matches_prefix        = ["raw/"]
    }
    action {
      type = "Delete"
    }
  }

  # Processed data to Nearline after 30 days
  lifecycle_rule {
    condition {
      age            = 30
      matches_prefix = ["processed/"]
    }
    action {
      type          = "SetStorageClass"
      storage_class = "NEARLINE"
    }
  }

  # Versioning with cleanup
  versioning {
    enabled = true
  }

  lifecycle_rule {
    condition {
      num_newer_versions = 3
      with_state         = "ARCHIVED"
    }
    action {
      type = "Delete"
    }
  }
}
```

### Step 5: Use Autoclass for Unpredictable Access

```hcl
# For buckets with unpredictable access patterns
resource "google_storage_bucket" "user_uploads" {
  name     = "user-uploads-prod"
  location = "US"

  # Autoclass automatically moves objects between classes
  # based on access patterns
  autoclass {
    enabled = true
  }

  # You cannot use lifecycle rules with Autoclass
  # Autoclass handles transitions automatically
}
```

```bash
# Enable Autoclass on existing bucket
gcloud storage buckets update gs://user-uploads \
  --enable-autoclass
```

### Step 6: Optimize Operations Costs

```bash
# Current: 500M Class A ($2,500), 2B Class B ($800)

# Reduce operations:
# 1. Use composite objects for small files
gsutil compose gs://bucket/part1 gs://bucket/part2 gs://bucket/combined

# 2. Batch requests
from google.cloud import storage
client = storage.Client()
bucket = client.bucket('data-lake-prod')

# Batch list operations
blobs = list(bucket.list_blobs(prefix='raw/', max_results=1000))

# 3. Use Cloud CDN for frequently accessed objects
# Caches at edge, reduces origin requests
```

### Step 7: Cost Calculation

```
BEFORE:
- Standard storage: 55TB × $0.020 = $1,100/month
- Operations: $3,300/month
- Egress + other: ~$45,600/month
- Total: ~$50,000/month

AFTER (projected):
Data Lake (45TB):
- raw/ 25TB:
  - 7 days Standard: 0.6TB × $0.020 = $12
  - 23 days Nearline: 1.9TB × $0.010 = $19
  - Coldline: 7TB × $0.004 = $28
  - Archive: 15.5TB × $0.0012 = $19
- processed/ 15TB Nearline: 15TB × $0.010 = $150
- archives/ 5TB Archive: 5TB × $0.0012 = $6
Subtotal: ~$234/month (was ~$900)

User Uploads (10TB with Autoclass):
- ~$150/month (was ~$200)

Operations (reduced with batching):
- ~$2,000/month (was $3,300)

New Total: ~$20,000/month (60% reduction)
```

### Step 8: Monitor and Validate

```hcl
# Alert on unexpected storage class changes
resource "google_monitoring_alert_policy" "storage_class_change" {
  display_name = "Unexpected Storage Class Distribution"

  conditions {
    display_name = "Standard storage exceeds threshold"

    condition_threshold {
      filter = <<-EOT
        resource.type="gcs_bucket" AND
        metric.type="storage.googleapis.com/storage/total_bytes" AND
        metric.labels.storage_class="STANDARD"
      EOT

      comparison      = "COMPARISON_GT"
      threshold_value = 10000000000000  # 10TB
      duration        = "3600s"

      aggregations {
        alignment_period   = "3600s"
        per_series_aligner = "ALIGN_MEAN"
      }
    }
  }

  notification_channels = [google_monitoring_notification_channel.email.id]
}
```


## Storage Class Decision Matrix

| Access Frequency | Data Age | Recommended Class |
|-----------------|----------|-------------------|
| Daily | < 30 days | Standard |
| Weekly | 30-90 days | Nearline |
| Monthly | 90-365 days | Coldline |
| Yearly/Never | > 365 days | Archive |
| Unpredictable | Any | Autoclass |

## Lifecycle Policy Best Practices

1. **Start conservative** - Move to Nearline first, monitor, then Coldline
2. **Consider retrieval costs** - Archive retrieval is expensive
3. **Use prefixes** - Different policies for different data types
4. **Test in non-prod** - Validate access patterns first
5. **Monitor transitions** - Use Cloud Monitoring metrics

<InterviewQuiz
  question="Why might moving all data directly to Archive storage class actually increase costs?"
  options={[
    "Archive storage has higher storage costs than Standard",
    "Archive has a 365-day minimum storage duration and expensive retrieval fees, so frequently accessed data costs more",
    "Archive storage doesn't support lifecycle policies",
    "Archive storage is only available in certain regions"
  ]}
  correctAnswer={1}
  explanation="Archive storage has a 365-day minimum storage duration - you pay for 365 days even if you delete earlier. Retrieval costs $0.05/GB, and there's a retrieval delay. If data is accessed monthly, Archive retrieval costs ($0.05 × 12 = $0.60/GB/year) far exceed Standard storage costs ($0.24/GB/year). Archive is only cost-effective for data accessed once a year or less."
/>

---

### Question 5: Cloud Run has 5-second cold starts causing timeouts. Optimize for low latency.

**Type:** Debugging | **Category:** Serverless

## The Scenario

Your Cloud Run service handles webhook callbacks that must respond within 3 seconds. After periods of inactivity, requests timeout:

```
POST /webhook/payment-callback
Status: 504 Gateway Timeout
Time: 8.2s

Logs show:
- Container start: 4.8s
- Application init: 2.1s
- Request processing: 1.3s
- Total: 8.2s (timeout at 3s)
```

The service scales down to zero during off-hours, and the first morning request always fails.

## The Challenge

Reduce cold start latency to under 1 second while balancing cost. Understand the factors that affect startup time and implement optimizations.


### Step 1: Analyze Cold Start Components

```
Cold Start Breakdown:
├── Container scheduling: ~200ms (GCP infrastructure)
├── Image pull: ~500-2000ms (depends on image size)
├── Container start: ~200ms (runtime initialization)
└── Application init: ~1000-5000ms (your code)

Total: 2-8 seconds typical
```

### Step 2: Optimize Container Image

```dockerfile
# BEFORE: Large, slow image (1.2GB)
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "server.js"]

# AFTER: Optimized image (150MB)
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM gcr.io/distroless/nodejs18-debian11
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
COPY src/ ./src/
CMD ["src/server.js"]
```

```dockerfile
# Python optimization
# BEFORE: 1GB image
FROM python:3.11

# AFTER: 200MB image
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

### Step 3: Optimize Application Startup

```javascript
// BEFORE: Blocking initialization
const express = require('express');
const { BigQuery } = require('@google-cloud/bigquery');
const { Storage } = require('@google-cloud/storage');

const app = express();
const bigquery = new BigQuery();  // Connects immediately
const storage = new Storage();    // Connects immediately

// Load all configs at startup
const config = loadAllConfigs();  // Blocks for 2s
const cache = warmCache();        // Blocks for 1s

app.listen(8080);

// AFTER: Lazy initialization
const express = require('express');
const app = express();

// Lazy-loaded clients
let bigquery, storage;

const getBigQuery = () => {
  if (!bigquery) {
    const { BigQuery } = require('@google-cloud/bigquery');
    bigquery = new BigQuery();
  }
  return bigquery;
};

const getStorage = () => {
  if (!storage) {
    const { Storage } = require('@google-cloud/storage');
    storage = new Storage();
  }
  return storage;
};

// Start listening immediately
const server = app.listen(process.env.PORT || 8080, () => {
  console.log('Server ready');  // Log this ASAP for Cloud Run
});

// Background initialization (non-blocking)
setImmediate(async () => {
  await warmCache();
  console.log('Cache warmed');
});
```

### Step 4: Configure Minimum Instances

```bash
# Set minimum instances for production
gcloud run services update payment-webhook \
  --min-instances=2 \
  --region=us-central1

# Use different settings per environment
# Dev: 0 (save costs)
# Staging: 1
# Prod: 2-5
```

```yaml
# Cloud Run service YAML
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: payment-webhook
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/minScale: "2"
        autoscaling.knative.dev/maxScale: "100"
    spec:
      containerConcurrency: 80
      timeoutSeconds: 30
      containers:
      - image: gcr.io/project/payment-webhook:latest
        resources:
          limits:
            cpu: "2"
            memory: "1Gi"
```

### Step 5: Enable Startup CPU Boost

```bash
# Startup CPU boost gives 2x CPU during startup
gcloud run services update payment-webhook \
  --cpu-boost \
  --region=us-central1

# Or in Terraform
resource "google_cloud_run_service" "webhook" {
  template {
    metadata {
      annotations = {
        "run.googleapis.com/startup-cpu-boost" = "true"
      }
    }
  }
}
```

### Step 6: Use Second Generation Execution Environment

```bash
# Gen2 has faster cold starts and better CPU
gcloud run services update payment-webhook \
  --execution-environment=gen2 \
  --region=us-central1
```

### Step 7: Optimize Resource Allocation

```yaml
# More CPU = faster startup
spec:
  containers:
  - resources:
      limits:
        cpu: "2"      # More CPU for faster init
        memory: "1Gi" # Enough for your app

# CPU allocation setting
metadata:
  annotations:
    # CPU always allocated (not just during requests)
    run.googleapis.com/cpu-throttling: "false"
```

### Step 8: Implement Health Checks

```javascript
// Fast health check endpoint
app.get('/_health', (req, res) => {
  res.status(200).send('OK');
});

// Separate readiness check (after initialization)
let isReady = false;

app.get('/_ready', (req, res) => {
  if (isReady) {
    res.status(200).send('Ready');
  } else {
    res.status(503).send('Not ready');
  }
});

// Mark ready after init completes
setImmediate(async () => {
  await initializeApp();
  isReady = true;
});
```

### Step 9: Monitor Cold Start Metrics

```bash
# View cold start metrics in Cloud Monitoring
gcloud logging read \
  'resource.type="cloud_run_revision" AND
   textPayload:"Cold start"' \
  --format="table(timestamp,textPayload)"

# Create alert for high cold start ratio
```

```hcl
resource "google_monitoring_alert_policy" "cold_start_alert" {
  display_name = "High Cold Start Ratio"

  conditions {
    display_name = "Cold starts > 10%"

    condition_threshold {
      filter = <<-EOT
        resource.type = "cloud_run_revision" AND
        metric.type = "run.googleapis.com/request_latencies" AND
        metric.labels.response_code_class = "2xx"
      EOT

      comparison      = "COMPARISON_GT"
      threshold_value = 5000  # 5 seconds
      duration        = "300s"

      aggregations {
        alignment_period     = "300s"
        per_series_aligner   = "ALIGN_PERCENTILE_99"
        cross_series_reducer = "REDUCE_MEAN"
      }
    }
  }
}
```

### Cold Start Optimization Summary

| Optimization | Impact | Cost Impact |
|-------------|--------|-------------|
| Smaller image | -1-3s startup | None |
| Lazy initialization | -1-2s startup | None |
| Startup CPU boost | -30% startup | None (free) |
| Gen2 execution | -20% startup | None |
| Min instances = 1 | Eliminates cold start | ~$30/month |
| Min instances = 2 | Eliminates cold start + HA | ~$60/month |


## Cold Start Decision Tree

```
Is latency critical?
    │
    ├── Yes → Set min instances ≥ 1
    │         + Optimize image
    │         + Enable CPU boost
    │
    └── No → Optimize image
             + Lazy initialization
             + Accept occasional cold starts
```


---

### Quick Check

**Why does lazy initialization of database clients help reduce cold start times?**

   A. It makes the database connection faster
-> B. **Cloud Run waits for the health check before routing traffic, so faster startup means faster health check response**
   C. Lazy initialization reduces container image size
   D. Database clients are not needed during cold starts

<details>
<summary>See Answer</summary>

Cloud Run considers a container ready when it starts listening on the port. If you initialize database connections, load configs, and warm caches before listening, that all adds to cold start time. With lazy initialization, the server starts listening immediately (fast cold start), and expensive operations happen in the background or on first use. The first request might be slightly slower, but subsequent cold starts are much faster.

</details>

---

### Question 6: A BigQuery query scans 10TB and costs $50 per run. Optimize it to under $5.

**Type:** Practical | **Category:** BigQuery

## The Scenario

A daily analytics report query costs $50 per execution:

```sql
-- Current query: scans 10TB, costs $50
SELECT
  user_id,
  COUNT(*) as event_count,
  SUM(revenue) as total_revenue
FROM `project.analytics.events`
WHERE event_date >= '2024-01-01'
GROUP BY user_id
ORDER BY total_revenue DESC
LIMIT 1000;
```

The `events` table is 50TB total, with new data added daily. The query runs multiple times per day for different date ranges.

## The Challenge

Reduce query costs by 90% using BigQuery optimization techniques: partitioning, clustering, materialized views, and query best practices.


### Step 1: Understand Current Table Structure

```sql
-- Check table size and partitioning
SELECT
  table_name,
  ROUND(total_bytes / POW(10,12), 2) as size_tb,
  ROUND(total_rows / POW(10,9), 2) as rows_billions,
  partition_expiration_ms,
  clustering_fields
FROM `project.analytics.INFORMATION_SCHEMA.TABLES`
WHERE table_name = 'events';

-- Check what columns are used in WHERE clauses
-- (from query history)
SELECT
  query,
  total_bytes_processed,
  total_slot_ms
FROM `region-us.INFORMATION_SCHEMA.JOBS_BY_PROJECT`
WHERE creation_time > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
  AND referenced_tables LIKE '%events%'
ORDER BY total_bytes_processed DESC
LIMIT 20;
```

### Step 2: Implement Partitioning

```sql
-- Create partitioned table
CREATE TABLE `project.analytics.events_partitioned`
PARTITION BY DATE(event_timestamp)
CLUSTER BY user_id, event_type
AS SELECT * FROM `project.analytics.events`;

-- Or add partitioning to existing table schema
-- (requires table recreation)

-- Set partition expiration for old data
ALTER TABLE `project.analytics.events_partitioned`
SET OPTIONS (
  partition_expiration_days = 365,
  require_partition_filter = true  -- Prevent full scans!
);
```

### Step 3: Add Clustering

```sql
-- Clustering sorts data within partitions
-- Great for high-cardinality columns used in WHERE/JOIN

CREATE TABLE `project.analytics.events_optimized`
PARTITION BY DATE(event_timestamp)
CLUSTER BY user_id, event_type, country
AS SELECT * FROM `project.analytics.events`;

-- Optimized query now scans much less data
SELECT
  user_id,
  COUNT(*) as event_count,
  SUM(revenue) as total_revenue
FROM `project.analytics.events_optimized`
WHERE DATE(event_timestamp) >= '2024-01-01'  -- Uses partition
  AND user_id IN (SELECT user_id FROM active_users)  -- Uses clustering
GROUP BY user_id;
```

### Step 4: Use Materialized Views

```sql
-- Pre-compute common aggregations
CREATE MATERIALIZED VIEW `project.analytics.daily_user_stats`
PARTITION BY event_date
CLUSTER BY user_id
OPTIONS (
  enable_refresh = true,
  refresh_interval_minutes = 60
)
AS
SELECT
  DATE(event_timestamp) as event_date,
  user_id,
  event_type,
  COUNT(*) as event_count,
  SUM(revenue) as total_revenue,
  COUNT(DISTINCT session_id) as sessions
FROM `project.analytics.events_optimized`
GROUP BY 1, 2, 3;

-- Query the materialized view (much faster and cheaper)
SELECT
  user_id,
  SUM(event_count) as total_events,
  SUM(total_revenue) as total_revenue
FROM `project.analytics.daily_user_stats`
WHERE event_date >= '2024-01-01'
GROUP BY user_id
ORDER BY total_revenue DESC
LIMIT 1000;
```

### Step 5: Query Optimization Best Practices

```sql
-- BEFORE: Inefficient query patterns
SELECT *  -- Selects ALL columns
FROM events
WHERE EXTRACT(YEAR FROM event_timestamp) = 2024  -- Function prevents partition pruning
  AND JSON_EXTRACT(metadata, '$.campaign') = 'summer'  -- Scans all rows

-- AFTER: Optimized query
SELECT
  user_id,
  event_type,
  revenue  -- Only needed columns
FROM `project.analytics.events_optimized`
WHERE event_timestamp >= '2024-01-01'  -- Partition pruning works
  AND event_timestamp < '2025-01-01'
  AND campaign = 'summer'  -- Use extracted column, not JSON
```

```sql
-- Use approximate functions for large datasets
SELECT
  APPROX_COUNT_DISTINCT(user_id) as unique_users,  -- Much faster
  APPROX_QUANTILES(revenue, 100)[OFFSET(50)] as median_revenue
FROM events_optimized
WHERE event_date = CURRENT_DATE();

-- Avoid self-joins, use window functions
-- BEFORE: Self-join to get previous row
SELECT a.*, b.revenue as prev_revenue
FROM events a
JOIN events b ON a.user_id = b.user_id
  AND a.event_timestamp = (
    SELECT MIN(event_timestamp)
    FROM events
    WHERE user_id = a.user_id
      AND event_timestamp > b.event_timestamp
  );

-- AFTER: Window function
SELECT
  *,
  LAG(revenue) OVER (PARTITION BY user_id ORDER BY event_timestamp) as prev_revenue
FROM events_optimized;
```

### Step 6: Implement Slot Reservations for Predictable Costs

```bash
# For predictable workloads, flat-rate pricing is cheaper
# 100 slots = ~$2,000/month vs pay-per-query

# Create reservation
bq mk --reservation \
  --project_id=project \
  --location=US \
  --slots=100 \
  analytics-reservation

# Assign projects to reservation
bq mk --reservation_assignment \
  --project_id=project \
  --location=US \
  --reservation_id=analytics-reservation \
  --job_type=QUERY \
  --assignee_type=PROJECT \
  --assignee_id=analytics-project
```

### Step 7: Use BI Engine for Dashboard Queries

```sql
-- Create BI Engine reservation for sub-second queries
-- Caches data in memory

-- Best for:
-- - Dashboard queries
-- - Repeated aggregations
-- - Time-series data

-- Configure via Console or API
-- 1GB BI Engine = ~$40/month
-- Dramatically faster for cached queries
```

### Cost Comparison

```
BEFORE:
- Table: 50TB, no partitioning
- Query scans: 10TB (filters in WHERE)
- Cost: 10TB × $5/TB = $50 per query
- Daily cost (5 runs): $250

AFTER:
- Table: 50TB, partitioned by day, clustered
- Query scans: 500GB (partition pruning + clustering)
- Cost: 0.5TB × $5/TB = $2.50 per query
- With materialized view: 50GB scan = $0.25

Optimization impact:
- Partitioning: 80% reduction (only scan needed dates)
- Clustering: 50% additional (within partition)
- Materialized view: 90% additional (pre-aggregated)
- Total: 95%+ reduction
```

### Monitoring Query Costs

```sql
-- Find expensive queries
SELECT
  user_email,
  query,
  ROUND(total_bytes_processed / POW(10,12), 4) as tb_processed,
  ROUND(total_bytes_processed / POW(10,12) * 5, 2) as estimated_cost_usd
FROM `region-us.INFORMATION_SCHEMA.JOBS_BY_PROJECT`
WHERE creation_time > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
  AND job_type = 'QUERY'
  AND state = 'DONE'
ORDER BY total_bytes_processed DESC
LIMIT 20;

-- Set up cost controls
-- In BigQuery settings: set maximum bytes billed per query
```


## BigQuery Optimization Checklist

| Technique | Cost Reduction | Implementation |
|-----------|---------------|----------------|
| Partition by date | 50-90% | `PARTITION BY DATE(timestamp)` |
| Clustering | 20-50% additional | `CLUSTER BY column1, column2` |
| Select specific columns | 30-70% | Avoid `SELECT *` |
| Materialized views | 80-95% | Pre-compute aggregations |
| BI Engine | N/A (faster) | Cache for dashboards |
| Flat-rate pricing | Predictable | For high-volume workloads |

---

### Question 7: Cloud SQL connections are exhausted and failover takes too long. Fix the database setup.

**Type:** Practical | **Category:** Databases

## The Scenario

Your Cloud SQL PostgreSQL instance is experiencing issues:

```
Error: FATAL: remaining connection slots are reserved for
non-replication superuser connections

Monitoring shows:
- Active connections: 100/100 (max)
- Connection attempts: 500/sec (mostly rejected)
- Failover time during maintenance: 3 minutes
- Application errors during failover: 2,847
```

Multiple GKE pods connect directly to Cloud SQL, and during failovers or high load, the application becomes unavailable.

## The Challenge

Design a resilient database architecture with proper connection management, high availability, and minimal failover impact.


### Step 1: Understand the Connection Problem

```bash
# Check current connections
gcloud sql connect my-instance --user=postgres

postgres=> SELECT count(*) FROM pg_stat_activity;
 count
-------
   100

postgres=> SELECT usename, application_name, count(*)
FROM pg_stat_activity
GROUP BY usename, application_name
ORDER BY count DESC;

# Typical issue: Each pod opens 10 connections
# 20 pods × 10 connections = 200 (but max is 100)
```

### Step 2: Implement Cloud SQL Auth Proxy with Connection Pooling

```yaml
# Deploy Cloud SQL Proxy as sidecar
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
spec:
  template:
    spec:
      serviceAccountName: app-service-account
      containers:
      - name: api
        image: gcr.io/project/api:latest
        env:
        - name: DB_HOST
          value: "127.0.0.1"  # Connect via proxy
        - name: DB_PORT
          value: "5432"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url

      - name: cloud-sql-proxy
        image: gcr.io/cloud-sql-connectors/cloud-sql-proxy:2.7.0
        args:
          - "--structured-logs"
          - "--private-ip"
          - "--auto-iam-authn"  # Use Workload Identity
          - "project:region:instance"
        securityContext:
          runAsNonRoot: true
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
```

### Step 3: Add PgBouncer for Connection Pooling

```yaml
# PgBouncer deployment for connection pooling
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pgbouncer
spec:
  replicas: 2
  template:
    spec:
      containers:
      - name: pgbouncer
        image: bitnami/pgbouncer:latest
        ports:
        - containerPort: 6432
        env:
        - name: POSTGRESQL_HOST
          value: "10.0.0.5"  # Cloud SQL private IP
        - name: POSTGRESQL_DATABASE
          value: "mydb"
        - name: PGBOUNCER_POOL_MODE
          value: "transaction"  # Best for web apps
        - name: PGBOUNCER_MAX_CLIENT_CONN
          value: "1000"  # Accept up to 1000 app connections
        - name: PGBOUNCER_DEFAULT_POOL_SIZE
          value: "20"  # Use only 20 real DB connections
        - name: PGBOUNCER_MIN_POOL_SIZE
          value: "10"
        - name: PGBOUNCER_RESERVE_POOL_SIZE
          value: "5"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: pgbouncer
spec:
  selector:
    app: pgbouncer
  ports:
  - port: 5432
    targetPort: 6432
```

### Step 4: Configure Cloud SQL High Availability

```hcl
resource "google_sql_database_instance" "main" {
  name             = "production-db"
  database_version = "POSTGRES_15"
  region           = "us-central1"

  settings {
    tier              = "db-custom-4-16384"  # 4 vCPU, 16GB RAM
    availability_type = "REGIONAL"  # Enables HA with standby

    backup_configuration {
      enabled                        = true
      point_in_time_recovery_enabled = true
      start_time                     = "03:00"
      transaction_log_retention_days = 7

      backup_retention_settings {
        retained_backups = 30
        retention_unit   = "COUNT"
      }
    }

    ip_configuration {
      ipv4_enabled    = false  # Disable public IP
      private_network = google_compute_network.main.id

      # Allow connections from GKE
      authorized_networks {
        name  = "gke-cluster"
        value = "10.0.0.0/8"
      }
    }

    database_flags {
      name  = "max_connections"
      value = "200"  # Reasonable limit with pooling
    }

    database_flags {
      name  = "log_min_duration_statement"
      value = "1000"  # Log slow queries > 1s
    }

    maintenance_window {
      day          = 7  # Sunday
      hour         = 3  # 3 AM
      update_track = "stable"
    }

    insights_config {
      query_insights_enabled  = true
      query_string_length     = 4096
      record_application_tags = true
      record_client_address   = true
    }
  }

  deletion_protection = true
}

# Read replica for read scaling
resource "google_sql_database_instance" "replica" {
  name                 = "production-db-replica"
  master_instance_name = google_sql_database_instance.main.name
  region               = "us-central1"
  database_version     = "POSTGRES_15"

  replica_configuration {
    failover_target = false
  }

  settings {
    tier              = "db-custom-4-16384"
    availability_type = "ZONAL"

    ip_configuration {
      ipv4_enabled    = false
      private_network = google_compute_network.main.id
    }
  }
}
```

### Step 5: Implement Application Retry Logic

```python


from sqlalchemy import create_engine
from sqlalchemy.exc import OperationalError

def create_db_engine():
    return create_engine(
        DATABASE_URL,
        pool_size=5,           # Connections per worker
        max_overflow=10,       # Extra connections when needed
        pool_pre_ping=True,    # Verify connections before use
        pool_recycle=1800,     # Recycle connections after 30min
        connect_args={
            "connect_timeout": 10,
            "application_name": "api-server",
        }
    )

def execute_with_retry(engine, query, max_retries=3):
    """Execute query with exponential backoff retry."""
    for attempt in range(max_retries):
        try:
            with engine.connect() as conn:
                return conn.execute(query)
        except OperationalError as e:
            if attempt == max_retries - 1:
                raise

            # Exponential backoff with jitter
            wait_time = (2 ** attempt) + random.uniform(0, 1)
            print(f"Database error, retrying in {wait_time:.2f}s: {e}")
            time.sleep(wait_time)
```

```javascript
// Node.js with pg-pool
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 10,                    // Max connections in pool
  idleTimeoutMillis: 30000,   // Close idle connections after 30s
  connectionTimeoutMillis: 10000,
  application_name: 'api-server',
});

async function queryWithRetry(sql, params, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await pool.query(sql, params);
    } catch (error) {
      if (attempt === maxRetries - 1) throw error;

      // Check if retryable error
      if (error.code === 'ECONNREFUSED' ||
          error.code === '57P01' ||  // admin_shutdown
          error.code === '57P02') {  // crash_shutdown
        const waitTime = Math.pow(2, attempt) * 1000 + Math.random() * 1000;
        console.log(`Database error, retrying in ${waitTime}ms`);
        await new Promise(r => setTimeout(r, waitTime));
      } else {
        throw error;
      }
    }
  }
}
```

### Step 6: Monitor Database Health

```hcl
# Alert on connection exhaustion
resource "google_monitoring_alert_policy" "db_connections" {
  display_name = "Cloud SQL Connection Exhaustion"

  conditions {
    display_name = "Connections > 80%"

    condition_threshold {
      filter = <<-EOT
        resource.type="cloudsql_database" AND
        metric.type="cloudsql.googleapis.com/database/postgresql/num_backends"
      EOT

      comparison      = "COMPARISON_GT"
      threshold_value = 160  # 80% of 200 max
      duration        = "300s"

      aggregations {
        alignment_period   = "60s"
        per_series_aligner = "ALIGN_MEAN"
      }
    }
  }
}

# Alert on replication lag (for read replicas)
resource "google_monitoring_alert_policy" "replication_lag" {
  display_name = "Cloud SQL Replication Lag"

  conditions {
    display_name = "Replication lag > 60s"

    condition_threshold {
      filter = <<-EOT
        resource.type="cloudsql_database" AND
        metric.type="cloudsql.googleapis.com/database/replication/replica_lag"
      EOT

      comparison      = "COMPARISON_GT"
      threshold_value = 60
      duration        = "300s"
    }
  }
}
```


## Connection Architecture

```
                    ┌─────────────────────┐
                    │   Application Pods   │
                    │  (100+ instances)    │
                    └──────────┬──────────┘
                               │ 1000 connections
                               ▼
                    ┌─────────────────────┐
                    │     PgBouncer       │
                    │  (connection pool)  │
                    └──────────┬──────────┘
                               │ 20-50 connections
                               ▼
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
    ┌─────────────────┐              ┌─────────────────┐
    │  Primary (HA)   │──replication─▶│  Read Replica   │
    │   (writes)      │              │    (reads)      │
    └─────────────────┘              └─────────────────┘
```

## Failover Behavior

| Configuration | Failover Time | Data Loss | Cost |
|--------------|---------------|-----------|------|
| Single zone | Manual (hours) | Possible | $ |
| Regional HA | ~60 seconds | None | $$$ |
| With read replicas | Reads continue | None | $$$$ |


---

### Quick Check

**Why should you use transaction pooling mode in PgBouncer instead of session pooling for web applications?**

   A. Transaction pooling is faster
   B. Session pooling doesn't work with PostgreSQL
-> C. **Transaction pooling returns connections to the pool after each transaction, allowing many more app connections than database connections**
   D. Transaction pooling provides better security

<details>
<summary>See Answer</summary>

In session pooling, a database connection is held for the entire client session. In transaction pooling, connections are returned to the pool after each transaction completes. For web applications with short-lived requests, this means 1000 application connections can share 20 database connections. Session pooling would require 1000 database connections, exceeding Cloud SQL limits.

</details>

---

### Question 8: Pub/Sub messages are processed out of order and some are lost. Implement reliable messaging.

**Type:** Practical | **Category:** Messaging

## The Scenario

Your order processing system uses Pub/Sub but has issues:

```
Publisher: Order created → Order updated → Order shipped
Subscriber receives: Order shipped → Order created → Order updated

Issues observed:
- Messages processed out of order
- Some messages processed twice (duplicate charges!)
- Messages occasionally lost (customers never notified)
- Dead letter queue growing with failed messages
```

## The Challenge

Implement reliable Pub/Sub messaging with ordering guarantees, exactly-once processing, and proper error handling.


### Step 1: Enable Message Ordering

```bash
# Create topic with message ordering enabled
gcloud pubsub topics create orders \
  --message-ordering

# Create subscription with ordering
gcloud pubsub subscriptions create orders-processor \
  --topic=orders \
  --enable-message-ordering \
  --ack-deadline=60
```

```python
# Publisher: Use ordering key for related messages
from google.cloud import pubsub_v1

publisher = pubsub_v1.PublisherClient()
topic_path = publisher.topic_path('project', 'orders')

def publish_order_event(order_id: str, event_type: str, data: dict):
    """Publish order event with ordering guarantee."""
    message_data = json.dumps({
        'order_id': order_id,
        'event_type': event_type,
        'data': data,
        'timestamp': datetime.utcnow().isoformat()
    }).encode('utf-8')

    # ordering_key ensures messages with same key are delivered in order
    future = publisher.publish(
        topic_path,
        message_data,
        ordering_key=order_id  # All events for same order are ordered
    )

    return future.result()

# Usage:
publish_order_event('order-123', 'created', {'items': [...]})
publish_order_event('order-123', 'updated', {'status': 'paid'})
publish_order_event('order-123', 'shipped', {'tracking': 'UPS123'})
# These will be delivered in order!
```

### Step 2: Implement Idempotent Processing

```python

from google.cloud import firestore

db = firestore.Client()

def process_message_idempotently(message):
    """Process message exactly once using deduplication."""

    # Generate unique message ID
    message_id = message.message_id
    # Or create your own: hashlib.sha256(message.data).hexdigest()

    # Check if already processed
    doc_ref = db.collection('processed_messages').document(message_id)

    @firestore.transactional
    def process_in_transaction(transaction):
        doc = doc_ref.get(transaction=transaction)

        if doc.exists:
            print(f"Message {message_id} already processed, skipping")
            return False

        # Process the message
        data = json.loads(message.data.decode('utf-8'))
        result = process_order_event(data)

        # Mark as processed
        transaction.set(doc_ref, {
            'processed_at': firestore.SERVER_TIMESTAMP,
            'result': result
        })

        return True

    transaction = db.transaction()
    return process_in_transaction(transaction)


def callback(message):
    try:
        if process_message_idempotently(message):
            message.ack()
        else:
            message.ack()  # Already processed, still ack
    except Exception as e:
        print(f"Error processing message: {e}")
        message.nack()  # Will be redelivered
```

### Step 3: Configure Dead Letter Topic

```hcl
# Terraform configuration
resource "google_pubsub_topic" "orders" {
  name = "orders"
}

resource "google_pubsub_topic" "orders_dead_letter" {
  name = "orders-dead-letter"
}

resource "google_pubsub_subscription" "orders_processor" {
  name  = "orders-processor"
  topic = google_pubsub_topic.orders.name

  # Enable message ordering
  enable_message_ordering = true

  # Acknowledgment deadline
  ack_deadline_seconds = 60

  # Retry policy
  retry_policy {
    minimum_backoff = "10s"
    maximum_backoff = "600s"  # 10 minutes max
  }

  # Dead letter policy
  dead_letter_policy {
    dead_letter_topic     = google_pubsub_topic.orders_dead_letter.id
    max_delivery_attempts = 5
  }

  # Message retention
  message_retention_duration = "604800s"  # 7 days

  # Expiration (never expire)
  expiration_policy {
    ttl = ""
  }
}

# Grant Pub/Sub permission to publish to dead letter topic
resource "google_pubsub_topic_iam_member" "dead_letter_publisher" {
  topic  = google_pubsub_topic.orders_dead_letter.name
  role   = "roles/pubsub.publisher"
  member = "serviceAccount:service-${var.project_number}@gcp-sa-pubsub.iam.gserviceaccount.com"
}

# Subscription to process dead letters
resource "google_pubsub_subscription" "dead_letter_processor" {
  name  = "orders-dead-letter-processor"
  topic = google_pubsub_topic.orders_dead_letter.name

  # Keep dead letters longer for investigation
  message_retention_duration = "2592000s"  # 30 days
}
```

### Step 4: Implement Robust Subscriber

```python
from google.cloud import pubsub_v1
from concurrent.futures import TimeoutError


class OrderProcessor:
    def __init__(self):
        self.subscriber = pubsub_v1.SubscriberClient()
        self.subscription_path = self.subscriber.subscription_path(
            'project', 'orders-processor'
        )
        self.streaming_pull_future = None

    def callback(self, message):
        """Process message with proper error handling."""
        try:
            # Extend ack deadline for long processing
            message.modify_ack_deadline(60)

            data = json.loads(message.data.decode('utf-8'))
            order_id = message.ordering_key

            print(f"Processing {data['event_type']} for order {order_id}")

            # Process with idempotency
            if self.process_idempotently(message.message_id, data):
                message.ack()
                print(f"Successfully processed message {message.message_id}")
            else:
                message.ack()  # Already processed
                print(f"Duplicate message {message.message_id}, skipped")

        except json.JSONDecodeError as e:
            # Invalid message format - send to dead letter
            print(f"Invalid JSON, sending to dead letter: {e}")
            message.nack()

        except TransientError as e:
            # Temporary failure - retry
            print(f"Transient error, will retry: {e}")
            message.nack()

        except PermanentError as e:
            # Permanent failure - don't retry
            print(f"Permanent error, sending to dead letter: {e}")
            message.ack()  # Ack to prevent infinite retries
            self.publish_to_dead_letter(message, str(e))

    def start(self):
        """Start the subscriber with flow control."""
        flow_control = pubsub_v1.types.FlowControl(
            max_messages=100,        # Max outstanding messages
            max_bytes=10 * 1024 * 1024,  # 10 MB
        )

        self.streaming_pull_future = self.subscriber.subscribe(
            self.subscription_path,
            callback=self.callback,
            flow_control=flow_control,
        )

        print(f"Listening on {self.subscription_path}")

        # Handle shutdown gracefully
        def shutdown(signum, frame):
            print("Shutting down...")
            self.streaming_pull_future.cancel()
            sys.exit(0)

        signal.signal(signal.SIGTERM, shutdown)
        signal.signal(signal.SIGINT, shutdown)

        try:
            self.streaming_pull_future.result()
        except TimeoutError:
            self.streaming_pull_future.cancel()
            self.streaming_pull_future.result()

if __name__ == "__main__":
    processor = OrderProcessor()
    processor.start()
```

### Step 5: Monitor and Alert

```hcl
# Alert on dead letter queue growth
resource "google_monitoring_alert_policy" "dead_letter_alert" {
  display_name = "Pub/Sub Dead Letter Queue Growing"

  conditions {
    display_name = "Dead letter messages > 100"

    condition_threshold {
      filter = <<-EOT
        resource.type="pubsub_subscription" AND
        resource.labels.subscription_id="orders-dead-letter-processor" AND
        metric.type="pubsub.googleapis.com/subscription/num_undelivered_messages"
      EOT

      comparison      = "COMPARISON_GT"
      threshold_value = 100
      duration        = "300s"

      aggregations {
        alignment_period   = "60s"
        per_series_aligner = "ALIGN_MEAN"
      }
    }
  }

  notification_channels = [google_monitoring_notification_channel.email.id]
}

# Alert on oldest unacked message
resource "google_monitoring_alert_policy" "message_age_alert" {
  display_name = "Pub/Sub Messages Backing Up"

  conditions {
    display_name = "Oldest message > 5 minutes"

    condition_threshold {
      filter = <<-EOT
        resource.type="pubsub_subscription" AND
        resource.labels.subscription_id="orders-processor" AND
        metric.type="pubsub.googleapis.com/subscription/oldest_unacked_message_age"
      EOT

      comparison      = "COMPARISON_GT"
      threshold_value = 300  # 5 minutes
      duration        = "60s"
    }
  }
}
```

### Step 6: Handle Ordering Key Failures

```python
from google.api_core.exceptions import FailedPrecondition

def publish_with_ordering_recovery(publisher, topic, order_id, data):
    """Publish with automatic recovery from ordering failures."""
    try:
        future = publisher.publish(
            topic,
            data.encode('utf-8'),
            ordering_key=order_id
        )
        return future.result(timeout=30)

    except FailedPrecondition as e:
        # Ordering key is paused due to previous failure
        print(f"Resuming ordering key {order_id}")
        publisher.resume_publish(topic, order_id)

        # Retry the publish
        future = publisher.publish(
            topic,
            data.encode('utf-8'),
            ordering_key=order_id
        )
        return future.result(timeout=30)
```


## Message Delivery Guarantees

| Configuration | Ordering | Duplicates | Use Case |
|--------------|----------|------------|----------|
| Default | No guarantee | Possible | Independent events |
| Ordering key | Within key | Possible | Related events |
| + Idempotency | Within key | Prevented | Financial transactions |
| + Exactly-once | Within key | Prevented (native) | Critical processing |

## Pub/Sub Best Practices

1. **Use ordering keys** for related messages (same order, user, etc.)
2. **Implement idempotent processing** - assume duplicates will happen
3. **Configure dead letter topics** - don't lose failed messages
4. **Set appropriate ack deadlines** - match your processing time
5. **Use flow control** - prevent subscriber overload


---

### Quick Check

**Why does enabling message ordering in Pub/Sub only guarantee order for messages with the same ordering key?**

   A. Pub/Sub can only track one ordering key at a time
-> B. **Different ordering keys may be processed by different subscribers in parallel, so global ordering is impossible without blocking**
   C. It's a limitation of the underlying storage system
   D. Global ordering is available but costs extra

<details>
<summary>See Answer</summary>

Pub/Sub is designed for high throughput with parallel processing. Messages with different ordering keys can be delivered to different subscribers simultaneously. Only messages with the same ordering key are guaranteed to be delivered in order to a single subscriber. Global ordering would require serializing all messages through a single processor, which would severely limit throughput and scalability.

</details>

---

### Question 9: Design a global load balancing architecture with Cloud CDN for a multi-region application.

**Type:** Architecture | **Category:** Load Balancing

## The Scenario

Your application serves users globally and needs:
- Low latency for users worldwide (< 100ms)
- High availability (99.99% uptime target)
- Automatic failover between regions
- Static content cached at edge locations
- SSL termination with managed certificates

Current setup: Single region deployment with 300ms latency for users in Asia and Europe.

## The Challenge

Design a multi-region architecture using Global HTTP(S) Load Balancer, Cloud CDN, and backend services in multiple regions. Explain the traffic flow and failover behavior.


### Architecture Overview

```
                         Users Worldwide
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Global Anycast IP  │
                    │    (Single IP)       │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │  Global HTTP(S) LB   │
                    │  + Cloud CDN         │
                    │  + Cloud Armor       │
                    └──────────┬──────────┘
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
          ▼                    ▼                    ▼
    ┌───────────┐        ┌───────────┐        ┌───────────┐
    │us-central1│        │ europe-   │        │  asia-    │
    │ Backend   │        │  west1    │        │  east1    │
    │ Service   │        │  Backend  │        │  Backend  │
    └─────┬─────┘        └─────┬─────┘        └─────┬─────┘
          │                    │                    │
          ▼                    ▼                    ▼
    ┌───────────┐        ┌───────────┐        ┌───────────┐
    │ GKE / GCE │        │ GKE / GCE │        │ GKE / GCE │
    │  + Cloud  │        │  + Cloud  │        │  + Cloud  │
    │   SQL     │        │   SQL     │        │   SQL     │
    └───────────┘        └───────────┘        └───────────┘
```

### Step 1: Create Backend Services in Multiple Regions

```hcl
# Instance group in us-central1
resource "google_compute_instance_group_manager" "us" {
  name               = "app-mig-us"
  base_instance_name = "app"
  zone               = "us-central1-a"

  version {
    instance_template = google_compute_instance_template.app.id
  }

  target_size = 3

  named_port {
    name = "http"
    port = 8080
  }

  auto_healing_policies {
    health_check      = google_compute_health_check.app.id
    initial_delay_sec = 300
  }
}

# Instance group in europe-west1
resource "google_compute_instance_group_manager" "eu" {
  name               = "app-mig-eu"
  base_instance_name = "app"
  zone               = "europe-west1-b"

  version {
    instance_template = google_compute_instance_template.app.id
  }

  target_size = 3

  named_port {
    name = "http"
    port = 8080
  }
}

# Instance group in asia-east1
resource "google_compute_instance_group_manager" "asia" {
  name               = "app-mig-asia"
  base_instance_name = "app"
  zone               = "asia-east1-a"

  version {
    instance_template = google_compute_instance_template.app.id
  }

  target_size = 3

  named_port {
    name = "http"
    port = 8080
  }
}
```

### Step 2: Configure Health Checks

```hcl
resource "google_compute_health_check" "app" {
  name               = "app-health-check"
  check_interval_sec = 5
  timeout_sec        = 5
  healthy_threshold  = 2
  unhealthy_threshold = 3

  http_health_check {
    port         = 8080
    request_path = "/health"
  }

  log_config {
    enable = true
  }
}
```

### Step 3: Create Backend Service with Multiple Backends

```hcl
resource "google_compute_backend_service" "app" {
  name                  = "app-backend"
  protocol              = "HTTP"
  port_name             = "http"
  timeout_sec           = 30
  health_checks         = [google_compute_health_check.app.id]
  load_balancing_scheme = "EXTERNAL_MANAGED"

  # Enable Cloud CDN
  enable_cdn = true

  cdn_policy {
    cache_mode = "CACHE_ALL_STATIC"
    default_ttl = 3600
    max_ttl     = 86400

    cache_key_policy {
      include_host         = true
      include_protocol     = true
      include_query_string = false
    }
  }

  # Connection draining for graceful shutdown
  connection_draining_timeout_sec = 300

  # Logging
  log_config {
    enable      = true
    sample_rate = 1.0
  }

  # US backend (primary for US users)
  backend {
    group           = google_compute_instance_group_manager.us.instance_group
    balancing_mode  = "UTILIZATION"
    capacity_scaler = 1.0
    max_utilization = 0.8
  }

  # EU backend
  backend {
    group           = google_compute_instance_group_manager.eu.instance_group
    balancing_mode  = "UTILIZATION"
    capacity_scaler = 1.0
    max_utilization = 0.8
  }

  # Asia backend
  backend {
    group           = google_compute_instance_group_manager.asia.instance_group
    balancing_mode  = "UTILIZATION"
    capacity_scaler = 1.0
    max_utilization = 0.8
  }
}
```

### Step 4: Configure URL Map and SSL

```hcl
# URL Map for routing
resource "google_compute_url_map" "app" {
  name            = "app-url-map"
  default_service = google_compute_backend_service.app.id

  host_rule {
    hosts        = ["app.example.com"]
    path_matcher = "app"
  }

  path_matcher {
    name            = "app"
    default_service = google_compute_backend_service.app.id

    # Route static content to CDN-optimized backend
    path_rule {
      paths   = ["/static/*", "/images/*", "/css/*", "/js/*"]
      service = google_compute_backend_service.static.id
    }

    # Route API calls to backend service
    path_rule {
      paths   = ["/api/*"]
      service = google_compute_backend_service.api.id
    }
  }
}

# Google-managed SSL certificate
resource "google_compute_managed_ssl_certificate" "app" {
  name = "app-ssl-cert"

  managed {
    domains = ["app.example.com", "www.example.com"]
  }
}

# HTTPS proxy
resource "google_compute_target_https_proxy" "app" {
  name             = "app-https-proxy"
  url_map          = google_compute_url_map.app.id
  ssl_certificates = [google_compute_managed_ssl_certificate.app.id]

  ssl_policy = google_compute_ssl_policy.modern.id
}

# SSL policy (modern TLS only)
resource "google_compute_ssl_policy" "modern" {
  name            = "modern-ssl-policy"
  profile         = "MODERN"
  min_tls_version = "TLS_1_2"
}

# Global forwarding rule
resource "google_compute_global_forwarding_rule" "https" {
  name                  = "app-https-forwarding"
  target                = google_compute_target_https_proxy.app.id
  port_range            = "443"
  ip_address            = google_compute_global_address.app.address
  load_balancing_scheme = "EXTERNAL_MANAGED"
}

# Static global IP
resource "google_compute_global_address" "app" {
  name = "app-global-ip"
}
```

### Step 5: Add Cloud Armor for Security

```hcl
resource "google_compute_security_policy" "app" {
  name = "app-security-policy"

  # Default rule - allow
  rule {
    action   = "allow"
    priority = "2147483647"
    match {
      versioned_expr = "SRC_IPS_V1"
      config {
        src_ip_ranges = ["*"]
      }
    }
    description = "Default allow rule"
  }

  # Block known bad IPs
  rule {
    action   = "deny(403)"
    priority = "1000"
    match {
      versioned_expr = "SRC_IPS_V1"
      config {
        src_ip_ranges = ["192.0.2.0/24"]  # Example blocked range
      }
    }
    description = "Block malicious IPs"
  }

  # Rate limiting
  rule {
    action   = "rate_based_ban"
    priority = "2000"
    match {
      versioned_expr = "SRC_IPS_V1"
      config {
        src_ip_ranges = ["*"]
      }
    }
    rate_limit_options {
      conform_action = "allow"
      exceed_action  = "deny(429)"
      rate_limit_threshold {
        count        = 1000
        interval_sec = 60
      }
      ban_duration_sec = 600
    }
    description = "Rate limit - 1000 req/min per IP"
  }

  # OWASP ModSecurity rules
  rule {
    action   = "deny(403)"
    priority = "3000"
    match {
      expr {
        expression = "evaluatePreconfiguredExpr('sqli-v33-stable')"
      }
    }
    description = "SQL injection protection"
  }
}

# Attach security policy to backend
resource "google_compute_backend_service" "app" {
  # ... other config ...
  security_policy = google_compute_security_policy.app.id
}
```

### Step 6: Configure DNS

```hcl
resource "google_dns_record_set" "app" {
  name         = "app.example.com."
  managed_zone = google_dns_managed_zone.example.name
  type         = "A"
  ttl          = 300
  rrdatas      = [google_compute_global_address.app.address]
}
```

### Traffic Flow

```
1. User in Tokyo requests app.example.com
2. DNS resolves to global anycast IP
3. Request enters Google's network at Tokyo edge
4. Global LB routes to asia-east1 backend (closest)
5. If Asia region unhealthy, routes to US or EU
6. Static content served from CDN cache
7. Response returns via same path
```


## Load Balancing Comparison

| Feature | Global HTTP(S) | Regional | Network LB |
|---------|---------------|----------|------------|
| Scope | Global | Regional | Regional |
| Protocol | HTTP/HTTPS | HTTP/HTTPS | TCP/UDP |
| CDN | Yes | No | No |
| Anycast | Yes | No | No |
| SSL termination | Yes | Yes | No |
| Use case | Web apps | Internal | Low-level |

## Failover Behavior

| Scenario | Behavior | Recovery Time |
|----------|----------|---------------|
| Instance unhealthy | Routes to healthy instances | ~10 seconds |
| Zone failure | Routes to other zones | ~10 seconds |
| Region failure | Routes to other regions | ~10 seconds |
| All backends unhealthy | Returns 502 | Manual intervention |

---

### Question 10: Build a monitoring dashboard and alerting system that catches issues before users notice.

**Type:** Practical | **Category:** Observability

## The Scenario

Your team discovers issues from user complaints, not monitoring:

```
Timeline of typical incident:
09:00 - Error rate increases to 5%
09:15 - Database connections exhausted
09:30 - First user complaint received
09:45 - Team starts investigating
10:00 - Root cause identified
10:30 - Issue resolved

Total user impact: 1.5 hours
```

You need a monitoring system that would have caught this at 09:00.

## The Challenge

Design a comprehensive monitoring and alerting strategy using Cloud Monitoring, with SLIs/SLOs, proactive alerts, and runbooks for common issues.


### Step 1: Define Service Level Indicators (SLIs)

```yaml
# SLIs based on user experience
availability_sli:
  description: "Percentage of successful requests"
  calculation: |
    successful_requests / total_requests
  good_threshold: "&gt; 99.9%"

latency_sli:
  description: "P95 request latency"
  calculation: |
    95th percentile of request duration
  good_threshold: "&lt; 500ms"

error_rate_sli:
  description: "Percentage of 5xx errors"
  calculation: |
    5xx_responses / total_responses
  good_threshold: "&lt; 0.1%"
```

### Step 2: Create Custom Metrics

```python
from google.cloud import monitoring_v3


client = monitoring_v3.MetricServiceClient()
project_name = f"projects/{PROJECT_ID}"

def record_custom_metric(metric_type, value, labels=None):
    """Record a custom metric to Cloud Monitoring."""
    series = monitoring_v3.TimeSeries()
    series.metric.type = f"custom.googleapis.com/{metric_type}"

    if labels:
        series.metric.labels.update(labels)

    series.resource.type = "global"

    point = monitoring_v3.Point()
    point.value.double_value = value
    point.interval.end_time.seconds = int(time.time())
    series.points = [point]

    client.create_time_series(name=project_name, time_series=[series])

# Usage: Track business metrics
record_custom_metric("orders/processing_time", 1.5, {"status": "success"})
record_custom_metric("payments/amount", 99.99, {"currency": "USD"})
```

### Step 3: Create Monitoring Dashboard

```hcl
resource "google_monitoring_dashboard" "app" {
  dashboard_json = jsonencode({
    displayName = "Application Health Dashboard"

    mosaicLayout = {
      columns = 12
      tiles = [
        # Request Rate
        {
          width  = 4
          height = 4
          widget = {
            title = "Request Rate"
            xyChart = {
              dataSets = [{
                timeSeriesQuery = {
                  timeSeriesFilter = {
                    filter = "metric.type=\"loadbalancing.googleapis.com/https/request_count\" resource.type=\"https_lb_rule\""
                    aggregation = {
                      alignmentPeriod  = "60s"
                      perSeriesAligner = "ALIGN_RATE"
                    }
                  }
                }
              }]
            }
          }
        },

        # Error Rate
        {
          width  = 4
          height = 4
          xPos   = 4
          widget = {
            title = "Error Rate (%)"
            xyChart = {
              dataSets = [{
                timeSeriesQuery = {
                  timeSeriesFilterRatio = {
                    numerator = {
                      filter = "metric.type=\"loadbalancing.googleapis.com/https/request_count\" metric.labels.response_code_class=\"500\""
                    }
                    denominator = {
                      filter = "metric.type=\"loadbalancing.googleapis.com/https/request_count\""
                    }
                  }
                }
              }]
              thresholds = [{
                value     = 0.01
                color     = "YELLOW"
                direction = "ABOVE"
              }, {
                value     = 0.05
                color     = "RED"
                direction = "ABOVE"
              }]
            }
          }
        },

        # P95 Latency
        {
          width  = 4
          height = 4
          xPos   = 8
          widget = {
            title = "P95 Latency (ms)"
            xyChart = {
              dataSets = [{
                timeSeriesQuery = {
                  timeSeriesFilter = {
                    filter = "metric.type=\"loadbalancing.googleapis.com/https/total_latencies\""
                    aggregation = {
                      alignmentPeriod    = "60s"
                      perSeriesAligner   = "ALIGN_PERCENTILE_95"
                    }
                  }
                }
              }]
            }
          }
        },

        # Database Connections
        {
          width  = 6
          height = 4
          yPos   = 4
          widget = {
            title = "Cloud SQL Connections"
            xyChart = {
              dataSets = [{
                timeSeriesQuery = {
                  timeSeriesFilter = {
                    filter = "metric.type=\"cloudsql.googleapis.com/database/postgresql/num_backends\""
                  }
                }
              }]
            }
          }
        },

        # GKE Pod Status
        {
          width  = 6
          height = 4
          xPos   = 6
          yPos   = 4
          widget = {
            title = "GKE Pod Status"
            xyChart = {
              dataSets = [{
                timeSeriesQuery = {
                  timeSeriesFilter = {
                    filter = "metric.type=\"kubernetes.io/container/restart_count\" resource.type=\"k8s_container\""
                    aggregation = {
                      alignmentPeriod  = "300s"
                      perSeriesAligner = "ALIGN_DELTA"
                    }
                  }
                }
              }]
            }
          }
        }
      ]
    }
  })
}
```

### Step 4: Create Alert Policies

```hcl
# Critical: High Error Rate
resource "google_monitoring_alert_policy" "high_error_rate" {
  display_name = "[CRITICAL] High Error Rate"
  combiner     = "OR"

  conditions {
    display_name = "Error rate > 5%"

    condition_threshold {
      filter = <<-EOT
        metric.type="loadbalancing.googleapis.com/https/request_count"
        AND metric.labels.response_code_class="500"
      EOT

      aggregations {
        alignment_period     = "60s"
        per_series_aligner   = "ALIGN_RATE"
        cross_series_reducer = "REDUCE_SUM"
      }

      denominator_filter = <<-EOT
        metric.type="loadbalancing.googleapis.com/https/request_count"
      EOT

      denominator_aggregations {
        alignment_period     = "60s"
        per_series_aligner   = "ALIGN_RATE"
        cross_series_reducer = "REDUCE_SUM"
      }

      comparison      = "COMPARISON_GT"
      threshold_value = 0.05
      duration        = "60s"

      trigger {
        count = 1
      }
    }
  }

  notification_channels = [
    google_monitoring_notification_channel.pagerduty.id,
    google_monitoring_notification_channel.slack_critical.id
  ]

  documentation {
    content = <<-EOT
      ## High Error Rate Alert

      ### Impact
      Users are experiencing errors. Error rate exceeds 5%.

      ### Runbook
      1. Check Cloud Logging for error patterns:
         `resource.type="k8s_container" severity>=ERROR`

      2. Check backend health:
         `gcloud compute backend-services get-health app-backend --global`

      3. Check database connections:
         `gcloud sql instances describe production-db`

      4. Recent deployments:
         `kubectl rollout history deployment/api-server`

      ### Escalation
      If not resolved in 15 minutes, page on-call manager.
    EOT
    mime_type = "text/markdown"
  }

  alert_strategy {
    auto_close = "1800s"  # Auto-close after 30 min if resolved
  }
}

# Warning: Elevated Latency
resource "google_monitoring_alert_policy" "elevated_latency" {
  display_name = "[WARNING] Elevated Latency"
  combiner     = "OR"

  conditions {
    display_name = "P95 latency > 1s"

    condition_threshold {
      filter = "metric.type=\"loadbalancing.googleapis.com/https/total_latencies\""

      aggregations {
        alignment_period   = "300s"
        per_series_aligner = "ALIGN_PERCENTILE_95"
      }

      comparison      = "COMPARISON_GT"
      threshold_value = 1000  # 1 second in ms
      duration        = "300s"
    }
  }

  notification_channels = [
    google_monitoring_notification_channel.slack_warning.id
  ]
}

# Anomaly Detection: Traffic Drop
resource "google_monitoring_alert_policy" "traffic_anomaly" {
  display_name = "[WARNING] Traffic Anomaly Detected"
  combiner     = "OR"

  conditions {
    display_name = "Traffic below expected range"

    condition_threshold {
      filter = "metric.type=\"loadbalancing.googleapis.com/https/request_count\""

      aggregations {
        alignment_period     = "300s"
        per_series_aligner   = "ALIGN_RATE"
        cross_series_reducer = "REDUCE_SUM"
      }

      # Compare to same time last week
      comparison      = "COMPARISON_LT"
      threshold_value = 100  # Adjust based on baseline

      # Use forecast for dynamic threshold
      forecast_options {
        forecast_horizon = "3600s"
      }

      duration = "600s"
    }
  }
}
```

### Step 5: Set Up Notification Channels

```hcl
# PagerDuty for critical alerts
resource "google_monitoring_notification_channel" "pagerduty" {
  display_name = "PagerDuty"
  type         = "pagerduty"

  labels = {
    service_key = var.pagerduty_service_key
  }
}

# Slack for warnings
resource "google_monitoring_notification_channel" "slack_warning" {
  display_name = "Slack #alerts-warning"
  type         = "slack"

  labels = {
    channel_name = "#alerts-warning"
  }

  sensitive_labels {
    auth_token = var.slack_token
  }
}

# Email for daily digests
resource "google_monitoring_notification_channel" "email" {
  display_name = "Team Email"
  type         = "email"

  labels = {
    email_address = "team@example.com"
  }
}
```

### Step 6: Create SLO Monitoring

```hcl
resource "google_monitoring_slo" "availability" {
  service      = google_monitoring_custom_service.app.service_id
  slo_id       = "availability-slo"
  display_name = "99.9% Availability"

  goal                = 0.999
  rolling_period_days = 30

  request_based_sli {
    good_total_ratio {
      good_service_filter = <<-EOT
        metric.type="loadbalancing.googleapis.com/https/request_count"
        metric.labels.response_code_class!="500"
      EOT
      total_service_filter = <<-EOT
        metric.type="loadbalancing.googleapis.com/https/request_count"
      EOT
    }
  }
}

# Alert when burning through error budget too fast
resource "google_monitoring_alert_policy" "slo_burn_rate" {
  display_name = "SLO Burn Rate Alert"
  combiner     = "OR"

  conditions {
    display_name = "Burning error budget too fast"

    condition_threshold {
      filter = <<-EOT
        select_slo_burn_rate(
          "projects/${var.project}/services/${google_monitoring_custom_service.app.service_id}/serviceLevelObjectives/${google_monitoring_slo.availability.slo_id}",
          "1h"
        )
      EOT

      comparison      = "COMPARISON_GT"
      threshold_value = 10  # 10x normal burn rate
      duration        = "0s"
    }
  }
}
```


## Alert Severity Matrix

| Severity | Response Time | Notification | Examples |
|----------|--------------|--------------|----------|
| Critical | 5 minutes | PagerDuty + Slack | &gt;5% errors, service down |
| Warning | 30 minutes | Slack | &gt;1% errors, high latency |
| Info | Next business day | Email | SLO trending down |

## Golden Signals

| Signal | Metric | Alert Threshold |
|--------|--------|-----------------|
| Latency | P95 response time | &gt; 1 second |
| Traffic | Requests per second | Anomaly detection |
| Errors | 5xx error rate | &gt; 0.1% |
| Saturation | CPU/Memory usage | &gt; 80% |

---

### Question 11: Implement a CI/CD pipeline with Cloud Build that deploys to GKE with canary releases.

**Type:** Practical | **Category:** CI/CD

## The Scenario

Your team deploys manually to GKE:
- Deployments take 2 hours of engineer time
- No consistent testing before deployment
- Rollbacks are painful and slow
- A bad deployment last month caused 4 hours of downtime

You need an automated pipeline with safety guardrails.

## The Challenge

Design and implement a CI/CD pipeline using Cloud Build that builds, tests, and deploys to GKE with canary releases. Include automated rollback on failure.


> **Common Mistake:** A junior engineer might deploy directly to production without staging, skip tests to speed up deployment, use kubectl apply without health checks, or implement canary manually with percentage-based replica counts. These approaches risk production outages, miss bugs, and make rollbacks difficult.

> **Senior Engineer Approach:** A senior engineer implements a multi-stage pipeline: build and test, push to Artifact Registry, deploy to staging, run integration tests, canary deployment to production with automated metrics validation, then gradual rollout or automatic rollback.

### Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Cloud Build Pipeline                      │
├─────────────────────────────────────────────────────────────────┤
│  PR Trigger                    Push to Main                      │
│      │                              │                            │
│      ▼                              ▼                            │
│  ┌────────┐                    ┌────────┐                       │
│  │  Lint  │                    │ Build  │                       │
│  │  Test  │                    │  Test  │                       │
│  └────────┘                    └───┬────┘                       │
│                                    │                            │
│                                    ▼                            │
│                              ┌─────────────┐                    │
│                              │Push to      │                    │
│                              │Artifact     │                    │
│                              │Registry     │                    │
│                              └──────┬──────┘                    │
│                                     │                            │
│                    ┌────────────────┴────────────────┐          │
│                    │                                 │          │
│                    ▼                                 ▼          │
│              ┌───────────┐                   ┌─────────────┐   │
│              │  Deploy   │                   │   Deploy    │   │
│              │  Staging  │                   │   Canary    │   │
│              └─────┬─────┘                   │   (10%)     │   │
│                    │                         └──────┬──────┘   │
│                    ▼                                │          │
│              ┌───────────┐                         │          │
│              │Integration│           ┌─────────────┤          │
│              │   Tests   │           │             │          │
│              └───────────┘           ▼             ▼          │
│                              ┌─────────────┐ ┌──────────┐    │
│                              │   Metrics   │ │ Rollback │    │
│                              │ Validation  │ │ (if bad) │    │
│                              └──────┬──────┘ └──────────┘    │
│                                     │                         │
│                                     ▼                         │
│                              ┌─────────────┐                  │
│                              │   Full      │                  │
│                              │  Rollout    │                  │
│                              └─────────────┘                  │
└─────────────────────────────────────────────────────────────────┘
```

### Step 1: Cloud Build Configuration

```yaml
# cloudbuild.yaml
steps:
  # Step 1: Run linting and unit tests
  - id: 'test'
    name: 'node:18'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        npm ci
        npm run lint
        npm run test:unit

  # Step 2: Build Docker image
  - id: 'build'
    name: 'gcr.io/cloud-builders/docker'
    args:
      - 'build'
      - '-t'
      - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO}/${_SERVICE}:${SHORT_SHA}'
      - '-t'
      - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO}/${_SERVICE}:latest'
      - '--cache-from'
      - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO}/${_SERVICE}:latest'
      - '.'

  # Step 3: Run security scan
  - id: 'scan'
    name: 'gcr.io/cloud-builders/gcloud'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        gcloud artifacts docker images scan \
          ${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO}/${_SERVICE}:${SHORT_SHA} \
          --format='json' > /workspace/scan-results.json

        # Fail if critical vulnerabilities found
        CRITICAL=$(cat /workspace/scan-results.json | jq '.vulnerabilities[] | select(.severity=="CRITICAL")' | wc -l)
        if [ "$CRITICAL" -gt "0" ]; then
          echo "Critical vulnerabilities found!"
          exit 1
        fi

  # Step 4: Push to Artifact Registry
  - id: 'push'
    name: 'gcr.io/cloud-builders/docker'
    args:
      - 'push'
      - '--all-tags'
      - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO}/${_SERVICE}'

  # Step 5: Deploy to staging
  - id: 'deploy-staging'
    name: 'gcr.io/cloud-builders/gke-deploy'
    args:
      - 'run'
      - '--filename=k8s/'
      - '--image=${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO}/${_SERVICE}:${SHORT_SHA}'
      - '--cluster=${_STAGING_CLUSTER}'
      - '--location=${_REGION}'
      - '--namespace=staging'

  # Step 6: Run integration tests against staging
  - id: 'integration-tests'
    name: 'gcr.io/cloud-builders/gcloud'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        # Wait for deployment to be ready
        kubectl rollout status deployment/${_SERVICE} -n staging --timeout=300s

        # Run integration tests
        npm run test:integration -- --baseUrl=https://staging.example.com

  # Step 7: Deploy canary to production (10%)
  - id: 'deploy-canary'
    name: 'gcr.io/cloud-builders/gke-deploy'
    args:
      - 'run'
      - '--filename=k8s/canary/'
      - '--image=${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO}/${_SERVICE}:${SHORT_SHA}'
      - '--cluster=${_PROD_CLUSTER}'
      - '--location=${_REGION}'
      - '--namespace=production'

  # Step 8: Validate canary metrics
  - id: 'validate-canary'
    name: 'gcr.io/cloud-builders/gcloud'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        # Wait for canary to receive traffic
        sleep 300

        # Check error rate for canary
        ERROR_RATE=$(gcloud monitoring metrics-scopes list \
          --filter="metric.type=custom.googleapis.com/http/error_rate AND resource.labels.version=canary" \
          --format="value(points[0].value.doubleValue)")

        if (( $(echo "$ERROR_RATE > 0.01" | bc -l) )); then
          echo "Canary error rate too high: $ERROR_RATE"
          exit 1
        fi

        echo "Canary validation passed"

  # Step 9: Full production rollout
  - id: 'deploy-production'
    name: 'gcr.io/cloud-builders/gke-deploy'
    args:
      - 'run'
      - '--filename=k8s/production/'
      - '--image=${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO}/${_SERVICE}:${SHORT_SHA}'
      - '--cluster=${_PROD_CLUSTER}'
      - '--location=${_REGION}'
      - '--namespace=production'

substitutions:
  _REGION: us-central1
  _REPO: app-images
  _SERVICE: api-server
  _STAGING_CLUSTER: staging-cluster
  _PROD_CLUSTER: prod-cluster

options:
  machineType: 'E2_HIGHCPU_8'
  logging: CLOUD_LOGGING_ONLY

timeout: '1800s'
```

### Step 2: Canary Deployment Manifests

```yaml
# k8s/canary/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server-canary
  labels:
    app: api-server
    version: canary
spec:
  replicas: 1  # Small canary
  selector:
    matchLabels:
      app: api-server
      version: canary
  template:
    metadata:
      labels:
        app: api-server
        version: canary
    spec:
      containers:
      - name: api
        image: IMAGE_PLACEHOLDER  # Replaced by Cloud Build
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: "250m"
            memory: "512Mi"
          limits:
            cpu: "500m"
            memory: "1Gi"
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
---
# k8s/canary/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: api-server
spec:
  selector:
    app: api-server
    # No version selector - routes to both stable and canary
  ports:
  - port: 80
    targetPort: 8080
```

### Step 3: Production Deployment with Rolling Update

```yaml
# k8s/production/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
  labels:
    app: api-server
    version: stable
spec:
  replicas: 5
  selector:
    matchLabels:
      app: api-server
      version: stable
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 2
  template:
    metadata:
      labels:
        app: api-server
        version: stable
    spec:
      containers:
      - name: api
        image: IMAGE_PLACEHOLDER
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: "250m"
            memory: "512Mi"
          limits:
            cpu: "500m"
            memory: "1Gi"
```

### Step 4: Cloud Build Trigger Configuration

```hcl
# Terraform configuration for triggers
resource "google_cloudbuild_trigger" "pr_trigger" {
  name        = "pr-validation"
  description = "Run tests on pull requests"

  github {
    owner = "myorg"
    name  = "myrepo"

    pull_request {
      branch = "^main$"
    }
  }

  filename = "cloudbuild-pr.yaml"
}

resource "google_cloudbuild_trigger" "deploy_trigger" {
  name        = "deploy-to-production"
  description = "Build and deploy on push to main"

  github {
    owner = "myorg"
    name  = "myrepo"

    push {
      branch = "^main$"
    }
  }

  filename = "cloudbuild.yaml"

  # Required approvals for production
  approval_config {
    approval_required = true
  }
}
```

### Step 5: Automatic Rollback Script

```yaml
# cloudbuild-rollback.yaml
steps:
  - id: 'rollback'
    name: 'gcr.io/cloud-builders/kubectl'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        gcloud container clusters get-credentials ${_CLUSTER} --region=${_REGION}

        # Get previous revision
        PREV_REVISION=$(kubectl rollout history deployment/${_SERVICE} -n production | tail -3 | head -1 | awk '{print $1}')

        # Rollback to previous
        kubectl rollout undo deployment/${_SERVICE} -n production --to-revision=$PREV_REVISION

        # Wait for rollback
        kubectl rollout status deployment/${_SERVICE} -n production --timeout=300s

        # Delete canary
        kubectl delete deployment ${_SERVICE}-canary -n production --ignore-not-found

substitutions:
  _SERVICE: api-server
  _CLUSTER: prod-cluster
  _REGION: us-central1
```

### Step 6: Service Account Permissions

```hcl
resource "google_service_account" "cloudbuild" {
  account_id   = "cloudbuild-deployer"
  display_name = "Cloud Build Deployer"
}

# GKE access
resource "google_project_iam_member" "cloudbuild_gke" {
  project = var.project
  role    = "roles/container.developer"
  member  = "serviceAccount:${google_service_account.cloudbuild.email}"
}

# Artifact Registry access
resource "google_project_iam_member" "cloudbuild_artifact" {
  project = var.project
  role    = "roles/artifactregistry.writer"
  member  = "serviceAccount:${google_service_account.cloudbuild.email}"
}

# Logging access
resource "google_project_iam_member" "cloudbuild_logs" {
  project = var.project
  role    = "roles/logging.logWriter"
  member  = "serviceAccount:${google_service_account.cloudbuild.email}"
}
```


## Deployment Strategy Comparison

| Strategy | Risk | Rollback Speed | Complexity |
|----------|------|----------------|------------|
| Big Bang | High | Slow (redeploy) | Low |
| Rolling Update | Medium | Medium (rollback) | Low |
| Blue-Green | Low | Fast (switch) | Medium |
| Canary | Very Low | Fast (delete canary) | High |

## Pipeline Best Practices

1. **Immutable images** - Tag with commit SHA, not 'latest'
2. **Run tests before deploy** - Unit, integration, security
3. **Deploy to staging first** - Catch issues before production
4. **Use canary deployments** - Validate with real traffic
5. **Automate rollbacks** - Don't rely on manual intervention


---

### Quick Check

**Why should canary deployments validate metrics rather than just checking if pods are healthy?**

   A. Pod health checks are not reliable in GKE
-> B. **Pods can be healthy while serving errors or slow responses that impact users**
   C. Metrics validation is faster than health checks
   D. GKE requires metrics validation for canary deployments

<details>
<summary>See Answer</summary>

A pod can pass health checks (responding to /health) while still having issues like database query errors, slow external API calls, or logic bugs that cause 500 errors. Metrics validation checks real user-facing signals like error rate, latency percentiles, and success rate. This catches issues that health checks miss, preventing bad deployments from reaching all users.

</details>

---

### Question 12: Your GCP bill increased 40% last month. Identify waste and implement cost controls.

**Type:** Practical | **Category:** Cost Optimization

## The Scenario

Your GCP bill jumped from $50,000 to $70,000 last month:

```
Cost breakdown:
├── Compute Engine: $25,000 (+50%)
├── BigQuery: $15,000 (+100%)
├── Cloud Storage: $12,000 (+20%)
├── GKE: $10,000 (+30%)
└── Other: $8,000 (+10%)

Finance is asking for explanations and a plan to reduce costs.
```

## The Challenge

Analyze the cost increase, identify optimization opportunities, implement cost controls, and create a sustainable cost management strategy.


> **Common Mistake:** A junior engineer might immediately delete resources to cut costs, downsize all instances without analysis, turn off non-production environments entirely, or ignore the problem hoping it resolves. These approaches cause outages, performance issues, block development, or let costs spiral further.

> **Senior Engineer Approach:** A senior engineer uses billing reports and cost analysis to identify specific causes, implements committed use discounts for predictable workloads, right-sizes resources based on utilization data, sets up budgets and alerts, and creates a culture of cost awareness with showback/chargeback.

### Step 1: Analyze Cost Breakdown

```bash
# Export detailed billing to BigQuery for analysis
bq query --use_legacy_sql=false '
SELECT
  service.description as service,
  sku.description as sku,
  project.id as project,
  SUM(cost) as total_cost,
  SUM(usage.amount) as usage_amount,
  usage.unit
FROM `billing_export.gcp_billing_export_v1_*`
WHERE _PARTITIONTIME >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
GROUP BY 1, 2, 3, 6
ORDER BY total_cost DESC
LIMIT 50'

# Find cost spikes by day
bq query --use_legacy_sql=false '
SELECT
  DATE(usage_start_time) as date,
  service.description as service,
  SUM(cost) as daily_cost
FROM `billing_export.gcp_billing_export_v1_*`
WHERE _PARTITIONTIME >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
GROUP BY 1, 2
ORDER BY 1, 3 DESC'
```

### Step 2: Identify Compute Waste

```bash
# Find idle VMs (low CPU utilization)
gcloud recommender recommendations list \
  --project=my-project \
  --location=us-central1-a \
  --recommender=google.compute.instance.IdleResourceRecommender \
  --format="table(content.overview.resourceName,content.overview.utilizationStats)"

# Find oversized VMs
gcloud recommender recommendations list \
  --project=my-project \
  --location=us-central1-a \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --format="table(content.overview.resourceName,content.overview.recommendedMachineType)"

# List unattached disks
gcloud compute disks list \
  --filter="NOT users:*" \
  --format="table(name,sizeGb,zone,status)"
```

### Step 3: Implement Committed Use Discounts

```hcl
# 1-year commitment for predictable workloads (37% discount)
resource "google_compute_commitment" "cpu_commitment" {
  name   = "cpu-1year-commitment"
  region = "us-central1"
  type   = "COMPUTE_OPTIMIZED"
  plan   = "TWELVE_MONTH"

  resources {
    type   = "VCPU"
    amount = 100  # 100 vCPUs committed
  }

  resources {
    type   = "MEMORY"
    amount = 400  # 400 GB RAM committed
  }
}

# 3-year commitment for stable workloads (57% discount)
resource "google_compute_commitment" "cpu_commitment_3yr" {
  name   = "cpu-3year-commitment"
  region = "us-central1"
  type   = "GENERAL_PURPOSE"
  plan   = "THIRTY_SIX_MONTH"

  resources {
    type   = "VCPU"
    amount = 50
  }

  resources {
    type   = "MEMORY"
    amount = 200
  }
}
```

### Step 4: Right-Size GKE Clusters

```hcl
# Enable cluster autoscaler with appropriate limits
resource "google_container_node_pool" "primary" {
  name       = "primary-pool"
  cluster    = google_container_cluster.main.name
  location   = "us-central1"

  # Autoscaling based on actual demand
  autoscaling {
    min_node_count  = 2   # Minimum for HA
    max_node_count  = 20  # Cap costs
    location_policy = "BALANCED"
  }

  node_config {
    # Use E2 instances for cost efficiency
    machine_type = "e2-standard-4"

    # Spot VMs for non-critical workloads (60-91% discount)
    spot = true

    # Only request what you need
    labels = {
      workload = "general"
    }
  }

  management {
    auto_repair  = true
    auto_upgrade = true
  }
}

# Separate node pool for critical workloads (on-demand)
resource "google_container_node_pool" "critical" {
  name    = "critical-pool"
  cluster = google_container_cluster.main.name

  autoscaling {
    min_node_count = 3
    max_node_count = 10
  }

  node_config {
    machine_type = "e2-standard-4"
    spot         = false  # On-demand for reliability

    taint {
      key    = "workload"
      value  = "critical"
      effect = "NO_SCHEDULE"
    }
  }
}
```

### Step 5: Optimize BigQuery Costs

```sql
-- Find expensive queries
SELECT
  user_email,
  query,
  ROUND(total_bytes_processed / POW(10,12), 2) as tb_processed,
  ROUND(total_bytes_processed / POW(10,12) * 5, 2) as cost_usd
FROM `region-us.INFORMATION_SCHEMA.JOBS_BY_PROJECT`
WHERE creation_time > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
  AND job_type = 'QUERY'
ORDER BY total_bytes_processed DESC
LIMIT 20;

-- Set maximum bytes billed per query (prevent runaway costs)
-- In BigQuery console or via API:
-- maximumBytesBilled = 10737418240 (10GB)
```

```hcl
# Consider flat-rate pricing for heavy usage
# Break-even: ~$10,000/month in on-demand = 100 slots worth

resource "google_bigquery_reservation" "default" {
  name     = "default-reservation"
  location = "US"
  slot_capacity = 100

  # Autoscale slots based on demand
  autoscale {
    max_slots = 200
  }
}
```

### Step 6: Set Up Budgets and Alerts

```hcl
resource "google_billing_budget" "project_budget" {
  billing_account = var.billing_account_id
  display_name    = "Monthly Project Budget"

  budget_filter {
    projects = ["projects/${var.project_id}"]
  }

  amount {
    specified_amount {
      currency_code = "USD"
      units         = "60000"  # $60,000 budget
    }
  }

  threshold_rules {
    threshold_percent = 0.5
    spend_basis       = "CURRENT_SPEND"
  }

  threshold_rules {
    threshold_percent = 0.8
    spend_basis       = "CURRENT_SPEND"
  }

  threshold_rules {
    threshold_percent = 1.0
    spend_basis       = "CURRENT_SPEND"
  }

  threshold_rules {
    threshold_percent = 1.0
    spend_basis       = "FORECASTED_SPEND"
  }

  all_updates_rule {
    monitoring_notification_channels = [
      google_monitoring_notification_channel.email.id,
      google_monitoring_notification_channel.slack.id
    ]
    disable_default_iam_recipients = false
  }
}
```

### Step 7: Implement Resource Labels for Cost Attribution

```hcl
# Standard labels for all resources
locals {
  standard_labels = {
    environment = var.environment
    team        = var.team
    service     = var.service
    cost_center = var.cost_center
  }
}

resource "google_compute_instance" "app" {
  # ...
  labels = local.standard_labels
}

resource "google_storage_bucket" "data" {
  # ...
  labels = local.standard_labels
}
```

```sql
-- Query costs by team
SELECT
  labels.value as team,
  SUM(cost) as total_cost
FROM `billing_export.gcp_billing_export_v1_*`
CROSS JOIN UNNEST(labels) as labels
WHERE labels.key = 'team'
  AND _PARTITIONTIME >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
GROUP BY 1
ORDER BY 2 DESC;
```

### Step 8: Automate Cost Optimization

```yaml
# Cloud Function to stop dev instances at night
# functions/auto-stop/main.py

from google.cloud import compute_v1

def stop_dev_instances(event, context):
    """Stop all instances labeled environment=dev."""
    client = compute_v1.InstancesClient()
    project = 'my-project'

    # List all zones
    zones_client = compute_v1.ZonesClient()
    zones = zones_client.list(project=project)

    for zone in zones:
        instances = client.list(project=project, zone=zone.name)

        for instance in instances:
            labels = instance.labels or {}
            if labels.get('environment') == 'dev' and instance.status == 'RUNNING':
                print(f"Stopping {instance.name} in {zone.name}")
                client.stop(project=project, zone=zone.name, instance=instance.name)
```

### Cost Optimization Summary

| Category | Action | Estimated Savings |
|----------|--------|-------------------|
| Compute | Right-size VMs | 20-40% |
| Compute | Committed use (1yr) | 37% |
| Compute | Spot VMs (non-critical) | 60-91% |
| GKE | Cluster autoscaler | 30-50% |
| BigQuery | Partitioning/clustering | 50-90% |
| Storage | Lifecycle policies | 40-70% |
| All | Delete unused resources | 10-20% |


## Cost Optimization Checklist

| Check | Tool | Frequency |
|-------|------|-----------|
| Idle VMs | Recommender | Weekly |
| Oversized VMs | Recommender | Weekly |
| Unattached disks | gcloud compute | Weekly |
| Expensive queries | INFORMATION_SCHEMA | Daily |
| Budget status | Billing console | Daily |
| Commitment coverage | Billing reports | Monthly |

## Cost Governance Model

```
                    ┌─────────────────┐
                    │   FinOps Team   │
                    │  (Centralized)  │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
        ┌──────────┐  ┌──────────┐  ┌──────────┐
        │  Team A  │  │  Team B  │  │  Team C  │
        │  Budget  │  │  Budget  │  │  Budget  │
        └──────────┘  └──────────┘  └──────────┘
```


---

### Quick Check

**Why are Committed Use Discounts more cost-effective than Sustained Use Discounts for predictable workloads?**

   A. Committed Use Discounts are automatically applied
   B. Sustained Use Discounts require minimum 1-year commitment
-> C. **Committed Use Discounts offer up to 57% off (3-year) vs 30% for Sustained Use**
   D. Sustained Use Discounts only apply to Compute Engine

<details>
<summary>See Answer</summary>

Sustained Use Discounts (SUD) are automatic discounts up to 30% for running instances for a significant portion of the month. Committed Use Discounts (CUD) require an upfront commitment but offer much larger discounts: 37% for 1-year and 57% for 3-year commitments. For predictable, stable workloads, CUDs provide significantly better savings, though they require capacity planning and commitment.

</details>

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*
