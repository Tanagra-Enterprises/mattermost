FROM ubuntu:22.04

# Build arguments for versions
ARG NODE_VERSION
ARG GO_VERSION

# Install system dependencies including build tools and canvas dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    bash \
    openssh-client \
    sqlite3 \
    python3 \
    python3-pip \
    build-essential \
    make \
    g++ \
    gcc \
    libc6-dev \
    libcairo2-dev \
    libjpeg-dev \
    libpango1.0-dev \
    libgif-dev \
    librsvg2-dev \
    libpixman-1-dev \
    pkg-config \
    autoconf \
    automake \
    libtool \
    nasm \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/* \
    && ln -sf /usr/bin/python3 /usr/bin/python

# Install Node.js
RUN curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /usr/share/keyrings/nodesource.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/nodesource.gpg] https://deb.nodesource.com/node_${NODE_VERSION}.x nodistro main" \
    > /etc/apt/sources.list.d/nodesource.list && \
    apt-get update && apt-get install -y nodejs

# Install Go
RUN curl -L "https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz" | tar -C /usr/local -xzf - && \
    ln -s /usr/local/go/bin/go /usr/local/bin/go && \
    ln -s /usr/local/go/bin/gofmt /usr/local/bin/gofmt

# Create claude user (UID 1001) with restricted access
RUN useradd -m -s /bin/bash -u 1001 claude && \
    chown -R claude:claude /home/claude

# Create entrypoint script as root (must be before USER)
RUN echo '#!/bin/bash' > /entrypoint.sh && \
    echo 'echo "Claude Code container started"' >> /entrypoint.sh && \
    echo 'echo "Workspace mounted at: /workspace"' >> /entrypoint.sh && \
    echo 'tail -f /dev/null' >> /entrypoint.sh && \
    chmod +x /entrypoint.sh

# Set working directory and change ownership
WORKDIR /workspace
RUN chown -R claude:claude /workspace

# Switch to claude user
USER claude

# Setup npm install location
ENV NPM_CONFIG_PREFIX=/home/claude/.npm-global
ENV PATH="/home/claude/.npm-global/bin:${PATH}"

# Disable VCS stamping for Go builds by default
ENV GOFLAGS="-buildvcs=false"

RUN npm install -g @anthropic-ai/claude-code

ENTRYPOINT ["/entrypoint.sh"]