---
sidebar_position: 4
---

# MOP UX

**mop-ux** represents the user-facing workspace for customizing alerts and dashboards.

## Overview

mop-ux demonstrates how end users can customize their observability experience without modifying the core platform. It serves as a reference implementation for user workspaces.

## Purpose

While platform teams maintain mop-core, individual users and teams need to:

- Create custom dashboards for their services
- Set up team-specific alerts
- Define personalized views and queries
- Share visualizations within their teams

mop-ux shows how to accomplish this while maintaining separation from the platform layer.

## Structure

```
mop-ux/
├── dashboards/         # Custom Grafana dashboards
├── alerts/            # User-defined alert rules
├── queries/           # Saved PromQL and LogQL queries
└── templates/         # Reusable dashboard templates
```

## Custom Dashboards

### Creating Dashboards

Users can create dashboards in several ways:

1. **Through Grafana UI**: Create dashboards interactively
2. **JSON Definitions**: Define dashboards as code
3. **Templates**: Use pre-built templates

Example dashboard definition:

```json
{
  "dashboard": {
    "title": "My Service Dashboard",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total{service='my-service'}[5m])"
          }
        ]
      }
    ]
  }
}
```

### Dashboard Organization

Organize dashboards by:

- Team or service
- Environment (dev, staging, prod)
- Purpose (overview, debugging, capacity)

## Custom Alerts

### Alert Rules

Define team-specific alerts:

```yaml
groups:
  - name: my-service-alerts
    rules:
      - alert: HighErrorRate
        expr: |
          rate(http_requests_total{service="my-service",status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: warning
          team: platform
        annotations:
          summary: "High error rate for my-service"
          description: "Error rate is {{ $value }} requests/sec"
```

### Alert Routing

Configure where alerts are sent:

```yaml
route:
  receiver: team-slack
  group_by: ['alertname', 'service']
  routes:
    - match:
        severity: critical
      receiver: team-pagerduty
```

## Saved Queries

### PromQL Queries

Store frequently used queries:

```promql
# Service health
up{service="my-service"} == 1

# Error rate
rate(http_requests_total{status=~"5.."}[5m])

# P95 latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

### LogQL Queries

Common log queries:

```logql
# Error logs
{service="my-service"} |= "error"

# Performance logs
{service="my-service"} | json | latency > 1000
```

## Best Practices

### Dashboard Design

- **Focus on metrics that matter**: Don't overwhelm with data
- **Use consistent naming**: Follow platform conventions
- **Add documentation**: Describe what each panel shows
- **Set appropriate time ranges**: Default to relevant timeframes

### Alert Design

- **Alert on symptoms, not causes**: Focus on user impact
- **Avoid alert fatigue**: Be selective about what triggers alerts
- **Include context**: Add helpful annotations and runbooks
- **Test alerts**: Verify they fire when expected

## Integration with mop-core

mop-ux integrates seamlessly with mop-core:

- Dashboards query Prometheus in mop-core
- Alerts are evaluated by Prometheus
- Logs are retrieved from Loki
- All data sources are pre-configured

## Version Control

All customizations are stored in Git:

- Track changes over time
- Review before applying
- Roll back when needed
- Share across teams

## Related Components

- [mop-core](./mop-core.md): Provides the data sources for mop-ux
- [mop-backstage](./mop-backstage.md): Portal for discovering dashboards
