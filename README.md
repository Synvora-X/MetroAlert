<p align="center">
  <!-- Imagen temporal: se planea reemplazarla por material propio del proyecto -->
  <img src="docs/assets/brand/metro_gif.gif" alt="Metro de Medellín" width="800">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/estado-en%20construcci%C3%B3n-orange" alt="Estado: en construcción">
  <img src="https://img.shields.io/badge/backend-NestJS%2012-E0234E" alt="Backend: NestJS 12">
  <img src="https://img.shields.io/badge/frontend-Next.js%2016.4-000000" alt="Frontend: Next.js 16.4">
  <img src="https://img.shields.io/badge/docs-11%20documentos%20%2B%206%20ADR-blue" alt="Documentación: 11 documentos y 6 ADR">
</p>

# MetroAlert

Aplicación web progresiva (PWA) para el Valle de Aburrá (Antioquia, Colombia). Cuando hay una contingencia en el Metro de Medellín (interrupción, suspensión, cierre de estación o retraso), MetroAlert muestra qué está afectado y propone **hasta 3 rutas alternativas** con tiempo estimado.

No reemplaza a las apps de navegación: es una herramienta especializada para contingencias del transporte público. El contexto del producto, el alcance del MVP y las reglas del proyecto viven en [AGENTS.md](AGENTS.md).

## 🎯 El problema

Saber que hay una contingencia no dice qué hacer para continuar el viaje. MetroAlert conecta "hay una contingencia" con "qué hago ahora": identifica la afectación y ofrece alternativas concretas con pasos comprensibles (Metro, buses integrados o alimentadores, caminata y transbordos).

Las afirmaciones sobre el impacto del problema (dependencia del Metro, tiempo perdido buscando alternativas, información dispersa) son **hipótesis por validar**; todavía no hay cifras oficiales verificadas.

## 🚧 Estado del proyecto

El proyecto está en **fase de fundamentos** (ver `AGENTS.md` §15): la documentación de base está creada y el código es todavía un esqueleto. No hay flujo funcional de punta a punta.

| Componente | Estado real |
|---|---|
| Documentación de fundamentos | ✅ 11 documentos en `docs/` y 6 ADR (3 aceptados, 3 propuestos) |
| Backend (NestJS 12) | 🟡 Esqueleto: arranca, compila y pasa lint/formato; sin módulos de dominio ni endpoints |
| Frontend (Next.js 16.4.5) | 🟡 Scaffold con App Router, Tailwind 4 y Prettier; sin cliente API, mapa, PWA ni pruebas |
| CI (GitHub Actions) | ✅ Valida backend y frontend en `push` y PR hacia `main`; pasos de test comentados |
| Docker / Compose | 🟡 Dockerfiles multi-etapa y dos compose (desarrollo y producción local) configurados; falta validación de punta a punta |
| Base de datos (Supabase / PostgreSQL) | ⛔ No conectada; modelo de datos conceptual en `docs/modelo-de-datos.md` |
| Ingesta de alertas (Python o n8n) | ⛔ No implementada; ADR-002 sigue propuesto |
| Mapa (Leaflet + OpenStreetMap) | ⛔ No implementado; ADR-004 sigue propuesto |

Leyenda: ✅ verificado · 🟡 parcial · ⛔ no implementado.

## 🧱 Stack

| Herramienta | Versión | Uso |
|---|---|---|
| NestJS + TypeScript | 12.0.1 / 6.0.2 | API, lógica de negocio y cálculo de rutas |
| Next.js + React + Tailwind | 16.4.5 / 19.3.0 / 4.3.4 | Interfaz, mapa y capacidades PWA |
| pnpm | 11.6.0 | Gestión de dependencias (Node 22) |
| Vitest | 4.1.2 | Pruebas del backend (configurado, aún sin suites) |
| oxlint + Prettier (backend) / ESLint + Prettier (frontend) | 1.58.0 / 3.x | Lint y formato |
| GitHub Actions | — | Integración continua |

Las versiones salen de `backend/package.json` y `frontend/package.json`. Que una tecnología aparezca aquí no implica que esté implementada.

## 📁 Estructura

```
MetroAlert/
├── backend/                     # API NestJS 12
│   ├── src/                     # main.ts (puerto 5000) y app.module.ts
│   ├── test/                    # e2e de esqueleto
│   ├── Dockerfile               # etapas development, build y production
│   └── .env.example             # variables sin valores
├── frontend/                    # Next.js 16.4.5 con App Router
│   ├── app/                     # layout, página y estilos globales
│   └── Dockerfile               # etapas development, build y production
├── docs/                        # documentación de fundamentos
│   └── adr/                     # ADR-001 … ADR-006
├── .github/workflows/ci.yml     # pipeline de CI (backend y frontend)
├── docker-compose.yml           # producción local
├── docker-compose.dev.yml       # desarrollo con volúmenes
├── AGENTS.md                    # reglas permanentes del proyecto
└── MEMORY.md                    # estado temporal del trabajo
```

Existe además un `AGENTS.md` y un `MEMORY.md` locales en `backend/` y en `frontend/`, con reglas propias de cada capa. No reemplazan a los de la raíz.

## ✅ Requisitos

- **Node.js 22** (versión usada por el CI).
- **pnpm 11.6.0** (`corepack enable` la activa según `packageManager`).
- **Docker** opcional, solo si se levanta con Compose.

## 🚀 Puesta en marcha

Cada componente instala sus propias dependencias. El repositorio no tiene instalación desde la raíz.

### Backend

```bash
cd backend
cp .env.example .env     # completar solo lo que se necesite; nunca versionar valores
pnpm install
pnpm start:dev           # http://localhost:5000
```

### Frontend

```bash
cd frontend
pnpm install
pnpm dev                 # http://localhost:3000
```

### Docker (opcional)

```bash
# Desarrollo, con el código local montado dentro de los contenedores
docker compose -f docker-compose.dev.yml up --build

# Producción local, usando las etapas de producción de cada Dockerfile
docker compose up --build -d
```

Los compose leen `backend/.env` y `frontend/.env`; ambos están fuera de Git. El detalle completo, los puertos y los límites conocidos están en [`docs/despliegue.md`](docs/despliegue.md) §5.

## 🧪 Pruebas y calidad

| Componente | Comandos | Estado |
|---|---|---|
| Backend | `pnpm test`, `pnpm test:e2e`, `pnpm test:cov` | Configurados, sin suites: `pnpm test` no encuentra pruebas y el e2e es el esqueleto de NestJS (falla con 404) |
| Backend | `pnpm lint`, `pnpm format:check`, `pnpm build` | Verificados en local y en CI |
| Frontend | `pnpm lint`, `pnpm format:check`, `pnpm build` | Verificados en CI; sin runner de pruebas |

Los pasos de test del CI están comentados a propósito hasta que existan suites reales. La estrategia y los pendientes viven en [`docs/testing.md`](docs/testing.md).

## 📚 Documentación

| Documento | Contiene |
|---|---|
| [AGENTS.md](AGENTS.md) | Documento rector: producto, alcance del MVP, dominio, reglas de trabajo |
| [MEMORY.md](MEMORY.md) | Estado actual, decisiones recientes y siguientes pasos |
| [docs/requerimientos.md](docs/requerimientos.md) | Requerimientos funcionales y no funcionales, historias de usuario y cuestiones abiertas |
| [docs/arquitectura.md](docs/arquitectura.md) | Componentes, flujo y límites entre capas |
| [docs/dominio.md](docs/dominio.md) | Glosario, estados y reglas de negocio ampliadas |
| [docs/modelo-de-datos.md](docs/modelo-de-datos.md) | Modelo conceptual, lógico y físico propuesto (PostGIS) |
| [docs/api.md](docs/api.md) | Contrato REST propuesto del MVP |
| [docs/fuentes-de-datos.md](docs/fuentes-de-datos.md) | Fuentes investigadas, condiciones de uso y estado de verificación |
| [docs/testing.md](docs/testing.md) | Estrategia de pruebas por componente y estado real del CI |
| [docs/despliegue.md](docs/despliegue.md) | Entornos, Docker, CI, CD y variables |
| [docs/tareas-de-investigacion.md](docs/tareas-de-investigacion.md) | Plan de investigación priorizado (INV-01 … INV-35) |
| [docs/funcionalidades-post-mvp.md](docs/funcionalidades-post-mvp.md) | Hoja de ruta post-MVP, no aprobada para el alcance actual |
| [docs/trazabilidad.md](docs/trazabilidad.md) | Matriz F01–F05 → RF → HU → RNF → INV → PM |
| [docs/adr/](docs/adr/README.md) | Decisiones arquitectónicas y su estado |

Varios documentos están creados pero **incompletos**; la tabla de pendientes está en `AGENTS.md` §13.

## 🤝 Convenciones de trabajo

- **Ramas:** una por tarea, con prefijo `feat/`, `fix/`, `docs/` o `chore/`. No se trabaja directamente sobre `main`.
- **Commits:** [Conventional Commits](https://www.conventionalcommits.org/es/v1.0.0/), uno por cambio lógico completado.
- **Pull Requests:** todo cambio entra a `main` por PR y el CI debe pasar.
- **Secretos:** solo `.env.example` versionado; credenciales de Supabase y de hosting nunca en Git, y la `service_role` jamás en el frontend.
- **Datos:** cada dato declara su origen (`oficial`, `abierto`, `estimado` o `simulado`); no se presentan estimaciones como hechos.

## 🗺️ Siguientes pasos

1. Ejecutar INV-01 … INV-09 y completar `docs/fuentes-de-datos.md` con evidencia real de las fuentes.
2. Cerrar los ADR-002 (ingesta), ADR-003 (alojamiento) y ADR-004 (tiles de producción).
3. Prueba de concepto del motor de rutas: Dijkstra sobre una red simulada de 10–20 nodos con exclusión de segmentos afectados.
4. Primer módulo real del backend: `alerts`, siguiendo `docs/arquitectura.md`.
5. Frontend: shell de la aplicación, cliente API, contingencias, mapa y PWA.

## ⚠️ Fuera del MVP

Predicción de fallas o congestión, seguimiento GPS de buses en tiempo real, integración con transporte privado, apps nativas, analítica avanzada e integraciones institucionales sin autorización. Nada de esto se implementa sin aprobación explícita; el detalle está en `AGENTS.md` §2 y en [`docs/funcionalidades-post-mvp.md`](docs/funcionalidades-post-mvp.md).

## 📄 Licencia

Sin licencia definida todavía (`TODO(confirmar)`). El código no debe reutilizarse ni distribuirse hasta que el proyecto la declare.
