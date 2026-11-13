# ADR-005 — Modelo de datos y PostGIS

| Campo | Valor |
|---|---|
| Estado | aceptado |
| Fecha | 2026-11-06 |
| Decisores | Desarrollador |
| Documentos relacionados | `AGENTS.md` §4 y §6, `docs/arquitectura.md` §7, `docs/modelo-de-datos.md` (por crear) |

## Contexto

El proyecto necesita representar estaciones, paraderos, líneas, conexiones y geometrías; calcular cercanía; dibujar recorridos; y, más adelante, construir un mapa de impacto. El motor de rutas necesita un grafo con pesos, no una base de datos de rutas.

## Decisión

1. **Habilitar PostGIS** en la base PostgreSQL/Supabase desde el inicio.
2. Representar nodos con `geography(Point, 4326)` y segmentos con `geometry(LineString, 4326)`; las geometrías pueden ser nulas si la fuente no las entrega.
3. Usar índices GiST para consultas espaciales.
4. **No usar pgRouting.** El grafo se construye en la aplicación y el cálculo de rutas vive en `route-engine`.
5. Modelar la granularidad de afectación **por conexión (arista) entre nodos consecutivos**. Un cierre de estación se traduce en las aristas incidentes; un corte parcial, en las aristas del tramo.
6. Guardar en cada segmento la fuente y la fecha de última actualización para poder marcarlo como desactualizado.

## Consecuencias

**Positivas:**
- Consultas de cercanía y geometrías sin trabajo manual.
- El mapa de impacto y las rutas accesibles tienen base técnica.
- El motor de rutas permanece independiente de la base de datos.

**Negativas:**
- El esquema y las migraciones son más complejos.
- La calidad depende de que las fuentes entreguen coordenadas y geometrías válidas.

**Mitigaciones:**
- Geometrías opcionales y datos marcados como `simulado` o `por verificar`.
- Pruebas de importación con fixtures antes de conectar fuentes reales.
- Validación de coordenadas y sistemas de referencia en la capa de ingesta.

## Alternativas consideradas

- **Sin PostGIS:** suficiente para un grafo lógico, pero insuficiente para cercanía, mapas de impacto y geometrías.
- **pgRouting:** añade dependencia y acopla el cálculo a la base; el MVP no lo necesita.
- **Granularidad por estación o por línea:** más simple, pero no representa cortes parciales ni mantenimientos por tramo.

## Referencias

- `AGENTS.md` §6 (reglas de dominio).
- `docs/arquitectura.md` §7 (Route Engine).
- Conversación "Plan de MetroAlert": representación de segmentos y PostGIS.
