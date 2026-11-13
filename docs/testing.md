# Estrategia de pruebas — MetroAlert

| Campo | Valor |
|---|---|
| Estado | creado |
| Fecha | 2026-11-05 |
| Fuentes | `AGENTS.md` §10, `docs/requerimientos.md`, `docs/arquitectura.md`, `backend/package.json`, `.github/workflows/ci.yml` |
| Propósito | Definir qué se prueba, con qué herramienta y en qué nivel, por componente. |

**Para agentes:**
- Usa este documento para decidir dónde va una prueba y qué debe verificar.
- No asumas que las pruebas existen: al 2026-11-05 solo hay el esqueleto e2e de NestJS y no hay pruebas de dominio.
- La fuente de verdad del comportamiento son los RF/HU; la matriz está en `docs/trazabilidad.md`.

---

## 1. Estado actual

- Backend NestJS 12 con Vitest 4 y dos configuraciones: unitaria (`vitest.config.ts`) y e2e (`vitest.config.e2e.ts`); Supertest 7 para HTTP.
- `pnpm test` no encuentra archivos `.spec.ts` y falla con "No test files found".
- `pnpm test:e2e` ejecuta `test/app.e2e-spec.ts`, el esqueleto de NestJS: espera `GET /` 200 y recibe 404 porque `AppModule` no tiene controladores.
- El CI (`.github/workflows/ci.yml`) ejecuta instalación, `lint`, `format:check`, `audit` y `build`; los pasos de pruebas están comentados.
- No hay base de datos conectada.
- El frontend existe como scaffold de Next.js 16.4.5, React 19.3 y Tailwind 4.3.4; no tiene runner ni suites. `pnpm lint` está disponible.

## 2. Principios

- Probar comportamiento observable a través de interfaces públicas, no detalles internos.
- Un test por regla; cubrir éxito, error y límites.
- Pruebas deterministas: sin red, sin fuentes externas y con fechas/reloj controlados.
- Datos de prueba identificados como `simulado`; nunca datos de producción ni secretos.
- No modificar pruebas para que pasen; corregir la causa.
- La cobertura es una señal, no un objetivo aislado.

## 3. Niveles y herramientas

| Nivel | Herramienta | Alcance | Cuándo |
|---|---|---|---|
| Unitarias | Vitest | dominio, route-engine, validaciones, sharing | desde la primera regla de negocio |
| Integración | Vitest + Supertest / Supabase local | repositorios, adaptadores, API | cuando exista base de datos o adaptador |
| E2E | Vitest + Supertest | flujo HTTP crítico | cuando existan endpoints reales |
| Frontend | Vitest + Testing Library / Playwright (por decidir) | componentes, accesibilidad, flujo PWA | el scaffold existe; runner no configurado |
| Carga | Herramienta por decidir | contingencia + consultas concurrentes | cuando el flujo esté estable |

No se agregan dependencias de prueba sin justificarlo y aprobarlo (`AGENTS.md` §9).

## 4. Por componente

### 4.1 `alerts`

- Estados `activa`, `resuelta`, `vigencia indeterminada`; transiciones válidas e inválidas.
- Una publicación no verificada no se convierte en contingencia activa.
- Duplicados, cambios de estado y fecha de última actualización.
- Datos faltantes se conservan sin valor; no se inventan campos.

### 4.2 `transport`

- Importadores GTFS/ArcGIS con fixtures locales; sin red.
- Validación de nombres, coordenadas WGS84, líneas y paraderos.
- Duplicados, registros incompletos y geometrías ausentes.
- Contrato del repositorio con una implementación en memoria.

### 4.3 `route-engine`

- Grafo controlado de 10–20 nodos con resultados calculados a mano.
- Dijkstra: camino mínimo, empates, nodos aislados, conexiones duplicadas, red sin salida.
- K-caminos (Yen): hasta 3 alternativas, deduplicación y orden.
- Exclusión de aristas afectadas por contingencias activas confirmadas; reversión al resolverse.
- `T_total = T_caminata + T_espera + T_transporte + T_transbordos` con parámetros configurables y determinismo.
- Sin datos suficientes: respuesta explícita, sin rutas inventadas.
- Metadatos de estimación: método, frescura y nivel de confianza.

### 4.4 `geolocation`

- Coordenadas válidas y fuera de rango; distancias; nodos cercanos.
- La ubicación no se persiste sin decisión documentada.

### 4.5 `sharing`

- El mensaje incluye origen, destino, medios, estaciones/paraderos, tiempo aproximado, contingencia y última actualización.
- Funciona sin cuenta ni autenticación.
- No se promete envío por API oficial; el mecanismo queda en CA-02.

### 4.6 API y presentación

- Validación de entrada y errores controlados sin detalles internos.
- Códigos HTTP correctos (200/400/404/422 según el caso).
- Supertest contra `AppModule` real; el esqueleto actual debe reemplazarse por el flujo del MVP.

### 4.7 Persistencia (cuando exista)

- Migraciones aplicables y reversibles; RLS; aislamiento por entorno.
- Repositorios contra una base de prueba local, nunca producción.

### 4.8 Ingesta (cuando exista)

- Un adaptador por fuente con fixtures; fuente caída → degradación e informe.
- Duplicados, cambios de estado y errores de estructura.
- Sin scraping de X como dependencia del MVP.

### 4.9 Frontend y PWA

- El scaffold existe (Next.js 16.4.5); no hay runner de pruebas configurado.
- Estados de carga, error y vacío; etiquetas confirmado/estimado/no disponible.
- Accesibilidad y contraste; offline con fecha de actualización visible.
- Compartir y navegación principal.
- Cuando se agregue runner, probar la frontera cliente del mapa y el flujo de compartir.

## 5. Datos y fixtures

- Ubicación propuesta: `backend/test/fixtures/` (por crear), con JSON/GeoJSON pequeños y legibles.
- Cada fixture declara origen `simulado` y el escenario que representa.
- Fechas fijas o reloj inyectable; evitar `Date.now()` en aserciones.
- Sin secretos, datos personales ni respuestas reales de producción.

## 6. Convenciones

- Unitarias: `src/**/*.spec.ts`; e2e: `test/**/*.e2e-spec.ts`.
- Un `describe` por caso de uso y un `it` por regla observable.
- Nombres que digan qué se espera y en qué condición.
- Evitar snapshots frágiles; usarlos solo para formatos estables y revisados.

## 7. CI

- Estado actual: `lint`, `format:check`, `audit` y `build`; pruebas comentadas.
- Propuesta: habilitar `pnpm test` cuando exista la primera prueba de dominio; luego `test:cov`; e2e cuando existan endpoints.
- No habilitar el esqueleto actual: falla con 404 y no representa el MVP.
- Cualquier cambio al workflow requiere aprobación (`AGENTS.md` §9).
- El CI valida el frontend con `lint`, `format:check`, `audit`, `build` y `tsc --noEmit`.

## 8. Cobertura y criterios de terminado

- Sin umbral numérico inicial; se define al medir (CA-05 para rendimiento).
- Toda RF/HU implementada debe tener al menos una prueba que la nombre.
- Reglas críticas obligatorias: RF-10/RNF-03 (exclusión de segmentos), RF-12 (sin datos), RF-13/RF-14 (tiempo y metadatos), RF-02/RF-03 (estados y verificación).
- Una funcionalidad no está terminada sin pruebas (`AGENTS.md` §10).
- `docs/dominio.md` es la fuente del lenguaje y las reglas; `docs/modelo-de-datos.md` traduce esas reglas a tablas; `docs/api.md` define el contrato HTTP.
- Trazabilidad: `docs/trazabilidad.md`.

## 9. Pendientes

- Reemplazar `test/app.e2e-spec.ts` por el flujo real del MVP.
- Definir runner de frontend y pruebas de accesibilidad.
- Definir base de datos de prueba y estrategia de migraciones.
- Definir herramienta de carga y umbrales medidos.
- Habilitar las pruebas en CI cuando exista la primera suite.
