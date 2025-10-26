# Dockerfile to fix goharbor/trivy-adapter-photon for ARM64
# Problem: The official image has x86_64 binaries that won't run on ARM64
# Solution: Replace both Trivy scanner and scanner-trivy adapter with ARM64 versions

# Stage 1: Get ARM64 Trivy scanner from official image
FROM docker.io/aquasec/trivy:latest AS trivy-source

# Stage 2: Build ARM64 scanner-trivy from source
FROM golang:1.23-alpine AS scanner-builder

WORKDIR /build

# Clone and build harbor-scanner-trivy
RUN apk add --no-cache git && \
    git clone --depth 1 --branch v0.33.1 https://github.com/goharbor/harbor-scanner-trivy.git . && \
    CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -o scanner-trivy ./cmd/scanner-trivy/

# Stage 3: Final image with ARM64 binaries
FROM goharbor/trivy-adapter-photon:v2.14.0

# Copy ARM64 Trivy scanner binary from official Trivy image
COPY --from=trivy-source /usr/local/bin/trivy /usr/local/bin/trivy

# Copy ARM64 scanner-trivy adapter binary from builder
COPY --from=scanner-builder /build/scanner-trivy /home/scanner/bin/scanner-trivy

# Keep the original entrypoint from goharbor image
# Note: Kubernetes deployment must override command to: ["/home/scanner/bin/scanner-trivy"]
# This bypasses the x86 shell in the base image entrypoint script
