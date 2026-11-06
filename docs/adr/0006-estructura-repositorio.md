# ADR-006 — Estructura del repositorio

| Campo | Valor |
|---|---|
| Estado | aceptado |
| Fecha | 2026-11-06 |
| Decisores | Desarrollador |
| Documentos relacionados | `AGENTS.md` §12, `docs/arquitectura.md` §3 |

## Contexto

El repositorio contiene hoy solo el backend NestJS en `backend/`. El MVP requerirá un frontend Next.js, contratos compartidos y, más adelante, un proceso de ingesta. Se necesita una estructura que permita compartir tipos sin duplicarlos y mantener un solo CI.

## Decisión

Adoptar un **monorepo** con este destino:

```text
apps/
  api/          # NestJS
  web/          # Next.js (PWA)
packages/
  contracts/    # tipos y DTOs compartidos
  config/       # configuración compartida (opcional)
docs/
```

La migración es **incremental**: `backend/` se mantiene como está hasta un PR dedicado que mueva el código a `apps/api` y actualice el CI en el mismo cambio. No se mueven archivos durante la fase de documentación.

## Consecuencias

**Positivas:**
- Contratos compartidos entre frontend y backend.
- Un solo pipeline y una sola historia de versiones.
- Espacio claro para `apps/web` y para la ingesta.

**Negativas:**
- El repositorio crece y el CI debe filtrar por rutas.
- Un cambio compartido puede afectar a varias aplicaciones.

**Mitigaciones:**
- `packages/contracts` solo contiene interfaces de comunicación, nunca dominio ni implementaciones.
- El CI se actualiza en el mismo PR de migración.
- No se crean carpetas vacías: cada `app` o `package` nace cuando tiene contenido real.

## Alternativas consideradas

- **Repositorios separados:** aislamiento, pero duplica contratos y coordinación.
- **Mantener `backend/` en la raíz y añadir `frontend/`:** posible, pero no escala a contratos compartidos ni a la ingesta.
- **Monorepo con apps en la raíz:** menos convencional y más difícil de filtrar en CI.

## Referencias

- `AGENTS.md` §12 (estructura y comandos).
- `docs/arquitectura.md` §3.
