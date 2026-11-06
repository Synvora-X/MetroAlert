# MEMORY.md — Estado de trabajo

> Memoria temporal del proyecto (máx. 100–150 líneas). Las reglas permanentes viven en `AGENTS.md` y el detalle técnico en `/docs`. No se registran secretos ni credenciales.

---

## Estado actual (al 2026-11-05)

- Proyecto en fase de **fundamentos** (orden de construcción §15): `backend/` es el esqueleto NestJS 12 (`src/main.ts` + `src/app.module.ts`, sin módulos de dominio); no hay frontend, ni base de datos conectada, ni ingesta.
- Documentación de fundamentos creada y revisada:
  - `docs/requerimientos.md` — RF-01…RF-16, HU-01…HU-06 (GWT), RNF-01…RNF-14, post-MVP y cuestiones abiertas CA-01…CA-05. Estado: creado.
  - `docs/arquitectura.md` — propuesta de referencia (monolito modular + Clean Architecture; módulos `alerts`, `transport`, `route-engine`, `geolocation`, `sharing`) y decisiones por ratificar en ADRs. Estado: creado.
  - `docs/tareas-de-investigacion.md` — plan de estudio con 35 tareas (INV-01…INV-35) en 6 asignaturas, prioridades P0–P3. Estado: creado.
  - `docs/funcionalidades-post-mvp.md` — hoja de ruta post-MVP ("MetroAlerta 2.0") desmenuzada paso a paso (PM-01…PM-25, versiones V2/V3/V4). Estado: creado. Nada de allí está implementado ni aprobado para entrar al MVP.
  - `docs/trazabilidad.md` — matriz F01–F05 → RF → HU → RNF → INV → PM y correspondencia INV ↔ PM. Estado: creado.
  - `docs/testing.md` — estrategia de pruebas por componente, estado real del CI y pendientes. Estado: creado.
- La tabla de documentos de `AGENTS.md` §13 quedó actualizada con seis documentos creados (requerimientos, arquitectura, tareas, funcionalidades-post-mvp, trazabilidad y testing).

## Tareas en progreso

- Ninguna en ejecución. Siguiente paso concreto: **Asignatura 1 (P0)** del plan de investigación — investigar y fichar fuentes de datos (INV-01…INV-09) para redactar `docs/fuentes-de-datos.md`, bloqueante del modelo de datos y del Route Engine.

## Backlog corto

1. INV-01…INV-09 — fuentes de datos (P0 bloqueante).
2. INV-10…INV-13 — validar cifras del problema y diferenciación (P0/P1).
3. Prueba de concepto: Dijkstra + red simulada de 10–20 nodos + exclusión de segmentos (cierre de Asignatura 3).
4. Primer módulo backend real: `alerts` (INV-20), siguiendo `docs/arquitectura.md` §4–§5.

## Decisiones recientes (con fecha)

- 2026-11-05 — Requerimientos y arquitectura se basan en la conversación "Plan de MetroAlerta" (ChatGPT, 2026-11-03) más `AGENTS.md`. La conversación es segunda fuente: cuando contradice al código o a `AGENTS.md`, mandan estos últimos.
- 2026-11-05 — La propuesta de arquitectura del documento queda explícita como **propuesta de referencia** hasta que existan ADRs; el código real (esqueleto NestJS) manda sobre todo lo escrito.
- 2026-11-05 — `docs/tareas-de-investigacion.md` se agregó a la tabla de documentos de `AGENTS.md` §13 y se creó este `MEMORY.md` en la raíz (aprobado por el desarrollador en sesión).
- 2026-11-05 — Se creó `docs/funcionalidades-post-mvp.md`: la sección "MetroAlerta 2.0" de la conversación queda desmenuzada paso a paso como hoja de ruta propuesta (V2 asistencia, V3 institucional, V4 investigación). No es compromiso: ninguna funcionalidad entra al MVP sin aprobación explícita; las exclusiones de `AGENTS.md` §2 se mantienen.
- 2026-11-05 — Se recuperaron los enlaces del chat desde el JSON del backend-api: la extracción `.md` no conservó los `href`, pero las URLs de los documentos eran correctas. Ahora se citan con la marca "verificar" por vigencia.

## ADRs pendientes (TODO → formalizar)

1. Estilo arquitectónico global (monolito modular vs alternativas).
2. Ingesta: Python vs n8n.
3. Alojamiento del backend y de la ingesta.
4. Proveedor de tiles para producción.
5. Modelo de datos definitivo (incl. PostGIS y granularidad contingencia→segmento).
6. Estructura del repositorio: monorepo vs separados.

Detalle y opciones en `docs/arquitectura.md` §10.

## Problemas / bloqueos abiertos

- Sin fuentes de datos validadas ni muestras inspeccionadas (riesgo principal).
- Cuestiones abiertas de `docs/requerimientos.md`: CA-01 (gestión manual de alertas), CA-02 (mecanismo de WhatsApp), CA-03/CA-05 (parámetros y umbrales por confirmar), CA-04 (granularidad contingencia→segmento).
- `docs/dominio.md`, `docs/modelo-de-datos.md`, `docs/api.md`, `docs/despliegue.md` y `docs/adr/` siguen vacíos o sin crear.
- `backend/MEMORY.md` está vacío y duplicaría este archivo: decidir si se elimina o si se usa solo para memoria específica del backend.

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
