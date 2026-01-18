# Complete AWS Interview Guide

Master AWS with 12 real-world interview questions covering Lambda, VPC, DynamoDB, RDS, ECS, API Gateway, and cost optimization. Practice scenarios that mirror actual senior cloud engineer challenges.

**Companies that ask these questions:** Amazon | Netflix | Airbnb | Stripe | Slack | Lyft

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | Lambda functions are timing out when accessing RDS in a VPC.... | Debugging | Lambda |
| 2 | Design a multi-tier VPC architecture with public, private, a... | Architecture | Networking |
| 3 | DynamoDB is throttling requests and costs are high. Optimize... | Practical | DynamoDB |
| 4 | RDS connections are exhausted and failover takes too long. F... | Practical | Databases |
| 5 | Implement S3 with CloudFront for secure, cached content deli... | Practical | Storage & CDN |
| 6 | ECS tasks are failing with exit code 137 and health check fa... | Debugging | ECS |
| 7 | Messages are being lost and processed multiple times. Implem... | Practical | Messaging |
| 8 | Design a scalable API Gateway with throttling, caching, and ... | Architecture | API Gateway |
| 9 | Production incidents take hours to detect. Implement CloudWa... | Practical | Observability |
| 10 | IAM policies are too permissive. Implement least privilege a... | Practical | IAM & Security |
| 11 | Build a CI/CD pipeline with CodePipeline that deploys to ECS... | Practical | CI/CD |
| 12 | Your AWS bill increased 40% last month. Identify waste and i... | Practical | Cost Optimization |

---

## What You'll Learn

- Debug Lambda timeout and cold start issues in VPC configurations
- Design multi-tier VPC architectures with proper security groups and NAT
- Optimize DynamoDB for high-throughput workloads with proper partition keys
- Configure RDS for high availability with Multi-AZ and read replicas
- Implement S3 with CloudFront for secure, performant content delivery
- Troubleshoot ECS container failures and memory issues
- Build reliable messaging with SQS dead letter queues and SNS fan-out
- Design API Gateway with throttling, caching, and JWT authorization
- Implement CloudWatch alarms with anomaly detection
- Secure IAM policies with least privilege and permission boundaries
- Build CI/CD pipelines with CodePipeline and blue-green deployments
- Optimize AWS costs with Savings Plans and right-sizing

---

## Interview Questions

### Question 1: Lambda functions are timing out when accessing RDS in a VPC. Debug the connectivity issue.

**Type:** Debugging | **Category:** Lambda

## The Scenario

Your Lambda function suddenly started timing out:

```
Task timed out after 30.00 seconds
START RequestId: abc-123
END RequestId: abc-123
REPORT RequestId: abc-123 Duration: 30003.45 ms Billed Duration: 30000 ms Memory Size: 128 MB Max Memory Used: 45 MB Init Duration: 2534.12 ms
```

The function was working fine until you moved it into a VPC to access an RDS database.

## The Challenge

Debug why the Lambda function times out in a VPC, identify the root cause, and implement a fix that maintains security while enabling connectivity.


### Step 1: Understand the VPC Networking Issue

```
Lambda VPC Connectivity:
┌─────────────────────────────────────────────────────────────────┐
│                           VPC                                    │
│  ┌─────────────────┐              ┌─────────────────┐           │
│  │  Public Subnet  │              │  Private Subnet │           │
│  │                 │              │                 │           │
│  │  ┌───────────┐  │              │  ┌───────────┐  │           │
│  │  │    NAT    │  │              │  │  Lambda   │  │           │
│  │  │  Gateway  │◄─┼──────────────┼──│ Function  │  │           │
│  │  └─────┬─────┘  │              │  └───────────┘  │           │
│  │        │        │              │        │        │           │
│  └────────┼────────┘              └────────┼────────┘           │
│           │                                │                     │
│           ▼                                ▼                     │
│    Internet Gateway                 RDS (same VPC)              │
│           │                                                      │
└───────────┼──────────────────────────────────────────────────────┘
            ▼
      AWS Services
   (Secrets Manager,
    CloudWatch, etc.)
```

### Step 2: Diagnose the Issue

```bash
# Check Lambda VPC configuration
aws lambda get-function-configuration \
  --function-name my-function \
  --query '{VpcConfig: VpcConfig, Timeout: Timeout}'

# Check if subnets have route to NAT Gateway
aws ec2 describe-route-tables \
  --filters "Name=association.subnet-id,Values=subnet-abc123" \
  --query 'RouteTables[].Routes[]'

# Verify security group allows outbound traffic
aws ec2 describe-security-groups \
  --group-ids sg-lambda123 \
  --query 'SecurityGroups[].{Egress: IpPermissionsEgress}'

# Check NAT Gateway status
aws ec2 describe-nat-gateways \
  --filter "Name=vpc-id,Values=vpc-123" \
  --query 'NatGateways[].{State: State, SubnetId: SubnetId}'
```

### Step 3: Fix with Terraform - NAT Gateway Approach

```hcl
# VPC Configuration
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
}

# Public subnet for NAT Gateway
resource "aws_subnet" "public" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "public-${count.index}"
  }
}

# Private subnets for Lambda
resource "aws_subnet" "private" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 10}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "private-${count.index}"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
}

# Elastic IP for NAT Gateway
resource "aws_eip" "nat" {
  domain = "vpc"
}

# NAT Gateway in public subnet
resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public[0].id

  depends_on = [aws_internet_gateway.main]
}

# Route table for public subnets
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
}

# Route table for private subnets (Lambda)
resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main.id
  }
}

# Associate private subnets with private route table
resource "aws_route_table_association" "private" {
  count          = 2
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private.id
}

# Lambda security group
resource "aws_security_group" "lambda" {
  name   = "lambda-sg"
  vpc_id = aws_vpc.main.id

  # Allow all outbound traffic
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# RDS security group
resource "aws_security_group" "rds" {
  name   = "rds-sg"
  vpc_id = aws_vpc.main.id

  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.lambda.id]
  }
}

# Lambda function
resource "aws_lambda_function" "main" {
  function_name = "my-function"
  runtime       = "python3.11"
  handler       = "handler.main"
  timeout       = 30
  memory_size   = 256

  vpc_config {
    subnet_ids         = aws_subnet.private[*].id
    security_group_ids = [aws_security_group.lambda.id]
  }

  environment {
    variables = {
      DB_HOST = aws_db_instance.main.endpoint
    }
  }
}
```

### Step 4: Better Approach - VPC Endpoints (No NAT costs)

```hcl
# VPC Endpoint for Secrets Manager (Interface endpoint)
resource "aws_vpc_endpoint" "secretsmanager" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.${var.region}.secretsmanager"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.vpc_endpoint.id]
  private_dns_enabled = true
}

# VPC Endpoint for CloudWatch Logs
resource "aws_vpc_endpoint" "logs" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.${var.region}.logs"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.vpc_endpoint.id]
  private_dns_enabled = true
}

# VPC Endpoint for S3 (Gateway endpoint - free)
resource "aws_vpc_endpoint" "s3" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.${var.region}.s3"
  vpc_endpoint_type = "Gateway"
  route_table_ids   = [aws_route_table.private.id]
}

# VPC Endpoint for DynamoDB (Gateway endpoint - free)
resource "aws_vpc_endpoint" "dynamodb" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.${var.region}.dynamodb"
  vpc_endpoint_type = "Gateway"
  route_table_ids   = [aws_route_table.private.id]
}

# Security group for VPC endpoints
resource "aws_security_group" "vpc_endpoint" {
  name   = "vpc-endpoint-sg"
  vpc_id = aws_vpc.main.id

  ingress {
    from_port       = 443
    to_port         = 443
    protocol        = "tcp"
    security_groups = [aws_security_group.lambda.id]
  }
}
```

### Step 5: Optimize Cold Starts

```python
# handler.py - Connection reuse pattern


from psycopg2 import pool

# Initialize connection pool OUTSIDE handler (reused across invocations)
connection_pool = None

def get_connection_pool():
    global connection_pool
    if connection_pool is None:
        connection_pool = psycopg2.pool.SimpleConnectionPool(
            minconn=1,
            maxconn=5,
            host=os.environ['DB_HOST'],
            database=os.environ['DB_NAME'],
            user=os.environ['DB_USER'],
            password=os.environ['DB_PASSWORD'],
            connect_timeout=5
        )
    return connection_pool

def main(event, context):
    pool = get_connection_pool()
    conn = pool.getconn()

    try:
        with conn.cursor() as cur:
            cur.execute("SELECT * FROM orders WHERE id = %s", (event['order_id'],))
            result = cur.fetchone()
            return {'statusCode': 200, 'body': result}
    finally:
        pool.putconn(conn)
```

### Step 6: Use RDS Proxy for Better Connection Management

```hcl
resource "aws_db_proxy" "main" {
  name                   = "rds-proxy"
  debug_logging          = false
  engine_family          = "POSTGRESQL"
  idle_client_timeout    = 1800
  require_tls            = true
  vpc_security_group_ids = [aws_security_group.rds_proxy.id]
  vpc_subnet_ids         = aws_subnet.private[*].id

  auth {
    auth_scheme               = "SECRETS"
    iam_auth                  = "REQUIRED"
    secret_arn                = aws_secretsmanager_secret.db_credentials.arn
  }
}

resource "aws_db_proxy_default_target_group" "main" {
  db_proxy_name = aws_db_proxy.main.name

  connection_pool_config {
    max_connections_percent      = 100
    max_idle_connections_percent = 50
    connection_borrow_timeout    = 120
  }
}

resource "aws_db_proxy_target" "main" {
  db_proxy_name          = aws_db_proxy.main.name
  target_group_name      = aws_db_proxy_default_target_group.main.name
  db_instance_identifier = aws_db_instance.main.id
}
```


## Common Lambda VPC Issues

| Issue | Symptom | Solution |
|-------|---------|----------|
| No internet access | Timeout calling AWS APIs | Add NAT Gateway or VPC endpoints |
| ENI creation slow | Cold starts over 10s | Use provisioned concurrency |
| Connection exhaustion | "Too many connections" | Use RDS Proxy |
| DNS resolution fails | "Name resolution failed" | Enable DNS support in VPC |
| Security group blocks | Timeout on specific ports | Check egress rules |

<InterviewQuiz
  question="Why do Lambda functions in a VPC lose internet connectivity by default?"
  options={[
    "AWS disables internet access for security",
    "Lambda functions in a VPC use private IPs with no route to the internet gateway",
    "VPC Lambda functions have restricted IAM permissions",
    "The Lambda execution role needs additional policies"
  ]}
  correctAnswer={1}
  explanation="When a Lambda function is attached to a VPC, it receives a private IP address from the subnet's CIDR range. Private IPs cannot route directly through an Internet Gateway - they need NAT (Network Address Translation). The solution is either a NAT Gateway in a public subnet (costs ~$32/month + data transfer) or VPC endpoints for specific AWS services (interface endpoints cost ~$7/month each, gateway endpoints for S3/DynamoDB are free)."
/>

---

### Question 2: Design a multi-tier VPC architecture with public, private, and database subnets.

**Type:** Architecture | **Category:** Networking

## The Scenario

You're designing the network architecture for a new application:

```
Requirements:
├── Web tier: Public-facing load balancer
├── App tier: Private EC2/ECS instances
├── Data tier: RDS and ElastiCache
├── Security: No direct internet access to app/data tiers
├── Compliance: All traffic logged
└── High availability: Multi-AZ deployment
```

## The Challenge

Design a secure, scalable VPC architecture that properly isolates tiers, enables necessary connectivity, and follows AWS best practices.


> **Common Mistake:** A junior engineer might put everything in public subnets, use one security group for all resources, skip NAT Gateways to save costs, or use a single AZ. These approaches create security risks, violate least privilege, break high availability, and fail compliance audits.

> **Senior Engineer Approach:** A senior engineer designs with proper subnet tiers, security groups per resource type, NAT Gateways for outbound traffic, VPC Flow Logs for auditing, and multi-AZ deployment for high availability.

### Step 1: VPC Architecture Overview

```
Multi-Tier VPC Architecture:
┌─────────────────────────────────────────────────────────────────────────────┐
│                                    VPC                                       │
│                              10.0.0.0/16                                     │
│                                                                              │
│   Availability Zone A                    Availability Zone B                 │
│   ┌──────────────────────┐              ┌──────────────────────┐            │
│   │  Public Subnet       │              │  Public Subnet       │            │
│   │  10.0.1.0/24         │              │  10.0.2.0/24         │            │
│   │  ┌────────────────┐  │              │  ┌────────────────┐  │            │
│   │  │ NAT Gateway    │  │              │  │ NAT Gateway    │  │            │
│   │  │ ALB            │  │              │  │ ALB            │  │            │
│   │  └────────────────┘  │              │  └────────────────┘  │            │
│   └──────────────────────┘              └──────────────────────┘            │
│                                                                              │
│   ┌──────────────────────┐              ┌──────────────────────┐            │
│   │  Private Subnet      │              │  Private Subnet      │            │
│   │  10.0.11.0/24        │              │  10.0.12.0/24        │            │
│   │  ┌────────────────┐  │              │  ┌────────────────┐  │            │
│   │  │ EC2/ECS Tasks  │  │              │  │ EC2/ECS Tasks  │  │            │
│   │  │ Lambda         │  │              │  │ Lambda         │  │            │
│   │  └────────────────┘  │              │  └────────────────┘  │            │
│   └──────────────────────┘              └──────────────────────┘            │
│                                                                              │
│   ┌──────────────────────┐              ┌──────────────────────┐            │
│   │  Database Subnet     │              │  Database Subnet     │            │
│   │  10.0.21.0/24        │              │  10.0.22.0/24        │            │
│   │  ┌────────────────┐  │              │  ┌────────────────┐  │            │
│   │  │ RDS Primary    │  │◄────────────►│  │ RDS Standby    │  │            │
│   │  │ ElastiCache    │  │              │  │ ElastiCache    │  │            │
│   │  └────────────────┘  │              │  └────────────────┘  │            │
│   └──────────────────────┘              └──────────────────────┘            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Step 2: VPC and Subnet Configuration

```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "production-vpc"
  }
}

# Public Subnets (for ALB, NAT Gateway)
resource "aws_subnet" "public" {
  count                   = 2
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index + 1}.0/24"
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "public-${data.aws_availability_zones.available.names[count.index]}"
    Tier = "public"
  }
}

# Private Subnets (for application tier)
resource "aws_subnet" "private" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 11}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "private-${data.aws_availability_zones.available.names[count.index]}"
    Tier = "private"
  }
}

# Database Subnets (isolated tier)
resource "aws_subnet" "database" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 21}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "database-${data.aws_availability_zones.available.names[count.index]}"
    Tier = "database"
  }
}

# DB Subnet Group
resource "aws_db_subnet_group" "main" {
  name       = "main-db-subnet-group"
  subnet_ids = aws_subnet.database[*].id

  tags = {
    Name = "Main DB Subnet Group"
  }
}
```

### Step 3: Internet Gateway and NAT Gateways

```hcl
# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main-igw"
  }
}

# Elastic IPs for NAT Gateways
resource "aws_eip" "nat" {
  count  = 2
  domain = "vpc"

  tags = {
    Name = "nat-eip-${count.index}"
  }
}

# NAT Gateways (one per AZ for HA)
resource "aws_nat_gateway" "main" {
  count         = 2
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id

  tags = {
    Name = "nat-gw-${count.index}"
  }

  depends_on = [aws_internet_gateway.main]
}
```

### Step 4: Route Tables

```hcl
# Public Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "public-rt"
  }
}

resource "aws_route_table_association" "public" {
  count          = 2
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

# Private Route Tables (one per AZ for AZ-local NAT)
resource "aws_route_table" "private" {
  count  = 2
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[count.index].id
  }

  tags = {
    Name = "private-rt-${count.index}"
  }
}

resource "aws_route_table_association" "private" {
  count          = 2
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}

# Database Route Table (no internet access)
resource "aws_route_table" "database" {
  vpc_id = aws_vpc.main.id

  # No default route - isolated from internet

  tags = {
    Name = "database-rt"
  }
}

resource "aws_route_table_association" "database" {
  count          = 2
  subnet_id      = aws_subnet.database[count.index].id
  route_table_id = aws_route_table.database.id
}
```

### Step 5: Security Groups

```hcl
# ALB Security Group
resource "aws_security_group" "alb" {
  name        = "alb-sg"
  description = "Security group for Application Load Balancer"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "HTTPS from internet"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTP for redirect"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "alb-sg"
  }
}

# Application Security Group
resource "aws_security_group" "app" {
  name        = "app-sg"
  description = "Security group for application tier"
  vpc_id      = aws_vpc.main.id

  ingress {
    description     = "Traffic from ALB"
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "app-sg"
  }
}

# Database Security Group
resource "aws_security_group" "database" {
  name        = "database-sg"
  description = "Security group for database tier"
  vpc_id      = aws_vpc.main.id

  ingress {
    description     = "PostgreSQL from app tier"
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
  }

  # No egress rule needed for RDS (managed by AWS)

  tags = {
    Name = "database-sg"
  }
}

# ElastiCache Security Group
resource "aws_security_group" "cache" {
  name        = "cache-sg"
  description = "Security group for ElastiCache"
  vpc_id      = aws_vpc.main.id

  ingress {
    description     = "Redis from app tier"
    from_port       = 6379
    to_port         = 6379
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
  }

  tags = {
    Name = "cache-sg"
  }
}
```

### Step 6: VPC Flow Logs

```hcl
# CloudWatch Log Group for Flow Logs
resource "aws_cloudwatch_log_group" "flow_logs" {
  name              = "/aws/vpc/flow-logs"
  retention_in_days = 30
}

# IAM Role for Flow Logs
resource "aws_iam_role" "flow_logs" {
  name = "vpc-flow-logs-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "vpc-flow-logs.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy" "flow_logs" {
  name = "vpc-flow-logs-policy"
  role = aws_iam_role.flow_logs.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Action = [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "logs:DescribeLogGroups",
        "logs:DescribeLogStreams"
      ]
      Resource = "*"
    }]
  })
}

# VPC Flow Logs
resource "aws_flow_log" "main" {
  vpc_id                   = aws_vpc.main.id
  traffic_type             = "ALL"
  log_destination_type     = "cloud-watch-logs"
  log_destination          = aws_cloudwatch_log_group.flow_logs.arn
  iam_role_arn             = aws_iam_role.flow_logs.arn
  max_aggregation_interval = 60

  tags = {
    Name = "main-vpc-flow-logs"
  }
}
```

### Step 7: VPC Endpoints for AWS Services

```hcl
# Gateway Endpoints (free)
resource "aws_vpc_endpoint" "s3" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.${var.region}.s3"
  vpc_endpoint_type = "Gateway"
  route_table_ids   = concat(
    aws_route_table.private[*].id,
    [aws_route_table.database.id]
  )

  tags = {
    Name = "s3-endpoint"
  }
}

resource "aws_vpc_endpoint" "dynamodb" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.${var.region}.dynamodb"
  vpc_endpoint_type = "Gateway"
  route_table_ids   = aws_route_table.private[*].id

  tags = {
    Name = "dynamodb-endpoint"
  }
}

# Interface Endpoints (for services that need them)
resource "aws_vpc_endpoint" "ecr_api" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.${var.region}.ecr.api"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.vpc_endpoints.id]
  private_dns_enabled = true

  tags = {
    Name = "ecr-api-endpoint"
  }
}

resource "aws_vpc_endpoint" "ecr_dkr" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.${var.region}.ecr.dkr"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.vpc_endpoints.id]
  private_dns_enabled = true

  tags = {
    Name = "ecr-dkr-endpoint"
  }
}

resource "aws_vpc_endpoint" "secretsmanager" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.${var.region}.secretsmanager"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.vpc_endpoints.id]
  private_dns_enabled = true

  tags = {
    Name = "secretsmanager-endpoint"
  }
}

# Security Group for VPC Endpoints
resource "aws_security_group" "vpc_endpoints" {
  name        = "vpc-endpoints-sg"
  description = "Security group for VPC endpoints"
  vpc_id      = aws_vpc.main.id

  ingress {
    description     = "HTTPS from VPC"
    from_port       = 443
    to_port         = 443
    protocol        = "tcp"
    cidr_blocks     = [aws_vpc.main.cidr_block]
  }

  tags = {
    Name = "vpc-endpoints-sg"
  }
}
```

### Step 8: Network ACLs (Defense in Depth)

```hcl
# Database NACL - additional layer of security
resource "aws_network_acl" "database" {
  vpc_id     = aws_vpc.main.id
  subnet_ids = aws_subnet.database[*].id

  # Allow inbound PostgreSQL from private subnets only
  ingress {
    protocol   = "tcp"
    rule_no    = 100
    action     = "allow"
    cidr_block = "10.0.11.0/24"
    from_port  = 5432
    to_port    = 5432
  }

  ingress {
    protocol   = "tcp"
    rule_no    = 101
    action     = "allow"
    cidr_block = "10.0.12.0/24"
    from_port  = 5432
    to_port    = 5432
  }

  # Allow ephemeral ports for return traffic
  ingress {
    protocol   = "tcp"
    rule_no    = 200
    action     = "allow"
    cidr_block = "10.0.0.0/16"
    from_port  = 1024
    to_port    = 65535
  }

  # Allow outbound to private subnets
  egress {
    protocol   = "tcp"
    rule_no    = 100
    action     = "allow"
    cidr_block = "10.0.11.0/24"
    from_port  = 1024
    to_port    = 65535
  }

  egress {
    protocol   = "tcp"
    rule_no    = 101
    action     = "allow"
    cidr_block = "10.0.12.0/24"
    from_port  = 1024
    to_port    = 65535
  }

  tags = {
    Name = "database-nacl"
  }
}
```


## VPC Design Best Practices

| Component | Recommendation | Purpose |
|-----------|---------------|---------|
| CIDR Block | /16 for VPC, /24 for subnets | Room for growth |
| AZs | Minimum 2, prefer 3 | High availability |
| NAT Gateway | One per AZ | AZ-independent failover |
| Security Groups | Per resource type | Least privilege |
| NACLs | Database tier | Defense in depth |
| Flow Logs | All traffic | Compliance and debugging |
| VPC Endpoints | S3, DynamoDB, ECR | Reduce NAT costs |

---

### Question 3: DynamoDB is throttling requests and costs are high. Optimize the table design.

**Type:** Practical | **Category:** DynamoDB

## The Scenario

Your DynamoDB table is having issues:

```
Current Problems:
├── ProvisionedThroughputExceededException: 500+ per hour
├── Monthly cost: $2,400 (10x expected)
├── Hot partition detected: "USER#premium"
├── Table size: 50GB
├── Read capacity: 5,000 RCU (provisioned)
├── Write capacity: 1,000 WCU (provisioned)
└── GSI count: 5 (all with same partition key)
```

## The Challenge

Optimize the DynamoDB table design to eliminate throttling, reduce costs, and handle traffic spikes without over-provisioning.


### Step 1: Analyze Current Access Patterns

```bash
# Check table metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/DynamoDB \
  --metric-name ConsumedReadCapacityUnits \
  --dimensions Name=TableName,Value=orders \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-02T00:00:00Z \
  --period 3600 \
  --statistics Sum

# Check for throttling
aws cloudwatch get-metric-statistics \
  --namespace AWS/DynamoDB \
  --metric-name ThrottledRequests \
  --dimensions Name=TableName,Value=orders \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-02T00:00:00Z \
  --period 300 \
  --statistics Sum

# Describe table
aws dynamodb describe-table --table-name orders \
  --query '{PartitionKey: Table.KeySchema, ItemCount: Table.ItemCount, Size: Table.TableSizeBytes}'
```

### Step 2: Fix Hot Partition with Write Sharding

```python
# BEFORE: Hot partition key
# PK: "USER#premium" receives 80% of writes

# AFTER: Write sharding


from boto3.dynamodb.conditions import Key

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('orders')

SHARD_COUNT = 10

def get_sharded_pk(user_type: str) -> str:
    """Add random shard suffix to distribute writes."""
    shard = random.randint(0, SHARD_COUNT - 1)
    return f"{user_type}#{shard}"

def put_order(order: dict):
    """Write order with sharded partition key."""
    item = {
        'PK': get_sharded_pk(order['user_type']),
        'SK': f"ORDER#{order['order_id']}",
        'user_id': order['user_id'],
        'amount': order['amount'],
        'created_at': order['created_at'],
        # Store original PK for queries
        'user_type': order['user_type']
    }
    table.put_item(Item=item)

def query_orders_by_type(user_type: str) -> list:
    """Query all shards in parallel."""
    import concurrent.futures

    def query_shard(shard: int):
        response = table.query(
            KeyConditionExpression=Key('PK').eq(f"{user_type}#{shard}")
        )
        return response['Items']

    all_items = []
    with concurrent.futures.ThreadPoolExecutor(max_workers=SHARD_COUNT) as executor:
        futures = [executor.submit(query_shard, i) for i in range(SHARD_COUNT)]
        for future in concurrent.futures.as_completed(futures):
            all_items.extend(future.result())

    return all_items
```

### Step 3: Optimized Table Design

```hcl
resource "aws_dynamodb_table" "orders" {
  name         = "orders"
  billing_mode = "PAY_PER_REQUEST"  # On-demand for variable traffic
  hash_key     = "PK"
  range_key    = "SK"

  attribute {
    name = "PK"
    type = "S"
  }

  attribute {
    name = "SK"
    type = "S"
  }

  attribute {
    name = "GSI1PK"
    type = "S"
  }

  attribute {
    name = "GSI1SK"
    type = "S"
  }

  # Single GSI with overloaded keys (instead of 5 GSIs)
  global_secondary_index {
    name            = "GSI1"
    hash_key        = "GSI1PK"
    range_key       = "GSI1SK"
    projection_type = "ALL"
  }

  # TTL for automatic cleanup
  ttl {
    attribute_name = "expires_at"
    enabled        = true
  }

  # Point-in-time recovery
  point_in_time_recovery {
    enabled = true
  }

  tags = {
    Environment = "production"
  }
}
```

### Step 4: Single Table Design Pattern

```python
"""
Single Table Design - Multiple entity types in one table

Entity Types:
- USER: User profile data
- ORDER: Order records
- PRODUCT: Product catalog

Access Patterns:
1. Get user by ID
2. Get all orders for a user
3. Get order by ID
4. Get orders by status (GSI)
5. Get products by category (GSI)
"""

# Item structure examples
items = [
    # User entity
    {
        'PK': 'USER#user123',
        'SK': 'PROFILE',
        'entity_type': 'USER',
        'email': 'user@example.com',
        'name': 'John Doe',
        'GSI1PK': 'USER',  # For listing all users
        'GSI1SK': 'user123'
    },
    # Order entity
    {
        'PK': 'USER#user123',
        'SK': 'ORDER#2024-01-15#order456',
        'entity_type': 'ORDER',
        'order_id': 'order456',
        'amount': 99.99,
        'status': 'SHIPPED',
        'GSI1PK': 'ORDER#SHIPPED',  # For querying by status
        'GSI1SK': '2024-01-15#order456'
    },
    # Product entity
    {
        'PK': 'PRODUCT#prod789',
        'SK': 'DETAILS',
        'entity_type': 'PRODUCT',
        'name': 'Widget',
        'price': 29.99,
        'GSI1PK': 'CATEGORY#electronics',  # For querying by category
        'GSI1SK': 'PRODUCT#prod789'
    }
]

# Access pattern implementations
def get_user(user_id: str):
    """Get user profile."""
    response = table.get_item(
        Key={'PK': f'USER#{user_id}', 'SK': 'PROFILE'}
    )
    return response.get('Item')

def get_user_orders(user_id: str, limit: int = 20):
    """Get all orders for a user, sorted by date."""
    response = table.query(
        KeyConditionExpression=Key('PK').eq(f'USER#{user_id}') & Key('SK').begins_with('ORDER#'),
        ScanIndexForward=False,  # Descending order
        Limit=limit
    )
    return response['Items']

def get_orders_by_status(status: str, limit: int = 100):
    """Get orders by status using GSI."""
    response = table.query(
        IndexName='GSI1',
        KeyConditionExpression=Key('GSI1PK').eq(f'ORDER#{status}'),
        Limit=limit
    )
    return response['Items']

def get_products_by_category(category: str):
    """Get products by category using GSI."""
    response = table.query(
        IndexName='GSI1',
        KeyConditionExpression=Key('GSI1PK').eq(f'CATEGORY#{category}')
    )
    return response['Items']
```

### Step 5: Batch Operations for Efficiency

```python
from boto3.dynamodb.types import TypeSerializer


dynamodb = boto3.client('dynamodb')
serializer = TypeSerializer()

def batch_write_orders(orders: list):
    """Batch write up to 25 items at a time."""
    # DynamoDB limit: 25 items per batch
    batch_size = 25

    for i in range(0, len(orders), batch_size):
        batch = orders[i:i + batch_size]

        request_items = {
            'orders': [
                {
                    'PutRequest': {
                        'Item': {k: serializer.serialize(v) for k, v in order.items()}
                    }
                }
                for order in batch
            ]
        }

        response = dynamodb.batch_write_item(RequestItems=request_items)

        # Handle unprocessed items (throttling)
        unprocessed = response.get('UnprocessedItems', {})
        while unprocessed:
            import time
            time.sleep(0.1)  # Exponential backoff in production
            response = dynamodb.batch_write_item(RequestItems=unprocessed)
            unprocessed = response.get('UnprocessedItems', {})

def batch_get_orders(order_keys: list):
    """Batch get up to 100 items at a time."""
    batch_size = 100
    all_items = []

    for i in range(0, len(order_keys), batch_size):
        batch = order_keys[i:i + batch_size]

        request_items = {
            'orders': {
                'Keys': [
                    {
                        'PK': {'S': key['PK']},
                        'SK': {'S': key['SK']}
                    }
                    for key in batch
                ]
            }
        }

        response = dynamodb.batch_get_item(RequestItems=request_items)
        all_items.extend(response['Responses']['orders'])

    return all_items
```

### Step 6: Cost Optimization Strategies

```hcl
# On-demand for unpredictable traffic
resource "aws_dynamodb_table" "orders_ondemand" {
  name         = "orders"
  billing_mode = "PAY_PER_REQUEST"
  # Automatically scales
  # Pay only for what you use
  # Best for: variable traffic, new applications
}

# Provisioned with auto-scaling for predictable traffic
resource "aws_dynamodb_table" "orders_provisioned" {
  name         = "orders"
  billing_mode = "PROVISIONED"
  read_capacity  = 100
  write_capacity = 50
}

resource "aws_appautoscaling_target" "read" {
  max_capacity       = 1000
  min_capacity       = 100
  resource_id        = "table/${aws_dynamodb_table.orders_provisioned.name}"
  scalable_dimension = "dynamodb:table:ReadCapacityUnits"
  service_namespace  = "dynamodb"
}

resource "aws_appautoscaling_policy" "read" {
  name               = "DynamoDBReadAutoScaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.read.resource_id
  scalable_dimension = aws_appautoscaling_target.read.scalable_dimension
  service_namespace  = aws_appautoscaling_target.read.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "DynamoDBReadCapacityUtilization"
    }
    target_value       = 70.0
    scale_in_cooldown  = 60
    scale_out_cooldown = 60
  }
}

# Reserved capacity for consistent workloads (save up to 76%)
# Must be purchased through AWS Console or CLI
```

### Step 7: Query Optimization

```python
# Use ProjectionExpression to reduce read costs
def get_order_summary(user_id: str, order_id: str):
    """Get only needed attributes."""
    response = table.get_item(
        Key={'PK': f'USER#{user_id}', 'SK': f'ORDER#{order_id}'},
        ProjectionExpression='order_id, amount, #s, created_at',
        ExpressionAttributeNames={'#s': 'status'}  # 'status' is reserved
    )
    return response.get('Item')

# Use FilterExpression sparingly (still consumes full read capacity)
def get_recent_high_value_orders(user_id: str, min_amount: float):
    """Filter after query - use with caution."""
    response = table.query(
        KeyConditionExpression=Key('PK').eq(f'USER#{user_id}') & Key('SK').begins_with('ORDER#'),
        FilterExpression='amount >= :min_amount',
        ExpressionAttributeValues={':min_amount': min_amount}
    )
    return response['Items']

# Better: Use GSI with amount in sort key
def get_high_value_orders_optimized(min_amount: float):
    """Query GSI where amount is in the key."""
    response = table.query(
        IndexName='GSI-ByAmount',
        KeyConditionExpression=Key('entity_type').eq('ORDER') & Key('amount').gte(min_amount)
    )
    return response['Items']
```


## DynamoDB Cost Optimization Summary

| Strategy | Savings | When to Use |
|----------|---------|-------------|
| On-demand billing | Variable | Unpredictable traffic |
| Auto-scaling | 30-50% | Predictable patterns |
| Reserved capacity | Up to 76% | Consistent baseline |
| Single table design | 50%+ storage | Multiple entity types |
| ProjectionExpression | Per-query | Large items, few attributes needed |
| TTL | Storage costs | Temporary data |


---

### Quick Check

**Why does a hot partition cause throttling even when you have unused capacity in other partitions?**

   A. DynamoDB has a bug in capacity distribution
-> B. **Each partition has a hard limit of 3,000 RCU and 1,000 WCU regardless of total table capacity**
   C. Hot partitions automatically disable auto-scaling
   D. DynamoDB throttles to prevent data corruption

<details>
<summary>See Answer</summary>

DynamoDB distributes capacity across partitions. Each partition can handle maximum 3,000 RCU and 1,000 WCU. If you have 10,000 RCU total but one partition key receives 80% of traffic, that partition hits its 3,000 RCU limit while other partitions sit idle. The solution is to design partition keys that distribute traffic evenly, or use write sharding for known hot keys. Adaptive capacity helps but has limits.

</details>

---

### Question 4: RDS connections are exhausted and failover takes too long. Fix the database setup.

**Type:** Practical | **Category:** Databases

## The Scenario

Your RDS PostgreSQL database is having issues:

```
Current Problems:
├── "FATAL: too many connections" errors
├── Application timeout during failover: 2-3 minutes
├── Max connections: 100 (db.t3.medium)
├── Active connections: 95-100 constantly
├── Lambda functions: 50 concurrent
├── ECS tasks: 20 replicas
└── Connection pooling: None
```

## The Challenge

Fix the connection exhaustion problem, reduce failover time, and implement proper connection management for a highly available database setup.


> **Common Mistake:** A junior engineer might just upgrade to a larger instance, increase max_connections parameter, or create connections on every request. These approaches increase costs without fixing the root cause, can cause OOM issues, and create connection storms.

> **Senior Engineer Approach:** A senior engineer implements RDS Proxy for connection pooling, configures Multi-AZ with proper DNS caching settings, uses connection retry logic, and right-sizes the instance based on actual workload needs.

### Step 1: Understand the Connection Problem

```
Connection Math:
├── Lambda: 50 concurrent × 1 connection = 50 connections
├── ECS: 20 tasks × 10 pool size = 200 connections
├── Total needed: 250 connections
├── Available: 100 connections
└── Result: Connection exhaustion

With RDS Proxy:
├── Lambda: 50 concurrent → Proxy → 10 DB connections
├── ECS: 20 tasks → Proxy → 40 DB connections
├── Total DB connections: 50 (pin on transaction)
└── Result: Efficient connection reuse
```

### Step 2: Deploy RDS Proxy

```hcl
# Store database credentials in Secrets Manager
resource "aws_secretsmanager_secret" "db_credentials" {
  name = "rds/postgres/credentials"
}

resource "aws_secretsmanager_secret_version" "db_credentials" {
  secret_id = aws_secretsmanager_secret.db_credentials.id
  secret_string = jsonencode({
    username = "admin"
    password = random_password.db.result
  })
}

# RDS Proxy
resource "aws_db_proxy" "main" {
  name                   = "orders-db-proxy"
  debug_logging          = false
  engine_family          = "POSTGRESQL"
  idle_client_timeout    = 1800
  require_tls            = true
  vpc_security_group_ids = [aws_security_group.rds_proxy.id]
  vpc_subnet_ids         = aws_subnet.database[*].id

  auth {
    auth_scheme               = "SECRETS"
    client_password_auth_type = "POSTGRES_SCRAM_SHA_256"
    iam_auth                  = "REQUIRED"
    secret_arn                = aws_secretsmanager_secret.db_credentials.arn
  }

  tags = {
    Name = "orders-db-proxy"
  }
}

# Proxy target group configuration
resource "aws_db_proxy_default_target_group" "main" {
  db_proxy_name = aws_db_proxy.main.name

  connection_pool_config {
    max_connections_percent      = 100
    max_idle_connections_percent = 50
    connection_borrow_timeout    = 120
    init_query                   = "SET timezone='UTC'"
    session_pinning_filters      = ["EXCLUDE_VARIABLE_SETS"]
  }
}

# Associate proxy with RDS instance
resource "aws_db_proxy_target" "main" {
  db_proxy_name          = aws_db_proxy.main.name
  target_group_name      = aws_db_proxy_default_target_group.main.name
  db_instance_identifier = aws_db_instance.main.id
}

# Security group for RDS Proxy
resource "aws_security_group" "rds_proxy" {
  name        = "rds-proxy-sg"
  description = "Security group for RDS Proxy"
  vpc_id      = aws_vpc.main.id

  ingress {
    description     = "PostgreSQL from app tier"
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
  }

  egress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.rds.id]
  }
}
```

### Step 3: Configure Multi-AZ RDS

```hcl
resource "aws_db_instance" "main" {
  identifier     = "orders-db"
  engine         = "postgres"
  engine_version = "15.4"
  instance_class = "db.r6g.large"

  allocated_storage     = 100
  max_allocated_storage = 500
  storage_type          = "gp3"
  storage_encrypted     = true
  kms_key_id            = aws_kms_key.rds.arn

  db_name  = "orders"
  username = "admin"
  password = random_password.db.result

  # High Availability
  multi_az = true

  # Networking
  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.rds.id]
  publicly_accessible    = false

  # Performance
  parameter_group_name = aws_db_parameter_group.main.name

  # Backup
  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "Mon:04:00-Mon:05:00"

  # Monitoring
  performance_insights_enabled    = true
  monitoring_interval             = 60
  monitoring_role_arn             = aws_iam_role.rds_monitoring.arn
  enabled_cloudwatch_logs_exports = ["postgresql", "upgrade"]

  # Deletion protection
  deletion_protection = true
  skip_final_snapshot = false
  final_snapshot_identifier = "orders-db-final"

  tags = {
    Name = "orders-db"
  }
}

# Parameter group for connection optimization
resource "aws_db_parameter_group" "main" {
  family = "postgres15"
  name   = "orders-db-params"

  # Connection settings
  parameter {
    name  = "max_connections"
    value = "200"
  }

  # Reduce failover detection time
  parameter {
    name  = "tcp_keepalives_idle"
    value = "60"
  }

  parameter {
    name  = "tcp_keepalives_interval"
    value = "10"
  }

  parameter {
    name  = "tcp_keepalives_count"
    value = "3"
  }

  # Performance
  parameter {
    name  = "shared_buffers"
    value = "{DBInstanceClassMemory/4}"
  }

  parameter {
    name  = "work_mem"
    value = "65536"
  }
}
```

### Step 4: Connection Retry Logic

```python

from psycopg2 import OperationalError


from functools import wraps

logger = logging.getLogger(__name__)

def with_retry(max_attempts=3, backoff_factor=2):
    """Decorator for database operations with retry logic."""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None

            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except OperationalError as e:
                    last_exception = e
                    error_msg = str(e).lower()

                    # Retryable errors
                    retryable = any([
                        'connection refused' in error_msg,
                        'connection reset' in error_msg,
                        'connection timed out' in error_msg,
                        'server closed the connection' in error_msg,
                        'ssl connection has been closed' in error_msg,
                    ])

                    if not retryable:
                        raise

                    wait_time = backoff_factor ** attempt
                    logger.warning(
                        f"Database connection error (attempt {attempt + 1}/{max_attempts}). "
                        f"Retrying in {wait_time}s: {e}"
                    )
                    time.sleep(wait_time)

            raise last_exception

        return wrapper
    return decorator

class DatabaseConnection:
    """Connection wrapper with automatic reconnection."""

    def __init__(self, dsn: str):
        self.dsn = dsn
        self._connection = None

    @property
    def connection(self):
        if self._connection is None or self._connection.closed:
            self._connection = self._create_connection()
        return self._connection

    @with_retry(max_attempts=5, backoff_factor=2)
    def _create_connection(self):
        return psycopg2.connect(
            self.dsn,
            connect_timeout=5,
            options='-c statement_timeout=30000'
        )

    @with_retry(max_attempts=3)
    def execute(self, query: str, params=None):
        with self.connection.cursor() as cursor:
            cursor.execute(query, params)
            if cursor.description:
                return cursor.fetchall()
            self.connection.commit()
            return cursor.rowcount

    def close(self):
        if self._connection:
            self._connection.close()
```

### Step 5: Lambda with IAM Authentication

```python


def get_db_token():
    """Generate IAM authentication token for RDS Proxy."""
    client = boto3.client('rds')

    token = client.generate_db_auth_token(
        DBHostname=os.environ['DB_PROXY_ENDPOINT'],
        Port=5432,
        DBUsername=os.environ['DB_USER'],
        Region=os.environ['AWS_REGION']
    )
    return token

# Connection pool that works with RDS Proxy
connection = None

def get_connection():
    """Get or create database connection."""
    global connection

    if connection is None or connection.closed:
        connection = psycopg2.connect(
            host=os.environ['DB_PROXY_ENDPOINT'],
            port=5432,
            database=os.environ['DB_NAME'],
            user=os.environ['DB_USER'],
            password=get_db_token(),
            sslmode='require',
            connect_timeout=5
        )

    return connection

def handler(event, context):
    """Lambda handler with connection reuse."""
    conn = get_connection()

    try:
        with conn.cursor() as cursor:
            cursor.execute(
                "SELECT * FROM orders WHERE id = %s",
                (event['order_id'],)
            )
            result = cursor.fetchone()

        return {
            'statusCode': 200,
            'body': {'order': result}
        }

    except Exception as e:
        # Reset connection on error
        global connection
        if connection:
            connection.close()
            connection = None
        raise
```

### Step 6: Read Replicas for Read Scaling

```hcl
# Read replica
resource "aws_db_instance" "read_replica" {
  identifier          = "orders-db-replica"
  replicate_source_db = aws_db_instance.main.identifier
  instance_class      = "db.r6g.large"

  # Replica-specific settings
  multi_az            = false
  publicly_accessible = false

  # Performance
  parameter_group_name = aws_db_parameter_group.replica.name

  # Monitoring
  performance_insights_enabled = true
  monitoring_interval          = 60
  monitoring_role_arn          = aws_iam_role.rds_monitoring.arn

  tags = {
    Name = "orders-db-replica"
  }
}

# RDS Proxy endpoint for read replicas
resource "aws_db_proxy_endpoint" "read_only" {
  db_proxy_name          = aws_db_proxy.main.name
  db_proxy_endpoint_name = "read-only"
  vpc_subnet_ids         = aws_subnet.database[*].id
  target_role            = "READ_ONLY"
  vpc_security_group_ids = [aws_security_group.rds_proxy.id]
}
```

### Step 7: Monitoring and Alerting

```hcl
# Connection usage alarm
resource "aws_cloudwatch_metric_alarm" "db_connections" {
  alarm_name          = "rds-high-connections"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "DatabaseConnections"
  namespace           = "AWS/RDS"
  period              = 300
  statistic           = "Average"
  threshold           = 150  # 75% of max_connections=200
  alarm_description   = "Database connections are high"
  alarm_actions       = [aws_sns_topic.alerts.arn]

  dimensions = {
    DBInstanceIdentifier = aws_db_instance.main.identifier
  }
}

# Proxy connection alarm
resource "aws_cloudwatch_metric_alarm" "proxy_connections" {
  alarm_name          = "rds-proxy-connections"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "ClientConnections"
  namespace           = "AWS/RDS/Proxy"
  period              = 60
  statistic           = "Sum"
  threshold           = 900  # RDS Proxy limit is 1000
  alarm_description   = "RDS Proxy connections are high"
  alarm_actions       = [aws_sns_topic.alerts.arn]

  dimensions = {
    ProxyName = aws_db_proxy.main.name
  }
}

# Replication lag alarm
resource "aws_cloudwatch_metric_alarm" "replication_lag" {
  alarm_name          = "rds-replication-lag"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 3
  metric_name         = "ReplicaLag"
  namespace           = "AWS/RDS"
  period              = 60
  statistic           = "Maximum"
  threshold           = 30  # 30 seconds
  alarm_description   = "Read replica lag is high"
  alarm_actions       = [aws_sns_topic.alerts.arn]

  dimensions = {
    DBInstanceIdentifier = aws_db_instance.read_replica.identifier
  }
}
```


## RDS High Availability Summary

| Feature | Benefit | Failover Time |
|---------|---------|---------------|
| Multi-AZ | Automatic failover | 60-120 seconds |
| RDS Proxy | Connection pooling | Near-instant |
| Read Replicas | Read scaling | Manual promotion |
| Aurora | Faster failover | Under 30 seconds |


---

### Quick Check

**Why does RDS Proxy significantly reduce failover time compared to direct RDS connections?**

   A. RDS Proxy uses faster network connections
-> B. **RDS Proxy maintains connection state and handles DNS changes transparently, so applications don't need to re-establish connections**
   C. RDS Proxy is deployed in more availability zones
   D. RDS Proxy uses a different failover mechanism than RDS

<details>
<summary>See Answer</summary>

During RDS failover, the DNS endpoint changes to point to the new primary. Direct connections fail because they cache the old IP and must wait for DNS TTL to expire, then establish new connections. RDS Proxy maintains a pool of connections to both the primary and standby, handles the DNS change internally, and multiplexes client connections to the new primary transparently. Applications experience brief query delays instead of connection failures.

</details>

---

### Question 5: Implement S3 with CloudFront for secure, cached content delivery with signed URLs.

**Type:** Practical | **Category:** Storage & CDN

## The Scenario

You need to serve static assets and user-uploaded content:

```
Requirements:
├── Static assets: JS, CSS, images (public)
├── User content: PDFs, documents (private, signed URLs)
├── Global users: Low latency worldwide
├── Security: No direct S3 access, HTTPS only
├── Cost: Optimize data transfer costs
└── Performance: Cache control, compression
```

## The Challenge

Implement a secure, performant content delivery architecture using S3 as origin and CloudFront as CDN, with proper cache policies and access controls.


> **Common Mistake:** A junior engineer might make the S3 bucket public, skip CloudFront, use S3 website hosting for everything, or ignore cache headers. These approaches expose data to unauthorized access, increase latency for global users, result in higher costs, and cause poor cache performance.

> **Senior Engineer Approach:** A senior engineer uses Origin Access Control (OAC) to restrict S3 access, configures appropriate cache policies per content type, implements signed URLs for private content, and uses CloudFront functions for edge processing.

### Step 1: Architecture Overview

```
Content Delivery Architecture:
                                    ┌─────────────────┐
                                    │   CloudFront    │
                                    │   Functions     │
                                    │  (URL rewrite)  │
                                    └────────┬────────┘
                                             │
┌─────────────┐     ┌───────────────────────────────────────┐
│   Client    │────►│           CloudFront Distribution      │
└─────────────┘     │                                        │
                    │  ┌─────────┐  ┌─────────┐  ┌────────┐ │
                    │  │ Cache   │  │ Cache   │  │ Cache  │ │
                    │  │ Behavior│  │ Behavior│  │Behavior│ │
                    │  │ /static │  │ /api/*  │  │  /*    │ │
                    │  └────┬────┘  └────┬────┘  └───┬────┘ │
                    └───────┼────────────┼──────────┼───────┘
                            │            │          │
                            ▼            ▼          ▼
                    ┌───────────┐  ┌─────────┐  ┌───────────┐
                    │  S3 Bucket│  │   ALB   │  │ S3 Bucket │
                    │  (static) │  │  (API)  │  │ (private) │
                    └───────────┘  └─────────┘  └───────────┘
```

### Step 2: S3 Buckets Configuration

```hcl
# Static assets bucket
resource "aws_s3_bucket" "static" {
  bucket = "myapp-static-assets"
}

resource "aws_s3_bucket_versioning" "static" {
  bucket = aws_s3_bucket.static.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "static" {
  bucket = aws_s3_bucket.static.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# Block all public access
resource "aws_s3_bucket_public_access_block" "static" {
  bucket = aws_s3_bucket.static.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# Private content bucket
resource "aws_s3_bucket" "private" {
  bucket = "myapp-private-content"
}

resource "aws_s3_bucket_versioning" "private" {
  bucket = aws_s3_bucket.private.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "private" {
  bucket = aws_s3_bucket.private.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = aws_kms_key.s3.arn
    }
  }
}

resource "aws_s3_bucket_public_access_block" "private" {
  bucket = aws_s3_bucket.private.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# Lifecycle policy for cost optimization
resource "aws_s3_bucket_lifecycle_configuration" "private" {
  bucket = aws_s3_bucket.private.id

  rule {
    id     = "move-to-ia"
    status = "Enabled"

    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }

    transition {
      days          = 90
      storage_class = "GLACIER_IR"
    }

    noncurrent_version_transition {
      noncurrent_days = 30
      storage_class   = "STANDARD_IA"
    }

    noncurrent_version_expiration {
      noncurrent_days = 90
    }
  }
}
```

### Step 3: CloudFront Distribution with OAC

```hcl
# Origin Access Control
resource "aws_cloudfront_origin_access_control" "main" {
  name                              = "s3-oac"
  description                       = "OAC for S3 buckets"
  origin_access_control_origin_type = "s3"
  signing_behavior                  = "always"
  signing_protocol                  = "sigv4"
}

# CloudFront Distribution
resource "aws_cloudfront_distribution" "main" {
  enabled             = true
  is_ipv6_enabled     = true
  comment             = "Main CDN distribution"
  default_root_object = "index.html"
  price_class         = "PriceClass_100"  # US, Canada, Europe
  aliases             = ["cdn.example.com"]

  # Static assets origin
  origin {
    domain_name              = aws_s3_bucket.static.bucket_regional_domain_name
    origin_id                = "S3-static"
    origin_access_control_id = aws_cloudfront_origin_access_control.main.id
  }

  # Private content origin
  origin {
    domain_name              = aws_s3_bucket.private.bucket_regional_domain_name
    origin_id                = "S3-private"
    origin_access_control_id = aws_cloudfront_origin_access_control.main.id
  }

  # API origin
  origin {
    domain_name = aws_lb.api.dns_name
    origin_id   = "ALB-api"

    custom_origin_config {
      http_port              = 80
      https_port             = 443
      origin_protocol_policy = "https-only"
      origin_ssl_protocols   = ["TLSv1.2"]
    }
  }

  # Default behavior (static assets)
  default_cache_behavior {
    allowed_methods        = ["GET", "HEAD", "OPTIONS"]
    cached_methods         = ["GET", "HEAD"]
    target_origin_id       = "S3-static"
    viewer_protocol_policy = "redirect-to-https"
    compress               = true

    cache_policy_id          = aws_cloudfront_cache_policy.static.id
    origin_request_policy_id = aws_cloudfront_origin_request_policy.s3.id

    function_association {
      event_type   = "viewer-request"
      function_arn = aws_cloudfront_function.url_rewrite.arn
    }
  }

  # Private content behavior (signed URLs required)
  ordered_cache_behavior {
    path_pattern           = "/private/*"
    allowed_methods        = ["GET", "HEAD"]
    cached_methods         = ["GET", "HEAD"]
    target_origin_id       = "S3-private"
    viewer_protocol_policy = "https-only"
    compress               = true

    # Require signed URLs
    trusted_key_groups = [aws_cloudfront_key_group.main.id]

    cache_policy_id = aws_cloudfront_cache_policy.private.id
  }

  # API behavior (no caching)
  ordered_cache_behavior {
    path_pattern           = "/api/*"
    allowed_methods        = ["DELETE", "GET", "HEAD", "OPTIONS", "PATCH", "POST", "PUT"]
    cached_methods         = ["GET", "HEAD"]
    target_origin_id       = "ALB-api"
    viewer_protocol_policy = "https-only"
    compress               = true

    cache_policy_id          = aws_cloudfront_cache_policy.api.id
    origin_request_policy_id = aws_cloudfront_origin_request_policy.api.id
  }

  # Custom error responses
  custom_error_response {
    error_code         = 404
    response_code      = 200
    response_page_path = "/index.html"  # SPA fallback
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    acm_certificate_arn      = aws_acm_certificate.cdn.arn
    ssl_support_method       = "sni-only"
    minimum_protocol_version = "TLSv1.2_2021"
  }

  tags = {
    Name = "main-cdn"
  }
}

# S3 bucket policy for CloudFront OAC
resource "aws_s3_bucket_policy" "static" {
  bucket = aws_s3_bucket.static.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid       = "AllowCloudFrontOAC"
        Effect    = "Allow"
        Principal = {
          Service = "cloudfront.amazonaws.com"
        }
        Action   = "s3:GetObject"
        Resource = "${aws_s3_bucket.static.arn}/*"
        Condition = {
          StringEquals = {
            "AWS:SourceArn" = aws_cloudfront_distribution.main.arn
          }
        }
      }
    ]
  })
}
```

### Step 4: Cache Policies

```hcl
# Cache policy for static assets (long TTL)
resource "aws_cloudfront_cache_policy" "static" {
  name        = "static-assets"
  comment     = "Cache policy for static assets"
  default_ttl = 86400     # 1 day
  max_ttl     = 31536000  # 1 year
  min_ttl     = 1

  parameters_in_cache_key_and_forwarded_to_origin {
    cookies_config {
      cookie_behavior = "none"
    }
    headers_config {
      header_behavior = "none"
    }
    query_strings_config {
      query_string_behavior = "none"
    }
    enable_accept_encoding_brotli = true
    enable_accept_encoding_gzip   = true
  }
}

# Cache policy for private content (shorter TTL)
resource "aws_cloudfront_cache_policy" "private" {
  name        = "private-content"
  comment     = "Cache policy for private content"
  default_ttl = 3600    # 1 hour
  max_ttl     = 86400   # 1 day
  min_ttl     = 0

  parameters_in_cache_key_and_forwarded_to_origin {
    cookies_config {
      cookie_behavior = "none"
    }
    headers_config {
      header_behavior = "none"
    }
    query_strings_config {
      query_string_behavior = "none"
    }
    enable_accept_encoding_brotli = true
    enable_accept_encoding_gzip   = true
  }
}

# Cache policy for API (no caching)
resource "aws_cloudfront_cache_policy" "api" {
  name        = "api-no-cache"
  comment     = "No caching for API requests"
  default_ttl = 0
  max_ttl     = 0
  min_ttl     = 0

  parameters_in_cache_key_and_forwarded_to_origin {
    cookies_config {
      cookie_behavior = "all"
    }
    headers_config {
      header_behavior = "whitelist"
      headers {
        items = ["Authorization", "Content-Type"]
      }
    }
    query_strings_config {
      query_string_behavior = "all"
    }
  }
}

# Origin request policy for API
resource "aws_cloudfront_origin_request_policy" "api" {
  name    = "api-origin-request"
  comment = "Forward all headers to API"

  cookies_config {
    cookie_behavior = "all"
  }
  headers_config {
    header_behavior = "allViewer"
  }
  query_strings_config {
    query_string_behavior = "all"
  }
}
```

### Step 5: Signed URLs for Private Content

```python

from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import padding
from cryptography.hazmat.backends import default_backend


class CloudFrontSigner:
    def __init__(self, key_pair_id: str, private_key_pem: str):
        self.key_pair_id = key_pair_id
        self.private_key = serialization.load_pem_private_key(
            private_key_pem.encode(),
            password=None,
            backend=default_backend()
        )

    def create_signed_url(
        self,
        url: str,
        expires_at: datetime.datetime,
        ip_address: str = None
    ) -> str:
        """Generate a signed URL for CloudFront."""
        policy = self._create_policy(url, expires_at, ip_address)
        signature = self._sign(policy)

        # URL-safe base64
        policy_b64 = self._url_safe_b64(policy.encode())
        signature_b64 = self._url_safe_b64(signature)

        separator = '&' if '?' in url else '?'
        return f"{url}{separator}Policy={policy_b64}&Signature={signature_b64}&Key-Pair-Id={self.key_pair_id}"

    def _create_policy(
        self,
        url: str,
        expires_at: datetime.datetime,
        ip_address: str = None
    ) -> str:
        """Create CloudFront policy document."""
        policy = {
            "Statement": [{
                "Resource": url,
                "Condition": {
                    "DateLessThan": {
                        "AWS:EpochTime": int(expires_at.timestamp())
                    }
                }
            }]
        }

        if ip_address:
            policy["Statement"][0]["Condition"]["IpAddress"] = {
                "AWS:SourceIp": ip_address
            }

        return json.dumps(policy, separators=(',', ':'))

    def _sign(self, message: str) -> bytes:
        """Sign message with RSA private key."""
        return self.private_key.sign(
            message.encode(),
            padding.PKCS1v15(),
            None
        )

    def _url_safe_b64(self, data: bytes) -> str:
        """URL-safe base64 encoding."""
        return base64.b64encode(data).decode().replace('+', '-').replace('=', '_').replace('/', '~')

# Usage in Lambda
def get_signed_url(event, context):
    """Generate signed URL for private content."""
    secrets = boto3.client('secretsmanager')

    # Get private key from Secrets Manager
    secret = secrets.get_secret_value(SecretId='cloudfront/private-key')
    private_key = secret['SecretString']

    signer = CloudFrontSigner(
        key_pair_id=os.environ['CLOUDFRONT_KEY_PAIR_ID'],
        private_key_pem=private_key
    )

    # Generate URL valid for 1 hour
    expires_at = datetime.datetime.utcnow() + datetime.timedelta(hours=1)

    signed_url = signer.create_signed_url(
        url=f"https://cdn.example.com/private/{event['file_key']}",
        expires_at=expires_at,
        ip_address=event.get('requestContext', {}).get('identity', {}).get('sourceIp')
    )

    return {
        'statusCode': 200,
        'body': json.dumps({'url': signed_url})
    }
```

### Step 6: CloudFront Functions

```javascript
// URL Rewrite Function (viewer-request)
function handler(event) {
    var request = event.request;
    var uri = request.uri;

    // Add index.html to directory requests
    if (uri.endsWith('/')) {
        request.uri += 'index.html';
    }
    // Handle SPA routing - serve index.html for routes without extensions
    else if (!uri.includes('.')) {
        request.uri = '/index.html';
    }

    return request;
}
```

```hcl
resource "aws_cloudfront_function" "url_rewrite" {
  name    = "url-rewrite"
  runtime = "cloudfront-js-1.0"
  comment = "URL rewrite for SPA"
  publish = true
  code    = file("${path.module}/functions/url-rewrite.js")
}

# Security headers function
resource "aws_cloudfront_function" "security_headers" {
  name    = "security-headers"
  runtime = "cloudfront-js-1.0"
  comment = "Add security headers"
  publish = true
  code    = <<-EOF
    function handler(event) {
        var response = event.response;
        var headers = response.headers;

        headers['strict-transport-security'] = { value: 'max-age=63072000; includeSubdomains; preload' };
        headers['x-content-type-options'] = { value: 'nosniff' };
        headers['x-frame-options'] = { value: 'DENY' };
        headers['x-xss-protection'] = { value: '1; mode=block' };
        headers['referrer-policy'] = { value: 'strict-origin-when-cross-origin' };

        return response;
    }
  EOF
}
```


## CloudFront Best Practices

| Feature | Configuration | Purpose |
|---------|--------------|---------|
| Origin Access Control | Always for S3 | Prevent direct S3 access |
| Cache Policy | Per content type | Optimize cache hit ratio |
| Compression | Enable Brotli + Gzip | Reduce bandwidth costs |
| Price Class | Match user locations | Balance cost/performance |
| Signed URLs | Private content | Secure time-limited access |


---

### Quick Check

**Why should you use Origin Access Control (OAC) instead of Origin Access Identity (OAI) for new CloudFront distributions?**

   A. OAC is cheaper than OAI
-> B. **OAC supports all S3 features including SSE-KMS, while OAI has limitations with server-side encryption**
   C. OAI is deprecated and no longer works
   D. OAC provides faster content delivery

<details>
<summary>See Answer</summary>

Origin Access Control (OAC) is the recommended method for restricting S3 access to CloudFront. Unlike the older Origin Access Identity (OAI), OAC supports all S3 features including SSE-KMS encryption, S3 Object Lambda, and works with S3 buckets in all regions. OAC uses IAM-based authentication with SigV4, providing better security and broader compatibility. AWS recommends migrating from OAI to OAC for all distributions.

</details>

---

### Question 6: ECS tasks are failing with exit code 137 and health check failures. Debug the container issues.

**Type:** Debugging | **Category:** ECS

## The Scenario

Your ECS service is unstable:

```
Current Problems:
├── Tasks constantly restarting
├── Exit code 137 (OOMKilled)
├── Health check failures: 50%
├── Container insights: Memory spikes to 100%
├── Task definition: 512 CPU, 1024 MB memory
├── Application: Node.js API server
└── Error: "Container killed due to memory"
```

## The Challenge

Debug and fix the ECS task failures, optimize container resource allocation, and implement proper health checks and monitoring.


### Step 1: Understand Exit Codes

```
Common ECS Exit Codes:
├── 0: Normal exit (success)
├── 1: Application error
├── 137: SIGKILL (OOMKilled or manual stop)
├── 139: SIGSEGV (Segmentation fault)
├── 143: SIGTERM (Graceful shutdown)
└── 255: Exit status out of range

Exit code 137 = 128 + 9 (SIGKILL)
- Container exceeded memory limit
- Killed by orchestrator
```

### Step 2: Debug with ECS Exec

```bash
# Enable ECS Exec on the service
aws ecs update-service \
  --cluster production \
  --service api-service \
  --enable-execute-command

# Execute into running container
aws ecs execute-command \
  --cluster production \
  --task arn:aws:ecs:us-east-1:123456789:task/production/abc123 \
  --container api \
  --interactive \
  --command "/bin/sh"

# Inside container: Check memory usage
cat /sys/fs/cgroup/memory/memory.usage_in_bytes
cat /sys/fs/cgroup/memory/memory.limit_in_bytes

# Check Node.js heap
node -e "console.log(process.memoryUsage())"

# Check for memory leaks
node --inspect app.js
# Then use Chrome DevTools to analyze heap snapshots
```

### Step 3: Fix Task Definition

```hcl
resource "aws_ecs_task_definition" "api" {
  family                   = "api-task"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = "1024"   # 1 vCPU
  memory                   = "2048"   # 2 GB
  execution_role_arn       = aws_iam_role.ecs_execution.arn
  task_role_arn            = aws_iam_role.ecs_task.arn

  container_definitions = jsonencode([
    {
      name      = "api"
      image     = "${aws_ecr_repository.api.repository_url}:latest"
      essential = true

      # Resource limits
      cpu    = 896  # Leave some for sidecar
      memory = 1792 # Leave headroom for container overhead

      # Memory reservation (soft limit)
      memoryReservation = 1536

      portMappings = [
        {
          containerPort = 3000
          protocol      = "tcp"
        }
      ]

      # Environment for Node.js memory management
      environment = [
        {
          name  = "NODE_OPTIONS"
          value = "--max-old-space-size=1536"  # 75% of memory limit
        },
        {
          name  = "NODE_ENV"
          value = "production"
        }
      ]

      # Health check
      healthCheck = {
        command     = ["CMD-SHELL", "curl -f http://localhost:3000/health || exit 1"]
        interval    = 30
        timeout     = 5
        retries     = 3
        startPeriod = 60  # Give app time to start
      }

      # Logging
      logConfiguration = {
        logDriver = "awslogs"
        options = {
          "awslogs-group"         = "/ecs/api"
          "awslogs-region"        = "us-east-1"
          "awslogs-stream-prefix" = "api"
        }
      }

      # Graceful shutdown
      stopTimeout = 30

      # Linux parameters
      linuxParameters = {
        initProcessEnabled = true  # Enable init process for proper signal handling
      }
    },
    # Sidecar for metrics
    {
      name      = "datadog-agent"
      image     = "datadog/agent:latest"
      essential = false
      cpu       = 128
      memory    = 256

      environment = [
        {
          name  = "DD_API_KEY"
          value = "from-secrets-manager"
        },
        {
          name  = "ECS_FARGATE"
          value = "true"
        }
      ]
    }
  ])
}
```

### Step 4: Fix ALB Health Checks

```hcl
resource "aws_lb_target_group" "api" {
  name        = "api-tg"
  port        = 3000
  protocol    = "HTTP"
  vpc_id      = aws_vpc.main.id
  target_type = "ip"

  health_check {
    enabled             = true
    healthy_threshold   = 2
    unhealthy_threshold = 3
    timeout             = 5
    interval            = 30
    path                = "/health"
    port                = "traffic-port"
    protocol            = "HTTP"
    matcher             = "200"
  }

  # Important for graceful deployments
  deregistration_delay = 30

  stickiness {
    type            = "lb_cookie"
    cookie_duration = 86400
    enabled         = false
  }

  tags = {
    Name = "api-target-group"
  }
}

# ECS Service with proper deployment configuration
resource "aws_ecs_service" "api" {
  name            = "api-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.api.arn
  desired_count   = 3
  launch_type     = "FARGATE"

  # Enable ECS Exec for debugging
  enable_execute_command = true

  network_configuration {
    subnets          = aws_subnet.private[*].id
    security_groups  = [aws_security_group.ecs_tasks.id]
    assign_public_ip = false
  }

  load_balancer {
    target_group_arn = aws_lb_target_group.api.arn
    container_name   = "api"
    container_port   = 3000
  }

  # Deployment configuration
  deployment_configuration {
    maximum_percent         = 200
    minimum_healthy_percent = 100

    deployment_circuit_breaker {
      enable   = true
      rollback = true
    }
  }

  # Service discovery (optional)
  service_registries {
    registry_arn = aws_service_discovery_service.api.arn
  }

  # Ensure new tasks pass health checks before draining old ones
  health_check_grace_period_seconds = 60

  lifecycle {
    ignore_changes = [desired_count]  # Allow auto-scaling to manage
  }
}
```

### Step 5: Implement Proper Health Check Endpoint

```javascript
// health.js - Comprehensive health check
const express = require('express');
const router = express.Router();

// Simple liveness check (for container health)
router.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok' });
});

// Detailed readiness check (for ALB)
router.get('/health/ready', async (req, res) => {
  const checks = {
    database: false,
    cache: false,
    memory: false,
  };

  try {
    // Check database connection
    await db.query('SELECT 1');
    checks.database = true;
  } catch (err) {
    console.error('Database health check failed:', err);
  }

  try {
    // Check Redis connection
    await redis.ping();
    checks.cache = true;
  } catch (err) {
    console.error('Cache health check failed:', err);
  }

  // Check memory usage (fail if over 90%)
  const used = process.memoryUsage();
  const heapUsedPercent = (used.heapUsed / used.heapTotal) * 100;
  checks.memory = heapUsedPercent < 90;

  const allHealthy = Object.values(checks).every(Boolean);

  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? 'healthy' : 'unhealthy',
    checks,
    memory: {
      heapUsed: Math.round(used.heapUsed / 1024 / 1024) + 'MB',
      heapTotal: Math.round(used.heapTotal / 1024 / 1024) + 'MB',
      rss: Math.round(used.rss / 1024 / 1024) + 'MB',
    },
    uptime: process.uptime(),
  });
});

module.exports = router;
```

### Step 6: Graceful Shutdown

```javascript
// server.js - Handle shutdown signals
const express = require('express');
const app = express();

let server;
let isShuttingDown = false;

// Health check returns 503 during shutdown
app.get('/health', (req, res) => {
  if (isShuttingDown) {
    return res.status(503).json({ status: 'shutting down' });
  }
  res.status(200).json({ status: 'ok' });
});

// Start server
server = app.listen(3000, () => {
  console.log('Server started on port 3000');
});

// Graceful shutdown handler
async function gracefulShutdown(signal) {
  console.log(`Received ${signal}, starting graceful shutdown`);
  isShuttingDown = true;

  // Stop accepting new connections
  server.close(async () => {
    console.log('HTTP server closed');

    try {
      // Close database connections
      await db.end();
      console.log('Database connections closed');

      // Close Redis connections
      await redis.quit();
      console.log('Redis connections closed');

      process.exit(0);
    } catch (err) {
      console.error('Error during shutdown:', err);
      process.exit(1);
    }
  });

  // Force shutdown after timeout
  setTimeout(() => {
    console.error('Forced shutdown after timeout');
    process.exit(1);
  }, 25000); // Less than ECS stopTimeout (30s)
}

// Handle termination signals
process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
process.on('SIGINT', () => gracefulShutdown('SIGINT'));
```

### Step 7: Container Insights and Monitoring

```hcl
# Enable Container Insights
resource "aws_ecs_cluster" "main" {
  name = "production"

  setting {
    name  = "containerInsights"
    value = "enabled"
  }

  configuration {
    execute_command_configuration {
      logging = "OVERRIDE"

      log_configuration {
        cloud_watch_log_group_name = aws_cloudwatch_log_group.ecs_exec.name
      }
    }
  }
}

# CloudWatch alarms for ECS
resource "aws_cloudwatch_metric_alarm" "cpu_high" {
  alarm_name          = "ecs-cpu-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CpuUtilized"
  namespace           = "ECS/ContainerInsights"
  period              = 60
  statistic           = "Average"
  threshold           = 80
  alarm_description   = "ECS CPU utilization is high"
  alarm_actions       = [aws_sns_topic.alerts.arn]

  dimensions = {
    ClusterName = aws_ecs_cluster.main.name
    ServiceName = aws_ecs_service.api.name
  }
}

resource "aws_cloudwatch_metric_alarm" "memory_high" {
  alarm_name          = "ecs-memory-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "MemoryUtilized"
  namespace           = "ECS/ContainerInsights"
  period              = 60
  statistic           = "Average"
  threshold           = 80
  alarm_description   = "ECS memory utilization is high"
  alarm_actions       = [aws_sns_topic.alerts.arn]

  dimensions = {
    ClusterName = aws_ecs_cluster.main.name
    ServiceName = aws_ecs_service.api.name
  }
}

resource "aws_cloudwatch_metric_alarm" "task_count" {
  alarm_name          = "ecs-task-count-low"
  comparison_operator = "LessThanThreshold"
  evaluation_periods  = 2
  metric_name         = "RunningTaskCount"
  namespace           = "ECS/ContainerInsights"
  period              = 60
  statistic           = "Average"
  threshold           = 2
  alarm_description   = "ECS running task count is low"
  alarm_actions       = [aws_sns_topic.alerts.arn]

  dimensions = {
    ClusterName = aws_ecs_cluster.main.name
    ServiceName = aws_ecs_service.api.name
  }
}
```

### Step 8: Auto Scaling

```hcl
resource "aws_appautoscaling_target" "ecs" {
  max_capacity       = 10
  min_capacity       = 2
  resource_id        = "service/${aws_ecs_cluster.main.name}/${aws_ecs_service.api.name}"
  scalable_dimension = "ecs:service:DesiredCount"
  service_namespace  = "ecs"
}

# Scale based on CPU
resource "aws_appautoscaling_policy" "cpu" {
  name               = "cpu-scaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.ecs.resource_id
  scalable_dimension = aws_appautoscaling_target.ecs.scalable_dimension
  service_namespace  = aws_appautoscaling_target.ecs.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }
    target_value       = 70.0
    scale_in_cooldown  = 300
    scale_out_cooldown = 60
  }
}

# Scale based on memory
resource "aws_appautoscaling_policy" "memory" {
  name               = "memory-scaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.ecs.resource_id
  scalable_dimension = aws_appautoscaling_target.ecs.scalable_dimension
  service_namespace  = aws_appautoscaling_target.ecs.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageMemoryUtilization"
    }
    target_value       = 70.0
    scale_in_cooldown  = 300
    scale_out_cooldown = 60
  }
}
```


## ECS Debugging Checklist

| Issue | Check | Solution |
|-------|-------|----------|
| Exit code 137 | Memory limit | Increase memory, fix leaks |
| Health check fail | Endpoint, timeout | Increase startPeriod, fix endpoint |
| Image pull error | ECR permissions | Check execution role |
| Network timeout | Security groups | Allow egress to endpoints |
| Slow startup | Container size | Use smaller base images |

---

### Question 7: Messages are being lost and processed multiple times. Implement reliable SQS/SNS messaging.

**Type:** Practical | **Category:** Messaging

## The Scenario

Your messaging system has problems:

```
Current Issues:
├── Messages disappearing after 30 seconds
├── Same message processed 3-4 times
├── Order of messages not preserved
├── Failed messages lost forever
├── Consumer crashes mid-processing
└── No visibility into message flow
```

## The Challenge

Implement a reliable messaging architecture using SQS and SNS with proper visibility timeout, dead letter queues, and idempotent processing.


> **Common Mistake:** A junior engineer might use default SQS settings, skip dead letter queues, process messages without idempotency, or ignore visibility timeout configuration. These approaches cause message loss, duplicate processing, debugging nightmares, and data inconsistency.

> **Senior Engineer Approach:** A senior engineer configures visibility timeout based on processing time, implements dead letter queues for failed messages, makes consumers idempotent, uses message deduplication, and sets up comprehensive monitoring.

### Step 1: Understanding the Message Flow

```
Reliable Messaging Architecture:
┌─────────────┐     ┌─────────────┐     ┌─────────────────────────────────┐
│  Publisher  │────►│    SNS      │────►│           SQS Queues            │
└─────────────┘     │   Topic     │     │  ┌─────────┐    ┌────────────┐  │
                    │             │     │  │ Orders  │    │ Inventory  │  │
                    │ Fan-out to  │────►│  │  Queue  │    │   Queue    │  │
                    │  multiple   │     │  └────┬────┘    └─────┬──────┘  │
                    │subscribers  │     │       │               │         │
                    └─────────────┘     │       ▼               ▼         │
                                        │  ┌─────────┐    ┌────────────┐  │
                                        │  │ Orders  │    │ Inventory  │  │
                                        │  │   DLQ   │    │    DLQ     │  │
                                        │  └─────────┘    └────────────┘  │
                                        └─────────────────────────────────┘
                                                │               │
                                                ▼               ▼
                                        ┌─────────────┐ ┌─────────────────┐
                                        │   Orders    │ │    Inventory    │
                                        │   Lambda    │ │     Lambda      │
                                        └─────────────┘ └─────────────────┘
```

### Step 2: Configure SQS with Dead Letter Queue

```hcl
# Dead Letter Queue
resource "aws_sqs_queue" "orders_dlq" {
  name                      = "orders-dlq"
  message_retention_seconds = 1209600  # 14 days

  tags = {
    Name = "orders-dlq"
  }
}

# Main Queue
resource "aws_sqs_queue" "orders" {
  name                       = "orders-queue"
  visibility_timeout_seconds = 300  # 5 minutes (6x Lambda timeout)
  message_retention_seconds  = 345600  # 4 days
  receive_wait_time_seconds  = 20  # Long polling
  delay_seconds              = 0

  # Dead letter queue configuration
  redrive_policy = jsonencode({
    deadLetterTargetArn = aws_sqs_queue.orders_dlq.arn
    maxReceiveCount     = 3  # Move to DLQ after 3 failures
  })

  # Enable SSE
  sqs_managed_sse_enabled = true

  tags = {
    Name = "orders-queue"
  }
}

# Allow DLQ to receive messages
resource "aws_sqs_queue_redrive_allow_policy" "orders_dlq" {
  queue_url = aws_sqs_queue.orders_dlq.id

  redrive_allow_policy = jsonencode({
    redrivePermission = "byQueue"
    sourceQueueArns   = [aws_sqs_queue.orders.arn]
  })
}

# FIFO Queue for ordered processing
resource "aws_sqs_queue" "orders_fifo" {
  name                        = "orders-queue.fifo"
  fifo_queue                  = true
  content_based_deduplication = false  # We'll provide deduplication IDs
  deduplication_scope         = "messageGroup"
  fifo_throughput_limit       = "perMessageGroupId"
  visibility_timeout_seconds  = 300

  redrive_policy = jsonencode({
    deadLetterTargetArn = aws_sqs_queue.orders_fifo_dlq.arn
    maxReceiveCount     = 3
  })

  tags = {
    Name = "orders-fifo-queue"
  }
}
```

### Step 3: SNS Topic with Fan-out

```hcl
# SNS Topic
resource "aws_sns_topic" "orders" {
  name = "orders-topic"

  # Enable SSE
  kms_master_key_id = "alias/aws/sns"

  tags = {
    Name = "orders-topic"
  }
}

# Subscribe SQS queues to SNS
resource "aws_sns_topic_subscription" "orders_queue" {
  topic_arn = aws_sns_topic.orders.arn
  protocol  = "sqs"
  endpoint  = aws_sqs_queue.orders.arn

  # Filter policy - only receive certain messages
  filter_policy = jsonencode({
    event_type = ["order.created", "order.updated"]
  })

  # Enable raw message delivery (no SNS wrapper)
  raw_message_delivery = true
}

resource "aws_sns_topic_subscription" "inventory_queue" {
  topic_arn = aws_sns_topic.orders.arn
  protocol  = "sqs"
  endpoint  = aws_sqs_queue.inventory.arn

  filter_policy = jsonencode({
    event_type = ["order.created"]
  })

  raw_message_delivery = true
}

# SQS policy to allow SNS to send messages
resource "aws_sqs_queue_policy" "orders" {
  queue_url = aws_sqs_queue.orders.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid       = "AllowSNS"
        Effect    = "Allow"
        Principal = {
          Service = "sns.amazonaws.com"
        }
        Action   = "sqs:SendMessage"
        Resource = aws_sqs_queue.orders.arn
        Condition = {
          ArnEquals = {
            "aws:SourceArn" = aws_sns_topic.orders.arn
          }
        }
      }
    ]
  })
}
```

### Step 4: Publishing Messages

```python


from datetime import datetime

sns = boto3.client('sns')
sqs = boto3.client('sqs')

def publish_order_event(order: dict, event_type: str):
    """Publish order event to SNS with message attributes."""
    message = {
        'order_id': order['id'],
        'customer_id': order['customer_id'],
        'total': order['total'],
        'items': order['items'],
        'timestamp': datetime.utcnow().isoformat(),
    }

    response = sns.publish(
        TopicArn='arn:aws:sns:us-east-1:123456789:orders-topic',
        Message=json.dumps(message),
        MessageAttributes={
            'event_type': {
                'DataType': 'String',
                'StringValue': event_type
            },
            'customer_tier': {
                'DataType': 'String',
                'StringValue': order.get('customer_tier', 'standard')
            }
        },
        # For FIFO topics
        # MessageGroupId=order['customer_id'],
        # MessageDeduplicationId=f"{order['id']}-{event_type}"
    )

    return response['MessageId']

def send_to_fifo_queue(order: dict):
    """Send message to FIFO queue with deduplication."""
    response = sqs.send_message(
        QueueUrl='https://sqs.us-east-1.amazonaws.com/123456789/orders-queue.fifo',
        MessageBody=json.dumps(order),
        MessageGroupId=order['customer_id'],  # Orders for same customer processed in order
        MessageDeduplicationId=f"order-{order['id']}-{order['version']}"  # Prevent duplicates
    )

    return response['MessageId']
```

### Step 5: Idempotent Consumer

```python


from functools import wraps

dynamodb = boto3.resource('dynamodb')
idempotency_table = dynamodb.Table('message-idempotency')

def idempotent(func):
    """Decorator to ensure idempotent message processing."""
    @wraps(func)
    def wrapper(event, context):
        for record in event['Records']:
            message_id = record['messageId']
            receipt_handle = record['receiptHandle']

            # Check if already processed
            try:
                response = idempotency_table.get_item(
                    Key={'message_id': message_id}
                )
                if 'Item' in response:
                    print(f"Message {message_id} already processed, skipping")
                    continue
            except Exception as e:
                print(f"Error checking idempotency: {e}")

            # Process message
            try:
                result = func(record)

                # Mark as processed
                idempotency_table.put_item(
                    Item={
                        'message_id': message_id,
                        'processed_at': datetime.utcnow().isoformat(),
                        'result': str(result),
                        'ttl': int((datetime.utcnow() + timedelta(days=7)).timestamp())
                    },
                    ConditionExpression='attribute_not_exists(message_id)'
                )

            except dynamodb.meta.client.exceptions.ConditionalCheckFailedException:
                print(f"Message {message_id} processed by another consumer")

            except Exception as e:
                print(f"Error processing message: {e}")
                raise  # Let SQS retry

    return wrapper

@idempotent
def process_order(record):
    """Process a single order message."""
    body = json.loads(record['body'])
    order_id = body['order_id']

    # Idempotent operation - use order_id as idempotency key
    result = db.execute("""
        INSERT INTO orders (id, customer_id, total, status)
        VALUES (%s, %s, %s, 'processing')
        ON CONFLICT (id) DO NOTHING
        RETURNING id
    """, (order_id, body['customer_id'], body['total']))

    if result:
        # Order was inserted, process it
        process_order_items(order_id, body['items'])
        update_inventory(body['items'])
        send_confirmation_email(body['customer_id'], order_id)

    return {'order_id': order_id, 'status': 'processed'}
```

### Step 6: Lambda Consumer with Batch Processing

```hcl
resource "aws_lambda_function" "order_processor" {
  function_name = "order-processor"
  runtime       = "python3.11"
  handler       = "handler.process_orders"
  timeout       = 50  # Less than visibility_timeout / 6
  memory_size   = 256

  role = aws_iam_role.lambda_execution.arn

  environment {
    variables = {
      IDEMPOTENCY_TABLE = aws_dynamodb_table.idempotency.name
    }
  }

  dead_letter_config {
    target_arn = aws_sqs_queue.lambda_dlq.arn
  }
}

resource "aws_lambda_event_source_mapping" "sqs_trigger" {
  event_source_arn                   = aws_sqs_queue.orders.arn
  function_name                      = aws_lambda_function.order_processor.arn
  batch_size                         = 10
  maximum_batching_window_in_seconds = 5

  # Partial batch failure reporting
  function_response_types = ["ReportBatchItemFailures"]

  # Scaling configuration
  scaling_config {
    maximum_concurrency = 10
  }
}
```

```python
# Lambda handler with partial batch failure reporting
def process_orders(event, context):
    """Process SQS messages with partial failure reporting."""
    batch_item_failures = []

    for record in event['Records']:
        try:
            body = json.loads(record['body'])
            process_single_order(body)

        except Exception as e:
            print(f"Error processing message {record['messageId']}: {e}")
            batch_item_failures.append({
                'itemIdentifier': record['messageId']
            })

    # Return failed items for retry
    return {
        'batchItemFailures': batch_item_failures
    }
```

### Step 7: DLQ Processing and Monitoring

```python


sqs = boto3.client('sqs')
cloudwatch = boto3.client('cloudwatch')

def process_dlq():
    """Process messages from dead letter queue."""
    dlq_url = 'https://sqs.us-east-1.amazonaws.com/123456789/orders-dlq'

    while True:
        response = sqs.receive_message(
            QueueUrl=dlq_url,
            MaxNumberOfMessages=10,
            WaitTimeSeconds=20,
            AttributeNames=['All'],
            MessageAttributeNames=['All']
        )

        messages = response.get('Messages', [])
        if not messages:
            break

        for message in messages:
            body = json.loads(message['Body'])
            attributes = message.get('Attributes', {})

            print(f"DLQ Message: {message['MessageId']}")
            print(f"  Receive Count: {attributes.get('ApproximateReceiveCount')}")
            print(f"  First Received: {attributes.get('ApproximateFirstReceiveTimestamp')}")
            print(f"  Body: {body}")

            # Analyze and fix the issue, then either:
            # 1. Reprocess manually
            # 2. Redrive to source queue
            # 3. Archive for investigation

            # Delete from DLQ after handling
            sqs.delete_message(
                QueueUrl=dlq_url,
                ReceiptHandle=message['ReceiptHandle']
            )

def redrive_dlq_messages():
    """Redrive messages from DLQ back to source queue."""
    sqs.start_message_move_task(
        SourceArn='arn:aws:sqs:us-east-1:123456789:orders-dlq',
        DestinationArn='arn:aws:sqs:us-east-1:123456789:orders-queue',
        MaxNumberOfMessagesPerSecond=10
    )
```

```hcl
# CloudWatch Alarms
resource "aws_cloudwatch_metric_alarm" "dlq_messages" {
  alarm_name          = "orders-dlq-has-messages"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "ApproximateNumberOfMessagesVisible"
  namespace           = "AWS/SQS"
  period              = 300
  statistic           = "Sum"
  threshold           = 0
  alarm_description   = "Messages in DLQ indicate processing failures"
  alarm_actions       = [aws_sns_topic.alerts.arn]

  dimensions = {
    QueueName = aws_sqs_queue.orders_dlq.name
  }
}

resource "aws_cloudwatch_metric_alarm" "queue_age" {
  alarm_name          = "orders-queue-old-messages"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "ApproximateAgeOfOldestMessage"
  namespace           = "AWS/SQS"
  period              = 300
  statistic           = "Maximum"
  threshold           = 3600  # 1 hour
  alarm_description   = "Messages sitting in queue too long"
  alarm_actions       = [aws_sns_topic.alerts.arn]

  dimensions = {
    QueueName = aws_sqs_queue.orders.name
  }
}
```


## SQS/SNS Best Practices

| Setting | Value | Reason |
|---------|-------|--------|
| Visibility Timeout | 6x processing time | Allow retries without duplicates |
| Long Polling | 20 seconds | Reduce empty responses |
| DLQ Max Receive | 3-5 | Balance retries and alerting |
| Message Retention | 4-14 days | Time to fix issues |
| Batch Size | 10 | Efficient processing |


---

### Quick Check

**Why should visibility timeout be set to at least 6 times the Lambda function timeout?**

   A. AWS requires this ratio for SQS-Lambda integration
-> B. **To account for Lambda cold starts and potential retries, ensuring messages aren't processed multiple times**
   C. To reduce SQS costs by minimizing API calls
   D. Lambda functions run faster with longer visibility timeouts

<details>
<summary>See Answer</summary>

When Lambda processes SQS messages, it may need to retry on failure. If visibility timeout is too short, the message becomes visible again while Lambda is still processing or retrying, causing duplicate processing. The 6x rule accounts for: initial processing attempt, up to 2 retries within Lambda, cold start time, and a safety buffer. For a 50-second Lambda timeout, set visibility timeout to at least 300 seconds (5 minutes).

</details>

---

### Question 8: Design a scalable API Gateway with throttling, caching, and Lambda integration.

**Type:** Architecture | **Category:** API Gateway

## The Scenario

You need to build a production API:

```
Requirements:
├── REST API for mobile and web clients
├── Expected traffic: 10,000 requests/second peak
├── Latency: p99 under 200ms
├── Authentication: API keys + JWT
├── Rate limiting per customer tier
├── Caching for GET requests
├── Backend: Lambda functions
└── Multi-stage: dev, staging, production
```

## The Challenge

Design an API Gateway architecture with proper throttling, caching strategies, authorization, and Lambda integration patterns.


> **Common Mistake:** A junior engineer might create one API for all environments, skip throttling configuration, use proxy integration for everything, or ignore caching. These approaches mix environments, allow abuse, increase Lambda costs, and hurt performance.

> **Senior Engineer Approach:** A senior engineer designs with separate stages, implements usage plans with API keys, configures method-level caching, uses request validation, and chooses the right integration type for each endpoint.

### Step 1: API Gateway Architecture

```
API Gateway Architecture:
┌─────────────────────────────────────────────────────────────────────┐
│                        API Gateway                                   │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    Custom Domain                             │   │
│  │               api.example.com                                │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    Stages                                    │   │
│  │   /prod    /staging    /dev                                  │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              Request Flow                                    │   │
│  │  Request → Auth → Throttle → Validate → Cache → Backend     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    Integrations                              │   │
│  │   Lambda    HTTP Proxy    AWS Services    Mock               │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### Step 2: Create REST API with Terraform

```hcl
# REST API
resource "aws_api_gateway_rest_api" "main" {
  name        = "orders-api"
  description = "Orders REST API"

  endpoint_configuration {
    types = ["REGIONAL"]  # or EDGE for global
  }

  # Binary media types (for file uploads)
  binary_media_types = [
    "image/*",
    "application/pdf"
  ]

  tags = {
    Environment = "production"
  }
}

# API Resources
resource "aws_api_gateway_resource" "orders" {
  rest_api_id = aws_api_gateway_rest_api.main.id
  parent_id   = aws_api_gateway_rest_api.main.root_resource_id
  path_part   = "orders"
}

resource "aws_api_gateway_resource" "order_id" {
  rest_api_id = aws_api_gateway_rest_api.main.id
  parent_id   = aws_api_gateway_resource.orders.id
  path_part   = "{orderId}"
}

# GET /orders - List orders
resource "aws_api_gateway_method" "list_orders" {
  rest_api_id   = aws_api_gateway_rest_api.main.id
  resource_id   = aws_api_gateway_resource.orders.id
  http_method   = "GET"
  authorization = "CUSTOM"
  authorizer_id = aws_api_gateway_authorizer.jwt.id

  # Require API key
  api_key_required = true

  # Request parameters
  request_parameters = {
    "method.request.querystring.limit"  = false
    "method.request.querystring.cursor" = false
  }
}

# Lambda integration
resource "aws_api_gateway_integration" "list_orders" {
  rest_api_id             = aws_api_gateway_rest_api.main.id
  resource_id             = aws_api_gateway_resource.orders.id
  http_method             = aws_api_gateway_method.list_orders.http_method
  integration_http_method = "POST"
  type                    = "AWS_PROXY"
  uri                     = aws_lambda_function.list_orders.invoke_arn

  # Timeout
  timeout_milliseconds = 29000  # Max 29 seconds
}

# POST /orders - Create order
resource "aws_api_gateway_method" "create_order" {
  rest_api_id   = aws_api_gateway_rest_api.main.id
  resource_id   = aws_api_gateway_resource.orders.id
  http_method   = "POST"
  authorization = "CUSTOM"
  authorizer_id = aws_api_gateway_authorizer.jwt.id

  api_key_required = true

  # Request validation
  request_validator_id = aws_api_gateway_request_validator.body.id
  request_models = {
    "application/json" = aws_api_gateway_model.create_order.name
  }
}

# Request validator
resource "aws_api_gateway_request_validator" "body" {
  name                        = "validate-body"
  rest_api_id                 = aws_api_gateway_rest_api.main.id
  validate_request_body       = true
  validate_request_parameters = true
}

# Request model for validation
resource "aws_api_gateway_model" "create_order" {
  rest_api_id  = aws_api_gateway_rest_api.main.id
  name         = "CreateOrderRequest"
  content_type = "application/json"

  schema = jsonencode({
    "$schema" = "http://json-schema.org/draft-04/schema#"
    type      = "object"
    required  = ["customerId", "items"]
    properties = {
      customerId = {
        type      = "string"
        minLength = 1
      }
      items = {
        type     = "array"
        minItems = 1
        items = {
          type     = "object"
          required = ["productId", "quantity"]
          properties = {
            productId = { type = "string" }
            quantity  = { type = "integer", minimum = 1 }
          }
        }
      }
    }
  })
}
```

### Step 3: JWT Authorizer

```hcl
# Lambda authorizer for JWT validation
resource "aws_api_gateway_authorizer" "jwt" {
  name                   = "jwt-authorizer"
  rest_api_id            = aws_api_gateway_rest_api.main.id
  type                   = "TOKEN"
  authorizer_uri         = aws_lambda_function.authorizer.invoke_arn
  authorizer_credentials = aws_iam_role.authorizer.arn

  # Cache authorization results
  authorizer_result_ttl_in_seconds = 300

  identity_source = "method.request.header.Authorization"
}

# Authorizer Lambda
resource "aws_lambda_function" "authorizer" {
  function_name = "api-authorizer"
  runtime       = "python3.11"
  handler       = "authorizer.handler"
  role          = aws_iam_role.authorizer_lambda.arn

  filename         = "authorizer.zip"
  source_code_hash = filebase64sha256("authorizer.zip")

  environment {
    variables = {
      JWT_SECRET_ARN = aws_secretsmanager_secret.jwt_secret.arn
      ISSUER         = "https://auth.example.com"
    }
  }
}
```

```python
# authorizer.py


from functools import lru_cache

secrets = boto3.client('secretsmanager')

@lru_cache(maxsize=1)
def get_jwt_secret():
    response = secrets.get_secret_value(SecretId=os.environ['JWT_SECRET_ARN'])
    return response['SecretString']

def handler(event, context):
    token = event['authorizationToken'].replace('Bearer ', '')

    try:
        payload = jwt.decode(
            token,
            get_jwt_secret(),
            algorithms=['HS256'],
            issuer=os.environ['ISSUER']
        )

        return generate_policy(
            payload['sub'],
            'Allow',
            event['methodArn'],
            context={
                'userId': payload['sub'],
                'email': payload.get('email'),
                'tier': payload.get('tier', 'free')
            }
        )

    except jwt.ExpiredSignatureError:
        raise Exception('Unauthorized')
    except jwt.InvalidTokenError:
        raise Exception('Unauthorized')

def generate_policy(principal_id, effect, resource, context=None):
    policy = {
        'principalId': principal_id,
        'policyDocument': {
            'Version': '2012-10-17',
            'Statement': [{
                'Action': 'execute-api:Invoke',
                'Effect': effect,
                'Resource': resource
            }]
        }
    }

    if context:
        policy['context'] = context

    return policy
```

### Step 4: Usage Plans and Throttling

```hcl
# API Key
resource "aws_api_gateway_api_key" "customer" {
  name    = "customer-api-key"
  enabled = true
}

# Usage plan - Free tier
resource "aws_api_gateway_usage_plan" "free" {
  name = "free-tier"

  api_stages {
    api_id = aws_api_gateway_rest_api.main.id
    stage  = aws_api_gateway_stage.prod.stage_name

    throttle {
      path        = "/orders/GET"
      burst_limit = 10
      rate_limit  = 5
    }
  }

  throttle_settings {
    burst_limit = 50
    rate_limit  = 20
  }

  quota_settings {
    limit  = 10000
    period = "MONTH"
  }
}

# Usage plan - Pro tier
resource "aws_api_gateway_usage_plan" "pro" {
  name = "pro-tier"

  api_stages {
    api_id = aws_api_gateway_rest_api.main.id
    stage  = aws_api_gateway_stage.prod.stage_name
  }

  throttle_settings {
    burst_limit = 500
    rate_limit  = 200
  }

  quota_settings {
    limit  = 1000000
    period = "MONTH"
  }
}

# Usage plan - Enterprise (no quota)
resource "aws_api_gateway_usage_plan" "enterprise" {
  name = "enterprise-tier"

  api_stages {
    api_id = aws_api_gateway_rest_api.main.id
    stage  = aws_api_gateway_stage.prod.stage_name
  }

  throttle_settings {
    burst_limit = 5000
    rate_limit  = 2000
  }
}

# Associate API key with usage plan
resource "aws_api_gateway_usage_plan_key" "customer" {
  key_id        = aws_api_gateway_api_key.customer.id
  key_type      = "API_KEY"
  usage_plan_id = aws_api_gateway_usage_plan.pro.id
}
```

### Step 5: Caching Configuration

```hcl
# Stage with caching
resource "aws_api_gateway_stage" "prod" {
  stage_name    = "prod"
  rest_api_id   = aws_api_gateway_rest_api.main.id
  deployment_id = aws_api_gateway_deployment.main.id

  cache_cluster_enabled = true
  cache_cluster_size    = "0.5"  # GB

  variables = {
    environment = "production"
  }

  access_log_settings {
    destination_arn = aws_cloudwatch_log_group.api_access.arn
    format = jsonencode({
      requestId          = "$context.requestId"
      ip                 = "$context.identity.sourceIp"
      requestTime        = "$context.requestTime"
      httpMethod         = "$context.httpMethod"
      path               = "$context.path"
      status             = "$context.status"
      latency            = "$context.responseLatency"
      integrationLatency = "$context.integrationLatency"
    })
  }
}

# Method settings for caching
resource "aws_api_gateway_method_settings" "list_orders" {
  rest_api_id = aws_api_gateway_rest_api.main.id
  stage_name  = aws_api_gateway_stage.prod.stage_name
  method_path = "orders/GET"

  settings {
    caching_enabled      = true
    cache_ttl_in_seconds = 60

    logging_level      = "INFO"
    data_trace_enabled = false
    metrics_enabled    = true

    throttling_burst_limit = 500
    throttling_rate_limit  = 200
  }
}

# Disable caching for mutations
resource "aws_api_gateway_method_settings" "create_order" {
  rest_api_id = aws_api_gateway_rest_api.main.id
  stage_name  = aws_api_gateway_stage.prod.stage_name
  method_path = "orders/POST"

  settings {
    caching_enabled = false
    logging_level   = "INFO"
    metrics_enabled = true
  }
}
```

### Step 6: Custom Domain and WAF

```hcl
# Custom domain
resource "aws_api_gateway_domain_name" "main" {
  domain_name              = "api.example.com"
  regional_certificate_arn = aws_acm_certificate.api.arn

  endpoint_configuration {
    types = ["REGIONAL"]
  }
}

# Base path mapping
resource "aws_api_gateway_base_path_mapping" "main" {
  api_id      = aws_api_gateway_rest_api.main.id
  stage_name  = aws_api_gateway_stage.prod.stage_name
  domain_name = aws_api_gateway_domain_name.main.domain_name
  base_path   = "v1"
}

# WAF Web ACL
resource "aws_wafv2_web_acl" "api" {
  name  = "api-gateway-waf"
  scope = "REGIONAL"

  default_action {
    allow {}
  }

  # Rate limiting rule
  rule {
    name     = "RateLimitRule"
    priority = 1

    override_action {
      none {}
    }

    statement {
      rate_based_statement {
        limit              = 10000
        aggregate_key_type = "IP"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "RateLimitRule"
      sampled_requests_enabled   = true
    }
  }

  # AWS managed rules
  rule {
    name     = "AWSManagedRulesCommonRuleSet"
    priority = 2

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesCommonRuleSet"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "CommonRuleSet"
      sampled_requests_enabled   = true
    }
  }

  # SQL injection protection
  rule {
    name     = "AWSManagedRulesSQLiRuleSet"
    priority = 3

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesSQLiRuleSet"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "SQLiRuleSet"
      sampled_requests_enabled   = true
    }
  }

  visibility_config {
    cloudwatch_metrics_enabled = true
    metric_name                = "ApiGatewayWAF"
    sampled_requests_enabled   = true
  }
}

# Associate WAF with API Gateway
resource "aws_wafv2_web_acl_association" "api" {
  resource_arn = aws_api_gateway_stage.prod.arn
  web_acl_arn  = aws_wafv2_web_acl.api.arn
}
```

### REST API vs HTTP API

```
REST API (API Gateway):
├── Full features (caching, WAF, request validation)
├── Usage plans and API keys
├── Request/response transformation
├── Private integrations
├── Higher latency (~15ms overhead)
└── Higher cost

HTTP API:
├── Simpler, faster (~5ms overhead)
├── JWT authorizer built-in
├── OIDC and OAuth 2.0 support
├── Auto-deploy
├── 70% cheaper than REST API
└── Missing: caching, request validation, WAF (use CloudFront)
```


## API Gateway Best Practices

| Feature | Configuration | Purpose |
|---------|--------------|---------|
| Caching | 60s for GET, disabled for mutations | Reduce backend calls |
| Throttling | Per method based on tier | Prevent abuse |
| Validation | Request body and parameters | Fail fast, protect backend |
| Logging | Access + execution logs | Debugging and compliance |
| WAF | Rate limit + managed rules | Security |


---

### Quick Check

**Why should you use request validation in API Gateway instead of validating in Lambda?**

   A. API Gateway validation is faster than Lambda
   B. Lambda cannot validate JSON schemas
-> C. **Invalid requests are rejected before invoking Lambda, saving costs and reducing attack surface**
   D. API Gateway validation is more accurate

<details>
<summary>See Answer</summary>

Every Lambda invocation costs money (per invocation + duration). If you validate in Lambda, you pay for invocations that immediately fail validation. With API Gateway validation, invalid requests receive 400 errors before reaching Lambda - no invocation, no cost. This also reduces your attack surface since malformed requests never reach your code. For high-volume APIs, this can save significant money and improve security.

</details>

---

### Question 9: Production incidents take hours to detect. Implement CloudWatch alarms and dashboards.

**Type:** Practical | **Category:** Observability

## The Scenario

Your monitoring is inadequate:

```
Current State:
├── CloudWatch agent: Not installed
├── Alarms: Only EC2 CPU above 80%
├── Dashboards: None
├── Log retention: Default (never expire, high costs)
├── MTTD (Mean Time to Detect): 2-4 hours
├── MTTR (Mean Time to Recover): 4-6 hours
└── Last incident: Customers reported before team noticed
```

## The Challenge

Implement comprehensive monitoring with CloudWatch metrics, custom alarms, centralized logging, and actionable dashboards.


> **Common Mistake:** A junior engineer might create alarms for every metric, use static thresholds that cause alert fatigue, skip log aggregation, or create dashboards with too many widgets. These approaches cause noise, miss real issues, make debugging difficult, and provide no actionable insights.

> **Senior Engineer Approach:** A senior engineer implements composite alarms for meaningful alerts, uses anomaly detection for dynamic thresholds, centralizes logs with proper retention, creates focused dashboards, and implements metric filters for business KPIs.

### Step 1: CloudWatch Alarm Strategy

```hcl
# SNS Topic for alerts
resource "aws_sns_topic" "alerts" {
  name = "production-alerts"
}

resource "aws_sns_topic_subscription" "pagerduty" {
  topic_arn = aws_sns_topic.alerts.arn
  protocol  = "https"
  endpoint  = "https://events.pagerduty.com/integration/xxx/enqueue"
}

# High error rate alarm
resource "aws_cloudwatch_metric_alarm" "api_error_rate" {
  alarm_name          = "api-high-error-rate"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  threshold           = 5

  metric_query {
    id          = "error_rate"
    expression  = "(errors/requests)*100"
    label       = "Error Rate %"
    return_data = true
  }

  metric_query {
    id = "errors"
    metric {
      metric_name = "5XXError"
      namespace   = "AWS/ApiGateway"
      period      = 300
      stat        = "Sum"
      dimensions = {
        ApiName = "orders-api"
        Stage   = "prod"
      }
    }
  }

  metric_query {
    id = "requests"
    metric {
      metric_name = "Count"
      namespace   = "AWS/ApiGateway"
      period      = 300
      stat        = "Sum"
      dimensions = {
        ApiName = "orders-api"
        Stage   = "prod"
      }
    }
  }

  alarm_description = "API error rate exceeds 5%"
  alarm_actions     = [aws_sns_topic.alerts.arn]
  ok_actions        = [aws_sns_topic.alerts.arn]

  treat_missing_data = "notBreaching"
}

# Anomaly detection alarm for latency
resource "aws_cloudwatch_metric_alarm" "latency_anomaly" {
  alarm_name          = "api-latency-anomaly"
  comparison_operator = "GreaterThanUpperThreshold"
  evaluation_periods  = 3
  threshold_metric_id = "anomaly_band"

  metric_query {
    id          = "latency"
    return_data = true
    metric {
      metric_name = "Latency"
      namespace   = "AWS/ApiGateway"
      period      = 300
      stat        = "p99"
      dimensions = {
        ApiName = "orders-api"
        Stage   = "prod"
      }
    }
  }

  metric_query {
    id          = "anomaly_band"
    expression  = "ANOMALY_DETECTION_BAND(latency, 2)"
    label       = "Latency Anomaly Band"
    return_data = true
  }

  alarm_description = "API latency is abnormally high"
  alarm_actions     = [aws_sns_topic.alerts.arn]
}

# Composite alarm for service health
resource "aws_cloudwatch_composite_alarm" "service_unhealthy" {
  alarm_name = "service-unhealthy"

  alarm_rule = join(" OR ", [
    "ALARM(${aws_cloudwatch_metric_alarm.api_error_rate.alarm_name})",
    "ALARM(${aws_cloudwatch_metric_alarm.latency_anomaly.alarm_name})",
    "ALARM(${aws_cloudwatch_metric_alarm.lambda_errors.alarm_name})"
  ])

  alarm_description = "Service is unhealthy - multiple issues detected"
  alarm_actions     = [aws_sns_topic.alerts.arn]
}
```

### Step 2: Custom Metrics

```python

from datetime import datetime

cloudwatch = boto3.client('cloudwatch')

def put_business_metrics(order_data: dict):
    """Publish custom business metrics."""
    cloudwatch.put_metric_data(
        Namespace='OrdersApp',
        MetricData=[
            {
                'MetricName': 'OrderValue',
                'Value': order_data['total_amount'],
                'Unit': 'None',
                'Dimensions': [
                    {'Name': 'Environment', 'Value': 'production'},
                    {'Name': 'Region', 'Value': order_data['region']}
                ],
                'Timestamp': datetime.utcnow()
            },
            {
                'MetricName': 'OrderCount',
                'Value': 1,
                'Unit': 'Count',
                'Dimensions': [
                    {'Name': 'Environment', 'Value': 'production'},
                    {'Name': 'ProductCategory', 'Value': order_data['category']}
                ]
            },
            {
                'MetricName': 'ProcessingTime',
                'Value': order_data['processing_time_ms'],
                'Unit': 'Milliseconds',
                'Dimensions': [
                    {'Name': 'Environment', 'Value': 'production'}
                ]
            }
        ]
    )

# Using EMF (Embedded Metric Format) for Lambda


def emit_emf_metric(metric_name: str, value: float, dimensions: dict):
    """Emit metric using Embedded Metric Format."""
    emf_log = {
        "_aws": {
            "Timestamp": int(datetime.utcnow().timestamp() * 1000),
            "CloudWatchMetrics": [{
                "Namespace": "OrdersApp",
                "Dimensions": [list(dimensions.keys())],
                "Metrics": [{"Name": metric_name, "Unit": "None"}]
            }]
        },
        metric_name: value,
        **dimensions
    }
    print(json.dumps(emf_log))
```

### Step 3: Log Aggregation

```hcl
# Centralized log group with retention
resource "aws_cloudwatch_log_group" "app_logs" {
  name              = "/app/orders-service"
  retention_in_days = 30

  tags = {
    Environment = "production"
    Service     = "orders"
  }
}

# Metric filter for errors
resource "aws_cloudwatch_log_metric_filter" "errors" {
  name           = "error-count"
  pattern        = "[timestamp, level=ERROR, ...]"
  log_group_name = aws_cloudwatch_log_group.app_logs.name

  metric_transformation {
    name          = "ErrorCount"
    namespace     = "OrdersApp/Logs"
    value         = "1"
    default_value = "0"
  }
}

# Metric filter for specific errors
resource "aws_cloudwatch_log_metric_filter" "payment_failures" {
  name           = "payment-failures"
  pattern        = "{ $.error_type = \"PaymentFailed\" }"
  log_group_name = aws_cloudwatch_log_group.app_logs.name

  metric_transformation {
    name      = "PaymentFailures"
    namespace = "OrdersApp/Business"
    value     = "1"
    dimensions = {
      ErrorCode = "$.error_code"
    }
  }
}

# Subscription filter to stream logs
resource "aws_cloudwatch_log_subscription_filter" "to_elasticsearch" {
  name            = "logs-to-elasticsearch"
  log_group_name  = aws_cloudwatch_log_group.app_logs.name
  filter_pattern  = ""
  destination_arn = aws_lambda_function.log_shipper.arn
}
```

### Step 4: CloudWatch Dashboard

```hcl
resource "aws_cloudwatch_dashboard" "main" {
  dashboard_name = "orders-service-prod"

  dashboard_body = jsonencode({
    widgets = [
      # Row 1: Key Metrics
      {
        type   = "metric"
        x      = 0
        y      = 0
        width  = 6
        height = 6
        properties = {
          title  = "Request Count"
          region = "us-east-1"
          metrics = [
            ["AWS/ApiGateway", "Count", "ApiName", "orders-api", "Stage", "prod",
             { stat = "Sum", period = 60 }]
          ]
          view = "timeSeries"
        }
      },
      {
        type   = "metric"
        x      = 6
        y      = 0
        width  = 6
        height = 6
        properties = {
          title  = "Error Rate (%)"
          region = "us-east-1"
          metrics = [
            [{ expression = "(m1/m2)*100", label = "Error Rate", id = "e1" }],
            ["AWS/ApiGateway", "5XXError", "ApiName", "orders-api", "Stage", "prod",
             { stat = "Sum", period = 60, id = "m1", visible = false }],
            ["AWS/ApiGateway", "Count", "ApiName", "orders-api", "Stage", "prod",
             { stat = "Sum", period = 60, id = "m2", visible = false }]
          ]
          view = "timeSeries"
          annotations = {
            horizontal = [
              { label = "Critical", value = 5, color = "#ff0000" }
            ]
          }
        }
      },
      {
        type   = "metric"
        x      = 12
        y      = 0
        width  = 6
        height = 6
        properties = {
          title  = "Latency (p50, p90, p99)"
          region = "us-east-1"
          metrics = [
            ["AWS/ApiGateway", "Latency", "ApiName", "orders-api", "Stage", "prod",
             { stat = "p50", period = 60, label = "p50" }],
            ["...", { stat = "p90", period = 60, label = "p90" }],
            ["...", { stat = "p99", period = 60, label = "p99" }]
          ]
          view = "timeSeries"
        }
      },
      {
        type   = "metric"
        x      = 18
        y      = 0
        width  = 6
        height = 6
        properties = {
          title  = "Active Alarms"
          region = "us-east-1"
          alarms = [
            aws_cloudwatch_metric_alarm.api_error_rate.arn,
            aws_cloudwatch_metric_alarm.latency_anomaly.arn
          ]
        }
      },

      # Row 2: Lambda Metrics
      {
        type   = "metric"
        x      = 0
        y      = 6
        width  = 8
        height = 6
        properties = {
          title  = "Lambda Invocations & Errors"
          region = "us-east-1"
          metrics = [
            ["AWS/Lambda", "Invocations", "FunctionName", "process-order",
             { stat = "Sum", period = 60 }],
            ["AWS/Lambda", "Errors", "FunctionName", "process-order",
             { stat = "Sum", period = 60, color = "#ff0000" }]
          ]
          view = "timeSeries"
        }
      },
      {
        type   = "metric"
        x      = 8
        y      = 6
        width  = 8
        height = 6
        properties = {
          title  = "Lambda Duration"
          region = "us-east-1"
          metrics = [
            ["AWS/Lambda", "Duration", "FunctionName", "process-order",
             { stat = "Average", period = 60 }],
            ["...", { stat = "Maximum", period = 60 }]
          ]
          view = "timeSeries"
        }
      },
      {
        type   = "metric"
        x      = 16
        y      = 6
        width  = 8
        height = 6
        properties = {
          title  = "Lambda Concurrent Executions"
          region = "us-east-1"
          metrics = [
            ["AWS/Lambda", "ConcurrentExecutions", "FunctionName", "process-order",
             { stat = "Maximum", period = 60 }]
          ]
          view = "timeSeries"
        }
      },

      # Row 3: Business Metrics
      {
        type   = "metric"
        x      = 0
        y      = 12
        width  = 12
        height = 6
        properties = {
          title  = "Order Value (Hourly)"
          region = "us-east-1"
          metrics = [
            ["OrdersApp", "OrderValue", "Environment", "production",
             { stat = "Sum", period = 3600 }]
          ]
          view = "timeSeries"
        }
      },
      {
        type   = "metric"
        x      = 12
        y      = 12
        width  = 12
        height = 6
        properties = {
          title  = "Orders by Category"
          region = "us-east-1"
          metrics = [
            ["OrdersApp", "OrderCount", "ProductCategory", "electronics",
             { stat = "Sum", period = 3600 }],
            ["...", "clothing", { stat = "Sum", period = 3600 }],
            ["...", "books", { stat = "Sum", period = 3600 }]
          ]
          view = "timeSeries"
        }
      },

      # Row 4: Logs
      {
        type   = "log"
        x      = 0
        y      = 18
        width  = 24
        height = 6
        properties = {
          title  = "Recent Errors"
          region = "us-east-1"
          query  = <<-EOT
            SOURCE '/app/orders-service'
            | filter level = 'ERROR'
            | sort @timestamp desc
            | limit 100
          EOT
        }
      }
    ]
  })
}
```

### Step 5: CloudWatch Logs Insights Queries

```sql
-- Find slow requests
fields @timestamp, @message
| filter @message like /duration/
| parse @message "duration: * ms" as duration
| filter duration > 1000
| sort duration desc
| limit 100

-- Error breakdown by type
fields @timestamp, error_type, error_message
| filter level = 'ERROR'
| stats count(*) as count by error_type
| sort count desc

-- Request volume by endpoint
fields @timestamp, path, method
| filter @message like /request/
| stats count(*) as requests by path, method
| sort requests desc

-- Find cold starts
fields @timestamp, @message
| filter @message like /Init Duration/
| parse @message "Init Duration: * ms" as init_duration
| stats avg(init_duration) as avg_init, max(init_duration) as max_init by bin(1h)

-- Trace request through logs
fields @timestamp, @message, request_id
| filter request_id = 'abc-123'
| sort @timestamp asc
```

### Step 6: X-Ray Tracing

```hcl
# Enable X-Ray for Lambda
resource "aws_lambda_function" "process_order" {
  # ... other config

  tracing_config {
    mode = "Active"
  }
}

# X-Ray sampling rules
resource "aws_xray_sampling_rule" "main" {
  rule_name      = "orders-sampling"
  priority       = 1000
  reservoir_size = 5
  fixed_rate     = 0.05  # 5% of requests
  url_path       = "*"
  host           = "*"
  http_method    = "*"
  service_type   = "*"
  service_name   = "orders-service"
  version        = 1
}
```


## CloudWatch Best Practices

| Component | Recommendation | Purpose |
|-----------|---------------|---------|
| Alarms | Composite with anomaly detection | Reduce false positives |
| Logs | 30-90 day retention | Balance cost and debugging |
| Dashboards | 4-6 key metrics | Quick health assessment |
| Metrics | Use EMF in Lambda | Lower cost, simpler code |
| X-Ray | 5% sampling | Tracing without overhead |

<InterviewQuiz
  question="Why should you use anomaly detection alarms instead of static threshold alarms?"
  options={[
    "Anomaly detection is cheaper than static thresholds",
    "Static thresholds cannot monitor Lambda functions",
    "Anomaly detection learns normal patterns and adjusts thresholds automatically, reducing false positives from expected traffic variations",
    "AWS requires anomaly detection for production workloads"
  ]}
  correctAnswer={2}
  explanation="Static thresholds like 'latency above 500ms' don't account for normal variations. During peak hours, 400ms might be normal; at night, even 200ms might indicate a problem. Anomaly detection uses ML to learn your metric patterns (including daily/weekly seasonality) and creates dynamic upper/lower bounds. It alerts only when metrics deviate significantly from learned behavior, dramatically reducing alert fatigue while catching real issues."
/>

---

### Question 10: IAM policies are too permissive. Implement least privilege access with proper role design.

**Type:** Practical | **Category:** IAM & Security

## The Scenario

Your AWS security audit revealed issues:

```
Audit Findings:
├── 5 IAM users with AdministratorAccess
├── Lambda execution role: AmazonDynamoDBFullAccess
├── EC2 instances: Full S3 access (s3:*)
├── No MFA enforcement
├── Access keys older than 90 days: 12
├── Unused IAM roles: 23
└── Cross-account access: Unclear trust policies
```

## The Challenge

Implement least privilege IAM policies, proper role design, and security best practices to pass compliance audits.


> **Common Mistake:** A junior engineer might use AWS managed policies for everything, grant full service access, skip conditions, or create one role for all resources. These approaches violate least privilege, increase blast radius, miss security controls, and make auditing difficult.

> **Senior Engineer Approach:** A senior engineer creates resource-specific policies with conditions, uses IAM Access Analyzer to find unused permissions, implements permission boundaries, enforces MFA, and designs roles following the separation of duties principle.

### Step 1: Analyze Current Permissions

```bash
# Find overly permissive policies
aws iam list-policies --scope Local --query 'Policies[*].[PolicyName,Arn]'

# Check policy for wildcards
aws iam get-policy-version \
  --policy-arn arn:aws:iam::123456789:policy/MyPolicy \
  --version-id v1 \
  --query 'PolicyVersion.Document'

# Find unused roles
aws iam list-roles --query 'Roles[?contains(RoleName, `unused`)]'

# Use IAM Access Analyzer
aws accessanalyzer create-analyzer \
  --analyzer-name account-analyzer \
  --type ACCOUNT

# Generate policy based on CloudTrail
aws accessanalyzer generate-policy \
  --policy-generation-details principalArn=arn:aws:iam::123456789:role/LambdaRole
```

### Step 2: Lambda Execution Role (Least Privilege)

```hcl
# BEFORE: Overly permissive
resource "aws_iam_role_policy_attachment" "bad_example" {
  role       = aws_iam_role.lambda.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess"  # Too broad!
}

# AFTER: Least privilege
resource "aws_iam_role" "lambda_orders" {
  name = "orders-lambda-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "lambda.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy" "lambda_orders" {
  name = "orders-lambda-policy"
  role = aws_iam_role.lambda_orders.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      # CloudWatch Logs - only for this function
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Resource = "arn:aws:logs:us-east-1:${data.aws_caller_identity.current.account_id}:log-group:/aws/lambda/process-orders:*"
      },
      # DynamoDB - specific table, specific actions
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:PutItem",
          "dynamodb:UpdateItem",
          "dynamodb:Query"
        ]
        Resource = [
          aws_dynamodb_table.orders.arn,
          "${aws_dynamodb_table.orders.arn}/index/*"
        ]
        # Condition: Only allow from VPC
        Condition = {
          StringEquals = {
            "aws:SourceVpc" = aws_vpc.main.id
          }
        }
      },
      # SQS - specific queue, limited actions
      {
        Effect = "Allow"
        Action = [
          "sqs:ReceiveMessage",
          "sqs:DeleteMessage",
          "sqs:GetQueueAttributes"
        ]
        Resource = aws_sqs_queue.orders.arn
      },
      # Secrets Manager - specific secret
      {
        Effect = "Allow"
        Action = "secretsmanager:GetSecretValue"
        Resource = aws_secretsmanager_secret.db_password.arn
      },
      # KMS - for decryption
      {
        Effect = "Allow"
        Action = "kms:Decrypt"
        Resource = aws_kms_key.app.arn
        Condition = {
          StringEquals = {
            "kms:EncryptionContext:SecretARN" = aws_secretsmanager_secret.db_password.arn
          }
        }
      }
    ]
  })
}
```

### Step 3: EC2 Instance Role

```hcl
resource "aws_iam_role" "ec2_app" {
  name = "app-server-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy" "ec2_app" {
  name = "app-server-policy"
  role = aws_iam_role.ec2_app.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      # S3 - specific bucket, specific prefix
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:PutObject"
        ]
        Resource = "arn:aws:s3:::${aws_s3_bucket.app_data.id}/uploads/*"
      },
      {
        Effect = "Allow"
        Action = "s3:ListBucket"
        Resource = aws_s3_bucket.app_data.arn
        Condition = {
          StringLike = {
            "s3:prefix" = ["uploads/*"]
          }
        }
      },
      # Parameter Store - specific path
      {
        Effect = "Allow"
        Action = [
          "ssm:GetParameter",
          "ssm:GetParameters",
          "ssm:GetParametersByPath"
        ]
        Resource = "arn:aws:ssm:us-east-1:${data.aws_caller_identity.current.account_id}:parameter/app/prod/*"
      },
      # CloudWatch - scoped to application
      {
        Effect = "Allow"
        Action = [
          "cloudwatch:PutMetricData"
        ]
        Resource = "*"
        Condition = {
          StringEquals = {
            "cloudwatch:namespace" = "OrdersApp"
          }
        }
      }
    ]
  })
}

# Instance profile
resource "aws_iam_instance_profile" "ec2_app" {
  name = "app-server-profile"
  role = aws_iam_role.ec2_app.name
}
```

### Step 4: Permission Boundaries

```hcl
# Permission boundary for all developer-created roles
resource "aws_iam_policy" "developer_boundary" {
  name = "DeveloperPermissionBoundary"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      # Allow: Common development resources
      {
        Effect = "Allow"
        Action = [
          "lambda:*",
          "dynamodb:*",
          "sqs:*",
          "sns:*",
          "s3:*",
          "logs:*"
        ]
        Resource = "*"
        Condition = {
          StringEquals = {
            "aws:RequestedRegion" = ["us-east-1", "us-west-2"]
          }
        }
      },
      # Deny: Security-sensitive actions
      {
        Effect   = "Deny"
        Action   = [
          "iam:*",
          "organizations:*",
          "account:*"
        ]
        Resource = "*"
      },
      # Deny: Production resources
      {
        Effect = "Deny"
        Action = "*"
        Resource = "*"
        Condition = {
          StringEquals = {
            "aws:ResourceTag/Environment" = "production"
          }
        }
      },
      # Deny: Creating IAM users (force roles)
      {
        Effect   = "Deny"
        Action   = [
          "iam:CreateUser",
          "iam:CreateAccessKey"
        ]
        Resource = "*"
      }
    ]
  })
}

# Apply boundary to developer role
resource "aws_iam_role" "developer" {
  name = "developer-role"

  permissions_boundary = aws_iam_policy.developer_boundary.arn

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        AWS = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:root"
      }
      Condition = {
        Bool = {
          "aws:MultiFactorAuthPresent" = "true"
        }
      }
    }]
  })
}
```

### Step 5: Cross-Account Access

```hcl
# Role in Account B that Account A can assume
resource "aws_iam_role" "cross_account" {
  name = "cross-account-deploy-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        AWS = "arn:aws:iam::111111111111:role/cicd-role"
      }
      Action = "sts:AssumeRole"
      Condition = {
        StringEquals = {
          "sts:ExternalId" = "unique-external-id-12345"
        }
      }
    }]
  })
}

# Policy for what Account A can do in Account B
resource "aws_iam_role_policy" "cross_account" {
  name = "cross-account-deploy-policy"
  role = aws_iam_role.cross_account.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "lambda:UpdateFunctionCode",
          "lambda:UpdateFunctionConfiguration"
        ]
        Resource = "arn:aws:lambda:us-east-1:222222222222:function:*"
      },
      {
        Effect = "Allow"
        Action = [
          "s3:PutObject"
        ]
        Resource = "arn:aws:s3:::deploy-artifacts-222222222222/*"
      }
    ]
  })
}
```

### Step 6: Service Control Policies (Organizations)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "RequireIMDSv2",
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringNotEquals": {
          "ec2:MetadataHttpTokens": "required"
        }
      }
    },
    {
      "Sid": "DenyLeaveOrganization",
      "Effect": "Deny",
      "Action": "organizations:LeaveOrganization",
      "Resource": "*"
    },
    {
      "Sid": "RequireEncryption",
      "Effect": "Deny",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": "*",
      "Condition": {
        "Null": {
          "s3:x-amz-server-side-encryption": "true"
        }
      }
    },
    {
      "Sid": "DenyRootUser",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringLike": {
          "aws:PrincipalArn": "arn:aws:iam::*:root"
        }
      }
    },
    {
      "Sid": "RestrictRegions",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "us-east-1",
            "us-west-2"
          ]
        },
        "ForAnyValue:StringNotLike": {
          "aws:PrincipalArn": [
            "arn:aws:iam::*:role/OrganizationAccountAccessRole"
          ]
        }
      }
    }
  ]
}
```

### Step 7: Enforce MFA

```hcl
# Policy requiring MFA for sensitive actions
resource "aws_iam_policy" "require_mfa" {
  name = "RequireMFAForSensitiveActions"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      # Allow basic read actions without MFA
      {
        Effect = "Allow"
        Action = [
          "iam:GetAccountPasswordPolicy",
          "iam:GetAccountSummary",
          "iam:ListMFADevices"
        ]
        Resource = "*"
      },
      # Allow managing own MFA
      {
        Effect = "Allow"
        Action = [
          "iam:CreateVirtualMFADevice",
          "iam:EnableMFADevice",
          "iam:ResyncMFADevice"
        ]
        Resource = [
          "arn:aws:iam::*:mfa/$${aws:username}",
          "arn:aws:iam::*:user/$${aws:username}"
        ]
      },
      # Deny everything else without MFA
      {
        Effect = "Deny"
        NotAction = [
          "iam:CreateVirtualMFADevice",
          "iam:EnableMFADevice",
          "iam:GetAccountPasswordPolicy",
          "iam:GetAccountSummary",
          "iam:ListMFADevices",
          "iam:ResyncMFADevice",
          "sts:GetSessionToken"
        ]
        Resource = "*"
        Condition = {
          BoolIfExists = {
            "aws:MultiFactorAuthPresent" = "false"
          }
        }
      }
    ]
  })
}
```


## IAM Security Checklist

| Principle | Implementation | Purpose |
|-----------|---------------|---------|
| Least privilege | Specific resources + actions | Limit blast radius |
| No wildcards | Explicit ARNs | Predictable access |
| Conditions | VPC, MFA, tags | Context-aware security |
| Permission boundaries | Limit delegation | Prevent privilege escalation |
| Regular review | IAM Access Analyzer | Remove unused permissions |

## Policy Evaluation Order

```
1. Explicit Deny in any policy → DENY
2. SCP Deny → DENY
3. Permission Boundary Deny → DENY
4. Session Policy Deny → DENY
5. Identity Policy Allow? → Check Resource Policy
6. Resource Policy Allow? → ALLOW
7. Default → DENY
```


---

### Quick Check

**Why should you use IAM roles instead of IAM users with access keys for applications?**

   A. IAM roles are cheaper than IAM users
-> B. **Roles provide temporary credentials that auto-rotate, eliminating the risk of leaked long-term access keys**
   C. IAM users cannot access AWS services
   D. Roles have higher API rate limits

<details>
<summary>See Answer</summary>

IAM user access keys are long-term credentials that must be manually rotated and can be leaked (committed to git, shared in logs, etc.). IAM roles provide temporary credentials (typically 1-12 hours) that automatically rotate. If temporary credentials are leaked, the exposure window is limited. For EC2, use instance profiles; for Lambda, use execution roles; for ECS, use task roles. Reserve IAM users only for human CLI access with MFA.

</details>

---

### Question 11: Build a CI/CD pipeline with CodePipeline that deploys to ECS with blue-green deployments.

**Type:** Practical | **Category:** CI/CD

## The Scenario

Your deployment process is problematic:

```
Current State:
├── Manual deployments via SSH
├── Deployment time: 45 minutes
├── Rollback: Manual, takes 1+ hour
├── Downtime: 5-10 minutes per deployment
├── Testing: Manual, often skipped
└── Last failed deployment: 3 days recovery
```

## The Challenge

Build a fully automated CI/CD pipeline using AWS CodePipeline, CodeBuild, and CodeDeploy with blue-green deployments to ECS for zero-downtime releases.


> **Common Mistake:** A junior engineer might deploy directly to production, skip testing stages, use rolling deployments without health checks, or have no rollback strategy. These approaches cause production outages, release bugs, create extended downtime, and make recovery difficult.

> **Senior Engineer Approach:** A senior engineer implements multi-stage pipelines with proper testing, uses blue-green deployments for instant rollback, configures deployment health checks, and automates everything from commit to production.

### Step 1: Pipeline Architecture

```
CI/CD Pipeline Architecture:
┌────────────┐    ┌────────────┐    ┌────────────┐    ┌────────────┐
│   Source   │───►│   Build    │───►│   Test     │───►│  Deploy    │
│            │    │            │    │            │    │  (Staging) │
│  GitHub    │    │ CodeBuild  │    │ CodeBuild  │    │   ECS      │
│  Webhook   │    │ + ECR Push │    │ + Reports  │    │            │
└────────────┘    └────────────┘    └────────────┘    └─────┬──────┘
                                                            │
                                                     ┌──────▼──────┐
                                                     │  Manual     │
                                                     │  Approval   │
                                                     └──────┬──────┘
                                                            │
                                    ┌────────────┐    ┌─────▼──────┐
                                    │  Rollback  │◄───│   Deploy   │
                                    │  (Auto)    │    │   (Prod)   │
                                    └────────────┘    │ Blue-Green │
                                                      └────────────┘
```

### Step 2: CodePipeline Configuration

```hcl
resource "aws_codepipeline" "main" {
  name     = "orders-service-pipeline"
  role_arn = aws_iam_role.codepipeline.arn

  artifact_store {
    location = aws_s3_bucket.artifacts.bucket
    type     = "S3"

    encryption_key {
      id   = aws_kms_key.artifacts.arn
      type = "KMS"
    }
  }

  # Source Stage
  stage {
    name = "Source"

    action {
      name             = "Source"
      category         = "Source"
      owner            = "AWS"
      provider         = "CodeStarSourceConnection"
      version          = "1"
      output_artifacts = ["source_output"]

      configuration = {
        ConnectionArn    = aws_codestarconnections_connection.github.arn
        FullRepositoryId = "myorg/orders-service"
        BranchName       = "main"
      }
    }
  }

  # Build Stage
  stage {
    name = "Build"

    action {
      name             = "Build"
      category         = "Build"
      owner            = "AWS"
      provider         = "CodeBuild"
      input_artifacts  = ["source_output"]
      output_artifacts = ["build_output"]
      version          = "1"

      configuration = {
        ProjectName = aws_codebuild_project.build.name
      }
    }
  }

  # Test Stage
  stage {
    name = "Test"

    action {
      name             = "UnitTests"
      category         = "Build"
      owner            = "AWS"
      provider         = "CodeBuild"
      input_artifacts  = ["source_output"]
      output_artifacts = ["test_output"]
      version          = "1"
      run_order        = 1

      configuration = {
        ProjectName = aws_codebuild_project.test.name
      }
    }

    action {
      name             = "IntegrationTests"
      category         = "Build"
      owner            = "AWS"
      provider         = "CodeBuild"
      input_artifacts  = ["source_output"]
      version          = "1"
      run_order        = 2

      configuration = {
        ProjectName = aws_codebuild_project.integration_test.name
      }
    }
  }

  # Deploy to Staging
  stage {
    name = "DeployStaging"

    action {
      name            = "Deploy"
      category        = "Deploy"
      owner           = "AWS"
      provider        = "ECS"
      input_artifacts = ["build_output"]
      version         = "1"

      configuration = {
        ClusterName = aws_ecs_cluster.staging.name
        ServiceName = aws_ecs_service.staging.name
        FileName    = "imagedefinitions.json"
      }
    }
  }

  # Manual Approval
  stage {
    name = "Approval"

    action {
      name     = "ManualApproval"
      category = "Approval"
      owner    = "AWS"
      provider = "Manual"
      version  = "1"

      configuration = {
        CustomData         = "Please review staging deployment before production"
        NotificationArn    = aws_sns_topic.approvals.arn
        ExternalEntityLink = "https://staging.example.com"
      }
    }
  }

  # Deploy to Production (Blue-Green)
  stage {
    name = "DeployProduction"

    action {
      name            = "Deploy"
      category        = "Deploy"
      owner           = "AWS"
      provider        = "CodeDeployToECS"
      input_artifacts = ["build_output"]
      version         = "1"

      configuration = {
        ApplicationName                = aws_codedeploy_app.main.name
        DeploymentGroupName            = aws_codedeploy_deployment_group.production.deployment_group_name
        TaskDefinitionTemplateArtifact = "build_output"
        TaskDefinitionTemplatePath     = "taskdef.json"
        AppSpecTemplateArtifact        = "build_output"
        AppSpecTemplatePath            = "appspec.yaml"
      }
    }
  }
}
```

### Step 3: CodeBuild Projects

```hcl
resource "aws_codebuild_project" "build" {
  name         = "orders-service-build"
  description  = "Build Docker image and push to ECR"
  service_role = aws_iam_role.codebuild.arn

  artifacts {
    type = "CODEPIPELINE"
  }

  environment {
    compute_type                = "BUILD_GENERAL1_MEDIUM"
    image                       = "aws/codebuild/amazonlinux2-x86_64-standard:4.0"
    type                        = "LINUX_CONTAINER"
    image_pull_credentials_type = "CODEBUILD"
    privileged_mode             = true  # Required for Docker builds

    environment_variable {
      name  = "AWS_ACCOUNT_ID"
      value = data.aws_caller_identity.current.account_id
    }

    environment_variable {
      name  = "ECR_REPO"
      value = aws_ecr_repository.main.repository_url
    }

    environment_variable {
      name  = "AWS_DEFAULT_REGION"
      value = var.region
    }
  }

  source {
    type      = "CODEPIPELINE"
    buildspec = "buildspec.yml"
  }

  cache {
    type  = "S3"
    location = "${aws_s3_bucket.artifacts.bucket}/cache"
  }

  logs_config {
    cloudwatch_logs {
      group_name  = "/codebuild/orders-service"
      stream_name = "build"
    }
  }
}

resource "aws_codebuild_project" "test" {
  name         = "orders-service-test"
  description  = "Run unit and integration tests"
  service_role = aws_iam_role.codebuild.arn

  artifacts {
    type = "CODEPIPELINE"
  }

  environment {
    compute_type = "BUILD_GENERAL1_MEDIUM"
    image        = "aws/codebuild/amazonlinux2-x86_64-standard:4.0"
    type         = "LINUX_CONTAINER"
  }

  source {
    type      = "CODEPIPELINE"
    buildspec = "buildspec-test.yml"
  }

  # Test reports
  logs_config {
    cloudwatch_logs {
      group_name  = "/codebuild/orders-service"
      stream_name = "test"
    }
  }
}
```

### Step 4: Buildspec Files

```yaml
# buildspec.yml - Build and push Docker image
version: 0.2

env:
  variables:
    DOCKER_BUILDKIT: "1"

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - IMAGE_TAG=${COMMIT_HASH:=latest}

  build:
    commands:
      - echo Building Docker image...
      - docker build -t $ECR_REPO:$IMAGE_TAG -t $ECR_REPO:latest .

  post_build:
    commands:
      - echo Pushing Docker image...
      - docker push $ECR_REPO:$IMAGE_TAG
      - docker push $ECR_REPO:latest
      - echo Writing image definitions...
      - printf '[{"name":"api","imageUri":"%s"}]' $ECR_REPO:$IMAGE_TAG > imagedefinitions.json
      - echo Writing task definition...
      - envsubst < taskdef-template.json > taskdef.json

artifacts:
  files:
    - imagedefinitions.json
    - taskdef.json
    - appspec.yaml

cache:
  paths:
    - '/root/.cache/**/*'
```

```yaml
# buildspec-test.yml - Run tests
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 18
    commands:
      - npm ci

  build:
    commands:
      - echo Running tests...
      - npm run test:coverage

  post_build:
    commands:
      - echo Running linting...
      - npm run lint

reports:
  jest-reports:
    files:
      - 'coverage/clover.xml'
    file-format: CLOVERXML

  junit-reports:
    files:
      - 'junit.xml'
    file-format: JUNITXML

artifacts:
  files:
    - coverage/**/*
  base-directory: '.'
```

### Step 5: Blue-Green Deployment with CodeDeploy

```hcl
resource "aws_codedeploy_app" "main" {
  compute_platform = "ECS"
  name             = "orders-service"
}

resource "aws_codedeploy_deployment_group" "production" {
  app_name               = aws_codedeploy_app.main.name
  deployment_group_name  = "production"
  deployment_config_name = "CodeDeployDefault.ECSAllAtOnce"
  service_role_arn       = aws_iam_role.codedeploy.arn

  ecs_service {
    cluster_name = aws_ecs_cluster.production.name
    service_name = aws_ecs_service.production.name
  }

  deployment_style {
    deployment_option = "WITH_TRAFFIC_CONTROL"
    deployment_type   = "BLUE_GREEN"
  }

  blue_green_deployment_config {
    deployment_ready_option {
      action_on_timeout = "CONTINUE_DEPLOYMENT"
      wait_time_in_minutes = 5
    }

    terminate_blue_instances_on_deployment_success {
      action                           = "TERMINATE"
      termination_wait_time_in_minutes = 60
    }
  }

  load_balancer_info {
    target_group_pair_info {
      prod_traffic_route {
        listener_arns = [aws_lb_listener.https.arn]
      }

      test_traffic_route {
        listener_arns = [aws_lb_listener.test.arn]
      }

      target_group {
        name = aws_lb_target_group.blue.name
      }

      target_group {
        name = aws_lb_target_group.green.name
      }
    }
  }

  auto_rollback_configuration {
    enabled = true
    events  = ["DEPLOYMENT_FAILURE", "DEPLOYMENT_STOP_ON_ALARM"]
  }

  alarm_configuration {
    enabled = true
    alarms  = [
      aws_cloudwatch_metric_alarm.error_rate.alarm_name,
      aws_cloudwatch_metric_alarm.latency.alarm_name
    ]
  }
}
```

### Step 6: AppSpec for ECS

```yaml
# appspec.yaml
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: <TASK_DEFINITION>
        LoadBalancerInfo:
          ContainerName: "api"
          ContainerPort: 3000
        PlatformVersion: "LATEST"
        NetworkConfiguration:
          AwsvpcConfiguration:
            Subnets:
              - "subnet-xxx"
              - "subnet-yyy"
            SecurityGroups:
              - "sg-xxx"
            AssignPublicIp: "DISABLED"

Hooks:
  - BeforeInstall: "LambdaFunctionToValidateBeforeInstall"
  - AfterInstall: "LambdaFunctionToValidateAfterInstall"
  - BeforeAllowTraffic: "LambdaFunctionToValidateBeforeTraffic"
  - AfterAllowTraffic: "LambdaFunctionToValidateAfterTraffic"
```

### Step 7: Deployment Hooks (Lambda)

```python


codedeploy = boto3.client('codedeploy')

def before_allow_traffic(event, context):
    """Validate deployment before switching traffic."""
    deployment_id = event['DeploymentId']
    lifecycle_event_hook_execution_id = event['LifecycleEventHookExecutionId']

    try:
        # Run health checks against green environment
        response = requests.get(
            'http://internal-green-alb/health',
            timeout=10
        )

        if response.status_code == 200:
            # Health check passed
            codedeploy.put_lifecycle_event_hook_execution_status(
                deploymentId=deployment_id,
                lifecycleEventHookExecutionId=lifecycle_event_hook_execution_id,
                status='Succeeded'
            )
        else:
            raise Exception(f"Health check failed: {response.status_code}")

    except Exception as e:
        print(f"Validation failed: {e}")
        codedeploy.put_lifecycle_event_hook_execution_status(
            deploymentId=deployment_id,
            lifecycleEventHookExecutionId=lifecycle_event_hook_execution_id,
            status='Failed'
        )

def after_allow_traffic(event, context):
    """Validate deployment after switching traffic."""
    deployment_id = event['DeploymentId']
    lifecycle_event_hook_execution_id = event['LifecycleEventHookExecutionId']

    try:
        # Run smoke tests against production
        run_smoke_tests()

        codedeploy.put_lifecycle_event_hook_execution_status(
            deploymentId=deployment_id,
            lifecycleEventHookExecutionId=lifecycle_event_hook_execution_id,
            status='Succeeded'
        )

    except Exception as e:
        print(f"Smoke tests failed: {e}")
        # This will trigger automatic rollback
        codedeploy.put_lifecycle_event_hook_execution_status(
            deploymentId=deployment_id,
            lifecycleEventHookExecutionId=lifecycle_event_hook_execution_id,
            status='Failed'
        )
```

### Step 8: Pipeline Notifications

```hcl
resource "aws_codestarnotifications_notification_rule" "pipeline" {
  name        = "orders-pipeline-notifications"
  detail_type = "FULL"
  resource    = aws_codepipeline.main.arn

  event_type_ids = [
    "codepipeline-pipeline-pipeline-execution-started",
    "codepipeline-pipeline-pipeline-execution-failed",
    "codepipeline-pipeline-pipeline-execution-succeeded",
    "codepipeline-pipeline-manual-approval-needed"
  ]

  target {
    address = aws_sns_topic.pipeline_notifications.arn
  }
}

# Slack integration via Lambda
resource "aws_sns_topic_subscription" "slack" {
  topic_arn = aws_sns_topic.pipeline_notifications.arn
  protocol  = "lambda"
  endpoint  = aws_lambda_function.slack_notifier.arn
}
```


## CI/CD Best Practices

| Stage | Implementation | Purpose |
|-------|---------------|---------|
| Source | GitHub with webhooks | Trigger on push |
| Build | Docker multi-stage | Smaller images |
| Test | Unit + Integration | Catch bugs early |
| Staging | Full environment | Validate changes |
| Approval | Manual gate | Human oversight |
| Production | Blue-green | Zero downtime |
| Rollback | Automatic on alarm | Fast recovery |

---

### Question 12: Your AWS bill increased 40% last month. Identify waste and implement cost controls.

**Type:** Practical | **Category:** Cost Optimization

## The Scenario

Your AWS costs are out of control:

```
Cost Breakdown (Last Month):
├── EC2: $45,000 (40% increase)
├── RDS: $12,000 (stable)
├── S3: $8,000 (20% increase)
├── NAT Gateway: $6,000 (100% increase!)
├── Data Transfer: $5,000
├── Lambda: $3,000
└── Total: $79,000 (+40% from previous month)

Issues Found:
├── Dev instances running 24/7
├── Unattached EBS volumes: 50TB
├── Old snapshots: 100TB
├── No Reserved Instances
└── No Savings Plans
```

## The Challenge

Identify cost optimization opportunities, implement proper tagging for cost allocation, set up budgets and alerts, and reduce the bill by at least 30%.


> **Common Mistake:** A junior engineer might just turn off instances randomly, delete resources without checking dependencies, or ignore the data transfer costs. These approaches cause outages, lose data, and miss the biggest savings opportunities.

> **Senior Engineer Approach:** A senior engineer analyzes Cost Explorer data, implements tagging for attribution, uses Compute Optimizer for right-sizing, leverages Savings Plans, and addresses each cost category systematically with proper controls.

### Step 1: Cost Analysis with Cost Explorer

```bash
# Get cost breakdown by service
aws ce get-cost-and-usage \
  --time-period Start=2024-01-01,End=2024-01-31 \
  --granularity MONTHLY \
  --metrics "UnblendedCost" \
  --group-by Type=DIMENSION,Key=SERVICE

# Find unused resources
# Unattached EBS volumes
aws ec2 describe-volumes \
  --filters "Name=status,Values=available" \
  --query 'Volumes[*].[VolumeId,Size,CreateTime]'

# Old snapshots (over 90 days)
aws ec2 describe-snapshots --owner-ids self \
  --query 'Snapshots[?StartTime<=`2023-10-01`].[SnapshotId,VolumeSize,StartTime]'

# Idle EC2 instances (low CPU)
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-xxx \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-31T00:00:00Z \
  --period 86400 \
  --statistics Average

# Unattached Elastic IPs
aws ec2 describe-addresses \
  --query 'Addresses[?AssociationId==`null`].[PublicIp,AllocationId]'
```

### Step 2: Cost Allocation Tags

```hcl
# Enforce tagging with AWS Config
resource "aws_config_config_rule" "required_tags" {
  name = "required-tags"

  source {
    owner             = "AWS"
    source_identifier = "REQUIRED_TAGS"
  }

  input_parameters = jsonencode({
    tag1Key   = "Environment"
    tag1Value = "production,staging,development"
    tag2Key   = "Team"
    tag3Key   = "CostCenter"
  })

  scope {
    compliance_resource_types = [
      "AWS::EC2::Instance",
      "AWS::RDS::DBInstance",
      "AWS::S3::Bucket",
      "AWS::Lambda::Function"
    ]
  }
}

# Tag policy in AWS Organizations
resource "aws_organizations_policy" "tag_policy" {
  name    = "required-tags"
  type    = "TAG_POLICY"
  content = jsonencode({
    tags = {
      Environment = {
        tag_key = {
          @@assign = "Environment"
        }
        tag_value = {
          @@assign = ["production", "staging", "development"]
        }
        enforced_for = {
          @@assign = ["ec2:instance", "rds:db", "s3:bucket"]
        }
      }
      Team = {
        tag_key = {
          @@assign = "Team"
        }
      }
      CostCenter = {
        tag_key = {
          @@assign = "CostCenter"
        }
      }
    }
  })
}
```

### Step 3: Savings Plans and Reserved Instances

```hcl
# Check recommendations
# aws ce get-savings-plans-purchase-recommendation

# Compute Savings Plans (most flexible)
# Covers EC2, Lambda, and Fargate
# 1-year no upfront: ~20% savings
# 1-year all upfront: ~30% savings
# 3-year all upfront: ~50% savings

# Example: Purchase via CLI
# aws savingsplans create-savings-plan \
#   --savings-plan-offering-id xxx \
#   --commitment 10.0 \
#   --savings-plan-type ComputeSavingsPlans

# Reserved Instances for RDS
resource "aws_db_instance" "main" {
  # ... configuration

  # Use Reserved Instance pricing
  # Purchase separately via console or CLI
  # aws rds purchase-reserved-db-instances-offering
}

# Cost comparison table
#
# Resource Type    | On-Demand | 1yr No Upfront | 1yr All Upfront | 3yr All Upfront
# EC2 m5.xlarge   | $140/mo   | $90/mo (-36%)  | $84/mo (-40%)   | $56/mo (-60%)
# RDS db.r5.large | $175/mo   | $110/mo (-37%) | $100/mo (-43%)  | $67/mo (-62%)
```

### Step 4: Right-Sizing with Compute Optimizer

```bash
# Enable Compute Optimizer
aws compute-optimizer update-enrollment-status --status Active

# Get EC2 recommendations
aws compute-optimizer get-ec2-instance-recommendations \
  --query 'instanceRecommendations[*].[instanceArn,currentInstanceType,recommendationOptions[0].instanceType,recommendationOptions[0].estimatedMonthlySavings]'

# Get Lambda recommendations
aws compute-optimizer get-lambda-function-recommendations \
  --query 'lambdaFunctionRecommendations[*].[functionArn,currentMemorySize,memorySizeRecommendationOptions[0].memorySize]'
```

```hcl
# Auto-shutdown for dev/test environments
resource "aws_autoscaling_schedule" "scale_down" {
  scheduled_action_name  = "scale-down-night"
  min_size               = 0
  max_size               = 0
  desired_capacity       = 0
  recurrence             = "0 20 * * MON-FRI"  # 8 PM weekdays
  autoscaling_group_name = aws_autoscaling_group.dev.name
}

resource "aws_autoscaling_schedule" "scale_up" {
  scheduled_action_name  = "scale-up-morning"
  min_size               = 2
  max_size               = 4
  desired_capacity       = 2
  recurrence             = "0 8 * * MON-FRI"  # 8 AM weekdays
  autoscaling_group_name = aws_autoscaling_group.dev.name
}

# Instance Scheduler for EC2/RDS
# Use AWS Instance Scheduler solution
# https://aws.amazon.com/solutions/implementations/instance-scheduler-on-aws/
```

### Step 5: S3 Cost Optimization

```hcl
resource "aws_s3_bucket_lifecycle_configuration" "main" {
  bucket = aws_s3_bucket.data.id

  # Move to cheaper storage classes
  rule {
    id     = "transition-to-ia"
    status = "Enabled"

    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }

    transition {
      days          = 90
      storage_class = "GLACIER_IR"  # Instant Retrieval
    }

    transition {
      days          = 180
      storage_class = "GLACIER"
    }

    transition {
      days          = 365
      storage_class = "DEEP_ARCHIVE"
    }
  }

  # Delete old versions
  rule {
    id     = "delete-old-versions"
    status = "Enabled"

    noncurrent_version_transition {
      noncurrent_days = 30
      storage_class   = "STANDARD_IA"
    }

    noncurrent_version_expiration {
      noncurrent_days = 90
    }
  }

  # Abort incomplete multipart uploads
  rule {
    id     = "abort-multipart"
    status = "Enabled"

    abort_incomplete_multipart_upload {
      days_after_initiation = 7
    }
  }

  # Delete expired objects
  rule {
    id     = "expire-logs"
    status = "Enabled"

    filter {
      prefix = "logs/"
    }

    expiration {
      days = 30
    }
  }
}

# Enable S3 Intelligent-Tiering for unknown access patterns
resource "aws_s3_bucket_intelligent_tiering_configuration" "main" {
  bucket = aws_s3_bucket.data.id
  name   = "entire-bucket"

  tiering {
    access_tier = "ARCHIVE_ACCESS"
    days        = 90
  }

  tiering {
    access_tier = "DEEP_ARCHIVE_ACCESS"
    days        = 180
  }
}
```

### Step 6: NAT Gateway Optimization

```hcl
# Use VPC endpoints instead of NAT Gateway
# Gateway endpoints (FREE): S3, DynamoDB
resource "aws_vpc_endpoint" "s3" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.${var.region}.s3"
  vpc_endpoint_type = "Gateway"
  route_table_ids   = aws_route_table.private[*].id
}

resource "aws_vpc_endpoint" "dynamodb" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.${var.region}.dynamodb"
  vpc_endpoint_type = "Gateway"
  route_table_ids   = aws_route_table.private[*].id
}

# Interface endpoints (~$7/month each, but saves NAT data transfer)
resource "aws_vpc_endpoint" "ecr_api" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.${var.region}.ecr.api"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.vpc_endpoints.id]
  private_dns_enabled = true
}

# NAT Gateway cost breakdown:
# - Hourly charge: $0.045/hour = ~$32/month per NAT GW
# - Data processing: $0.045/GB
# - Cross-AZ: Additional $0.01/GB
#
# If processing 1TB/month through NAT: $32 + $45 = $77/month
# With VPC endpoints for S3/ECR: Saves most of that data transfer
```

### Step 7: Budget Alerts

```hcl
resource "aws_budgets_budget" "monthly" {
  name              = "monthly-budget"
  budget_type       = "COST"
  limit_amount      = "60000"
  limit_unit        = "USD"
  time_unit         = "MONTHLY"
  time_period_start = "2024-01-01_00:00"

  cost_filter {
    name   = "TagKeyValue"
    values = ["user:Environment$production"]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 80
    threshold_type             = "PERCENTAGE"
    notification_type          = "ACTUAL"
    subscriber_email_addresses = ["team@example.com"]
    subscriber_sns_topic_arns  = [aws_sns_topic.budget_alerts.arn]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 100
    threshold_type             = "PERCENTAGE"
    notification_type          = "FORECASTED"
    subscriber_email_addresses = ["team@example.com"]
    subscriber_sns_topic_arns  = [aws_sns_topic.budget_alerts.arn]
  }
}

# Per-service budgets
resource "aws_budgets_budget" "ec2" {
  name              = "ec2-budget"
  budget_type       = "COST"
  limit_amount      = "35000"
  limit_unit        = "USD"
  time_unit         = "MONTHLY"
  time_period_start = "2024-01-01_00:00"

  cost_filter {
    name   = "Service"
    values = ["Amazon Elastic Compute Cloud - Compute"]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 90
    threshold_type             = "PERCENTAGE"
    notification_type          = "ACTUAL"
    subscriber_sns_topic_arns  = [aws_sns_topic.budget_alerts.arn]
  }
}
```

### Step 8: Automated Cleanup

```python

from datetime import datetime, timedelta

ec2 = boto3.client('ec2')

def cleanup_unused_resources():
    """Find and optionally delete unused resources."""
    savings = 0

    # Unattached EBS volumes
    volumes = ec2.describe_volumes(
        Filters=[{'Name': 'status', 'Values': ['available']}]
    )['Volumes']

    for vol in volumes:
        age = (datetime.now(vol['CreateTime'].tzinfo) - vol['CreateTime']).days
        if age > 30:
            print(f"Unused volume: {vol['VolumeId']}, Size: {vol['Size']}GB, Age: {age} days")
            savings += vol['Size'] * 0.10  # ~$0.10/GB/month

            # Uncomment to delete
            # ec2.delete_volume(VolumeId=vol['VolumeId'])

    # Old snapshots
    snapshots = ec2.describe_snapshots(OwnerIds=['self'])['Snapshots']
    cutoff = datetime.now(snapshots[0]['StartTime'].tzinfo) - timedelta(days=90)

    for snap in snapshots:
        if snap['StartTime'] < cutoff:
            print(f"Old snapshot: {snap['SnapshotId']}, Size: {snap['VolumeSize']}GB")
            savings += snap['VolumeSize'] * 0.05  # ~$0.05/GB/month

    # Unattached Elastic IPs
    addresses = ec2.describe_addresses()['Addresses']
    for addr in addresses:
        if 'AssociationId' not in addr:
            print(f"Unused EIP: {addr['PublicIp']}")
            savings += 3.60  # ~$3.60/month

    print(f"\nEstimated monthly savings: ${savings:.2f}")

if __name__ == '__main__':
    cleanup_unused_resources()
```


## Cost Optimization Summary

| Strategy | Potential Savings | Implementation Effort |
|----------|------------------|----------------------|
| Savings Plans (1yr) | 20-30% on compute | Low (purchase decision) |
| Right-sizing | 20-40% | Medium (analysis needed) |
| Dev shutdown | 65% on dev/test | Medium (scheduling) |
| S3 lifecycle | 50-80% on storage | Low (configuration) |
| VPC endpoints | 50%+ on NAT | Low (infrastructure) |
| Spot instances | 60-90% | High (architecture change) |


---

### Quick Check

**Why do NAT Gateway costs often spike unexpectedly?**

   A. NAT Gateways have hidden fees
-> B. **NAT Gateways charge both hourly AND per-GB data processing, and AWS service calls from private subnets all go through NAT unless you use VPC endpoints**
   C. NAT Gateways are more expensive in certain regions
   D. NAT Gateways charge extra during peak hours

<details>
<summary>See Answer</summary>

NAT Gateway has two cost components: hourly charge (~$32/month) and data processing ($0.045/GB). When Lambda functions, ECS tasks, or EC2 instances in private subnets call AWS APIs (S3, DynamoDB, Secrets Manager, CloudWatch, ECR), all that traffic goes through NAT Gateway unless you configure VPC endpoints. A single Lambda function pulling images from ECR can transfer gigabytes daily. Gateway endpoints for S3 and DynamoDB are free; interface endpoints cost ~$7/month but eliminate NAT data transfer costs for those services.

</details>

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*
