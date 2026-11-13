# ADR-001 — Estilo arquitectónico global

| Campo | Valor |
|---|---|
| Estado | aceptado |
| Fecha | 2026-11-06 |
| Decisores | Desarrollador |
| Documentos relacionados | `AGENTS.md` §5, `docs/arquitectura.md` §1 |

## Contexto

MetroAlert es una PWA para el Valle de Aburrá con un backend NestJS, un frontend Next.js y un proceso de ingesta separado. El equipo es pequeño y el MVP debe validar el flujo completo antes de escalar. El cálculo de rutas y la gestión de contingencias son el núcleo del dominio, pero no justifican una arquitectura distribuida.

## Decisión

Adoptar un **monolito modular con Clean Architecture y DDD ligero**:

- Un solo backend desplegable (NestJS), dividido en módulos con límites explícitos: `alerts`, `transport`, `route-engine`, `geolocation`, `sharing`.
- Capas `domain`, `application`, `infrastructure` y `presentation` dentro de cada módulo, con dependencias hacia adentro.
- El dominio no depende de Supabase, n8n, Leaflet ni de proveedores concretos.
- La ingesta es un proceso separado, no un microservicio ni una arquitectura orientada a eventos.

## Consecuencias

**Positivas:**
- Un solo despliegue y una sola base de código para el MVP.
- Reglas de negocio testeables sin infraestructura.
- Límites claros para incorporar módulos futuros sin reescribir el backend.

**Negativas:**
- Exige disciplina para no mezclar capas ni duplicar lógica entre módulos.
- Un monolito mal modularizado se vuelve difícil de mantener.

**Mitigaciones:**
- Revisión de límites en cada módulo nuevo.
- Pruebas de dominio sin frameworks.
- ADR para cualquier cambio de estilo arquitectónico.

## Alternativas consideradas

- **Microservicios:** descartados por alcance, coste operativo y tamaño del equipo.
- **Next.js solo con funciones serverless:** no resuelve el motor de rutas ni la ingesta con la claridad necesaria.
- **Backend único sin capas:** más rápido al inicio, pero acopla el dominio a la base de datos y a los controladores.

## Referencias

- `AGENTS.md` §5 (responsabilidades y límites).
- `docs/arquitectura.md` §1 y §5.
