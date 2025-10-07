---
sidebar_position: 1
---

# Architecture Overview

The MOP Platform follows a modular architecture designed to separate concerns between platform operations, user customization, and application deployment.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Developer Portal                   │
│                   (mop-backstage)                    │
└─────────────────────────────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          │               │               │
┌─────────▼────────┐ ┌───▼────────┐ ┌───▼─────────┐
│   Platform Team  │ │  End Users │ │   App Teams │
│   (mop-core)     │ │  (mop-ux)  │ │ (mop-spill) │
└──────────────────┘ └────────────┘ └─────────────┘
```

## Three-Tier Model

### 1. Platform Layer (mop-core)

The platform layer provides the foundational observability stack:

- **Prometheus**: Metrics collection and storage
- **Grafana**: Visualization and dashboarding
- **Loki**: Log aggregation
- **Alertmanager**: Alert routing and management
- **Tempo**: Distributed tracing (optional)

Platform teams maintain this layer and ensure its reliability and performance.

### 2. User Customization Layer (mop-ux)

End users can customize their observability experience:

- Custom dashboards
- User-specific alerts
- Team-specific views
- Custom queries and visualizations

This layer demonstrates how users interact with the platform without modifying core components.

### 3. Application Layer (mop-spill)

Application teams deploy monitored workloads:

- Sample applications with instrumentation
- Service discovery integration
- Automatic metrics collection
- Log forwarding configuration

## Component Interactions

### Data Flow

1. Applications (mop-spill) emit metrics, logs, and traces
2. Platform components (mop-core) collect and store this data
3. Users (mop-ux) visualize and alert on the data
4. Developer portal (mop-backstage) provides unified access

### Infrastructure Management

- **mop-infra**: Provisions cloud resources and Kubernetes clusters
- **mop-cli**: Provides command-line tools for operations
- **mop-docs**: Documents the entire system

## Design Principles

1. **Separation of Concerns**: Clear boundaries between platform, users, and applications
2. **GitOps-Ready**: All configurations stored in Git
3. **Cloud-Native**: Built for Kubernetes and cloud environments
4. **Extensible**: Easy to add new components and integrations
5. **Observable by Default**: Built-in instrumentation and monitoring

## Next Steps

See the [Components](../components/mop-core.md) section for detailed information about each MOP component.
