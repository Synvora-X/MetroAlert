# ADR-002 — Tecnología de ingesta

| Campo | Valor |
|---|---|
| Estado | propuesto |
| Fecha | 2026-11-06 |
| Decisores | Desarrollador (pendiente) |
| Documentos relacionados | `AGENTS.md` §4 y §5, `docs/arquitectura.md` §8 |

## Contexto

La ingesta debe consultar fuentes externas, normalizar, validar, evitar duplicados, detectar cambios de estado y registrar errores. Es un proceso separado del backend y no debe acoplar el dominio a un proveedor. El MVP puede arrancar con registro manual/simulado, pero la arquitectura debe permitir incorporar fuentes reales.

## Opciones

### Python

- Ventajas: tipado, pruebas, ecosistema de datos, control explícito de adaptadores.
- Costes: un lenguaje y un runtime adicionales; despliegue y CI propios.
- Encaja si el equipo mantiene pruebas y separa cada fuente en un adaptador.

### n8n

- Ventajas: prototipado visual rápido, conectores listos, menos código inicial.
- Costes: un servicio adicional que operar; menor testabilidad y versionado; lógica repartida en nodos.
- Encaja si se prioriza velocidad de integración sobre control fino.

## Criterios de decisión

1. Facilidad para probar y versionar adaptadores por fuente.
2. Coste de despliegue y operación.
3. Capacidad de manejar errores, reintentos y cambios de estado.
4. Comodidad real del equipo con la herramienta.
5. Independencia del dominio y de la UI.

## Decisión

**Pendiente.** Se tomará después de INV-01…INV-07, cuando se conozcan las fuentes y su formato. Hasta entonces, la ingesta del MVP es manual/simulada y el proceso queda desacoplado.

## Consecuencias

- La decisión no bloquea el MVP porque el registro manual está permitido.
- Cualquier elección debe respetar `AGENTS.md` §5: cambiar una fuente no obliga a tocar el dominio ni la UI.
- Si se elige n8n, la lógica crítica debe quedar versionada y documentada; si se elige Python, cada adaptador debe tener pruebas con fixtures.

## Referencias

- `AGENTS.md` §4 y §5.
- `docs/arquitectura.md` §8.
- `docs/tareas-de-investigacion.md` (INV-01…INV-09).
