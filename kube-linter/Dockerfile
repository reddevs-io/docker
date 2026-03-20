# Builder stage
FROM golang:1-alpine AS builder

LABEL version="1.0.3"

# Install necessary tools for building
RUN apk upgrade && \
  apk add --no-cache curl bash openssl git

# Install kube-linter (clone source and force grpc >= v1.79.3 to remediate CVE-2026-33186)
RUN git clone https://github.com/stackrox/kube-linter.git /tmp/kube-linter && \
  cd /tmp/kube-linter && \
  go get google.golang.org/grpc@v1.79.3 && \
  go mod tidy && \
  go build -o /usr/local/bin/kube-linter ./cmd/kube-linter/

# Install kubectl (built from source to ensure Go stdlib CVEs are patched)
ENV KUBECTL_VERSION=v1.35.0
RUN git clone --depth 1 --branch ${KUBECTL_VERSION} https://github.com/kubernetes/kubernetes.git /tmp/kubernetes && \
  cd /tmp/kubernetes && \
  go build -o /usr/local/bin/kubectl ./cmd/kubectl/

# Install helm
RUN curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4 && \
    chmod 700 get_helm.sh && \
    ./get_helm.sh

# Final stage
FROM alpine:3

# Upgrade OS packages to patch known CVEs (e.g. zlib CVE-2026-22184, CVE-2026-27171)
RUN apk upgrade --no-cache

# Copy the installed binaries from the builder stage
COPY --from=builder /usr/local/bin/kube-linter /usr/local/bin/kube-linter
COPY --from=builder /usr/local/bin/kubectl /usr/local/bin/kubectl
COPY --from=builder /usr/local/bin/helm /usr/local/bin/helm

# Install runtime dependencies
RUN apk add --no-cache ca-certificates

# Set PATH to include the installed tools
ENV PATH="/usr/local/bin:${PATH}"

# Verify installations
RUN kube-linter version && kubectl version --client && helm version

# Set the entrypoint to shell
ENTRYPOINT ["/bin/sh"]
