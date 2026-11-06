# ADR-003 — Alojamiento del backend y la ingesta

| Campo | Valor |
|---|---|
| Estado | propuesto |
| Fecha | 2026-11-06 |
| Decisores | Desarrollador (pendiente) |
| Documentos relacionados | `AGENTS.md` §4, `docs/arquitectura.md` §10 |

## Contexto

El frontend se desplegará en Vercel. El backend NestJS y el proceso de ingesta necesitan un alojamiento que soporte variables de entorno, logs, despliegues reproducibles y, si se elige, Docker. La base de datos prevista es Supabase/PostgreSQL.

## Opciones

- **Railway:** citado en la conversación para la ingesta; despliegue gestionado y variables de entorno.
- **Render:** ejemplo de plataforma gestionada; evaluar regiones, cold starts y coste.
- **Otro proveedor gestionado:** cualquier opción que cumpla los criterios.
- **Autoalojado:** mayor control, pero añade operación y seguridad que el MVP no justifica.

## Criterios de decisión

1. Coste y límites del plan gratuito o inicial.
2. Región y latencia hacia Supabase y los usuarios del Valle de Aburrá.
3. Soporte de variables de entorno y secretos.
4. Logs, salud y facilidad de rollback.
5. Soporte de Docker o buildpacks para NestJS.
6. Independencia: no debe obligar a cambiar el dominio.

## Decisión

**Pendiente.** Vercel queda para el frontend; el backend y la ingesta se deciden después de validar el MVP local y el CI. La arquitectura no depende de un proveedor concreto.

## Consecuencias

- El despliegue no bloquea el desarrollo local ni las pruebas.
- Cualquier proveedor debe permitir separar entornos y secretos.
- Si se usa Docker en desarrollo, el mismo artefacto debería poder desplegarse.

## Referencias

- `AGENTS.md` §4 (decisiones abiertas).
- `docs/arquitectura.md` §10 (ADR-003).
- `docs/despliegue.md` (por crear).
