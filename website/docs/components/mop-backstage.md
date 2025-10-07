---
sidebar_position: 5
---

# MOP Backstage

**mop-backstage** provides a developer portal and service catalog powered by Backstage.

## Overview

mop-backstage serves as the central hub for developers to discover services, access documentation, and interact with the observability platform.

## Features

### Service Catalog

Discover all services in your organization:

- Service metadata and ownership
- API documentation
- Links to dashboards and alerts
- Dependency graphs

### Documentation

Centralized documentation portal:

- Technical documentation (TechDocs)
- Architecture diagrams
- Runbooks and troubleshooting guides
- Best practices

### Observability Integration

Direct links to observability resources:

- Service-specific Grafana dashboards
- Alert status and history
- Log queries for each service
- Metrics explorer

### Developer Tools

Built-in tools for developers:

- Create new services from templates
- Deploy to monitoring clusters
- Configure alerts and dashboards
- Access CI/CD pipelines

## Service Catalog Structure

### Service Definitions

Services are defined in `catalog-info.yaml`:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: my-service
  description: A microservice that handles user authentication
  annotations:
    grafana.com/dashboard-url: https://grafana.example.com/d/my-service
    prometheus.io/rule: https://prometheus.example.com/alerts#my-service
spec:
  type: service
  lifecycle: production
  owner: platform-team
  system: authentication
```

### Integration with MOP

Backstage connects to MOP components:

```yaml
# Observability links
annotations:
  mop.platform/grafana-dashboard: my-service-overview
  mop.platform/alert-rules: my-service-alerts
  mop.platform/log-query: '{service="my-service"}'
```

## Configuration

### Backend Integration

Configure mop-backstage to connect to your MOP platform:

```yaml
# app-config.yaml
backend:
  baseUrl: http://localhost:7007

integrations:
  grafana:
    - url: https://grafana.mop-system.svc.cluster.local
  prometheus:
    - url: https://prometheus.mop-system.svc.cluster.local
```

### Authentication

Integrate with your identity provider:

```yaml
auth:
  providers:
    google:
      development:
        clientId: ${GOOGLE_CLIENT_ID}
        clientSecret: ${GOOGLE_CLIENT_SECRET}
```

## Plugins

### MOP-Specific Plugins

- **MOP Dashboard Plugin**: Embed Grafana dashboards
- **MOP Alerts Plugin**: View alert status
- **MOP Logs Plugin**: Quick log access

### Community Plugins

Useful Backstage plugins:

- **Kubernetes Plugin**: View cluster resources
- **GitLab/GitHub Plugin**: Repository integration
- **PagerDuty Plugin**: On-call information
- **Cost Insights Plugin**: Resource costs

## User Workflows

### Discovering Services

1. Browse the service catalog
2. Search by name, team, or technology
3. View service details and dependencies
4. Access observability resources

### Creating Services

1. Choose a template
2. Provide service details
3. Generate repository and manifests
4. Automatically configure monitoring

### Troubleshooting

1. Navigate to service in catalog
2. Check alert status
3. View recent logs
4. Access service dashboard
5. Contact service owner

## Deployment

### Prerequisites

```bash
node --version  # >= 18.x
yarn --version  # >= 1.22.x
```

### Local Development

```bash
cd mop-backstage
yarn install
yarn dev
```

### Production Deployment

```bash
yarn build
yarn build-image
kubectl apply -f kubernetes/
```

## Benefits

- **Single Source of Truth**: Centralized service information
- **Improved Discovery**: Easy to find services and resources
- **Better Collaboration**: Clear ownership and contact information
- **Faster Onboarding**: New developers can quickly understand the system
- **Integrated Experience**: Seamless access to observability tools

## Related Components

- [mop-core](./mop-core.md): Provides observability data
- [mop-ux](./mop-ux.md): Custom dashboards accessible through Backstage
- [mop-spill](./mop-spill.md): Sample applications in the catalog
