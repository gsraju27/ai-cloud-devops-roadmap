# DevOps Engineer Roadmap 2026

A step-by-step guide to becoming a DevOps Engineer in 6 months.

```
                                    DEVOPS ENGINEER ROADMAP
    ┌─────────────────────────────────────────────────────────────────────────────────┐
    │                                                                                 │
    │   Month 1          Month 2          Month 3          Month 4          Month 5-6│
    │   ────────         ────────         ────────         ────────         ─────────│
    │                                                                                 │
    │   ┌───────┐        ┌───────┐        ┌───────┐        ┌───────┐        ┌───────┐│
    │   │ Linux │───────>│Docker │───────>│  K8s  │───────>│Terraform│─────>│  CI/CD││
    │   │ & Git │        │       │        │       │        │  & IaC │       │  & SRE││
    │   └───────┘        └───────┘        └───────┘        └───────┘        └───────┘│
    │       │                │                │                │                │    │
    │       v                v                v                v                v    │
    │   ┌───────┐        ┌───────┐        ┌───────┐        ┌───────┐        ┌───────┐│
    │   │ Bash  │        │Compose│        │ Helm  │        │  AWS   │        │Monitor││
    │   │Script │        │       │        │       │        │  GCP   │        │ Logs  ││
    │   └───────┘        └───────┘        └───────┘        └───────┘        └───────┘│
    │                                                                                 │
    └─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Month 1: Linux & Version Control

### Week 1-2: Linux Fundamentals

**Learn:**
- File system navigation and permissions
- Process management (ps, top, kill, systemctl)
- Package management (apt, yum, dnf)
- User and group management
- SSH and remote access

**Practice:**
```bash
# Essential commands to master
ls, cd, pwd, mkdir, rm, cp, mv
chmod, chown, chgrp
ps aux, top, htop, kill
systemctl start/stop/status
ssh, scp, rsync
```

**Resources:**
- [Linux Interview Questions](../interview-questions/programming/python.md) (includes Linux basics)
- Linux Foundation Free Course

### Week 3-4: Git & Bash Scripting

**Learn:**
- Git workflow (clone, branch, commit, merge, rebase)
- Resolving merge conflicts
- Git hooks and workflows
- Bash scripting fundamentals
- Cron jobs and automation

**Practice:**
```bash
# Git commands to master
git clone, branch, checkout, merge, rebase
git log --oneline --graph
git stash, cherry-pick
git reset --hard/soft

# Write scripts for:
- Log rotation
- Backup automation
- Health check monitoring
```

---

## Month 2: Containers & Docker

### Week 1-2: Docker Fundamentals

**Learn:**
- Container vs VM concepts
- Dockerfile best practices
- Image layers and caching
- Docker networking (bridge, host, overlay)
- Volume management

**Practice:**
```dockerfile
# Master these Dockerfile patterns
FROM, WORKDIR, COPY, RUN, CMD, ENTRYPOINT
Multi-stage builds
.dockerignore optimization
```

**Interview Prep:**
- [Docker Interview Questions](../interview-questions/devops/docker.md) - 12 real-world scenarios

### Week 3-4: Docker Compose & Registry

**Learn:**
- Multi-container applications
- Service dependencies
- Environment management
- Private registry setup
- Image security scanning

**Practice:**
```yaml
# Build a multi-service app
- Web server (Nginx)
- Application (Node.js/Python)
- Database (PostgreSQL)
- Cache (Redis)
```

---

## Month 3: Kubernetes

### Week 1-2: Kubernetes Core Concepts

**Learn:**
- Pods, Deployments, Services
- ConfigMaps and Secrets
- Namespaces and RBAC
- Resource requests and limits

**Practice:**
```yaml
# Resources to create
- Deployment with rolling updates
- Service (ClusterIP, NodePort, LoadBalancer)
- ConfigMap for environment variables
- Secret for sensitive data
```

### Week 3-4: Advanced Kubernetes

**Learn:**
- Ingress controllers
- Persistent volumes and claims
- StatefulSets for databases
- Helm charts
- Horizontal Pod Autoscaler

**Interview Prep:**
- [Kubernetes Interview Questions](../interview-questions/devops/kubernetes.md) - 10 production scenarios

---

## Month 4: Infrastructure as Code

### Week 1-2: Terraform Fundamentals

**Learn:**
- HCL syntax and structure
- Providers and resources
- State management
- Variables and outputs
- Modules

**Practice:**
```hcl
# Build infrastructure for:
- VPC with public/private subnets
- EC2 instances with security groups
- RDS database
- S3 bucket with policies
```

### Week 3-4: Cloud Platform (AWS/GCP/Azure)

**Learn:**
- Core services (Compute, Storage, Networking)
- IAM and security
- Load balancers and auto-scaling
- Managed Kubernetes (EKS/GKE/AKS)

**Interview Prep:**
- [Terraform Interview Questions](../interview-questions/devops/terraform.md)
- [AWS Interview Questions](../interview-questions/cloud/aws.md)

---

## Month 5-6: CI/CD & Observability

### Week 1-2: CI/CD Pipelines

**Learn:**
- GitHub Actions workflows
- Jenkins pipelines
- Build, test, deploy stages
- Blue-green and canary deployments
- GitOps with ArgoCD/Flux

**Practice:**
```yaml
# Build a complete pipeline
- Lint and test code
- Build Docker image
- Push to registry
- Deploy to Kubernetes
- Run integration tests
- Notify on Slack
```

**Interview Prep:**
- [GitHub Actions Interview Questions](../interview-questions/devops/github-actions.md)
- [Jenkins Interview Questions](../interview-questions/devops/jenkins.md)

### Week 3-4: Monitoring & SRE

**Learn:**
- Prometheus metrics
- Grafana dashboards
- Log aggregation (ELK/Loki)
- Alerting strategies
- Incident response
- SLIs, SLOs, SLAs

**Practice:**
- Set up Prometheus + Grafana stack
- Create alerts for CPU, memory, errors
- Build dashboards for applications
- Implement log aggregation

---

## Skills Checklist

### Must Have (Required for most DevOps roles)

- [ ] Linux command line proficiency
- [ ] Git workflow and branching strategies
- [ ] Docker containerization
- [ ] Kubernetes deployment and debugging
- [ ] At least one cloud platform (AWS preferred)
- [ ] Infrastructure as Code (Terraform)
- [ ] CI/CD pipeline creation
- [ ] Basic monitoring and logging

### Good to Have (Senior roles)

- [ ] Service mesh (Istio/Linkerd)
- [ ] Security scanning and compliance
- [ ] Cost optimization
- [ ] Disaster recovery planning
- [ ] Multiple cloud platforms
- [ ] Advanced networking

### Nice to Have (Staff/Principal roles)

- [ ] Platform engineering
- [ ] Developer experience optimization
- [ ] FinOps practices
- [ ] Chaos engineering
- [ ] Custom tooling development

---

## Interview Preparation

### Technical Questions by Topic

| Topic | Questions | Link |
|-------|-----------|------|
| Docker | 12 | [docker.md](../interview-questions/devops/docker.md) |
| Kubernetes | 10 | [kubernetes.md](../interview-questions/devops/kubernetes.md) |
| Terraform | 12 | [terraform.md](../interview-questions/devops/terraform.md) |
| AWS | 12 | [aws.md](../interview-questions/cloud/aws.md) |
| GitHub Actions | 12 | [github-actions.md](../interview-questions/devops/github-actions.md) |
| Jenkins | 12 | [jenkins.md](../interview-questions/devops/jenkins.md) |

### System Design Topics

1. **Design a CI/CD pipeline** for a microservices application
2. **Design a monitoring system** for 100+ services
3. **Design a deployment strategy** for zero-downtime releases
4. **Design a disaster recovery plan** for a multi-region application
5. **Design an infrastructure** for a startup scaling to 1M users

---

## Salary Expectations (2026)

| Level | US (Remote) | India (Metro) |
|-------|-------------|---------------|
| Junior DevOps | $80-110K | ₹8-15 LPA |
| Mid DevOps | $120-160K | ₹15-30 LPA |
| Senior DevOps | $160-200K | ₹30-50 LPA |
| Staff/Principal | $200-300K | ₹50-80 LPA |

---

## Next Steps

1. **Start with fundamentals** - Don't skip Linux and Git
2. **Build projects** - Theory without practice is useless
3. **Get certified** - AWS SAA, CKA, Terraform Associate
4. **Contribute to open source** - Real-world experience
5. **Practice interviews** - Use the questions in this repo

---

*Ready to practice on real infrastructure? [Try DeployU](https://deployu.ai?ref=github) - Deploy Kubernetes clusters and cloud applications without the billing risk.*
