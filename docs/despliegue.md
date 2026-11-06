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

- Existe solo `backend/` (NestJS 12, TypeScript, pnpm, Vitest, oxlint y Prettier).
- El frontend Next.js todavía no existe.
- No hay Dockerfile ni `docker-compose`.
- No hay CD ni despliegue de backend.
- El CI está configurado en `.github/workflows/ci.yml` y corre en `push` a `main` y en PR hacia `main`.
- La base de datos Supabase/PostgreSQL no está conectada.
- El repositorio es un monorepo **en transición**: el destino es `apps/api` + `apps/web` + `packages/contracts`; `backend/` se mantiene hasta el PR de migración (ADR-006).

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
- Variables del frontend (propuestas): URL pública de la API y credenciales de lectura estrictamente necesarias.
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
```

El backend escucha en `PORT` o 5000 por defecto. `pnpm test` no encuentra suites todavía y `pnpm test:e2e` falla con 404 porque el e2e es el esqueleto de NestJS; está documentado en `docs/testing.md`.

## 5. Docker y Compose (propuesta)

Todavía no existe. Propuesta mínima:

- `backend/Dockerfile` multi-stage con pnpm y Node 22.
- `docker-compose.yml` para desarrollo: API y, cuando exista, PostgreSQL con PostGIS.
- Sin secretos en la imagen; variables por entorno.
- No se agrega Docker hasta que el MVP local y el CI estén estables.

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

Los pasos de pruebas están comentados hasta que existan suites. El CI no despliega.

Reglas del repositorio (`AGENTS.md` §13): un cambio entra por PR a `main`; el workflow debe pasar antes del merge; no se trabaja directamente sobre `main`.

## 7. CD y despliegue (propuesta)

- **Frontend:** Vercel, cuando exista `apps/web`.
- **Backend e ingesta:** pendiente de ADR-003. Opciones por evaluar: Railway (citado en la conversación), otro proveedor gestionado o autoalojamiento.
- **Base de datos:** Supabase/PostgreSQL con PostGIS; migraciones versionadas.
- **Migraciones:** se ejecutan en un paso controlado, nunca automáticamente desde el frontend.
- **Rollback:** conservar el artefacto anterior y un respaldo de la base antes de migraciones destructivas.

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
2. Crear `apps/web` cuando exista frontend.
3. Crear `packages/contracts` solo cuando haya tipos compartidos reales.
4. Actualizar `ci.yml`, `package.json` y rutas en el mismo PR de migración.
5. No mover código durante la fase de documentación.

## 9. Runbook mínimo

1. Instalar dependencias: `cd backend && pnpm install`.
2. Levantar: `cd backend && pnpm start:dev`.
3. Verificar salud: `GET /health` (cuando exista).
4. Probar: `pnpm test`, `pnpm test:e2e`.
5. Calidad: `pnpm lint`, `pnpm format:check`, `pnpm audit`, `pnpm build`.
6. Ante un fallo de despliegue: volver al artefacto anterior y revisar logs; no borrar datos.

## 10. Decisiones abiertas

- ADR-002: Python vs. n8n para la ingesta.
- ADR-003: alojamiento del backend y la ingesta.
- ADR-004: proveedor de tiles para producción.
- Versiones de Docker y Node para producción.
- Estrategia de migraciones y respaldos.
- Dominio y certificados.
- Límites de CI/CD gratuitos y secretos.

## 11. Cómo entra en acción

- **Desarrollo:** comandos y variables locales.
- **CI:** validaciones automáticas en cada PR.
- **Producción:** frontend en Vercel; backend/ingesta según ADR-003.
- **Migraciones:** base de datos y despliegue controlado.
- **Operación:** runbook y rollback.

## 12. Cuándo actualizar este documento

- Al agregar Docker, CI, CD o un entorno nuevo.
- Al cambiar variables, dominios o proveedores.
- Al ejecutar la migración al monorepo.
- Si el workflow real cambia.
