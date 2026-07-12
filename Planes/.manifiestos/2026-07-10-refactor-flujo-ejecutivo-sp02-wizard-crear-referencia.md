# Manifiesto — SP-02: Wizard de Crear Referencia (3 pasos)

Sub-plan: [[2026-07-10-refactor-flujo-ejecutivo-sp02-wizard-crear-referencia]]
Rama: `refactor/customs-operation-sp02` en ambos repos (encadenada desde
`refactor/customs-operation-sp01`, diff acumulado de SP-03+SP-01 intacto).
Estado: **parcialmente completado** — bloqueo real en la migración Prisma
(infraestructura, no de código), resto de tareas completo.

## Archivos tocados

### Front (`carmi-digital`)
- `app/(customerPortal)/references/createReference/page.tsx` — orquestador del
  wizard: reordenado a 3 pasos (básicos → documentos → auditoría), retirado el
  paso "Documentos Comerciales", y eliminadas ~640 líneas de estado/handlers
  que quedaban muertos como consecuencia directa de quitar ese paso
  (auto-búsqueda de proveedor, mapeo de `invoices`/`packingLists`, modal de
  factura duplicada).
- `app/(customerPortal)/references/createReference/create-reference-context.tsx`
  — lista de pasos del wizard actualizada a 3.
- `app/(customerPortal)/references/createReference/components/stepBasicDataReference/index.tsx`
  — campo "orden de compra"; ocultamiento de selectores de cliente/tráfico/
  instrucciones cuando el creador es el cliente-portal (ver decisión abierta
  más abajo).
- `lib/schemas/new-reference-basicData.ts` — validación del nuevo campo OC.
- `components/Ref/ReferenceMain.tsx` — se corrigió un import muerto hacia
  `reference-stepper` (componente no usado en ninguna ruta activa).
- Eliminado: `app/(customerPortal)/references/reference-stepper/index.tsx`
  (wizard alterno con placeholders, código muerto confirmado por grep: solo
  lo importaba `ReferenceMain.tsx`, que a su vez no está montado en ninguna
  ruta activa).

### Back (`carmi-odin-api-v2`)
- `prisma/schema.prisma` — campo `purchaseOrder String? @map("client_purchase_order")`
  en `Reference`, con comentario documentando el bloqueo de migración (ver
  abajo). Mapeado a una columna que YA existe físicamente en la DB de este
  entorno.
- `src/references/dtos/create-reference.dto.ts` — campo OC expuesto en el DTO
  de creación con validadores class-validator siguiendo el patrón existente.
- `src/references/services/references.service.ts` — persiste/retorna el
  campo OC en creación y en `GET /references/:id`.

## Por qué de cada cambio
1. **Orden 3 pasos**: decisión M9 ya tomada en el sub-plan (datos básicos
   primero porque la auditoría de extracción los necesita) — aplicada tal
   cual.
2. **Retirar "Documentos Comerciales" del wizard sin borrar el componente**:
   el propio sub-plan dice que esa sección se REUBICA al detalle en SP-05,
   no se elimina como feature. Borrarla habría anticipado trabajo de otro
   sub-plan fuera de este alcance.
3. **Campo OC**: pedido explícito del sub-plan (M9: el cliente busca por su
   OC).
4. **Dos sabores cliente/ejecutivo**: se reutilizó `user.IsInternalUser`, ya
   presente en el contexto de sesión/auth pero sin usar en ninguna vista —
   es la distinción de rol reutilizable más cercana encontrada en el repo.
5. **Limpieza `reference-stepper`**: código muerto confirmado por grep antes
   de borrar, según regla del proyecto.

## Desviaciones y decisiones abiertas (requieren confirmación humana)
- **`stepCommercialDocsReference`**: queda en el repo sin cablear al wizard
  (no es código muerto: material para SP-05). No tocar su implementación
  interna, solo su desconexión del wizard ya se hizo aquí.

## Cierre 2026-07-12 — decisiones validadas por el usuario

1. **Distinción cliente/ejecutivo (tarea 4)**: el usuario confirmó que
   `user.IsInternalUser` es el criterio correcto para distinguir el sabor
   cliente vs. ejecutivo del wizard. No se requirió ningún cambio de código:
   se verificó que `stepBasicDataReference/index.tsx:67` sigue leyendo
   `user?.user?.IsInternalUser` tal como quedó implementado. Decisión cerrada.

2. **`stepChecklistGlosa`**: **eliminado**. Razón confirmada por el usuario:
   la glosa se está retirando del proceso automático de operación (Decisión
   #5 del paraguas — el paso GLOSA desaparece del stepper de despacho; la
   glosa la asume el DGO a nivel de referencia, ya implementado en
   SP-04/SP-05). Este step del wizard de creación queda obsoleto y redundante
   con ese modelo nuevo.
   - Eliminado: `app/(customerPortal)/references/createReference/components/stepChecklistGlosa/`
     (incluye `index.tsx` y `components/step4-checklist-glosa.tsx`,
     `components/traffic-light.tsx`).
   - Eliminado también `lib/schemas/new-reference-checklist-glosa.ts` — schema
     exclusivo de ese step, sin más referencias en el repo (confirmado por
     grep antes de borrar).

3. **`stepInvoicesReference`**: **eliminado**. Se investigó primero si algo
   del flujo actual (SP-01/SP-02/SP-04/SP-05 ya implementados) dependía de
   este step o de su export `StepInvoicesReference`; grep en todo el repo
   (excluyendo `node_modules`/`.next`) no encontró ninguna referencia externa
   a `stepInvoicesReference`, a `StepInvoicesReference` ni a su tipo
   `Factura` fuera del propio archivo. Código muerto confirmado.
   - Eliminado: `app/(customerPortal)/references/createReference/components/stepInvoicesReference/index.tsx`.

### Archivos tocados en este cierre (front, `carmi-digital`)
- Eliminados (git rm):
  - `app/(customerPortal)/references/createReference/components/stepChecklistGlosa/index.tsx`
  - `app/(customerPortal)/references/createReference/components/stepChecklistGlosa/components/step4-checklist-glosa.tsx`
  - `app/(customerPortal)/references/createReference/components/stepChecklistGlosa/components/traffic-light.tsx`
  - `app/(customerPortal)/references/createReference/components/stepInvoicesReference/index.tsx`
  - `lib/schemas/new-reference-checklist-glosa.ts`
- Sin cambios de código (solo verificación): `stepBasicDataReference/index.tsx`
  (criterio `IsInternalUser` confirmado, ya correcto).

### Verificación del cierre
- **Front**: `npx tsc --noEmit` — sin errores nuevos (los 4 errores presentes
  son preexistentes y no relacionados: módulos `three` y `mammoth` faltantes
  en `features/locations/warehouse/components/Trailer3D.tsx` y
  `features/zeus-shell/components/Renderers/DocxRenderer.tsx`).
  `npx eslint` sobre los archivos tocados — 0 errores nuevos, solo warnings
  preexistentes (`any`/variables no usadas).
- **Playwright**: NO ejecutado — sigue sin resolverse el bloqueo de
  migración Prisma del entorno (campo OC, ver sección de bloqueo abajo), que
  impide levantar el flujo de creación de referencia contra un backend
  funcional. Pendiente de correr una vez resuelto ese bloqueo.
- **Bloqueo de migración Prisma**: sigue abierto, sin cambios en este cierre
  (fuera del alcance de esta sesión — requiere decisión humana sobre el
  entorno, ver sección debajo).

## Bloqueo: migración Prisma (tarea 3, campo OC)
`npx prisma migrate dev --name add-purchase-order-to-reference` falla con
P3015: existen dos carpetas de migración vacías y NO versionadas en git
(`prisma/migrations/20260710180000_add_reference_client_purchase_order` y
`prisma/migrations/20260710220000_sp04_expediente_document_metadata_checklist_history`)
que SÍ figuran como aplicadas en `_prisma_migrations` de la DB de este
entorno — la columna `client_purchase_order` ya existe físicamente en
`references` (confirmado vía `information_schema.columns`), pero falta el
`migration.sql` correspondiente en el historial versionado.

Conforme a la regla dura del proyecto (nunca escribir SQL de migración a
mano), NO se generó el archivo faltante ni se corrió `migrate resolve`. En
su lugar, el campo Prisma se mapeó (`@map`) a la columna ya existente para
que el ORM funcione en runtime sin duplicar la columna. **Se requiere
decisión humana** sobre cómo reconciliar el historial de migraciones de
este entorno (parece un remanente de otra sesión/sub-plan en paralelo,
posiblemente SP-04, que dejó migraciones a medio generar). Una vez resuelto
el historial, sí correr la CLI de Prisma para dejar la migración de OC
correctamente versionada.

## Verificación
- **Back**: `npx tsc --noEmit`, `eslint`, `jest src/references` (21/21 tests)
  — sin errores nuevos.
- **Front**: `npx tsc --noEmit` y `eslint` sobre los archivos tocados — 0
  errores (solo warnings preexistentes de `any`/variables no usadas).
- **Playwright**: NO ejecutado — no se confirmó un dev server corriendo y el
  backend tiene el bloqueo de migración arriba descrito. Pendiente de correr
  una vez resuelto el historial de migraciones y con el entorno levantado.

## Estado del sub-plan
Tareas 1, 2, 5 completas; tarea 4 completa y validada por el usuario
(2026-07-12); tarea 3 completa en código, con migración bloqueada
(infraestructura, sin resolver en este cierre). Sub-plan queda 🟡 solo por
ese bloqueo de migración, ajeno al código de este sub-plan.
