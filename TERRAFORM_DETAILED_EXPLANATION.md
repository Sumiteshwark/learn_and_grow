# 🚀 **Terraform Deep Dive - Complete Infrastructure as Code Guide**

## 📋 **Table of Contents**
- [What is Terraform?](#what-is-terraform)
- [Terraform in Sentinel](#terraform-in-sentinel)
- [Core Concepts](#core-concepts)
- [Terraform CLI Commands](#terraform-cli-commands)
- [Providers and Resources](#providers-and-resources)
- [State Management](#state-management)
- [Modules and Workspaces](#modules-and-workspaces)
- [Variables and Outputs](#variables-and-outputs)
- [Expressions and Functions](#expressions-and-functions)
- [Remote State and Locking](#remote-state-and-locking)
- [Testing and Validation](#testing-and-validation)
- [Security Best Practices](#security-best-practices)
- [Production Deployment](#production-deployment)
- [CI/CD Integration](#ci-cd-integration)
- [Advanced Features](#advanced-features)
- [Real-World Use Cases](#real-world-use-cases)

---

## 🤔 **What is Terraform?**

**Terraform** is an open-source **Infrastructure as Code (IaC)** tool by HashiCorp that enables you to define, provision, and manage infrastructure as code. It supports multiple cloud providers and services through a declarative configuration language.

### **Core Characteristics:**
- ✅ **Declarative**: Describe what you want, not how to do it
- ✅ **Multi-Cloud**: AWS, Azure, GCP, and 300+ providers
- ✅ **Version Control**: Infrastructure changes are tracked in Git
- ✅ **Dependency Management**: Automatic resource dependency resolution
- ✅ **State Management**: Tracks infrastructure state
- ✅ **Idempotent**: Safe to run multiple times

### **Why Terraform?**
```hcl
# Traditional Manual Setup (Error-prone)
# 1. Go to AWS Console
# 2. Create VPC manually
# 3. Create subnets manually
# 4. Configure security groups
# 5. Launch EC2 instances
# 6. Hope you didn't miss anything...

# Terraform (Automated & Reproducible)
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = { Name = "main-vpc" }
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
}

resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.public.id
}
```

---

## 🎯 **Terraform in Sentinel**

### **Sentinel's Infrastructure Needs:**
```hcl
# 1. AWS Infrastructure for API & Worker Services
resource "aws_ecs_cluster" "sentinel" {
  name = "sentinel-cluster"
}

resource "aws_ecs_service" "api" {
  name            = "sentinel-api"
  cluster         = aws_ecs_cluster.sentinel.id
  task_definition = aws_ecs_task_definition.api.arn
  desired_count   = 3

  load_balancer {
    target_group_arn = aws_lb_target_group.api.arn
    container_name   = "api"
    container_port   = 3000
  }
}

# 2. Database Infrastructure
resource "aws_rds_cluster" "sentinel_db" {
  cluster_identifier = "sentinel-postgres"
  engine            = "aurora-postgresql"
  master_username   = var.db_username
  master_password   = var.db_password
  database_name     = "sentinel"
}

# 3. Redis for Caching & Queues
resource "aws_elasticache_cluster" "redis" {
  cluster_id      = "sentinel-redis"
  engine         = "redis"
  node_type      = "cache.t3.micro"
  num_cache_nodes = 1
}

# 4. Monitoring & Logging
resource "aws_cloudwatch_log_group" "sentinel" {
  name              = "/ecs/sentinel"
  retention_in_days = 30
}
```

### **Architecture Integration:**
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Terraform     │────│   AWS Provider   │────│   Cloud Infra   │
│ Configuration   │    │                  │    │   Resources     │
│   (.tf files)   │    │ ┌─────────────┐ │    │                 │
│                 │    │ │ ECS Cluster │◄├───►│ ┌─────────────┐ │
│ ┌─────────────┐ │    │ │ RDS Aurora  │ │    │ │  Sentinel   │ │
│ │ Modules &   │◄├───►│ │ ElastiCache │ │    │ │   Services  │ │
│ │ Variables   │ │    │ │ CloudWatch  │ │    │ │             │ │
│ └─────────────┘ │    │ └─────────────┘ │    │ └─────────────┘ │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

---

## 🏗️ **Core Concepts**

### **1. Providers**
```hcl
# Configure AWS Provider
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
  # Authentication via AWS CLI, IAM roles, or environment variables
}
```

### **2. Resources**
```hcl
# Infrastructure resources you want to create
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleInstance"
  }
}

# Data sources (read existing infrastructure)
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}
```

### **3. Configuration Language (HCL)**
```hcl
# Blocks, Arguments, and Expressions
resource "aws_security_group" "web" {
  name_prefix = "web-sg-"  # Argument
  description = "Security group for web servers"

  # Nested blocks
  ingress {
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

  # Expressions and references
  tags = {
    Name        = "web-security-group"
    Environment = var.environment
    Project     = local.project_name
  }
}
```

---

## 💻 **Terraform CLI Commands**

### **Basic Workflow Commands**
```bash
# Initialize working directory
terraform init

# Format configuration files
terraform fmt

# Validate configuration
terraform validate

# Plan changes
terraform plan

# Apply changes
terraform apply

# Destroy infrastructure
terraform destroy
```

### **Advanced Commands**
```bash
# Plan with specific variables
terraform plan -var="environment=production"

# Apply with auto-approve (dangerous in production)
terraform apply -auto-approve

# Target specific resources
terraform apply -target=aws_instance.example

# Refresh state (sync with real infrastructure)
terraform refresh

# Show current state
terraform show

# List resources in state
terraform state list

# Move resources in state
terraform state mv aws_instance.old aws_instance.new

# Remove resources from state
terraform state rm aws_instance.example

# Import existing resources
terraform import aws_instance.example i-1234567890abcdef0
```

### **Workspace Commands**
```bash
# Create new workspace
terraform workspace new development

# Switch workspace
terraform workspace select production

# List workspaces
terraform workspace list

# Show current workspace
terraform workspace show

# Delete workspace
terraform workspace delete development
```

### **Output and Debugging**
```bash
# Show outputs
terraform output

# Show specific output
terraform output instance_ip

# Debug mode
TF_LOG=DEBUG terraform apply

# JSON output for CI/CD
terraform plan -json

# Save plan to file
terraform plan -out=tfplan
terraform apply tfplan
```

---

## 🔌 **Providers and Resources**

### **Popular Providers**
```hcl
# AWS Provider
provider "aws" {
  region = "us-east-1"
  # Authentication methods:
  # 1. AWS CLI credentials
  # 2. IAM roles (EC2/ECS)
  # 3. Environment variables
  # 4. Shared credentials file
}

# Azure Provider
provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
}

# Google Cloud Provider
provider "google" {
  project = var.project_id
  region  = "us-central1"
}

# Kubernetes Provider
provider "kubernetes" {
  config_path = "~/.kube/config"
}

# Docker Provider
provider "docker" {
  host = "unix:///var/run/docker.sock"
}
```

### **Resource Types and Examples**

#### **Compute Resources**
```hcl
# EC2 Instance
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type

  tags = {
    Name = "web-server"
  }
}

# ECS Cluster and Service
resource "aws_ecs_cluster" "main" {
  name = "main-cluster"
}

resource "aws_ecs_service" "app" {
  name            = "app-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = var.desired_count
}
```

#### **Networking Resources**
```hcl
# VPC and Subnets
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = { Name = "main-vpc" }
}

resource "aws_subnet" "public" {
  count             = 3
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index)
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = { Name = "public-subnet-${count.index + 1}" }
}

# Security Groups
resource "aws_security_group" "web" {
  name_prefix = "web-sg-"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

#### **Storage Resources**
```hcl
# S3 Bucket
resource "aws_s3_bucket" "app_data" {
  bucket = "my-app-data-${random_string.suffix.result}"

  tags = {
    Environment = var.environment
  }
}

resource "aws_s3_bucket_versioning" "app_data" {
  bucket = aws_s3_bucket.app_data.id
  versioning_configuration {
    status = "Enabled"
  }
}

# RDS Database
resource "aws_db_instance" "postgres" {
  identifier             = "postgres-db"
  engine                = "postgres"
  engine_version        = "13.7"
  instance_class        = "db.t3.micro"
  allocated_storage     = 20
  username             = var.db_username
  password             = var.db_password
  db_name              = "app"
  skip_final_snapshot  = true
}
```

#### **Load Balancing**
```hcl
# Application Load Balancer
resource "aws_lb" "app" {
  name               = "app-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets           = aws_subnet.public[*].id

  tags = {
    Name = "app-alb"
  }
}

resource "aws_lb_target_group" "app" {
  name     = "app-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher            = "200"
    path               = "/health"
    port               = "traffic-port"
    protocol           = "HTTP"
    timeout            = 5
    unhealthy_threshold = 2
  }
}

resource "aws_lb_listener" "app" {
  load_balancer_arn = aws_lb.app.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}
```

---

## 📊 **State Management**

### **Local State**
```hcl
# terraform.tfstate (automatically created)
{
  "version": 4,
  "terraform_version": "1.0.0",
  "serial": 1,
  "lineage": "abcd1234-...",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "aws_instance",
      "name": "example",
      "provider_config_key": "aws",
      "instances": [
        {
          "schema_version": 1,
          "attributes": {
            "id": "i-1234567890abcdef0",
            "instance_type": "t2.micro",
            "ami": "ami-0c55b159cbfafe1d0"
          }
        }
      ]
    }
  ]
}
```

### **Remote State Configuration**
```hcl
# S3 Backend
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "sentinel/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

# Azure Backend
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-rg"
    storage_account_name = "terraformsa"
    container_name       = "tfstate"
    key                  = "sentinel.tfstate"
  }
}

# GCS Backend
terraform {
  backend "gcs" {
    bucket = "terraform-state-bucket"
    prefix = "sentinel"
  }
}
```

### **State Locking**
```hcl
# DynamoDB for S3 backend locking
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }

  tags = {
    Name = "Terraform Locks"
  }
}
```

---

## 📦 **Modules and Workspaces**

### **Creating Modules**
```hcl
# modules/vpc/main.tf
variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
}

variable "environment" {
  description = "Environment name"
  type        = string
}

resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr

  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name        = "${var.environment}-igw"
    Environment = var.environment
  }
}

output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "igw_id" {
  description = "Internet Gateway ID"
  value       = aws_internet_gateway.main.id
}
```

### **Using Modules**
```hcl
# main.tf
module "vpc" {
  source      = "./modules/vpc"
  vpc_cidr    = "10.0.0.0/16"
  environment = var.environment
}

module "web_servers" {
  source          = "./modules/web-servers"
  vpc_id          = module.vpc.vpc_id
  igw_id          = module.vpc.igw_id
  environment     = var.environment
  instance_count  = var.web_server_count
}

# Remote module from Terraform Registry
module "consul" {
  source  = "hashicorp/consul/aws"
  version = "0.11.0"

  servers = 3
}
```

### **Workspaces for Environments**
```hcl
# Use workspace name for environment-specific config
locals {
  environment = terraform.workspace
  common_tags = {
    Environment = local.environment
    Project     = "sentinel"
    ManagedBy   = "terraform"
  }
}

# Environment-specific variables
variable "instance_count" {
  description = "Number of instances"
  type        = map(number)
  default = {
    development = 1
    staging     = 2
    production  = 6
  }
}

resource "aws_instance" "app" {
  count         = var.instance_count[local.environment]
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = merge(local.common_tags, {
    Name = "app-${local.environment}-${count.index + 1}"
  })
}
```

---

## 🔧 **Variables and Outputs**

### **Variable Types and Validation**
```hcl
# Input Variables
variable "environment" {
  description = "Environment name"
  type        = string
  default     = "development"

  validation {
    condition     = contains(["development", "staging", "production"], var.environment)
    error_message = "Environment must be one of: development, staging, production."
  }
}

variable "instance_count" {
  description = "Number of EC2 instances"
  type        = number
  default     = 1

  validation {
    condition     = var.instance_count > 0 && var.instance_count <= 10
    error_message = "Instance count must be between 1 and 10."
  }
}

variable "tags" {
  description = "Common tags for resources"
  type = map(string)
  default = {
    Project     = "sentinel"
    ManagedBy   = "terraform"
  }
}

variable "security_groups" {
  description = "List of security group IDs"
  type        = list(string)
}

variable "database_config" {
  description = "Database configuration"
  type = object({
    engine   = string
    version  = string
    instance = string
  })

  default = {
    engine   = "postgres"
    version  = "13.7"
    instance = "db.t3.micro"
  }
}
```

### **Variable Files**
```hcl
# terraform.tfvars
environment     = "production"
instance_count  = 3
instance_type   = "t3.medium"

tags = {
  Environment = "production"
  Project     = "sentinel"
  Owner       = "platform-team"
}

# environment.auto.tfvars (auto-loaded)
vpc_cidr = "10.0.0.0/16"

# production.tfvars
database_config = {
  engine   = "aurora-postgresql"
  version  = "13.7"
  instance = "db.r6g.large"
}
```

### **Outputs**
```hcl
# outputs.tf
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "subnet_ids" {
  description = "List of subnet IDs"
  value       = aws_subnet.public[*].id
}

output "load_balancer_dns" {
  description = "Load balancer DNS name"
  value       = aws_lb.app.dns_name
}

output "database_endpoint" {
  description = "Database endpoint"
  value       = aws_db_instance.postgres.endpoint
  sensitive   = true  # Hide in output
}

output "instance_ips" {
  description = "Public IPs of EC2 instances"
  value = {
    for instance in aws_instance.app :
    instance.tags["Name"] => instance.public_ip
  }
}
```

---

## ⚡ **Expressions and Functions**

### **Built-in Functions**
```hcl
# String functions
locals {
  environment = var.environment
  project_name = "sentinel"
  resource_prefix = "${local.project_name}-${local.environment}"

  # String manipulation
  upper_env = upper(local.environment)
  lower_env = lower(local.environment)
  title_env = title(local.environment)

  # String replacement
  db_name = replace("sentinel-{env}-db", "{env}", local.environment)
}

# Collection functions
locals {
  availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]
  subnet_cidrs = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]

  # List operations
  first_az = element(local.availability_zones, 0)
  az_count = length(local.availability_zones)

  # Map operations
  common_tags = {
    Environment = var.environment
    Project     = "sentinel"
  }

  instance_tags = merge(local.common_tags, {
    Name = "web-server"
    Type = "application"
  })
}

# Numeric and conditional functions
locals {
  instance_count = var.environment == "production" ? 6 : 2
  disk_size = max(20, var.min_disk_size)

  # Math operations
  total_memory = var.instance_count * var.memory_per_instance
  reserved_ips = ceil(var.instance_count * 1.2)
}

# File and template functions
locals {
  user_data = templatefile("${path.module}/templates/user-data.sh.tpl", {
    db_endpoint = aws_db_instance.postgres.endpoint
    redis_endpoint = aws_elasticache_cluster.redis.cache_nodes[0].address
  })

  cloud_config = yamlencode({
    packages = ["nginx", "docker"]
    write_files = [{
      path    = "/etc/nginx/nginx.conf"
      content = file("${path.module}/configs/nginx.conf")
    }]
  })
}
```

### **Dynamic Blocks**
```hcl
# Dynamic ingress rules
resource "aws_security_group" "web" {
  name_prefix = "web-sg-"
  vpc_id      = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.allowed_ports

    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Dynamic subnets
resource "aws_subnet" "private" {
  count = length(var.private_subnet_cidrs)

  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]

  tags = {
    Name = "private-subnet-${count.index + 1}"
    Type = "private"
  }
}
```

---

## 🔒 **Remote State and Locking**

### **Remote State Backends**

#### **S3 Backend with DynamoDB Locking**
```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "sentinel/${terraform.workspace}/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
    kms_key_id     = "alias/terraform-bucket-key"
  }
}

# Create the S3 bucket and DynamoDB table
resource "aws_s3_bucket" "terraform_state" {
  bucket = "terraform-state-bucket"

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

resource "aws_s3_bucket_public_access_block" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }

  server_side_encryption {
    enabled = true
  }
}
```

#### **Terraform Cloud/Enterprise**
```hcl
terraform {
  cloud {
    organization = "sentinel-org"

    workspaces {
      name = "sentinel-${terraform.workspace}"
    }
  }
}

# Or for Terraform Enterprise
terraform {
  backend "remote" {
    organization = "sentinel-org"

    workspaces {
      name = "sentinel-infrastructure"
    }
  }
}
```

### **State Operations**
```bash
# Force unlock (use carefully!)
terraform force-unlock LOCK_ID

# Migrate state to new backend
terraform init -migrate-state

# Reconfigure backend
terraform init -reconfigure

# Pull remote state
terraform state pull > terraform.tfstate

# Push local state to remote
terraform state push terraform.tfstate
```

---

## 🧪 **Testing and Validation**

### **Pre-deployment Validation**
```bash
# Validate syntax and configuration
terraform validate

# Check formatting
terraform fmt -check

# Plan with detailed output
terraform plan -detailed-exitcode

# Validate with tflint
tflint

# Security scanning with checkov
checkov -f main.tf
```

### **Unit Tests with Terratest**
```go
// test/terraform_test.go
package test

import (
    "testing"
    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/stretchr/testify/assert"
)

func TestTerraformSentinel(t *testing.T) {
    t.Parallel()

    terraformOptions := &terraform.Options{
        TerraformDir: "../",
        Vars: map[string]interface{}{
            "environment": "test",
        },
    }

    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)

    // Test VPC creation
    vpcID := terraform.Output(t, terraformOptions, "vpc_id")
    assert.NotEmpty(t, vpcID)

    // Test security group rules
    sgID := terraform.Output(t, terraformOptions, "security_group_id")
    assert.NotEmpty(t, sgID)

    // Verify instance count
    instanceCount := terraform.Output(t, terraformOptions, "instance_count")
    assert.Equal(t, "2", instanceCount)
}
```

### **Policy as Code with Sentinel**
```hcl
# sentinel.hcl
policy "restrict_instance_types" {
    source = "./restrict_instance_types.sentinel"
    enforcement_level = "hard-mandatory"
}

policy "require_tags" {
    source = "./require_tags.sentinel"
    enforcement_level = "soft-mandatory"
}

# restrict_instance_types.sentinel
import "tfplan"

allowed_types = ["t2.micro", "t2.small", "t3.micro", "t3.small"]

allEC2Instances = filter tfplan.resource_changes as _, rc {
    rc.type is "aws_instance"
}

violations = filter allEC2Instances as address, rc {
    not contains(allowed_types, rc.change.after.instance_type)
}

main = rule {
    length(violations) is 0
}
```

---

## 🔐 **Security Best Practices**

### **Credential Management**
```hcl
# Use AWS IAM roles (preferred)
provider "aws" {
  region = "us-east-1"
  # No credentials needed - uses IAM role
}

# Environment variables (for local development)
provider "aws" {
  region = "us-east-1"
  # AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY from env
}

# Shared credentials file
provider "aws" {
  region                  = "us-east-1"
  shared_credentials_file = "~/.aws/credentials"
  profile                 = "sentinel"
}

# Assume role
provider "aws" {
  region = "us-east-1"

  assume_role {
    role_arn     = "arn:aws:iam::123456789012:role/TerraformRole"
    session_name = "terraform-session"
  }
}
```

### **Secrets Management**
```hcl
# AWS Secrets Manager
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/database/password"
}

locals {
  db_password = jsondecode(data.aws_secretsmanager_secret_version.db_password.secret_string)["password"]
}

# Azure Key Vault
data "azurerm_key_vault_secret" "db_password" {
  name         = "database-password"
  key_vault_id = data.azurerm_key_vault.example.id
}

# HashiCorp Vault
provider "vault" {
  address = "https://vault.example.com:8200"
  token   = var.vault_token
}

data "vault_generic_secret" "db_creds" {
  path = "database/creds/sentinel-role"
}
```

### **Secure State Storage**
```hcl
# S3 bucket with encryption and access controls
resource "aws_s3_bucket" "terraform_state" {
  bucket = "sentinel-terraform-state"

  versioning {
    enabled = true
  }

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
      bucket_key_enabled = true
    }
  }

  # Block all public access
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# Bucket policy for restricted access
resource "aws_s3_bucket_policy" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Deny"
        Principal = "*"
        Action = "s3:*"
        Resource = [
          aws_s3_bucket.terraform_state.arn,
          "${aws_s3_bucket.terraform_state.arn}/*"
        ]
        Condition = {
          Bool = {
            "aws:SecureTransport" = "false"
          }
        }
      }
    ]
  })
}
```

### **Input Validation and Sanitization**
```hcl
variable "environment" {
  description = "Environment name"
  type        = string

  validation {
    condition     = can(regex("^(development|staging|production)$", var.environment))
    error_message = "Environment must be development, staging, or production."
  }
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string

  validation {
    condition = contains([
      "t2.micro", "t2.small", "t3.micro", "t3.small",
      "t3.medium", "t3.large", "m5.large", "m5.xlarge"
    ], var.instance_type)
    error_message = "Instance type must be from the approved list."
  }
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)

  validation {
    condition = alltrue([
      for tag in keys(var.tags) : can(regex("^[a-zA-Z0-9_-]+$", tag))
    ])
    error_message = "Tag keys must contain only alphanumeric characters, underscores, and hyphens."
  }

  validation {
    condition = alltrue([
      for value in values(var.tags) : length(value) <= 256
    ])
    error_message = "Tag values must be 256 characters or less."
  }
}
```

---

## 🚀 **Production Deployment**

### **Deployment Pipeline**
```yaml
# .github/workflows/terraform.yml
name: 'Terraform'

on:
  push:
    branches: [ main ]
    paths: [ 'terraform/**' ]
  pull_request:
    branches: [ main ]
    paths: [ 'terraform/**' ]

env:
  TF_VERSION: '1.5.0'
  TG_VERSION: '0.48.0'

jobs:
  terraform:
    name: 'Terraform'
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v3

    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: ${{ env.TF_VERSION }}

    - name: Setup Terragrunt
      run: |
        wget -O terragrunt https://github.com/gruntwork-io/terragrunt/releases/download/v${TG_VERSION}/terragrunt_linux_amd64
        chmod +x terragrunt
        sudo mv terragrunt /usr/local/bin/

    - name: Terraform Format
      run: terraform fmt -check
      working-directory: terraform

    - name: Terraform Init
      run: terraform init
      working-directory: terraform

    - name: Terraform Validate
      run: terraform validate
      working-directory: terraform

    - name: Terraform Plan
      id: plan
      run: |
        terraform plan -no-color -out=tfplan
        terraform show -no-color tfplan
      working-directory: terraform
      continue-on-error: true

    - name: Update Pull Request
      uses: actions/github-script@v6
      if: github.event_name == 'pull_request'
      env:
        PLAN: "terraform\n${{ steps.plan.outputs.stdout }}"
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
        script: |
          const output = `#### Terraform Format and Validate 🖌\`${{ steps.fmt.outcome }}\`
          #### Terraform Plan 📖\`${{ steps.plan.outcome }}\`

          <details><summary>Show Plan</summary>

          \`\`\`\n
          ${process.env.PLAN}
          \`\`\`

          </details>

          *Pushed by: @${{ github.actor }}, Action: \`${{ github.event_name }}\`*`;

          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: output
          })

    - name: Terraform Plan Status
      if: steps.plan.outcome == 'failure'
      run: exit 1

    - name: Terraform Apply
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
      run: terraform apply -auto-approve tfplan
      working-directory: terraform
```

### **Multi-Environment Deployment**
```hcl
# terragrunt.hcl
remote_state {
  backend = "s3"
  config = {
    bucket         = "sentinel-terraform-state"
    key            = "${path_relative_to_include()}/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

inputs = {
  environment = path_relative_to_include()
  region      = "us-east-1"

  # Environment-specific configurations
  instance_count = {
    development = 1
    staging     = 2
    production  = 6
  }[path_relative_to_include()]

  instance_type = {
    development = "t3.micro"
    staging     = "t3.medium"
    production  = "m5.large"
  }[path_relative_to_include()]
}
```

### **Immutable Infrastructure**
```hcl
# Blue-Green Deployment Strategy
resource "aws_lb_listener_rule" "app" {
  listener_arn = aws_lb_listener.app.arn
  priority     = 100

  action {
    type             = "forward"
    target_group_arn = var.active_target_group_arn
  }

  condition {
    path_pattern {
      values = ["/*"]
    }
  }
}

# Rolling Update with Zero Downtime
resource "aws_ecs_service" "app" {
  name            = "sentinel-app"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn

  desired_count = var.desired_count

  deployment_controller {
    type = "ECS"
  }

  deployment_circuit_breaker {
    enable   = true
    rollback = true
  }

  # Blue-Green deployment
  deployment_configuration {
    maximum_percent         = 200
    minimum_healthy_percent = 100
  }

  load_balancer {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = "app"
    container_port   = 3000
  }
}
```

---

## 🔄 **CI/CD Integration**

### **GitHub Actions Advanced Setup**
```yaml
# .github/workflows/terraform-advanced.yml
name: 'Terraform CI/CD'

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

concurrency:
  group: terraform-${{ github.ref }}
  cancel-in-progress: true

env:
  TF_VERSION: '1.5.0'
  TFLINT_VERSION: '0.47.0'
  TFSEC_VERSION: '1.28.0'

jobs:
  terraform:
    name: 'Terraform'
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v3

    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: ${{ env.TF_VERSION }}
        terraform_wrapper: false

    - name: Setup TFLint
      run: |
        curl -s https://raw.githubusercontent.com/terraform-linters/tflint/master/install_linux.sh | bash

    - name: Setup TFSec
      run: |
        curl -s https://raw.githubusercontent.com/aquasecurity/tfsec/master/scripts/install_linux.sh | bash

    - name: Cache Terraform
      uses: actions/cache@v3
      with:
        path: ~/.terraform.d/plugin-cache
        key: terraform-${{ hashFiles('**/*.tf') }}
        restore-keys: |
          terraform-

    - name: Terraform Format
      run: terraform fmt -recursive -check

    - name: TFLint
      run: tflint --recursive

    - name: TFSec
      run: tfsec --concise-output

    - name: Terraform Init
      run: terraform init -backend=false

    - name: Terraform Validate
      run: terraform validate

    - name: Checkov
      uses: bridgecrewio/checkov-action@v12
      with:
        framework: terraform
        output_format: cli
        output_file_path: console,checkov-report.json

    - name: Terraform Plan
      id: plan
      run: |
        terraform plan -no-color -out=tfplan
        echo "plan_output<<EOF" >> $GITHUB_OUTPUT
        terraform show -no-color tfplan >> $GITHUB_OUTPUT
        echo "EOF" >> $GITHUB_OUTPUT
      continue-on-error: true

    - name: Update Pull Request
      uses: actions/github-script@v6
      if: github.event_name == 'pull_request'
      with:
        script: |
          const { data: comments } = await github.rest.issues.listComments({
            owner: context.repo.owner,
            repo: context.repo.repo,
            issue_number: context.issue.number,
          });

          const botComment = comments.find(comment =>
            comment.user.type === 'Bot' &&
            comment.body.includes('#### Terraform Plan 📖')
          );

          const output = `#### Terraform Checks 🛡️
          * Format: \`${{ steps.fmt.outcome }}\`
          * Lint: \`${{ steps.tflint.outcome }}\`
          * Security: \`${{ steps.tfsec.outcome }}\`
          * Validate: \`${{ steps.validate.outcome }}\`

          #### Terraform Plan 📖\`${{ steps.plan.outcome }}\`

          <details><summary>Show Plan</summary>

          \`\`\`\n
          ${{ steps.plan.outputs.plan_output }}
          \`\`\`

          </details>

          *Pushed by: @${{ github.actor }}, Action: \`${{ github.event_name }}\`*`;

          if (botComment) {
            github.rest.issues.updateComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              comment_id: botComment.id,
              body: output
            });
          } else {
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            });
          }

    - name: Terraform Apply
      if: |
        github.ref == 'refs/heads/main' &&
        github.event_name == 'push' &&
        steps.plan.outcome == 'success'
      run: terraform apply -auto-approve tfplan
```

### **Atlantis for GitOps**
```yaml
# atlantis.yaml
version: 3
projects:
- name: sentinel-infrastructure
  dir: terraform
  workspace: default
  terraform_version: v1.5.0
  delete_source_branch_on_merge: false
  autoplan:
    when_modified: ["*.tf", "*.tfvars"]
    enabled: true
  apply_requirements:
  - approved
  - mergeable
  - undiverged
  workflow: sentinel

workflows:
  sentinel:
    plan:
      steps:
      - init
      - plan
      - show
    apply:
      steps:
      - apply
```

---

## 🏆 **Advanced Features**

### **1. Terraform Registry and Modules**
```hcl
# Official AWS VPC Module
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "sentinel-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  enable_vpn_gateway = false

  tags = {
    Terraform = "true"
    Environment = var.environment
  }
}

# Kubernetes Module
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 19.0"

  cluster_name    = "sentinel-cluster"
  cluster_version = "1.27"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  eks_managed_node_groups = {
    general = {
      desired_size = 3
      min_size     = 1
      max_size     = 10

      instance_types = ["t3.medium"]
      capacity_type  = "ON_DEMAND"
    }
  }
}
```

### **2. Resource Graph and Dependencies**
```hcl
# Implicit Dependencies (Terraform automatically detects)
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.public.id  # Implicit dependency
}

# Explicit Dependencies
resource "aws_eip" "web" {
  vpc = true

  # Explicit dependency
  depends_on = [aws_internet_gateway.main]
}

# Resource with multiple dependencies
resource "aws_route_table_association" "public" {
  count = length(var.public_subnets)

  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id

  depends_on = [
    aws_subnet.public,
    aws_route_table.public,
    aws_internet_gateway.main
  ]
}
```

### **3. Data Sources and External Resources**
```hcl
# AWS Data Sources
data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_caller_identity" "current" {}

data "aws_region" "current" {}

# External Data Source
data "external" "example" {
  program = ["python3", "${path.module}/scripts/get_instance_data.py"]

  query = {
    instance_id = aws_instance.example.id
  }
}

# Template Data Source
data "template_file" "user_data" {
  template = file("${path.module}/templates/user-data.sh")

  vars = {
    db_host     = aws_db_instance.postgres.address
    db_port     = aws_db_instance.postgres.port
    redis_host  = aws_elasticache_cluster.redis.cache_nodes[0].address
  }
}

# Cloud-init Data Source
data "cloudinit_config" "example" {
  gzip          = true
  base64_encode = true

  part {
    content_type = "text/cloud-config"
    content = yamlencode({
      packages = ["nginx"]
      runcmd = [
        "systemctl enable nginx",
        "systemctl start nginx"
      ]
    })
  }
}
```

### **4. Provisioners and Local Execution**
```hcl
# File Provisioner
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type

  provisioner "file" {
    source      = "conf/nginx.conf"
    destination = "/etc/nginx/nginx.conf"

    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file(var.private_key_path)
      host        = self.public_ip
    }
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
      "sudo systemctl enable nginx",
      "sudo systemctl start nginx"
    ]

    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file(var.private_key_path)
      host        = self.public_ip
    }
  }
}

# Local Provisioner
resource "null_resource" "generate_inventory" {
  provisioner "local-exec" {
    command = "echo '${aws_instance.web.public_ip}' > inventory.ini"
  }

  depends_on = [aws_instance.web]
}

# Local Provisioner with Triggers
resource "null_resource" "ansible_playbook" {
  triggers = {
    instance_ids = join(",", aws_instance.web[*].id)
  }

  provisioner "local-exec" {
    command = "ansible-playbook -i inventory.ini playbook.yml"
  }

  depends_on = [null_resource.generate_inventory]
}
```

### **5. Custom Providers and Plugins**
```hcl
# Custom Provider Configuration
terraform {
  required_providers {
    custom = {
      source  = "example.com/example/custom"
      version = "~> 1.0"
    }
  }
}

provider "custom" {
  endpoint = var.custom_api_endpoint
  api_key  = var.custom_api_key
}

# Using Custom Resources
resource "custom_database" "app" {
  name     = "sentinel-db"
  size     = "small"
  version  = "13.7"
  backups  = true

  settings = {
    timezone = "UTC"
    encoding = "UTF8"
  }
}
```

---

## 🌍 **Real-World Use Cases**

### **1. Multi-Environment Infrastructure**
```hcl
# environments/development/main.tf
module "network" {
  source = "../../modules/network"

  environment     = "development"
  vpc_cidr        = "10.0.0.0/16"
  az_count        = 2
  enable_nat_gw   = false  # Save costs in dev
}

module "compute" {
  source = "../../modules/compute"

  environment     = "development"
  vpc_id          = module.network.vpc_id
  subnet_ids      = module.network.public_subnet_ids
  instance_count  = 1
  instance_type   = "t3.micro"
}

# environments/production/main.tf
module "network" {
  source = "../../modules/network"

  environment     = "production"
  vpc_cidr        = "10.0.0.0/16"
  az_count        = 3
  enable_nat_gw   = true
}

module "compute" {
  source = "../../modules/compute"

  environment     = "production"
  vpc_id          = module.network.vpc_id
  subnet_ids      = module.network.private_subnet_ids
  instance_count  = 6
  instance_type   = "m5.large"
}

module "monitoring" {
  source = "../../modules/monitoring"

  environment = "production"
  vpc_id      = module.network.vpc_id
  alarm_sns_topic = module.notifications.sns_topic_arn
}
```

### **2. Microservices Infrastructure**
```hcl
# services/api-gateway/main.tf
module "api_gateway" {
  source = "../../modules/api-gateway"

  environment = var.environment
  vpc_id      = var.vpc_id

  services = {
    users    = module.users_service.endpoint
    orders   = module.orders_service.endpoint
    payments = module.payments_service.endpoint
  }

  domain_name = var.domain_name
  certificate_arn = var.certificate_arn
}

# services/users/main.tf
module "users_service" {
  source = "../../modules/ecs-service"

  name          = "users"
  environment   = var.environment
  vpc_id        = var.vpc_id
  subnet_ids    = var.private_subnet_ids

  container_image = "${var.ecr_repository_url}:latest"
  container_port  = 3000

  desired_count = var.desired_count
  cpu           = 256
  memory        = 512

  environment_variables = {
    DATABASE_URL = var.database_url
    REDIS_URL    = var.redis_url
    JWT_SECRET   = var.jwt_secret
  }

  secrets = {
    DATABASE_PASSWORD = var.database_password_secret_arn
  }
}

# databases/main.tf
module "rds" {
  source = "../../modules/rds"

  environment     = var.environment
  vpc_id          = var.vpc_id
  subnet_ids      = var.database_subnet_ids

  identifier      = "sentinel-${var.environment}"
  engine          = "aurora-postgresql"
  engine_version  = "13.7"
  instance_class  = var.instance_class

  database_name   = "sentinel"
  master_username = var.master_username
  master_password = var.master_password

  backup_retention_period = var.backup_retention_period
  preferred_backup_window = "03:00-04:00"
  preferred_maintenance_window = "sun:04:00-sun:05:00"

  monitoring_role_arn = var.monitoring_role_arn
  monitoring_interval = 60
}
```

### **3. Infrastructure Testing Suite**
```hcl
# test/integration/main.tf
terraform {
  required_providers {
    test = {
      source = "terraform.io/builtin/test"
    }
  }
}

resource "test_assertions" "vpc" {
  component = "vpc"

  equal "vpc_cidr" {
    value = aws_vpc.main.cidr_block
    expected = "10.0.0.0/16"
  }

  check "subnet_count" {
    assert = length(aws_subnet.public) == 3
  }
}

resource "test_assertions" "security" {
  component = "security_groups"

  check "web_sg_ports" {
    assert = contains([80, 443], aws_security_group.web.ingress[0].from_port)
  }

  check "database_sg_restricted" {
    assert = aws_security_group.database.ingress[0].cidr_blocks[0] != "0.0.0.0/0"
  }
}

# Integration test
run "integration_test" {
  assert {
    condition = output.load_balancer_dns != ""
    error_message = "Load balancer DNS should not be empty"
  }

  assert {
    condition = can(cidrhost(output.vpc_cidr, 1))
    error_message = "VPC CIDR should be valid"
  }
}
```

### **4. Cost Optimization**
```hcl
# Cost-aware resource selection
locals {
  instance_types = {
    development = "t3.micro"
    staging     = "t3.medium"
    production  = "m5.large"
  }

  # Use spot instances for non-production
  use_spot_instances = var.environment != "production"
}

resource "aws_instance" "app" {
  count = var.instance_count

  ami           = var.ami_id
  instance_type = local.instance_types[var.environment]

  # Spot instance configuration for cost savings
  dynamic "instance_market_options" {
    for_each = local.use_spot_instances ? [1] : []

    content {
      market_type = "spot"
      spot_options {
        max_price = var.spot_max_price
        spot_instance_type = "persistent"
      }
    }
  }

  tags = {
    Name        = "sentinel-app-${var.environment}-${count.index + 1}"
    Environment = var.environment
    CostCenter  = "platform"
  }
}

# Auto-scaling for cost efficiency
resource "aws_appautoscaling_target" "ecs_target" {
  max_capacity       = 10
  min_capacity       = 1
  resource_id        = "service/${aws_ecs_cluster.main.name}/${aws_ecs_service.app.name}"
  scalable_dimension = "ecs:service:DesiredCount"
  service_namespace  = "ecs"
}

resource "aws_appautoscaling_policy" "ecs_policy_cpu" {
  name               = "cpu-autoscaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.ecs_target.resource_id
  scalable_dimension = aws_appautoscaling_target.ecs_target.scalable_dimension
  service_namespace  = aws_appautoscaling_target.ecs_target.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }
    target_value = 70.0
  }
}

# Scheduled scaling for predictable workloads
resource "aws_appautoscaling_scheduled_action" "scale_down" {
  name               = "scale-down-business-hours"
  service_namespace  = aws_appautoscaling_target.ecs_target.service_namespace
  resource_id        = aws_appautoscaling_target.ecs_target.resource_id
  scalable_dimension = aws_appautoscaling_target.ecs_target.scalable_dimension
  schedule           = "cron(0 18 * * ? *)"  # 6 PM daily

  scalable_target_action {
    min_capacity = 1
    max_capacity = 3
  }
}
```

---

## 🎯 **Conclusion**

Terraform is the **de facto standard** for Infrastructure as Code, enabling teams to:

### **✅ Strengths:**
- **Declarative**: Define what you want, Terraform handles the how
- **Multi-Cloud**: Single tool for AWS, Azure, GCP, and more
- **Version Control**: Infrastructure changes tracked in Git
- **Modular**: Reusable components and modules
- **State Management**: Tracks resource state and dependencies
- **Planning**: Preview changes before applying
- **Idempotent**: Safe to run multiple times

### **🔧 Best Practices:**
- **Use modules** for reusable infrastructure components
- **Implement remote state** with locking for team collaboration
- **Validate configurations** before applying changes
- **Use workspaces** for environment separation
- **Follow naming conventions** and tagging standards
- **Implement security scanning** in CI/CD pipelines
- **Document your infrastructure** with comments and READMEs

### **🚀 Production Ready:**
Terraform powers infrastructure for:
- **Netflix** (large-scale AWS deployments)
- **Spotify** (microservices infrastructure)
- **Uber** (global infrastructure management)
- **Capital One** (cloud migration and governance)
- **Atlassian** (infrastructure automation)

This comprehensive guide covers everything from basic concepts to advanced production deployment patterns, making it the ultimate Terraform reference for infrastructure engineers and DevOps teams! 🚀

---

## 📚 **Additional Resources**

- [Terraform Official Documentation](https://www.terraform.io/docs)
- [Terraform Registry](https://registry.terraform.io/)
- [Terraform Best Practices](https://www.terraform.io/docs/cloud/guides/recommended-practices/index.html)
- [Terraform Language Documentation](https://www.terraform.io/language)
- [HashiCorp Learn](https://learn.hashicorp.com/terraform)
- [Terraform GitHub Repository](https://github.com/hashicorp/terraform)
- [Terraform Community Forum](https://discuss.hashicorp.com/c/terraform-core/27)
- [Awesome Terraform](https://github.com/shuaibiyy/awesome-terraform)
