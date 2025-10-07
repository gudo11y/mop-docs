---
sidebar_position: 3
---

# Deploying Applications

Learn how to deploy applications that are automatically monitored by the MOP Platform.

## Overview

When you deploy applications to clusters monitored by MOP, they can be automatically discovered and monitored if properly instrumented.

## Prerequisites

- Access to a Kubernetes cluster with MOP installed
- `kubectl` configured to access the cluster
- Application container image
- Basic understanding of Kubernetes

## Instrumentation Checklist

Before deploying, ensure your application:

- ✅ Exposes metrics on `/metrics` endpoint
- ✅ Uses structured logging
- ✅ Includes health check endpoints
- ✅ Has appropriate resource limits
- ✅ Uses consistent labels

## Deployment Manifest

### Basic Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-application
  namespace: default
  labels:
    app: my-application
    team: platform
    environment: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-application
  template:
    metadata:
      labels:
        app: my-application
        version: v1.0.0
      annotations:
        # Prometheus scraping
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: app
        image: my-application:v1.0.0
        ports:
        - name: http
          containerPort: 8080
        - name: metrics
          containerPort: 8080
        env:
        - name: SERVICE_NAME
          value: "my-application"
        - name: ENVIRONMENT
          value: "production"
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: my-application
  labels:
    app: my-application
spec:
  selector:
    app: my-application
  ports:
  - name: http
    port: 80
    targetPort: 8080
  type: ClusterIP
```

## Service Discovery

### Prometheus Annotations

MOP automatically discovers services with these annotations:

```yaml
annotations:
  prometheus.io/scrape: "true"    # Enable scraping
  prometheus.io/port: "8080"      # Metrics port
  prometheus.io/path: "/metrics"  # Metrics endpoint
```

### Labels for Filtering

Use consistent labels:

```yaml
labels:
  app: my-application           # Application name
  team: platform                # Owning team
  environment: production       # Environment
  version: v1.0.0              # Version
  component: api               # Component type
```

## Log Configuration

### Structured Logging

Emit logs to stdout/stderr in JSON format:

```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "info",
  "service": "my-application",
  "environment": "production",
  "message": "Request processed successfully",
  "request_id": "abc123",
  "user_id": "user456",
  "duration_ms": 42,
  "status": 200
}
```

### Log Labels

Loki automatically adds labels from Kubernetes:

- `namespace`: Kubernetes namespace
- `pod`: Pod name
- `container`: Container name
- `app`: From pod labels

## Metrics Endpoint

### Required Metrics

Your application should export:

```
# Request metrics
http_requests_total{method, path, status}
http_request_duration_seconds{method, path}

# System metrics
process_cpu_seconds_total
process_resident_memory_bytes

# Application metrics
database_connections_active
cache_hits_total
cache_misses_total
```

### Example Metrics Output

```
# HELP http_requests_total Total HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="GET",path="/api/users",status="200"} 1523

# HELP http_request_duration_seconds HTTP request duration
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{method="GET",path="/api/users",le="0.1"} 1200
http_request_duration_seconds_bucket{method="GET",path="/api/users",le="0.5"} 1500
http_request_duration_seconds_bucket{method="GET",path="/api/users",le="+Inf"} 1523
http_request_duration_seconds_sum{method="GET",path="/api/users"} 95.3
http_request_duration_seconds_count{method="GET",path="/api/users"} 1523
```

## Deployment Process

### 1. Deploy Application

```bash
kubectl apply -f deployment.yaml
```

### 2. Verify Deployment

```bash
# Check pod status
kubectl get pods -l app=my-application

# Check logs
kubectl logs -l app=my-application --tail=50

# Verify metrics endpoint
kubectl port-forward svc/my-application 8080:80
curl http://localhost:8080/metrics
```

### 3. Verify Monitoring

```bash
# Check Prometheus targets
kubectl port-forward -n mop-system svc/prometheus 9090:9090
# Visit http://localhost:9090/targets

# Query metrics in Prometheus
curl 'http://localhost:9090/api/v1/query?query=up{app="my-application"}'
```

### 4. Create Dashboard

Create a dashboard for your application:

```bash
# Copy dashboard template
cp dashboards/template.json dashboards/my-application.json

# Edit dashboard
# Import to Grafana
```

### 5. Configure Alerts

Create alert rules:

```yaml
groups:
  - name: my-application
    rules:
      - alert: MyApplicationDown
        expr: up{app="my-application"} == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "My Application is down"
```

Apply alerts:

```bash
kubectl apply -f alerts.yaml
```

## ConfigMap for Configuration

### Application Config

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-application-config
data:
  config.yaml: |
    server:
      port: 8080
      metrics_path: /metrics
    logging:
      level: info
      format: json
    monitoring:
      service_name: my-application
      environment: production
```

### Use in Deployment

```yaml
spec:
  containers:
  - name: app
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: my-application-config
```

## Secrets Management

### Database Credentials

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-application-secrets
type: Opaque
stringData:
  database-url: "postgresql://user:pass@db:5432/mydb"
  api-key: "secret-api-key"
```

### Use in Deployment

```yaml
spec:
  containers:
  - name: app
    env:
    - name: DATABASE_URL
      valueFrom:
        secretKeyRef:
          name: my-application-secrets
          key: database-url
```

## Multi-Environment Deployment

### Kustomize Structure

```
k8s/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/
│   │   └── kustomization.yaml
│   ├── staging/
│   │   └── kustomization.yaml
│   └── production/
│       └── kustomization.yaml
```

### Deploy to Environment

```bash
# Deploy to development
kubectl apply -k k8s/overlays/dev

# Deploy to production
kubectl apply -k k8s/overlays/production
```

## Rolling Updates

### Update Strategy

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

### Perform Update

```bash
# Update image
kubectl set image deployment/my-application \
  app=my-application:v1.1.0

# Watch rollout
kubectl rollout status deployment/my-application

# Rollback if needed
kubectl rollout undo deployment/my-application
```

## Troubleshooting

### Pods Not Starting

```bash
# Check pod status
kubectl describe pod <pod-name>

# View logs
kubectl logs <pod-name>

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Metrics Not Appearing

1. Verify annotations are correct
2. Check metrics endpoint is accessible
3. Verify Prometheus targets in Prometheus UI
4. Check pod labels match service discovery config

### High Resource Usage

```bash
# Check resource usage
kubectl top pods -l app=my-application

# Adjust limits if needed
kubectl set resources deployment/my-application \
  --limits=cpu=1000m,memory=1Gi
```

## Best Practices

1. **Use health checks**: Implement `/health` and `/ready` endpoints
2. **Set resource limits**: Prevent resource exhaustion
3. **Use rolling updates**: Zero-downtime deployments
4. **Version your images**: Never use `latest` tag
5. **Monitor from day one**: Deploy monitoring with application
6. **Test in lower environments**: Staging before production
7. **Document your metrics**: Explain what each metric means

## Next Steps

- [Creating Dashboards](./creating-dashboards.md) for your application
- [Configuring Alerts](./configuring-alerts.md) for your service
- [Troubleshooting Guide](./troubleshooting.md) for common issues
