# Funcionalidades post-MVP — Hoja de ruta "MetroAlerta 2.0"

| Campo | Valor |
|---|---|
| Estado | creado |
| Fecha | 2026-11-05 |
| Fuentes | Conversación "Plan de MetroAlerta" (ChatGPT, 2026-11-03, <https://chatgpt.com/share/6aea752c-6ae0-83e9-8e29-6135665fc569>), sección "MetroAlerta 2.0" + `AGENTS.md` (§2, §5, §8) |
| Relación con otros documentos | `docs/requerimientos.md` §4 lista estas candidatas en una línea cada una; este documento las desmenuza paso a paso. El alcance y las exclusiones mandatorias están en `AGENTS.md` §2 (fuente de verdad). El plan de estudio asociado vive en `docs/tareas-de-investigacion.md`, Asignatura 6; la correspondencia `INV ↔ PM` está en `docs/trazabilidad.md`. |

**Advertencia de alcance (leer antes de usar este documento):**

- Este documento es una **hoja de ruta propuesta**, no un compromiso. Ninguna funcionalidad de aquí entra al MVP sin aprobación explícita del desarrollador (`AGENTS.md` §2).
- Las exclusiones de `AGENTS.md` §2 se mantienen aunque la versión futura se apruebe: predicción de fallas del Metro o de congestión vial e IA predictiva, seguimiento GPS de buses en tiempo real, integración con transporte privado, apps nativas, analítica avanzada de movilidad y modelos de optimización de tráfico, notificaciones masivas avanzadas e integraciones institucionales sin autorizaciones. La V4 (§4 de este documento) toca esas fronteras: exige aprobación y justificación técnica propias.
- La fuente de verdad sobre qué funcionalidades existen realmente hoy es el código; nada de lo descrito aquí está implementado.
- La idea central de la evolución (conversación): pasar de **planificador de rutas** a **sistema de asistencia al usuario durante interrupciones** (¿qué sucede? ¿cómo me afecta? ¿qué opciones tengo? ¿cuál me conviene? ¿cómo me mantengo informado?).

**Convenciones de este documento:**

- Cada funcionalidad tiene ID `PM-xx` (post-MVP). Los pasos describen qué hacer, en qué orden, y qué debe existir antes (prerrequisito).
- **Valor** y **dificultad** son la estimación de la conversación (conversación §11.1), no una medición: sirven para ordenar el estudio, no para planificar fechas.
- Las reglas de honestidad de `AGENTS.md` §8 aplican a todo: los ejemplos numéricos de la conversación son ilustrativos (simulados), nunca datos reales.

---

## 1. Hoja de ruta en cuatro versiones

La conversación propone organizar la evolución en cuatro versiones (secciones 11 y 15). El MVP completado es la base de todo lo demás (Etapa 1 = núcleo: requisitos, arquitectura, fuentes, grafo, Route Engine, contingencias, mapa y alternativas).

| Versión | Objetivo | Funcionalidades principales |
|---|---|---|
| **V2 — Asistencia al pasajero** | Convertir la app en acompañante del desplazamiento | Seguimiento y clasificación de contingencias, trayectos favoritos, notificaciones personalizadas, mantenimientos programados, rutas según preferencias, rutas accesibles, replanificación, instrucciones paso a paso, reportes ciudadanos moderados, comparador de impacto, retroalimentación, compartir estado, modo offline, multilenguaje |
| **V3 — Plataforma institucional** | Herramientas para Metro/AMVA y operadores ("MetroAlerta Control") | Dashboard de contingencias, mapa de impacto, analítica histórica, simulador de escenarios, panel de calidad de datos, API institucional |
| **V4 — Investigación futura** | Evolución avanzada de asistencia y análisis | Predicción de demanda, información operativa en tiempo real, modelos avanzados de estimación y simulación compleja |

**Priorización de la conversación** (valor vs. dificultad, conversación §11.1):

| Funcionalidad | Valor | Dificultad |
|---|---|---|
| Seguimiento de contingencias | Muy alto | Media |
| Mantenimientos programados | Muy alto | Media |
| Notificaciones personalizadas | Muy alto | Media-alta |
| Rutas según preferencias | Muy alto | Media-alta |
| Rutas accesibles | Muy alto | Alta |
| Simulación de escenarios | Muy alto | Alta |
| Dashboard institucional | Muy alto | Media-alta |
| Reportes ciudadanos | Alto | Media-alta |
| Replanificación | Alto | Alta |
| Predicción de demanda | Alto | Muy alta |

Las cuatro que la conversación recomienda investigar primero tras el MVP: mantenimientos programados, notificaciones de recorridos habituales, rutas accesibles y simulador de escenarios.

---

## 2. V2 — Asistencia al pasajero

### PM-01 — Línea de tiempo y detalle de contingencia

**Objetivo:** cada contingencia tiene una página de detalle con su ciclo de vida (detección → confirmación → actualizaciones → normalización), en lugar de aparecer y desaparecer de forma mágica.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Muy alto | Media | MVP: módulo `alerts` con estados y última actualización (RF-01…RF-04) |

**Pasos:**

1. Registrar cada cambio de estado de una contingencia como un evento fechado (quién/cuándo: detección, confirmación, actualización, normalización). El historial es solo agregación de lo ya persistido: no inventar eventos.
2. Endpoint de detalle por contingencia (contenido detallado en `docs/api.md`, pendiente).
3. UI de línea de tiempo con fecha y hora de cada evento y la fecha de la última actualización visible (RNF-05).
4. Marcar explícitamente los datos simulados/ilustrativos si algún evento proviene de prueba (RF-16 del MVP).

**Riesgos:** la página de detalle no debe presentar una simulación como si hubiera ocurrido en tiempo real (advertencia explícita de la conversación).

### PM-02 — Clasificación de contingencias y nivel de severidad

**Objetivo:** tipificar cada incidencia y asignarle severidad con reglas documentadas y revisables.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Alto | Media | PM-01 |

**Pasos:**

1. Definir el catálogo de tipos en `docs/dominio.md` (pendiente): interrupción total, interrupción parcial, cierre de estación, mantenimiento programado, retrasos, afectación externa (p. ej. vial sobre rutas alimentadoras).
2. Definir los niveles de severidad y la regla de cálculo que los asigna; la regla queda escrita junto con la fórmula, no en código disperso.
3. Alimentar la severidad con la clasificación del tipo y criterios objetivos (p. ej. número de conexiones suspendidas). La severidad calculada por reglas debe poder revisarse y sobreescribirse por un administrador.
4. Probar con escenarios unitarios: mismo escenario → misma severidad siempre (determinismo).

**Riesgos:** no depender exclusivamente de una clasificación automática; toda severidad sin datos suficientes se informa como no disponible o indeterminada (`AGENTS.md` §6).

### PM-03 — Trayectos favoritos y notificaciones personalizadas

**Objetivo:** el usuario guarda su recorrido habitual (origen/destino frecuentes, líneas de interés, horarios de interés) y recibe una notificación cuando una contingencia puede afectarlo.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Muy alto | Media-alta | MVP + PM-01/PM-02; PWA instalada; permiso de notificaciones |

**Pasos:**

1. Modelo de preferencias por usuario: usuario anónimo o con cuenta (decisión previa para cerrar; impacta en el futuro módulo de preferencias/clientes, post-MVP; ver PC-02), trayectos guardados, suscripciones por línea, horarios de interés, notificación de cierres programados y de actualizaciones de contingencias.
2. Sustitución/apoyo: Web Push API + Service Workers + PWA; NestJS gestiona suscripciones. Verificar la compatibilidad por navegador (las notificaciones web dependen del navegador y del permiso del dispositivo; no se promete nada sin verificación).
3. Evaluación de impacto por trayecto: al activarse una contingencia, comprobar si sus segmentos intersectan el trayecto guardado y solo entonces notificar.
4. Contenido del mensaje: análogo al formato de PM-13 (origen, destino, contexto de la contingencia, enlace a las alternativas).
5. Gestión de consentimiento y salida: el usuario puede cancelar cada suscripción y todas a la vez; no se envían notificaciones masivas dirigidas a todos (excluidas por `AGENTS.md` §2).

**Riesgos:** el modelo de cuentas del MVP (CA-01 de `requerimientos.md`) debe resolverse antes; no crear el módulo de clientes sin decisión documentada.

### PM-04 — Rutas según preferencias del usuario

**Objetivo:** dejar de optimizar solo por tiempo: el usuario elige qué prioriza (más rápida / menos caminata / menos transbordos) y el Route Engine aplica ponderaciones.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Muy alto | Media-alta | MVP: Route Engine funcional (RF-10…RF-14) |

**Pasos:**

1. Generalizar la función de costo del motor (definición de la conversación):
   `C(r) = w_t·T + w_w·W + w_x·X + w_c·C`, donde `T` = tiempo total, `W` = tiempo caminando, `X` = cantidad de transbordos, `C` = costo monetario estimado y `w` = ponderaciones según la preferencia elegida.
2. Ajustar los pesos para comparación: homogeneizar unidades antes de ponderar (minutos, minutos a pie, conteo, moneda) — decisión a documentar con la fórmula corregida.
3. Generar alternativas con perfiles distintos (más rápida, menos caminata, menos transbordos) y presentarlas comparadas (tabla de tiempo y características por alternativa).
4. Pruebas unitarias del selector con redes controladas: dos preferencias distintas → recomendaciones distintas para el mismo desplazamiento.

**Riesgos:** el costo monetario requiere datos de tarifas oficiales con origen explícito; sin fuente validada se excluye ese término (o se marca estimado).

### PM-05 — Rutas accesibles y estado de infraestructura

**Objetivo:** permitir recorridos según necesidades de accesibilidad (silla de ruedas, movilidad reducida, coches de bebé, minimizar escaleras) y advertir cuando falta información.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Muy alto | Alta (por los datos) | MVP + PM-04; datos de accesibilidad por nodo |

**Pasos:**

1. Modelo de datos de accesibilidad por nodo (estación/paradero): ascensor operativo, acceso a nivel, escaleras, rampa. Estado temporal por nodo: ascensores y escaleras eléctricas fuera de servicio (información verificable; incluye el punto 8.6 de la conversación: "estado de infraestructura").
2. Fuente de datos primero: investigar de dónde sale esta información (INV-33 del plan de estudio) antes de diseñar el cálculo; sin fuente confiable, no se construye.
3. Restricciones en el grafo: excluir o penalizar nodos/aristas no accesibles según la preferencia (evitar estaciones sin ascensores operativos, priorizar conexiones con acceso a nivel, evitar recorridos con escaleras, identificar paraderos accesibles).
4. Advertencia obligatoria cuando un nodo del recorrido no tiene información de accesibilidad ("no hay información suficiente" — regla de honestidad).
5. Caso especial de la conversación: durante mantenimientos, una estación puede seguir operando pero con su acceso temporalmente fuera de servicio: el estado del ascensor debe poder deshabilitar el acceso aunque la estación siga activa.

**Riesgos:** es la funcionalidad con mayor dependencia de datos que probablemente no existen aún; complejidad alta por eso, no por el algoritmo.

### PM-06 — Comparador de impacto de la contingencia

**Objetivo:** mostrar cuánto afecta la contingencia al desplazamiento del usuario: viaje habitual vs. mejor alternativa, tiempo adicional estimado e indicador de impacto.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Alto | Media | MVP (RF-10…RF-14) + trayecto habitual (PM-03) |

**Pasos:**

1. Obtener el tiempo estimado del recorrido habitual (mismo método de estimación del MVP, con sus parámetros).
2. Obtener el tiempo estimado de la mejor alternativa disponible bajo la contingencia vigente.
3. Presentar la diferencia como **comparación entre dos estimaciones**, con la nota de que no es una medición en tiempo real (ejemplo ilustrativo de la conversación: habitual 28 min, alternativa 42 min, +14 min).
4. Indicador de impacto por viaje (sin afectación / baja / media / alta) calculado por reglas documentadas a partir de esa diferencia; reglas en `docs/dominio.md`.

### PM-07 — Replanificación dinámica del viaje

**Objetivo:** si aparece una nueva contingencia mientras el usuario está en viaje, el sistema recalcula y le ofrece una alternativa que evita la nueva afectación.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Alto | Alta | MVP + PM-01 (seguimiento) |

**Pasos:**

1. Punto de partida simple (recomendación de la conversación): no rastrear continuamente la ubicación. Actualizar las alternativas cuando cambia una contingencia y permitir al usuario solicitar una nueva planificación ("volver a planificar").
2. Si el recorrido en curso usa un segmento que quedó afectado, recalcular desde la posición de referencia más reciente disponible y emitir el mensaje con contexto ("tu recorrido ha cambiado: encontramos una alternativa que evita la nueva contingencia").
3. Mantener la sesión del viaje y el historial de alternativas consideradas para explicar el cambio.
4. Prueba de escenario simulada: contingencia nueva en mitad del recorrido (la ruta anterior deja de ser conveniente) → el sistema propone una alternativa válida.

**Riesgos:** no confundir replanificación con seguimiento en tiempo real (excluido por `AGENTS.md` §2).

### PM-08 — Comparación con información de incertidumbre

**Objetivo:** cada alternativa explica qué tan confiable es su recomendación: fuente, última actualización, método de estimación, información faltante y nivel de confianza.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Alto | Media | MVP + metadatos de estimación (`docs/arquitectura.md` §7, regla 4) |

**Pasos:**

1. Reutilizar los metadatos obligatorios de cada estimación del MVP (método: `SCHEDULED` o `ESTIMATED`; fecha de última actualización de los datos; nivel de confianza `HIGH`/`MEDIUM`/`LOW` derivado de reglas documentadas).
2. Añadir por alternativa: fuente de cada conjunto de datos, fecha de última actualización, información faltante (geometrías, horarios, paraderos sin coordenadas válidas).
3. Vista de comparación que incluya esa columna de información junto a tiempo y características.
4. Prueba: una alternativa basada en datos incompletos se presenta con su nivel de confianza y su información faltante, nunca como equivalente a una basada en datos oficiales.

### PM-09 — Mantenimientos programados: calendario y alertas anticipadas

**Objetivo:** aprovechar que parte de las afectaciones se conocen con anticipación: calendario de mantenimientos, cierres temporales y modificaciones operativas, con aviso previo al usuario.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Muy alto | Media | MVP (F01) + PM-03 (notificaciones) |

**Pasos:**

1. Modelo de mantenimiento programado: fecha de inicio y fin, estaciones/conexiones afectadas, tipo de afectación, fuente oficial.
2. Fuente de datos de mantenimientos: investigar primero (INV-31 del plan de estudio); sin fuente verificable para las fechas, no se promete el aviso anticipado.
3. Calendario consultable (qué está programado, cuándo empieza y termina, qué afecta, qué alternativas recomienda el sistema para ese periodo).
4. Alertas anticipadas: notificación previa a usuarios cuyo recorrido habitual se verá afectado, siempre que la fuente oficial permita conocer las fechas (ejemplo de la conversación: "el próximo sábado se realizará un mantenimiento que afectará tu recorrido habitual").
5. Relación con las contingencias: **PC-06 resuelto**: el mantenimiento es una entidad separada (`Maintenance`) con estados propios; no es un cuarto estado de alerta. Al entrar en curso puede crear o vincularse a una alerta.

### PM-10 — Reportes ciudadanos con moderación

**Objetivo:** los usuarios observan situaciones no registradas aún (paradero fuera de servicio, fila inusualmente larga, conexión peatonal cerrada, bus que no realiza su recorrido, dificultad de acceso) y las reportan; el sistema los valida antes de publicarlos.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Alto | Media-alta | MVP + PM-01 (seguimiento) |

**Pasos:**

1. Formulario de reporte: tipo de reporte (catálogo cerrado), ubicación (con permiso, reglas de `AGENTS.md` §9), descripción opcional. El ejemplo de payload de la conversación usa tipos de código (`BUS_STOP_UNAVAILABLE`), latitud/longitud y fecha de creación: el contrato concreto se define en `docs/api.md`.
2. Un reporte **no** se convierte automáticamente en verdad oficial: pasa por validación.
3. Estados de verificación (propuesta de la conversación): `Pending` (recibido, sin verificar), `Under review` (en revisión o con corroboración pendiente), `Verified` (corroborado por fuentes confiables). Los estados en inglés serían nombres de código y el idioma final depende de la decisión de idioma de `AGENTS.md` §13.
4. Corroboración por coincidencia: número de reportes similares en el mismo lugar como señal interna para revisión. No publicar automáticamente una afectación crítica solo porque varios usuarios la reportaron (regla explícita de la conversación).
5. Fecha de expiración, fuente del reporte, historial de modificaciones y moderación administrativa.
6. Pruebas: reporte aislado → no genera contingencia activa; reporte corroborado por fuentes confiables → puede convertirse en contingencia según flujo del MVP (verificación obligatoria).

**Riesgos:** abuso y spam (anti-abuso), dependencia de una comunidad activa para que el valor se materialice, y la regla de negocio de `AGENTS.md` §6: una publicación no es automáticamente una contingencia activa — un reporte menos que una publicación.

### PM-11 — Instrucciones paso a paso del recorrido

**Objetivo:** explicar en lenguaje natural cada paso del recorrido: dónde caminar, dónde abordar, dónde transbordar, dónde descender.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Alto | Media | MVP (RF-11: secuencia de segmentos) |

**Pasos:**

1. Transformar la secuencia de segmentos (medio, estaciones/paraderos, transbordos) en texto guiado, sin tecnicismos (reglas de `AGENTS.md` §11).
2. Reordenar, no calcular: las instrucciones son presentación del resultado del motor; el orden lo define la ruta calculada.
3. Estados de error/vacío y lenguaje consistentemente con el resto de la PWA.

### PM-12 — Retroalimentación del viaje

**Objetivo:** al terminar el viaje, el usuario informa si la alternativa fue útil y si el tiempo estimado se acercó al tiempo real observado. Esa información calibra las estimaciones del MVP (CA-03 de `requerimientos.md`).

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Alto | Baja-media | MVP (RF-13/RF-14) |

**Pasos:**

1. Pregunta posterior al viaje (utilidad sí/no; tiempo real observado opcional).
2. Separar la retroalimentación de las consultas: honestidad en el uso de los datos (las consultas de la app no son volumen real de pasajeros; ver PM-18).
3. Uso: ajuste de parámetros de estimación por medición, con registro de la fuente (origen: retroalimentación de usuarios).

### PM-13 — Compartir el estado de una contingencia

**Objetivo:** evolucionar F05: compartir no solo la ruta, sino el estado del servicio y el contexto de la recomendación.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Alto | Baja | MVP (RF-15) |

**Pasos:**

1. Formato de mensaje con: estado de la contingencia (incl. severidad/clasificación si existe), origen, destino, medios con estaciones/paraderos, tiempo marcado como aproximado y última actualización.
2. Mecanismo de envío: cierra CA-02 de `requerimientos.md` (enlace universal vs. API oficial).

### PM-14 — Consulta con conectividad limitada (modo offline)

**Objetivo:** consultar la última información descargada, mapas básicos y rutas guardadas, mostrando claramente que los datos pueden estar desactualizados.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Alto | Media | MVP (PWA, RNF-10) |

**Pasos:**

1. Determinar qué se cachea (recursos estáticos, última red conocida, rutas guardadas) y con qué fecha de actualización, que siempre se muestra (`AGENTS.md` §11: la PWA no garantiza alertas en tiempo real sin conexión).
2. Diferenciación visual confirmada/estimada/no disponible ya presente en el MVP (RF-16) aplicada al estado offline.
3. La información disponible almacenada localmente siempre lleva su fecha de actualización.

### PM-15 — Multilenguaje (español e inglés)

**Objetivo:** español e inglés inicialmente, útil para visitantes y turistas que desconocen el sistema.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Alto | Baja-media | MVP (solo el idioma de la interfaz y de los mensajes; la documentación del proyecto sigue en español por `AGENTS.md` §13) |

**Pasos:**

1. Identificar todo el texto visible de la PWA (incluido el mensaje compartible y las notificaciones).
2. Centralizar las cadenas de texto (sin textos embebidos en componentes) y definir la convención de nombres de claves.
3. Traducción y revisión del contenido, con los textos de tiempos y estados con la misma redacción de cautela que en español ("aproximado", "estimado").

---

## 3. V3 — Plataforma institucional ("MetroAlerta Control")

La conversación propone dos productos conectados con el mismo backend y permisos distintos: **MetroAlerta Passenger** (V2) y **MetroAlerta Control** (V3, para operadores y entidades).

### PM-16 — Dashboard de contingencias

**Objetivo:** panel administrativo para Metro/AMVA: contingencias activas, historial de interrupciones, duración de las afectaciones, líneas y estaciones más afectadas, consultas de rutas, alternativas más solicitadas y tiempos adicionales estimados, con gráficos y mapas.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Muy alto | Media-alta | MVP + PM-01/PM-02 |

**Pasos:**

1. Escuchar el historial de contingencias del MVP (eventos de PM-01) como fuente del dashboard.
2. Definir cada indicador con su fuente exacta: datos operativos oficiales vs. datos de uso de la aplicación vs. estimaciones del sistema vs. reportes ciudadanos (distinción obligatoria de la conversación).
3. Visualizaciones por indicador (tablas y gráficos).
4. **Regla de honestidad:** las consultas de la app no representan la totalidad de pasajeros afectados; no presentarlas como volumen real de pasajeros del Metro.

### PM-17 — Mapa de impacto

**Objetivo:** seleccionar una contingencia y ver visualmente estaciones afectadas, conexiones suspendidas, paraderos cercanos, alternativas disponibles y zonas con menor conectividad. Demuestra PostGIS y grafos: identificar nodos aislados y estaciones con menos conexiones disponibles.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Muy alto | Media-alta | MVP + PM-02 |

**Pasos:**

1. Representar el grafo de transporte sobre el mapa con el estado de vigencia de cada arista.
2. Consultas de conectividad (nodos que pierden conectividad, estaciones con menos conexiones), apoyadas en PostGIS si el ADR-005 (modelo de datos) lo justifica.
3. Capas visuales por categoría (estaciones, conexiones, paraderos, alternativas, zonas con menor conectividad).
4. Consistente con el resto del producto: los datos desactualizados se marcan como tales.

### PM-18 — Análisis histórico de contingencias

**Objetivo:** responder preguntas como: ¿qué líneas presentan más interrupciones registradas? ¿cuánto duran las contingencias? ¿qué estaciones aparecen frecuentemente involucradas? ¿qué alternativas se consultan más? ¿en qué periodos se producen más consultas?

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Muy alto | Media-alta | PM-16 |

**Pasos:**

1. Tablas de agregación sobre el historial de contingencias y el registro de consultas (anonimizadas; ver `AGENTS.md` §9: no registrar información sensible innecesaria).
2. Tres familias de indicadores, claramente separadas en la presentación: datos operativos oficiales, datos de uso de la app, estimaciones del sistema y reportes ciudadanos.
3. Ventanas temporales y filtros por línea/estación.
4. Cada indicador lleva su metodología y población de referencia (regla de honestidad de `AGENTS.md` §1).

### PM-19 — Simulador de escenarios de afectación

**Objetivo:** laboratorio de simulación de la red: modificar temporalmente las condiciones (una estación fuera de servicio, una conexión suspendida, varias conexiones cerradas, una ruta de bus fuera de operación, tiempos de espera incrementados) y analizar qué sucede, sin tocar la red real.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Muy alto | Alta | MVP: Route Engine + PM-09 (mantenimientos futuros) + PC-06 (modelo de mantenimientos) |

**Pasos:**

1. El motor recibe una **copia lógica** (lógica, no física) de la red y aplica las restricciones temporales del escenario (estaciones, conexiones, fechas de inicio y fin).
2. Calcular sobre la copia: rutas que siguen disponibles, pares origen-destino sin conexión, tiempo adicional estimado, cantidad de alternativas, nodos que pierden conectividad.
3. Para el caso de mantenimientos futuros (conversación §4.3): administrador selecciona línea, estaciones, conexiones, fechas → la plataforma muestra qué alternativas tendrían los pasajeros, estaciones con mayor dependencia de las conexiones afectadas y sectores que podrían necesitar mayor información. La conversación lo marca como **gran oportunidad B2B**.
4. Los escenarios simulados quedan separados de las contingencias reales y se marcan como simulación en toda la presentación.
5. **Delimitación de la conversación:** esto es un *simulador de escenarios de afectación de la red de transporte*, no un gemelo digital completo. Solo evolucionaría hacia un gemelo digital con datos operativos, históricos y en tiempo real.
6. Demuestra sin IA: teoría de grafos, caminos mínimos, conectividad, SIG, simulación, optimización, procesamiento y visualización.

**Riesgos:** la simulación no debe presentarse como si hubiera funcionado en operación real (misma advertencia que PM-01).

### PM-20 — API institucional

**Objetivo:** API pública/institucional con autenticación, permisos, límites de consumo y documentación OpenAPI, para que otras aplicaciones consulten MetroAlerta con acuerdo de integración y condiciones de acceso.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Muy alto | Media-alta | V3 completada (PM-16…PM-18) |

**Pasos:**

1. Definir el contrato de la API en `docs/api.md` (hoy pendiente). La conversación ofrece **ejemplos propuestos, no existentes**: `GET /api/v1/incidents`, `GET /api/v1/incidents/active`, `GET /api/v1/transport/network`, `GET /api/v1/routes/alternatives`, `GET /api/v1/maintenance/events`.
2. Formalizar la relación con el ecosistema (conversación §9): Metro (estado del sistema), operadores de buses (rutas y servicios), AMVA (datos territoriales), ciudadanos (reportes y observaciones) → MetroAlerta (normalización, validación, integración, análisis) → consumidores: pasajeros (PWA) y entidades (dashboard y API).
3. Autenticación, permisos y límites de consumo por clave/consumidor; acuerdos de integración documentados.
4. **Regla del proyecto:** las políticas de uso de cada fuente externa (ver INV-08 del plan de estudio) quedan primero confirmadas; la API de MetroAlerta no puede ofrecer lo que no tiene derecho a redistribuir.

### PM-21 — Panel de calidad de datos (Data Health)

**Objetivo:** demostrar madurez de ingeniería y evitar que el sistema recomiende rutas con información incompleta sin advertirlo: fecha de última actualización por conjunto de datos, fuente de procedencia, porcentaje de registros completos, conexiones sin geometría, paraderos sin coordenadas válidas, rutas sin información de horarios, duplicados y fuentes que dejaron de responder.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Alto | Media | MVP + ingesta (o registro manual de fuentes) |

**Pasos:**

1. Registrar por conjunto de datos: fuente, última actualización y controles de completitud (los indicadores de la lista anterior).
2. Panel interno con los indicadores, con valores reales medidos (los números de la conversación son ficticios, ilustrativos).
3. Alertas internas cuando una fuente deja de responder (fallback ya presente en RNF-09 del MVP: la app se degrada, no cae).
4. Mostrar al usuario, cuando aplique, la advertencia de que una recomendación se basa en datos incompletos (conecta con PM-08).

### PM-22 — Gestión de fuentes e integración con operadores

**Objetivo:** plataforma que integra diferentes fuentes de información (Metro, operadores de buses, AMVA, ciudadanos) sin que cada proveedor arrastre su formato al dominio.

| Valor | Dificultad | Prerrequisito |
|---|---|---|
| Alto | Media-alta | MVP + decisión de ingesta (ADR-002: Python vs n8n) |

**Pasos:**

1. Módulo de ingestión de datos (`Data Ingestion`) con la responsabilidad de normalizar, validar y registrar información externa (conversación §12 y `AGENTS.md` §5: la ingesta es un proceso separado).
2. Un adaptador por fuente: cada proveedor mantiene su formato en su adaptador; el dominio solo conoce el modelo interno.
3. Gestión de fuentes: activas, condiciones de uso, última sincronización (conecta con PM-21).
4. Firmar acuerdos de integración antes de publicar cualquier dato de terceros (regla de `AGENTS.md` §7 sobre condiciones de uso y redistribución).

---

## 4. V4 — Investigación futura

Ninguna de estas entra al MVP ni a la V3 sin aprobación explícita y justificación técnica; varias tocan las exclusiones de `AGENTS.md` §2 y requieren aprobación diferenciada.

### PM-23 — Predicción de demanda de alternativas

Estimar qué alternativas podrían recibir mayor interés ante una contingencia, basándose en patrones históricos de uso. Advertencia de la conversación: **esto no significa conocer cuántas personas subirán a cada bus**; para ocupación real se necesitarían datos adicionales de los operadores. Dificultad: muy alta.

### PM-24 — Información operativa en tiempo real y analítica predictiva

Integración con información operativa en tiempo real, analítica predictiva y modelos avanzados de estimación. **Frontera con `AGENTS.md` §2:** la predicción de fallas del Metro y de congestión vial, el seguimiento GPS de buses en tiempo real y cualquier IA predictiva están excluidos del MVP; entrar aquí exige aprobación explícita y un ADR que justifique la excepción.

### PM-25 — Simulación de escenarios complejos e integración de nuevos operadores

Escenarios compuestos (varias restricciones simultáneas con parámetros cambiantes) y la incorporación progresiva de nuevos operadores e instituciones. Dificultad: muy alta.

---

## 5. Cómo encaja esto en la arquitectura

La conversación (§12) mantiene el monolito modular y añade progresivamente estos módulos al backend (mapean los PM de arriba):

| Módulo nuevo | Responsabilidad | Funcionalidades PM |
|---|---|---|
| `maintenance` | Mantenimientos y cierres programados | PM-09 (y PM-19 en lo que refiere a escenarios futuros) |
| `notifications` | Suscripciones, trayectos favoritos y notificaciones | PM-03, PM-09 (alertas anticipadas) |
| `accessibility` | Restricciones y características de accesibilidad | PM-05 |
| `citizen-reports` | Reportes y moderación | PM-10 |
| `analytics` | Indicadores y estadísticas | PM-16, PM-17, PM-18 |
| `simulation` | Escenarios hipotéticos de afectación | PM-19 |
| `administration` | Gestión de usuarios administrativos y permisos | PM-16…PM-21 (y CA-01 del MVP) |
| `data-ingestion` | Normalización y validación de fuentes externas | PM-21, PM-22 |

Reglas arquitectónicas de la conversación, compatibles con `AGENTS.md` §5:

- `analytics` no consulta directamente las tablas internas de todos los módulos: consume contratos de los módulos.
- `simulation` no modifica la red real de transporte: opera sobre una copia lógica separada de las contingencias reales.
- Un módulo `users` para pasajeros sigue sin crearse hasta que exista la necesidad real que lo justifique (decisión pendiente ligada a CA-01/PM-03).

---

## 6. Orden de trabajo recomendado entre versiones

Siguiendo la Etapa 2–4 de la conversación (§15):

1. **Etapa 1 — Núcleo (MVP):** requisitos, arquitectura, fuentes de datos, grafo, Route Engine, contingencias, mapa, alternativas. (Equivale al alcance actual del proyecto.)
2. **Etapa 2 — Diferenciar el producto (V2):** mantenimientos programados (PM-09), alertas personalizadas (PM-03), comparación de alternativas (PM-04, PM-06, PM-08), rutas accesibles (PM-05), seguimiento de contingencias (PM-01, PM-02).
3. **Etapa 3 — Valor institucional (V3):** dashboard de operaciones (PM-16), mapa de impacto (PM-17), analítica histórica (PM-18), simulación de escenarios (PM-19), API para integraciones (PM-20), calidad de datos (PM-21) y gestión de fuentes (PM-22).
4. **Etapa 4 — Funcionalidades avanzadas (V4):** demanda de transporte (PM-23), predicción (PM-24), información operativa en tiempo real (PM-24), modelos avanzados de estimación (PM-24, PM-25).

Consejo de la conversación: no intentar implementar todo; construir una base sólida y que cada funcionalidad nueva tenga una justificación dentro del producto. Todo paso de una versión a la siguiente exige aprobación explícita del desarrollador y, cuando aplique, ADR correspondiente.

---

## 7. Cuestiones abiertas de este documento

| ID | Cuestión | Estado |
|---|---|---|
| PC-01 | Alcance exacto de V2 después del MVP: ¿cuáles PM se aprueban y en qué orden? | Por decidir (propone ADR) |
| PC-02 | Modelo de clientes para pasajeros (anónimo vs. con cuenta), requerido por PM-03 y PM-12; depende de CA-01 de `requerimientos.md`. | Por decidir |
| PC-03 | Fuente de datos de accesibilidad (PM-05) y de mantenimientos (PM-09): sin fuente verificada no se construyen. | INV-31 e INV-33 del plan de estudio |
| PC-04 | Contrato de la API institucional (PM-20): los endpoints de la conversación son ejemplos propuestos, no existentes. | Por definir en `docs/api.md` |
| PC-05 | Posición de V4 frente a las exclusiones de `AGENTS.md` §2: aprobación y ADR dedicados. | Por decidir |
| PC-06 | Modelo de estados de los mantenimientos programados. | **Resuelto:** entidad separada `Maintenance`; no es un cuarto estado de alerta. Ver `docs/dominio.md` y `docs/modelo-de-datos.md`. |
