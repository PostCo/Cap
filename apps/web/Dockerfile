# syntax=docker.io/docker/dockerfile:1

FROM node:22-alpine AS base
RUN corepack enable

FROM base AS builder
WORKDIR /app
# COPY . .

# Copy everything and debug
COPY . .

# Debug: Let's see what we actually have
RUN echo "=== Current directory ===" && pwd
RUN echo "=== Files in current directory ===" && ls -la
RUN echo "=== Files in parent directory ===" && ls -la ../ || echo "No parent directory"
RUN echo "=== Files in grandparent directory ===" && ls -la ../../ || echo "No grandparent directory"
RUN echo "=== Looking for pnpm-lock.yaml ===" && find . -name "pnpm-lock.yaml" -type f || echo "No pnpm-lock.yaml found"

RUN corepack enable pnpm
RUN --mount=type=cache,id=pnpm,target=/root/.local/share/pnpm/store pnpm i --frozen-lockfile

ARG NEXT_PUBLIC_DOCKER_BUILD=true
ENV NEXT_PUBLIC_WEB_URL=http://localhost:3000
ENV NEXT_PUBLIC_CAP_AWS_BUCKET=capso
ENV NEXT_PUBLIC_CAP_AWS_REGION=us-east-1

RUN pnpm run build:web


# 3. Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001


# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=builder --chown=nextjs:nodejs /app/apps/web/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/apps/web/.next/static ./apps/web/.next/static
COPY --from=builder --chown=nextjs:nodejs /app/apps/web/public ./apps/web/public
COPY --from=builder --chown=nextjs:nodejs /app/packages/database/migrations ./apps/web/migrations


USER nextjs

EXPOSE 3000

CMD HOSTNAME="0.0.0.0" node apps/web/server.js
