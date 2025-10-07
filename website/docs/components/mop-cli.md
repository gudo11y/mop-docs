---
sidebar_position: 2
---

# MOP CLI

**mop-cli** provides command-line tools for managing and interacting with the MOP Platform.

## Overview

The MOP CLI simplifies common operations like deploying components, managing configurations, and troubleshooting issues.

## Installation

```bash
# Install via package manager (example)
npm install -g @mop-platform/cli

# Or download binary
curl -L https://github.com/mop-platform/mop-cli/releases/latest/download/mop-cli-linux-amd64 -o mop
chmod +x mop
sudo mv mop /usr/local/bin/
```

## Common Commands

### Platform Management

```bash
# Check platform status
mop status

# Deploy or update components
mop deploy --component prometheus
mop deploy --all

# Restart services
mop restart grafana
```

### Configuration

```bash
# View current configuration
mop config view

# Update configuration
mop config set prometheus.retention 30d

# Validate configuration
mop config validate
```

### Troubleshooting

```bash
# View logs
mop logs prometheus
mop logs --follow grafana

# Run health checks
mop health-check

# Debug mode
mop debug --component loki
```

## Features

### GitOps Integration

- Sync configurations from Git repositories
- Automatic drift detection
- Rollback capabilities

### Multi-Cluster Support

Manage multiple clusters from a single CLI:

```bash
mop context list
mop context switch production
mop context use staging
```

### Plugin System

Extend functionality with plugins:

```bash
mop plugin install alerting-extensions
mop plugin list
```

## Configuration File

The CLI uses a configuration file at `~/.mop/config.yaml`:

```yaml
default_context: production
contexts:
  production:
    api_endpoint: https://mop-api.prod.example.com
    namespace: mop-system
  staging:
    api_endpoint: https://mop-api.staging.example.com
    namespace: mop-staging
```

## Related Components

- [mop-core](./mop-core.md): The components managed by mop-cli
- [mop-infra](./mop-infra.md): Infrastructure provisioning
