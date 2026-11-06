# ADR-004 — Proveedor de tiles para producción

| Campo | Valor |
|---|---|
| Estado | propuesto |
| Fecha | 2026-11-06 |
| Decisores | Desarrollador (pendiente) |
| Documentos relacionados | `AGENTS.md` §4 y §7, `docs/arquitectura.md` §2 |

## Contexto

El mapa usa Leaflet + OpenStreetMap. El servidor público de tiles de OSM tiene una política de uso que limita el tráfico alto; no es una opción segura para producción sin evaluar sus condiciones.

## Opciones

- **OSM público:** aceptable para desarrollo y prototipo de bajo tráfico, con atribución y respeto de la política.
- **Tiles autoalojados:** control y privacidad, pero requieren infraestructura y operación.
- **Proveedor comercial:** SLA y capacidad, con coste y condiciones que deben revisarse.

## Criterios de decisión

1. Política de uso y atribución.
2. Límites de tráfico y coste.
3. Cobertura y calidad del mapa para el Valle de Aburrá.
4. Privacidad y registro de peticiones.
5. Facilidad de integración con Leaflet.

## Decisión

**Pendiente.** Para desarrollo se puede usar OSM público con atribución y tráfico bajo. Para producción se evaluará un proveedor o autoalojamiento después de medir el uso real; no se presenta OSM público como solución de producción.

## Consecuencias

- La UI debe mostrar siempre la atribución de OpenStreetMap.
- El proveedor no debe filtrarse a la lógica de dominio ni al Route Engine.
- Cambiar de proveedor no debe obligar a reescribir la UI.

## Referencias

- `AGENTS.md` §4 y §7.
- `docs/arquitectura.md` §2.
- Política de uso de OpenStreetMap (verificar antes de producción).
