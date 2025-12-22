# Node.js Auto-Instrumentation Images for Legacy Versions

This directory contains Docker images for Node.js auto-instrumentation supporting **legacy Node.js versions** (10 and 12) that are no longer supported by current OpenTelemetry releases.

## 🎯 Purpose

Enable Last9 customers running legacy Node.js applications to use auto-instrumentation via the OpenTelemetry operator by providing images with compatible OpenTelemetry versions.

## 📊 Compatibility Matrix

| Node Version | EOL Date | OpenTelemetry Version | Image Tag | Status |
|--------------|----------|----------------------|-----------|--------|
| **10.x** | April 2021 | 0.25.0 | `node10` | ⚠️ EOL - Security Risk |
| **12.x** | April 2022 | 0.27.0 | `node12` | ⚠️ EOL - Security Risk |
| **14.x+** | Active/Maintenance | Latest | `latest` | ✅ Recommended |

## 🏗️ Image Structure

Each image follows the OpenTelemetry operator's autoinstrumentation pattern:

```
/autoinstrumentation/
├── node_modules/           # OpenTelemetry packages
│   ├── @opentelemetry/
│   └── ...
└── package.json           # Version reference
```

The operator copies these files into the target container and injects them via `NODE_OPTIONS` or `--require`.

## 🔨 Building Images

### Build Node 10 Image

```bash
cd node10
docker build -t ghcr.io/last9/autoinstrumentation-nodejs:node10 .
```

### Build Node 12 Image

```bash
cd node12
docker build -t ghcr.io/last9/autoinstrumentation-nodejs:node12 .
```

### Push to Registry

```bash
# Node 10
docker push ghcr.io/last9/autoinstrumentation-nodejs:node10

# Node 12
docker push ghcr.io/last9/autoinstrumentation-nodejs:node12
```

## 📦 Package Versions

### Node 10 (OpenTelemetry 0.25.0)

```json
{
  "@opentelemetry/api": "1.0.4",
  "@opentelemetry/sdk-node": "0.25.0",
  "@opentelemetry/auto-instrumentations-node": "0.25.0",
  "@opentelemetry/exporter-trace-otlp-http": "0.25.0"
}
```

### Node 12 (OpenTelemetry 0.27.0)

```json
{
  "@opentelemetry/api": "1.0.4",
  "@opentelemetry/sdk-node": "0.27.0",
  "@opentelemetry/auto-instrumentations-node": "0.27.3",
  "@opentelemetry/exporter-trace-otlp-http": "0.27.0"
}
```

## 🚀 Usage in Kubernetes

### Step 1: Update Instrumentation Resource

Specify the legacy Node image in your `Instrumentation` resource:

```yaml
apiVersion: opentelemetry.io/v1alpha1
kind: Instrumentation
metadata:
  name: l9-instrumentation
  namespace: last9
spec:
  nodejs:
    # Use legacy Node 12 image
    image: ghcr.io/last9/autoinstrumentation-nodejs:node12

    env:
    - name: OTEL_EXPORTER_OTLP_ENDPOINT
      value: http://otel-collector-service.last9.svc.cluster.local:4318
    - name: OTEL_EXPORTER_OTLP_PROTOCOL
      value: http/protobuf
    - name: OTEL_SERVICE_NAME
      value: my-node12-app
```

### Step 2: Annotate Your Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-legacy-node-app
spec:
  template:
    metadata:
      annotations:
        instrumentation.opentelemetry.io/inject-nodejs: "true"
    spec:
      containers:
      - name: app
        image: my-node12-app:latest
        # Your Node 12 application
```

### Step 3: Deploy and Verify

```bash
kubectl apply -f instrumentation.yaml
kubectl apply -f deployment.yaml

# Verify instrumentation was injected
kubectl get pods
# Should show 2/2 containers (app + init container)

# Check logs
kubectl logs <pod-name> -c app
```

## ⚙️ Configuration Options

All standard OpenTelemetry environment variables are supported:

| Variable | Purpose | Example |
|----------|---------|---------|
| `OTEL_SERVICE_NAME` | Service identifier | `legacy-api` |
| `OTEL_RESOURCE_ATTRIBUTES` | Custom attributes | `env=prod,team=backend` |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | Last9 endpoint | `https://otlp.last9.io` |
| `OTEL_EXPORTER_OTLP_HEADERS` | Auth headers | `Authorization=Basic TOKEN` |
| `OTEL_TRACES_SAMPLER` | Sampling strategy | `always_on` or `traceidratio` |

## 🆚 Comparison: Operator vs Manual Instrumentation

| Aspect | Operator (This Approach) | Manual Instrumentation |
|--------|-------------------------|------------------------|
| **Code Changes** | Zero | Install packages + init code |
| **Image Selection** | Specify in `Instrumentation` CRD | Install in package.json |
| **Updates** | Change image tag | Update package versions |
| **Environment** | Kubernetes only | Anywhere (VMs, Lambda, K8s) |
| **Flexibility** | Limited to auto-instrumentations | Full control |

## 📚 Manual Instrumentation (Alternative)

If you're not using Kubernetes or prefer manual control, install the compatible OpenTelemetry version directly:

### For Node 10

```bash
npm install \
  @opentelemetry/sdk-node@0.25.0 \
  @opentelemetry/auto-instrumentations-node@0.25.0 \
  @opentelemetry/exporter-trace-otlp-http@0.25.0
```

### For Node 12

```bash
npm install \
  @opentelemetry/sdk-node@0.27.0 \
  @opentelemetry/auto-instrumentations-node@0.27.3 \
  @opentelemetry/exporter-trace-otlp-http@0.27.0
```

Then initialize in your application:

```javascript
// app.js or index.js (must be FIRST import)
const { NodeSDK } = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-http');

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter({
    url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT + '/v1/traces',
    headers: {
      Authorization: process.env.OTEL_EXPORTER_OTLP_HEADERS,
    },
  }),
  instrumentations: [getNodeAutoInstrumentations()],
});

sdk.start();

// Your application code...
const express = require('express');
const app = express();
// ...
```

## ⚠️ Security Warnings

### Node 10 (EOL: April 2021)
- **Critical security vulnerabilities** - No longer receiving updates
- **Not recommended for production** - Use only for migration purposes
- **Upgrade path required** - Plan migration to Node 18+ ASAP

### Node 12 (EOL: April 2022)
- **Security vulnerabilities exist** - No longer receiving updates
- **Limited support** - Use only during migration window
- **Upgrade recommended** - Move to Node 18+ or 20+

## 🔄 Migration Strategy

### Phase 1: Enable Instrumentation (Now)
- Use legacy images to get observability immediately
- Identify performance bottlenecks and errors

### Phase 2: Plan Upgrade (1-3 months)
- Review application dependencies for Node 18/20 compatibility
- Test application with newer Node versions
- Update CI/CD pipelines

### Phase 3: Upgrade Node (3-6 months)
- Deploy to Node 18+ or 20+
- Switch to latest OpenTelemetry images
- Monitor for issues

## 🐛 Troubleshooting

### Instrumentation Not Working

**Check operator logs:**
```bash
kubectl logs -n last9 deployment/opentelemetry-operator-controller-manager
```

**Verify image was pulled:**
```bash
kubectl describe pod <pod-name> | grep Image:
# Should show ghcr.io/last9/autoinstrumentation-nodejs:node12
```

**Check init container logs:**
```bash
kubectl logs <pod-name> -c opentelemetry-auto-instrumentation
```

### No Traces Appearing

**Verify environment variables:**
```bash
kubectl exec <pod-name> -- env | grep OTEL
```

**Check application logs:**
```bash
kubectl logs <pod-name> -c app
# Look for OpenTelemetry initialization messages
```

## 📖 Additional Resources

- [OpenTelemetry JS v0.25.0 Docs](https://github.com/open-telemetry/opentelemetry-js/tree/v0.25.0)
- [OpenTelemetry JS v0.27.0 Docs](https://github.com/open-telemetry/opentelemetry-js/tree/v0.27.0)
- [Node.js Release Schedule](https://nodejs.org/en/about/releases/)
- [Last9 Documentation](https://docs.last9.io)

## 🤝 Contributing

Found an issue or want to add support for another Node version? Open an issue or PR!

## 📄 License

MIT License
