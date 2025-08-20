---
title: "Infrastructure as Code: Setting Up EKS and RDS with Terraform in Minutes"
datePublished: Wed Aug 13 2025 05:00:48 GMT+0000 (Coordinated Universal Time)
cuid: cmejkq6ki000502l441zi23ix
slug: infrastructure-as-code-setting-up-eks-and-rds-with-terraform-in-minutes-fee68ac34e29
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1755670231830/c5ea6737-16a8-4b4b-ad41-370c73377c3a.png

---

In today’s time of automation managing infrastructure manually is time-consuming and difficult to scale. **Terraform**, the popular Infrastructure as Code (IaC) tool, allows us to provision, manage, and version your cloud resources declaratively.

In this guide, i’ll walk through building a fully functional **EKS (Elastic Kubernetes Service) cluster** and a **PostgreSQL RDS database** on AWS using Terraform.

### Why Use Terraform for AWS EKS and RDS?

* **Consistency:** Deploy the same environment repeatedly without manual mistakes.
    
* **Version Control:** Keep infrastructure changes in Git alongside your application code.
    
* **Automation:** Provision complex AWS resources like VPCs, EKS, and RDS in a single command.
    
* **Integration:** Easy to integrate with CI/CD pipelines for continuous delivery
    

### Prerequisites

Before we start, make sure you have:

* An AWS account with sufficient IAM permissions.
    
* Terraform installed (v1.6 or higher recommended).
    
* `kubectl` installed for interacting with the EKS cluster.
    
* Basic knowledge of AWS, Kubernetes, and PostgreSQL.
    

### Project Structure

Here’s how our Terraform project is organized:

```markdown
terraform/
├── main.tf # Provider and shared data sources
├── variables.tf # All configurable variables
├── outputs.tf # Outputs for EKS & RDS details
├── eks.tf # EKS cluster and node group resources
└── rds.tf # RDS PostgreSQL instance
```

### Step 1: Configure AWS Provider

In main.tf, we define the AWS provider and shared default VPC/subnets:

```markdown
terraform {
required_version = ">= 1.6.0"

required_providers {
aws = {
source = "hashicorp/aws"
version = "~> 5.0"
}
}
}

provider "aws" {
region = var.aws_region
}

# Shared default VPC and subnets for both EKS and RDS
data "aws_vpc" "default" {
default = true
}

data "aws_subnets" "default" {
filter {
name = "vpc-id"
values = [data.aws_vpc.default.id]
}

filter {
name = "availability-zone"
values = ["us-east-1a", "us-east-1b", "us-east-1c"] # only supported ones
}
}
```

### Step 2: Define Variables

All configurable parameters are stored in `variables.tf`:

```markdown
variable "aws_region" {
description = "AWS region to deploy resources in"
type = string
default = "us-east-1"
}

variable "eks_cluster_name" {
description = "Name of the EKS cluster"
type = string
default = "expensetracker"
}

variable "rds_instance_identifier" {
description = "RDS instance name"
type = string
default = "expense-tracker-db"
}

variable "rds_db_name" {
description = "Database name in RDS"
type = string
default = "expense_tracker"
}

variable "rds_username" {
description = "Master DB username"
type = string
default = "postgres"
}

variable "rds_password" {
description = "Master DB password"
type = string
sensitive = true
}

variable "rds_instance_class" {
description = "RDS instance type"
type = string
default = "db.t3.micro"
}
```

### Step 3: Create the EKS Cluster

The `eks.tf` file provisions:

* IAM roles for the cluster and node group
    
* The EKS cluster
    
* A managed node group
    

```markdown
# EKS Cluster IAM Role
resource "aws_iam_role" "eks_cluster_role" {
name = "${var.eks_cluster_name}-cluster-role"

assume_role_policy = jsonencode({
Version = "2012-10-17",
Statement = [{
Effect = "Allow",
Principal = {
Service = "eks.amazonaws.com"
},
Action = "sts:AssumeRole"
}]
})
}

resource "aws_iam_role_policy_attachment" "eks_cluster_policy" {
role = aws_iam_role.eks_cluster_role.name
policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
}

# EKS Cluster
resource "aws_eks_cluster" "this" {
name = var.eks_cluster_name
role_arn = aws_iam_role.eks_cluster_role.arn

vpc_config {
subnet_ids = data.aws_subnets.default.ids
}

depends_on = [aws_iam_role_policy_attachment.eks_cluster_policy]
}

# Node Group IAM Role
resource "aws_iam_role" "eks_node_role" {
name = "${var.eks_cluster_name}-node-role"

assume_role_policy = jsonencode({
Version = "2012-10-17",
Statement = [{
Effect = "Allow",
Principal = {
Service = "ec2.amazonaws.com"
},
Action = "sts:AssumeRole"
}]
})
}

resource "aws_iam_role_policy_attachment" "node_AmazonEKSWorkerNodePolicy" {
role = aws_iam_role.eks_node_role.name
policy_arn = "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy"
}

resource "aws_iam_role_policy_attachment" "node_AmazonEC2ContainerRegistryReadOnly" {
role = aws_iam_role.eks_node_role.name
policy_arn = "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
}

resource "aws_iam_role_policy_attachment" "node_AmazonEKS_CNI_Policy" {
role = aws_iam_role.eks_node_role.name
policy_arn = "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy"
}

# Node Group
resource "aws_eks_node_group" "default" {
cluster_name = aws_eks_cluster.this.name
node_group_name = "${var.eks_cluster_name}-node-group"
node_role_arn = aws_iam_role.eks_node_role.arn
subnet_ids = data.aws_subnets.default.ids

scaling_config {
desired_size = 2
max_size = 3
min_size = 1
}

instance_types = ["t2.large"]

depends_on = [
aws_iam_role_policy_attachment.node_AmazonEKSWorkerNodePolicy,
aws_iam_role_policy_attachment.node_AmazonEC2ContainerRegistryReadOnly,
aws_iam_role_policy_attachment.node_AmazonEKS_CNI_Policy
]
}
```

### Step 4: Create PostgreSQL RDS

The `rds.tf` provisions a **PostgreSQL database** with:

```markdown
# Security Group for RDS
resource "aws_security_group" "rds_sg" {
name = "rds-postgres-sg"
description = "Allow PostgreSQL access"
vpc_id = data.aws_vpc.default.id

ingress {
description = "PostgreSQL access"
from_port = 5432
to_port = 5432
protocol = "tcp"
cidr_blocks = ["0.0.0.0/0"]
}

egress {
from_port = 0
to_port = 0
protocol = "-1"
cidr_blocks = ["0.0.0.0/0"]
}
}

# Subnet group for RDS (uses the same filtered default-VPC subnets)
resource "aws_db_subnet_group" "default" {
name = "default-rds-subnet-group"
subnet_ids = data.aws_subnets.default.ids
}

# RDS PostgreSQL instance (no engine_version -> let AWS choose a supported one)
resource "aws_db_instance" "this" {
identifier = var.rds_instance_identifier
engine = "postgres"
instance_class = var.rds_instance_class
allocated_storage = 20

db_name = var.rds_db_name
username = var.rds_username
password = var.rds_password

publicly_accessible = true
skip_final_snapshot = true
db_subnet_group_name = aws_db_subnet_group.default.name
vpc_security_group_ids = [aws_security_group.rds_sg.id]

backup_retention_period = 7
storage_encrypted = true
deletion_protection = false
}
```

### Step 5: Outputs

Finally, we define outputs for easy access:

```markdown
output "eks_cluster_endpoint" {
description = "EKS cluster endpoint"
value = aws_eks_cluster.this.endpoint
}

output "eks_cluster_name" {
description = "EKS cluster name"
value = aws_eks_cluster.this.name
}

output "rds_endpoint" {
description = "RDS PostgreSQL endpoint"
value = aws_db_instance.this.endpoint
}
```

### Step 6: Deploy the Infrastructure

Run the following commands:

```markdown
terraform init
terraform plan -var="rds_password=YourStrongPassword123"
terraform apply -var="rds_password=YourStrongPassword123" -auto-approve
```

* Terraform will provision **EKS cluster**, **node group**, and **RDS instance**.
    
* Outputs will give the EKS API endpoint and PostgreSQL endpoint for connecting your apps.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1755670227193/efb7e735-e0a2-4faa-a838-1614abe90d17.png align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1755670229681/546ee949-42b0-4d74-a620-605f9eca9dfe.png align="left")

### Step 7: Connect EKS to RDS

1. Update your Kubernetes deployment with a **secret** for DB credentials:
    

```markdown
kubectl create secret generic db-credentials \
--from-literal=username=expensetrackeradmin \
--from-literal=password=YourStrongPassword123
```

Reference this secret in your app’s deployment manifest to connect to RDS.