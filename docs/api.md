# API — MetroAlert

| Campo | Valor |
|---|---|
| Estado | creado |
| Fecha | 2026-11-06 |
| Fuentes | `docs/requerimientos.md`, `docs/arquitectura.md`, `docs/dominio.md`, `docs/modelo-de-datos.md` |
| Propósito | Definir el contrato REST entre el frontend, el backend y futuras integraciones. |
| Autoridad | Es una **propuesta de contrato**; no hay endpoints implementados. Si el código contradice este documento, manda el código. |

**Para agentes:**
- No inventes endpoints ni campos: este documento es la propuesta vigente hasta que exista código.
- Toda respuesta debe incluir fuente, frescura y nivel de confianza cuando aplique.
- Los ejemplos usan datos simulados y están marcados como tales.

---

## 1. Convenciones

- Base: `/api/v1`.
- Formato: JSON UTF-8.
- Fechas: ISO 8601 con zona (`2026-11-06T12:00:00-05:00`).
- IDs: UUID.
- Recursos en plural, sustantivos en inglés para rutas y campos; documentación en español.
- Errores controlados; nunca se exponen stacks, SQL ni claves.
- Paginación: `limit` (default 20, max 100) y `offset`.
- Versionado: cambios incompatibles pasan a `/api/v2`.

## 2. Autenticación y permisos

| Área | Acceso |
|---|---|
| Alertas verificadas, red, rutas y compartir | Público, sin cuenta de pasajero |
| Administración de alertas | Requiere token de administrador (`Authorization: Bearer <token>`) |
| API institucional | Post-MVP; requiere acuerdo, autenticación y límites |

El MVP no crea cuentas de pasajeros. La administración usa un rol protegido; la `service_role` de Supabase nunca llega al frontend.

## 3. Formato de error

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "El destino está fuera del área de cobertura.",
    "details": [
      { "field": "destination", "reason": "OUT_OF_BOUNDS" }
    ]
  }
}
```

Códigos HTTP: `400` entrada inválida, `401` no autenticado, `403` sin permiso, `404` no encontrado, `409` conflicto de estado, `422` regla de negocio incumplida, `500` error interno.

## 4. Metadatos de datos

Toda respuesta que incluya datos de red, alertas o estimaciones puede incluir:

```json
{
  "data_origin": "oficial | abierto | estimado | simulado",
  "data_updated_at": "2026-11-06T12:00:00-05:00",
  "confidence": "HIGH | MEDIUM | LOW",
  "method": "SCHEDULED | ESTIMATED"
}
```

## 5. Endpoints del MVP

### 5.1 Salud

`GET /health`

```json
{
  "status": "ok",
  "version": "0.1.0",
  "time": "2026-11-06T12:00:00-05:00"
}
```

### 5.2 Alertas y contingencias

`GET /alerts`

Parámetros: `state` (`activa`, `resuelta`, `vigencia_indeterminada`), `line_id`, `node_id`, `verified` (`true`/`false`, solo admin), `limit`, `offset`.

Respuesta pública: solo alertas verificadas. El administrador puede consultar no verificadas con su token.

```json
{
  "items": [
    {
      "id": "11111111-1111-1111-1111-111111111111",
      "type": "interrupcion_parcial",
      "state": "activa",
      "verification": "verificada",
      "severity": "alta",
      "title": "Interrupción parcial en el tramo A",
      "description": "Ejemplo simulado.",
      "published_at": "2026-11-06T11:40:00-05:00",
      "last_updated": "2026-11-06T12:00:00-05:00",
      "data_origin": "simulado",
      "affected": {
        "line_ids": ["..."],
        "node_ids": ["..."],
        "segment_ids": ["..."]
      }
    }
  ],
  "pagination": { "limit": 20, "offset": 0, "total": 1 }
}
```

`GET /alerts/:id`

Devuelve el detalle y las relaciones de afectación. `404` si no existe o no es visible.

### 5.3 Administración de alertas

`POST /admin/alerts`

Crea una alerta. Requiere token de administrador.

```json
{
  "type": "cierre_estacion",
  "description": "Cierre temporal simulado.",
  "published_at": "2026-11-06T11:40:00-05:00",
  "source_id": "22222222-2222-2222-2222-222222222222",
  "affected": {
    "line_ids": ["..."],
    "node_ids": ["..."],
    "segment_ids": ["..."]
  }
}
```

`PATCH /admin/alerts/:id`

Actualiza estado, verificación, severidad, descripción o relaciones de afectación. Las transiciones inválidas responden `409` o `422`.

```json
{
  "state": "resuelta",
  "verification": "verificada",
  "severity": "media"
}
```

### 5.4 Red de transporte

`GET /transport/network`

Devuelve líneas, nodos y segmentos para el mapa. Soporta `line_id` y `bbox` (`minLng,minLat,maxLng,maxLat`).

```json
{
  "lines": [{ "id": "...", "code": "A", "name": "Línea A", "mode": "metro", "color": "#..." }],
  "nodes": [{ "id": "...", "name": "Estación A", "type": "station", "lat": 6.2, "lng": -75.5 }],
  "segments": [{ "id": "...", "from_node_id": "...", "to_node_id": "...", "mode": "metro", "estimated_minutes": 3 }],
  "data_origin": "simulado",
  "data_updated_at": "2026-11-06T12:00:00-05:00"
}
```

`GET /transport/nodes/nearby?lat=6.2&lng=-75.5&radius=800`

Devuelve nodos cercanos; `radius` en metros, máximo a definir. Usa PostGIS.

### 5.5 Alternativas de ruta

`POST /routes/alternatives`

```json
{
  "origin": { "lat": 6.2, "lng": -75.5 },
  "destination": { "lat": 6.25, "lng": -75.56 },
  "preferences": {
    "profile": "fastest | least_walking | fewest_transfers",
    "max_alternatives": 3
  }
}
```

Respuesta:

```json
{
  "request_id": "...",
  "generated_at": "2026-11-06T12:00:00-05:00",
  "alternatives": [
    {
      "id": "alt-1",
      "total_minutes": 42,
      "walking_minutes": 8,
      "waiting_minutes": 5,
      "transfers": 1,
      "segments": [
        { "from": "Estación A", "to": "Estación B", "mode": "metro", "minutes": 12, "source": "simulado" }
      ],
      "method": "ESTIMATED",
      "confidence": "MEDIUM",
      "data_updated_at": "2026-11-06T12:00:00-05:00"
    }
  ],
  "metadata": {
    "max_alternatives": 3,
    "affected_contingencies": ["..."],
    "data_origin": "simulado"
  }
}
```

Sin datos suficientes:

```json
{
  "request_id": "...",
  "alternatives": [],
  "reason": "NO_DATA",
  "message": "No hay alternativas con los datos disponibles."
}
```

Reglas: máximo 3; nunca se usan segmentos afectados por contingencias activas confirmadas; no se inventan rutas; el tiempo es una aproximación.

### 5.6 Compartir por WhatsApp

`POST /share/whatsapp`

```json
{
  "alternative": { "...": "objeto de alternativa" },
  "contingency_id": "..."
}
```

Respuesta:

```json
{
  "text": "MetroAlert: ...",
  "url": "https://wa.me/?text=...",
  "data_origin": "simulado"
}
```

El backend construye el mensaje y devuelve el enlace; **no envía** por API oficial. El frontend puede usar Web Share API y `wa.me` como respaldo.

## 6. Validación y errores de negocio

- Coordenadas fuera del área de cobertura: `422` con `OUT_OF_BOUNDS`.
- Origen o destino inválido: `400` con detalle por campo.
- Sin alternativas: `200` con `alternatives: []` y `reason: "NO_DATA"`.
- Alerta no verificada consultada públicamente: no se lista; `404` en detalle.
- Transición de estado inválida: `409`/`422`.

## 7. Fuera del MVP

- Endpoints de mantenimientos (PM-09) y notificaciones (PM-03).
- API institucional con claves y límites (PM-20).
- Historial analítico (PM-18).
- Reportes ciudadanos (PM-10).
- OpenAPI generado automáticamente: evaluar cuando exista implementación.

## 8. Decisiones abiertas

- Límite de peticiones y caché.
- Formato exacto de `bbox` y paginación por cursor.
- Autenticación administrativa concreta (Supabase Auth vs. token de servicio).
- Contrato de la API institucional (PC-04).
- Estrategia de versionado y compatibilidad.

## 9. Cómo entra en acción

- **`alerts`:** controladores de consulta y administración.
- **`transport`:** red y nodos cercanos.
- **`route-engine`:** cálculo de alternativas.
- **`sharing`:** mensaje y enlace.
- **Frontend:** consume el contrato mediante `lib/api`; no consulta Supabase ni calcula rutas. Los tipos compartidos irán en `packages/contracts` cuando se migre al monorepo.
- **Pruebas:** Supertest para contratos y errores; fixtures simulados.

## 10. Cuándo actualizar este documento

- Al implementar o cambiar un endpoint.
- Al cerrar una decisión de autenticación, paginación o versión.
- Al crear el contrato institucional.
- Si el código contradice este documento, se actualiza el documento y se señala el cambio.
