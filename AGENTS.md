# AGENTS.md — MetroAlerta

> Contexto general del proyecto para cualquier agente de IA. No está orientado a una capa en particular (frontend, backend, ingesta): describe qué es MetroAlerta, qué reglas rigen todo el sistema y cómo se trabaja en él.
>
> - Si algo de este archivo contradice el código real, **el código manda**: señala la contradicción antes de hacer cambios grandes.
> - `TODO(confirmar)` marca datos que el desarrollador aún debe definir. No los inventes ni los des por hechos.
> - Este archivo no registra progreso ni tareas: el estado actual, las tareas y las decisiones recientes viven en `MEMORY.md` (ver sección 14). Para saber qué está implementado de verdad, inspecciona además el repositorio.

**Jerarquía de fuentes (si hay conflicto):**
1. El código y la configuración real del repositorio.
2. Este archivo (`AGENTS.md`).
3. Los documentos de `/docs` (cada uno manda sobre su materia; ver tabla de la sección 13).
4. `MEMORY.md` (estado temporal).
5. La conversación "Plan de MetroAlerta" (contexto histórico; no es fuente de verdad).

Toda contradicción se señala antes de actuar; no se silencia. **Estados de madurez:** `propuesta` (sin aprobar), `decisión` (ADR o aprobación explícita) e `implementado` (verificado en el código). **Lectura sugerida:** `AGENTS.md` → `MEMORY.md` → documento dueño del tema (sección 13).

---

## 1. Qué es MetroAlerta

PWA para el Valle de Aburrá (Antioquia, Colombia). Cuando hay una contingencia en el Metro de Medellín (interrupción, suspensión, cierre de estación, retraso), muestra qué está afectado y propone **hasta 3 rutas alternativas** con tiempo estimado.

- **Problema que resuelve:** saber que hay una contingencia no dice qué hacer para continuar el viaje. MetroAlerta conecta "hay una contingencia" con "qué hago ahora".
- **No reemplaza** a las apps de navegación: es una herramienta especializada para contingencias.
- **Usuarios:** personas que usan el transporte público a diario (estudiantes, trabajadores), desde el celular, con poco tiempo para decidir.
- **Propósito:** proyecto de portafolio y prototipo funcional para validar la idea con posibles aliados (Metro de Medellín, Área Metropolitana del Valle de Aburrá, otras entidades de movilidad).
- Las ideas comerciales (licenciamiento, piloto, soporte) son **hipótesis**: no hay precios aprobados, contratos ni autorizaciones institucionales. Las decisiones comerciales no deben alterar la arquitectura ni ampliar el alcance.

Las afirmaciones sobre el problema (dependencia del Metro, tiempo perdido buscando alternativas, información dispersa) son **hipótesis por validar**. No presentes cifras o porcentajes de impacto como oficiales sin fuente, fecha, metodología y población de referencia.

## 2. Alcance del MVP

Objetivo: demostrar el flujo completo (identificar contingencia → mostrar afectación → ofrecer hasta 3 alternativas), aunque inicialmente use datos simulados o recopilados previamente.

| Id | Funcionalidad | Resumen |
|---|---|---|
| F01 | Ingesta y gestión de alertas | Obtener alertas de fuentes verificables. Cada alerta guarda, cuando exista el dato: tipo, línea, estaciones, descripción, fecha de publicación, estado, fuente y última actualización. |
| F02 | Mapa | Leaflet + OpenStreetMap: red del Metro, estaciones, líneas, paraderos, rutas integradas (si hay datos), ubicación del usuario (con permiso), contingencias y recorridos alternativos. |
| F03 | Rutas alternativas | Dado origen, destino y contingencias vigentes, calcular hasta 3 alternativas con pasos comprensibles (Metro, buses integrados/alimentadores, caminata, transbordos). |
| F04 | Tiempo estimado | Aproximación basada en distancias, velocidades promedio configurables y tiempos de caminata/bus/transbordo. Nunca un tiempo garantizado. |
| F05 | Compartir por WhatsApp | Mensaje legible con origen, destino, ruta, medios, estaciones/paraderos, tiempo estimado y contingencia relacionada. |

**Fuera del MVP (no implementar sin aprobación explícita):**

- Predicción de fallas del Metro o de congestión vial, y cualquier IA predictiva.
- Seguimiento GPS de buses en tiempo real.
- Integración con plataformas de transporte privado.
- Apps nativas Android/iOS.
- Analítica avanzada de movilidad y modelos de optimización de tráfico.
- Notificaciones masivas avanzadas.
- Integraciones institucionales que requieran autorizaciones aún no obtenidas.
- Si quieres más información sobre futuras funcionalidades, revisa `docs/funcionalidades-post-mvp.md` (sección 1: hoja de ruta; sección 7: cuestiones abiertas).

La visión a largo plazo no es el alcance actual: ninguna funcionalidad futura entra al MVP automáticamente.

## 3. Flujo principal del producto

1. Una fuente publica información sobre una posible contingencia.
2. La ingesta la recibe, valida su estructura y procedencia, y la normaliza.
3. Se identifica la línea, estación o zona afectada (si es posible).
4. Se crea o actualiza la contingencia en la base de datos.
5. El frontend consulta el estado actualizado.
6. El usuario indica origen y destino.
7. El backend calcula alternativas y tiempos estimados.
8. Se muestran hasta 3 alternativas; el usuario elige una y puede compartirla.

## 4. Stack

| Componente | Tecnología | Versión |
|---|---|---|
| Frontend | Next.js + TypeScript | TODO(confirmar) |
| Backend | NestJS + TypeScript | 12.0.1 (verificado en `backend/package.json`) |
| Base de datos | Supabase / PostgreSQL | TODO(confirmar) |
| Mapas | Leaflet + OpenStreetMap | TODO(confirmar) |
| Ingesta | Python o n8n (por decidir) | TODO(confirmar) |
| Contenedores | Docker | TODO(confirmar) |
| CI/CD | GitHub Actions | CI configurado en `.github/workflows/ci.yml`; CD pendiente |
| Despliegue frontend | Vercel | — |

Que una tecnología aparezca aquí **no significa que esté configurada o implementada**. Verifícalo en el repositorio antes de usarla.

Versiones verificadas en el backend: NestJS 12.0.1, TypeScript 6.0.2, pnpm 11.6.0, Vitest 4.1.2, oxlint 1.58.0, prettier 3.4.2 y Supertest 7.0.0. El CI está configurado; CD sigue `TODO(confirmar)`. Las versiones del frontend, base de datos, ingesta y Docker siguen `TODO(confirmar)`.

**Decisiones abiertas** (no las tomes por tu cuenta; proponlas como ADR):

- Python vs n8n para la ingesta.
- Dónde se aloja el backend y el servicio de ingesta.
- Proveedor de tiles del mapa para producción (el servidor público de OpenStreetMap tiene una política de uso que limita el tráfico alto).
- Modelo de datos físico y migraciones (ADR-005 aceptado: PostGIS sí, pgRouting no; ver `docs/modelo-de-datos.md`).

## 5. Responsabilidad de cada componente

| Componente | Responsable de | No debe |
|---|---|---|
| Frontend | UI, navegación, mapa, selección de origen/destino, mostrar alternativas, compartir, capacidades PWA, estados de carga/error/vacío | Calcular rutas ni contener reglas de negocio centrales |
| Backend | Endpoints, reglas de negocio, cálculo de rutas y tiempos, gestión de alertas, validación y seguridad | Poner lógica de negocio en controladores |
| Base de datos | Persistir alertas, contingencias, líneas, estaciones, paraderos, rutas, segmentos y parámetros de estimación | Tener tablas o campos sin una necesidad de dominio clara |
| Ingesta | Consultar fuentes, normalizar, validar, evitar duplicados, detectar cambios de estado, registrar errores | Mezclarse con la presentación, o forzar cambios en el resto del sistema cuando cambia una fuente |

Principios de arquitectura que sí cambian cómo trabajas:

- Cambiar de fuente de información no debe obligar a modificar el dominio ni la UI.
- Una falla en una fuente externa **no** debe tumbar la aplicación: se degrada y se informa.
- El dominio no depende de proveedores concretos (Supabase, n8n, etc.).
- No introduzcas microservicios, colas ni brokers sin una justificación técnica concreta. El servicio de ingesta es un proceso separado, no una arquitectura de microservicios.

## 6. Dominio y reglas de negocio

**Glosario**

- **Línea:** infraestructura y estaciones que conforman un servicio de transporte.
- **Estación:** punto de acceso o conexión del Metro.
- **Paradero:** lugar donde se aborda o desciende de un servicio.
- **Ruta de transporte:** recorrido que conecta puntos con un medio de transporte.
- **Alerta:** comunicación sobre una situación relevante para la movilidad.
- **Contingencia:** situación que altera parcial o totalmente un servicio.
- **Origen / Destino:** inicio y fin del desplazamiento del usuario.
- **Ruta alternativa:** secuencia de segmentos que permite llegar al destino evitando una afectación o usando otros medios.
- **Transbordo:** cambio de servicio o medio durante el viaje.
- **Tiempo estimado:** aproximación del tiempo para completar una ruta.

**Reglas**

- Una alerta **no** es automáticamente una contingencia activa. No interpretes cualquier publicación como contingencia confirmada.
- Toda alerta tiene uno de tres estados: **activa**, **resuelta** o **vigencia indeterminada**.
- Una contingencia puede afectar una o varias estaciones, o solo parte de una línea, sin suspender todo el servicio.
- Nunca recomiendes segmentos afectados según información confirmada.
- Los recorridos mantienen una secuencia geográfica coherente.
- Máximo 3 alternativas. Si no hay datos suficientes para construir una, **no la generes**: informa que no hay alternativas con los datos disponibles.
- Los tiempos se calculan con parámetros identificables y configurables (velocidades, tiempos de transbordo), nunca con valores mágicos.
- La información desactualizada se marca como tal.

## 7. Datos y fuentes externas

La disponibilidad y calidad de los datos es el principal riesgo técnico del proyecto.

**Fuentes a investigar:** Metro de Medellín, Área Metropolitana del Valle de Aburrá, portales de datos abiertos, información geográfica del sistema, OpenStreetMap, comunicados y canales oficiales (X, Instagram, Facebook...). Prioriza fuentes oficiales, abiertas y verificables.

**Reglas**

- Identifica siempre la fuente y, cuando corresponda, la fecha de obtención.
- Todo dato tiene un **origen explícito**: `oficial`, `abierto`, `estimado` o `simulado`. Nunca los mezcles ni los presentes como equivalentes.
- Que una fuente sea consultable desde un navegador no la hace pública: verifica condiciones de uso y redistribución antes de extraer datos.
- No dependas de una sola fuente sin necesidad.
- Maneja errores y datos no disponibles de forma explícita.
- La información geográfica vive separada de los componentes visuales, para actualizarla sin tocar la UI.
- Si usas OpenStreetMap, muestra la atribución requerida y respeta su política de uso.
- Toda fuente citada debe ser rastreable a un enlace o a una fuente identificable. Si el enlace no se conserva, se marca `por localizar` y no se presenta como cita.

**Datos simulados:** permitidos para desarrollo y pruebas, siempre identificados como simulados, separados de los reales, con estructura compatible con el modelo esperado y capaces de reproducir escenarios de contingencia.

## 8. Reglas de honestidad (aplican a todo)

Nunca:

- Inventes endpoints, tablas, campos, coordenadas, estaciones, paraderos, recorridos, fuentes oficiales ni métodos de librerías.
- Asumas que una API o integración existe sin comprobarlo en documentación oficial.
- Presentes estimaciones o datos simulados como datos reales.
- Afirmes que algo está terminado sin haberlo verificado.

Si falta información: investiga en documentación oficial, o pregunta cuando la decisión afecte de forma importante al diseño.

## 9. Seguridad y límites

**Siempre**

- Validar entradas del usuario y datos de fuentes externas (no confiar ciegamente en ellos).
- Mantener secretos en variables de entorno; el repositorio solo lleva `.env.example` sin valores.
- Configurar permisos de Supabase de forma explícita (RLS en tablas expuestas).
- Pedir la ubicación del usuario con su autorización y usarla solo para lo que la requiere. No almacenarla en la base de datos salvo que se decida y documente.
- Evitar registrar información sensible innecesaria; los errores al usuario no exponen detalles internos.

**Preguntar primero**

- Agregar una dependencia nueva.
- Crear migraciones o cambiar el esquema de la base de datos.
- Cambiar el contrato de la API.
- Agregar o cambiar una fuente externa de datos.
- Tocar Docker, CI/CD o configuración de despliegue.
- Eliminar código funcional o hacer refactorizaciones amplias.
- Cualquier funcionalidad fuera del MVP.

**Nunca**

- Subir secretos o `.env` con valores reales al repositorio.
- Exponer credenciales privilegiadas (por ejemplo la `service role key` de Supabase) en el frontend.
- Modificar o desactivar pruebas para que pasen, o cambiar el comportamiento esperado solo para satisfacerlas.
- Ocultar un error con una solución temporal en lugar de corregir su causa.
- Acceder a entornos de producción. TODO(confirmar) qué entornos existen y qué está permitido.

## 10. Calidad y definición de "terminado"

**Pruebas**

- Unitarias: reglas de negocio, validaciones y cálculos (estado de alertas, rutas, estimación de tiempos).
- Integración: servicios, repositorios y base de datos cuando corresponda.
- E2E: el flujo principal descrito en la sección 3.
- Carga: una contingencia grande puede disparar consultas simultáneas; se evalúa cuando el flujo principal esté estable.
- Cubre éxito, error y casos límite. No escribas pruebas solo para subir la cobertura. Usa las herramientas de testing ya configuradas.

**Una funcionalidad está terminada cuando:**

1. Cumple el requerimiento definido.
2. Respeta las responsabilidades de la sección 5.
3. Maneja los casos de error relevantes.
4. Tiene pruebas apropiadas y no introduce regresiones conocidas.
5. Usa datos reales o simulados correctamente identificados.
6. Está integrada con los componentes que corresponden.
7. Está documentada si su complejidad lo requiere.
8. Fue verificada con los comandos reales del proyecto (sección 12).

Que el código compile no es suficiente.

## 11. Experiencia de usuario, accesibilidad y PWA

El usuario probablemente está en una situación de incertidumbre y con poco tiempo.

- **Mobile-first**, responsive, contraste adecuado, jerarquía visual clara.
- Las contingencias activas se distinguen de un vistazo; lo importante se encuentra rápido y no se satura con información secundaria.
- Lenguaje sin tecnicismos innecesarios; recorridos fáciles de interpretar.
- Siempre existen estados de carga, error y ausencia de información.
- Se distingue visualmente entre información **confirmada**, **estimada** y **no disponible**, y se indica cuándo se actualizó por última vez.
- Los tiempos se muestran como aproximaciones y se aclara cuando no hay datos de tráfico ni posición de vehículos en tiempo real.

**PWA:** manifest correcto, instalable, service worker cuando corresponda y caché de recursos estáticos. La PWA **no** garantiza alertas en tiempo real sin conexión: todo dato guardado localmente se muestra con su fecha de actualización.

## 12. Comandos y estructura del repositorio

Antes de usar un comando, comprueba que existe en los scripts del proyecto. No lo supongas.

Los comandos actuales corresponden solo al backend (`backend/`), que es lo único implementado. Ejecutar desde la raíz del repositorio.

| Acción | Comando |
|---|---|
| Instalar dependencias | `cd backend && pnpm install` |
| Levantar en desarrollo | `cd backend && pnpm start:dev` |
| Pruebas | `cd backend && pnpm test` · `cd backend && pnpm test:e2e` · `cd backend && pnpm test:cov` |
| Lint / formato | `cd backend && pnpm lint` · `cd backend && pnpm format` · `cd backend && pnpm format:check` |
| Build | `cd backend && pnpm build` |
| Migraciones | TODO(confirmar) — no hay base de datos conectada. |
| Docker | TODO(confirmar) — no hay Dockerfile ni compose. |

**Estructura:** existe solo `backend/` (NestJS 12: `src/main.ts`, `src/app.module.ts`, `test/`); el frontend todavía no existe. ADR-006 acepta un monorepo con destino `apps/api`, `apps/web` y `packages/contracts`; la migración es incremental y `backend/` se mantiene hasta el PR dedicado. El backend escucha en `PORT` o 5000 por defecto.

## 13. Git y documentación

- **Commits:** siguen la convención [Conventional Commits](https://www.conventionalcommits.org/es/v1.0.0/): `tipo(alcance opcional): descripción`.
  - Un commit por funcionalidad o cambio lógico completado; no se mezclan cambios no relacionados.
  - La descripción es breve y empieza con un verbo en infinitivo (agregar, corregir, actualizar).
  - Tipos principales:
    - `feat`: nueva funcionalidad (toda funcionalidad nueva empieza con este tipo).
    - `fix`: corrección de errores.
    - `docs`: cambios en documentación.
    - `refactor`: mejora interna del código sin cambiar su comportamiento.
    - `test`: pruebas nuevas o modificadas.
    - `chore`: mantenimiento, dependencias o configuración.
    - `ci`: cambios en workflows o integración continua.
  - Ejemplo: `feat(rutas): agregar cálculo de ruta alternativa ante cierre de estación`.
- **Ramas:** una rama por tarea, con responsabilidad clara y prefijo según el tipo de cambio (`feat/`, `fix/`, `docs/`, `chore/`). Ejemplo: `feat/alertas-metro`. No se trabaja directamente sobre `main`.
- **Pull Requests:** todo cambio entra a `main` mediante PR. El workflow (CI) debe pasar antes de hacer el merge.
- **Limpieza de ramas:** después del merge, la rama del PR se elimina para mantener el repositorio ordenado.
- **`.gitignore`:** adecuado al stack del proyecto. Nunca se versionan archivos `.env`, credenciales, `node_modules/` ni artefactos de build.

- Documenta las decisiones técnicas importantes como ADR. Si una decisión cambia, actualiza los documentos afectados.
- Idioma: la documentación y la comunicación con el desarrollador son en español. TODO(confirmar) idioma de los identificadores en el código.

**Documentación de referencia (`/docs`)**

Este archivo da el contexto y las reglas esenciales; el detalle vive en `/docs`. Lee el documento que corresponda antes de trabajar en ese tema.

| Documento | Contiene | Léelo antes de... | Estado |
|---|---|---|---|
| `docs/fuentes-de-datos.md` | Fuentes investigadas, condiciones de uso, formato y fecha de obtención | Trabajar en ingesta o datos geográficos | creado |
| `docs/requerimientos.md` | Requerimientos funcionales (RF), historias de usuario (HU), no funcionales (RNF) y cuestiones abiertas (CA) | Definir o cambiar alcance | creado |
| `docs/dominio.md` | Glosario, estados y reglas de negocio ampliadas | Tocar alertas, rutas o tiempos | creado |
| `docs/arquitectura.md` | Componentes, flujo y límites entre capas | Cambios que cruzan componentes | creado |
| `docs/adr/` | Decisiones arquitectónicas, una por archivo | Proponer o revisar decisiones técnicas | creado |
| `docs/modelo-de-datos.md` | Tablas, relaciones y diagramas | Migraciones o consultas | creado |
| `docs/api.md` | Endpoints y contratos | Cambiar o consumir la API | creado |
| `docs/testing.md` | Estrategia de pruebas por componente | Escribir o cambiar pruebas | creado |
| `docs/despliegue.md` | Docker, CI/CD, entornos y variables | Tocar infraestructura | creado |
| `docs/tareas-de-investigacion.md` | Plan de estudio e investigación priorizado (tareas INV-01…INV-35, prioridades P0–P3) | Empezar trabajo de investigación o retomar el plan | creado |
| `docs/funcionalidades-post-mvp.md` | Hoja de ruta post-MVP ("MetroAlerta 2.0") desmenuzada paso a paso (PM-01…PM-25, versiones V2/V3/V4) | Diseñar o proponer funcionalidades futuras | creado |
| `docs/trazabilidad.md` | Matriz de trazabilidad F01–F05 → RF → HU → RNF → INV → PM y correspondencia INV ↔ PM | Revisar cobertura o cambiar un ID | creado |

**Documentos creados con información pendiente**

`creado` significa que el archivo existe, no que esté completo. Los documentos con datos o decisiones abiertas se listan aquí y no se presentan como terminados.

| Documento | Qué falta | Se completa cuando | Cómo completarlo |
|---|---|---|---|
| `docs/requerimientos.md` | CA-03 y CA-05; parámetros y umbrales | Se validen los parámetros y se midan los umbrales | Medir velocidades y tiempos con fuentes; ejecutar pruebas de carga; actualizar la tabla de cuestiones abiertas |
| `docs/dominio.md` | Umbrales de severidad y reglas de parámetros | Se cierren las reglas pendientes | Definir reglas y umbrales de severidad; documentar parámetros y fuentes; revisar PC-02/04/05 |
| `docs/arquitectura.md` | ADR-002/003/004 propuestos; módulos sin implementar | Se acepten los ADR y exista código | Cerrar ADR-002/003/004; implementar módulos; actualizar diagramas y estados |
| `docs/adr/` | ADR-002, ADR-003 y ADR-004 propuestos | Se acepten ingesta, alojamiento y tiles | Comparar opciones, decidir, cambiar estado a aceptado y actualizar arquitectura/trazabilidad |
| `docs/fuentes-de-datos.md` | INV-01…INV-09 sin ejecutar | Se verifiquen las fuentes y sus condiciones | Ejecutar INV-01…09; registrar URL, fecha, muestra, licencia y estado; descartar fuentes no viables |
| `docs/modelo-de-datos.md` | Modelo físico pendiente de fuentes y migraciones | Se validen muestras y se creen migraciones | Validar muestras; crear migraciones y RLS; medir índices; actualizar el ER |
| `docs/api.md` | Contrato propuesto; endpoints sin implementar | Se implementen los endpoints y se congelen decisiones | Implementar endpoints; congelar auth, paginación y versionado; generar OpenAPI y pruebas |
| `docs/despliegue.md` | CD, Docker y hosting sin decidir | Se acepten ADR-002/003 y se configure el despliegue | Cerrar ADR-002/003; definir Docker, entornos y CD; actualizar workflow y runbook |
| `docs/tareas-de-investigacion.md` | INV-01…INV-35 sin ejecutar | Se completen las asignaturas | Ejecutar asignaturas; marcar `[x]`; actualizar documentos destino |
| `docs/funcionalidades-post-mvp.md` | PC-01…PC-05 (PC-06 resuelto) | Se apruebe el alcance post-MVP | Aprobar alcance V2/V3/V4; cerrar PC-01…05; convertir PM aprobadas en tareas |
| `docs/trazabilidad.md` | Huecos de cobertura | Se implementen los RF | Implementar RF; actualizar matriz; cerrar huecos |
| `docs/testing.md` | Sin suites; CI sin pruebas | Se implementen las pruebas y se habilite CI | Crear suites unitarias/e2e; reemplazar esqueleto; habilitar tests en CI |

La columna **“Cómo completarlo”** es la ruta mínima de trabajo; no sustituye las tareas de investigación ni las decisiones registradas en los ADR.

Al crear un documento nuevo, agrégalo aquí si le falta información. Cuando se cierre su última fila, retíralo de esta tabla.

**Familias de identificadores y formato de referencias:**

| Familia | Significado | Documento dueño |
|---|---|---|
| `F0x` | Funcionalidades del MVP | `AGENTS.md` sección 2 |
| `RF-xx` | Requerimientos funcionales | `docs/requerimientos.md` |
| `RNF-xx` | Requerimientos no funcionales | `docs/requerimientos.md` |
| `HU-xx` | Historias de usuario | `docs/requerimientos.md` |
| `CA-xx` | Cuestiones abiertas del MVP | `docs/requerimientos.md` |
| `INV-xx` | Tareas de investigación | `docs/tareas-de-investigacion.md` |
| `PM-xx` | Funcionalidades post-MVP | `docs/funcionalidades-post-mvp.md` |
| `PC-xx` | Cuestiones abiertas post-MVP | `docs/funcionalidades-post-mvp.md` |
| `ADR-xxx` | Decisiones arquitectónicas | `docs/adr/` |

Formato de referencia: dentro del mismo documento, `§X`; entre documentos, `AGENTS.md` §X o `docs/<documento>.md` §X; para el chat, `conversación §X`. Si el documento no está claro por el contexto, indícalo siempre.

Reglas:

- Un documento con estado `pendiente` **no existe todavía**: no supongas su contenido ni lo inventes; dilo y propón crearlo.
- Cuando un detalle vive en un documento, este archivo conserva solo la regla esencial y el enlace. No dupliques información entre ambos.
- Si un documento contradice este archivo, señálalo antes de actuar.
- Cuando se cree un documento, cambia su estado a `creado` en esta tabla.
- Si el documento no está en la tabla, agrégalo con estado `creado` en el mismo cambio en que se crea.
- Un archivo vacío no cuenta como documento creado: su estado sigue `pendiente` y no se supone su contenido.

## 14. Cómo trabajar

**Memoria de trabajo (`MEMORY.md`, en la raíz del repositorio)**

Este archivo define las reglas permanentes; `MEMORY.md` guarda lo temporal. AGENTS.md ordena y MEMORY.md ejecuta: si se contradicen, manda AGENTS.md.

- **Al iniciar una sesión:** lee `AGENTS.md`, `MEMORY.md` y la tabla de la sección 13; abre solo el documento dueño del tema. Resume en 2–3 líneas dónde quedó el trabajo y cuál es el siguiente paso, y pregunta si el desarrollador quiere cambiar la prioridad. Si MEMORY.md no existe, dilo y propón crearlo; no supongas el estado del proyecto.
- **Al terminar cada tarea:** actualízalo. Mueve lo terminado, registra las decisiones nuevas con fecha y actualiza los siguientes pasos y los bloqueos.
- **Qué va ahí:** estado actual, sesión actual, tareas en progreso, backlog corto, decisiones recientes, problemas abiertos, contexto temporal e historial breve de sesiones.
- **Qué no va:** reglas permanentes (van en este archivo), detalle técnico extenso (va en `/docs`), conversaciones copiadas, ni secretos, tokens o credenciales.
- **Poda:** máximo 100–150 líneas. Elimina tareas completadas hace más de 2 semanas, resume en una línea las sesiones de más de 5 atrás y borra las decisiones que ya quedaron en este archivo o en un ADR.
- **Promoción:** si algo temporal se vuelve permanente (una regla, una convención), **propón** moverlo a este archivo o a `/docs`. No lo muevas sin aprobación del desarrollador.

**Antes de cambiar algo**

1. Entiende qué se pide y qué parte del sistema involucra.
2. Revisa la estructura y los archivos relacionados; no modifiques archivos sin entender su responsabilidad.
3. Comprueba si ya existe algo reutilizable. No dupliques funcionalidad ni cambies tecnologías sin justificación.
4. Identifica efectos secundarios y propón algo coherente con la arquitectura actual.

**Cuándo presentar un plan antes de implementar:** si el cambio toca más de 3 archivos, agrega una dependencia, cambia el esquema de datos o el contrato de la API, o crea un módulo nuevo. El plan incluye objetivo, archivos, cambios, dependencias, riesgos y pruebas. Los cambios pequeños y claros se hacen directamente.

**Al proponer una solución:** explica qué problema resuelve, cómo funciona, qué limitaciones tiene y qué alternativas existen. Prefiere lo simple y mantenible sobre lo popular. No apliques un patrón de diseño sin una necesidad concreta.

**Dependencias:** primero comprueba si el proyecto ya tiene solución; luego evalúa mantenimiento y compatibilidad, y explica por qué se necesita.

**Errores:** encuentra la causa raíz y manéjalos en la capa que corresponde. Evita `try/catch` indiscriminados.

**Código:** nombres descriptivos, funciones pequeñas con una sola responsabilidad, sin valores mágicos (usa constantes o configuración), comentarios solo donde aclaren algo complejo. TypeScript con tipado estricto.

**Con el desarrollador:** actúa como colaborador técnico, no solo como generador de código. Explica los conceptos cada vez que realices algo, advierte sobre riesgos y, si una propuesta suya tiene problemas técnicos, dilos y ofrece alternativas. Ninguna decisión inicial es definitiva.

## 15. Orden de construcción

1. **Fundamentos:** validar fuentes de datos, requerimientos, arquitectura, modelo de datos y reglas de negocio.
2. **Funcionalidad principal:** gestión de contingencias, ingesta, estaciones y líneas en el mapa, origen/destino, alternativas y tiempos.
3. **Experiencia:** interfaz responsive, visualización y comparación de rutas, compartir, PWA.
4. **Calidad y despliegue:** pruebas E2E y de carga, optimización, Docker, CI/CD, despliegue.

Las pruebas unitarias y de integración acompañan cada funcionalidad (sección 10); el paso 4 completa esa base, no la reemplaza. El orden puede cambiar si hay una dependencia técnica o un hallazgo que lo justifique. El avance real de cada fase se registra en `MEMORY.md`, no aquí.

## 16. Principio rector

La prioridad no es construir más funcionalidades, sino resolver correctamente el problema principal del usuario con una solución técnicamente sólida, útil y verificable. Ante la duda, prioriza:

1. Información confiable.
2. Alternativas de movilidad coherentes.
3. Estimaciones transparentes.
4. Experiencia sencilla.
5. Arquitectura mantenible y evolución incremental.
