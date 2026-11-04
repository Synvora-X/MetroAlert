<p align="center">
  <!-- Imagen temporal: se planea reemplazarla por material propio del proyecto -->
  <img src="https://mir-s3-cdn-cf.behance.net/project_modules/source/24313a33389971.56a99479800dc.gif" alt="Metro de Medellín" width="640">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/estado-en%20construcci%C3%B3n-orange" alt="Estado: en construcción">
</p>

# MetroAlerta

Aplicación web progresiva (PWA) para el Valle de Aburrá (Antioquia, Colombia). Cuando hay una contingencia en el Metro de Medellín (interrupción, suspensión, cierre de estación o retraso), MetroAlerta muestra qué está afectado y propone **hasta 3 rutas alternativas** con tiempo estimado.

No reemplaza a las apps de navegación: es una herramienta especializada para contingencias del transporte público. El contexto del producto, el alcance del MVP y las reglas del proyecto viven en [AGENTS.md](AGENTS.md).

## 🚧 En construcción

El proyecto apenas inicia. Hoy existe únicamente el esqueleto del **backend**; el resto de componentes están previstos pero no implementados.

| Componente | Estado |
|---|---|
| Backend (NestJS) | ✅ Esqueleto funcional: arranca y compila |
| Frontend (Next.js) | ⏳ Previsto, sin implementar |
| Base de datos (Supabase / PostgreSQL) | ⏳ Previsto, sin implementar |
| Ingesta de alertas (Python o n8n) | ⏳ Previsto, sin implementar |
| Mapa (Leaflet + OpenStreetMap) | ⏳ Previsto, sin implementar |

## Stack actual

| Herramienta | Versión | Uso |
|---|---|---|
| NestJS + TypeScript | 12 / 6 | API, lógica de negocio y cálculos |
| pnpm | 11.6 | Gestión de dependencias (Node 22) |
| Vitest | 4 | Pruebas unitarias y e2e (configurado, aún sin pruebas) |
| oxlint + Prettier | 1.x / 3 | Lint y formato de código |
| GitHub Actions | — | CI: lint, formato, audit y build (pruebas desactivadas hasta que existan) |

## Comandos

Todos los comandos se ejecutan dentro de `backend/`:

| Acción | Comando |
|---|---|
| Instalar dependencias | `pnpm install` |
| Levantar en desarrollo | `pnpm start:dev` |
| Build | `pnpm build` |
| Lint | `pnpm lint` |
| Formato | `pnpm format` / `pnpm format:check` |
| Pruebas | `pnpm test` / `pnpm test:e2e` |

## Estructura

```
.
├── backend/
│   ├── src/                  # main.ts (bootstrap, puerto 5000) y app.module.ts
│   └── test/                 # pruebas e2e
└── .github/workflows/ci.yml  # pipeline de CI
```

## Documentación

- [AGENTS.md](AGENTS.md) — documento rector: qué es MetroAlerta, alcance del MVP, dominio, reglas de trabajo y protocolo de agentes. La documentación de detalle (`/docs`: arquitectura, modelo de datos, API, etc.) está pendiente; la lista completa está en AGENTS.md, sección 13.

## Crédito de la portada

- Animación de encabezado: *Metro de Medellín*, de [David López Ramírez](https://www.behance.net/davidlo) — [Behance, 2015](https://www.behance.net/gallery/33389971/Metro-de-Medellin). Imagen temporal; será reemplazada por material propio del proyecto.
