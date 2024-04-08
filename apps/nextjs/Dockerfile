# builder packages
FROM --platform=linux/arm64 node:20.12.0-alpine3.18 AS builder
RUN apk add --no-cache libc6-compat openssl1.1-compat && apk update
WORKDIR /app
COPY . .
RUN corepack enable
RUN pnpm i --no-frozen-lockfile && pnpm build
RUN pnpm turbo prune --scope=@acme/nextjs --docker

# build web
FROM --platform=linux/arm64 node:20.12.0-alpine3.18 AS installer
RUN apk add --no-cache libc6-compat openssl1.1-compat && apk update
COPY --from=builder /app/out/full /app/out/full
COPY --from=builder /app/node_modules /app/out/full/node_modules
COPY --from=builder /app/.env /app/out/full/.env
WORKDIR /app/out/full
ENV NODE_ENV production
ENV CI         true
# 暂时跳过环境变量校验
ENV SKIP_ENV_VALIDATION true
RUN pwd
RUN corepack enable
RUN pnpm i --no-frozen-lockfile --prefer-offline --prod=false && pnpm build:web

# runner
FROM --platform=linux/arm64 node:20.12.0-alpine3.18 AS runner
RUN apk add --no-cache libc6-compat openssl1.1-compat && apk update
COPY --from=installer /app/out/full /app/out/full
WORKDIR /app/out/full
RUN corepack enable
EXPOSE 3000
ENV PORT 3000
ENV NODE_ENV production
ENV HOSTNAME 0.0.0.0
ENV NEXT_TELEMETRY_DEBUG 1
# 暂时跳过环境变量校验
ENV SKIP_ENV_VALIDATION true
CMD ["pnpm", "start:web"]
