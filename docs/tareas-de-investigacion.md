# Plan de estudio e investigación — MetroAlerta

| Campo | Valor |
|---|---|
| Estado | creado |
| Fecha | 2026-11-05 |
| Fuentes | Conversación "Plan de MetroAlerta" (ChatGPT, 2026-11-03, <https://chatgpt.com/share/6aea752c-6ae0-83e9-8e29-6135665fc569>) + `AGENTS.md` |
| Propósito | Plan de estudio para crear el proyecto: qué investigar, en qué orden, qué entrega cada tarea y a qué documento alimenta. |
| Trazabilidad | Matriz F01–F05 → RF → HU → RNF → INV → PM en `docs/trazabilidad.md`. |

**Nota de extracción (2026-11-05):** la extracción inicial del chat conservó el texto pero no los `href` de las citas. Los enlaces de este documento se recuperaron del JSON original del chat; se marcan **verificar** porque su vigencia no está comprobada.

**Cómo se usa:**
- Marcar tareas con `[x]` al completarse. Al terminar cada una, registrar el avance en `MEMORY.md` y actualizar el documento de destino.
- **Pista:** dato mencionado en la conversación o en `AGENTS.md` que sirve como punto de partida, pero **debe verificarse** (vigencia, fuente, condiciones) antes de usarse. Una pista no es un dato confirmado.
- Los enlaces recuperados del chat son pistas, no confirmaciones: cada tarea debe verificar que sigan vigentes y sus condiciones de uso (INV-08).
- Normalmente las tareas completadas alimentan documentos que aún están pendientes según `AGENTS.md` §13: `fuentes-de-datos.md`, `dominio.md`, `modelo-de-datos.md`, `api.md`, `testing.md`, `despliegue.md`, `adr/*`.

**Reglas de honestidad que aplican a toda investigación:**
- Todo dato identificado lleva fuente, fecha de obtención y (si aplica) condiciones de uso.
- Las cifras del problema (en la idea original: 1,2 millones de usuarios, 41.000 pasajeros/hora, 29%) son hipótesis hasta tener fuente, fecha, metodología y población de referencia.
- No se inventan endpoints, datasets, coordenadas ni fuentes: lo que no se pueda verificar se reporta como "no disponible / por confirmar".

**Prioridades:**

| Nivel | Significado |
|---|---|
| P0 | Bloqueante para diseñar el modelo de datos y el Route Engine. Hacer antes de escribir código de dominio. |
| P1 | Necesaria para implementar correctamente la primera versión funcional. |
| P2 | Necesaria para calidad, experiencia y despliegue. |
| P3 | Estudio post-MVP. No implementar sin aprobación explícita (`AGENTS.md` §2). |

El orden general es consistente con el orden de construcción de `AGENTS.md` §15 (fundamentos → funcionalidad → experiencia → calidad). Las prioridades indican cuándo **estudiar** cada cosa, no cuándo se implementa todo.

---

## Asignatura 1 — Fuentes de datos (P0 · bloqueante)

> La disponibilidad y la calidad de los datos es el principal riesgo técnico del proyecto (`AGENTS.md` §7). Antes de diseñar el Route Engine debe existir una muestra real inspeccionada: no construir un algoritmo esperando datos que todavía no se tienen.

| ID | Pri. | Tarea y pregunta que responde | Entregable | Pistas (verificar) | Estado |
|---|---|---|---|---|---|
| INV-01 | P0 | Red del Metro: ¿el Metro de Medellín publica datos estructurados (API, dataset, servicio geográfico) con líneas, estaciones y estado del sistema? ¿Qué permiten sus condiciones de uso? | Ficha por fuente en `docs/fuentes-de-datos.md` | Sitio oficial del Metro: <https://www.metrodemedellin.gov.co/> y <https://www.metrodemedellin.gov.co/usuarios/sistema-integrado/> (recuperadas del chat; verificar vigencia) | [ ] |
| INV-02 | P0 | Servicio ArcGIS del Metro: ¿sigue vigente, qué capas entrega (Metro, cables, tranvía, BRT) y bajo qué condiciones? La conversación indica fecha de actualización de 2023 — verificar vigencia. | Ficha + muestra descargada | Servicio ArcGIS <https://services3.arcgis.com/ARUXxWVTK25ENUfB/ArcGIS/rest/services/Sistema_Metro2/FeatureServer> (Metro, cable, tranvía y BRT); alternativas: <https://services1.arcgis.com/Qrk4Z5vQ94JXkdYM/arcgis/rest/services/Estaciones_lineas_A_y_B_majo/FeatureServer> y <https://services1.arcgis.com/Qrk4Z5vQ94JXkdYM/arcgis/rest/services/Estaciones_lineas_metro_A_B/FeatureServer> (verificar cuál sigue vigente) | [ ] |
| INV-03 | P0 | GTFS de Medellín: ¿qué cubre ese dataset? ¿Cuál es su vigencia y calidad? La conversación advierte horarios/frecuencias que "necesitan correcciones" — verificarlo en el catálogo de origen. ¿Existe GTFS Realtime? | Ficha + muestra descargada (stops, routes, trips, shapes…) | GTFS en TUMI Datahub: <https://hub.tumidata.org/es/dataset/gtfs_metro_medellin_medelln> (recuperado del chat; el resultado de búsqueda indica última actualización 2025-10-31 — verificar) | [ ] |
| INV-04 | P0 | Paraderos y rutas del AMVA: ¿qué datasets de movilidad/transporte publica el Área Metropolitana (paraderos, rutas alimentadoras/integradas, frecuencias)? ¿Formatos, actualización y condiciones? | Ficha + muestras | Portal de datos abiertos del AMVA: <https://datosabiertos.metropol.gov.co/>; página institucional: <https://www.metropol.gov.co/area/Paginas/datos%20abiertos%20amva.aspx> (recuperadas del chat; verificar vigencia) | [ ] |
| INV-05 | P0 | Datos abiertos del municipio: ¿qué aporta Medata (Alcaldía de Medellín) sobre red de transporte o movilidad? | Ficha | <https://www.medellin.gov.co/es/transparencia/medata/> (recuperada del chat; verificar vigencia) | [ ] |
| INV-06 | P0 | OpenStreetMap: ¿qué datos de la red (líneas, estaciones, paraderos) están disponibles y bajo qué licencia/atribución? ¿Qué exige la política de uso si se consulta el servidor público de tiles? | Ficha (incluye la atribución requerida en la UI) | OpenStreetMap: <https://www.openstreetmap.org/> (licencia ODbL y política de uso; alimenta RNF-13; verificar términos de tiles) | [ ] |
| INV-07 | P0 | Alertas y estado del sistema: ¿de dónde saldrán las contingencias reales? ¿Existe canal oficial estructurado (no solo publicación en web o redes)? ¿Qué condiciones limitan su ingesta? Resultado esperable: el MVP arranca con registro manual/simulado, como permite `AGENTS.md` (F01). | Ficha de fuente de alertas + decisión de arranque | App y canales del Metro: <https://www.metrodemedellin.gov.co/al-dia/descarga-nuestra-app>; ejemplos de avisos de contingencia: <https://www.metrodemedellin.gov.co/al-dia/noticias/rutas-plan-de-continuidad-y-alternativas-de-movilidad-novedad-linea-a> y <https://www.metrodemedellin.gov.co/al-dia/noticias/debido-a-novedad-tecnica-en-la-catenaria-las-estaciones-sabaneta-y-la-estrella-se-encuentran-fuera-de-servicio> (recuperadas del chat; verificar). Scraping de X descartado como dependencia del MVP | [ ] |
| INV-08 | P0 | Condiciones de uso y licencias: ¿se puede usar, almacenar y mostrar cada dato? ¿Qué atribución exige cada fuente? | Sección de condiciones en `docs/fuentes-de-datos.md`, una por fuente | Resultado de INV-01…INV-07 | [ ] |
| INV-09 | P0 | Muestras e inspección: ¿qué campos reales existen (nombre, coordenadas, líneas, horarios, geometrías)? ¿Qué tan frescos son? ¿Cómo se mapean al modelo? ¿Qué estructura tendrán los datos simulados? | Inventario de campos y mapeo propuesto en `docs/modelo-de-datos.md`; escenarios de contingencia simulada en `docs/dominio.md` | Salidas de INV-02, INV-03, INV-04 | [ ] |

**Cierre de la asignatura (entregable consolidado):** `docs/fuentes-de-datos.md` completa y validada. Hasta que exista, **no** se fija el modelo de datos ni se implementa el Route Engine con datos "esperados".

---

## Asignatura 2 — Problema, usuarios y competencia (P0/P1)

| ID | Pri. | Tarea y pregunta que responde | Entregable | Pistas (verificar) | Estado |
|---|---|---|---|---|---|
| INV-10 | P0 | Validar las cifras del problema: ¿de dónde salen realmente las cifras (1,2M de usuarios, 41.000 pasajeros/hora, 29%)? ¿Cuáles se documentan con fuente/fecha/metodología y cuáles se descartan? Entre tanto, tratarlas como hipótesis. | Sección "hipótesis y evidencia" del material del pitch | Cifras de la idea original (sin verificación al día de hoy) | [ ] |
| INV-11 | P1 | Diferenciación real: ¿qué hace hoy la app oficial del Metro (planificación multimodal entre origen y destino, combinación tren/cable/tranvía/buses, costos y tiempos, estado de líneas)? Advertencia de la conversación: ya existe planificación de viajes, por lo que la propuesta de valor se valida contra lo que no está cubierto — p. ej. alternativas proactivas ante la contingencia, el detalle de la conexión afectada y la comparación de impacto. | Análisis de diferenciación (insumo para `requerimientos.md` y el pitch) | Descripción actual del sitio oficial / app del Metro; verificar contra la propia aplicación | [ ] |
| INV-12 | P1 | Comportamiento del usuario ante una contingencia: ¿qué hacen hoy (canales usados, señalización, grupos, prensa) y qué información falta para decidir? Estudio cualitativo propio, con método y fecha. | Notas de observación/entrevistas | Investigación propia (no cifras ajenas) | [ ] |
| INV-13 | P2 | Precedentes de otras ciudades: ¿qué podrían importar sin construirlos ya? La conversación cita alertas de interrupción (Trafi) y un canal de WhatsApp de una agencia de transporte (BSAG Bremen). | Notas de referencia, marcadas como post-MVP | Trafi: <https://medium.com/trafi/trafi-introducing-real-time-disruption-alerts-5e8740627f14> y <https://www.trafi.com/post/trafi-introducing-real-time-disruption-alerts>; BSAG: <https://www.bsag.de/unternehmen/aktuelles/meldung/whatsapp-kanal-bsag-fahrgastinformation>; otros: Transit App (<https://ayuda.transitapp.com/article/482-recibir-notificaciones-sobre-incidencias-en-tu-ruta>), Citymapper (<https://ridewithvia.com/resources/how-citymapper-handles-unplanned-disruptions>), Translink (<https://vivogroup.com.au/web-design-portfolio/translink-translink-disruption-management-system>), TfL (<https://www.creativereview.co.uk/transport-for-london-tfl-go-app/>), viaje accesible (<https://thesayr.com/blog/how-to-plan-an-accessible-public-transport-journey>) — todas recuperadas del chat; verificar | [ ] |

---

## Asignatura 3 — Algoritmos y Route Engine (P1)

> Corazón técnico del proyecto. Estudio en progresión estricta: Dijkstra primero, alternativas después, A* solo si la red lo exige. Regla de la conversación: aprender cada tecnología cuando la necesite el proyecto, no todas de antemano.

| ID | Pri. | Tarea | Entregable | Referencia de estudio | Estado |
|---|---|---|---|---|---|
| INV-14 | P1 | Grafos aplicados al transporte multimodal: nodos (estaciones/paraderos), aristas (conexiones con modo: Metro/bus/caminata), pesos (tiempo estimado), dirigido vs no dirigido, y la contingencia como exclusión temporal de aristas. | Modelo de red conceptual documentado en `docs/dominio.md` | Teoría básica de grafos (cursos/documentación estándar) | [ ] |
| INV-15 | P1 | Dijkstra implementado en TypeScript sobre una red controlada de 10–20 nodos: complejidad, propiedades (pesos no negativos) y casos límite — empates de costo, nodos aislados, conexiones duplicadas, redes sin salida. | Implementación propia (primera versión sin librería) + pruebas unitarias con resultados calculados de antemano | Descripción del algoritmo + resoluciones manuales para verificar | [ ] |
| INV-16 | P1 | Calcular hasta 3 alternativas útiles y diferentes: estrategia de K-caminos (algoritmo de Yen sobre Dijkstra), eliminación de rutas casi idénticas y criterios de orden (tiempo, transbordos, caminata). Se implementa después de validar Dijkstra. | Diseño + pruebas del selector de alternativas | Algoritmo de Yen (K-shortest paths) | [ ] |
| INV-17 | P1 | Función de costo del viaje: `T_total = T_caminata + T_espera + T_transporte + T_transbordos`, con parámetros configurables (tabla de parámetros en `docs/arquitectura.md` §7) y origen de cada valor alimentado por la Asignatura 1. Los valores provisionales se marcan como **estimados**. | Definición paramétrica en `config/` + fórmula documentada | Formulación propia, validada con escenarios simulados | [ ] |
| INV-18 | P1 | Diferido: A* y heurísticas. Evaluar solo si Dijkstra resulta lento con el tamaño real de la red; no implementar de entrada. | Decisión anotada en el ADR correspondiente cuando aplique | Comparación Dijkstra vs A* | [ ] |
| INV-19 | P1 | Geometrías y representación: GeoJSON LineString por segmento, coordenadas válidas (WGS84), cómo dibujar cada modo en Leaflet (caminata punteada, bus con otro estilo…). | Convención de segmentos compartida backend/frontend, formalizada luego en `docs/api.md` | Documentación de GeoJSON y Leaflet (verificar la API real de la librería) | [ ] |

**Prueba de concepto (cierre de la asignatura):** red simulada de 10–20 nodos + Dijkstra + exclusión del tramo afectado por una contingencia simulada + máximo 3 alternativas + pruebas unitarias. Valida el corazón del producto antes de conectar datos reales.

---

## Asignatura 4 — Fundamentos de implementación (P1/P2)

> El backend NestJS ya existe en el repo (NestJS 12 con Vitest, oxlint, prettier y Supertest, según `backend/package.json`); el frontend todavía no existe. Versiones concretas: TODO(confirmar) en `AGENTS.md` §4.

| ID | Pri. | Tarea | Entregable | Nota del repo / referencia | Estado |
|---|---|---|---|---|---|
| INV-20 | P1 | NestJS: módulos, providers, DI, eventos internos. El paquete `@nestjs/observe` ya está instalado — entender de qué se encarga antes de depender de él. Organización en módulos según `docs/arquitectura.md` §4–§5. | Convención aplicada en el primer módulo real (`alerts`) | `@nestjs/observe` presente; NestJS 12 | [ ] |
| INV-21 | P1 | Next.js: App Router, Server vs Client Components, carga client-side de Leaflet (necesita funciones del navegador), estados de carga/error/vacío. | Decisiones frontend documentadas (README del frontend cuando exista) | Verificar contra la documentación oficial de Next.js | [ ] |
| INV-22 | P2 | Supabase/PostgreSQL: esquema y migraciones, permisos/RLS (`AGENTS.md` §9), conexión desde Nest (driver o SDK — decisión concreta) y necesidad real de PostGIS (según INV-09 y ADR-005). | `docs/modelo-de-datos.md` + guía de conexión | Instancia separada para desarrollo; no reutilizar entornos | [ ] |
| INV-23 | P2 | Pruebas: Vitest ya configurado (unitarias + config e2e separada), Supertest para endpoints; evaluar Playwright cuando exista interfaz. Estrategia por componente en `docs/testing.md`. | `docs/testing.md` | Scripts del repo: `test`, `test:cov`, `test:e2e` | [ ] |
| INV-24 | P2 | Docker/Compose para desarrollo y variables de entorno (`.env.example` sin valores; entornos de desarrollo, pruebas y producción separados). | `docs/despliegue.md` (por crear) | Versiones de Docker TODO(confirmar) | [ ] |
| INV-25 | P2 | CI en GitHub Actions: pipeline básica con los scripts existentes (`lint` [oxlint], `build`, `test`, `test:e2e`) en PR y en push a `main`. | Workflows + `docs/despliegue.md` | Nota: el repo usa oxlint, no ESLint (la conversación menciona ESLint) | [ ] |
| INV-26 | P2 | PWA: manifest válido, instalable, service worker cuando corresponda, caché de recursos estáticos; qué significa "sin conexión" — datos guardados localmente con su fecha (`AGENTS.md` §11). | Guía PWA aplicada al frontend | Lighthouse como auditoría técnica | [ ] |

---

## Asignatura 5 — Producto y experiencia de usuario (P2)

| ID | Pri. | Tarea | Entregable | Estado |
|---|---|---|---|---|
| INV-27 | P2 | Definición de pantallas clave y sus estados: inicio (contingencias activas + selección de origen/destino), resultados (≤3 alternativas comparables), detalle de ruta (mapa + pasos por segmento) y admin (si la decisión de CA-01 lo requiere). Reglas de `AGENTS.md` §11: carga/error/vacío siempre, lenguaje sin tecnicismos. | Bocetos + criterios por pantalla | [ ] |
| INV-28 | P2 | Contenido exacto del mensaje compartible por WhatsApp (RF-15: origen, destino, medios con estaciones/paraderos, tiempo marcado como aproximado, contingencia relacionada) y mecanismo de envío (enlace universal vs. API oficial — cierra CA-02). | Formato de mensaje aprobado + decisión de CA-02 | [ ] |
| INV-29 | P2 | Accesibilidad y legibilidad: contraste, jerarquía visual, contingencias activas distinguibles de un vistazo, etiquetas confirmado/estimado/no disponible (RF-16). | Checklist visual previo a entregar pantallas | [ ] |
| INV-30 | P2 | Métricas del MVP y de la demo institucional: tiempo de cálculo, alternativas válidas, frescura de los datos y error del tiempo estimado (cuando exista validación). Regla de honestidad: las simulaciones no se presentan como operación real, y las consultas de la app no se presentan como volumen real de pasajeros. | Plan de métricas con métodos de medición definidos | [ ] |

---

## Asignatura 6 — Post-MVP (P3 · solo estudiar, no implementar)

> Ninguna de estas entra al MVP sin aprobación explícita (`AGENTS.md` §2). Son las ideas de la conversación ("MetroAlerta 2.0"), ordenadas por el valor propuesto para después del MVP.

| ID | Tema | Qué estudiar (sin construir) | Estado |
|---|---|---|---|
| INV-31 | Mantenimientos programados y simulación para entidades (calendario de cierres y alertas anticipadas; el análisis "qué pasaría si cerramos un tramo" se señala en la conversación como oportunidad institucional) | Fuentes de información de mantenimientos (ligadas a INV-07); reglas para separar escenarios simulados de datos reales | [ ] |
| INV-32 | Notificaciones personalizadas / trayectos favoritos (Web Push, Service Workers, permisos de navegador; sin app nativa; sin notificaciones masivas, excluidas en `AGENTS.md` §2) | Documentación de Web Push/PushManager; compatibilidad real de navegadores | [ ] |
| INV-33 | Rutas accesibles (ascensores operativos por estación, accesibilidad de paraderos); complejidad alta porque dependen de datos confiables que probablemente no existen aún | Fuentes por investigar; diseño de datos de accesibilidad | [ ] |
| INV-34 | Analítica institucional (históricos, mapa de impacto con PostGIS y grafos, predicción de demanda — la predicción explícitamente post-MVP) | Diseño de dashboards; límites de privacidad y de uso de datos de consultas | [ ] |
| INV-35 | Reportes ciudadanos con moderación (estados pendiente / en revisión / verificado, expiración de reportes, anti-abuso; un reporte no es automáticamente una contingencia activa) | Diseño del modelo de moderación; precedentes de crowdsourcing en transporte | [ ] |

---

## Ruta de aprendizaje (paralela al desarrollo)

| Tema | Qué aprender | Cuándo |
|---|---|---|
| Ingeniería de requisitos | Historias de usuario, criterios de aceptación (GWT), RF/RNF | Ya documentado en `requerimientos.md`; se refuerza en el primer planning real |
| Arquitectura | Clean Architecture, monolito modular, DDD ligero | Reflejada en `arquitectura.md`; se aprende con el primer módulo |
| Grafos y algoritmos | Dijkstra, K-caminos (Yen), A* (diferido) | Asignatura 3, antes del Route Engine |
| Bases de datos | PostgreSQL, migraciones, permisos/RLS, PostGIS (si aplica) | INV-22, antes de la primera migración |
| Datos de transporte | GTFS, GeoJSON, coordenadas WGS84 | Con la Asignatura 1 e INV-19 |
| Backend | NestJS 12: módulos, providers, DI, eventos internos | INV-20, junto al primer módulo |
| Frontend | Next.js App Router, Server/Client Components, Leaflet client-side | INV-21, junto al primer componente |
| Pruebas | Vitest (ya configurado), Supertest; Playwright después | INV-23 |
| Lint/format | oxlint + prettier (lo que el repo ya usa — no ESLint) | Ahora, desde el primer commit real |
| DevOps | Docker, variables de entorno, entornos, GitHub Actions | INV-24/INV-25, antes del primer despliegue |
| Seguridad | Validación de entradas, secretos en env, RLS, ubicación con permiso | Continuo — reglas en `AGENTS.md` §9 |
| Patrones de diseño | Repository, Adapter, Strategy, DI — solo con problema concreto que los justifique | En cada módulo, según la necesidad |

---

## Orden de trabajo recomendado (resumen)

1. **Asignatura 1 (P0)** → `fuentes-de-datos.md` + muestras inspeccionadas.
2. **Asignatura 2 (P0/P1)** → hipótesis validadas + diferenciación real.
3. **Prueba de concepto (Asignatura 3)** → Dijkstra + red simulada + exclusión de segmentos + pruebas.
4. **Asignatura 4 (P1/P2)** → módulos backend reales siguiendo `arquitectura.md`.
5. **Asignatura 5 (P2)** → pantallas, mensaje compartible, métricas.
6. **Asignatura 6 (P3)** → solo estudio; toda construcción necesita propuesta y aprobación previa.

Al completar tareas o asignaturas: actualizar `MEMORY.md`, mover los resultados a los documentos objetivos (`fuentes-de-datos.md`, `dominio.md`, `modelo-de-datos.md`, `testing.md`, `despliegue.md`, `adr/`) y marcar aquí el estado `[x]`. La correspondencia entre tareas y funcionalidades post-MVP está en `docs/trazabilidad.md`.
