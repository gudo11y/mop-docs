---
sidebar_position: 1
---

# MOP Core

**mop-core** is the heart of the MOP Platform, providing the foundational observability stack.

## Overview

mop-core bundles and configures industry-standard open-source observability tools into a cohesive platform that's ready for enterprise use.

## Components

### Prometheus

Metrics collection and time-series database:

- Scrapes metrics from instrumented applications
- Provides powerful query language (PromQL)
- Supports service discovery
- Integrates with Alertmanager

### Grafana

Visualization and dashboarding:

- Create and share dashboards
- Multi-datasource support (Prometheus, Loki, Tempo)
- Alerting and notifications
- User and team management

### Loki

Log aggregation system:

- Horizontally scalable
- Cost-effective log storage
- Label-based indexing
- LogQL query language

### Alertmanager

Alert routing and management:

- Deduplication and grouping
- Routing to various receivers (email, Slack, PagerDuty)
- Silencing and inhibition rules
- Alert lifecycle management

## Configuration

mop-core provides pre-configured setups optimized for:

- High availability
- Scalability
- Security best practices
- Resource efficiency

## Deployment

### Kubernetes

Deploy using Helm charts or Kubernetes manifests:

```bash
kubectl apply -f manifests/
```

### Resource Requirements

Minimum recommended resources:

- **Prometheus**: 4 CPU, 8GB RAM
- **Grafana**: 2 CPU, 4GB RAM
- **Loki**: 4 CPU, 8GB RAM
- **Alertmanager**: 1 CPU, 2GB RAM

## Monitoring the Monitor

mop-core includes self-monitoring capabilities:

- Platform health dashboards
- Resource usage tracking
- Performance metrics
- Alerting on platform issues

## Related Components

- [mop-cli](./mop-cli.md): Command-line tools for managing mop-core
- [mop-infra](./mop-infra.md): Infrastructure provisioning
- [mop-ux](./mop-ux.md): User customizations built on mop-core
