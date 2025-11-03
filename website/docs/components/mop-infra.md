---
sidebar_position: 3
---

# MOP Infrastructure

**mop-infra** contains Infrastructure-as-Code (IaC) for provisioning and managing MOP Platform resources.

## Overview

mop-infra uses Terraform and Kubernetes manifests to define and provision the infrastructure required for running the MOP Platform.

## Structure

```
mop-infra/
├── terraform/           # Cloud infrastructure
│   ├── aws/            # AWS resources
│   ├── gcp/            # GCP resources
│   └── azure/          # Azure resources
├── kubernetes/          # Kubernetes manifests
│   ├── namespaces/
│   ├── storage/
│   └── networking/
└── helm/               # Helm chart configurations
```

## Terraform Modules

### Cloud Resources

Provisions cloud-specific resources:

- Kubernetes clusters (EKS, GKE, AKS)
- Storage buckets for long-term data
- Load balancers and ingress
- IAM roles and service accounts
- Networking (VPCs, subnets, security groups)

### Example: AWS Deployment

```hcl
module "mop_cluster" {
  source = "./terraform/aws/eks"

  cluster_name    = "mop-production"
  cluster_version = "1.28"

  node_groups = {
    monitoring = {
      instance_types = ["m5.xlarge"]
      desired_size   = 3
      min_size       = 2
      max_size       = 5
    }
  }
}

module "mop_storage" {
  source = "./terraform/aws/s3"

  bucket_name = "mop-longterm-metrics"
  lifecycle_rules = {
    archive_after_days = 90
    delete_after_days  = 365
  }
}
```

## Kubernetes Manifests

### Namespace Setup

Creates necessary namespaces:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: mop-system
  labels:
    name: mop-system
    managed-by: mop-infra
```

### Storage Classes

Configures storage for persistent data:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: mop-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  fsType: ext4
```

## Deployment

### Prerequisites

```bash
terraform version  # >= 1.5.0
kubectl version    # >= 1.24.0
helm version       # >= 3.0.0
```

### Provisioning Infrastructure

```bash
cd terraform/aws  # or gcp/azure

# Initialize Terraform
terraform init

# Review planned changes
terraform plan

# Apply changes
terraform apply
```

### Deploying Kubernetes Resources

```bash
cd kubernetes

# Apply namespaces
kubectl apply -f namespaces/

# Apply storage
kubectl apply -f storage/

# Apply networking
kubectl apply -f networking/
```

## Environment Management

### Workspaces

Use Terraform workspaces for different environments:

```bash
terraform workspace new production
terraform workspace new staging
terraform workspace select production
```

### Variables

Environment-specific variables in `terraform.tfvars`:

```hcl
environment = "production"
region      = "us-west-2"

cluster_config = {
  node_count       = 5
  node_type        = "m5.2xlarge"
  enable_autoscale = true
}

monitoring_config = {
  prometheus_retention = "30d"
  loki_retention      = "7d"
}
```

## GitOps Integration

mop-infra is designed for GitOps workflows:

1. Infrastructure changes are committed to Git
2. CI/CD pipelines validate changes
3. Approved changes are applied automatically
4. State is tracked in remote backends

## Related Components

- [mop-core](./mop-core.md): Deployed on infrastructure provisioned by mop-infra
- [mop-cli](./mop-cli.md): Tools for managing infrastructure
