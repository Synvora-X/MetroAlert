# Trazabilidad — MetroAlert

| Campo | Valor |
|---|---|
| Estado | creado |
| Fecha | 2026-11-05 |
| Fuentes | `AGENTS.md` (§2, §6, §13) + `docs/requerimientos.md` + `docs/arquitectura.md` + `docs/tareas-de-investigacion.md` + `docs/funcionalidades-post-mvp.md` |
| Propósito | Mapa único de IDs y matriz de trazabilidad F01–F05 → RF → HU → RNF → INV → PM. No duplica el contenido de los documentos: solo enlaza y detecta huecos. |
| Autoridad | La leyenda de familias de IDs vive en `AGENTS.md` §13; este documento la aplica y mantiene la matriz. |

**Para agentes:**
- Usa este archivo para responder "¿qué requerimiento cubre esta funcionalidad?", "¿qué tarea investiga este dato?" y "¿qué decisión está abierta?".
- No asumas que una fila con correspondencia está implementada: el estado real está en el código y en `MEMORY.md`.
- Si cambias un ID, actualiza esta matriz en el mismo cambio.

---

## 1. Cómo se usa

- La jerarquía de fuentes y el formato de referencias están en el bloque inicial y en la sección 13 de `AGENTS.md`.
- Esta matriz es de **cobertura**, no de estado: `propuesta` / `decisión` / `implementado` se indican en cada documento.
- Las correspondencias son aproximadas cuando un ID cubre varias filas; se marca "—" cuando no hay relación directa.

## 2. Familias de IDs

Resumen operativo; la leyenda autoritativa está en `AGENTS.md` §13.

| Familia | Significado | Documento dueño |
|---|---|---|
| `F0x` | Funcionalidades del MVP | `AGENTS.md` §2 |
| `RF-xx` | Requerimientos funcionales | `docs/requerimientos.md` |
| `RNF-xx` | Requerimientos no funcionales | `docs/requerimientos.md` |
| `HU-xx` | Historias de usuario | `docs/requerimientos.md` |
| `CA-xx` | Cuestiones abiertas del MVP | `docs/requerimientos.md` |
| `INV-xx` | Tareas de investigación | `docs/tareas-de-investigacion.md` |
| `PM-xx` | Funcionalidades post-MVP | `docs/funcionalidades-post-mvp.md` |
| `PC-xx` | Cuestiones abiertas post-MVP | `docs/funcionalidades-post-mvp.md` |
| `ADR-xxx` | Decisiones arquitectónicas | `docs/adr/` |

## 3. Matriz F01–F05 → RF → HU → RNF → INV → PM

| Funcionalidad | RF | HU | RNF | INV | PM |
|---|---|---|---|---|---|
| **F01 — Ingesta y gestión de alertas** | RF-01, RF-02, RF-03, RF-04 | HU-01 | RNF-05, RNF-09, RNF-11, RNF-13 | INV-01, INV-05, INV-07, INV-08, INV-09 | PM-01, PM-02, PM-10 |
| **F02 — Mapa** | RF-05, RF-06, RF-07, RF-08 | HU-01, HU-06 | RNF-01, RNF-05, RNF-10, RNF-12, RNF-13 | INV-01, INV-02, INV-06 | PM-14, PM-15, PM-17 |
| **F03 — Rutas alternativas** | RF-09, RF-10, RF-11, RF-12 | HU-02, HU-03, HU-04 | RNF-03, RNF-06, RNF-08, RNF-11 | INV-14, INV-15, INV-16, INV-17, INV-18, INV-19 | PM-04, PM-05, PM-07, PM-08, PM-19 |
| **F04 — Tiempo estimado** | RF-13, RF-14 | HU-04, HU-06 | RNF-04, RNF-05, RNF-11 | INV-17, INV-19 | PM-06, PM-08, PM-12 |
| **F05 — Compartir por WhatsApp** | RF-15 | HU-05 | RNF-04, RNF-05, RNF-06 | INV-28 | PM-13 |
| **Transversal** | RF-16 | HU-06 | RNF-05, RNF-12 | INV-29 | PM-14, PM-15 |

Lectura de la matriz: una celda con varios IDs significa que la funcionalidad se apoya en todos ellos; no implica que estén implementados.

## 4. Correspondencia INV ↔ PM

| Bloque INV | Tema | PM relacionados |
|---|---|---|
| INV-01…INV-09 | Fuentes de datos y condiciones de uso | PM-01, PM-02, PM-09, PM-19 (datos de base) |
| INV-10…INV-13 | Problema, usuarios y competencia | PM-01, PM-02, PM-06 (contexto y precedentes) |
| INV-14…INV-19 | Algoritmos, Route Engine y geometrías | PM-04, PM-06, PM-07, PM-08, PM-19 |
| INV-20…INV-26 | Fundamentos de implementación | PM-20, PM-21, PM-22 (infraestructura, API y calidad) |
| INV-27…INV-30 | Producto y experiencia de usuario | PM-11, PM-13, PM-14, PM-15, PM-16 |
| INV-31 | Mantenimientos y simulación | PM-09, PM-19, PM-25 |
| INV-32 | Notificaciones y trayectos favoritos | PM-03 |
| INV-33 | Accesibilidad | PM-05 |
| INV-34 | Analítica institucional | PM-16, PM-17, PM-18, PM-23, PM-24 |
| INV-35 | Reportes ciudadanos | PM-10 |

## 5. Decisiones abiertas (índice)

Índice de navegación; el detalle y el estado completo viven en cada documento dueño.

| ID | Tema | Documento dueño | Estado |
|---|---|---|---|
| CA-01 | Gestión manual de alertas (panel protegido vs. base de datos/simulados) | `docs/requerimientos.md` | **Resuelto:** panel mínimo protegido |
| CA-02 | Mecanismo de envío para compartir por WhatsApp | `docs/requerimientos.md` | **Resuelto:** `wa.me` + Web Share |
| CA-03 | Valores de los parámetros de estimación | `docs/requerimientos.md` | TODO(confirmar) |
| CA-04 | Granularidad contingencia → segmentos afectados | `docs/requerimientos.md` | **Resuelto:** por conexión (ADR-005) |
| CA-05 | Umbrales de rendimiento | `docs/requerimientos.md` | TODO(confirmar) |
| PC-01 | Alcance exacto de V2 tras el MVP | `docs/funcionalidades-post-mvp.md` | Por decidir |
| PC-02 | Modelo de clientes para pasajeros (anónimo vs. cuenta) | `docs/funcionalidades-post-mvp.md` | Por decidir |
| PC-03 | Fuentes de accesibilidad y de mantenimientos | `docs/funcionalidades-post-mvp.md` | INV-31 e INV-33 |
| PC-04 | Contrato de la API institucional | `docs/funcionalidades-post-mvp.md` | Por definir en `docs/api.md` |
| PC-05 | Posición de V4 frente a las exclusiones de `AGENTS.md` §2 | `docs/funcionalidades-post-mvp.md` | Por decidir |
| PC-06 | Modelo de estados de los mantenimientos programados | `docs/funcionalidades-post-mvp.md` | **Resuelto:** entidad `Maintenance` separada |
| ADR-001 | Estilo arquitectónico global | `docs/adr/0001-estilo-arquitectonico.md` | Aceptado |
| ADR-002 | Tecnología de ingesta (Python vs. n8n) | `docs/adr/0002-tecnologia-ingesta.md` | Propuesto |
| ADR-003 | Alojamiento del backend y de la ingesta | `docs/adr/0003-alojamiento.md` | Propuesto |
| ADR-004 | Proveedor de tiles para producción | `docs/adr/0004-tiles-produccion.md` | Propuesto |
| ADR-005 | Modelo de datos y PostGIS | `docs/adr/0005-modelo-datos-postgis.md` | Aceptado |
| ADR-006 | Estructura del repositorio (monorepo) | `docs/adr/0006-estructura-repositorio.md` | Aceptado |

## 6. Huecos y notas de cobertura

- **F01**: CA-01 resuelto (panel mínimo protegido); el contrato de administración está en `docs/api.md`.
- **RNF-14**: sin umbrales medidos todavía (CA-05).
- **PM-01 y PM-02**: dependen de la fuente de alertas (INV-07) y de la taxonomía de `docs/dominio.md`.
- **PM-09 y PM-19**: dependen de la fuente de mantenimientos (INV-31); PC-06 resuelto.
- **PM-05**: depende de datos de accesibilidad (INV-33), que probablemente no existen aún.
- **PM-20**: el contrato propuesto está en `docs/api.md`; los endpoints de la conversación son ejemplos, no implementaciones.
- **PM-21 y PM-22**: dependen de `docs/fuentes-de-datos.md` y de ADR-002.
- **PM-24**: toca exclusiones de `AGENTS.md` §2; requiere aprobación y ADR (PC-05).
- No quedan documentos pendientes en la tabla de `AGENTS.md` §13. La completitud de cada documento se sigue en la sección “Documentos creados con información pendiente”.

## 7. Mantenimiento

- Al crear o cambiar un ID: actualizar la fila correspondiente en esta matriz y en el documento dueño.
- Al cerrar una decisión: cambiar su estado aquí y registrar el ADR o la aprobación.
- Al aprobar una funcionalidad post-MVP: moverla al alcance del MVP solo si se actualiza `AGENTS.md` §2 y se documenta el ADR.

Última revisión: 2026-11-06.
