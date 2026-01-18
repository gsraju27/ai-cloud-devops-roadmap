# Cloud Engineer Roadmap 2026

A step-by-step guide to becoming a Cloud Engineer/Architect in 6 months.

```
                                    CLOUD ENGINEER ROADMAP
    ┌─────────────────────────────────────────────────────────────────────────────────┐
    │                                                                                 │
    │   Month 1          Month 2          Month 3          Month 4          Month 5-6│
    │   ────────         ────────         ────────         ────────         ─────────│
    │                                                                                 │
    │   ┌───────┐        ┌───────┐        ┌───────┐        ┌───────┐        ┌───────┐│
    │   │Linux &│───────>│  AWS  │───────>│  AWS  │───────>│  IaC  │───────>│  Arch ││
    │   │Network│        │ Core  │        │Advance│        │Terraform│      │ Design││
    │   └───────┘        └───────┘        └───────┘        └───────┘        └───────┘│
    │       │                │                │                │                │    │
    │       v                v                v                v                v    │
    │   ┌───────┐        ┌───────┐        ┌───────┐        ┌───────┐        ┌───────┐│
    │   │TCP/IP │        │EC2,S3 │        │Lambda │        │ State  │        │Multi- ││
    │   │  DNS  │        │VPC,IAM│        │ECS,RDS│        │Modules │        │ Cloud ││
    │   └───────┘        └───────┘        └───────┘        └───────┘        └───────┘│
    │                                                                                 │
    └─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Month 1: Foundations

### Week 1-2: Linux & Networking

**Learn:**
- Linux command line essentials
- File permissions and users
- Process and service management
- TCP/IP networking basics
- DNS, HTTP/HTTPS, SSL/TLS
- Firewalls and security groups

**Practice:**
```bash
# Networking commands
ping, traceroute, netstat, ss
curl, wget, dig, nslookup
iptables basics
tcpdump for debugging
```

### Week 3-4: Cloud Fundamentals

**Learn:**
- Cloud computing models (IaaS, PaaS, SaaS)
- Shared responsibility model
- Regions and availability zones
- Cloud pricing models
- Basic security concepts

---

## Month 2: AWS Core Services

### Week 1-2: Compute & Storage

**Learn:**
- EC2 instances, AMIs, instance types
- EBS volumes and snapshots
- S3 buckets, policies, lifecycle
- Elastic Load Balancing
- Auto Scaling Groups

**Practice:**
```
Build:
- Launch EC2 with custom AMI
- Configure S3 with versioning and lifecycle
- Set up ALB with target groups
- Create Auto Scaling with launch templates
```

### Week 3-4: Networking & Security

**Learn:**
- VPC architecture
- Subnets (public/private)
- Route tables and Internet Gateway
- NAT Gateway
- Security Groups vs NACLs
- IAM users, roles, policies

**Interview Prep:**
- [AWS Interview Questions](../interview-questions/cloud/aws.md) - Start with VPC and IAM questions

---

## Month 3: AWS Advanced Services

### Week 1-2: Serverless & Containers

**Learn:**
- Lambda functions and triggers
- API Gateway
- DynamoDB
- ECS and Fargate
- ECR for container images

**Practice:**
```
Build:
- Lambda + API Gateway REST API
- DynamoDB CRUD operations
- ECS service with Fargate
- Event-driven architecture with SQS/SNS
```

### Week 3-4: Databases & Caching

**Learn:**
- RDS (MySQL, PostgreSQL)
- Multi-AZ and Read Replicas
- Aurora and Aurora Serverless
- ElastiCache (Redis, Memcached)
- Database migration strategies

**Interview Prep:**
- Complete all [AWS Interview Questions](../interview-questions/cloud/aws.md)

---

## Month 4: Infrastructure as Code

### Week 1-2: Terraform Fundamentals

**Learn:**
- HCL syntax
- Providers and resources
- Variables, outputs, locals
- State management
- Remote state with S3

**Practice:**
```hcl
# Build complete infrastructure
- VPC with public/private subnets
- EC2 instances
- RDS database
- S3 buckets
- IAM roles and policies
```

### Week 3-4: Terraform Advanced

**Learn:**
- Modules and module registry
- Workspaces for environments
- Import existing resources
- Terraform Cloud/Enterprise
- Best practices and patterns

**Interview Prep:**
- [Terraform Interview Questions](../interview-questions/devops/terraform.md)

---

## Month 5: Multi-Cloud & Specialization

### Week 1-2: Second Cloud Platform

**Choose one:**

**Azure:**
- Virtual Machines, App Service
- Azure Storage, Blob
- Azure AD, RBAC
- Virtual Networks

**GCP:**
- Compute Engine, Cloud Run
- Cloud Storage, BigQuery
- IAM, Service Accounts
- VPC, Cloud NAT

**Interview Prep:**
- [Azure Interview Questions](../interview-questions/cloud/azure.md)
- [GCP Interview Questions](../interview-questions/cloud/gcp.md)

### Week 3-4: Specialization Track

**Choose based on interest:**

| Track | Focus Areas |
|-------|-------------|
| Security | IAM, encryption, compliance, WAF |
| Data | S3, Redshift, Glue, Athena |
| DevOps | CI/CD, containers, K8s |
| Serverless | Lambda, Step Functions, EventBridge |

---

## Month 6: Architecture & Certification

### Week 1-2: Architecture Patterns

**Learn:**
- Well-Architected Framework
- High availability patterns
- Disaster recovery strategies
- Cost optimization
- Performance tuning

**Practice:**
```
Design exercises:
- 3-tier web application
- Event-driven microservices
- Multi-region active-active
- Hybrid cloud architecture
```

### Week 3-4: Certification Prep

**Recommended path:**
1. AWS Solutions Architect Associate (SAA-C03)
2. AWS Developer Associate (optional)
3. AWS SysOps Administrator (optional)
4. Terraform Associate

---

## AWS Services Cheat Sheet

### Compute
| Service | Use Case |
|---------|----------|
| EC2 | Virtual servers |
| Lambda | Serverless functions |
| ECS/Fargate | Containers |
| EKS | Managed Kubernetes |

### Storage
| Service | Use Case |
|---------|----------|
| S3 | Object storage |
| EBS | Block storage for EC2 |
| EFS | Shared file system |
| Glacier | Archive storage |

### Database
| Service | Use Case |
|---------|----------|
| RDS | Relational databases |
| DynamoDB | NoSQL key-value |
| ElastiCache | In-memory caching |
| Aurora | High-performance SQL |

### Networking
| Service | Use Case |
|---------|----------|
| VPC | Virtual network |
| Route 53 | DNS |
| CloudFront | CDN |
| API Gateway | API management |

---

## Skills Checklist

### Must Have

- [ ] Linux command line
- [ ] Networking fundamentals (TCP/IP, DNS, HTTP)
- [ ] AWS core services (EC2, S3, VPC, IAM)
- [ ] Infrastructure as Code (Terraform or CloudFormation)
- [ ] Basic scripting (Python or Bash)
- [ ] Git version control

### Good to Have

- [ ] Containers (Docker, Kubernetes basics)
- [ ] CI/CD pipelines
- [ ] Monitoring (CloudWatch, Prometheus)
- [ ] Second cloud platform
- [ ] Database administration

### Nice to Have

- [ ] Service mesh
- [ ] Advanced security (pentesting, compliance)
- [ ] FinOps and cost optimization
- [ ] Multi-cloud architecture

---

## Interview Preparation

### Questions by Service

| Topic | Questions | Link |
|-------|-----------|------|
| AWS | 12 | [aws.md](../interview-questions/cloud/aws.md) |
| Azure | 12 | [azure.md](../interview-questions/cloud/azure.md) |
| GCP | 12 | [gcp.md](../interview-questions/cloud/gcp.md) |
| Terraform | 12 | [terraform.md](../interview-questions/devops/terraform.md) |

### Common Architecture Questions

1. Design a highly available web application
2. Migrate an on-premise application to AWS
3. Design a disaster recovery strategy (RTO: 1 hour, RPO: 5 minutes)
4. Optimize a $50K/month AWS bill
5. Design a multi-region active-active architecture

---

## Salary Expectations (2026)

| Level | US (Remote) | India (Metro) |
|-------|-------------|---------------|
| Junior Cloud Engineer | $75-100K | ₹6-12 LPA |
| Cloud Engineer | $110-150K | ₹12-25 LPA |
| Senior Cloud Engineer | $150-190K | ₹25-45 LPA |
| Cloud Architect | $180-250K | ₹40-70 LPA |

---

## Certifications Roadmap

```
                    ┌─────────────────────────┐
                    │  AWS Solutions Architect │
                    │      Associate (SAA)     │
                    └───────────┬─────────────┘
                                │
            ┌───────────────────┼───────────────────┐
            │                   │                   │
            v                   v                   v
    ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
    │  Developer    │   │   SysOps      │   │  Terraform    │
    │  Associate    │   │  Associate    │   │  Associate    │
    └───────────────┘   └───────────────┘   └───────────────┘
            │                   │
            └─────────┬─────────┘
                      v
            ┌───────────────────┐
            │ Solutions Architect│
            │   Professional    │
            └───────────────────┘
```

---

## Next Steps

1. **Create an AWS Free Tier account**
2. **Follow this roadmap week by week**
3. **Build real projects** - Not just tutorials
4. **Document everything** - Start a blog or GitHub
5. **Get certified** - Proves your knowledge

---

*Want to practice on real AWS infrastructure without billing risk? [Try DeployU](https://deployu.ai?ref=github) - Deploy real cloud projects with zero surprise bills.*
