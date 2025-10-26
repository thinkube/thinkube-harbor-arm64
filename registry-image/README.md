# Harbor Registry ARM64

Fix for `goharbor/harbor-registry-photon` to run on ARM64.

## Problem

The ranichowdary/registry-harbor image contains an x86_64 binary at `/usr/bin/registry_DO_NOT_USE_GC`.

## Solution

Replace with ARM64 binary from official `docker.io/registry:2.8.3`.

## Building

```bash
podman build -t harbor-registry-arm64:v2.14.0 .
```

## Binary Source

The `registry` binary is extracted from:
```bash
podman pull --platform=linux/arm64 docker.io/registry:2.8.3
podman create --name temp registry:2.8.3
podman cp temp:/bin/registry ./registry
podman rm temp
```

Version: registry 2.8.3 (ARM64)
