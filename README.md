# Helm Charts

Helm charts for deploying applications to Kubernetes. Charts are packaged and 
published to GitHub Container Registry as OCI artifacts, making them easy to 
version and consume in deployment pipelines.

## How It Works

This repository contains Helm charts stored in the `charts/` directory.
Each chart follows standard Helm structure with templates, values, and metadata.

### sample-java-api Chart

The `sample-java-api` chart deploys a simple Java API application with:

- **Deployment** - Single replica by default, configurable via `replicaCount`
- **Service** - ClusterIP service exposing the application
- **Health Probes** - Liveness and readiness checks against `/health` endpoint
- **Port Configuration** - Single `http_port` parameter controls:
  - Container port
  - Service port
  - Health probe port
  - `SERVER_PORT` environment variable injected into the pod

Templates are kept minimal - no Ingress, HPA, or ServiceAccount. The chart 
focuses on core deployment needs.

## Configuration

### Key Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `http_port` | `8080` | HTTP port for the application (used across container, service, and probes) |
| `image.repository` | `ghcr.io/pauloo27/sample-java-api` | Docker image repository |
| `image.tag` | `latest` | Image tag to deploy |
| `image.pullPolicy` | `IfNotPresent` | Image pull policy |
| `replicaCount` | `1` | Number of pod replicas |
| `resources` | `{}` | Pod resource requests and limits |

## How to Use

```bash
# Install or upgrade from OCI registry
helm upgrade --install my-app oci://ghcr.io/pauloo27/helm-charts/sample-java-api \
  --version v0.1.1 \
  --namespace production \
  --create-namespace \
  --set image.tag=abc1234 \
  --set http_port=9000

# Or use a custom values file
helm upgrade --install my-app oci://ghcr.io/pauloo27/helm-charts/sample-java-api \
  --values custom-values.yaml
```

Example values file:
```yaml
http_port: 9000
image:
  tag: "abc1234"
resources:
  limits:
    memory: "1Gi"
```

## Integration with Other Repos

### Consumes from

**deployment-workflows**
- Calls `helm-push.yml` reusable workflow to package and push charts to GHCR
- Triggered automatically when version tags (e.g., `v0.1.1`) are pushed to this repo

### Provides to

**sample-java-api**
- The application repo pulls the `sample-java-api` chart from GHCR during deployments
- Uses environment-specific values files (`helm/values.dev.yml`, 
  `helm/values.prod.yml`) to override chart defaults
- Chart reference: `oci://ghcr.io/pauloo27/helm-charts/sample-java-api:v0.1.1`

## Publishing

Charts are automatically packaged and pushed to GHCR when version tags are created:

```bash
git tag v0.1.1
git push --tags
```

## Assumptions and Shortcuts

- **OCI Registry**: Uses GitHub Container Registry (GHCR) for storing Helm 
  charts as OCI artifacts.
- **Simplified Chart**: Removed optional components (Ingress, HPA, 
  ServiceAccount) to keep the chart focused on core deployment needs
- **Single Port**: Application only exposes one HTTP port, configured via 
  `http_port` parameter that's reused across all relevant resources
- **Manual Versioning**: Chart version in `Chart.yaml` must be manually updated 
  before tagging - no automatic version bumping
- **No Dependencies**: Chart has no external dependencies, making it 
  self-contained and simple to deploy
