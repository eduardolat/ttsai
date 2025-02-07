FROM denoland/deno:debian-2.1.9 AS deno
FROM golang:1.23.5-bookworm AS golang

# Set the base image
FROM debian:12.7

# Declare ARG to make it available in the RUN commands
ARG TARGETPLATFORM
RUN echo "Building for ${TARGETPLATFORM}"
RUN if [ "${TARGETPLATFORM}" != "linux/amd64" ] && [ "${TARGETPLATFORM}" != "linux/arm64" ]; then \
        echo "Unsupported architecture: ${TARGETPLATFORM}" && \
        exit 1; \
    fi

# Set the general environment variables, and move to temp dir
ENV DEBIAN_FRONTEND="noninteractive"
ENV PATH="$PATH:/usr/local/go/bin"
ENV PATH="$PATH:/usr/local/dl/bin"
ENV GOBIN="/usr/local/go/bin"
RUN mkdir -p /app/temp /usr/local/dl/bin
WORKDIR /app/temp

# Install deno from docker image
COPY --from=deno /usr/bin/deno /usr/local/bin/deno

# Install golang from docker image
COPY --from=golang /usr/local/go /usr/local/go

# Add PostgreSQL repository and install system dependencies
# https://www.postgresql.org/download/linux/debian/
RUN apt update && apt install -y \
        wget zip unzip p7zip-full tzdata git python3 python3-pip && \
    rm -rf /var/lib/apt/lists/*

# Install downloadable binaries
RUN set -e && \
    if [ "${TARGETPLATFORM}" = "linux/arm64" ]; then \
        echo "Downloading arm64 binaries" && \
        # Install task
        wget --no-verbose https://github.com/go-task/task/releases/download/v3.38.0/task_linux_arm64.tar.gz && \
        tar -xzf task_linux_arm64.tar.gz && \
        mv ./task /usr/local/dl/bin/task && \
        # Install golangci-lint
        wget --no-verbose https://github.com/golangci/golangci-lint/releases/download/v1.60.3/golangci-lint-1.60.3-linux-arm64.tar.gz && \
        tar -xzf golangci-lint-1.60.3-linux-arm64.tar.gz && \
        mv ./golangci-lint-1.60.3-linux-arm64/golangci-lint /usr/local/dl/bin/golangci-lint; \
    else \
        echo "Downloading amd64 binaries" && \
        # Install task
        wget --no-verbose https://github.com/go-task/task/releases/download/v3.38.0/task_linux_amd64.tar.gz && \
        tar -xzf task_linux_amd64.tar.gz && \
        mv ./task /usr/local/dl/bin/task && \
        # Install golangci-lint
        wget --no-verbose https://github.com/golangci/golangci-lint/releases/download/v1.60.3/golangci-lint-1.60.3-linux-amd64.tar.gz && \
        tar -xzf golangci-lint-1.60.3-linux-amd64.tar.gz && \
        mv ./golangci-lint-1.60.3-linux-amd64/golangci-lint /usr/local/dl/bin/golangci-lint; \
    fi && \
    # Make binaries executable
    chmod +x /usr/local/dl/bin/*

# Default git config
# https://github.com/golangci/golangci-lint/issues/4033
RUN git config --global --add safe.directory '*' && \
    git config --global core.fileMode false

# Go to the app dir, delete the temporary dir and create backups dir
WORKDIR /app
RUN rm -rf /app/temp

##############
# START HERE #
##############
