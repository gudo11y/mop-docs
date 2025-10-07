---
sidebar_position: 1
---

# Creating Dashboards

Learn how to create effective Grafana dashboards for your services.

## Overview

Dashboards provide visual insights into your system's health and performance. This guide covers best practices for creating useful, maintainable dashboards.

## Dashboard Structure

### Information Hierarchy

Organize information from general to specific:

1. **Overview Row**: High-level health indicators (RED metrics)
2. **Details Rows**: Detailed metrics by component
3. **Debugging Row**: Low-level metrics for troubleshooting

### Example Layout

```
┌─────────────────────────────────────────────────┐
│ Service Health (UP/DOWN)  | Request Rate | SLO │
├─────────────────────────────────────────────────┤
│ Request Rate    | Error Rate    | P95 Latency   │
├─────────────────────────────────────────────────┤
│ CPU Usage       | Memory Usage  | Disk I/O      │
├─────────────────────────────────────────────────┤
│ Active Connections | Queue Depth | Cache Hit % │
└─────────────────────────────────────────────────┘
```

## Creating Your First Dashboard

### 1. Access Grafana

```bash
# Port forward to Grafana
kubectl port-forward -n mop-system svc/grafana 3000:80

# Open in browser
open http://localhost:3000
```

### 2. Create New Dashboard

1. Click **+ → Dashboard**
2. Click **Add new panel**
3. Configure your first panel

### 3. Add a Panel

#### Request Rate Panel

```promql
sum(rate(http_requests_total{service="my-service"}[5m]))
```

Settings:
- **Panel Title**: "Request Rate"
- **Unit**: "requests/sec"
- **Legend**: "{{method}} {{endpoint}}"

#### Error Rate Panel

```promql
sum(rate(http_requests_total{service="my-service",status=~"5.."}[5m]))
/
sum(rate(http_requests_total{service="my-service"}[5m]))
```

Settings:
- **Panel Title**: "Error Rate"
- **Unit**: "percentunit"
- **Thresholds**:
  - Green: < 0.01 (1%)
  - Yellow: < 0.05 (5%)
  - Red: ≥ 0.05

#### Latency Panel

```promql
histogram_quantile(0.95,
  sum(rate(http_request_duration_seconds_bucket{service="my-service"}[5m])) by (le)
)
```

Settings:
- **Panel Title**: "P95 Latency"
- **Unit**: "seconds"
- **Thresholds**:
  - Green: < 0.1s
  - Yellow: < 0.5s
  - Red: ≥ 0.5s

## Dashboard Variables

### Create Variables for Filtering

Add variables to make dashboards reusable:

#### Service Variable

- **Name**: `service`
- **Type**: Query
- **Query**: `label_values(http_requests_total, service)`
- **Multi-value**: No

#### Environment Variable

- **Name**: `environment`
- **Type**: Query
- **Query**: `label_values(http_requests_total, environment)`
- **Multi-value**: No

#### Use in Queries

```promql
sum(rate(http_requests_total{service="$service", environment="$environment"}[5m]))
```

## Dashboard Templates

### RED Metrics Template

Every service should have RED metrics:

```json
{
  "title": "${service} - Overview",
  "rows": [
    {
      "title": "RED Metrics",
      "panels": [
        {
          "title": "Request Rate",
          "targets": [{
            "expr": "sum(rate(http_requests_total{service=\"$service\"}[5m]))"
          }]
        },
        {
          "title": "Error Rate",
          "targets": [{
            "expr": "sum(rate(http_requests_total{service=\"$service\",status=~\"5..\"}[5m])) / sum(rate(http_requests_total{service=\"$service\"}[5m]))"
          }]
        },
        {
          "title": "Duration (P95)",
          "targets": [{
            "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket{service=\"$service\"}[5m])) by (le))"
          }]
        }
      ]
    }
  ]
}
```

## Best Practices

### Query Performance

- **Use recording rules** for complex queries
- **Limit time ranges** for expensive queries
- **Use appropriate intervals**: `[5m]` for most, `[1m]` for critical
- **Avoid regex** in metric names when possible

### Visual Design

- **Consistent colors**: Use theme colors
- **Appropriate visualization**: Line charts for time series, gauges for current state
- **Clear titles**: Descriptive panel titles
- **Units**: Always set appropriate units

### Dashboard Organization

- **Logical grouping**: Group related metrics
- **Rows with collapse**: Use rows for organization
- **Consistent layouts**: Similar panels should have similar sizes
- **Link to related dashboards**: Use links in panel descriptions

## Dashboard as Code

### Export Dashboard

```bash
# Export dashboard JSON
curl -H "Authorization: Bearer $GRAFANA_API_KEY" \
  http://localhost:3000/api/dashboards/uid/my-dashboard > dashboard.json
```

### Version Control

Store dashboards in Git:

```
mop-ux/dashboards/
├── service-overview.json
├── platform-health.json
└── user-metrics.json
```

### Import Dashboard

```bash
# Import via API
curl -X POST -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GRAFANA_API_KEY" \
  -d @dashboard.json \
  http://localhost:3000/api/dashboards/db
```

## Troubleshooting

### No Data in Panel

1. Check query in Explore
2. Verify time range
3. Confirm metric exists in Prometheus
4. Check label filters

### Slow Dashboard

1. Reduce time range
2. Simplify queries
3. Use recording rules
4. Increase panel interval

### Dashboard Not Found

1. Check folder permissions
2. Verify dashboard UID
3. Ensure datasource is configured

## Next Steps

- [Configuring Alerts](./configuring-alerts.md)
- [Advanced PromQL](./advanced-promql.md)
- [Dashboard Templates](./dashboard-templates.md)
