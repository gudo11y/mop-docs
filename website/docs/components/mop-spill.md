---
sidebar_position: 6
---

# MOP Spill

**mop-spill** contains sample applications that demonstrate how to deploy monitored workloads to the MOP Platform.

## Overview

mop-spill serves as a reference implementation showing how application teams should instrument and deploy their services to be observable by the MOP Platform.

## Purpose

mop-spill demonstrates:

- How to instrument applications for observability
- Integration with service discovery
- Best practices for metrics, logs, and traces
- Example deployments with monitoring annotations

## Sample Applications

### Example Service

A simple HTTP service demonstrating basic instrumentation:

```go
package main

import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    httpRequestsTotal = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total number of HTTP requests",
        },
        []string{"method", "endpoint", "status"},
    )
)

func init() {
    prometheus.MustRegister(httpRequestsTotal)
}
```

### Kubernetes Deployment

Deploy with monitoring annotations:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-service
  labels:
    app: example-service
spec:
  replicas: 3
  template:
    metadata:
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: app
        image: mop-spill/example-service:latest
        ports:
        - containerPort: 8080
          name: http
```

## Instrumentation

### Metrics

Export standard metrics:

- **RED metrics**: Rate, Errors, Duration
- **USE metrics**: Utilization, Saturation, Errors
- **Custom metrics**: Business-specific measurements

```python
from prometheus_client import Counter, Histogram, Gauge

# Request counter
requests_total = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

# Request duration
request_duration = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'endpoint']
)

# Active connections
active_connections = Gauge(
    'active_connections',
    'Number of active connections'
)
```

### Logging

Structured logging with context:

```javascript
const logger = require('pino')();

logger.info({
  service: 'example-service',
  method: 'GET',
  path: '/api/users',
  duration: 45,
  status: 200,
  user_id: '12345'
}, 'Request completed');
```

### Tracing (Optional)

Distributed tracing with OpenTelemetry:

```go
import (
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/trace"
)

func handleRequest(ctx context.Context) {
    tracer := otel.Tracer("example-service")
    ctx, span := tracer.Start(ctx, "handle_request")
    defer span.End()

    // Your business logic here
}
```

## Service Discovery

### Prometheus Service Discovery

Services are automatically discovered via Kubernetes service discovery:

```yaml
# Prometheus scrape config (in mop-core)
- job_name: 'kubernetes-pods'
  kubernetes_sd_configs:
    - role: pod
  relabel_configs:
    - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
      action: keep
      regex: true
```

### Labels and Tags

Consistent labeling for filtering and grouping:

```yaml
metadata:
  labels:
    app: example-service
    team: platform
    environment: production
    version: v1.2.3
```

## Example Applications

### 1. HTTP API Service

RESTful API with full instrumentation:

```
mop-spill/http-api/
├── main.go
├── Dockerfile
├── kubernetes/
│   ├── deployment.yaml
│   └── service.yaml
└── monitoring/
    ├── dashboard.json
    └── alerts.yaml
```

### 2. Worker Service

Background job processor:

```
mop-spill/worker/
├── worker.py
├── Dockerfile
├── kubernetes/
│   └── deployment.yaml
└── monitoring/
    ├── dashboard.json
    └── alerts.yaml
```

### 3. Database Application

Application with database metrics:

```
mop-spill/db-app/
├── app.js
├── Dockerfile
├── kubernetes/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
└── monitoring/
    ├── dashboard.json
    └── alerts.yaml
```

## Best Practices

### Instrumentation

- Expose `/metrics` endpoint in Prometheus format
- Use structured logging with consistent fields
- Include correlation IDs for request tracing
- Set appropriate metric labels

### Deployment

- Add Prometheus scrape annotations
- Set resource requests and limits
- Configure health checks
- Use rolling updates

### Monitoring

- Create service-specific dashboards
- Define SLIs and SLOs
- Set up meaningful alerts
- Document expected behavior

## Getting Started

### Deploy a Sample App

```bash
cd mop-spill/http-api
kubectl apply -f kubernetes/

# Verify deployment
kubectl get pods -l app=example-service

# Check metrics
kubectl port-forward svc/example-service 8080:8080
curl http://localhost:8080/metrics
```

### View in Grafana

1. Navigate to Grafana
2. Import the dashboard from `monitoring/dashboard.json`
3. View metrics for your service

### Test Alerts

```bash
# Generate high error rate
for i in {1..100}; do
  curl http://example-service/error
done

# Check Alertmanager
kubectl port-forward -n mop-system svc/alertmanager 9093:9093
```

## Related Components

- [mop-core](./mop-core.md): Collects metrics from mop-spill applications
- [mop-ux](./mop-ux.md): Dashboards for visualizing mop-spill metrics
- [mop-backstage](./mop-backstage.md): Service catalog entries for mop-spill apps
