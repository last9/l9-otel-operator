# Last9 OpenTelemetry Operator Setup

Automated setup script for deploying OpenTelemetry Operator, Collector, Kubernetes monitoring, and Events collection to your Kubernetes cluster with Last9 integration.

## Features

- ✅ **One-command installation** - Deploy everything with a single command
- ✅ **Flexible deployment options** - Install only what you need (logs, traces, metrics, events)
- ✅ **Auto-instrumentation** - Automatic instrumentation for Java, Python, Node.js, and more
- ✅ **Kubernetes monitoring** - Full cluster observability with kube-prometheus-stack
- ✅ **Events collection** - Capture and forward Kubernetes events
- ✅ **Cluster identification** - Automatic cluster name detection and attribution
- ✅ **Tolerations support** - Deploy on tainted nodes (control-plane, spot instances, etc.)
- ✅ **Environment customization** - Override deployment environment and cluster name

## Quick Start

### Prerequisites

- `kubectl` configured to access your Kubernetes cluster
- `helm` (v3+) installed

### Option 1: Install Everything (Recommended)

Installs OpenTelemetry Operator, Collector, Kubernetes monitoring stack, and Events agent:

```bash
./last9-otel-setup.sh \
  token="Basic <your-base64-token>" \
  endpoint="<your-otlp-endpoint>" \
  monitoring-endpoint="<your-metrics-endpoint>" \
  username="<your-username>" \
  password="<your-password>"
```

### Quick Install (One-liner)

```bash
curl -fsSL https://raw.githubusercontent.com/last9/l9-otel-operator/main/last9-otel-setup.sh | bash -s -- \
  token="Basic <your-token>" \
  endpoint="<your-otlp-endpoint>" \
  monitoring-endpoint="<your-metrics-endpoint>" \
  username="<user>" \
  password="<pass>"
```

## Installation Options

### Option 2: Traces Only (Operator + Collector)

For applications that need distributed tracing:

```bash
./last9-otel-setup.sh operator-only \
  token="Basic <your-token>" \
  endpoint="<your-otlp-endpoint>"
```

### Option 3: Logs Only (Collector without Operator)

For log collection use cases:

```bash
./last9-otel-setup.sh logs-only \
  token="Basic <your-token>" \
  endpoint="<your-otlp-endpoint>"
```

### Option 4: Metrics Only (Kubernetes Monitoring)

For cluster metrics and monitoring:

```bash
./last9-otel-setup.sh monitoring-only \
  monitoring-endpoint="<your-metrics-endpoint>" \
  username="<your-username>" \
  password="<your-password>"
```

### Option 5: Kubernetes Events Only

For Kubernetes events collection:

```bash
./last9-otel-setup.sh events-only \
  endpoint="<your-otlp-endpoint>" \
  token="Basic <your-base64-token>" \
  monitoring-endpoint="<your-metrics-endpoint>"
```

## Advanced Configuration

### Override Cluster Name

```bash
./last9-otel-setup.sh \
  token="..." \
  endpoint="..." \
  cluster="prod-us-east-1"
```

If not provided, the cluster name is auto-detected from `kubectl config current-context`.

### Set Deployment Environment

```bash
./last9-otel-setup.sh \
  token="..." \
  endpoint="..." \
  env="production"
```

Default: `staging` for collector, `local` for auto-instrumentation.

### Deploy with Tolerations

For deploying on nodes with taints (e.g., control-plane, monitoring nodes):

```bash
./last9-otel-setup.sh \
  token="..." \
  endpoint="..." \
  tolerations-file=/path/to/tolerations.yaml
```

**Example tolerations files** are provided in the `examples/` directory:
- `tolerations-all-nodes.yaml` - Deploy on all nodes including control-plane
- `tolerations-monitoring-nodes.yaml` - Deploy on dedicated monitoring nodes
- `tolerations-spot-instances.yaml` - Deploy on spot/preemptible instances
- `tolerations-multi-taint.yaml` - Handle multiple taints
- `tolerations-nodeSelector-only.yaml` - Use nodeSelector without tolerations

## Configuration Files

| File | Description |
|------|-------------|
| `last9-otel-collector-values.yaml` | OpenTelemetry Collector (DaemonSet) for logs and traces |
| `last9-otel-collector-metrics-deployment-values.yaml` | OpenTelemetry Collector (Deployment) for application metrics scraping |
| `k8s-monitoring-values.yaml` | Kube-prometheus-stack configuration for cluster metrics |
| `last9-kube-events-agent-values.yaml` | Events collection agent configuration |
| `collector-svc.yaml` | Collector service for application instrumentation |
| `instrumentation.yaml` | Auto-instrumentation configuration |
| `deploy.yaml` | Sample application deployment with auto-instrumentation |
| `tolerations.yaml` | Sample tolerations configuration |

### Placeholders

The following placeholders are automatically replaced during installation:

- `{{AUTH_TOKEN}}` - Your Last9 authorization token
- `{{OTEL_ENDPOINT}}` - Your OTEL endpoint URL
- `{{MONITORING_ENDPOINT}}` - Your metrics endpoint URL

## Uninstallation

### Uninstall Everything

```bash
./last9-otel-setup.sh uninstall-all
```

### Uninstall Specific Components

```bash
# Uninstall only monitoring stack
./last9-otel-setup.sh uninstall function="uninstall_last9_monitoring"

# Uninstall only events agent
./last9-otel-setup.sh uninstall function="uninstall_events_agent"

# Uninstall OpenTelemetry components (operator + collector)
./last9-otel-setup.sh uninstall
```

## Verification

After installation, verify the deployment:

```bash
# Check all pods in last9 namespace
kubectl get pods -n last9

# Check collector logs
kubectl logs -n last9 -l app.kubernetes.io/name=opentelemetry-collector

# Check monitoring stack
kubectl get prometheus -n last9

# Check events agent
kubectl get pods -n last9 -l app.kubernetes.io/name=last9-kube-events-agent
```

## Auto-Instrumentation

The script automatically sets up instrumentation for:

- ☕ **Java** - Automatic OTLP export
- 🐍 **Python** - Automatic OTLP export
- 🟢 **Node.js** - Automatic OTLP export
- 🔵 **Go** - Manual instrumentation supported
- 💎 **Ruby** - Coming soon

## Application Metrics Scraping (Optional)

The OpenTelemetry Collector can automatically discover and scrape application metrics using Kubernetes service discovery with Prometheus-compatible scraping.

**Note:** This is an optional feature that uses a **separate Deployment collector** to avoid duplicate metric scraping.

### Architecture: Dual Collector Setup

For optimal metrics collection, we recommend running two separate collectors:

```
┌─────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────┐   ┌─────────────────────────┐  │
│  │   DaemonSet Collector   │   │  Deployment Collector   │  │
│  │   (one per node)        │   │  (single instance)      │  │
│  ├─────────────────────────┤   ├─────────────────────────┤  │
│  │ • Logs (filelog)        │   │ • App metrics scraping  │  │
│  │ • Traces (OTLP)         │   │ • Prometheus SD         │  │
│  │ • Push-based telemetry  │   │ • No duplicate scrapes  │  │
│  └───────────┬─────────────┘   └────────────┬────────────┘  │
│              │                              │               │
└──────────────┼──────────────────────────────┼───────────────┘
               │                              │
               ▼                              ▼
          Last9 OTLP                 Last9 Prometheus
          Endpoint                   Remote Write
```

| Collector | Mode | Purpose | Config File |
|-----------|------|---------|-------------|
| Logs & Traces | DaemonSet | Node-local log/trace collection | `last9-otel-collector-values.yaml` |
| Metrics | Deployment | Centralized metrics scraping | `last9-otel-collector-metrics-deployment-values.yaml` |

**Why separate collectors?**
- **DaemonSet** is required for logs (reads `/var/log/pods` on each node) and traces (low-latency local endpoint)
- **Deployment** is better for metrics scraping — a single instance avoids duplicate scrapes that would occur with DaemonSet

### Enable Metrics Scraping

**Step 1: Configure Last9 Metrics Endpoint**

Before deploying, update these placeholders in `last9-otel-collector-metrics-deployment-values.yaml`:
- `{{LAST9_METRICS_ENDPOINT}}` - Your Last9 Prometheus remote write URL
- `{{LAST9_METRICS_USERNAME}}` - Your Last9 metrics username
- `{{LAST9_METRICS_PASSWORD}}` - Your Last9 metrics password

**Step 2: Deploy the Metrics Collector**

```bash
# Deploy dedicated metrics collector (Deployment mode)
helm upgrade --install last9-otel-collector-metrics open-telemetry/opentelemetry-collector \
  --namespace last9 \
  --values last9-otel-collector-metrics-deployment-values.yaml
```

This runs alongside your existing DaemonSet collector without conflict.

### Quick Start

Add these annotations to your pod template or service to enable automatic metrics scraping:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    metadata:
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"  # Optional, defaults to /metrics
```

That's it! Your application metrics will be automatically:
- **Discovered** - No manual configuration needed
- **Scraped** - Every 30 seconds by default
- **Enriched** - With pod, namespace, node labels
- **Exported** - To Last9 via Prometheus remote write

### How It Works

1. **Automatic Discovery** - OTel Collector watches Kubernetes API for all pods/services
2. **Annotation-Based Filtering** - Only scrapes resources with `prometheus.io/scrape: "true"`
3. **Metadata Enrichment** - Adds Kubernetes labels automatically (pod, namespace, node, app)
4. **Direct Export** - Sends metrics to Last9 Prometheus endpoint

### Supported Annotations

| Annotation | Required | Default | Description |
|------------|----------|---------|-------------|
| `prometheus.io/scrape` | Yes | - | Set to "true" to enable scraping |
| `prometheus.io/port` | Yes | - | Port number exposing /metrics |
| `prometheus.io/path` | No | /metrics | HTTP path for metrics endpoint |

### Scaling

This setup scales automatically:
- **1 service** → Automatically scraped
- **1000 services** → Automatically scraped
- **No configuration changes needed** when adding new services

### Metrics Configuration

**File:** `last9-otel-collector-metrics-deployment-values.yaml`

This runs as a separate Deployment alongside the DaemonSet collector:
- Single instance avoids duplicate metric scrapes
- Clear separation: DaemonSet for logs/traces, Deployment for metrics
- Independent scaling and resource allocation

### Verification

Check if metrics are being scraped:

```bash
# Check metrics collector deployment is running
kubectl get deployment -n last9 last9-otel-collector-metrics

# Check metrics collector logs for scraping activity
kubectl logs -n last9 -l app.kubernetes.io/name=last9-otel-collector-metrics | grep kubernetes-pods

# Port-forward to metrics collector
kubectl port-forward -n last9 deployment/last9-otel-collector-metrics 8888:8888

# Check scrape status
curl http://localhost:8888/metrics | grep scrape_samples_scraped
```

### Uninstall Metrics Collector

```bash
helm uninstall last9-otel-collector-metrics --namespace last9
```
