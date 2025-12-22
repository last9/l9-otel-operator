# Multi-stage build for Last9 Node.js 10 auto-instrumentation image
# Based on OpenTelemetry operator autoinstrumentation pattern
# Uses OpenTelemetry JS v0.25.0 (last version supporting Node 10)

# Build stage
FROM node:10-alpine AS build

WORKDIR /operator-build

# Copy package.json
COPY package.json ./

# Install dependencies
RUN npm install --production --ignore-scripts

# Final stage - minimal image
FROM busybox

# Copy node_modules from build stage
COPY --from=build /operator-build/node_modules /autoinstrumentation/node_modules

# Copy package.json for version reference
COPY --from=build /operator-build/package.json /autoinstrumentation/package.json

# Set proper permissions
RUN chmod -R go+r /autoinstrumentation

# The operator will copy these files into the target container
# and inject them via NODE_OPTIONS or --require flag
