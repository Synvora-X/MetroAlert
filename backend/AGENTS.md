# AGENTS.md — Backend MetroAlerta

| Campo | Valor |
|---|---|
| Estado | creado |
| Fecha | 2026-11-06 |
| Alcance | Solo `backend/` |
| Precedencia | `../AGENTS.md` > este archivo > `../MEMORY.md` > `backend/MEMORY.md` |

> Este archivo complementa `../AGENTS.md`. No repite el alcance del producto, las funcionalidades F01–F05, las exclusiones del MVP ni las reglas completas de seguridad: esos temas viven en la raíz. Aquí solo se define cómo se construye el backend.

## 1. Lectura y precedencia

Antes de trabajar en `backend/`, lee en este orden:

1. `../AGENTS.md`
2. `backend/AGENTS.md` (este archivo)
3. `../MEMORY.md`
4. `backend/MEMORY.md`
5. El documento dueño del tema: `../docs/arquitectura.md`, `../docs/dominio.md`, `../docs/modelo-de-datos.md`, `../docs/api.md` o `../docs/testing.md`.

Si hay contradicción, gana el archivo de mayor precedencia y se señala antes de actuar.

## 2. Estado actual del backend

- NestJS 12 con TypeScript, ESM (`"type": "module"`) y `moduleResolution: nodenext`.
- `strict: true`; los imports relativos usan extensión `.js` (ver `src/main.ts`).
- Vitest 4 con dos configuraciones: unitaria (`vitest.config.ts`) y e2e (`vitest.config.e2e.ts`).
- Supertest 7, oxlint y Prettier configurados.
- pnpm como gestor; `backend/pnpm-workspace.yaml` declara un workspace local con `packages: ["."]`.
- Solo existen `src/main.ts` y `src/app.module.ts`; no hay módulos de dominio ni base de datos conectada.
- `pnpm test` no encuentra suites y `pnpm test:e2e` falla con 404 porque el e2e es el esqueleto de NestJS.
- `@nestjs/observe` está instalado, pero su integración está incompleta (ver §13).
- ADR-006 acepta un monorepo con destino `apps/api`; la migración es incremental y todavía no se ejecutó.

## 3. Comandos

Desde la raíz del repositorio:

| Acción | Comando |
|---|---|
| Instalar | `cd backend && pnpm install` |
| Desarrollo | `cd backend && pnpm start:dev` |
| Producción local | `cd backend && pnpm start:prod` |
| Tests unitarios | `cd backend && pnpm test` |
| Tests e2e | `cd backend && pnpm test:e2e` |
| Cobertura | `cd backend && pnpm test:cov` |
| Lint | `cd backend && pnpm lint` |
| Formato | `cd backend && pnpm format` / `pnpm format:check` |
| Build | `cd backend && pnpm build` |

`pnpm test` y `pnpm test:e2e` fallan hoy por el estado del proyecto; no los presentes como verdes. El CI mantiene esos pasos comentados.

## 4. Estructura y módulos

Cada funcionalidad del backend vive en un módulo de feature:

```text
src/
├── main.ts
├── app.module.ts
├── modules/
│   ├── alerts/
│   │   ├── domain/
│   │   ├── application/
│   │   ├── infrastructure/
│   │   ├── presentation/
│   │   └── alerts.module.ts
│   ├── transport/
│   ├── route-engine/
│   ├── geolocation/
│   └── sharing/
├── shared/
└── config/
```

Reglas:

- No crear todas las carpetas de golpe: cada módulo nace cuando se implementa su primera funcionalidad.
- `<module>.module.ts` es el **ensamblaje de NestJS**: declara controladores, providers, imports, exports y tokens de DI. No contiene reglas de negocio.
- `shared/` contiene filtros, pipes, errores y utilidades técnicas; nunca reglas de negocio.
- `config/` contiene configuración tipada y parámetros; nunca secretos.
- No crear `users/` hasta que exista una necesidad real y una decisión documentada.
- El destino del monorepo es `apps/api`; no mover archivos sin un PR de migración que actualice el CI.

## 5. Capas y dependencia

| Capa | Contiene | No debe contener |
|---|---|---|
| `domain` | Entidades, value objects, reglas, invariantes e interfaces de repositorios. | Decoradores de NestJS, SQL, HTTP, DTOs de transporte. |
| `application` | Casos de uso y DTOs de entrada/salida de la aplicación. | Controladores, SQL, detalles de Express/Fastify. |
| `infrastructure` | Repositorios Supabase, adaptadores GTFS/ArcGIS y clientes externos. | Reglas de negocio. |
| `presentation` | Controladores HTTP, DTOs de request/response, validación y mapeo. | Lógica de negocio o acceso directo a la base. |
| `<module>.module.ts` | Ensamblaje de providers, controllers, imports y exports. | Reglas de negocio. |

Regla: **las dependencias apuntan hacia adentro** (`presentation → application → domain`). `infrastructure` implementa las interfaces del dominio o de la aplicación. El módulo de NestJS es el único lugar que puede ver todas las capas para ensamblarlas.

Ejemplo: `CreateAlert` vive en `application`; usa la interfaz `AlertRepository` de `domain`; `SupabaseAlertRepository` la implementa en `infrastructure`; `AlertsController` solo llama al caso de uso.

## 6. Convenciones NestJS y TypeScript

- Inyección de dependencias por constructor; no usar service locator ni instanciar dependencias dentro de un caso de uso.
- `main.ts` solo hace bootstrap; no contiene lógica de negocio ni configuración extensa.
- Módulos de NestJS para ensamblar; providers para casos de uso y adaptadores.
- Imports relativos con `.js` por ESM/NodeNext.
- TypeScript estricto; evitar `any` sin justificación escrita.
- Nombres de archivos, clases, métodos y variables en inglés.
- Documentación `.md`, comentarios de producto y mensajes visibles al usuario en español; logs técnicos en inglés.
- Sin valores mágicos: constantes, configuración o parámetros nombrados.
- DTOs de request/response en `presentation`; modelos de aplicación en `application`; entidades en `domain`. No reutilizar modelos de base de datos como entidades de dominio.
- La validación de entrada se define con el primer endpoint; no agregar dependencias de validación sin aprobación.

## 7. API y errores

- El contrato vive en `../docs/api.md`.
- Base `/api/v1`; JSON; fechas ISO 8601; IDs UUID.
- Validar en el límite y responder con códigos HTTP correctos.
- Errores controlados con forma estable; nunca exponer stacks, SQL, claves ni detalles internos.
- Administración protegida; sin cuentas de pasajeros en el MVP.
- Los endpoints son propuestas hasta que exista implementación y pruebas de contrato.

## 8. Persistencia

- El modelo vive en `../docs/modelo-de-datos.md`.
- PostGIS habilitado; sin pgRouting; grafo construido en la aplicación.
- Repositorios en `infrastructure`; nunca SQL en controladores ni en `domain`.
- RLS y permisos explícitos; la `service_role` solo se usa en el backend.
- Migraciones requieren aprobación (`../AGENTS.md` §9).
- Cada dato guarda fuente y frescura cuando aplique; no se usan datos de producción en desarrollo ni pruebas.

## 9. Route Engine

- Construye el grafo desde nodos y segmentos; granularidad por conexión entre nodos consecutivos.
- Dijkstra primero; K-caminos (Yen) después; A* solo si el tamaño real de la red lo exige.
- Máximo 3 alternativas; si no hay datos, respuesta explícita sin inventar rutas.
- Excluir segmentos afectados por contingencias activas confirmadas.
- Tiempo con parámetros configurables; sin valores mágicos.
- Determinismo: mismo grafo y mismas restricciones producen el mismo resultado.
- Sin lógica de mapas, UI ni gestión de alertas.

## 10. Alertas y dominio

- Tres estados de alerta: `activa`, `resuelta`, `vigencia indeterminada`.
- Una alerta no es automáticamente una contingencia; solo una alerta verificada se muestra como contingencia.
- El mantenimiento es una entidad separada, no un cuarto estado.
- La severidad se deriva de reglas documentadas y es revisable.
- Las transiciones y los invariantes están en `../docs/dominio.md`.

## 11. Pruebas

- Estrategia en `../docs/testing.md`.
- Unitarias junto al dominio (`*.spec.ts`); e2e en `test/` (`*.e2e-spec.ts`).
- Probar comportamiento a través de interfaces públicas; sin detalles internos.
- Fixtures deterministas y marcados como simulados; sin red ni datos de producción.
- No modificar pruebas para que pasen; corregir la causa.
- El esqueleto e2e actual debe reemplazarse por el flujo real del MVP.

## 12. Seguridad

- Secretos en variables de entorno; `.env.example` sin valores.
- `SUPABASE_SERVICE_ROLE_KEY` solo en backend; nunca en el frontend.
- Validar entradas y datos de fuentes externas.
- No registrar información sensible innecesaria.
- La ubicación del usuario no se persiste sin decisión documentada.
- No acceder a producción desde el entorno local.

## 13. Observabilidad (`@nestjs/observe`)

`@nestjs/observe` es el SDK de observabilidad de NestJS: engancha el ciclo de vida de las peticiones y envía peticiones, trabajos, errores, logs y trazas a NestJS Observe.

**Estado actual: integración incompleta.**

- `createObserveModule()` se usa en `app.module.ts` y `ObserveInstrument` se pasa a `NestFactory.create`.
- Falta importar `ObserveModule.forRoot({ appKey, appSecret, serviceId })` y no hay credenciales.
- Por tanto, no se considera observabilidad activa ni se usa como fuente de logs o métricas.

Reglas:

- No agregar credenciales de Observe al repositorio.
- No depender de `@nestjs/observe` para salud, logs o métricas del MVP.
- Su alcance y decisión se registran en INV-20.
- Si se activa, usar variables de entorno y verificar coste, privacidad y retención.

## 14. Definición de terminado

Una tarea del backend está terminada cuando:

1. Cumple el RF/HU correspondiente.
2. Tiene pruebas del nivel adecuado y pasan.
3. `pnpm lint`, `pnpm format:check` y `pnpm build` pasan.
4. No introduce regresiones conocidas.
5. Actualiza la documentación afectada.
6. Actualiza `backend/MEMORY.md` con el estado real.
7. No agrega secretos ni datos de producción.

Que el código compile no es suficiente.

## 15. No hacer

- Agregar dependencias, migraciones o cambios de contrato de API sin aprobación.
- Introducir microservicios, colas o brokers.
- Poner lógica de negocio en controladores.
- Inventar estaciones, paraderos, coordenadas, tiempos o fuentes.
- Acceder a la base de datos desde el frontend.
- Usar `@nestjs/observe` como requisito del MVP.
- Mover el backend a `apps/api` sin el PR de migración y la actualización del CI.

## 16. Cuándo actualizar este documento

- Al crear, mover o renombrar un módulo.
- Al cambiar una regla de capas, una convención o un comando.
- Al habilitar o retirar `@nestjs/observe`.
- Al ejecutar la migración al monorepo.
