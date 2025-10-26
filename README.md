# Harbor ARM64 Patches

Unofficial patches to enable Harbor v2.14.0 on ARM64 architecture.

> **Primary Support**: NVIDIA DGX Spark running Ubuntu 24.04 with Canonical k8s-snap
>
> **Status**: These patches will be deprecated once Harbor Project releases official ARM64 support.
>
> **Compatibility**: Tested on DGX Spark. Other ARM64 platforms may work but are not the primary focus for support.

## Background

Harbor v2.14.0 does not provide official ARM64 container images. The [ranichowdary/harbor-*](https://hub.docker.com/u/ranichowdary) multi-arch images (from Harbor's multiarch-platform-support branch) contain x86_64 binaries that fail on ARM64 systems.

These patches replace x86_64 binaries with ARM64 versions to enable Harbor deployment on ARM64 Kubernetes clusters.

## What's Fixed

### 1. Registry (`harbor-registry-photon`)
- **Problem**: ranichowdary image contains x86_64 `/usr/bin/registry_DO_NOT_USE_GC` binary
- **Solution**: Replace with ARM64 binary from official `docker.io/registry:2.8.3`

### 2. Trivy Adapter (`harbor-trivy-adapter-photon`)
- **Problem**: goharbor image contains x86_64 binaries:
  - `/usr/local/bin/trivy` (scanner)
  - `/home/scanner/bin/scanner-trivy` (adapter)
- **Solution**:
  - Replace Trivy with ARM64 binary from `docker.io/aquasec/trivy:latest`
  - Build `scanner-trivy` from source for ARM64

## Pre-built Images

**Registry (ARM64)**:
```
ghcr.io/thinkube/thinkube-harbor-arm64/harbor-registry-arm64:v2.14.0
```

**Trivy Adapter (ARM64)**:
```
ghcr.io/thinkube/thinkube-harbor-arm64/harbor-trivy-adapter-arm64:v2.14.0
```

**Note**: These 2 images are NOT sufficient for a complete Harbor deployment. You also need the other components (core, portal, jobservice, registryctl, database) from ranichowdary's images.

## Usage

### Harbor Helm Chart

```yaml
# Complete ARM64 configuration for Harbor v2.14.0
core:
  image:
    repository: ranichowdary/harbor-core
    tag: latest

portal:
  image:
    repository: ranichowdary/harbor-portal
    tag: latest

jobservice:
  image:
    repository: ranichowdary/jobservice-harbor
    tag: latest

registry:
  registry:
    image:
      repository: ghcr.io/thinkube/thinkube-harbor-arm64/harbor-registry-arm64
      tag: v2.14.0
  controller:
    image:
      repository: ranichowdary/harbor-registryctl
      tag: latest

database:
  internal:
    image:
      repository: ranichowdary/harbor-db
      tag: latest

redis:
  internal:
    image:
      repository: valkey/valkey
      tag: latest

trivy:
  enabled: true
  image:
    repository: ghcr.io/thinkube/thinkube-harbor-arm64/harbor-trivy-adapter-arm64
    tag: v2.14.0
```

### Ansible (Thinkube-style)

Update playbook to use patched images:
```yaml
- name: Pull patched Harbor images
  ansible.builtin.command:
    cmd: "podman pull ghcr.io/your-username/{{ item }}"
  loop:
    - harbor-registry-arm64:v2.14.0
    - harbor-trivy-adapter-arm64:v2.14.0
```

## Tested Configuration

- **Platform**: NVIDIA DGX Spark (ARM64)
- **OS**: Ubuntu 24.04 LTS
- **Kubernetes**: Canonical k8s-snap 1.34
- **Harbor**: v2.14.0
- **Architecture**: linux/arm64 (aarch64)

## Known Limitations

1. **Support focus**: Primary testing and support is for DGX Spark. Other ARM64 platforms (Raspberry Pi, AWS Graviton, Apple Silicon) may work but are not actively tested.

2. **Command override required**: Trivy adapter needs command override due to x86 shell in base image.

3. **Version specific**: Built for Harbor v2.14.0. Not compatible with other versions.

## Building Images Yourself

See individual component READMEs:
- [registry/README.md](registry/README.md) - Build registry fix
- [trivy-adapter/README.md](trivy-adapter/README.md) - Build trivy-adapter fix

## Migration Path

**When Harbor releases official ARM64 support:**

1. Stop using these patches immediately
2. Update to official Harbor images:
   ```yaml
   registry:
     registry:
       image:
         repository: goharbor/harbor-registry-photon
         tag: v2.15.0  # or whatever version has ARM64

   trivy:
     image:
       repository: goharbor/trivy-adapter-photon
       tag: v2.15.0
     command: []  # Remove override
   ```
3. Remove patched images from your registry

## Contributing

Track official ARM64 support progress:
- Harbor Issue: https://github.com/goharbor/harbor/issues/XXXXX (if exists)
- Harbor Discussions: https://github.com/goharbor/harbor/discussions

## License

Apache License 2.0 (same as Harbor Project)

## Acknowledgments

- Harbor Project (goharbor/harbor) - Original software
- ranichowdary - Multi-arch platform support effort
- Docker Distribution - Official registry:2 image
- Aqua Security - Official Trivy scanner

## Support

Support is focused on DGX Spark deployments. For issues:
- **Harbor functionality**: https://github.com/goharbor/harbor/issues
- **DGX Spark specific issues**: Open an issue in this repository
- **Other platforms**: Community best-effort only

---

**Note**: These patches are temporary until Harbor Project provides official ARM64 images. Prefer official releases when available.
