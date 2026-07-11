# Manifiesto — SP-01: Listado de Referencias

Sub-plan: [[2026-07-10-refactor-flujo-ejecutivo-sp01-listado-referencias]]
Fecha: 2026-07-11. Re-implementación desde cero (la versión anterior, con
commits `64402b45` / `db4f2678` / `0354dd90`, fue descartada por decisión del
usuario y sus ramas eliminadas en ambos repos).

## Ramas de trabajo (sin commits, working tree para revisión humana)
- `carmi-odin-api-v2`: `refactor/customs-operation-sp01`, creada desde `staging`.
- `carmi-digital`: `refactor/customs-operation-sp01`, creada desde
  `refactor/customs-operation-sp03` (encadenada — conserva el diff sin
  commitear de SP-03: `ReferenceDetailShell.tsx`, `ReferenceHeroHeader.tsx`,
  `ReferenceOverview.tsx`, borrado de `ReferenceTabs.tsx`, cambio en
  `[id]/page.tsx` y `public/firebase-messaging-sw.js`, ese diff no se tocó).

Nota: `carmi-odin-api-v2` tenía ya sobre `staging`, antes de que yo empezara,
un cambio sin commitear ajeno a este sub-plan en
`src/companies/companies.module.ts` — no lo toqué ni lo referencié.

## Archivos tocados

### Back — `carmi-odin-api-v2`
- `src/references/controllers/references.controller.ts`: se agregó el query
  param `trafficTypeId` (con su `@ApiQuery` de Swagger) al endpoint
  `GET /references` (`findAll`), y se agrega a `filters.trafficTypeId` cuando
  viene presente. El `service.findAll` ya esparce `otherFilters` directo al
  `where` de Prisma (`references.service.ts`, sin cambios), así que
  `trafficTypeId` llega tal cual como filtro sobre el campo `trafficTypeId`
  del modelo `Reference` — no fue necesario tocar el service.
- `src/references/controllers/references.controller.spec.ts`: se agregó el
  test `should filter by trafficTypeId and pass it in filters`, siguiendo el
  patrón exacto del test ya existente para `clientCompanyId`. De paso se
  corrigió con `eslint --fix` un `import/order` preexistente en el mismo
  archivo (dos líneas de import, sin relación con mi cambio; lo arreglé por
  estar ya tocando el archivo y para dejar el gate de lint verde).

### Front — `carmi-digital`
- `app/(customerPortal)/references/ui/ReferencesClient.tsx` (reescrito):
  ahora es un shell delgado — header (título + botón "Nueva Referencia") y un
  `Tabs` de nivel superior con dos vistas: "Tablero" (primaria, default) y
  "Tabla clásica" (secundaria). Mantiene el estado `tablePreset` para pasar un
  estatus preseleccionado cuando el usuario salta desde "Ver todas" de una
  columna del tablero a la tabla clásica.
- `app/(customerPortal)/references/components/ReferenceBoard.tsx` (nuevo):
  tablero primario versión mínima, 3 columnas mapeadas a `ReferenceStatus`
  (decisión ya documentada en el sub-plan, reutilizada tal cual):
  - "Por identificar" → `DRAFT`
  - "En espera de terceros" → `PENDING_QUOTE`
  - "En curso" → `QUOTED` + `APPROVED`
  Cada columna hace fetch a `GET /references?status=X&limit=5` (una llamada
  por estatus de la columna, mezcladas y ordenadas por `createdAt desc` en la
  columna "En curso"), muestra badge de total y hasta 5 tarjetas con link al
  detalle; si el total excede 5, aparece "Ver todas" que llama a
  `onViewAll(statuses)`.
- `app/(customerPortal)/references/components/ReferenceClassicTable.tsx`
  (nuevo, contiene toda la lógica que antes vivía directo en
  `ReferencesClient.tsx`: selección, diálogos de eliminar/reactivar, modal de
  creación de movimiento/operación, paginación server-side ya existente sin
  cambios). Cambios respecto al original:
  - Filtro de estatus ampliado de 3 a los 6 valores reales de
    `ReferenceStatus` (antes: DRAFT/APPROVED/CANCELLED; ahora agrega
    `PENDING_QUOTE`, `QUOTED`, `REJECTED`).
  - Nuevo filtro de cliente (`clientCompanyId`), usando el hook existente
    `useClients` (`hooks/use-clients.ts`).
  - Nuevo filtro de tráfico (`trafficTypeId`), usando el hook existente
    `useTransportModes` (`hooks/useTransportModes.ts`), que lista
    `TransportMode` (catálogo real detrás del campo `trafficTypeId`; no existe
    un enum "TrafficType" dedicado en el schema).
  - El hook interno `useReferences` ahora también envía `clientCompanyId` y
    `trafficTypeId` como query params al backend.
  - Acepta prop opcional `initialStatusFilter` para inicializar el filtro de
    estatus cuando se llega desde "Ver todas" del tablero (la tabla clásica
    solo soporta un único estatus a la vez, así que la columna "En curso" del
    tablero, que combina dos estatus, no puede preseleccionarse en la
    tabla — ahí solo se cambia de tab sin preset).
  - La barra contextual "N seleccionada(s) + Crear Operación" (antes en el
    header global) se movió dentro de este componente, arriba de los filtros,
    porque la selección múltiple solo tiene sentido en esta vista (el tablero
    es de solo lectura). El botón "Nueva Referencia" quedó fijo en el header
    del shell (`ReferencesClient.tsx`), visible siempre.

## Desviaciones respecto al original (justificadas)
- Se dividió la barra de acciones contextual (selección → "Crear Operación")
  del header global hacia dentro de `ReferenceClassicTable`, en vez de
  levantar el estado `selectedIds` al shell. Es la separación correcta dado
  que "Tablero" (SP-01, versión mínima) no implementa selección múltiple; no
  tenía sentido mantener ese estado en el componente padre solo para una
  vista hija.
- Se corrigió un lint preexistente (`import/order`) en
  `references.controller.spec.ts` no relacionado con mi cambio, ya que estaba
  tocando el archivo de todos modos (ver sección de tests arriba).

## Fuera de alcance (no tocado)
- Módulo `operations` (`customerPortal/operations`, `components/operations`,
  controllers `operations` del back): no se tocó. Sí se reutiliza el import
  existente `@/components/operations/CreateOperationModal` (ya estaba
  importado en el `ReferencesClient.tsx` original) — no se modificó ese
  componente.
- Bandeja de Entrada / Inbox completo (auditoría CEUS, guías sin identificar,
  priorización real por ETA/urgencia): SP-17, fuera de alcance de este
  sub-plan.

## Criterios de verificación
- **Gate estático**: verde.
  - Back: `npx jest src/references/controllers/references.controller.spec.ts`
    (12/12 ✅) y `npx jest src/references/services/references.service.spec.ts`
    (4/4 ✅); `npx eslint` sobre los archivos tocados sin errores; `tsc
    --noEmit` sin errores nuevos en el módulo `references`.
  - Front: `npx tsc --noEmit` sin errores nuevos en los archivos tocados
    (los únicos errores del proyecto son preexistentes, de módulos ajenos:
    `three`, `mammoth`); `npx eslint` sobre los 3 archivos tocados/nuevos: 0
    errores, 1 warning `no-explicit-any` (mismo patrón que ya tenía el
    archivo original, no introducido por este cambio).
- **Playwright**: NO ejecutado. Mismo motivo documentado en el cierre previo
  de este sub-plan: el entorno usa credenciales reales (`.env`) contra
  infraestructura viva sin supervisión humana en esta sesión — no se navegó
  el flujo en vivo. Queda pendiente de validación manual/QA humana antes de
  mergear.

## Pendiente / bloqueos
- Ninguno de alcance. Queda pendiente, fuera de lo que un agente sin
  supervisión debe decidir: correr Playwright contra el entorno real y
  revisar visualmente el tablero y los nuevos filtros con un humano presente.
