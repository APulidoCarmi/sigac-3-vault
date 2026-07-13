# Sub-plan SP-19: Configuración de datos de pedimento a nivel DGO

Parte de [[2026-07-10-refactor-flujo-ejecutivo]]. Sub-plan agregado
post-cierre del paraguas (2026-07-12), a partir de una revisión dirigida por
el dueño de producto contra [[2026-07-07 - Revision de pantallas flujo operación]]
(único meet del baúl que menciona DGO) y una auditoría del código real en la
rama `refactor/customs-operation-sp16b-legacy-purge`.

## Contexto

El meet 2026-07-07 (German + Ángel) estableció "DGO como fuente única de
verdad para armar la operación" y "un DGO por factura". El código ya
implementó gran parte de esto (SP-05, SP-06): el modelo `Dgo` tiene los
campos de pedimento (`aduana`, `clavePedimento`, `patente`, `regimen`,
`destino`, heredados de `Reference` si null), el Step 0 del wizard de crear
Operación ya selecciona DGOs directamente (no movimientos — confirmado y
mantenido por decisión del dueño de producto, 2026-07-12), y el backend crea
el `Pedimento` "mínimo" confiando en que sus datos vengan del DGO vinculado.

Pero la investigación encontró que la pieza que cierra el círculo —
**configurar esos datos de pedimento una sola vez en el DGO**, en vez de
recapturarlos/dejarlos ambiguos en el wizard de Operación — nunca se
construyó del lado de UI, y quedaron 3 gaps concretos alrededor de eso.

**Decisión del dueño de producto (2026-07-12):** el Step 0 del wizard de
Operación se queda como selección directa de DGO(s) (no se reconstruye para
partir de movimientos). El vínculo Movimiento↔DGO (SP-07) sigue siendo un
dato informativo en la pestaña Movimientos, sin integrarse al wizard de
creación de Operación — no hay tarea para esto en este sub-plan.

## D1 — hallazgos verificados contra el código real

### A. Falta UI para configurar pedimento a nivel DGO
El modelo (`Dgo.aduana/clavePedimento/patente/regimen/destino`,
`prisma/schema.prisma:3562-3598`) y el endpoint
(`PATCH /dgo/:id`, `dgo.controller.ts:95`, con bloqueo `assertNotLocked` una
vez que el DGO ya originó un pedimento) ya existen y funcionan. Pero
`ReferenceDGOTab.tsx` (333 líneas) es puramente de lectura — solo muestra
"Aduana {dgo.aduana} · Clave {dgo.clavePedimento} · Patente {dgo.patente} ·
Régimen {dgo.regimen}" sin ningún formulario/drawer para editarlos. Hoy esos
campos solo se llenan si vienen heredados de `Reference` o por seed directo
en BD — no hay flujo de usuario real.

### B. El wizard de Operación mezcla datos de pedimento con datos de la operación, y tiene un bug de grupos congelados
- La pestaña "Aduanas" de `stepCustomsInfo.tsx` (líneas 564-693) tiene inputs
  editables de "Aduana de Despacho"/"Patente"/"Mandatario" como **un único
  valor compartido de toda la operación** — ambiguo si esto es dato legítimo
  de la Operación (aduana de despacho real, patente del agente que
  despacha) o un remanente de "datos de pedimento" que debería venir de
  solo lectura desde el DGO. El fallback en `page.tsx:264-271`
  (`group.formData?.customsData?.X || customsData?.Y`) sugiere que ambos
  conceptos se están mezclando hoy.
- Bug de UX: `pedimentoGroups[i].formData.customsData` (store
  `create-operation.store.ts:216-229`) sí pre-llena por grupo desde el DGO
  (`initializeGroupsFromDgos`, líneas 486-518), pero no existe ningún
  selector de grupo activo en la UI (`setActivePedimentoGroupIndex`/
  `loadGroupFormData`/`saveCurrentFormDataToGroup` — cero usos en
  componentes, confirmado por grep). Con 2+ DGOs seleccionados, solo el
  grupo `[0]` es visible/editable; los grupos 2+ quedan congelados con lo
  que trajo el DGO en el Step 0, sin poder confirmarlos ni editarlos.
- Los campos `customsKey`/`customsRegime` (clave de pedimento y régimen) ya
  no tienen inputs en el wizard (removidos correctamente, se leen implícito
  del DGO) — para esos dos campos el problema ya no existe. El problema
  real está acotado a `customsOfficeId` (aduana) y `customsPatentId`
  (patente).

### C. Fuente de verdad ambigua en el backend para validaciones de negocio
`operations.service.ts::validarValorAgregadoPorClave` (líneas 812-854) usa
`dto.pedimentos[].pedimentoCode` (el valor que mandó el wizard) en vez de
`dgo.clavePedimento` resuelto server-side desde `group.dgoId`. Si el DGO ya
tiene la clave correcta configurada (gap A resuelto), el backend debería
resolverla ahí mismo en vez de confiar en lo que reenvía el cliente.

## Fuera de alcance
- Reconstruir el Step 0 para partir de movimientos en vez de DGOs (decisión
  ya tomada: se queda como está).
- Integrar el vínculo Movimiento↔DGO (SP-07) al flujo de creación de
  Operación — sigue siendo solo informativo en Movimientos.
- Cualquier cambio al modelo `Pedimento` de despacho (SP-16) o a la
  auditoría de campos de encabezado/partida (hallazgos de la auditoría de
  completitud DGO-vs-pedimento, que es un sub-plan/decisión aparte).

## Tareas

- [ ] **Construir el formulario de configuración de pedimento en el DGO**:
  en `ReferenceDGOTab.tsx` (o su Sheet de Acciones), agregar edición de
  `aduana`/`clavePedimento`/`patente`/`regimen`/`destino`, llamando
  `PATCH /dgo/:id`. Debe respetar el bloqueo `assertNotLocked` (mostrar
  solo lectura si el DGO ya originó un pedimento, con mensaje explicativo).
- [ ] **Auditar y decidir qué campos de la pestaña "Aduanas" del wizard son
  legítimos de la Operación vs. remanentes de pedimento**: si
  `customsOfficeId`/`customsPatentId` en `stepCustomsInfo.tsx` representan
  la aduana/patente REAL de despacho de la operación (dato correcto a nivel
  operación, puede diferir del DGO en casos de consolidación), dejarlos
  pero renombrar/aclarar en UI que son de la Operación, no del pedimento.
  Si resultan ser duplicados del dato del DGO, quitarlos del wizard y
  mostrar de solo lectura lo que ya trae cada `pedimentoGroup` desde su DGO.
- [ ] **Arreglar el bug de grupos congelados**: implementar el selector de
  grupo activo en el wizard (usar `setActivePedimentoGroupIndex`/
  `loadGroupFormData`/`saveCurrentFormDataToGroup`, ya existentes en el
  store pero sin consumidor en UI) para que, con 2+ DGOs seleccionados, el
  usuario pueda ver/confirmar/ajustar los datos de cada pedimento por
  separado antes de crear la Operación.
- [ ] **Backend — resolver `pedimentoCode`/datos de pedimento server-side
  desde el DGO**, no desde el payload del cliente, en
  `validarValorAgregadoPorClave` y en cualquier otro punto de
  `operations.service.ts` que hoy confíe en `dto.pedimentos[].X` para un
  campo que el DGO ya tiene configurado.
- [ ] Verificar que crear una Operación con 2+ DGOs (de distinta
  aduana/clave si aplica) refleja correctamente cada pedimento con los
  datos de SU DGO, sin mezclarse entre grupos.

## Criterios de verificación
- Gate estático verde en ambos repos.
- Playwright: configurar aduana/clave/patente/régimen en un DGO sin
  pedimento aún (editable), confirmar que se bloquea tras firmar/generar
  pedimento; crear una Operación seleccionando 2 DGOs, confirmar que ambos
  grupos de pedimento muestran/permiten confirmar sus propios datos (no solo
  el grupo `[0]`); sin errores de consola.

## Estado
✍️ Redactado — listo para `/implementa`. Decisión de alcance (Step 0 se
queda con selección directa de DGOs) confirmada por el dueño de producto
(2026-07-12). Diagnóstico completo verificado contra meet
[[2026-07-07 - Revision de pantallas flujo operación]] y código real.
