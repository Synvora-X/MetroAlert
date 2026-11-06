# Fuentes de datos — MetroAlerta

| Campo | Valor |
|---|---|
| Estado | creado |
| Fecha | 2026-11-06 |
| Fuentes | Conversación "Plan de MetroAlerta" (URLs recuperadas del JSON del chat) + `AGENTS.md` §7 + `docs/tareas-de-investigacion.md` (INV-01…INV-09) |
| Propósito | Registrar qué fuentes existen, qué pueden aportar, bajo qué condiciones y con qué vigencia. |
| Autoridad | Ninguna fila es un dato confirmado hasta completar su verificación y registrar fecha y evidencia. |

**Para agentes:**
- No uses una fila como dato real: todas están `por verificar` hasta que se complete su ficha.
- No inventes campos, coberturas ni vigencias; si no se puede comprobar, se escribe `no disponible`.
- Toda fuente debe conservar URL, fecha de consulta, licencia y condiciones de uso.

---

## 1. Método de investigación

Para cada fuente:

1. Localizar el portal o servicio oficial y registrar la URL exacta.
2. Comprobar que responde y anotar la fecha de consulta.
3. Descargar o inspeccionar una muestra pequeña.
4. Identificar campos, formato, sistema de coordenadas, cobertura y frescura.
5. Leer condiciones de uso, licencia y atribución.
6. Registrar evidencia y estado: `verificada`, `por verificar` o `descartada`.
7. No usar la fuente en código hasta que la ficha esté completa.

## 2. Inventario de fuentes

| Fuente | Propietario | URL | Tipo | Aporta (potencial) | Licencia / condiciones | Verificada | Estado |
|---|---|---|---|---|---|---|---|
| **Metro de Medellín** | Metro de Medellín | <https://www.metrodemedellin.gov.co/> | Sitio oficial | Líneas, estaciones, sistema integrado, estado, noticias | Por revisar | — | por verificar |
| **Sistema integrado** | Metro de Medellín | <https://www.metrodemedellin.gov.co/usuarios/sistema-integrado/> | Página oficial | Líneas, cables, tranvía, buses, mapas | Por revisar | — | por verificar |
| **Transparencia Metro** | Metro de Medellín | <https://www.metrodemedellin.gov.co/quienes-somos/transparencia/> | Portal de transparencia | Datos abiertos, esquema de publicación | Por revisar | — | por verificar |
| **ArcGIS Sistema_Metro2** | ArcGIS / Metro | <https://services3.arcgis.com/ARUXxWVTK25ENUfB/ArcGIS/rest/services/Sistema_Metro2/FeatureServer> | Servicio geográfico | Capas de Metro, cable, tranvía y BRT | Por revisar | — | por verificar |
| **ArcGIS Estaciones A y B** | ArcGIS | <https://services1.arcgis.com/Qrk4Z5vQ94JXkdYM/arcgis/rest/services/Estaciones_lineas_A_y_B_majo/FeatureServer> | Servicio geográfico | Estaciones de líneas A y B | Por revisar | — | por verificar |
| **ArcGIS Estaciones Metro A/B** | ArcGIS | <https://services1.arcgis.com/Qrk4Z5vQ94JXkdYM/arcgis/rest/services/Estaciones_lineas_metro_A_B/FeatureServer> | Servicio geográfico | Estaciones de líneas A y B | Por revisar | — | por verificar |
| **GTFS Metro Medellín** | TUMI Datahub | <https://hub.tumidata.org/es/dataset/gtfs_metro_medellin_medelln> | Dataset GTFS | Paradas, rutas, viajes, horarios, geometrías | Por revisar | — | por verificar |
| **Datos Abiertos AMVA** | Área Metropolitana | <https://datosabiertos.metropol.gov.co/> | Portal de datos abiertos | Movilidad, territorio, paraderos, rutas | Por revisar | — | por verificar |
| **Datos Abiertos AMVA (página)** | Área Metropolitana | <https://www.metropol.gov.co/area/Paginas/datos%20abiertos%20amva.aspx> | Página institucional | Catálogo y condiciones | Por revisar | — | por verificar |
| **MEData** | Alcaldía de Medellín | <https://www.medellin.gov.co/es/transparencia/medata/> | Portal de datos públicos | Datos municipales, movilidad | Por revisar | — | por verificar |
| **OpenStreetMap** | Comunidad OSM | <https://www.openstreetmap.org/> | Datos geográficos abiertos | Red, estaciones, paraderos, geometrías | ODbL y atribución; política de tiles por revisar | — | por verificar |
| **Canales del Metro** | Metro de Medellín | Web, X, Instagram, Facebook | Comunicación | Avisos de contingencia y mantenimiento | Por revisar | — | por verificar |

> Las URLs se recuperaron del JSON original del chat; la extracción `.md` no conservó los `href`. Su presencia aquí no confirma vigencia ni condiciones.

## 3. Fichas por fuente

### 3.1 Metro de Medellín

**Preguntas:** ¿publica una API, dataset o servicio con líneas, estaciones y estado? ¿Qué permite su política? ¿Hay página de estado separada?

**Evidencia esperada:** URL, fecha, formato, cobertura, condiciones.

**Riesgo:** que el estado solo exista como noticia o red social, no como dato estructurado.

### 3.2 Servicios ArcGIS

**Preguntas:** ¿siguen vigentes? ¿Qué capas entregan? ¿La actualización de agosto de 2023 sigue siendo la última? ¿Permiten descarga o solo consulta?

**Evidencia esperada:** metadatos del servicio, capas, campos, sistema de coordenadas y licencia.

**Riesgo:** capas incompletas o sin actualización reciente.

### 3.3 GTFS de Medellín

**Preguntas:** ¿qué cubre? ¿Qué calidad tienen horarios y frecuencias? ¿Existe GTFS Realtime? ¿Es oficial o de terceros?

**Evidencia esperada:** archivos `agency`, `routes`, `stops`, `trips`, `stop_times`, `shapes`, `calendar`; fecha de actualización.

**Riesgo:** el catálogo advierte correcciones pendientes; no asumir que es oficial ni actualizado.

### 3.4 AMVA y MEData

**Preguntas:** ¿qué datasets de movilidad, paraderos, rutas o territorio publican? ¿Formatos, actualización y condiciones?

**Evidencia esperada:** catálogo, ficha del dataset, muestra y licencia.

**Riesgo:** datos agregados sin granularidad por paradero o sin coordenadas válidas.

### 3.5 OpenStreetMap

**Preguntas:** ¿qué datos de la red están disponibles? ¿Qué atribución exige la licencia? ¿Qué política aplica al servidor público de tiles?

**Evidencia esperada:** licencia ODbL, atribución visible y términos de uso de tiles.

**Riesgo:** usar el servidor público en producción sin respetar límites de tráfico.

### 3.6 Canales de alertas

**Preguntas:** ¿existe un canal oficial estructurado? ¿Las publicaciones en X, Instagram o Facebook son la única fuente? ¿Qué condiciones limitan su consulta?

**Evidencia esperada:** URL del canal, tipo de publicación, frecuencia y condiciones.

**Regla:** los canales se consultan manualmente en el MVP; **X no es una dependencia de scraping**. La ingesta automática se decide después de INV-07.

## 4. Qué inspeccionar en una muestra

- Nombres y códigos de líneas, estaciones y paraderos.
- Coordenadas y sistema de referencia (WGS84 esperado).
- Geometrías (punto, línea, polígono) y su precisión.
- Horarios, frecuencias y calendario.
- Campos obligatorios vacíos y duplicados.
- Fecha de última actualización y frecuencia de publicación.
- Licencia, atribución y condiciones de redistribución.

## 5. Licencias y atribución

- **OpenStreetMap:** licencia ODbL; mostrar atribución y respetar la política de uso.
- **Datos oficiales:** verificar términos antes de almacenar, mostrar o redistribuir.
- **Datos de terceros:** no se usan si no permiten redistribución o si exigen autorizaciones no obtenidas.
- **Datos simulados:** se identifican como `simulado` y se mantienen separados de los reales.

## 6. Decisión para el MVP

El MVP arranca con **registro manual/simulado** de alertas y datos de red recolectados previamente. La ingesta automática es opcional y no bloquea el flujo principal. Ninguna fuente se presenta como oficial hasta completar su ficha.

## 7. Registro de evidencia

| Fecha | Fuente | URL consultada | Resultado | Evidencia | Estado |
|---|---|---|---|---|---|
| — | — | — | — | — | por verificar |

## 8. Riesgos y preguntas abiertas

- Cobertura real y granularidad de paraderos y rutas.
- Vigencia de los servicios ArcGIS y del GTFS.
- Condiciones de uso y redistribución.
- Existencia de un canal oficial estructurado para alertas.
- Dependencia de una sola fuente y necesidad de respaldo.

## 9. Cómo entra en acción

- **`transport`:** importadores y modelos de líneas, estaciones, paraderos y segmentos.
- **`alerts`:** adaptadores de fuentes y registro manual.
- **Modelo de datos:** campos, tipos, geometrías y metadatos de fuente.
- **UI:** atribución de OpenStreetMap y fecha de actualización.
- **Calidad de datos:** panel de fuentes, frescura y errores (PM-21).

## 10. Cuándo actualizar este documento

- Al verificar o descartar una fuente.
- Al cambiar una URL, licencia o condición de uso.
- Al incorporar una fuente nueva.
- Al cerrar INV-01…INV-09.
