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
        ffmpeg wget zip unzip p7zip-full tzdata git python3 python3-pip && \
    rm -rf /var/lib/apt/lists/*

# Install torch CPU
RUN pip install --break-system-packages torch==2.4.1 torchvision==0.19.1 torchaudio==2.4.1 --index-url https://download.pytorch.org/whl/cpu

# Install F5-TTS
RUN pip install --break-system-packages git+https://github.com/jpgallegoar/Spanish-F5.git

# Install taskfile
RUN go install github.com/go-task/task/v3/cmd/task@v3.41.0

# Go to the app dir, delete the temporary dir and create backups dir
WORKDIR /app
RUN rm -rf /app/temp

##############
# START HERE #
##############
