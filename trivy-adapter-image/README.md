# Harbor Trivy Adapter ARM64

Fix for `goharbor/trivy-adapter-photon` to run on ARM64.

## Problem

The official image contains x86_64 binaries:
- `/usr/local/bin/trivy` - Trivy scanner
- `/home/scanner/bin/scanner-trivy` - Harbor adapter

## Solution

Replace both binaries with ARM64 versions:

1. **Trivy scanner**: Extract from `docker.io/aquasec/trivy:latest`
2. **scanner-trivy**: Build from source (harbor-scanner-trivy v0.33.1)

## Building

```bash
podman build -t harbor-trivy-adapter-arm64:v2.14.0 .
```

## Binary Sources

### trivy-arm64
```bash
podman pull --platform=linux/arm64 docker.io/aquasec/trivy:latest
podman create --name temp aquasec/trivy:latest
podman cp temp:/usr/local/bin/trivy ./trivy-arm64
podman rm temp
```

### scanner-trivy
```bash
git clone --depth 1 --branch v0.33.1 https://github.com/goharbor/harbor-scanner-trivy.git
cd harbor-scanner-trivy
CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -o scanner-trivy ./cmd/scanner-trivy/
```

## Deployment Note

The Trivy adapter requires a command override in Kubernetes:

```yaml
trivy:
  command: ["/home/scanner/bin/scanner-trivy"]
```

This bypasses the x86 shell in the base image.
