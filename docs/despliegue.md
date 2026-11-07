# Despliegue — MetroAlerta

| Campo | Valor |
|---|---|
| Estado | creado |
| Fecha | 2026-11-06 |
| Fuentes | `AGENTS.md` §4, §9 y §12, `docs/arquitectura.md`, `docs/adr/0003-alojamiento.md`, `docs/adr/0006-estructura-repositorio.md`, `.github/workflows/ci.yml`, `backend/package.json` |
| Propósito | Documentar cómo se ejecuta, prueba, integra y despliega el proyecto. |
| Autoridad | El CI real vive en `.github/workflows/ci.yml`; si este documento lo contradice, manda el workflow. |

**Para agentes:**
- No inventes entornos, dominios ni credenciales: lo que no esté verificado se marca `TODO(confirmar)`.
- No agregues secretos al repositorio.
- Cualquier cambio de Docker, CI/CD o despliegue requiere aprobación (`AGENTS.md` §9).

---

## 1. Estado actual

- Existen `backend/` (NestJS 12, TypeScript, pnpm, Vitest, oxlint y Prettier) y `frontend/` (Next.js 16.4.5, React 19.3, Tailwind 4.3.4, scaffold).
- El frontend no tiene API client, Leaflet, PWA ni pruebas; solo el scaffold de `create-next-app`.
- Hay `backend/Dockerfile`, `frontend/Dockerfile` y `docker-compose.yml` vacíos: son placeholders, no una implementación funcional.
- No hay CD ni despliegue de backend; Vercel está previsto para el frontend.
- El CI está configurado en `.github/workflows/ci.yml` y corre en `push` a `main` y en PR hacia `main`; valida backend y frontend.
- La base de datos Supabase/PostgreSQL no está conectada.
- El repositorio es un monorepo **en transición**: el destino es `apps/api` + `apps/web` + `packages/contracts`; `backend/` y `frontend/` se mantienen hasta el PR de migración (ADR-006).

## 2. Entornos

| Entorno | Uso | Datos | Estado |
|---|---|---|---|
| Local | Desarrollo y pruebas manuales | Simulados; nunca producción | Activo |
| CI | Lint, formato, auditoría y build | Sin base de datos | Activo |
| Producción | Demo y validación | Por definir; Supabase separado | Pendiente |

Reglas:

- los entornos no comparten credenciales ni bases de datos;
- no se accede a producción desde el entorno local;
- los datos simulados se identifican como `simulado`.

## 3. Variables y secretos

- El repositorio solo lleva `.env.example` sin valores.
- Variables del backend (propuestas): `PORT`, `DATABASE_URL`, `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `ADMIN_TOKEN` o credenciales de administrador.
- La `service_role` de Supabase es **solo backend**; nunca se expone en el frontend.
- Variables del frontend (propuestas): URL pública de la API (`NEXT_PUBLIC_API_URL`) y solo variables públicas; nunca secretos.
- `frontend/.gitignore` ignora `.env*`; si se crea `frontend/.env.example`, agregar `!.env.example` para versionarlo.
- Secretos de Vercel y del hosting se configuran en sus paneles, no en Git.
- `.env`, `.env.*.local`, credenciales y artefactos de build están fuera del repositorio.

## 4. Comandos locales verificados

Desde la raíz del repositorio:

```bash
cd backend && pnpm install
cd backend && pnpm start:dev
cd backend && pnpm test
cd backend && pnpm test:e2e
cd backend && pnpm test:cov
cd backend && pnpm lint
cd backend && pnpm format
cd backend && pnpm format:check
cd backend && pnpm build

cd frontend && pnpm install
cd frontend && pnpm dev
cd frontend && pnpm build
cd frontend && pnpm start
cd frontend && pnpm lint
cd frontend && pnpm format:check
```

El backend escucha en `PORT` o 5000 por defecto; el frontend usa el puerto 3000. `pnpm test` no encuentra suites todavía y `pnpm test:e2e` falla con 404 porque el e2e es el esqueleto de NestJS; está documentado en `docs/testing.md`. El frontend no tiene runner de pruebas.

## 5. Docker y Compose

### 5.1. Estructura

```text
backend/
  Dockerfile
  .dockerignore
frontend/
  Dockerfile
  .dockerignore
docker-compose.dev.yml
docker-compose.yml
```

Los Dockerfiles usan una imagen base `node:22-bookworm-slim` y activan pnpm 11.6.0 mediante Corepack. Ambos tienen tres etapas:

| Etapa | Propósito |
|---|---|
| `development` | Instala dependencias y ejecuta el servidor de desarrollo. |
| `build` | Instala dependencias y genera el build de la aplicación. |
| `production` | Instala dependencias de producción y copia los artefactos generados en la etapa `build`. |

Detalles por componente:

- **Backend:** la etapa de desarrollo ejecuta `pnpm start:dev`; la etapa de producción ejecuta `node dist/main.js`. Expone el puerto 5000.
- **Frontend:** la etapa de desarrollo ejecuta `pnpm dev --hostname 0.0.0.0`; la etapa de producción ejecuta `pnpm start`. Expone el puerto 3000.
- Los `.dockerignore` excluyen dependencias instaladas localmente, artefactos de build, cobertura, `.git`, archivos `.env` y logs del contexto de construcción.

### 5.2. Desarrollo

Levanta los dos servicios desde la raíz:

```bash
docker compose -f docker-compose.dev.yml up --build
```

Para ejecutar los contenedores en segundo plano:

```bash
docker compose -f docker-compose.dev.yml up --build -d
```

El entorno de desarrollo monta el código local de `backend/` y `frontend/` dentro de los contenedores. Usa volúmenes nombrados para conservar separados `node_modules` y el directorio `.next` del frontend, evitando que los montajes del código local los reemplacen.

Servicios y puertos:

- Backend: `http://localhost:5000`
- Frontend: `http://localhost:3000`

Comandos útiles:

```bash
docker compose -f docker-compose.dev.yml logs -f
docker compose -f docker-compose.dev.yml ps
docker compose -f docker-compose.dev.yml down
```

### 5.3. Producción local

Construye y levanta las imágenes con las etapas de producción:

```bash
docker compose up --build -d
```

Servicios y puertos:

- Backend: `http://localhost:5000`
- Frontend: `http://localhost:3000`

Comandos útiles:

```bash
docker compose logs -f
docker compose ps
docker compose down
```

Este perfil permite probar localmente los contenedores construidos para producción. No publica imágenes en un registry ni configura un servidor, dominio, TLS o proveedor de hosting.

### 5.4. Límites actuales

- Compose no configura todavía una base de datos; Supabase/PostgreSQL sigue pendiente de integración.
- No hay `healthcheck` ni `depends_on` configurados entre frontend y backend.
- No hay pipeline CI/CD para construir, publicar o desplegar imágenes.
- No se debe incluir información sensible en los archivos de entorno versionados ni en las imágenes.
- No se ha definido un usuario no root en las imágenes; revisar este punto antes de usar los contenedores en un entorno público.

## 6. CI actual

Workflow `.github/workflows/ci.yml`:

1. `actions/checkout`
2. `pnpm/action-setup` con `backend/package.json`
3. `actions/setup-node` con Node 22 y caché de pnpm
4. `pnpm install --frozen-lockfile`
5. `pnpm lint`
6. `pnpm format:check`
7. `pnpm audit`
8. `pnpm build`

Los pasos de pruebas están comentados hasta que existan suites. El CI no despliega y tiene dos jobs: `build-backend` y `build-frontend`. El job de frontend ejecuta instalación, `lint`, `audit`, `build`, `format:check` y `tsc --noEmit`.

Reglas del repositorio (`AGENTS.md` §13): un cambio entra por PR a `main`; el workflow debe pasar antes del merge; no se trabaja directamente sobre `main`.

## 7. CD y despliegue (propuesta)

- **Frontend:** Vercel, cuando se decida desplegar; hoy existe `frontend/` y el destino es `apps/web`.
- **Backend e ingesta:** pendiente de ADR-003. Opciones por evaluar: Railway (citado en la conversación), otro proveedor gestionado o autoalojamiento.
- **Base de datos:** Supabase/PostgreSQL con PostGIS; migraciones versionadas.
- **Migraciones:** se ejecutan en un paso controlado, nunca automáticamente desde el frontend.
- **Rollback:** conservar el artefacto anterior y un respaldo de la base antes de migraciones destructivas.
- **Docker Compose:** por ahora permite ejecutar localmente los perfiles de desarrollo y producción; no reemplaza la configuración de CD ni el despliegue externo.

## 8. Migración al monorepo

Destino:

```text
apps/
  api/
  web/
packages/
  contracts/
  config/        # opcional
docs/
```

Pasos previstos:

1. Crear `apps/api` y mover el backend.
2. Crear `apps/web` y mover el frontend.
3. Crear `packages/contracts` solo cuando haya tipos compartidos reales.
4. Actualizar `ci.yml`, `package.json` y rutas en el mismo PR de migración.
5. No mover código durante la fase de documentación.

## 9. Runbook mínimo

### Ejecución sin Docker

1. Instalar dependencias: `cd backend && pnpm install` y `cd frontend && pnpm install`.
2. Levantar backend: `cd backend && pnpm start:dev`; frontend: `cd frontend && pnpm dev`.
3. Verificar salud: `GET /health` (cuando exista).
4. Probar backend: `pnpm test`, `pnpm test:e2e`; frontend: sin runner todavía.
5. Calidad: `pnpm lint`, `pnpm format:check`, `pnpm audit`, `pnpm build`; frontend: `pnpm lint` y `pnpm build`.

### Ejecución con Docker

1. Verificar que existen los cuatro archivos de entorno bajo `docker/` y que contienen únicamente valores apropiados para el entorno local.
2. Desarrollo: `docker compose -f docker-compose.dev.yml up --build`.
3. Producción local: `docker compose up --build -d`.
4. Consultar estado y logs con `docker compose ps` y `docker compose logs -f` (agregar `-f docker-compose.dev.yml` para el entorno de desarrollo).
5. Detener los servicios con `docker compose down` (agregar `-f docker-compose.dev.yml` para el entorno de desarrollo).

Ante un fallo, revisar primero los logs de los servicios. En un despliegue externo, volver al artefacto anterior y revisar el estado de la base de datos; no borrar datos como medida de recuperación.

## 10. Decisiones abiertas

- ADR-002: Python vs. n8n para la ingesta.
- ADR-003: alojamiento del backend y la ingesta.
- ADR-004: proveedor de tiles para producción.
- Incorporar healthchecks y definir dependencias entre servicios cuando exista una condición real de disponibilidad.
- Definir usuario no root y endurecimiento de las imágenes antes del despliegue público.
- Integrar construcción/publicación de imágenes en CI/CD cuando se defina el proveedor.
- Runner de pruebas del frontend (Vitest/Testing Library/Playwright).
- Estrategia de migraciones y respaldos.
- Dominio y certificados.
- Límites de CI/CD gratuitos y secretos.

## 11. Cómo entra en acción

- **Desarrollo:** comandos pnpm locales o `docker-compose.dev.yml`.
- **Producción local:** `docker-compose.yml` con las etapas de producción.
- **CI:** validaciones automáticas en cada PR.
- **Producción:** frontend en Vercel; backend/ingesta según ADR-003.
- **Migraciones:** base de datos y despliegue controlado.
- **Operación:** runbook y rollback.

## 12. Cuándo actualizar este documento

- Al cambiar Dockerfiles, Compose, variables o puertos.
- Al agregar CI, CD o un entorno nuevo.
- Al cambiar variables, dominios o proveedores.
- Al ejecutar la migración al monorepo.
- Si el workflow real cambia.
