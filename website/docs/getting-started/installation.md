---
sidebar_position: 2
---

# Installation

This guide will walk you through installing and setting up the MOP Platform.

## Prerequisites

Before installing MOP, ensure you have the following:

- Kubernetes cluster (v1.24+)
- `kubectl` configured to access your cluster
- Helm 3.x installed
- Terraform (for infrastructure provisioning)
- Access to a container registry

## Installation Steps

### 1. Clone the Repositories

```bash
git clone https://github.com/mop-platform/mop-core.git
git clone https://github.com/mop-platform/mop-infra.git
git clone https://github.com/mop-platform/mop-cli.git
```

### 2. Deploy Infrastructure

Use the mop-infra repository to provision the necessary infrastructure:

```bash
cd mop-infra
terraform init
terraform plan
terraform apply
```

### 3. Install MOP Core

Deploy the observability stack using Helm or the provided manifests:

```bash
cd mop-core
# Follow the installation instructions in the mop-core README
```

### 4. Install MOP CLI

Install the command-line tools for managing the platform:

```bash
cd mop-cli
# Follow the installation instructions in the mop-cli README
```

### 5. Verify Installation

Check that all components are running:

```bash
kubectl get pods -n mop-system
```

## Next Steps

- [Configure your first dashboard](../guides/creating-dashboards.md)
- [Set up alerts](../guides/configuring-alerts.md)
- [Deploy a sample application](../guides/deploying-applications.md)
