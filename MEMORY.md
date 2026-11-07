# MEMORY.md — Estado de trabajo

> Memoria temporal del proyecto (máx. 100–150 líneas). Las reglas permanentes viven en `AGENTS.md` y el detalle técnico en `/docs`. No se registran secretos ni credenciales.

---

## Estado actual (al 2026-11-06)

- Proyecto en fase de **fundamentos** (orden de construcción §15): `backend/` es el esqueleto NestJS 12 (`src/main.ts` + `src/app.module.ts`, sin módulos de dominio); `frontend/` existe con Next.js 16.4.5 (scaffold), sin API, mapa, PWA ni pruebas; no hay base de datos conectada ni ingesta.
- Documentación de fundamentos creada y revisada:
  - `docs/requerimientos.md` — RF-01…RF-16, HU-01…HU-06 (GWT), RNF-01…RNF-14, post-MVP y cuestiones abiertas CA-01…CA-05. Estado: creado.
  - `docs/arquitectura.md` — propuesta de referencia (monolito modular + Clean Architecture; módulos `alerts`, `transport`, `route-engine`, `geolocation`, `sharing`) y decisiones registradas en `docs/adr/`. Estado: creado.
  - `docs/tareas-de-investigacion.md` — plan de estudio con 35 tareas (INV-01…INV-35) en 6 asignaturas, prioridades P0–P3. Estado: creado.
  - `docs/funcionalidades-post-mvp.md` — hoja de ruta post-MVP ("MetroAlerta 2.0") desmenuzada paso a paso (PM-01…PM-25, versiones V2/V3/V4). Estado: creado. Nada de allí está implementado ni aprobado para entrar al MVP.
  - `docs/trazabilidad.md` — matriz F01–F05 → RF → HU → RNF → INV → PM y correspondencia INV ↔ PM. Estado: creado.
  - `docs/testing.md` — estrategia de pruebas por componente, estado real del CI y pendientes. Estado: creado.
  - `docs/dominio.md` — glosario, estados, entidades e invariantes; CA-04 y PC-06 resueltos. Estado: creado.
  - `docs/fuentes-de-datos.md` — inventario de fuentes; todas `por verificar` hasta ejecutar INV-01…INV-09. Estado: creado.
  - `docs/modelo-de-datos.md` — modelo conceptual/lógico/físico propuesto: PostGIS, segmentos por conexión y mantenimiento separado. Estado: creado.
  - `docs/api.md` — contrato REST propuesto del MVP, administración protegida y `wa.me`. Estado: creado.
  - `docs/despliegue.md` — estado actual, CI, entornos, Docker/CD pendiente y migración a monorepo. Estado: creado.
  - `docs/adr/` — ADR-001, ADR-005 y ADR-006 aceptados; ADR-002, ADR-003 y ADR-004 propuestos. Estado: creado.
- La tabla de documentos de `AGENTS.md` §13 ya no tiene pendientes; la completitud se sigue en la sección “Documentos creados con información pendiente”.
- Reglas locales: `backend/AGENTS.md` y `backend/MEMORY.md`; `frontend/AGENTS.md` y `frontend/MEMORY.md`. No reemplazan este archivo ni `AGENTS.md`.

## Tareas en progreso

- Ninguna en ejecución. Siguiente paso concreto: **Asignatura 1 (P0)** — ejecutar INV-01…INV-09 y completar `docs/fuentes-de-datos.md` con evidencia real; luego validar el modelo de datos y el Route Engine.

## Backlog corto

1. INV-01…INV-09 — verificar fuentes y completar `docs/fuentes-de-datos.md`.
2. INV-10…INV-13 — validar cifras del problema y diferenciación (P0/P1).
3. Cerrar ADR-002/003/004 (ingesta, alojamiento y tiles).
4. Prueba de concepto: Dijkstra + red simulada de 10–20 nodos + exclusión de segmentos.
5. Primer módulo backend real: `alerts`, siguiendo `docs/arquitectura.md` §4–§5.
6. Frontend: shell, metadata en español, cliente API, contingencias, mapa y PWA.

## Decisiones recientes (con fecha)

- 2026-11-05 — Requerimientos y arquitectura se basan en la conversación "Plan de MetroAlerta" (ChatGPT, 2026-11-03) más `AGENTS.md`. La conversación es segunda fuente: cuando contradice al código o a `AGENTS.md`, mandan estos últimos.
- 2026-11-05 — La propuesta de arquitectura del documento queda explícita como **propuesta de referencia** hasta que existan ADRs; el código real (esqueleto NestJS) manda sobre todo lo escrito.
- 2026-11-05 — `docs/tareas-de-investigacion.md` se agregó a la tabla de documentos de `AGENTS.md` §13 y se creó este `MEMORY.md` en la raíz (aprobado por el desarrollador en sesión).
- 2026-11-05 — Se creó `docs/funcionalidades-post-mvp.md`: la sección "MetroAlerta 2.0" de la conversación queda desmenuzada paso a paso como hoja de ruta propuesta (V2 asistencia, V3 institucional, V4 investigación). No es compromiso: ninguna funcionalidad entra al MVP sin aprobación explícita; las exclusiones de `AGENTS.md` §2 se mantienen.
- 2026-11-05 — Se recuperaron los enlaces del chat desde el JSON del backend-api: la extracción `.md` no conservó los `href`, pero las URLs de los documentos eran correctas. Ahora se citan con la marca "verificar" por vigencia.
- 2026-11-06 — ADR-001 (monolito modular), ADR-005 (PostGIS + granularidad por conexión) y ADR-006 (monorepo) aceptados. CA-01 (panel admin protegido), CA-02 (`wa.me` + Web Share), CA-04 (conexión) y PC-06 (mantenimiento separado) resueltos.
- 2026-11-06 — Frontend confirmado con Next.js 16.4.5, React 19.3.0 y Tailwind 4.3.4; App Router, Server Components por defecto y documentación local de Next.js como fuente.
- 2026-11-06 — Docker queda como placeholder (Dockerfiles y compose vacíos/documentados), no implementado.

## ADRs

- **Aceptados:** ADR-001 (estilo arquitectónico), ADR-005 (PostGIS y granularidad) y ADR-006 (monorepo).
- **Propuestos:** ADR-002 (ingesta), ADR-003 (alojamiento) y ADR-004 (tiles).
- Detalle y estado en `docs/adr/README.md` y `docs/arquitectura.md` §10.

## Problemas / bloqueos abiertos

- Sin fuentes de datos verificadas ni muestras inspeccionadas (riesgo principal).
- Cuestiones abiertas de `docs/requerimientos.md`: CA-03 (parámetros de estimación) y CA-05 (umbrales de rendimiento).
- ADR-002, ADR-003 y ADR-004 siguen propuestos.
- `docs/modelo-de-datos.md` es conceptual; faltan muestras reales y migraciones.
- `docs/api.md` es un contrato propuesto; no hay endpoints implementados.
- Frontend sin cliente API, mapa, PWA ni runner de pruebas; `pnpm lint`, `pnpm format:check` y `pnpm build` disponibles y validados.
- Docker: archivos placeholder; no implementado.
- Las pruebas siguen sin suites y el CI mantiene los pasos de test comentados; el CI sí valida lint, formato, auditoría y build del frontend.

## Contexto temporal

- Pistas recuperadas del JSON del chat (vigencia por verificar): Metro (<https://www.metrodemedellin.gov.co/>), AMVA (<https://datosabiertos.metropol.gov.co/>), Medata (<https://www.medellin.gov.co/es/transparencia/medata/>), GTFS (<https://hub.tumidata.org/es/dataset/gtfs_metro_medellin_medelln>), ArcGIS (<https://services3.arcgis.com/ARUXxWVTK25ENUfB/ArcGIS/rest/services/Sistema_Metro2/FeatureServer>), Trafi (<https://medium.com/trafi/trafi-introducing-real-time-disruption-alerts-5e8740627f14>), BSAG (<https://www.bsag.de/unternehmen/aktuelles/meldung/whatsapp-kanal-bsag-fahrgastinformation>).
- La extracción `.md` no conserva enlaces; para citas usar el JSON del backend-api. Los documentos ya no atribuyen URLs sin poder rastrearlas.
- Cifras del problema (1,2 millones de usuarios, 41.000 pasajeros/hora, 29%): hipótesis sin verificar (INV-10); no presentarlas como hechos.
- Referencia de librerías/plugins: `@nestjs/observe` está instalado y en uso, pero sin entender su alcance (INV-20); el repo usa oxlint, no ESLint.

## Historial de sesiones

- 2026-11-05 — Se crearon los documentos de fundamentos a partir de la conversación compartida; revisión completa (typos, referencias y coherencia con `AGENTS.md` y con el código real); estados actualizados en `AGENTS.md` §13; creado este `MEMORY.md`.
- 2026-11-05 — Hoja de ruta post-MVP desmenuzada en `docs/funcionalidades-post-mvp.md` (PM-01…PM-25); fila agregada a `AGENTS.md` §13 con estado "creado".
- 2026-11-05 — Auditoría y enriquecimiento: enlaces recuperados del JSON del chat, `docs/trazabilidad.md` creado, referencias cruzadas corregidas y `AGENTS.md` actualizado (jerarquía de fuentes, familias de IDs, comandos reales).
- 2026-11-05 — Verificación final: IDs y tablas sin errores; los únicos enlaces "faltantes" son documentos pendientes (`dominio.md`, `api.md`, etc.); prueba de agente limpio 8/8. Se aclaró que un archivo vacío (`testing.md`) no cuenta como documento creado.
- 2026-11-05 — Estrategia de pruebas creada en `docs/testing.md`; se documentó que `pnpm test` no encuentra suites y que el e2e esqueleto falla con 404, por lo que el CI mantiene las pruebas comentadas.
- 2026-11-06 — Completados los documentos faltantes: dominio, fuentes, modelo de datos, API, despliegue y carpeta ADR; decisiones cerradas y tabla de completitud en `AGENTS.md` §13.
- 2026-11-06 — Creados `frontend/AGENTS.md` y `frontend/MEMORY.md`; documentados Next.js 16.4.5, la estructura frontend y Docker como placeholder no implementado.
- 2026-11-06 — Frontend con Prettier y CI separado; `pnpm audit` corregido con override de `minimatch >=11.0.1`.
