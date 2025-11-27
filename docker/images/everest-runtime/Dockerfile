# syntax=docker/dockerfile:1

# Build argument for the dist directory path
ARG DIST_PATH=./dist

# Stage 1: Copy dist artifact
FROM alpine:latest AS dist-copy
ARG DIST_PATH
COPY ${DIST_PATH} /dist

# Debug: Show the structure of dist directory
RUN echo "=== Dist directory structure ===" && \
    find /dist -type f | head -50 && \
    echo "=== Executable files ===" && \
    find /dist -type f -executable && \
    echo "=== Looking for everest-core ===" && \
    find /dist -name "*everest*" -o -name "*core*" | head -20

# Stage 2: Runtime container
FROM ghcr.io/everest/everest-ci/run-env-base:latest

# Copy the installed binaries from dist
COPY --from=dist-copy /dist /opt/everest

# Set working directory
WORKDIR /opt/everest

# Add common binary paths to PATH
ENV PATH="/opt/everest/bin:/opt/everest/libexec/everest:/opt/everest/libexec:${PATH}"

# Create a wrapper entrypoint script that finds the binary at runtime
RUN echo '#!/bin/bash\n\
set -e\n\
\n\
# Find the everest-core binary\n\
EVEREST_CORE=""\n\
\n\
# Check common locations first\n\
if [ -f /opt/everest/bin/everest-core ]; then\n\
    EVEREST_CORE="/opt/everest/bin/everest-core"\n\
elif [ -f /opt/everest/libexec/everest/everest-core ]; then\n\
    EVEREST_CORE="/opt/everest/libexec/everest/everest-core"\n\
elif [ -f /opt/everest/libexec/everest-core ]; then\n\
    EVEREST_CORE="/opt/everest/libexec/everest-core"\n\
else\n\
    # Search for it\n\
    EVEREST_CORE=$(find /opt/everest -name "everest-core" -type f -executable 2>/dev/null | head -1)\n\
fi\n\
\n\
if [ -z "$EVEREST_CORE" ] || [ ! -f "$EVEREST_CORE" ]; then\n\
    echo "ERROR: everest-core binary not found!" >&2\n\
    echo "Searched in /opt/everest" >&2\n\
    echo "Directory structure:" >&2\n\
    find /opt/everest -type f 2>/dev/null | head -30 >&2\n\
    echo "Executable files:" >&2\n\
    find /opt/everest -type f -executable 2>/dev/null >&2\n\
    exit 1\n\
fi\n\
\n\
echo "Using everest-core at: $EVEREST_CORE"\n\
exec "$EVEREST_CORE" "$@"\n\
' > /entrypoint.sh && chmod +x /entrypoint.sh

# Expose ports
EXPOSE 8849
EXPOSE 1883

# Health check (optional, adjust based on your setup)
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8849/health || exit 1

# Set entrypoint to use the wrapper script
ENTRYPOINT ["/entrypoint.sh"]

