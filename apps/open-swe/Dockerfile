# syntax=docker/dockerfile:1.4

############################
# Stage 1 – install deps   #
############################
FROM --platform=$BUILDPLATFORM node:18-alpine AS deps
WORKDIR /app

# Copy root manifests so Yarn can workspaces-install
COPY ../../package.json ../../yarn.lock ./
COPY ../../.yarn ./.yarn

# Enable Corepack (Yarn 3) and install *production* deps
RUN corepack enable && \
    yarn workspaces focus --production

############################
# Stage 2 – copy source & build
############################
FROM --platform=$BUILDPLATFORM node:18-alpine AS builder
WORKDIR /app

# Re-use node_modules from previous stage
COPY --from=deps /app/node_modules ./node_modules

# Copy monorepo source (only needed packages) – leverage Docker layer caching
# Hadolint ignore=DL3003
COPY ../../apps/open-swe ../../apps/open-swe
COPY ../../packages ../../packages

# Build the agent (ts → js)
RUN yarn workspace @openswe/agent build

############################
# Stage 3 – runtime image  #
############################
FROM --platform=$TARGETPLATFORM node:18-alpine AS runtime
ENV NODE_ENV=production
WORKDIR /app

# Copy production node_modules & built files
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/apps/open-swe/dist ./dist

# Expose LangGraph agent port
EXPOSE 2024

# Use tini for proper signal handling
RUN apk add --no-cache tini
ENTRYPOINT ["/sbin/tini", "--"]

# Start the server
CMD ["node", "./dist/server.js"]
