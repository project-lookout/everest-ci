# syntax=docker/dockerfile:1

# Build argument for the dist directory path
ARG DIST_PATH=./dist

# Stage 1: Copy dist artifact and find the binary
FROM alpine:latest AS dist-copy
ARG DIST_PATH
COPY ${DIST_PATH} /dist

# Find the everest-core binary and create a symlink in a known location
RUN find /dist -name "everest-core" -type f -executable | head -1 > /everest-core-path.txt && \
    if [ -s /everest-core-path.txt ]; then \
        EVEREST_CORE_PATH=$(cat /everest-core-path.txt); \
        mkdir -p /dist/bin; \
        ln -s "$EVEREST_CORE_PATH" /dist/bin/everest-core || \
        cp "$EVEREST_CORE_PATH" /dist/bin/everest-core; \
        chmod +x /dist/bin/everest-core; \
    fi && \
    echo "Dist directory structure:" && \
    find /dist -type f -name "*everest*" -o -name "*core*" | head -20 && \
    echo "Looking for binaries:" && \
    find /dist -type f -executable | head -20

# Stage 2: Runtime container
FROM ghcr.io/everest/everest-ci/run-env-base:latest

# Copy the installed binaries from dist
COPY --from=dist-copy /dist /opt/everest

# Set working directory
WORKDIR /opt/everest

# Add common binary paths to PATH
ENV PATH="/opt/everest/bin:/opt/everest/libexec/everest:/opt/everest/libexec:${PATH}"

# Find and set the correct entrypoint
RUN if [ -f /opt/everest/bin/everest-core ]; then \
        echo "Found everest-core at /opt/everest/bin/everest-core"; \
    elif EVEREST_CORE=$(find /opt/everest -name "everest-core" -type f -executable | head -1); then \
        echo "Found everest-core at $EVEREST_CORE"; \
        mkdir -p /opt/everest/bin; \
        ln -s "$EVEREST_CORE" /opt/everest/bin/everest-core || \
        cp "$EVEREST_CORE" /opt/everest/bin/everest-core; \
        chmod +x /opt/everest/bin/everest-core; \
    else \
        echo "ERROR: everest-core binary not found!"; \
        echo "Contents of /opt/everest:"; \
        ls -laR /opt/everest | head -50; \
        exit 1; \
    fi

# Expose ports
EXPOSE 8849
EXPOSE 1883

# Health check (optional, adjust based on your setup)
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8849/health || exit 1

# Set entrypoint to run EVerest
ENTRYPOINT ["/opt/everest/bin/everest-core"]

