# Backend image (multi-stage): Node 22 + pnpm 11.6 via Corepack.
# Stages: development, build, production. See docs/despliegue.md §5.
# Pending: non-root user and healthcheck before public deployment.

FROM node:22-bookworm-slim AS base

WORKDIR /app

RUN corepack enable && corepack prepare pnpm@11.6.0 --activate

# -------------------------
# Desarrollo

FROM base AS development

COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./

RUN pnpm install --frozen-lockfile

COPY . .

EXPOSE 5000

CMD ["pnpm", "start:dev"]


# -------------------------
# Compilación

FROM base AS build

COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./

RUN pnpm install --frozen-lockfile

COPY . .

RUN pnpm build


# -------------------------
# Producción

FROM base AS production

COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./

RUN pnpm install --prod --frozen-lockfile

COPY --from=build /app/dist ./dist

EXPOSE 5000

CMD ["node", "dist/main.js"]
