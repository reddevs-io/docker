# Builder stage
FROM golang:1-alpine AS builder

LABEL version="1.0.1"

# Install necessary tools for building
RUN apk add --no-cache curl bash openssl

# Install kube-linter
RUN GO111MODULE=on go install golang.stackrox.io/kube-linter/cmd/kube-linter@latest && \
  mv /go/bin/kube-linter /usr/local/bin/kube-linter

# Install kubectl
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
  chmod +x kubectl && \
  mv kubectl /usr/local/bin/

# Install helm
RUN curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 && \
  chmod 700 get_helm.sh && \
  VERIFY_CHECKSUM=false ./get_helm.sh && \
  rm get_helm.sh

# Final stage
FROM alpine:3

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
