<!-- BEGIN:nextjs-agent-rules -->

## This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

# AGENTS.md — Frontend MetroAlert

| Campo | Valor |
|---|---|
| Estado | creado |
| Fecha | 2026-11-06 |
| Alcance | Solo `frontend/` |
| Precedencia | `../AGENTS.md` > este archivo > `../MEMORY.md` > `frontend/MEMORY.md` |

> Este archivo complementa `../AGENTS.md`. No repite el alcance del producto, las funcionalidades F01–F05, las exclusiones del MVP ni las reglas completas de seguridad. El bloque `nextjs-agent-rules` es gestionado por Next.js: no edites dentro de sus marcadores.

## 1. Lectura y precedencia

Antes de trabajar en `frontend/`, lee en este orden:

1. `../AGENTS.md`
2. `frontend/AGENTS.md` (este archivo)
3. `../MEMORY.md`
4. `frontend/MEMORY.md`
5. La guía local de Next.js que corresponda en `node_modules/next/dist/docs/`.
6. El documento dueño del tema: `../docs/arquitectura.md`, `../docs/api.md`, `../docs/requerimientos.md` o `../docs/testing.md`.

Si hay contradicción, gana el archivo de mayor precedencia y se señala antes de actuar.

## 2. Estado actual

- Next.js 16.4.5, React 19.3.0 y Tailwind CSS 4.3.4.
- App Router en `app/`; `layout.tsx`, `page.tsx` y `globals.css` son el scaffold de `create-next-app`.
- `next.config.ts` usa `cacheComponents: true`, `partialPrefetching: true` y Turbopack para CSS.
- `app/layout.tsx` tiene `lang="en"` y metadata `"Create Next App"`: son placeholders.
- No hay cliente de API, Leaflet, manifest PWA, service worker ni pruebas.
- `frontend/Dockerfile` es multi-etapa (development, build y production) y se usa desde `docker-compose.dev.yml` y `docker-compose.yml`; ver `../docs/despliegue.md` §5.
- ADR-006 acepta un monorepo con destino `apps/web`; `frontend/` se mantiene hasta el PR de migración.

## 3. Comandos

Desde la raíz del repositorio:

| Acción | Comando |
|---|---|
| Instalar | `cd frontend && pnpm install` |
| Desarrollo | `cd frontend && pnpm dev` |
| Build | `cd frontend && pnpm build` |
| Producción local | `cd frontend && pnpm start` |
| Lint | `cd frontend && pnpm lint` |
| Formato | `cd frontend && pnpm format` / `pnpm format:check` |
| Tests | No existe `pnpm test`; no lo inventes. |

## 4. Regla de versión de Next.js

Next.js 16.4.5 incluye documentación versionada. Antes de escribir código, lee la guía relevante en `frontend/node_modules/next/dist/docs/`:

- `01-app/01-getting-started/02-project-structure.md`
- `01-app/01-getting-started/03-layouts-and-pages.md`
- `01-app/01-getting-started/05-server-and-client-components.md`
- `01-app/01-getting-started/06-fetching-data.md`
- `01-app/01-getting-started/08-caching.md`
- `01-app/01-getting-started/10-error-handling.md`
- `01-app/01-getting-started/12-images.md`
- `01-app/01-getting-started/14-metadata-and-og-images.md`
- `01-app/02-guides/offline-support.md`
- `01-app/02-guides/ai-agents.md`

No asumas APIs de Next.js antiguas. El bloque gestionado por Next.js se conserva y se commitea con el trabajo.

## 5. Estructura

```text
frontend/
├── app/            # rutas, layouts, loading, error, not-found
├── components/     # UI reutilizable (mapa, rutas, estados)
├── lib/            # cliente API, formato, configuración
├── public/         # assets estáticos
└── ...
```

Reglas:

- No crear todas las carpetas de golpe; cada una nace cuando tiene contenido real.
- No mover a `src/` ni a `apps/web` sin un PR de migración.
- `components/` contiene presentación; `lib/` contiene infraestructura de cliente; `app/` contiene rutas y composición.
- No crear `app/api/` para lógica de negocio: el backend es el dueño de la API.

## 6. Server y Client Components

- Server Components por defecto.
- `'use client'` solo para estado, eventos, efectos, `window`, `localStorage`, `navigator.geolocation`, Leaflet o Web Share.
- Mantener la frontera cliente pequeña: un componente de mapa o un botón compartir, no toda la página.
- No pasar secretos ni credenciales a Client Components.
- No usar APIs de navegador en módulos que se importen desde el servidor.

## 7. Datos y API

- El contrato vive en `../docs/api.md`.
- Las llamadas pasan por `lib/api`; no disperses `fetch` por los componentes.
- No consultar Supabase directamente ni usar la `service_role` en el frontend.
- No calcular rutas ni duplicar reglas de negocio: el backend devuelve las alternativas.
- Manejar siempre carga, error y vacío; no inventar datos si la API no responde.
- Los tipos compartidos irán en `packages/contracts` cuando se ejecute la migración al monorepo.

## 8. Mapa y Leaflet

- Leaflet se carga solo en el cliente, dentro de `components/map/`.
- No importar Leaflet en Server Components ni en el layout raíz.
- Mostrar la atribución de OpenStreetMap.
- El proveedor de tiles se configura por entorno; no hardcodear un proveedor de producción.
- El mapa recibe datos del backend; no calcula rutas.
- Sin datos, mostrar un estado vacío explícito, no un mapa “inventado”.

## 9. PWA y offline

- La PWA se implementa después del shell: manifest, iconos, service worker y caché.
- Leer la guía local `offline-support.md` antes de usar APIs de offline.
- No prometer alertas en tiempo real sin conexión.
- Todo dato guardado localmente se muestra con su fecha de actualización.
- La PWA no reemplaza una app nativa; las apps nativas están fuera del MVP.

## 10. UX y accesibilidad

- Mobile-first, responsive, contraste y jerarquía clara.
- Estados de carga, error y vacío siempre presentes.
- Distinguir visualmente **confirmado**, **estimado** y **no disponible**.
- Tiempos como aproximaciones; nunca como garantía ni “tiempo real”.
- Textos de interfaz en español, sin tecnicismos innecesarios.
- Navegación y controles utilizables con una mano y en poco tiempo.

## 11. Estilos

- Tailwind CSS 4 y tokens en `app/globals.css`.
- Evitar estilos inline y valores arbitrarios sin necesidad.
- Mantener modo claro/oscuro coherente.
- No agregar una librería de UI sin evaluar primero Tailwind y los componentes existentes.

## 12. Pruebas

- No hay runner de pruebas configurado.
- Si se agrega, proponer Vitest + Testing Library y Playwright; pedir aprobación antes de instalar.
- Probar estados de carga/error/vacío, accesibilidad, compartir y la frontera cliente del mapa.
- No modificar pruebas para que pasen; corregir la causa.

## 13. Seguridad

- Solo variables públicas en el cliente (`NEXT_PUBLIC_*`).
- Nunca `SUPABASE_SERVICE_ROLE_KEY` ni secretos en el bundle.
- Validar y tratar como no confiables las respuestas de la API.
- La ubicación se pide con permiso y no se persiste sin decisión documentada.
- No registrar datos sensibles innecesarios.
- `frontend/.gitignore` ignora `.env*`; si se crea `.env.example`, agregar `!.env.example`.

## 14. Definición de terminado

Una tarea del frontend está terminada cuando:

1. Cumple el RF/HU correspondiente.
2. Respeta las responsabilidades de `../AGENTS.md` §5.
3. Es responsive y accesible.
4. Tiene estados de carga, error y vacío.
5. Respeta el contrato de `../docs/api.md`.
6. `pnpm lint`, `pnpm format:check` y `pnpm build` pasan.
7. Tiene pruebas cuando exista runner.
8. Actualiza `frontend/MEMORY.md`.

Que compile no es suficiente.

## 15. No hacer

- Calcular rutas o duplicar reglas de negocio.
- Consultar Supabase directamente o usar la `service_role`.
- Inventar estaciones, paraderos, coordenadas, tiempos o datos.
- Prometer alertas en tiempo real sin conexión.
- Editar el bloque gestionado de Next.js.
- Agregar dependencias, PWA, Leaflet o pruebas sin aprobación.
- Mover el frontend a `apps/web` sin el PR de migración.

## 16. Cuándo actualizar este documento

- Al cambiar la estructura, la versión de Next.js o los comandos.
- Al implementar API, mapa, PWA o pruebas.
- Al ejecutar la migración al monorepo.