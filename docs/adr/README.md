# Decisiones arquitectónicas (ADR)

| ADR | Título | Estado | Archivo |
|---|---|---|---|
| ADR-001 | Estilo arquitectónico global | aceptado | [`0001-estilo-arquitectonico.md`](0001-estilo-arquitectonico.md) |
| ADR-002 | Tecnología de ingesta | propuesto | [`0002-tecnologia-ingesta.md`](0002-tecnologia-ingesta.md) |
| ADR-003 | Alojamiento del backend y la ingesta | propuesto | [`0003-alojamiento.md`](0003-alojamiento.md) |
| ADR-004 | Proveedor de tiles para producción | propuesto | [`0004-tiles-produccion.md`](0004-tiles-produccion.md) |
| ADR-005 | Modelo de datos y PostGIS | aceptado | [`0005-modelo-datos-postgis.md`](0005-modelo-datos-postgis.md) |
| ADR-006 | Estructura del repositorio | aceptado | [`0006-estructura-repositorio.md`](0006-estructura-repositorio.md) |

## Reglas

- Un ADR documenta **una** decisión. Si cambia, se crea otro que reemplaza al anterior; no se reescribe la historia.
- Estados válidos: `propuesto`, `aceptado`, `rechazado`, `reemplazado`.
- Un ADR solo pasa a `aceptado` con aprobación explícita del desarrollador.
- Las decisiones abiertas se mantienen en `propuesto` y se enlazan desde `docs/trazabilidad.md`.
- La fuente de verdad del estado es este índice y la cabecera de cada archivo.

## Plantilla

```markdown
# ADR-00X — Título

| Campo | Valor |
|---|---|
| Estado | propuesto / aceptado / rechazado / reemplazado |
| Fecha | AAAA-MM-DD |
| Decisores | Desarrollador |
| Documentos relacionados | ... |

## Contexto
Qué problema o fuerza obliga a decidir.

## Decisión
Qué se decide, en una frase verificable.

## Consecuencias
Positivas, negativas y mitigaciones.

## Alternativas consideradas
Opciones descartadas y por qué.

## Referencias
Documentos, fuentes o discusiones.
```
