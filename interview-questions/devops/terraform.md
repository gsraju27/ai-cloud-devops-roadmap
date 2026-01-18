# Complete Terraform & Infrastructure as Code Interview Guide

Master Terraform with 12 real-world interview questions covering state management, module design, multi-cloud architecture, debugging, and CI/CD pipelines. Practice scenarios that mirror actual senior DevOps/Platform engineer challenges.

**Companies that ask these questions:** HashiCorp | Google | Amazon | Microsoft | Netflix | Uber

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | Terraform apply is stuck because of a state lock. Another en... | Debugging | State Management |
| 2 | Your team copies the same VPC code across 20 repositories. D... | Architecture | Module Design |
| 3 | Resources were changed manually in the AWS console. Terrafor... | Debugging | Drift Detection |
| 4 | Design a Terraform architecture for managing 50+ AWS account... | Architecture | Multi-Account Strategy |
| 5 | Your company has 200 manually-created EC2 instances. Import ... | Practical | Resource Import |
| 6 | Database passwords are visible in terraform.tfstate. Impleme... | Practical | Security |
| 7 | Terraform fails with 'Error: Cycle' involving security group... | Debugging | Dependencies |
| 8 | Your team argues about workspaces vs directories for dev/sta... | Architecture | Environment Strategy |
| 9 | Implement a secure Terraform CI/CD pipeline with plan review... | Practical | CI/CD Integration |
| 10 | Terraform plan takes 15 minutes and times out. Your state fi... | Practical | Performance |
| 11 | After upgrading providers, terraform plan shows resources wi... | Debugging | Provider Management |
| 12 | Your Terraform modules have no tests. Implement a comprehens... | Practical | Testing |

---

## What You'll Learn

- Resolve state locking conflicts and corrupted state files
- Design reusable, composable Terraform modules
- Implement multi-account, multi-region AWS architectures
- Debug terraform plan/apply failures and dependency issues
- Import existing cloud resources into Terraform management
- Manage secrets securely without exposing them in state
- Implement workspace strategies for environment separation
- Build secure CI/CD pipelines with proper state management
- Optimize performance for large-scale infrastructure
- Test Terraform code with automated validation

---

## Interview Questions

### Question 1: Terraform apply is stuck because of a state lock. Another engineer's apply crashed mid-execution. Fix it.

**Type:** Debugging | **Category:** State Management

## The Scenario

Your teammate started a `terraform apply` but their laptop crashed mid-execution. Now everyone on the team sees this error:

```bash
$ terraform plan
Acquiring state lock. This may take a few moments...

Error: Error acquiring the state lock

Error message: ConditionalCheckFailedException: The conditional request failed
Lock Info:
  ID:        a1b2c3d4-5678-90ab-cdef-1234567890ab
  Path:      s3://company-terraform-state/prod/vpc/terraform.tfstate
  Operation: OperationTypeApply
  Who:       john@company-laptop
  Version:   1.5.0
  Created:   2024-01-15 10:30:45.123456 +0000 UTC
  Info:

Terraform acquires a state lock to protect against two processes
writing to state simultaneously. This error indicates that the
lock has not been released.
```

The original engineer is unavailable and production changes are blocked.

## The Challenge

Safely release the state lock, verify state integrity, and prevent this from happening again. Explain the risks and safeguards.


### Step 1: Verify the Lock is Actually Orphaned

Before force-unlocking, confirm no apply is running:

```bash
# Check if the lock holder's process might still be running
# Contact the engineer if possible - their apply might be in progress!

# Check CloudWatch or your monitoring for recent Terraform activity
aws cloudwatch get-metric-statistics \
  --namespace AWS/DynamoDB \
  --metric-name ConsumedWriteCapacityUnits \
  --dimensions Name=TableName,Value=terraform-locks \
  --start-time 2024-01-15T10:00:00Z \
  --end-time 2024-01-15T11:00:00Z \
  --period 300 \
  --statistics Sum

# Check DynamoDB directly for lock details
aws dynamodb get-item \
  --table-name terraform-locks \
  --key '{"LockID": {"S": "s3://company-terraform-state/prod/vpc/terraform.tfstate-md5"}}'
```

### Step 2: Force Unlock the State

```bash
# Use the Lock ID from the error message
terraform force-unlock a1b2c3d4-5678-90ab-cdef-1234567890ab

# You'll see a confirmation prompt:
Do you really want to force-unlock?
  Terraform will remove the lock on the remote state.
  This will allow local Terraform commands to modify this state, even though it
  may still be in use. Only 'yes' will be accepted to confirm.

  Enter a value: yes

# Output on success:
Terraform state has been successfully unlocked!
```

### Step 3: Verify State Integrity

**Critical:** After force-unlock, immediately check for issues:

```bash
# Run plan to see current state vs reality
terraform plan

# If plan shows unexpected changes, the previous apply may have partially completed
# Look for resources that were being created/modified

# Check state list to see all managed resources
terraform state list

# For specific resources that might be affected:
terraform state show aws_vpc.main
```

### Step 4: Handle Partial Apply Scenarios

If the previous apply partially completed:

```bash
# Option 1: If resources were partially created, import them
terraform import aws_instance.web i-1234567890abcdef0

# Option 2: If resources are in bad state, taint and recreate
terraform taint aws_instance.web
terraform apply

# Option 3: Refresh state to match reality
terraform refresh  # Deprecated in 1.5+, use:
terraform apply -refresh-only
```

### Step 5: Implement Safeguards

**Backend configuration with proper timeouts:**

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "prod/vpc/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"

    # Reduce lock timeout for faster failure detection
    # Default is 0 (wait forever)
  }
}
```

**Use Terraform Cloud/Enterprise for better lock management:**

```hcl
terraform {
  cloud {
    organization = "company"
    workspaces {
      name = "prod-vpc"
    }
  }
}
```

**CI/CD pipeline with automatic lock release:**

```yaml
# .github/workflows/terraform.yml
jobs:
  terraform:
    runs-on: ubuntu-latest
    timeout-minutes: 30  # Pipeline timeout prevents infinite locks
    steps:
      - uses: actions/checkout@v4

      - name: Terraform Apply
        run: terraform apply -auto-approve
        timeout-minutes: 20  # Step-level timeout

      # Cleanup step runs even if apply fails
      - name: Release Lock on Failure
        if: failure()
        run: |
          # Log for audit trail
          echo "Apply failed, checking for orphaned lock..."
          # Don't auto-unlock - notify team instead
          # terraform force-unlock would be dangerous here
```


## State Locking Deep Dive

| Backend | Lock Mechanism | Lock Location |
|---------|---------------|---------------|
| S3 | DynamoDB | Separate DynamoDB table |
| GCS | Native | Built into GCS |
| Azure Blob | Blob Lease | Built into Azure |
| Terraform Cloud | Native | Managed by TFC |
| Local | Filesystem | `.terraform.tfstate.lock.info` |

## DynamoDB Lock Table Schema

```hcl
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }

  tags = {
    Purpose = "Terraform state locking"
  }
}
```

## Emergency Procedures Runbook

```bash
#!/bin/bash
# emergency-unlock.sh - Use with extreme caution!

set -e

LOCK_ID=$1
STATE_PATH=$2

if [ -z "$LOCK_ID" ] || [ -z "$STATE_PATH" ]; then
  echo "Usage: ./emergency-unlock.sh <lock-id> <state-path>"
  exit 1
fi

echo "WARNING: About to force-unlock Terraform state"
echo "Lock ID: $LOCK_ID"
echo "State Path: $STATE_PATH"
echo ""
echo "Checklist before proceeding:"
echo "[ ] Confirmed lock holder is unavailable"
echo "[ ] Checked no apply is currently running"
echo "[ ] Have backup of current state"
echo ""
read -p "Type 'UNLOCK' to proceed: " confirm

if [ "$confirm" != "UNLOCK" ]; then
  echo "Aborted"
  exit 1
fi

# Create state backup first
terraform state pull > "state-backup-$(date +%Y%m%d-%H%M%S).json"

# Force unlock
terraform force-unlock -force "$LOCK_ID"

# Immediate verification
echo "Running terraform plan to verify state..."
terraform plan -detailed-exitcode || true

echo "Done. Review the plan output carefully!"
```

---

### Question 2: Your team copies the same VPC code across 20 repositories. Design a reusable module architecture.

**Type:** Architecture | **Category:** Module Design

## The Scenario

Your platform team maintains 20+ microservices, each with their own Terraform repository. Every repo has nearly identical VPC, security group, and networking code that was copy-pasted from a "template" repo:

```
repo-service-a/
  └── terraform/
      ├── vpc.tf          # 200 lines, copy-pasted
      ├── security.tf     # 150 lines, copy-pasted
      └── main.tf

repo-service-b/
  └── terraform/
      ├── vpc.tf          # Same 200 lines
      ├── security.tf     # Same 150 lines, but with a bug fix that wasn't propagated
      └── main.tf

# ... 18 more repos with drift between copies
```

A security vulnerability was found in the security group configuration. You need to patch 20 repos manually.

## The Challenge

Design a reusable module architecture that eliminates duplication, enables centralized updates, and allows per-service customization. Consider versioning, testing, and adoption strategy.


### Architecture Overview

```
terraform-modules/                    # Central modules repository
├── modules/
│   ├── vpc/                         # Single-purpose: VPC + subnets
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── versions.tf
│   │   └── README.md
│   ├── security-group/              # Single-purpose: SG rules
│   ├── alb/                         # Single-purpose: Load balancer
│   └── ecs-service/                 # Composed module using others
├── examples/
│   ├── simple-vpc/
│   └── complete-vpc/
├── test/
│   └── vpc_test.go
└── .github/
    └── workflows/
        ├── test.yml
        └── release.yml
```

### Module Design Principles

**1. Single Responsibility:**

```hcl
# modules/vpc/main.tf
# This module ONLY creates VPC infrastructure
# It does NOT create security groups, instances, etc.

resource "aws_vpc" "this" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = var.enable_dns_hostnames
  enable_dns_support   = var.enable_dns_support

  tags = merge(var.tags, {
    Name = var.name
  })
}

resource "aws_subnet" "public" {
  count = length(var.public_subnets)

  vpc_id                  = aws_vpc.this.id
  cidr_block              = var.public_subnets[count.index]
  availability_zone       = var.azs[count.index]
  map_public_ip_on_launch = true

  tags = merge(var.tags, {
    Name = "${var.name}-public-${var.azs[count.index]}"
    Tier = "public"
  })
}

resource "aws_subnet" "private" {
  count = length(var.private_subnets)

  vpc_id            = aws_vpc.this.id
  cidr_block        = var.private_subnets[count.index]
  availability_zone = var.azs[count.index]

  tags = merge(var.tags, {
    Name = "${var.name}-private-${var.azs[count.index]}"
    Tier = "private"
  })
}
```

**2. Sensible Defaults with Override Capability:**

```hcl
# modules/vpc/variables.tf
variable "name" {
  description = "Name prefix for all resources"
  type        = string
}

variable "cidr_block" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

variable "azs" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

variable "public_subnets" {
  description = "Public subnet CIDR blocks"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
}

variable "private_subnets" {
  description = "Private subnet CIDR blocks"
  type        = list(string)
  default     = ["10.0.11.0/24", "10.0.12.0/24", "10.0.13.0/24"]
}

variable "enable_nat_gateway" {
  description = "Create NAT gateways for private subnets"
  type        = bool
  default     = true
}

variable "single_nat_gateway" {
  description = "Use single NAT gateway (cost savings for non-prod)"
  type        = bool
  default     = false
}

variable "tags" {
  description = "Tags to apply to all resources"
  type        = map(string)
  default     = {}
}
```

**3. Rich Outputs for Composition:**

```hcl
# modules/vpc/outputs.tf
output "vpc_id" {
  description = "The ID of the VPC"
  value       = aws_vpc.this.id
}

output "vpc_cidr_block" {
  description = "The CIDR block of the VPC"
  value       = aws_vpc.this.cidr_block
}

output "public_subnet_ids" {
  description = "List of public subnet IDs"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "List of private subnet IDs"
  value       = aws_subnet.private[*].id
}

output "nat_gateway_ips" {
  description = "List of NAT Gateway public IPs"
  value       = aws_nat_gateway.this[*].public_ip
}

# Useful for downstream modules
output "azs" {
  description = "Availability zones used"
  value       = var.azs
}
```

### Versioning Strategy

```hcl
# In consumer's main.tf
module "vpc" {
  source  = "git::https://github.com/company/terraform-modules.git//modules/vpc?ref=v2.1.0"

  # Or using private registry:
  # source  = "app.terraform.io/company/vpc/aws"
  # version = "~> 2.1"

  name            = "service-a"
  cidr_block      = "10.1.0.0/16"
  enable_nat_gateway = true

  tags = {
    Environment = "production"
    Service     = "service-a"
  }
}
```

**Semantic Versioning Rules:**

| Change Type | Version Bump | Example |
|-------------|--------------|---------|
| Bug fix (backward compatible) | PATCH | 2.1.0 → 2.1.1 |
| New feature (backward compatible) | MINOR | 2.1.1 → 2.2.0 |
| Breaking change | MAJOR | 2.2.0 → 3.0.0 |

### Composed Modules for Common Patterns

```hcl
# modules/ecs-service/main.tf
# This module composes smaller modules for a complete ECS service

module "vpc" {
  source = "../vpc"

  name               = var.service_name
  enable_nat_gateway = true
  single_nat_gateway = var.environment != "production"

  tags = var.tags
}

module "alb" {
  source = "../alb"

  name              = var.service_name
  vpc_id            = module.vpc.vpc_id
  subnet_ids        = module.vpc.public_subnet_ids
  certificate_arn   = var.certificate_arn

  tags = var.tags
}

module "ecs_cluster" {
  source = "../ecs-cluster"

  name       = var.service_name
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnet_ids

  tags = var.tags
}
```

### Private Module Registry

```hcl
# Using Terraform Cloud Private Registry
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

module "vpc" {
  source  = "app.terraform.io/acme-corp/vpc/aws"
  version = "2.1.0"

  # ...
}
```

### Automated Release Pipeline

```yaml
# .github/workflows/release.yml
name: Release Module

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Validate version format
        run: |
          TAG=${GITHUB_REF#refs/tags/}
          if [[ ! $TAG =~ ^v[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
            echo "Invalid version format. Use semantic versioning (v1.2.3)"
            exit 1
          fi

      - name: Run tests
        run: |
          cd test
          go test -v -timeout 30m

      - name: Create GitHub Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          body: |
            See CHANGELOG.md for details
          draft: false
          prerelease: false
```


## Migration Strategy for Existing Repos

```bash
# Step 1: Create module with current behavior
# Step 2: Have one service adopt it
# Step 3: Iterate based on feedback
# Step 4: Roll out to remaining services

# In each service repo:
# Before:
# └── terraform/
#     ├── vpc.tf (200 lines)
#     └── main.tf

# After:
# └── terraform/
#     └── main.tf (20 lines using module)
```

---

### Question 3: Resources were changed manually in the AWS console. Terraform plan shows unexpected changes. Handle it.

**Type:** Debugging | **Category:** Drift Detection

## The Scenario

During an incident, an engineer manually modified security group rules in the AWS console to temporarily allow traffic. Now `terraform plan` shows:

```bash
$ terraform plan

aws_security_group.api: Refreshing state... [id=sg-0abc123def456789]

Terraform will perform the following actions:

  # aws_security_group_rule.api_ingress will be destroyed
  - resource "aws_security_group_rule" "api_ingress" {
      - cidr_blocks       = ["10.0.0.0/8"]
      - from_port         = 443
      - protocol          = "tcp"
      - security_group_id = "sg-0abc123def456789"
      - to_port           = 443
      - type              = "ingress"
    }

Plan: 0 to add, 0 to change, 1 to destroy.
```

The manually-added rule is critical for the temporary fix. Running `terraform apply` would break production.

## The Challenge

Understand why drift happens, how to detect it, and the options for reconciliation. Decide whether to adopt the manual change into Terraform or revert it.


### Step 1: Investigate the Drift

```bash
# See what Terraform thinks the current state is
terraform show

# See what's actually in AWS
aws ec2 describe-security-groups --group-ids sg-0abc123def456789

# Compare the two to understand the drift
# Look for:
# - Rules in AWS but not in Terraform (manually added)
# - Rules in Terraform but not in AWS (manually deleted)
# - Rules with different values (manually modified)
```

### Step 2: Understand the Context

Before changing anything:
- Check incident channel/tickets for why the change was made
- Verify if the temporary fix is still needed
- Confirm with the team whether to keep or revert

### Option A: Adopt Manual Change into Terraform

If the manual change should become permanent:

```hcl
# Add the new rule to your Terraform configuration
resource "aws_security_group_rule" "api_temp_ingress" {
  type              = "ingress"
  from_port         = 443
  to_port           = 443
  protocol          = "tcp"
  cidr_blocks       = ["10.0.0.0/8"]
  security_group_id = aws_security_group.api.id
  description       = "Temporary fix from incident INC-1234 - review for removal"
}
```

Then import the existing resource:

```bash
# Import the manually-created rule
terraform import aws_security_group_rule.api_temp_ingress sg-0abc123def456789_ingress_tcp_443_443_10.0.0.0/8

# Verify import worked
terraform plan
# Should show: No changes. Infrastructure is up-to-date.
```

### Option B: Revert Manual Change

If the manual change should be removed:

```bash
# Simply run apply - Terraform will remove the drift
terraform apply

# The manual rule will be deleted
```

### Option C: Refresh State Only

If you want to update state to match reality without changing anything:

```bash
# Terraform 1.5+
terraform apply -refresh-only

# This updates state to match AWS reality
# Future plans will show changes needed to match your config
```

### Implementing Drift Detection

**Scheduled drift detection in CI/CD:**

```yaml
# .github/workflows/drift-detection.yml
name: Terraform Drift Detection

on:
  schedule:
    - cron: '0 */4 * * *'  # Every 4 hours
  workflow_dispatch:

jobs:
  detect-drift:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Terraform Init
        run: terraform init

      - name: Detect Drift
        id: plan
        run: |
          terraform plan -detailed-exitcode -out=plan.tfplan 2>&1 | tee plan.log
          echo "exitcode=$?" >> $GITHUB_OUTPUT
        continue-on-error: true

      - name: Report Drift
        if: steps.plan.outputs.exitcode == '2'
        run: |
          echo "DRIFT DETECTED!"
          cat plan.log
          # Send alert to Slack/PagerDuty
          curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
            -H 'Content-Type: application/json' \
            -d '{"text": "Terraform drift detected in production! Check GitHub Actions for details."}'
```

**Using Terraform Cloud:**

```hcl
# Terraform Cloud has built-in drift detection
terraform {
  cloud {
    organization = "company"
    workspaces {
      name = "prod-infrastructure"
    }
  }
}

# In TFC settings, enable:
# - Health Assessments (drift detection)
# - Continuous Validation
```

### Preventing Drift

**1. AWS Service Control Policies (SCPs):**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyManualSecurityGroupChanges",
      "Effect": "Deny",
      "Action": [
        "ec2:AuthorizeSecurityGroupIngress",
        "ec2:AuthorizeSecurityGroupEgress",
        "ec2:RevokeSecurityGroupIngress",
        "ec2:RevokeSecurityGroupEgress"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:PrincipalArn": [
            "arn:aws:iam::*:role/TerraformExecutionRole"
          ]
        }
      }
    }
  ]
}
```

**2. AWS Config Rules:**

```hcl
resource "aws_config_config_rule" "security_group_drift" {
  name = "security-group-drift-detection"

  source {
    owner             = "AWS"
    source_identifier = "EC2_SECURITY_GROUP_ATTACHED_TO_ENI_PERIODIC"
  }

  scope {
    compliance_resource_types = ["AWS::EC2::SecurityGroup"]
  }
}
```

**3. Lifecycle ignore_changes (use sparingly):**

```hcl
# For resources that are expected to change outside Terraform
resource "aws_autoscaling_group" "web" {
  # ...

  lifecycle {
    ignore_changes = [
      desired_capacity,  # Changed by autoscaling
      target_group_arns, # Changed by deployments
    ]
  }
}
```


## Drift Reconciliation Decision Tree

```
Manual change detected
        │
        ▼
Is the change intentional?
        │
    ┌───┴───┐
    │       │
   Yes      No
    │       │
    ▼       ▼
Should it   terraform apply
be permanent? (reverts change)
    │
┌───┴───┐
│       │
Yes     No (temporary)
│       │
▼       ▼
Add to   Document & schedule
Terraform removal
+ import
```

## State Manipulation Commands

| Command | Use Case |
|---------|----------|
| `terraform import` | Add existing resource to state |
| `terraform state rm` | Remove resource from state (keeps in AWS) |
| `terraform state mv` | Rename/move resource in state |
| `terraform apply -refresh-only` | Update state without changing infra |
| `terraform taint` | Mark resource for recreation |

---

### Question 4: Design a Terraform architecture for managing 50+ AWS accounts with proper isolation and DRY principles.

**Type:** Architecture | **Category:** Multi-Account Strategy

## The Scenario

Your company is scaling from 5 AWS accounts to 50+ using AWS Organizations. Current setup:

```
terraform/
├── account-dev/
│   ├── main.tf      # 500 lines, mostly copy-pasted
│   ├── vpc.tf
│   └── iam.tf
├── account-staging/
│   ├── main.tf      # Same 500 lines with different values
│   ├── vpc.tf
│   └── iam.tf
└── account-prod/
    ├── main.tf      # Same again
    ├── vpc.tf
    └── iam.tf
```

Problems:
- Each account is managed separately with duplicated code
- No centralized governance or compliance
- Engineers must manually assume roles in each account
- State files scattered across S3 buckets

## The Challenge

Design a scalable multi-account architecture that centralizes management, enforces compliance, and allows teams to self-serve while maintaining security boundaries.


> **Common Mistake:** A junior engineer might create a single Terraform workspace with all 50 accounts, use a single state file for everything, or hardcode account IDs throughout. This leads to massive blast radius, slow plans, and security risks where one mistake affects all accounts.

> **Senior Engineer Approach:** A senior engineer designs a hub-and-spoke model with account vending automation, separate state per account/component, centralized modules with account-specific configurations, and proper IAM role assumption chains.

### Architecture Overview

```
terraform-infrastructure/
├── modules/                      # Shared, versioned modules
│   ├── account-baseline/        # SCPs, CloudTrail, Config, GuardDuty
│   ├── vpc/
│   ├── iam-roles/
│   └── security-baseline/
├── live/
│   ├── organization/            # AWS Organizations, SCPs
│   │   ├── main.tf
│   │   └── accounts.tf
│   ├── shared-services/         # Central logging, networking
│   │   └── main.tf
│   └── workloads/              # Account-specific configs
│       ├── dev/
│       │   ├── account-1/
│       │   └── account-2/
│       ├── staging/
│       └── prod/
├── terragrunt.hcl              # Root config
└── account-vending/            # New account automation
```

### Provider Configuration for Multi-Account

```hcl
# providers.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Default provider (management account)
provider "aws" {
  region = "us-east-1"
}

# Assume role into target account
provider "aws" {
  alias  = "target"
  region = var.aws_region

  assume_role {
    role_arn     = "arn:aws:iam::${var.target_account_id}:role/TerraformExecutionRole"
    session_name = "terraform-${var.environment}"
    external_id  = var.external_id  # Optional additional security
  }
}
```

### Account Vending Machine

```hcl
# account-vending/main.tf
# Automated new account creation with baseline configuration

resource "aws_organizations_account" "workload" {
  for_each = var.accounts

  name      = each.value.name
  email     = each.value.email
  parent_id = each.value.ou_id

  role_name = "OrganizationAccountAccessRole"

  lifecycle {
    ignore_changes = [role_name]
  }

  tags = {
    Environment = each.value.environment
    Team        = each.value.team
    CostCenter  = each.value.cost_center
  }
}

# Apply baseline to each new account
module "account_baseline" {
  source   = "../modules/account-baseline"
  for_each = aws_organizations_account.workload

  providers = {
    aws = aws.target
  }

  account_id   = each.value.id
  account_name = each.value.name
  environment  = var.accounts[each.key].environment

  # Baseline components
  enable_cloudtrail    = true
  enable_config        = true
  enable_guardduty     = true
  enable_security_hub  = true

  # Central logging account
  logging_account_id   = var.logging_account_id
  logging_bucket_name  = var.central_logging_bucket
}
```

### Terragrunt for DRY Configuration

```hcl
# terragrunt.hcl (root)
remote_state {
  backend = "s3"
  generate = {
    path      = "backend.tf"
    if_exists = "overwrite"
  }
  config = {
    bucket         = "company-terraform-state-${get_aws_account_id()}"
    key            = "${path_relative_to_include()}/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

generate "provider" {
  path      = "provider.tf"
  if_exists = "overwrite"
  contents  = <<EOF
provider "aws" {
  region = "${local.aws_region}"

  assume_role {
    role_arn = "arn:aws:iam::${local.account_id}:role/TerraformExecutionRole"
  }

  default_tags {
    tags = {
      ManagedBy   = "Terraform"
      Environment = "${local.environment}"
      Repository  = "terraform-infrastructure"
    }
  }
}
EOF
}
```

```hcl
# live/workloads/prod/account-1/terragrunt.hcl
include "root" {
  path = find_in_parent_folders()
}

locals {
  account_id  = "111111111111"
  environment = "prod"
  aws_region  = "us-east-1"
}

terraform {
  source = "../../../../modules//vpc"
}

inputs = {
  name               = "prod-vpc"
  cidr_block         = "10.1.0.0/16"
  enable_nat_gateway = true
  single_nat_gateway = false  # HA for prod
}
```

### Cross-Account IAM Role Chain

```hcl
# modules/iam-roles/main.tf
# Create execution role in each account

resource "aws_iam_role" "terraform_execution" {
  name = "TerraformExecutionRole"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          AWS = [
            "arn:aws:iam::${var.management_account_id}:role/TerraformCIRole",
            "arn:aws:iam::${var.management_account_id}:root"
          ]
        }
        Action = "sts:AssumeRole"
        Condition = {
          StringEquals = {
            "sts:ExternalId" = var.external_id
          }
        }
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "terraform_admin" {
  role       = aws_iam_role.terraform_execution.name
  policy_arn = "arn:aws:iam::aws:policy/AdministratorAccess"
}

# More restrictive policy for non-prod
resource "aws_iam_role_policy" "terraform_restrictions" {
  count = var.environment != "prod" ? 1 : 0
  name  = "TerraformRestrictions"
  role  = aws_iam_role.terraform_execution.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Deny"
        Action = [
          "organizations:*",
          "account:*"
        ]
        Resource = "*"
      }
    ]
  })
}
```

### Centralized State Management

```hcl
# One state bucket per account, centralized lock table
# State structure:
# s3://company-terraform-state-{account_id}/
#   └── {component}/terraform.tfstate

# Alternative: Single bucket with account prefix
# s3://company-terraform-state/
#   └── accounts/{account_id}/{component}/terraform.tfstate

resource "aws_s3_bucket" "terraform_state" {
  bucket = "company-terraform-state-${data.aws_caller_identity.current.account_id}"
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = aws_kms_key.terraform_state.arn
    }
  }
}
```

### CI/CD Pipeline for Multi-Account

```yaml
# .github/workflows/terraform.yml
name: Terraform Multi-Account

on:
  pull_request:
    paths:
      - 'live/**'
  push:
    branches: [main]
    paths:
      - 'live/**'

jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      accounts: ${{ steps.changes.outputs.accounts }}
    steps:
      - uses: actions/checkout@v4
      - id: changes
        run: |
          # Detect which account directories changed
          ACCOUNTS=$(git diff --name-only origin/main | grep "^live/workloads" | cut -d'/' -f3-4 | sort -u | jq -R -s -c 'split("\n")[:-1]')
          echo "accounts=$ACCOUNTS" >> $GITHUB_OUTPUT

  plan:
    needs: detect-changes
    runs-on: ubuntu-latest
    strategy:
      matrix:
        account: ${{ fromJson(needs.detect-changes.outputs.accounts) }}
      fail-fast: false
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::${{ secrets.MGMT_ACCOUNT_ID }}:role/GitHubActionsRole
          aws-region: us-east-1

      - name: Terragrunt Plan
        run: |
          cd live/workloads/${{ matrix.account }}
          terragrunt plan -out=plan.tfplan

  apply:
    if: github.ref == 'refs/heads/main'
    needs: [detect-changes, plan]
    runs-on: ubuntu-latest
    strategy:
      matrix:
        account: ${{ fromJson(needs.detect-changes.outputs.accounts) }}
      max-parallel: 5  # Limit concurrent applies
    environment: ${{ matrix.account }}  # Requires approval
    steps:
      - uses: actions/checkout@v4
      - name: Terragrunt Apply
        run: |
          cd live/workloads/${{ matrix.account }}
          terragrunt apply -auto-approve
```


## Account Organization Structure

```
Root
├── Security OU
│   ├── Log Archive (centralized logging)
│   └── Security Tooling (GuardDuty, Security Hub)
├── Infrastructure OU
│   ├── Shared Services (Transit Gateway, DNS)
│   └── Network (centralized networking)
├── Workloads OU
│   ├── Production OU
│   │   ├── prod-account-1
│   │   └── prod-account-2
│   ├── Staging OU
│   └── Development OU
└── Sandbox OU (isolated experimentation)
```


---

### Quick Check

**Why should each AWS account have its own Terraform state file instead of one global state?**

   A. AWS requires separate state files per account
-> B. **It reduces blast radius, improves performance, and allows parallel operations across accounts**
   C. Terraform cannot assume roles across accounts with shared state
   D. State files have a maximum size limit

<details>
<summary>See Answer</summary>

Separate state files per account (or per component) reduce blast radius - a corrupted state or failed apply only affects one account. It also improves performance since plans only refresh resources in that state. Additionally, teams can work on different accounts in parallel without state locking conflicts. This follows the principle of least privilege and isolation.

</details>

---

### Question 5: Your company has 200 manually-created EC2 instances. Import them into Terraform without downtime.

**Type:** Practical | **Category:** Resource Import

## The Scenario

Your company has 200 EC2 instances that were created manually over 3 years. Management wants everything in Terraform for consistency. Current state:

```bash
# Random naming, no tags, various configurations
$ aws ec2 describe-instances --query 'Reservations[].Instances[].{ID:InstanceId,Name:Tags[?Key==`Name`].Value|[0]}' --output table

| InstanceId         | Name              |
|--------------------|-------------------|
| i-0abc123def456789 | web-server-1      |
| i-0def456789abc012 | (none)            |
| i-0789abc012def345 | prod-api          |
| ... 197 more ...   |                   |
```

Constraints:
- Zero downtime - these are production servers
- Must match current configuration exactly
- Need to establish naming conventions going forward

## The Challenge

Design and execute an import strategy that brings all resources under Terraform management without any service disruption.


> **Common Mistake:** A junior engineer might try to import all 200 instances at once, write Terraform config from scratch without checking actual AWS settings, or skip verification steps. This risks configuration drift that triggers unwanted changes, missed resources, or state corruption.

> **Senior Engineer Approach:** A senior engineer uses a systematic approach: inventory resources, generate Terraform config that matches current state, import in batches, verify with targeted plans, and iterate. They use tools like terraformer or AWS-to-Terraform generators to bootstrap config, then refine manually.

### Phase 1: Inventory and Assessment

```bash
# Export all EC2 instances to JSON for analysis
aws ec2 describe-instances \
  --query 'Reservations[].Instances[]' \
  --output json > instances.json

# Categorize by purpose/environment
cat instances.json | jq -r '
  .[] |
  {
    id: .InstanceId,
    type: .InstanceType,
    vpc: .VpcId,
    subnet: .SubnetId,
    ami: .ImageId,
    name: (.Tags // [] | map(select(.Key == "Name")) | .[0].Value // "unnamed"),
    env: (.Tags // [] | map(select(.Key == "Environment")) | .[0].Value // "unknown")
  }
' > inventory.json

# Count by instance type
cat instances.json | jq -r '.[].InstanceType' | sort | uniq -c | sort -rn
```

### Phase 2: Generate Terraform Configuration

**Option A: Use Terraformer (recommended for large imports)**

```bash
# Install terraformer
brew install terraformer

# Generate Terraform code from existing AWS resources
terraformer import aws \
  --resources=ec2_instance,security_group,vpc,subnet \
  --regions=us-east-1 \
  --profile=production

# Output structure:
# generated/
#   └── aws/
#       └── ec2_instance/
#           ├── instance.tf
#           └── provider.tf
```

**Option B: Manual approach with import blocks (Terraform 1.5+)**

```hcl
# imports.tf - New in Terraform 1.5+

  to = aws_instance.web_server_1
  id = "i-0abc123def456789"
}


  to = aws_instance.prod_api
  id = "i-0789abc012def345"
}

# Generate config from imports
# terraform plan -generate-config-out=generated.tf
```

**Option C: Script-based config generation**

```bash
#!/bin/bash
# generate-tf-config.sh

aws ec2 describe-instances --output json | jq -r '
  .Reservations[].Instances[] |
  "resource \"aws_instance\" \"\(.Tags | map(select(.Key==\"Name\"))[0].Value // .InstanceId | gsub(\"-\"; \"_\"))\" {
    # Instance ID: \(.InstanceId)
    ami           = \"\(.ImageId)\"
    instance_type = \"\(.InstanceType)\"
    subnet_id     = \"\(.SubnetId)\"

    vpc_security_group_ids = [\(.SecurityGroups | map(\"\\\"\(.GroupId)\\\"\") | join(\", \"))]

    tags = {
      Name = \"\(.Tags | map(select(.Key==\"Name\"))[0].Value // \"unnamed\")\"
    }
  }
  "
' > generated_instances.tf
```

### Phase 3: Import in Batches

```bash
#!/bin/bash
# import-batch.sh - Import instances in manageable batches

BATCH_SIZE=10
INSTANCES=$(aws ec2 describe-instances --query 'Reservations[].Instances[].InstanceId' --output text)

count=0
for instance_id in $INSTANCES; do
  # Get instance name for resource naming
  name=$(aws ec2 describe-instances --instance-ids $instance_id \
    --query 'Reservations[].Instances[].Tags[?Key==`Name`].Value' --output text)

  # Sanitize name for Terraform resource name
  tf_name=$(echo "${name:-$instance_id}" | tr '-' '_' | tr '[:upper:]' '[:lower:]')

  echo "Importing $instance_id as aws_instance.$tf_name"
  terraform import "aws_instance.$tf_name" "$instance_id"

  ((count++))

  # Verify after each batch
  if [ $((count % BATCH_SIZE)) -eq 0 ]; then
    echo "=== Verifying batch (${count} instances imported) ==="
    terraform plan -detailed-exitcode
    if [ $? -eq 2 ]; then
      echo "WARNING: Plan shows changes. Review before continuing."
      read -p "Continue? (y/n) " -n 1 -r
      echo
      if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        exit 1
      fi
    fi
  fi
done
```

### Phase 4: Verify Configuration Matches Reality

```bash
# Run plan - should show NO changes if config matches
terraform plan

# If plan shows changes, your config doesn't match actual state
# Common mismatches:

# 1. Root volume settings
resource "aws_instance" "web" {
  # Add this block to match existing root volume
  root_block_device {
    volume_size           = 100  # Match actual size
    volume_type           = "gp3"
    delete_on_termination = true
    encrypted             = true
  }
}

# 2. Network interfaces
resource "aws_instance" "web" {
  # If instance has multiple ENIs, might need:
  network_interface {
    network_interface_id = "eni-0abc123"
    device_index         = 0
  }
}

# 3. IAM instance profile
resource "aws_instance" "web" {
  iam_instance_profile = "existing-profile-name"
}
```

### Phase 5: Handle Lifecycle Configurations

```hcl
# Prevent accidental changes to critical attributes
resource "aws_instance" "production_db" {
  # ... config ...

  lifecycle {
    # Prevent destruction without explicit approval
    prevent_destroy = true

    # Ignore changes made outside Terraform
    ignore_changes = [
      user_data,  # Often changed for bootstrapping
      tags["LastUpdated"],  # Auto-updated tags
    ]
  }
}
```

### Phase 6: Establish Naming Conventions

```hcl
# locals.tf - Standardize naming going forward
locals {
  name_prefix = "${var.environment}-${var.application}"

  common_tags = {
    Environment  = var.environment
    Application  = var.application
    ManagedBy    = "terraform"
    ImportedFrom = "manual"
    ImportDate   = "2024-01-15"
  }
}

# Use for new resources and gradually update imported ones
resource "aws_instance" "web" {
  # ... config ...

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-web-${count.index + 1}"
    Role = "web-server"
  })
}
```

### Handling Related Resources

```hcl
# Don't forget dependent resources!
# Security Groups

  to = aws_security_group.web
  id = "sg-0abc123def"
}

# EBS Volumes (if not root volumes)

  to = aws_ebs_volume.data
  id = "vol-0abc123def"
}

# Elastic IPs

  to = aws_eip.web
  id = "eipalloc-0abc123def"
}

# EIP Association

  to = aws_eip_association.web
  id = "eipassoc-0abc123def"
}
```


## Import Checklist

| Step | Action | Verification |
|------|--------|--------------|
| 1 | Inventory all resources | Count matches AWS console |
| 2 | Generate Terraform config | Syntax valid (`terraform validate`) |
| 3 | Import resources | State shows resources |
| 4 | Run `terraform plan` | **No changes** shown |
| 5 | Fix config mismatches | Re-run plan until clean |
| 6 | Test with `-target` | Individual resources stable |
| 7 | Document imported resources | README updated |

## Common Import Gotchas

```hcl
# 1. Security group rules - import the group, not individual rules
terraform import aws_security_group.web sg-0abc123
# Rules are imported with the group

# 2. Auto-scaling groups - instances are separate
terraform import aws_autoscaling_group.web my-asg
# Don't import the instances - ASG manages them

# 3. RDS with read replicas - import separately
terraform import aws_db_instance.primary my-db
terraform import aws_db_instance.replica my-db-replica

# 4. Resources with count/for_each
terraform import 'aws_instance.web[0]' i-0abc123
terraform import 'aws_instance.web["server-1"]' i-0def456
```

---

### Question 6: Database passwords are visible in terraform.tfstate. Implement secure secrets management.

**Type:** Practical | **Category:** Security

## The Scenario

A security audit revealed that database passwords are stored in plaintext in your Terraform state:

```bash
$ terraform state pull | jq '.resources[] | select(.type == "aws_db_instance") | .instances[].attributes.password'
"SuperSecretProd123!"

$ cat terraform.tfvars
db_password = "SuperSecretProd123!"
```

The state file is in S3 (encrypted at rest), but:
- Anyone with state access can read passwords
- Passwords are in version control (tfvars)
- No rotation mechanism exists
- Same password used across environments

## The Challenge

Implement a secrets management strategy that removes secrets from state, enables rotation, and follows security best practices.


### Strategy 1: AWS Secrets Manager (Recommended for AWS)

**Step 1: Create secrets outside Terraform**

```bash
# Create secret manually or via separate secure process
aws secretsmanager create-secret \
  --name "prod/database/password" \
  --secret-string "$(openssl rand -base64 32)" \
  --description "Production database password"
```

**Step 2: Reference secrets in Terraform**

```hcl
# data.tf - Read secret without storing in state
data "aws_secretsmanager_secret" "db_password" {
  name = "prod/database/password"
}

data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = data.aws_secretsmanager_secret.db_password.id
}

# main.tf - Use the secret
resource "aws_db_instance" "main" {
  identifier     = "production-db"
  engine         = "postgres"
  engine_version = "15.4"
  instance_class = "db.r6g.large"

  username = "admin"
  # Password comes from Secrets Manager - NOT stored in state
  password = data.aws_secretsmanager_secret_version.db_password.secret_string

  # Better: Use IAM authentication instead of passwords
  iam_database_authentication_enabled = true
}
```

**Step 3: Enable automatic rotation**

```hcl
resource "aws_secretsmanager_secret_rotation" "db_password" {
  secret_id           = data.aws_secretsmanager_secret.db_password.id
  rotation_lambda_arn = aws_lambda_function.rotate_secret.arn

  rotation_rules {
    automatically_after_days = 30
  }
}
```

### Strategy 2: HashiCorp Vault

```hcl
# provider.tf
provider "vault" {
  address = "https://vault.company.com"
  # Auth via VAULT_TOKEN env var or other method
}

# data.tf
data "vault_generic_secret" "db_creds" {
  path = "secret/data/production/database"
}

# main.tf
resource "aws_db_instance" "main" {
  identifier = "production-db"
  engine     = "postgres"

  username = data.vault_generic_secret.db_creds.data["username"]
  password = data.vault_generic_secret.db_creds.data["password"]
}
```

### Strategy 3: AWS SSM Parameter Store

```hcl
# For less sensitive config, SSM is simpler/cheaper
data "aws_ssm_parameter" "db_password" {
  name            = "/prod/database/password"
  with_decryption = true
}

resource "aws_db_instance" "main" {
  # ...
  password = data.aws_ssm_parameter.db_password.value
}
```

### Strategy 4: Generate Secrets in Terraform (Rotate via Replacement)

```hcl
# Generate random password - stored in state but rotatable
resource "random_password" "db" {
  length           = 32
  special          = true
  override_special = "!#$%&*()-_=+[]{}<>:?"
}

# Store in Secrets Manager
resource "aws_secretsmanager_secret" "db_password" {
  name = "prod/database/password"
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id     = aws_secretsmanager_secret.db_password.id
  secret_string = random_password.db.result
}

# To rotate: terraform taint random_password.db && terraform apply
```

### Strategy 5: IAM Authentication (Eliminate Passwords)

```hcl
# Best approach: No passwords at all!
resource "aws_db_instance" "main" {
  identifier = "production-db"
  engine     = "postgres"

  # Enable IAM auth
  iam_database_authentication_enabled = true

  # Still need master password for initial setup
  username = "admin"
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}

# Create IAM policy for database access
resource "aws_iam_policy" "rds_connect" {
  name = "rds-db-connect"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Action   = "rds-db:connect"
        Resource = "arn:aws:rds-db:${var.region}:${var.account_id}:dbuser:${aws_db_instance.main.resource_id}/app_user"
      }
    ]
  })
}
```

### Preventing Secrets in State

```hcl
# Use lifecycle to prevent sensitive data in state
resource "aws_db_instance" "main" {
  # ...

  lifecycle {
    ignore_changes = [password]
  }
}

# Mark outputs as sensitive
output "db_connection_string" {
  value     = "postgresql://${aws_db_instance.main.username}@${aws_db_instance.main.endpoint}"
  sensitive = true
}
```

### CI/CD Pipeline Security

```yaml
# .github/workflows/terraform.yml
jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      # Never echo secrets
      - name: Terraform Plan
        run: terraform plan
        env:
          # Secrets from GitHub Secrets, not tfvars
          TF_VAR_vault_token: ${{ secrets.VAULT_TOKEN }}

      # Mask any accidental secret exposure
      - name: Mask Secrets
        run: |
          echo "::add-mask::${{ secrets.DB_PASSWORD }}"
```

### State File Protection

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "prod/database/terraform.tfstate"
    region         = "us-east-1"

    # Encryption at rest
    encrypt        = true
    kms_key_id     = "arn:aws:kms:us-east-1:123456789:key/abc-123"

    # Encryption in transit (default)
    # Access logging
    # Versioning for recovery

    dynamodb_table = "terraform-locks"
  }
}

# Restrict state bucket access
resource "aws_s3_bucket_policy" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid       = "DenyUnencryptedObjectUploads"
        Effect    = "Deny"
        Principal = "*"
        Action    = "s3:PutObject"
        Resource  = "${aws_s3_bucket.terraform_state.arn}/*"
        Condition = {
          StringNotEquals = {
            "s3:x-amz-server-side-encryption" = "aws:kms"
          }
        }
      }
    ]
  })
}
```


## Secrets Management Comparison

| Method | Secrets in State | Rotation | Complexity | Cost |
|--------|-----------------|----------|------------|------|
| tfvars file | Yes | Manual | Low | Free |
| Environment vars | Yes | Manual | Low | Free |
| Secrets Manager data source | No | Auto | Medium | ~$0.40/secret/month |
| Vault | No | Auto | High | Vault license |
| SSM Parameter Store | No | Manual | Low | Free (standard) |
| IAM Auth | N/A | Automatic | Medium | Free |


---

### Quick Check

**Why does using a data source to read from Secrets Manager keep the secret out of Terraform state?**

   A. Data sources are never stored in state files
   B. Secrets Manager encrypts the response before sending to Terraform
   C. Data sources with sensitive data are automatically excluded from state
-> D. **Data source values ARE stored in state - you need additional measures like sensitive = true**

<details>
<summary>See Answer</summary>

This is a common misconception! Data source values ARE stored in Terraform state, including secrets from Secrets Manager. To truly protect secrets: 1) Mark outputs as sensitive, 2) Use sensitive = true on variables, 3) Consider using ignore_changes lifecycle, 4) Encrypt state at rest and restrict access. The benefit of Secrets Manager is centralized rotation and audit logging, not state protection.

</details>

---

### Question 7: Terraform fails with 'Error: Cycle' involving security groups and EC2 instances. Debug and fix it.

**Type:** Debugging | **Category:** Dependencies

## The Scenario

You're setting up a web application with frontend and backend servers that need to communicate. Terraform fails with:

```bash
$ terraform plan

Error: Cycle: aws_security_group.backend, aws_security_group.frontend

  on main.tf line 15:
  15: resource "aws_security_group" "frontend" {
```

Your configuration:

```hcl
resource "aws_security_group" "frontend" {
  name = "frontend-sg"

  ingress {
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    cidr_blocks     = ["0.0.0.0/0"]
  }

  # Frontend needs to call backend
  egress {
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.backend.id]  # References backend
  }
}

resource "aws_security_group" "backend" {
  name = "backend-sg"

  # Backend accepts traffic from frontend
  ingress {
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.frontend.id]  # References frontend
  }
}
```

## The Challenge

Understand why dependency cycles occur and implement the correct pattern to break them while maintaining the required security group relationships.


### Understanding the Cycle

```
frontend_sg ──references──► backend_sg
     ▲                           │
     │                           │
     └────────references─────────┘

Both need the other to exist first = impossible!
```

### Solution: Separate Security Group Rules

```hcl
# Step 1: Create security groups WITHOUT inline rules
resource "aws_security_group" "frontend" {
  name        = "frontend-sg"
  description = "Frontend web servers"
  vpc_id      = var.vpc_id

  tags = {
    Name = "frontend-sg"
  }
}

resource "aws_security_group" "backend" {
  name        = "backend-sg"
  description = "Backend API servers"
  vpc_id      = var.vpc_id

  tags = {
    Name = "backend-sg"
  }
}

# Step 2: Create rules as SEPARATE resources
resource "aws_security_group_rule" "frontend_ingress_http" {
  type              = "ingress"
  from_port         = 80
  to_port           = 80
  protocol          = "tcp"
  cidr_blocks       = ["0.0.0.0/0"]
  security_group_id = aws_security_group.frontend.id
  description       = "HTTP from internet"
}

resource "aws_security_group_rule" "frontend_egress_to_backend" {
  type                     = "egress"
  from_port                = 8080
  to_port                  = 8080
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.backend.id  # Now this works!
  security_group_id        = aws_security_group.frontend.id
  description              = "To backend API"
}

resource "aws_security_group_rule" "backend_ingress_from_frontend" {
  type                     = "ingress"
  from_port                = 8080
  to_port                  = 8080
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.frontend.id  # And this!
  security_group_id        = aws_security_group.backend.id
  description              = "From frontend servers"
}

resource "aws_security_group_rule" "backend_egress_all" {
  type              = "egress"
  from_port         = 0
  to_port           = 0
  protocol          = "-1"
  cidr_blocks       = ["0.0.0.0/0"]
  security_group_id = aws_security_group.backend.id
  description       = "Allow all outbound"
}
```

### Why This Works

```
Step 1: Create both security groups (no dependencies)
frontend_sg ──created──► (no references)
backend_sg  ──created──► (no references)

Step 2: Create rules (both SGs now exist)
frontend_egress_rule ──references──► backend_sg ✓ (exists)
backend_ingress_rule ──references──► frontend_sg ✓ (exists)
```

### Alternative: Self-Referencing Security Group

For cases where instances in the same security group need to communicate:

```hcl
resource "aws_security_group" "cluster" {
  name        = "cluster-sg"
  description = "Cluster nodes"
  vpc_id      = var.vpc_id
}

# Self-reference works with separate rules
resource "aws_security_group_rule" "cluster_internal" {
  type                     = "ingress"
  from_port                = 0
  to_port                  = 65535
  protocol                 = "tcp"
  self                     = true  # Special case for self-reference
  security_group_id        = aws_security_group.cluster.id
  description              = "Internal cluster communication"
}
```

### Debugging Cycles

```bash
# Visualize the dependency graph
terraform graph | dot -Tpng > graph.png

# Or use text output
terraform graph

# Look for bidirectional arrows between resources
# Example cycle in graph output:
# "aws_security_group.frontend" -> "aws_security_group.backend"
# "aws_security_group.backend" -> "aws_security_group.frontend"
```

### Common Cycle Patterns and Fixes

**Pattern 1: IAM Role and Policy**

```hcl
# WRONG: Cycle between role and policy
resource "aws_iam_role" "app" {
  assume_role_policy = jsonencode({
    Statement = [{
      Action = "sts:AssumeRole"
      Principal = { Service = "ec2.amazonaws.com" }
    }]
  })
}

resource "aws_iam_policy" "app" {
  policy = jsonencode({
    Statement = [{
      Action   = "s3:GetObject"
      Resource = "arn:aws:s3:::bucket/*"
    }]
  })
}

# This can cause issues if policy references role ARN
resource "aws_iam_role_policy_attachment" "app" {
  role       = aws_iam_role.app.name
  policy_arn = aws_iam_policy.app.arn
}
```

**Pattern 2: Lambda and CloudWatch Logs**

```hcl
# WRONG: Lambda needs log group, but log group name includes Lambda name
resource "aws_lambda_function" "app" {
  function_name = "my-function"
  # ...
  depends_on = [aws_cloudwatch_log_group.lambda]
}

resource "aws_cloudwatch_log_group" "lambda" {
  name = "/aws/lambda/${aws_lambda_function.app.function_name}"  # Cycle!
}

# RIGHT: Use a local or hardcode the name
locals {
  function_name = "my-function"
}

resource "aws_cloudwatch_log_group" "lambda" {
  name = "/aws/lambda/${local.function_name}"
}

resource "aws_lambda_function" "app" {
  function_name = local.function_name
  depends_on    = [aws_cloudwatch_log_group.lambda]
}
```

**Pattern 3: Route53 and ACM Certificate**

```hcl
# Certificate validation requires DNS record
# DNS record requires certificate ARN
# Use for_each with certificate_validation to break cycle

resource "aws_acm_certificate" "main" {
  domain_name       = "example.com"
  validation_method = "DNS"
}

resource "aws_route53_record" "cert_validation" {
  for_each = {
    for dvo in aws_acm_certificate.main.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }

  zone_id = var.zone_id
  name    = each.value.name
  type    = each.value.type
  records = [each.value.record]
  ttl     = 60
}

resource "aws_acm_certificate_validation" "main" {
  certificate_arn         = aws_acm_certificate.main.arn
  validation_record_fqdns = [for record in aws_route53_record.cert_validation : record.fqdn]
}
```


## Cycle Prevention Best Practices

| Pattern | Problem | Solution |
|---------|---------|----------|
| Inline SG rules | Mutual references | Separate `aws_security_group_rule` resources |
| Resource names | Resource A name depends on B | Use `locals` for names |
| IAM policies | Policy references resource being created | Use `aws_iam_policy_document` data source |
| Module outputs | Module A needs Module B output and vice versa | Restructure modules or use data sources |


---

### Quick Check

**Why does using separate aws_security_group_rule resources break a cycle between two security groups?**

   A. Separate rules are processed in a different Terraform phase
   B. aws_security_group_rule has special cycle-breaking logic
-> C. **The security groups are created first (without inline rules), then rules reference existing SGs**
   D. Terraform automatically detects and resolves cycles with separate rule resources

<details>
<summary>See Answer</summary>

When you use inline ingress/egress blocks, the security group resource itself has the dependency. By using separate aws_security_group_rule resources, you create the security groups first (they have no dependencies on each other), and then the rules can reference both existing security groups. The dependency graph becomes: create both SGs in parallel, then create all rules.

</details>

---

### Question 8: Your team argues about workspaces vs directories for dev/staging/prod. Design the right approach.

**Type:** Architecture | **Category:** Environment Strategy

## The Scenario

Your team has three environments (dev, staging, prod) and is debating how to structure Terraform:

**Engineer A:** "Let's use Terraform workspaces! One codebase, switch environments with `terraform workspace select prod`"

**Engineer B:** "No, workspaces are dangerous! Let's use separate directories for each environment."

Current structure being debated:

```
# Option A: Workspaces
terraform/
├── main.tf
├── variables.tf
└── terraform.tfvars  # Which environment's values?

# Option B: Directories
terraform/
├── dev/
│   ├── main.tf
│   └── terraform.tfvars
├── staging/
│   └── ...
└── prod/
    └── ...
```

## The Challenge

Design an environment strategy that balances code reuse, safety, and operational clarity. Consider the pros/cons of each approach and when each is appropriate.


> **Common Mistake:** A junior engineer might pick one approach dogmatically without understanding tradeoffs, use workspaces for completely different infrastructure between environments, or duplicate everything in directories losing DRY benefits. They might also mix workspace selection with manual tfvars files, leading to dangerous mistakes.

> **Senior Engineer Approach:** A senior engineer understands that workspaces are best for identical infrastructure with different scale (same resources, different sizes), while directories are better when environments have structural differences. The ideal solution often combines both: shared modules with environment-specific root configurations.

### Understanding Workspaces

```bash
# Workspaces create separate state files
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# State files:
# s3://bucket/env:/dev/terraform.tfstate
# s3://bucket/env:/staging/terraform.tfstate
# s3://bucket/env:/prod/terraform.tfstate

# Switch environments
terraform workspace select prod
terraform apply  # Applies to prod state
```

**Workspace-aware configuration:**

```hcl
# main.tf
locals {
  env_config = {
    dev = {
      instance_type = "t3.micro"
      instance_count = 1
      multi_az = false
    }
    staging = {
      instance_type = "t3.small"
      instance_count = 2
      multi_az = false
    }
    prod = {
      instance_type = "t3.large"
      instance_count = 3
      multi_az = true
    }
  }

  config = local.env_config[terraform.workspace]
}

resource "aws_instance" "web" {
  count         = local.config.instance_count
  instance_type = local.config.instance_type
  # ...
}
```

### When Workspaces Work Well

✅ **Good for:**
- Same infrastructure, different scale
- Temporary environments (feature branches)
- Simple projects with identical structure
- Single engineer/small team

```hcl
# Feature branch environments
# terraform workspace new feature-123
# terraform workspace delete feature-123

resource "aws_instance" "web" {
  count = terraform.workspace == "prod" ? 3 : 1

  tags = {
    Environment = terraform.workspace
  }
}
```

### When Workspaces Fail

❌ **Bad for:**
- Environments with different resources
- Different AWS accounts per environment
- Different providers/regions
- Large teams (workspace confusion risk)

```hcl
# DANGER: Easy to forget which workspace is selected
$ terraform workspace select prod
# ... go to meeting ...
# ... come back ...
$ terraform destroy  # Oops, just destroyed prod!
```

### The Directory Approach

```
terraform/
├── modules/              # Shared modules
│   ├── vpc/
│   ├── ecs-cluster/
│   └── rds/
├── environments/
│   ├── dev/
│   │   ├── main.tf      # Uses modules
│   │   ├── backend.tf   # Dev state location
│   │   └── terraform.tfvars
│   ├── staging/
│   │   ├── main.tf
│   │   ├── backend.tf
│   │   └── terraform.tfvars
│   └── prod/
│       ├── main.tf
│       ├── backend.tf   # Prod state location (different bucket!)
│       └── terraform.tfvars
```

**Environment-specific configuration:**

```hcl
# environments/prod/main.tf
module "vpc" {
  source = "../../modules/vpc"

  cidr_block         = "10.0.0.0/16"
  enable_nat_gateway = true
  single_nat_gateway = false  # HA for prod
}

module "ecs" {
  source = "../../modules/ecs-cluster"

  cluster_name   = "prod-cluster"
  instance_type  = "m5.xlarge"
  min_size       = 3
  max_size       = 10
}

# Prod-only resources
module "disaster_recovery" {
  source = "../../modules/dr"
  # Only prod has DR setup
}
```

```hcl
# environments/dev/main.tf
module "vpc" {
  source = "../../modules/vpc"

  cidr_block         = "10.1.0.0/16"
  enable_nat_gateway = true
  single_nat_gateway = true  # Cost savings for dev
}

module "ecs" {
  source = "../../modules/ecs-cluster"

  cluster_name   = "dev-cluster"
  instance_type  = "t3.medium"
  min_size       = 1
  max_size       = 2
}

# No DR module in dev
```

### Recommended: Hybrid with Terragrunt

```
infrastructure/
├── modules/                    # Versioned modules
│   └── ...
├── terragrunt.hcl             # Root config
└── live/
    ├── dev/
    │   ├── us-east-1/
    │   │   ├── vpc/
    │   │   │   └── terragrunt.hcl
    │   │   └── ecs/
    │   │       └── terragrunt.hcl
    │   └── account.hcl
    ├── staging/
    │   └── ...
    └── prod/
        ├── us-east-1/
        │   └── ...
        ├── us-west-2/          # Prod has multi-region
        │   └── ...
        └── account.hcl
```

```hcl
# live/terragrunt.hcl (root)
remote_state {
  backend = "s3"
  generate = {
    path      = "backend.tf"
    if_exists = "overwrite"
  }
  config = {
    bucket         = "terraform-state-${local.account_id}"
    key            = "${path_relative_to_include()}/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

```hcl
# live/prod/us-east-1/vpc/terragrunt.hcl
include "root" {
  path = find_in_parent_folders()
}

include "env" {
  path = find_in_parent_folders("account.hcl")
}

terraform {
  source = "../../../../modules//vpc"
}

inputs = {
  cidr_block         = "10.0.0.0/16"
  enable_nat_gateway = true
  single_nat_gateway = false
}
```

### Safety Mechanisms

**1. Different AWS accounts per environment:**

```hcl
# environments/prod/backend.tf
terraform {
  backend "s3" {
    bucket = "prod-terraform-state"  # In prod account
    key    = "infrastructure/terraform.tfstate"
    region = "us-east-1"
  }
}

# environments/prod/provider.tf
provider "aws" {
  region = "us-east-1"

  # Fail if wrong account
  assume_role {
    role_arn = "arn:aws:iam::PROD_ACCOUNT_ID:role/TerraformRole"
  }
}
```

**2. Account validation:**

```hcl
data "aws_caller_identity" "current" {}

locals {
  expected_account = "123456789012"  # Prod account ID
}

resource "null_resource" "account_check" {
  count = data.aws_caller_identity.current.account_id != local.expected_account ? "ERROR: Wrong AWS account!" : 0
}
```

**3. CI/CD environment isolation:**

```yaml
# .github/workflows/terraform.yml
jobs:
  plan-dev:
    environment: dev
    env:
      AWS_ROLE_ARN: ${{ secrets.DEV_AWS_ROLE }}
    steps:
      - run: cd environments/dev && terraform plan

  plan-prod:
    environment: production
    env:
      AWS_ROLE_ARN: ${{ secrets.PROD_AWS_ROLE }}
    steps:
      - run: cd environments/prod && terraform plan
```


## Decision Matrix

| Factor | Workspaces | Directories | Terragrunt |
|--------|-----------|-------------|------------|
| Code duplication | None | Some | Minimal |
| Environment isolation | Low | High | High |
| Accidental wrong env | High risk | Low risk | Low risk |
| Different resources per env | Hard | Easy | Easy |
| Different accounts | Tricky | Easy | Easy |
| Learning curve | Low | Low | Medium |
| CI/CD complexity | Medium | Low | Medium |

---

### Question 9: Implement a secure Terraform CI/CD pipeline with plan reviews, apply approvals, and state protection.

**Type:** Practical | **Category:** CI/CD Integration

## The Scenario

Your team manually runs `terraform apply` from laptops. Problems:
- No review process - anyone can apply anything
- No audit trail of who changed what
- State locking conflicts when multiple people work
- Credentials scattered on developer machines
- "Works on my machine" issues with different Terraform versions

## The Challenge

Design and implement a CI/CD pipeline that enforces code review, requires approval for production changes, maintains state integrity, and provides full audit trails.


> **Common Mistake:** A junior engineer might auto-approve all applies, store AWS credentials as plain environment variables, skip the plan output review, or allow direct pushes to main. This leads to unapproved infrastructure changes, credential exposure, and no meaningful review process.

> **Senior Engineer Approach:** A senior engineer implements a multi-stage pipeline: automated plan on PR, plan output posted for review, manual approval gates for production, OIDC authentication (no long-lived credentials), state locking, and comprehensive audit logging.

### Architecture Overview

```
PR Created
    │
    ▼
┌─────────────────┐
│  Format Check   │ ← terraform fmt -check
│   Validation    │ ← terraform validate
│   Lint (tflint) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Security Scan  │ ← tfsec, checkov
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Terraform Plan │ ← Posts plan to PR
└────────┬────────┘
         │
         ▼
    Code Review
    (Human reviews plan output)
         │
         ▼
    PR Merged
         │
         ▼
┌─────────────────┐
│  Terraform Plan │ ← Re-plan on main
└────────┬────────┘
         │
         ▼
    Manual Approval (prod)
         │
         ▼
┌─────────────────┐
│ Terraform Apply │
└─────────────────┘
```

### GitHub Actions Implementation

```yaml
# .github/workflows/terraform.yml
name: Terraform CI/CD

on:
  pull_request:
    branches: [main]
    paths:
      - 'terraform/**'
  push:
    branches: [main]
    paths:
      - 'terraform/**'

permissions:
  id-token: write    # For OIDC
  contents: read
  pull-requests: write

env:
  TF_VERSION: "1.6.0"
  TF_WORKING_DIR: "terraform"

jobs:
  # Stage 1: Validation
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Terraform Format Check
        run: terraform fmt -check -recursive
        working-directory: ${{ env.TF_WORKING_DIR }}

      - name: Terraform Init
        run: terraform init -backend=false
        working-directory: ${{ env.TF_WORKING_DIR }}

      - name: Terraform Validate
        run: terraform validate
        working-directory: ${{ env.TF_WORKING_DIR }}

  # Stage 2: Security Scanning
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: tfsec
        uses: aquasecurity/tfsec-action@v1.0.0
        with:
          working_directory: ${{ env.TF_WORKING_DIR }}
          soft_fail: false

      - name: Checkov
        uses: bridgecrewio/checkov-action@v12
        with:
          directory: ${{ env.TF_WORKING_DIR }}
          framework: terraform
          soft_fail: false

  # Stage 3: Plan
  plan:
    needs: [validate, security]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Configure AWS Credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Terraform Init
        run: terraform init
        working-directory: ${{ env.TF_WORKING_DIR }}

      - name: Terraform Plan
        id: plan
        run: |
          terraform plan -no-color -out=tfplan 2>&1 | tee plan.txt
          echo "plan<<EOF" >> $GITHUB_OUTPUT
          cat plan.txt >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT
        working-directory: ${{ env.TF_WORKING_DIR }}
        continue-on-error: true

      - name: Post Plan to PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const plan = `${{ steps.plan.outputs.plan }}`;
            const truncated = plan.length > 60000
              ? plan.substring(0, 60000) + '\n\n... (truncated)'
              : plan;

            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Terraform Plan Output

            \`\`\`hcl
            ${truncated}
            \`\`\`

            **Review the plan carefully before approving the PR.**`
            });

      - name: Save Plan Artifact
        uses: actions/upload-artifact@v4
        with:
          name: tfplan
          path: ${{ env.TF_WORKING_DIR }}/tfplan

      - name: Check Plan Status
        if: steps.plan.outcome == 'failure'
        run: exit 1

  # Stage 4: Apply (only on main, with approval)
  apply:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    needs: [plan]
    runs-on: ubuntu-latest
    environment: production  # Requires manual approval
    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Download Plan
        uses: actions/download-artifact@v4
        with:
          name: tfplan
          path: ${{ env.TF_WORKING_DIR }}

      - name: Terraform Init
        run: terraform init
        working-directory: ${{ env.TF_WORKING_DIR }}

      - name: Terraform Apply
        run: terraform apply -auto-approve tfplan
        working-directory: ${{ env.TF_WORKING_DIR }}
```

### OIDC Authentication (No Long-Lived Credentials)

```hcl
# In AWS account - create OIDC provider for GitHub
resource "aws_iam_openid_connect_provider" "github" {
  url             = "https://token.actions.githubusercontent.com"
  client_id_list  = ["sts.amazonaws.com"]
  thumbprint_list = ["6938fd4d98bab03faadb97b34396831e3780aea1"]
}

# Role that GitHub Actions will assume
resource "aws_iam_role" "github_actions" {
  name = "GitHubActionsTerraformRole"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Federated = aws_iam_openid_connect_provider.github.arn
        }
        Action = "sts:AssumeRoleWithWebIdentity"
        Condition = {
          StringEquals = {
            "token.actions.githubusercontent.com:aud" = "sts.amazonaws.com"
          }
          StringLike = {
            # Only allow from specific repo and branch
            "token.actions.githubusercontent.com:sub" = "repo:company/infrastructure:ref:refs/heads/main"
          }
        }
      }
    ]
  })
}
```

### Atlantis Alternative

```yaml
# atlantis.yaml
version: 3
projects:
  - name: production
    dir: terraform/environments/prod
    workspace: default
    autoplan:
      when_modified:
        - "*.tf"
        - "../modules/**/*.tf"
      enabled: true
    apply_requirements:
      - approved
      - mergeable

# Commands in PR:
# atlantis plan
# atlantis apply (requires approval)
```

### Multi-Environment Pipeline

```yaml
# .github/workflows/terraform-multi-env.yml
jobs:
  plan:
    strategy:
      matrix:
        environment: [dev, staging, prod]
    runs-on: ubuntu-latest
    steps:
      - name: Plan ${{ matrix.environment }}
        run: |
          cd terraform/environments/${{ matrix.environment }}
          terraform plan -out=tfplan

  apply-dev:
    needs: plan
    if: github.ref == 'refs/heads/main'
    environment: dev  # Auto-approve
    steps:
      - run: terraform apply

  apply-staging:
    needs: apply-dev
    environment: staging  # Manual approval
    steps:
      - run: terraform apply

  apply-prod:
    needs: apply-staging
    environment: production  # Requires 2 approvers
    steps:
      - run: terraform apply
```

### State Protection

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"

    # Additional protection
    kms_key_id = "arn:aws:kms:us-east-1:123:key/abc"
  }
}
```

```yaml
# S3 bucket policy - only CI/CD can modify state
{
  "Statement": [
    {
      "Sid": "DenyDeleteExceptFromCI",
      "Effect": "Deny",
      "Principal": "*",
      "Action": ["s3:DeleteObject", "s3:DeleteObjectVersion"],
      "Resource": "arn:aws:s3:::terraform-state/*",
      "Condition": {
        "StringNotEquals": {
          "aws:PrincipalArn": "arn:aws:iam::123:role/GitHubActionsTerraformRole"
        }
      }
    }
  ]
}
```


## Pipeline Security Checklist

| Control | Implementation |
|---------|---------------|
| No long-lived credentials | OIDC with short-lived tokens |
| Plan review | Post plan to PR comments |
| Apply approval | GitHub Environments with reviewers |
| Branch protection | Require PR, no direct push to main |
| State encryption | S3 SSE-KMS |
| State access logging | S3 access logs + CloudTrail |
| Audit trail | GitHub Actions logs + AWS CloudTrail |

---

### Question 10: Terraform plan takes 15 minutes and times out. Your state file has 2000+ resources. Optimize it.

**Type:** Practical | **Category:** Performance

## The Scenario

Your infrastructure has grown over 3 years. Every terraform plan now takes 15+ minutes:

```bash
$ time terraform plan
aws_instance.web[0]: Refreshing state...
aws_instance.web[1]: Refreshing state...
# ... 2000+ resources refreshing ...

# 15 minutes later...
Plan: 0 to add, 1 to change, 0 to destroy.

real    15m23.456s
```

CI/CD pipelines are timing out. Engineers are frustrated waiting. Meanwhile, AWS API rate limiting errors appear intermittently.

## The Challenge

Optimize Terraform performance for a large-scale infrastructure without sacrificing safety or manageability.


> **Common Mistake:** A junior engineer might add -parallelism=50 to speed up API calls, use -refresh=false to skip refresh entirely, or split everything into tiny states randomly. These approaches cause API rate limiting, risk applying stale plans, or create an unmaintainable mess of cross-state dependencies.

> **Senior Engineer Approach:** A senior engineer analyzes where time is spent, implements strategic state separation along service boundaries, uses -target for specific changes during development, configures provider parallelism appropriately, and considers data sources for read-only lookups.

### Step 1: Analyze Where Time Is Spent

```bash
# Enable trace logging
TF_LOG=TRACE terraform plan 2>&1 | tee plan.log

# Analyze API calls
grep "HTTP Request" plan.log | sort | uniq -c | sort -rn

# Count resources per type
terraform state list | cut -d'.' -f1-2 | sort | uniq -c | sort -rn

# Output might show:
# 500 aws_security_group_rule
# 400 aws_iam_policy_document
# 300 aws_route53_record
# 200 aws_instance
# ...
```

### Step 2: Strategic State Separation

**Bad: Random splits**
```
state-1.tfstate  # 700 random resources
state-2.tfstate  # 700 random resources
state-3.tfstate  # 700 random resources with cross-dependencies
```

**Good: Service/domain boundaries**
```
infrastructure/
├── network/           # VPC, subnets, NAT gateways (changes rarely)
│   └── terraform.tfstate
├── security/          # IAM roles, policies, SCPs
│   └── terraform.tfstate
├── data/             # RDS, ElastiCache, S3
│   └── terraform.tfstate
└── compute/          # ECS, EC2, ASGs
    └── terraform.tfstate
```

**Implementation:**

```hcl
# network/main.tf
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

output "vpc_id" {
  value = aws_vpc.main.id
}

output "private_subnet_ids" {
  value = aws_subnet.private[*].id
}
```

```hcl
# compute/main.tf
# Reference network state via data source
data "terraform_remote_state" "network" {
  backend = "s3"
  config = {
    bucket = "terraform-state"
    key    = "network/terraform.tfstate"
    region = "us-east-1"
  }
}

resource "aws_instance" "web" {
  subnet_id = data.terraform_remote_state.network.outputs.private_subnet_ids[0]
  # ...
}
```

### Step 3: Reduce Unnecessary Resources

**Remove redundant security group rules:**

```hcl
# BEFORE: 500 individual rules
resource "aws_security_group_rule" "allow_443" {
  # ...
}

# AFTER: Consolidated into security group
resource "aws_security_group" "web" {
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = var.allowed_cidrs  # Pass list instead of creating rules per CIDR
  }
}
```

**Use for_each instead of count for stable addressing:**

```hcl
# for_each creates fewer resources to track
resource "aws_route53_record" "services" {
  for_each = var.services

  name    = each.key
  type    = "A"
  zone_id = var.zone_id
  # ...
}
```

### Step 4: Provider Configuration Tuning

```hcl
# provider.tf
provider "aws" {
  region = "us-east-1"

  # Reduce parallelism to avoid rate limits
  # Default is 10, reduce if hitting limits
  max_retries = 5

  # Skip unnecessary metadata lookups
  skip_metadata_api_check     = true
  skip_region_validation      = true
  skip_credentials_validation = true
}
```

### Step 5: Use -target for Development

```bash
# During development, target specific resources
terraform plan -target=aws_instance.web

# Apply specific changes without full refresh
terraform apply -target=module.ecs_service

# WARNING: Only for development!
# Always run full plan before merging to main
```

### Step 6: Selective Refresh (Terraform 1.5+)

```bash
# Skip refresh entirely (use with caution!)
terraform plan -refresh=false

# Refresh only specific resources
terraform apply -refresh-only -target=aws_instance.web

# Better: Use refresh in a separate step
terraform refresh  # Run periodically
terraform plan -refresh=false  # Faster subsequent plans
```

### Step 7: State File Optimization

```bash
# Check state file size
aws s3 ls s3://terraform-state/terraform.tfstate --human-readable

# If state is huge, there might be orphaned resources
terraform state list | wc -l  # Count managed resources

# Remove resources that no longer exist
terraform state rm 'aws_instance.old_server'

# Compact state (remove old versions)
terraform state pull > state.json
terraform state push state.json
```

### Step 8: Consider Terraform Cloud

```hcl
# Terraform Cloud has optimized state handling
terraform {
  cloud {
    organization = "company"
    workspaces {
      name = "production"
    }
  }
}

# Benefits:
# - Remote execution (doesn't use your network)
# - Optimized state storage
# - Parallel runs across workspaces
# - Built-in cost estimation
```

### Performance Comparison

| State Size | Plan Time (Before) | Plan Time (After) |
|-----------|-------------------|-------------------|
| 2000 resources (single state) | 15 min | 15 min |
| 500 resources (network) | - | 2 min |
| 400 resources (security) | - | 1.5 min |
| 600 resources (data) | - | 3 min |
| 500 resources (compute) | - | 2 min |
| **Total (parallel)** | 15 min | **3 min** |

### Terragrunt for Parallel Execution

```bash
# Run plans for all modules in parallel
terragrunt run-all plan --parallelism=4

# Dependencies are automatically handled
# Independent modules run simultaneously
```

```hcl
# terragrunt.hcl
dependency "network" {
  config_path = "../network"
}

inputs = {
  vpc_id = dependency.network.outputs.vpc_id
}
```


## Quick Wins Checklist

| Action | Impact | Risk |
|--------|--------|------|
| Split state by domain | High | Medium (migration needed) |
| Reduce -parallelism | Medium | Low |
| Use -target (dev only) | High | Medium (incomplete plans) |
| Remove orphaned resources | Medium | Low |
| Consolidate SG rules | Medium | Low |
| Use for_each vs count | Medium | Low |

---

### Question 11: After upgrading providers, terraform plan shows resources will be destroyed and recreated. Prevent data loss.

**Type:** Debugging | **Category:** Provider Management

## The Scenario

You upgraded the AWS provider from 4.x to 5.x. Now terraform plan shows:

```bash
$ terraform plan

# aws_db_instance.production must be replaced
-/+ resource "aws_db_instance" "production" {
      ~ id                    = "prod-db" -> (known after apply)
      ~ endpoint              = "prod-db.xxx.us-east-1.rds.amazonaws.com" -> (known after apply)
      ~ engine_version        = "14.7" -> "14.7"  # No actual change!
      # (forces replacement)
      ...
    }

# aws_s3_bucket.data must be replaced
-/+ resource "aws_s3_bucket" "data" {
      ~ id     = "company-data-bucket" -> (known after apply)
      ~ bucket = "company-data-bucket" -> "company-data-bucket"
      # (forces replacement due to acl argument removal)
    }

Plan: 2 to add, 0 to change, 2 to destroy.
```

Your production database and S3 bucket will be destroyed! The data loss would be catastrophic.

## The Challenge

Understand why provider upgrades cause force-replacement, prevent data loss, and safely complete the upgrade.


> **Common Mistake:** A junior engineer might panic and revert the provider version, run apply hoping it works out, or manually delete the resources from state. Reverting delays necessary upgrades, apply would destroy production data, and state manipulation without understanding causes more problems.

> **Senior Engineer Approach:** A senior engineer reads the provider changelog for breaking changes, uses lifecycle prevent_destroy as a safety net, migrates deprecated arguments, uses moved blocks for renamed resources, and tests upgrades in non-prod first.

### Step 1: Understand What Changed

```bash
# Check provider changelog
# AWS Provider 5.0 Upgrade Guide:
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/guides/version-5-upgrade

# Common breaking changes in AWS 5.0:
# 1. S3 bucket ACL argument removed (use aws_s3_bucket_acl)
# 2. Default tags behavior changed
# 3. Some argument names changed
# 4. Resource attribute type changes
```

### Step 2: Add Safety Net First

```hcl
# Add prevent_destroy BEFORE upgrading provider
resource "aws_db_instance" "production" {
  identifier = "prod-db"
  # ...

  lifecycle {
    prevent_destroy = true
  }
}

resource "aws_s3_bucket" "data" {
  bucket = "company-data-bucket"

  lifecycle {
    prevent_destroy = true
  }
}

# Now if you accidentally run apply, it will fail safely:
# Error: Instance cannot be destroyed
```

### Step 3: Fix S3 Bucket Breaking Changes (AWS 5.0)

```hcl
# BEFORE (AWS Provider 4.x)
resource "aws_s3_bucket" "data" {
  bucket = "company-data-bucket"
  acl    = "private"  # Removed in 5.0!

  versioning {
    enabled = true
  }

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}

# AFTER (AWS Provider 5.x)
resource "aws_s3_bucket" "data" {
  bucket = "company-data-bucket"
}

# ACL is now separate resource
resource "aws_s3_bucket_acl" "data" {
  bucket = aws_s3_bucket.data.id
  acl    = "private"
}

# Versioning is now separate
resource "aws_s3_bucket_versioning" "data" {
  bucket = aws_s3_bucket.data.id
  versioning_configuration {
    status = "Enabled"
  }
}

# Encryption is now separate
resource "aws_s3_bucket_server_side_encryption_configuration" "data" {
  bucket = aws_s3_bucket.data.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```

### Step 4: Import New Resources

```bash
# The new separate resources need to be imported
terraform import aws_s3_bucket_acl.data company-data-bucket,private
terraform import aws_s3_bucket_versioning.data company-data-bucket
terraform import aws_s3_bucket_server_side_encryption_configuration.data company-data-bucket

# Verify no changes
terraform plan
# Should show: No changes.
```

### Step 5: Handle Attribute Type Changes

```hcl
# Sometimes attribute types change, causing false replacements
# Example: tags changed from map to object

# Check state for current value
terraform state show aws_db_instance.production

# If state has old format, refresh to update
terraform apply -refresh-only -target=aws_db_instance.production
```

### Step 6: Use Moved Blocks for Renames

```hcl
# If a resource type or name changed
moved {
  from = aws_s3_bucket_object.config
  to   = aws_s3_object.config  # Renamed in AWS 4.0
}

# Terraform will update state without recreating
```

### Upgrade Testing Strategy

```hcl
# versions.tf - Pin provider versions
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"  # Allows 5.x but not 6.0
    }
  }
}

# Step-by-step upgrade process:
# 1. Test in dev with new provider
# 2. Review plan output carefully
# 3. Fix breaking changes
# 4. Apply to dev
# 5. Repeat for staging
# 6. Finally upgrade prod
```

### Lock File Management

```bash
# .terraform.lock.hcl pins exact versions
# After testing, commit the lock file

# Update only specific provider
terraform init -upgrade=true

# View current locked versions
cat .terraform.lock.hcl

# If you need to update lock file
terraform providers lock -platform=linux_amd64 -platform=darwin_amd64
```

### Common Provider Upgrade Issues

| Provider | Version | Breaking Change | Fix |
|----------|---------|-----------------|-----|
| AWS | 4.0 | `aws_s3_bucket_object` → `aws_s3_object` | Use moved block |
| AWS | 5.0 | S3 bucket arguments split | Separate resources + import |
| AWS | 5.0 | Default tags handling | Update provider config |
| Google | 4.0 | Project field required | Add project to resources |
| Azure | 3.0 | Resource renames | Use moved blocks |

### Automated Upgrade Checking

```yaml
# .github/workflows/provider-check.yml
name: Check Provider Updates

on:
  schedule:
    - cron: '0 9 * * 1'  # Weekly on Monday

jobs:
  check-updates:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Check for Updates
        run: |
          terraform init
          terraform providers lock -platform=linux_amd64

          if git diff --exit-code .terraform.lock.hcl; then
            echo "No provider updates available"
          else
            echo "Provider updates available!"
            # Create PR with lock file changes
          fi
```

### Recovery from Bad Upgrade

```bash
# If you already applied and things broke:

# 1. Check state for backup versions
aws s3api list-object-versions \
  --bucket terraform-state \
  --prefix prod/terraform.tfstate

# 2. Restore previous state version
aws s3api get-object \
  --bucket terraform-state \
  --key prod/terraform.tfstate \
  --version-id "VERSION_ID" \
  previous-state.json

terraform state push previous-state.json

# 3. Revert provider version in versions.tf
# 4. Run terraform init -upgrade=false
# 5. Verify with terraform plan
```


## Provider Upgrade Checklist

| Step | Action | Verification |
|------|--------|--------------|
| 1 | Read upgrade guide | Understand breaking changes |
| 2 | Add prevent_destroy | Safety net in place |
| 3 | Upgrade in dev first | Plan shows expected changes |
| 4 | Fix deprecated arguments | No warnings in plan |
| 5 | Import new resources | State matches reality |
| 6 | Run plan | No unexpected destroys |
| 7 | Apply to dev | Successful, no data loss |
| 8 | Repeat for staging/prod | All environments updated |

---

### Question 12: Your Terraform modules have no tests. Implement a comprehensive testing strategy.

**Type:** Practical | **Category:** Testing

## The Scenario

Your team has 15 Terraform modules used across 50+ projects. Problems:

```bash
# Module changes break consumers
$ terraform plan
Error: Invalid count argument
  on .terraform/modules/vpc/main.tf line 45:
  count = var.enable_nat_gateway ? length(var.azs) : 0

  The "count" value depends on resource attributes that cannot be determined
  until apply...

# No one knows if modules work until production
# Breaking changes slip through code review
# CI only runs terraform validate (catches syntax, not logic)
```

## The Challenge

Implement a testing pyramid for Terraform: static analysis, unit tests, integration tests, and end-to-end tests. Balance test coverage with execution time and cost.


### Testing Pyramid for Terraform

```
                    ▲
                   ╱ ╲
                  ╱   ╲     End-to-End Tests
                 ╱     ╲    (Weekly, full environment)
                ╱───────╲
               ╱         ╲   Integration Tests
              ╱           ╲  (On merge, deploy to sandbox)
             ╱─────────────╲
            ╱               ╲  Unit Tests
           ╱                 ╲ (On PR, no deployment)
          ╱───────────────────╲
         ╱                     ╲ Static Analysis
        ╱                       ╲(On every commit)
       ╱─────────────────────────╲
```

### Layer 1: Static Analysis (Every Commit)

```yaml
# .github/workflows/static-analysis.yml
name: Static Analysis

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Terraform Format
        run: terraform fmt -check -recursive

      - name: TFLint
        uses: terraform-linters/setup-tflint@v4
      - run: |
          tflint --init
          tflint --recursive

      - name: tfsec Security Scan
        uses: aquasecurity/tfsec-action@v1.0.0

      - name: Checkov Policy Check
        uses: bridgecrewio/checkov-action@v12
```

**TFLint configuration:**

```hcl
# .tflint.hcl
plugin "aws" {
  enabled = true
  version = "0.27.0"
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

rule "terraform_naming_convention" {
  enabled = true
}

rule "terraform_documented_variables" {
  enabled = true
}

rule "terraform_documented_outputs" {
  enabled = true
}

# Catch common AWS mistakes
rule "aws_instance_invalid_type" {
  enabled = true
}

rule "aws_resource_missing_tags" {
  enabled = true
  tags    = ["Environment", "Owner", "Project"]
}
```

### Layer 2: Unit Tests (Terraform Test Framework)

```hcl
# tests/vpc_test.tftest.hcl
# Native Terraform testing (1.6+)

variables {
  name               = "test-vpc"
  cidr_block         = "10.0.0.0/16"
  azs                = ["us-east-1a", "us-east-1b"]
  public_subnets     = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets    = ["10.0.11.0/24", "10.0.12.0/24"]
  enable_nat_gateway = true
}

run "vpc_creates_correct_subnets" {
  command = plan  # Don't apply, just plan

  assert {
    condition     = length(aws_subnet.public) == 2
    error_message = "Expected 2 public subnets"
  }

  assert {
    condition     = length(aws_subnet.private) == 2
    error_message = "Expected 2 private subnets"
  }

  assert {
    condition     = aws_vpc.main.cidr_block == "10.0.0.0/16"
    error_message = "VPC CIDR block incorrect"
  }
}

run "nat_gateway_disabled" {
  variables {
    enable_nat_gateway = false
  }

  command = plan

  assert {
    condition     = length(aws_nat_gateway.main) == 0
    error_message = "NAT gateway should not be created when disabled"
  }
}
```

```bash
# Run tests
terraform test

# Output:
# tests/vpc_test.tftest.hcl... pass
#   run "vpc_creates_correct_subnets"... pass
#   run "nat_gateway_disabled"... pass
```

### Layer 3: Integration Tests (Terratest)

```go
// test/vpc_test.go
package test


    "testing"

    "github.com/gruntwork-io/terratest/modules/aws"
    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/stretchr/testify/assert"
)

func TestVpcModule(t *testing.T) {
    t.Parallel()

    // Use a unique name to avoid conflicts
    uniqueID := random.UniqueId()
    vpcName := fmt.Sprintf("test-vpc-%s", uniqueID)

    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        TerraformDir: "../modules/vpc",
        Vars: map[string]interface{}{
            "name":               vpcName,
            "cidr_block":         "10.0.0.0/16",
            "azs":                []string{"us-east-1a", "us-east-1b"},
            "enable_nat_gateway": true,
        },
        EnvVars: map[string]string{
            "AWS_DEFAULT_REGION": "us-east-1",
        },
    })

    // Clean up after test
    defer terraform.Destroy(t, terraformOptions)

    // Deploy
    terraform.InitAndApply(t, terraformOptions)

    // Validate outputs
    vpcID := terraform.Output(t, terraformOptions, "vpc_id")
    assert.NotEmpty(t, vpcID)

    // Validate actual AWS resources
    vpc := aws.GetVpcById(t, vpcID, "us-east-1")
    assert.Equal(t, "10.0.0.0/16", vpc.CidrBlock)

    // Validate subnets
    publicSubnets := terraform.OutputList(t, terraformOptions, "public_subnet_ids")
    assert.Equal(t, 2, len(publicSubnets))

    // Validate NAT gateway
    natGatewayIPs := terraform.OutputList(t, terraformOptions, "nat_gateway_ips")
    assert.Equal(t, 2, len(natGatewayIPs))
}

func TestVpcModuleWithoutNAT(t *testing.T) {
    t.Parallel()

    terraformOptions := &terraform.Options{
        TerraformDir: "../modules/vpc",
        Vars: map[string]interface{}{
            "enable_nat_gateway": false,
        },
    }

    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)

    // Verify no NAT gateways created
    natGatewayIPs := terraform.OutputList(t, terraformOptions, "nat_gateway_ips")
    assert.Equal(t, 0, len(natGatewayIPs))
}
```

```bash
# Run integration tests
cd test
go test -v -timeout 30m

# Run specific test
go test -v -run TestVpcModule -timeout 30m
```

### Layer 4: End-to-End Tests

```yaml
# .github/workflows/e2e-test.yml
name: E2E Tests

on:
  schedule:
    - cron: '0 2 * * 0'  # Weekly on Sunday
  workflow_dispatch:

jobs:
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy Complete Environment
        run: |
          cd examples/complete
          terraform init
          terraform apply -auto-approve

      - name: Run E2E Tests
        run: |
          # Test actual application behavior
          ./scripts/e2e-tests.sh

      - name: Cleanup
        if: always()
        run: |
          cd examples/complete
          terraform destroy -auto-approve
```

### Test Cost Optimization

```hcl
# tests/cost-optimized.tftest.hcl
# Use smaller instances for tests

variables {
  instance_type = "t3.micro"  # Override production default
  multi_az      = false       # Single AZ for tests
}

# Use mocks for expensive resources
mock_provider "aws" {
  mock_resource "aws_rds_cluster" {
    defaults = {
      endpoint = "mock-endpoint.cluster-xxx.us-east-1.rds.amazonaws.com"
      port     = 5432
    }
  }
}
```

### CI/CD Integration

```yaml
# .github/workflows/terraform-tests.yml
name: Terraform Tests

on:
  pull_request:
    paths:
      - 'modules/**'

jobs:
  # Fast: Run on every PR
  static:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: terraform fmt -check -recursive
      - run: tflint --recursive
      - run: tfsec .

  # Medium: Run on every PR
  unit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: terraform test

  # Slow: Run on merge to main
  integration:
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.TEST_AWS_ROLE }}
      - run: |
          cd test
          go test -v -timeout 30m
```


## Testing Strategy Summary

| Test Type | Runs When | Duration | Cost | Catches |
|-----------|-----------|----------|------|---------|
| Format/Lint | Every commit | Seconds | Free | Style, naming |
| tfsec/Checkov | Every commit | Seconds | Free | Security issues |
| terraform validate | Every commit | Seconds | Free | Syntax errors |
| terraform test | Every PR | Minutes | Free | Logic errors |
| Terratest | On merge | 10-30 min | $$ | Integration issues |
| E2E | Weekly | Hours | $$$ | Full system issues |


---

### Quick Check

**Why should Terraform integration tests (that deploy real resources) run on merge rather than on every PR commit?**

   A. Integration tests require special permissions that PR authors don't have
-> B. **Running tests on every commit would be too slow, expensive, and could hit API rate limits**
   C. Terraform doesn't support running tests on pull requests
   D. Integration tests can only run on the main branch

<details>
<summary>See Answer</summary>

Integration tests deploy real AWS resources, which costs money, takes 10-30 minutes, and consumes API quota. Running on every commit would make CI prohibitively slow and expensive. Instead, use fast static analysis and unit tests (terraform test with plan-only) on PRs, then run integration tests only when code is merged. This balances test coverage with cost and developer experience.

</details>

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*
