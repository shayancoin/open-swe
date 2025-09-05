# syntax=docker/dockerfile:1.4

############################
# Stage 1 – base deps      #
############################
FROM --platform=$BUILDPLATFORM node:18-alpine AS deps
WORKDIR /app
COPY ../../package.json ../../yarn.lock ./
COPY ../../.yarn ./.yarn
RUN corepack enable && yarn workspaces focus --production

############################
# Stage 2 – build web app  #
############################
FROM --platform=$BUILDPLATFORM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY ../../apps/web ../../apps/web
COPY ../../packages ../../packages
RUN yarn workspace @openswe/web build

############################
# Stage 3 – runtime        #
############################
FROM --platform=$TARGETPLATFORM node:18-alpine AS runtime
WORKDIR /app
ENV NODE_ENV=production
ENV PORT=8080
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/apps/web/.next ./.next
COPY --from=builder /app/apps/web/public ./public
COPY ../../apps/web/next.config.mjs ./next.config.mjs
EXPOSE 8080
RUN apk add --no-cache tini
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "./node_modules/next/dist/bin/next", "start", "-p", "8080"]
