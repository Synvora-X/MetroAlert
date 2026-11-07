# Placeholder: frontend Docker is not implemented yet.
# When implemented: multi-stage build for Next.js and a minimal runtime image.

FROM node:22-bookworm-slim AS base

WORKDIR /app

RUN corepack enable && corepack prepare pnpm@11.6.0 --activate

# -------------------------
# Desarrollo

FROM base AS development

COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./

RUN pnpm install --frozen-lockfile

COPY . .

EXPOSE 3000

CMD ["pnpm", "dev", "--hostname", "0.0.0.0"]


# -------------------------
# Compilación

FROM base AS build

ENV NEXT_TELEMETRY_DISABLED=1

COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./

RUN pnpm install --frozen-lockfile

COPY . .

RUN mkdir -p public && pnpm build


# -------------------------
# Producción

FROM base AS production

COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./

RUN pnpm install --prod --frozen-lockfile

COPY --from=build /app/.next ./.next
COPY --from=build /app/public ./public
COPY --from=build /app/next.config.ts ./next.config.ts

EXPOSE 3000

CMD ["pnpm", "start"]
