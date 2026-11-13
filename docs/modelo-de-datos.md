# Modelo de datos — MetroAlert

| Campo | Valor |
|---|---|
| Estado | creado |
| Fecha | 2026-11-06 |
| Fuentes | `docs/dominio.md`, `docs/requerimientos.md`, `docs/arquitectura.md`, `docs/adr/0005-modelo-datos-postgis.md`, `docs/fuentes-de-datos.md` |
| Propósito | Traducir el dominio a un modelo persistible, consultable y testeable. |
| Autoridad | Es una **propuesta de diseño**; no hay migraciones ni base de datos conectada. Si el código contradice este documento, manda el código. |

**Para agentes:**
- No crees migraciones a partir de este documento sin aprobación explícita (`AGENTS.md` §9).
- Toda tabla debe tener una razón de dominio; no agregues campos "por si acaso".
- Los datos reales no están validados: las muestras y fuentes siguen `por verificar`.

---

## 1. Decisiones que gobiernan el modelo

- **PostGIS habilitado** para puntos y geometrías; **sin pgRouting** (ADR-005).
- **Granularidad por conexión:** un segmento es la arista entre dos nodos consecutivos.
- **Contingencia = alerta verificada**; no se crea una tabla `contingencies` separada.
- **Mantenimiento** es una entidad separada; puede vincularse a una alerta cuando entra en curso.
- **Sin usuarios de pasajeros** en el MVP; la administración se protege con autenticación de administrador.
- **Sin datos de producción** en desarrollo ni en pruebas; los fixtures son simulados.

## 2. Modelo conceptual

```mermaid
erDiagram
    SOURCES ||--o{ ALERTS : proves
    SOURCES ||--o{ MAINTENANCE : proves
    SOURCES ||--o{ ESTIMATION_PARAMETERS : defines
    LINES ||--o{ LINE_NODES : contains
    NODES ||--o{ LINE_NODES : belongs
    NODES ||--o{ SEGMENTS : from
    NODES ||--o{ SEGMENTS : to
    ALERTS ||--o{ ALERT_LINES : affects
    LINES ||--o{ ALERT_LINES : affected
    ALERTS ||--o{ ALERT_NODES : affects
    NODES ||--o{ ALERT_NODES : affected
    ALERTS ||--o{ ALERT_SEGMENTS : affects
    SEGMENTS ||--o{ ALERT_SEGMENTS : affected
    MAINTENANCE ||--o{ MAINTENANCE_SEGMENTS : affects
    SEGMENTS ||--o{ MAINTENANCE_SEGMENTS : affected
    MAINTENANCE ||--o| ALERTS : activates
```

## 3. Modelo lógico

### 3.1 `sources`

Procedencia y condiciones de cada dato.

| Columna | Tipo | Restricciones |
|---|---|---|
| `id` | `uuid` | PK, `gen_random_uuid()` |
| `name` | `text` | not null |
| `url` | `text` | nullable; URL exacta si existe |
| `origin` | `text` | check: `oficial`, `abierto`, `estimado`, `simulado` |
| `license` | `text` | nullable |
| `terms` | `text` | nullable |
| `last_checked_at` | `timestamptz` | nullable |
| `status` | `text` | check: `verificada`, `por_verificar`, `descartada` |
| `created_at` / `updated_at` | `timestamptz` | not null, default `now()` |

### 3.2 `lines`

| Columna | Tipo | Restricciones |
|---|---|---|
| `id` | `uuid` | PK |
| `code` | `text` | unique, not null |
| `name` | `text` | not null |
| `mode` | `text` | check: `metro`, `cable`, `tram`, `brt`, `bus` |
| `color` | `text` | nullable; solo presentación |
| `created_at` / `updated_at` | `timestamptz` | not null |

### 3.3 `nodes`

Estaciones y paraderos.

| Columna | Tipo | Restricciones |
|---|---|---|
| `id` | `uuid` | PK |
| `name` | `text` | not null |
| `type` | `text` | check: `station`, `stop` |
| `location` | `geography(Point, 4326)` | nullable; si falta, `location_status = 'missing'` |
| `location_status` | `text` | check: `verified`, `approximate`, `missing` |
| `source_id` | `uuid` | FK `sources(id)` |
| `last_updated` | `timestamptz` | not null |
| `created_at` / `updated_at` | `timestamptz` | not null |

Índice: `GIST (location)`.

### 3.4 `line_nodes`

Relación muchos-a-muchos entre líneas y nodos, con orden.

| Columna | Tipo | Restricciones |
|---|---|---|
| `line_id` | `uuid` | FK `lines(id)`, PK compuesta |
| `node_id` | `uuid` | FK `nodes(id)`, PK compuesta |
| `sequence` | `integer` | not null, >= 0 |

### 3.5 `segments`

Aristas del grafo.

| Columna | Tipo | Restricciones |
|---|---|---|
| `id` | `uuid` | PK |
| `from_node_id` | `uuid` | FK `nodes(id)` |
| `to_node_id` | `uuid` | FK `nodes(id)` |
| `mode` | `text` | check: `metro`, `cable`, `tram`, `brt`, `bus`, `walk` |
| `estimated_minutes` | `integer` | not null, > 0 |
| `distance_meters` | `numeric` | nullable, >= 0 |
| `geometry` | `geometry(LineString, 4326)` | nullable |
| `is_bidirectional` | `boolean` | not null, default `false` |
| `source_id` | `uuid` | FK `sources(id)` |
| `last_updated` | `timestamptz` | not null |
| `created_at` / `updated_at` | `timestamptz` | not null |

Restricciones: `from_node_id <> to_node_id`; índice `GIST (geometry)`; índice compuesto `(from_node_id, to_node_id)`.

### 3.6 `alerts`

Una alerta es una comunicación; una contingencia es una alerta verificada.

| Columna | Tipo | Restricciones |
|---|---|---|
| `id` | `uuid` | PK |
| `type` | `text` | check: `interrupcion_total`, `interrupcion_parcial`, `cierre_estacion`, `mantenimiento`, `retraso`, `afectacion_externa` |
| `state` | `text` | check: `activa`, `resuelta`, `vigencia_indeterminada` |
| `verification` | `text` | check: `no_verificada`, `verificada` |
| `severity` | `text` | nullable; check: `baja`, `media`, `alta`, `critica` |
| `title` | `text` | nullable |
| `description` | `text` | not null |
| `published_at` | `timestamptz` | nullable |
| `last_updated` | `timestamptz` | not null |
| `source_id` | `uuid` | FK `sources(id)` |
| `created_at` / `updated_at` | `timestamptz` | not null |

> No existe tabla `contingencies`: se consulta como `alerts` con `verification = 'verificada'` y estado `activa` o `vigencia_indeterminada`.

### 3.7 Relaciones de afectación

| Tabla | Columnas | Regla |
|---|---|---|
| `alert_lines` | `alert_id`, `line_id` | Una alerta puede afectar varias líneas |
| `alert_nodes` | `alert_id`, `node_id` | Cierre o afectación de estación/paradero |
| `alert_segments` | `alert_id`, `segment_id` | Afectación de una conexión concreta |
| `maintenance_segments` | `maintenance_id`, `segment_id` | Tramo afectado por un mantenimiento |

Todas usan PK compuesta y `ON DELETE CASCADE` desde la entidad principal.

### 3.8 `maintenance`

| Columna | Tipo | Restricciones |
|---|---|---|
| `id` | `uuid` | PK |
| `type` | `text` | not null |
| `description` | `text` | not null |
| `starts_at` | `timestamptz` | not null |
| `ends_at` | `timestamptz` | nullable |
| `status` | `text` | check: `programado`, `en_curso`, `finalizado`, `cancelado` |
| `source_id` | `uuid` | FK `sources(id)` |
| `alert_id` | `uuid` | nullable, FK `alerts(id)`; se vincula al activarse |
| `created_at` / `updated_at` | `timestamptz` | not null |

### 3.9 `estimation_parameters`

| Columna | Tipo | Restricciones |
|---|---|---|
| `id` | `uuid` | PK |
| `key` | `text` | unique, not null; p. ej. `walk_speed_kmh` |
| `value` | `numeric` | not null, > 0 |
| `unit` | `text` | not null |
| `source_id` | `uuid` | FK `sources(id)` |
| `status` | `text` | check: `estimado`, `validado` |
| `updated_at` | `timestamptz` | not null |

> Los parámetros también pueden leerse desde `config/` o variables de entorno; la tabla permite administrarlos y auditar su origen.

## 4. Modelo físico y Supabase

- Extensiones: `postgis`; `pgcrypto` si se usa `gen_random_uuid()` en la versión de PostgreSQL elegida.
- Esquema `public` para el MVP; separar `auth`/`storage` de Supabase.
- Migraciones SQL versionadas; nunca cambios destructivos sin respaldo y aprobación.
- RLS:
  - Lectura pública de `lines`, `nodes`, `segments`, `alerts` verificadas y `maintenance` publicados.
  - Escritura solo con `service_role` o rol administrativo.
  - No exponer `sources.terms` ni datos internos innecesarios.
- `updated_at` se actualiza con trigger o en la capa de aplicación; elegir una sola forma.
- Las geometrías nulas se permiten, pero el registro se marca como incompleto.

## 5. Reglas de integridad

1. Un segmento no puede unir un nodo consigo mismo.
2. Un segmento pertenece a un modo válido y tiene tiempo positivo.
3. Una alerta tiene un solo estado y una sola verificación.
4. Una contingencia visible debe estar verificada.
5. Un mantenimiento no puede estar `en_curso` sin fecha de inicio.
6. Un mantenimiento `finalizado` no debe tener una alerta vinculada activa.
7. Los IDs de fuente deben existir; no se permiten fuentes huérfanas.
8. Las relaciones de afectación se eliminan en cascada con su entidad principal.

## 6. Datos simulados y pruebas

- Fixtures pequeños en `backend/test/fixtures/` (por crear), con escenarios de cierre, corte parcial y mantenimiento.
- Cada fixture declara `origin: simulado` y el escenario que representa.
- No se copian datos reales a los fixtures sin autorización.
- Las pruebas de integración usan una base local o contenedor, nunca producción.

## 7. Fuera del MVP

- Persistencia de rutas calculadas y consultas para analítica (PM-18).
- Usuarios de pasajeros y preferencias (PM-03).
- Historial de eventos de contingencia (PM-01) más allá de `created_at`/`updated_at`.
- Accesibilidad por nodo (PM-05).
- Reportes ciudadanos (PM-10).

## 8. Decisiones abiertas

- Umbrales de severidad.
- Valores de los parámetros de estimación (CA-03).
- Estrategia de migraciones y entorno de pruebas.
- Necesidad de `pgcrypto` según la versión de PostgreSQL.
- Índices adicionales después de medir consultas reales.

## 9. Cómo entra en acción

- **Migraciones:** crean las tablas y políticas descritas.
- **`transport`:** repositorios de líneas, nodos y segmentos.
- **`alerts`:** repositorios de alertas, relaciones de afectación y mantenimiento.
- **`route-engine`:** construye el grafo desde `nodes` y `segments`; excluye `alert_segments` confirmados.
- **`sharing`:** toma los datos de la alternativa y la contingencia.
- **Pruebas:** integridad, RLS, importación y consultas espaciales.

## 10. Cuándo actualizar este documento

- Al cambiar una entidad, una relación o una restricción.
- Al cerrar una decisión de modelado.
- Al crear una migración.
- Si una fuente real obliga a ajustar tipos o granularidad.
