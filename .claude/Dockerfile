# Build arguments for versions
ARG NODE_VERSION
ARG GO_VERSION

FROM node:${NODE_VERSION}-alpine

# Install system dependencies
RUN apk add --no-cache \
    git \
    curl \
    bash \
    openssh-client \
    sqlite

# Install Go to match Mattermost requirements
RUN curl -L "https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz" | tar -C /usr/local -xzf - && \
    ln -s /usr/local/go/bin/go /usr/local/bin/go && \
    ln -s /usr/local/go/bin/gofmt /usr/local/bin/gofmt

# Create claude user (UID 1001) with restricted access
RUN adduser -D -s /bin/bash -u 1001 claude
RUN chown -R claude:claude /home/claude

# Create entrypoint script as root (must be before USER)
RUN echo '#!/bin/bash' > /entrypoint.sh && \
    echo 'echo "Claude Code container started"' >> /entrypoint.sh && \
    echo 'echo "Workspace mounted at: /workspace"' >> /entrypoint.sh && \
    echo 'claude' >> /entrypoint.sh && \
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