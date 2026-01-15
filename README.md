# Helm Charts

Collection of Helm charts for deploying applications.

## Available Charts

- **sample-java-api** - Java API application

## Install from OCI Registry

```bash
helm install my-app oci://ghcr.io/pauloo27/helm-charts/sample-java-api --version v0.1.0
```

Or install the latest version:

```bash
helm install my-app oci://ghcr.io/pauloo27/helm-charts/sample-java-api
```

## Publishing

Charts are automatically published to GitHub Container Registry when tags are pushed:

```bash
git tag v0.1.0
git push --tags
```
