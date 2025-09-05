# syntax=docker/dockerfile:1.4

############################
# Stage 1 – dependencies   #
############################
FROM --platform=$BUILDPLATFORM node:18-alpine AS deps
WORKDIR /app
COPY ../../package.json ../../yarn.lock ./
COPY ../../.yarn ./.yarn
RUN corepack enable && yarn workspaces focus --production

############################
# Stage 2 – build v2 agent #
############################
FROM --platform=$BUILDPLATFORM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY ../../apps/open-swe-v2 ../../apps/open-swe-v2
COPY ../../packages ../../packages
RUN yarn workspace @openswe/agent-v2 build

############################
# Stage 3 – runtime        #
############################
FROM --platform=$TARGETPLATFORM node:18-alpine AS runtime
WORKDIR /app
ENV NODE_ENV=production
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/apps/open-swe-v2/dist ./dist
EXPOSE 2025
RUN apk add --no-cache tini
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "./dist/server.js"]
