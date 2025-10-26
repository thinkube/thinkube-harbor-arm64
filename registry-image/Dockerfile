# Dockerfile to fix ranichowdary/registry-harbor for ARM64
# Problem: The ranichowdary image has x86_64 registry binary that won't run on ARM64
# Solution: Replace it with ARM64 binary from official registry:2 image

FROM docker.io/registry:2.8.3 AS registry-source

FROM ranichowdary/registry-harbor:latest

# Copy ARM64 registry binary from official registry:2 image
# This replaces the x86_64 binary at /usr/bin/registry_DO_NOT_USE_GC
COPY --from=registry-source /bin/registry /usr/bin/registry_DO_NOT_USE_GC

# Keep the original entrypoint from ranichowdary image
# The entrypoint.sh script will run: /usr/bin/registry_DO_NOT_USE_GC serve /etc/registry/config.yml
