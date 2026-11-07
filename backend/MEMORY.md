# MEMORY.md — Backend MetroAlerta

> Memoria local del backend. No reemplaza `../MEMORY.md`: la raíz guarda el estado y las decisiones del proyecto; este archivo guarda el detalle de implementación del backend. Si se contradicen, manda `../AGENTS.md` y luego `../MEMORY.md`.

| Campo | Valor |
|---|---|
| Estado | creado |
| Fecha | 2026-11-06 |
| Alcance | Solo `backend/` |
| Autoridad | `../AGENTS.md` > `backend/AGENTS.md` > `../MEMORY.md` > este archivo |

## Estado actual (2026-11-06)

- NestJS 12 con TypeScript, ESM (`"type": "module"`) y `moduleResolution: nodenext`.
- Solo existen `src/main.ts` y `src/app.module.ts`; no hay módulos de dominio.
- Sin base de datos conectada; Supabase/PostgreSQL y PostGIS siguen pendientes.
- Sin frontend; el backend se prueba de forma aislada.
- `pnpm test` no encuentra suites; `pnpm test:e2e` falla con 404 porque el e2e es el esqueleto de NestJS.
- El CI ejecuta `lint`, `format:check`, `audit` y `build`; los tests están comentados.
- `backend/.env.example` existe con variables vacías: `PORT`, `DATABASE_URL`, `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `ADMIN_TOKEN`.
- `backend/pnpm-workspace.yaml` declara un workspace local con `packages: ["."]`; se revisará en la migración al monorepo.
- `@nestjs/observe` está instalado, pero la integración está incompleta: falta `ObserveModule.forRoot(...)` y no hay credenciales. No se considera observabilidad activa.

## Tareas en progreso

- Ninguna. Siguiente paso: crear `shared/` y `config/`, y el primer módulo real `alerts`.

## Backlog corto

1. `shared/config` + endpoint de salud.
2. Módulo `alerts` con estados, verificación y pruebas unitarias.
3. POC de `route-engine`: grafo de 10–20 nodos + Dijkstra + exclusión de segmentos.
4. Conexión a Supabase/PostGIS después de completar las fuentes.
5. Migración a `apps/api` con actualización del CI.

## Decisiones locales

- Código e identificadores en inglés; documentación `.md` y mensajes de usuario en español.
- Estructura por módulo: `domain/`, `application/`, `infrastructure/`, `presentation/` y `<module>.module.ts`.
- Dependencias hacia adentro; el dominio no conoce NestJS, Supabase ni HTTP.
- Vitest con config unitaria y e2e separadas; Supertest para HTTP.
- Puerto por defecto: 5000 (`PORT` si existe).
- Monorepo como destino (ADR-006), sin mover `backend/` todavía.
- `@nestjs/observe` queda como scaffold pendiente; no se usa como requisito del MVP.
- No se agregan dependencias, migraciones ni cambios de API sin aprobación.

## Estado por módulo

| Módulo | Estado | Nota |
|---|---|---|
| `alerts` | no iniciado | Primer módulo real; incluye estados y verificación |
| `transport` | no iniciado | Red, nodos y segmentos; espera fuentes |
| `route-engine` | no iniciado | Depende de `transport` y del POC de grafos |
| `geolocation` | no iniciado | Puede integrarse en `transport`/`route-engine` |
| `sharing` | no iniciado | Mensaje y enlace `wa.me` |
| `shared` | no iniciado | Filtros, errores y utilidades técnicas |
| `config` | no iniciado | Configuración tipada y parámetros |

## Comandos

```bash
cd backend && pnpm install
cd backend && pnpm start:dev
cd backend && pnpm test
cd backend && pnpm test:e2e
cd backend && pnpm test:cov
cd backend && pnpm lint
cd backend && pnpm format:check
cd backend && pnpm build
```

## Problemas / bloqueos

- Sin suites de prueba; el e2e actual no representa el dominio.
- Sin base de datos ni migraciones.
- Fuentes de datos sin verificar (`../docs/fuentes-de-datos.md`).
- `@nestjs/observe` incompleto; no usar como observabilidad real.
- Migración a monorepo pendiente; `pnpm-workspace.yaml` local se revisará entonces.
- El CI no ejecuta tests hasta que exista la primera suite.

## Últimos cambios

- 2026-11-06 — Creados `backend/AGENTS.md` y `backend/MEMORY.md`.
- 2026-11-06 — Agregado `backend/.env.example` sin valores.
- 2026-11-06 — Documentados el esqueleto, los comandos y los bloqueos actuales.

## Próximo paso

Crear `shared/` y `config/`, luego el módulo `alerts` con su primera prueba unitaria. Antes de conectar la base de datos, completar las fuentes y el modelo de datos.

## Protocolo de actualización

- Al terminar una tarea del backend, actualizar este archivo: estado, módulo, pruebas, bloqueos y próximo paso.
- Usar `../MEMORY.md` solo para estado o decisiones que cruzan componentes.
- No registrar secretos, credenciales ni conversaciones completas.
- Si una decisión cambia, actualizar el ADR o el documento dueño antes de reflejarla aquí.
