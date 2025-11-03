---
sidebar_position: 1
---

# Introduction

Welcome to the **MOP Platform** (Modern Observability Platform) documentation!

## What is MOP?

MOP is a comprehensive observability platform designed to provide enterprise-grade monitoring, logging, and alerting capabilities. It demonstrates a typical enterprise setup where platform teams maintain the observability stack while end users customize their experience.

## Platform Components

The MOP ecosystem consists of several key components:

- **mop-core**: The observability stack (Prometheus, Grafana, Loki, and more)
- **mop-cli**: Command-line tools for platform management
- **mop-infra**: Infrastructure-as-code (Terraform, Kubernetes manifests)
- **mop-backstage**: Developer portal and service catalog
- **mop-ux**: User-facing alerts and dashboards (customer workspace)
- **mop-spill**: Sample applications deployed to monitored clusters
- **mop-docs**: This documentation site

## Quick Start

To get started with MOP, check out our [Installation Guide](./installation.md) to learn how to deploy the platform in your environment.

## Architecture Overview

MOP follows a three-tier architecture:

1. **Platform Team**: Maintains the observability stack (mop-core)
2. **End Users**: Customize their observability experience (mop-ux)
3. **Application Teams**: Deploy monitored workloads (mop-spill)

For more details, see the [Architecture](../architecture/overview.md) section.
