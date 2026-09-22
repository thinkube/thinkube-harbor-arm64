# thinkube-harbor-arm64

Two patched Harbor v2.14.0 container images that let Thinkube run Harbor on an arm64 control plane.

## What it does

Harbor v2.14.0 does not provide official ARM64 container images. Community multi-arch images exist (ranichowdary/harbor-*), but some components contain x86_64 binaries that fail on ARM64 systems.

This repository builds **2 patched container images**:

1. **harbor-registry-arm64** - Registry component with an ARM64 binary
2. **harbor-trivy-adapter-arm64** - Trivy security scanner and its adapter with ARM64 binaries

A GitHub Actions workflow builds both images for `linux/arm64` and pushes them to the GitHub Container Registry.

## How it reaches a user

The images are release files used by the core Harbor deployment in [thinkube](https://github.com/thinkube/thinkube). When the Thinkube installer deploys Harbor (`ansible/40_thinkube/core/harbor/10_deploy.yaml`) and the control plane is arm64, the playbook sets these two images in the Harbor Helm values and then applies the two command patches described below. On amd64 the playbook uses the official Harbor images. The images are not installed on their own.

## What's Fixed

### 1. Registry (`harbor-registry-photon`)
- **Problem**: The community image `ranichowdary/registry-harbor` contains an x86_64 `/usr/bin/registry_DO_NOT_USE_GC` binary
- **Solution**: Replaced with the ARM64 binary from official `docker.io/registry:2.8.3`

### 2. Trivy Adapter (`harbor-trivy-adapter-photon`)
- **Problem**: The image `goharbor/trivy-adapter-photon:v2.14.0` contains x86_64 binaries:
  - `/usr/local/bin/trivy` (scanner)
  - `/home/scanner/bin/scanner-trivy` (adapter)
- **Solution**:
  - Replaced Trivy with the ARM64 binary from `docker.io/aquasec/trivy:latest`
  - Built `scanner-trivy` from source for ARM64 (goharbor/harbor-scanner-trivy `v0.33.1`, Go 1.23)

## Container Images

**Registry (ARM64)**:
```
ghcr.io/thinkube/thinkube-harbor-arm64/harbor-registry-arm64:v2.14.0
```

**Trivy Adapter (ARM64)**:
```
ghcr.io/thinkube/thinkube-harbor-arm64/harbor-trivy-adapter-arm64:v2.14.0
```

Each image is also tagged `latest`. The Thinkube playbook uses the `v2.14.0` tag.

These 2 images alone are not enough for a Harbor deployment. On arm64 the Thinkube playbook uses 8 images from three sources:

- **These patches** (2): registry, trivy-adapter
- **Community images** (5): core, portal, jobservice, registryctl, database (from ranichowdary/harbor-*)
- **Redis replacement** (1): valkey/valkey (BSD-licensed Redis alternative)

## Tested Configuration

- **Platform**: NVIDIA DGX Spark (ARM64/aarch64)
- **OS**: Ubuntu 24.04 LTS
- **Kubernetes**: kubeadm (managed by Thinkube)
- **Harbor**: v2.14.0
- **Storage**: Thinkube-managed storage classes

## Technical Details

### Why Command Patches Are Required

Two Harbor components require container command overrides on ARM64. After the Helm install, on arm64 only, the Thinkube playbook applies them with `kubectl patch`:

1. **harbor-portal** (deployment): the command is set to `["nginx", "-g", "daemon off;"]` to fix a duplicated command.
2. **harbor-trivy** (statefulset): the command is set to `["/home/scanner/bin/scanner-trivy"]`. This bypasses the entrypoint script of the base image, which runs an x86 shell. The trivy-adapter image keeps that entrypoint, so the override is needed.

### Image Build Process

All images are built via GitHub Actions with multi-stage Docker builds:
- Binaries are extracted from official upstream images, or built from source, at build time
- No binaries are committed to this git repository
- Built images are published to GitHub Container Registry (ghcr.io)
- The workflow runs on a push to `main`, on a `v*` tag, and by hand
- See `.github/workflows/build-and-push.yml` for build automation

## Version Compatibility

- **Harbor**: v2.14.0 only
- **Architecture**: linux/arm64 (aarch64)
- **Platform**: Thinkube on DGX Spark

The images are built on Harbor v2.14.0 images and are not compatible with other Harbor versions.

## Working on it

### Building Images Yourself

- [registry-image/README.md](registry-image/README.md) - Build registry fix
- [trivy-adapter-image/README.md](trivy-adapter-image/README.md) - Build trivy-adapter fix

### Issues

- **Thinkube deployment problems**: https://github.com/thinkube/thinkube/issues
- **Image build problems**: Open an issue in this repository
- **Harbor functionality**: https://github.com/goharbor/harbor/issues

## License

Apache License 2.0. See [LICENSE](LICENSE).

## Acknowledgments

- **Harbor Project** (goharbor/harbor) - Original software
- **ranichowdary** - Community multi-arch platform support effort
- **Docker Distribution** - Official registry:2 image
- **Aqua Security** - Official Trivy scanner
- **Valkey** - BSD-licensed Redis implementation
