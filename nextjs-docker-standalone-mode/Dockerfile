FROM node:20-alpine AS base
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable 

# 2. Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY . .
RUN pnpm install --frozen-lockfile
RUN pnpm build:nextjs-app

FROM base AS runner
WORKDIR /app

ENV NODE_ENV=production

RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

COPY --from=builder /app/apps/nextjs-app/public ./apps/nextjs-app/public
COPY --from=builder --chown=nextjs:nodejs /app/apps/nextjs-app/.next ./apps/nextjs-app/.next
COPY --from=builder /app/apps/nextjs-app/package.json ./apps/nextjs-app/package.json

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=builder --chown=nextjs:nodejs /app/apps/nextjs-app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/apps/nextjs-app/.next/static ./apps/nextjs-app/.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

WORKDIR /app/apps/nextjs-app

CMD ["node", "server.js"]
