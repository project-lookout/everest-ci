# syntax=docker/dockerfile:1

# Build argument for the dist directory path
ARG DIST_PATH=./dist

# Stage 1: Copy dist artifact
FROM alpine:latest AS dist-copy
ARG DIST_PATH
COPY ${DIST_PATH} /dist

# Stage 2: Runtime container
FROM ghcr.io/everest/everest-ci/run-env-base:latest

# Copy the installed binaries from dist
COPY --from=dist-copy /dist /opt/everest

# Set working directory
WORKDIR /opt/everest

# Add binaries to PATH
ENV PATH="/opt/everest/bin:${PATH}"

# # Expose ports (adjust based on your EVerest configuration)
# EXPOSE 8849  # Admin panel
# EXPOSE 1883  # MQTT (if used)

# Health check (optional, adjust based on your setup)
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8849/health || exit 1

# Set entrypoint to run EVerest
ENTRYPOINT ["/opt/everest/bin/everest-core"]

