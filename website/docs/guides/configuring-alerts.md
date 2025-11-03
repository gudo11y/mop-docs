---
sidebar_position: 2
---

# Configuring Alerts

Learn how to configure effective alerts that notify you of issues without causing alert fatigue.

## Alert Philosophy

### Alert on Symptoms, Not Causes

✅ **Good**: "Users are experiencing errors"
❌ **Bad**: "CPU usage is high"

High CPU might not impact users. Alert on user-facing symptoms.

### The Four Golden Signals

Focus alerts on:

1. **Latency**: How long requests take
2. **Traffic**: Demand on your system
3. **Errors**: Rate of failed requests
4. **Saturation**: Resource utilization

## Alert Rules

### Basic Structure

```yaml
groups:
  - name: my-service-alerts
    interval: 30s
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{service="my-service",status=~"5.."}[5m]))
          /
          sum(rate(http_requests_total{service="my-service"}[5m]))
          > 0.05
        for: 5m
        labels:
          severity: warning
          team: platform
          service: my-service
        annotations:
          summary: "High error rate on {{ $labels.service }}"
          description: "Error rate is {{ $value | humanizePercentage }} (threshold: 5%)"
          runbook_url: "https://docs.example.com/runbooks/high-error-rate"
```

### Key Components

- **expr**: PromQL query that triggers the alert
- **for**: Duration before alert fires
- **labels**: Metadata for routing and grouping
- **annotations**: Human-readable context

## Common Alert Rules

### Error Rate Alert

```yaml
- alert: HighErrorRate
  expr: |
    (
      sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
      /
      sum(rate(http_requests_total[5m])) by (service)
    ) > 0.05
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Service {{ $labels.service }} has high error rate"
    description: "Error rate: {{ $value | humanizePercentage }}"
```

### High Latency Alert

```yaml
- alert: HighLatency
  expr: |
    histogram_quantile(0.95,
      sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service)
    ) > 1.0
  for: 10m
  labels:
    severity: warning
  annotations:
    summary: "Service {{ $labels.service }} has high latency"
    description: "P95 latency: {{ $value }}s (threshold: 1s)"
```

### Service Down Alert

```yaml
- alert: ServiceDown
  expr: up{job="my-service"} == 0
  for: 2m
  labels:
    severity: critical
  annotations:
    summary: "Service {{ $labels.job }} is down"
    description: "Service has been down for more than 2 minutes"
```

### Resource Saturation Alert

```yaml
- alert: HighMemoryUsage
  expr: |
    (
      container_memory_usage_bytes{pod=~"my-service-.*"}
      /
      container_spec_memory_limit_bytes{pod=~"my-service-.*"}
    ) > 0.9
  for: 15m
  labels:
    severity: warning
  annotations:
    summary: "Pod {{ $labels.pod }} has high memory usage"
    description: "Memory usage: {{ $value | humanizePercentage }}"
```

## Alert Severity Levels

### Critical

System is unusable or severely degraded:

```yaml
severity: critical
```

Examples:
- Service is down
- 100% error rate
- Data loss occurring

**Action**: Page on-call engineer immediately

### Warning

System degraded but functional:

```yaml
severity: warning
```

Examples:
- High error rate (5-20%)
- Elevated latency
- High resource usage

**Action**: Notify team channel, investigate during business hours

### Info

Informational, no immediate action:

```yaml
severity: info
```

Examples:
- Deployment started
- Configuration changed
- Normal threshold exceeded

**Action**: Log for awareness

## Alertmanager Configuration

### Routing

Route alerts based on labels:

```yaml
route:
  receiver: default
  group_by: ['alertname', 'service']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h

  routes:
    # Critical alerts to PagerDuty
    - match:
        severity: critical
      receiver: pagerduty
      continue: true

    # Team-specific routing
    - match:
        team: platform
      receiver: platform-slack

    - match:
        team: data
      receiver: data-slack
```

### Receivers

#### Slack

```yaml
receivers:
  - name: platform-slack
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
        channel: '#platform-alerts'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
```

#### PagerDuty

```yaml
receivers:
  - name: pagerduty
    pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_KEY'
        description: '{{ .GroupLabels.alertname }}'
```

#### Email

```yaml
receivers:
  - name: email
    email_configs:
      - to: 'team@example.com'
        from: 'alerts@example.com'
        smarthost: 'smtp.gmail.com:587'
        auth_username: 'alerts@example.com'
        auth_password: 'YOUR_PASSWORD'
```

## Alert Best Practices

### Writing Good Alerts

1. **Include context**: Link to runbooks and dashboards
2. **Make them actionable**: Clear what to do
3. **Set appropriate thresholds**: Based on SLOs
4. **Use appropriate durations**: Avoid flapping

### Reducing Alert Fatigue

1. **Alert on symptoms**: Not causes
2. **Group related alerts**: Use `group_by`
3. **Set repeat intervals**: Don't spam
4. **Use inhibition rules**: Silence dependent alerts

### Testing Alerts

```bash
# Generate test alert
amtool alert add alertname=TestAlert \
  severity=warning \
  summary="This is a test"

# Check alert status
amtool alert query
```

## Inhibition Rules

Silence alerts based on other alerts:

```yaml
inhibit_rules:
  # Don't alert on high latency if service is down
  - source_match:
      alertname: ServiceDown
    target_match:
      alertname: HighLatency
    equal: ['service']

  # Don't alert on warnings if critical alert exists
  - source_match:
      severity: critical
    target_match:
      severity: warning
    equal: ['service', 'alertname']
```

## Silences

Temporarily mute alerts:

```bash
# Silence alert for maintenance window
amtool silence add \
  alertname=HighLatency \
  service=my-service \
  --duration=2h \
  --comment="Scheduled maintenance"

# List active silences
amtool silence query

# Expire silence
amtool silence expire <silence_id>
```

## Alert Templates

### Slack Template

```yaml
slack_configs:
  - api_url: 'YOUR_WEBHOOK'
    title: |
      [{{ .Status | toUpper }}] {{ .GroupLabels.alertname }}
    text: |
      *Summary:* {{ .CommonAnnotations.summary }}
      *Description:* {{ .CommonAnnotations.description }}
      *Severity:* {{ .CommonLabels.severity }}
      *Runbook:* {{ .CommonAnnotations.runbook_url }}
      *Dashboard:* https://grafana.example.com/d/{{ .CommonLabels.service }}
```

## Troubleshooting

### Alert Not Firing

1. Check expression in Prometheus UI
2. Verify `for` duration hasn't elapsed
3. Check Alertmanager logs
4. Verify routing configuration

### Too Many Alerts

1. Review alert thresholds
2. Increase `for` duration
3. Use inhibition rules
4. Consolidate related alerts

### Alert Routing Issues

1. Check label matchers
2. Verify receiver configuration
3. Test with `amtool`
4. Check Alertmanager logs

## Next Steps

- [Creating Dashboards](./creating-dashboards.md) - Learn how to visualize your metrics
- [Deploying Applications](./deploying-applications.md) - Deploy monitored applications
