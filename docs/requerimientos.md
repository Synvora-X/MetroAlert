# Requerimientos — MetroAlert

| Campo | Valor |
|---|---|
| Estado | creado |
| Fecha | 2026-11-05 |
| Fuentes | `AGENTS.md` (marco) + conversación "Plan de MetroAlert" (ChatGPT, 2026-11-03, <https://chatgpt.com/share/6aea752c-6ae0-83e9-8e29-6135665fc569>) |
| Relación con AGENTS.md | Las funcionalidades F01–F05 (sección 2 de `AGENTS.md`) son la versión resumida del alcance. Este documento las descompone en RF verificables. Si este documento contradice `AGENTS.md`, manda `AGENTS.md`. |
| Trazabilidad | Matriz F01–F05 → RF → HU → RNF → INV → PM en `docs/trazabilidad.md`. |
| Documentos derivados | `docs/dominio.md` (reglas), `docs/modelo-de-datos.md` (persistencia) y `docs/api.md` (contrato HTTP). |

**Convenciones:**
- **Prioridad:** Alta = el MVP no funciona sin esto. Media = esperable, pero el MVP se puede entregar sin esto si se justifica.
- **Criterios de aceptación:** en formato GWT (*Dado… / Cuando… / Entonces…*), escritos para poder convertirse en pruebas automatizadas (unitarias o E2E).
- No hay RF por funcionalidad futura: lo post-MVP se lista en la sección 4 de este documento.

---

## 1. Requerimientos funcionales (RF)

### F01 — Ingesta y gestión de alertas

| ID | Requerimiento | Prioridad |
|---|---|---|
| RF-01 | El sistema registra y actualiza alertas de forma manual y desde fuentes externas. Cada alerta guarda, **cuando exista el dato**: tipo, línea(s), estación(es) afectada(s), descripción, fecha de publicación, fuente y fecha-hora de última actualización. Los campos que no existan se dejan sin valor; **no se inventan**. | Alta |
| RF-02 | Toda alerta tiene exactamente uno de tres estados: **activa**, **resuelta** o **vigencia indeterminada**. El estado se persiste y se muestra siempre. | Alta |
| RF-03 | Una alerta se presenta como contingencia activa solo cuando está verificada. El sistema **no** interpreta automáticamente una publicación como contingencia confirmada. | Alta |
| RF-04 | El frontend puede consultar las contingencias actuales con su fuente, fecha de publicación y última actualización. | Alta |

### F02 — Mapa

| ID | Requerimiento | Prioridad |
|---|---|---|
| RF-05 | El mapa muestra la red del Metro (líneas y estaciones) con Leaflet + OpenStreetMap, incluyendo la atribución requerida. | Alta |
| RF-06 | El mapa muestra paraderos y rutas integradas **solo** cuando existan datos válidos. Si no hay datos, muestra un estado vacío explícito (nunca espacio en blanco ni datos sin identificar). | Media |
| RF-07 | El mapa muestra las contingencias vigentes y el recorrido de la alternativa seleccionada. | Alta |
| RF-08 | El sistema solicita la ubicación del usuario solo con permiso explícito, la usa únicamente donde la requiere (p. ej. como origen del cálculo) y **no la persiste** en la base de datos salvo decisión posterior documentada. | Alta |

### F03 — Rutas alternativas

| ID | Requerimiento | Prioridad |
|---|---|---|
| RF-09 | El sistema captura origen y destino del usuario. | Alta |
| RF-10 | Dado origen, destino y condiciones vigentes, el sistema calcula **hasta 3** alternativas que **evitan los segmentos afectados por contingencias activas confirmadas**. | Alta |
| RF-11 | Cada alternativa se describe como secuencia de segmentos con: medio (Metro, bus integrado/alimentador, caminata), estaciones/paraderos implicados, transbordos y orden de los pasos. | Alta |
| RF-12 | Si no hay datos suficientes para construir una alternativa válida, el sistema no la genera y responde explícitamente que no hay alternativas con los datos disponibles. | Alta |

### F04 — Tiempo estimado

| ID | Requerimiento | Prioridad |
|---|---|---|
| RF-13 | El tiempo total estimado de una alternativa se calcula como suma de: tiempo de caminata + tiempo de espera + tiempo de recorrido + tiempo de transbordos, usando **parámetros configurables e identificables** (velocidades, tiempos de espera/transbordo). Ningún valor mágico en el código. | Alta |
| RF-14 | El tiempo se presenta como **aproximación**, con el método de cálculo y la vigencia de los datos que lo sustentan. Nunca se presenta como tiempo garantizado ni "en tiempo real". | Alta |

### F05 — Compartir por WhatsApp

| ID | Requerimiento | Prioridad |
|---|---|---|
| RF-15 | El usuario puede compartir una alternativa por WhatsApp. El mensaje incluye: origen, destino, medios con estaciones/paraderos, tiempo estimado (marcado como aproximado) y contingencia relacionada. | Media |

### Transversal

| ID | Requerimiento | Prioridad |
|---|---|---|
| RF-16 | En toda la interfaz, cada dato se presenta diferenciado entre **confirmado**, **estimado** y **no disponible**, e indica cuándo se actualizó por última vez. | Alta |

---

## 2. Historias de usuario y criterios de aceptación

### HU-01 — Consultar contingencias

Como pasajero, quiero consultar las contingencias activas para saber si mi desplazamiento puede verse afectado.

**Criterios de aceptación:**
- Dada una alerta registrada con solo línea y descripción, cuando se guarda, entonces el sistema conserva esos campos y deja los demás sin valor (sin datos inventados).
- Dada una alerta publicada por una fuente, cuando no ha sido verificada, entonces **no** se muestra como contingencia activa.
- Dada una contingencia confirmada, cuando el usuario abre la pantalla principal, entonces ve sus datos esenciales (qué, dónde, cuándo se publicó y cuándo se actualizó).
- Dado que no hay contingencias activas, cuando el usuario abre la pantalla, entonces el sistema muestra un estado vacío explícito ("sin contingencias activas") con la fecha de la última consulta.

### HU-02 — Calcular alternativas

Como pasajero, quiero ingresar mi origen y destino para conocer alternativas de desplazamiento durante una contingencia.

**Criterios de aceptación:**
- Dada una contingencia activa, cuando el usuario envía origen y destino válidos, entonces recibe hasta 3 alternativas, cada una con segmentos y tiempo estimado.
- Dado que no hay datos suficientes para ninguna alternativa, cuando el usuario envía origen y destino, entonces recibe un mensaje explícito de "sin alternativas con los datos disponibles" (sin fabricar rutas).
- Dado un origen o destino inválido o fuera de rango, cuando se envían, entonces el sistema responde con un error controlado, sin exponer detalles internos.

### HU-03 — Evitar segmentos afectados

Como pasajero, quiero que las alternativas eviten los segmentos afectados para no recibir rutas que no puedo usar.

**Criterios de aceptación:**
- Dada una contingencia activa que afecta un segmento, cuando se calculan alternativas cuyo recorrido lo atravesaría, entonces ninguna alternativa presentada usa ese segmento.
- Dado que la contingencia se resuelve, cuando se calculan alternativas nuevas, entonces el segmento vuelve a estar disponible para los cálculos.
- Dada una contingencia en "vigencia indeterminada" para un segmento, cuando se presentan alternativas, la información se etiqueta según su nivel de confirmación (no como confirmada si no lo está).

### HU-04 — Comparar alternativas

Como pasajero, quiero comparar las alternativas para elegir la que mejor se adapta a mi situación.

**Criterios de aceptación:**
- Dadas 2 o más alternativas generadas, cuando se muestran, entonces el usuario ve por cada una: tiempo total estimado, transbordos, caminatas y el medio de cada segmento.
- Dada solo una alternativa viable, cuando se muestra, entonces va acompañada de la explicación de que no hay más con los datos disponibles.

### HU-05 — Compartir por WhatsApp

Como pasajero, quiero compartir una alternativa por WhatsApp para informar a otras personas cómo llegar.

**Criterios de aceptación:**
- Dada una alternativa visible, cuando el usuario elige compartir, entonces el mensaje incluye origen, destino, medios con estaciones/paraderos, tiempo estimado (marcado como aproximado) y la contingencia relacionada.
- Dado que el usuario comparte una alternativa, cuando no tiene cuenta ni sesión iniciada, entonces el enlace o mensaje generado funciona sin autenticación en MetroAlert.

### HU-06 — Confiar en la información

Como pasajero, quiero saber qué tan confiables y recientes son los datos que veo.

**Criterios de aceptación:**
- Dado cualquier dato mostrado (red, contingencias, tiempos), cuando se presenta en pantalla, entonces se indica la última actualización y se distingue entre confirmado / estimado / no disponible.
- Dado un dato desactualizado o fuera de validez, cuando se muestra, entonces la interfaz lo marca como desactualizado en lugar de presentarlo como vigente (ver RNF-11).

---

## 3. Requerimientos no funcionales (RNF)

| ID | Categoría | Requerimiento | Verificación propuesta |
|---|---|---|---|
| RNF-01 | UX | Mobile-first y responsive. La funcionalidad primaria se resuelve en un celular con una mano y poco tiempo. | Revisión visual en dispositivos / pruebas en varios tamaños |
| RNF-02 | Seguridad | Todos los endpoints y formularios validan sus entradas (tipo, rangos, coordenadas y demás campos de entrada). | Pruebas con entradas inválidas y límite |
| RNF-03 | Correctitud | Las rutas nunca atraviesan segmentos afectados por contingencias activas confirmadas. | Pruebas con redes controladas (redes pequeñas con resultado conocido) |
| RNF-04 | Transparencia | Toda estimación se identifica como aproximada y con su método. | Revisión de respuestas de API y UI |
| RNF-05 | Transparencia | Toda información indica fuente (u origen) y última actualización. | Revisión de API y UI |
| RNF-06 | Errores | Los errores se manejan en la capa que corresponde y el usuario nunca ve detalles internos (stacks, SQL, claves). | Pruebas de error; revisión de mensajes |
| RNF-07 | Secretos | Los secretos viven en variables de entorno; el repo solo lleva `.env.example` sin valores. La `service role key` de Supabase nunca llega al frontend. | Revisión de repo y despliegue |
| RNF-08 | Calidad | El backend tiene pruebas automatizadas: unitarias (reglas de negocio, validaciones, cálculos), integración (servicios/repositorios cuando aplique) y E2E del flujo principal. Cubre éxito, error y límites. | Suite de pruebas en CI |
| RNF-09 | Resiliencia | Una falla en una fuente externa **no** detiene la aplicación: se degrada (con la parte que queda disponible) y se informa. | Pruebas de simulación de falla de fuente |
| RNF-10 | PWA | Manifiesto válido, instalable, service worker cuando corresponda y caché de recursos estáticos. Los datos guardados localmente siempre se muestran con su fecha de actualización. | Auditoría Lighthouse + revisión manual |
| RNF-11 | Honestidad de recomendaciones | El sistema no presenta una ruta como operativa si sus datos están desactualizados o si no puede comprobar la disponibilidad de sus conexiones. Ante duda, informa que no hay información suficiente. | Pruebas con datos antiguos / incompletos |
| RNF-12 | Accesibilidad | Contraste y jerarquía visual adecuados; lenguaje sin tecnicismos innecesarios; las contingencias activas se distinguen de un vistazo. | Revisión manual + comprobación de contraste |
| RNF-13 | Datos externos | Se identifica siempre la fuente y, cuando aplique, la fecha de obtención de cada dato. Se respeta la atribución de OpenStreetMap y su política de uso. | Revisión de pantallas y doc de fuentes |
| RNF-14 | Rendimiento | Los umbrales concretos (tiempo de cálculo, carga de página) se definen **por medición**, no por intuición; las pruebas de carga se evalúan cuando el flujo principal sea estable. | TODO(confirmar) tras medir el primer flujo |

> La definición de "funcionalidad terminada" (checklist de 8 puntos) está en `AGENTS.md` §10; este documento no la duplica.

---

## 4. Fuera del alcance del MVP

La lista completa está en la sección 2 de `AGENTS.md` (es la fuente de verdad). El desglose paso a paso de las candidatas post-MVP está en `docs/funcionalidades-post-mvp.md` y su correspondencia con las tareas de investigación en `docs/trazabilidad.md`. Funcionalidades específicas identificadas en la conversación como **candidatas post-MVP** — ninguna entra al MVP sin aprobación explícita:

1. Mantenimientos programados (calendario + alertas anticipadas).
2. Notificaciones personalizadas de contingencias (trayectos favoritos).
3. Rutas accesibles (preferencias por accesibilidad).
4. Simulación de escenarios de afectación para entidades.
5. Analítica institucional (dashboards de contingencias).
6. Reportes ciudadanos con moderación.
7. Rutas según preferencias ponderadas (menos caminata / menos transbordos).
8. Replanificación dinámica de viaje en curso.

---

## 5. Cuestiones abiertas de este documento

| ID | Cuestión | Estado |
|---|---|---|
| CA-01 | Mecanismo de gestión manual de alertas para el MVP. | **Resuelto:** panel administrativo mínimo protegido; ver `docs/dominio.md` y `docs/api.md`. |
| CA-02 | Mecanismo de envío para Compartir por WhatsApp. | **Resuelto:** enlace universal `wa.me` + Web Share API; sin API oficial en el MVP. |
| CA-03 | Valores de los parámetros de estimación (velocidad de caminata, velocidad de bus, tiempo de espera, tiempo de transbordo). Requieren fuentes validadas; cualquier valor inicial se marca como **estimado**. | TODO(confirmar) |
| CA-04 | Granularidad del mapeo contingencia → segmentos afectados. | **Resuelto:** por conexión (arista) entre nodos consecutivos; ver ADR-005 y `docs/modelo-de-datos.md`. |
| CA-05 | Umbrales de rendimiento (RNF-14). | TODO(confirmar) después de medir |
