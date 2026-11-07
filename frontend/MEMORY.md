# MEMORY.md — Frontend MetroAlerta

> Memoria local del frontend. No reemplaza `../MEMORY.md`: la raíz guarda el estado y las decisiones del proyecto; este archivo guarda el detalle de implementación del frontend. Si se contradicen, manda `../AGENTS.md` y luego `../MEMORY.md`.

| Campo | Valor |
|---|---|
| Estado | creado |
| Fecha | 2026-11-06 |
| Alcance | Solo `frontend/` |
| Autoridad | `../AGENTS.md` > `frontend/AGENTS.md` > `../MEMORY.md` > este archivo |

## Estado actual (2026-11-06)

- Next.js 16.4.5, React 19.3.0 y Tailwind CSS 4.3.4.
- App Router en `app/`; `layout.tsx`, `page.tsx` y `globals.css` son el scaffold de `create-next-app`.
- `app/layout.tsx` tiene `lang="en"` y metadata `"Create Next App"`: placeholders por reemplazar.
- `next.config.ts` usa `cacheComponents: true`, `partialPrefetching: true` y Turbopack para CSS.
- Sin cliente de API, Leaflet, manifest PWA, service worker ni pruebas.
- `frontend/Dockerfile` es un placeholder vacío; Docker no está implementado.
- `frontend/.gitignore` ignora `.env*`; si se crea `.env.example`, agregar `!.env.example`.
- ADR-006: destino `apps/web`; `frontend/` se mantiene hasta el PR de migración.

## Tareas en progreso

- Ninguna. Siguiente paso: reemplazar el scaffold por el shell de MetroAlerta.

## Backlog corto

1. Shell y navegación principal.
2. Metadata de MetroAlerta y `lang="es"`.
3. Cliente API en `lib/api` contra `../docs/api.md`.
4. Lista de contingencias y estados de carga/error/vacío.
5. Mapa Leaflet en cliente.
6. Origen/destino y alternativas.
7. Compartir por `wa.me`/Web Share.
8. PWA: manifest, service worker y offline.
9. Runner de pruebas y primeras suites.

## Decisiones locales

- App Router y Server Components por defecto.
- `'use client'` solo para estado, eventos, APIs del navegador, Leaflet o Web Share.
- Tailwind CSS 4; tokens en `globals.css`.
- Sin lógica de negocio ni cálculo de rutas en el frontend.
- Sin acceso directo a Supabase.
- Textos de interfaz en español; código e identificadores en inglés.
- Leer la documentación local de Next.js antes de escribir código.
- Prettier + `eslint-config-prettier`; scripts `format`/`format:check`; override `minimatch >=11.0.1` para auditoría.
- Monorepo `apps/web` como destino, sin mover archivos todavía.

## Estado por pantalla / componente

| Pantalla / componente | Estado | Nota |
|---|---|---|
| Inicio | placeholder | `app/page.tsx` es el scaffold |
| Layout y metadata | pendiente | `lang="en"` y “Create Next App” |
| Contingencias | no iniciado | Depende de la API |
| Mapa | no iniciado | Leaflet solo cliente |
| Origen/destino | no iniciado | Formulario y validación |
| Alternativas | no iniciado | Consume `/routes/alternatives` |
| Compartir | no iniciado | `wa.me` + Web Share |
| PWA | no iniciado | Manifest y service worker |
| Pruebas | no iniciado | Sin runner configurado |

## Comandos

```bash
cd frontend && pnpm install
cd frontend && pnpm dev
cd frontend && pnpm build
cd frontend && pnpm start
cd frontend && pnpm lint
cd frontend && pnpm format:check
```

No existe `pnpm test`; no ejecutarlo ni inventarlo.

## Problemas / bloqueos

- API no implementada; el contrato es una propuesta.
- Sin datos reales verificados.
- Sin runner de pruebas.
- Sin Leaflet, PWA ni cliente API.
- `frontend/Dockerfile` vacío; Docker no implementado.
- Migración a monorepo pendiente.
- `.env*` ignorado; exceptuar `.env.example` si se crea.

## Últimos cambios

- 2026-11-06 — Creados `frontend/AGENTS.md` y `frontend/MEMORY.md`.
- 2026-11-06 — Documentado el scaffold de Next.js 16.4.5 y sus placeholders.
- 2026-11-06 — Agregados Prettier, `eslint-config-prettier`, scripts de formato y CI de frontend; `minimatch` actualizado para auditoría.

## Próximo paso

Crear el shell de MetroAlerta: layout en español, metadata correcta, navegación básica y cliente API inicial. Después, la lista de contingencias y el mapa.

## Protocolo de actualización

- Al terminar una tarea del frontend, actualizar este archivo: pantalla, componente, pruebas, bloqueos y próximo paso.
- Usar `../MEMORY.md` solo para estado o decisiones que cruzan componentes.
- No registrar secretos, credenciales ni conversaciones completas.
- Si una decisión cambia, actualizar el ADR o el documento dueño antes de reflejarla aquí.
