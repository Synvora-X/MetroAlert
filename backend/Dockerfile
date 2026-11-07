# Placeholder: backend Docker is not implemented yet.
# When implemented: multi-stage build with pnpm, Node 22 and a non-root user.

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
