# Manifiesto — SP-09: Previo (consulta) + Solicitar Previo + OS&D (consulta)

Implementado 2026-07-11/12 en la rama `refactor/customs-operation-sp09` (digital y
odin), encadenada desde `refactor/customs-operation-sp08`. Nada comiteado — diff en el
working tree para revisión humana.

## Investigación previa (resumen)

- `OSDReportForm.tsx` (`components/shipment-creation/`) NO se había movido en SP-01..08:
  seguía siendo un formulario de captura completo, usado exclusivamente por
  `ShipmentCreationModal.tsx` cuando `flowType === 'OSD'`. Ese flujo se abre desde
  `ReferenceShipmentsTerrestre.tsx` (`openOSDModal`, botón sobre un movimiento INBOUND)
  — o sea, hoy el **ejecutivo** puede filar un reporte OS&D como parte de la creación de
  movimientos, exactamente lo que el sub-plan pide cambiar a solo-consulta.
- Ya existe un patrón de solo-lectura hermano en `features/osd/components/`
  (`OsdReportsList`/`OsdReportDetail`, montado en `warehouse/osd/page.tsx`) y un cliente
  API completo (`features/osd/api/osdService.ts`) — se reusó `getOsdReports` (filtro por
  `origin`/`originId`) y `getTypeConfig`/`getStatusConfig` de `osd-config.ts` en vez de
  reescribir llamadas o mapeos de color/label.
- El hallazgo más importante en backend: `glosa-engine.service.ts` (SP-04) ya traía el
  contrato `PREVIO_CONSISTENCY_VALIDATOR` con un comentario explícito "SP-09 debe
  reemplazar este stub" — mismo patrón de DI + `forwardRef` que cerró
  `DGO_CONSISTENCY_VALIDATOR` en SP-05. El manifiesto de SP-05 confirma por su lado que
  el indicador de consistencia contra el Previo quedó fuera de esa pasada por no existir
  todavía una fuente de resultado del Previo — este sub-plan la crea.
- No existía ninguna fuente real de "Previo" en el backend: el enum `Appointmenttype`
  real (schema `warehouse`) no tiene un valor `PREVIO`/`RECONOCIMIENTO` — el tipo de
  cita "PREVIO" que sí existe en `ReferenceAppointments.tsx`/`CreateAppointmentModal.tsx`
  (front) es puramente de UI, desconectado del modelo real. No se tocó ese código (fuera
  del alcance de los 3 pasos declarados); queda documentado como posible fuente de
  confusión a reconciliar en un sub-plan futuro.

## Decisiones de diseño

- **Modelo `Previo` nuevo** (1 Previo por `Reference`, no N como el DGO): `status`
  (`PENDING|REQUESTED|IN_TABLE|COMPLETED`), `requestedTarget`
  (`WAREHOUSE|CUSTOMS_CLEARANCE`), `requestedAt/By`, `movementReference` (texto libre,
  no un picker de shipments — consistente con el nivel de detalle de otros modales de
  solicitud del repo como `CreateAppointmentModal`), `notes`, `consistencyResult` (Json,
  resultado que registrará almacén/trámite al completar el Previo — la ejecución física
  es fuera de alcance de SP-09, así que este campo queda `null` en la práctica hasta que
  exista ese flujo), `completedAt/By`.
- **Destino de la solicitud calculado, no elegido por el usuario**: TERRESTRE ->
  `WAREHOUSE`; AEREO/MARITIMO -> `CUSTOMS_CLEARANCE` (trámite y despacho). Mismo
  catálogo `TransportMode`/`reference.trafficType.code` y mismo default (TERRESTRE
  cuando el tráfico aún no se ha capturado) que usa `getRecintoCopy` en
  `ReferenceWarehouse.tsx` (SP-08) — se replicó la función en el frontend
  (`RequestPrevioModal`, solo para mostrarlo antes de confirmar) y en el backend
  (`PrevioService.resolveTarget`, autoritativa).
- **Cierre del stub `PREVIO_CONSISTENCY_VALIDATOR`**: se implementó
  `PrevioConsistencyValidatorService` (mismo patrón que `DgoConsistencyValidatorService`
  de SP-05) — busca el `Previo` de la referencia del documento; si no existe o no está
  `COMPLETED`, la validación no aplica (no bloquea, igual que el stub original); si está
  `COMPLETED`, compara `consistencyResult.hasDiscrepancies`. Registrado vía DI
  (`useExisting`) en `PrevioModule`, importado con `forwardRef` en
  `ReferenceDocumentsModule` (mismo ciclo `DgoModule`⇄`ReferenceDocumentsModule` ya
  resuelto en SP-04/SP-05, replicado para `PrevioModule`⇄`ReferenceDocumentsModule`).
- **Migración**: `npx prisma migrate dev --name add-previo-domain`. `migrate status`
  confirmó estado limpio antes (422 migraciones) y después (423, sin conflictos).

## Archivos tocados (odin)

- `prisma/schema.prisma`: modelo `Previo` + enums `PrevioStatus`/`PrevioRequestTarget`;
  relación `previos Previo[]` en `Reference`.
- `src/previo/` (nuevo, módulo completo):
  - `previo.module.ts`: registra `PrevioService`, `PrevioConsistencyValidatorService` y
    el binding `PREVIO_CONSISTENCY_VALIDATOR`; importa `ReferenceDocumentsModule` con
    `forwardRef`.
  - `dtos/previo.dto.ts`: `RequestPrevioDto`.
  - `services/previo.service.ts`: `ensureForReference` (1 Previo por Reference, PENDING
    si no existe), `findByReference`, `request` (calcula destino por tráfico, bloquea
    sobre un Previo ya `COMPLETED`).
  - `services/previo-consistency-validator.service.ts`: implementación real de
    `PrevioConsistencyValidator` (contrato de `GlosaEngineService`, SP-04).
  - `controllers/previo.controller.ts`: `GET /previo/reference/:referenceId`,
    `POST /previo/reference/:referenceId/request`.
  - `services/previo.service.spec.ts` (nuevo): tests de `ensureForReference` (crea/
    reutiliza), `request` (destino por tráfico TERRESTRE vs AEREO/MARITIMO, bloqueo
    sobre `COMPLETED`).
- `src/reference-documents/services/glosa-engine.service.ts`: se agrega el contrato
  `PREVIO_CONSISTENCY_VALIDATOR`/`PrevioConsistencyValidator` (análogo al de DGO) y se
  reemplaza el stub `validatePrevioConsistency()` (antes hardcodeado
  `applicable: false`) por una llamada real al validador inyectado (con el mismo
  fallback "no aplica" si el módulo no está cableado en ese punto de la ejecución).
- `src/reference-documents/reference-documents.module.ts`: importa `PrevioModule` con
  `forwardRef`, mismo patrón que `DgoModule`.
- `src/app.module.ts`: registra `PrevioModule`.

## Archivos tocados (digital)

- `components/shipment-creation/OSDReportForm.tsx`: reescrito de formulario de captura
  a vista de consulta de solo lectura. Símbolo exportado cambia de `OSDReportForm` a
  `OsdReportView` (nomenclatura unificada "OS&D"); mismo archivo (ruta declarada en el
  sub-plan). Consulta `getOsdReports({ origin: 'SHIPMENT', originId })` y muestra por
  reporte/log: tipo (Sobrante/Faltante/Daño vía `getTypeConfig`), estatus del reporte
  (`getStatusConfig`), cantidad, responsable/notas, descripción, evidencias (enlaces a
  `OsdLogFile`) y el movimiento afectado (`shipmentCode`).
- `components/shipment-creation/ShipmentCreationModal.tsx`: usa `OsdReportView` en vez
  de `OSDReportForm` (props `parentShipmentId`/`onClose`, ya no `userId`/`onSuccess`/
  `onCancel`); copy del header para `flowType === 'OSD'` cambia de "Reportar OS&D" /
  "Reporta sobrantes, faltantes o daños" a "Consultar OS&D" / "Consulta de solo lectura:
  sobrantes, faltantes o daños".
- `app/(customerPortal)/references/components/tabs/ReferencePrevioTab.tsx` (nuevo): tab
  Previo — estatus (Badge), destino/fecha de solicitud, referencia/movimiento afectado,
  notas, y el resultado de consistencia (verde/rojo, mismo lenguaje visual que
  `ReferenceDGOTab`/`DgoComparisonPanel`) o "no disponible" si el Previo no está
  `COMPLETED`. Botón "Solicitar Previo" (deshabilitado si ya `COMPLETED`).
- `app/(customerPortal)/references/components/modals/RequestPrevioModal.tsx` (nuevo):
  modal "Solicitar Previo" — muestra el destino calculado (Badge, solo informativo),
  campos opcionales `movementReference`/`notes`, `POST
  /previo/reference/:referenceId/request` vía `useMutation` + `odinFetch`/
  `withAuthHeaders` (mismo patrón que `ReferenceDGOTab`).
- `app/(customerPortal)/references/components/ReferenceDetailShell.tsx`: nueva sección
  `previo` (ícono `ClipboardCheck`, label "Previo") después de `warehouse` ("Recinto").

## Desviaciones / reducciones de alcance (documentadas explícitamente)

- **`ReferenceShipmentsTerrestre.tsx` (SP-07, ya cerrado) no se tocó**: el botón que
  abre el modal OSD (`openOSDModal`) sigue con su copy original ("Reportar OS&D" a
  nivel de esa tabla/badge). El modal que abre ahora es de solo lectura (ver arriba),
  así que el comportamiento ya es correcto (no se puede capturar), pero el texto del
  botón en esa tabla no se actualizó — está fuera de los 3 pasos declarados del
  sub-plan y pertenece al dominio de un sub-plan ya cerrado (SP-07). Se documenta como
  deuda cosmética menor, no bloqueante.
- **`consistencyResult` del Previo permanece `null` en la práctica**: no existe todavía
  un flujo para que almacén/trámite complete el Previo (fuera de alcance de SP-09, ver
  "Fuera de alcance" del sub-plan) — el modelo y el validador están listos para
  consumirlo apenas ese flujo exista.
- **Tipo de cita "PREVIO" en `ReferenceAppointments.tsx`/modales de citas** (front): no
  se tocó ni se reconcilió con el nuevo dominio `Previo` real — desconectado del enum
  real de Prisma desde antes de este sub-plan, fuera del alcance declarado (esos
  archivos no están en D1 de SP-09). Se deja anotado para una futura limpieza de
  nomenclatura.
- **`movementReference` es texto libre**, no un selector de movimientos/shipments de la
  referencia — el sub-plan no especifica una relación estructurada y no hay un patrón
  existente de picker de movimientos reusable sin ampliar alcance.

## Verificación

- Backend: `npx tsc --noEmit` sin errores nuevos (mismos errores preexistentes en specs
  ajenos a este sub-plan: `twilio.service.spec.ts`, `seal-resolver.service.spec.ts`,
  `company-patent-signature-config.service.spec.ts`, `audited.spec.ts`,
  `momp.e2e-spec.ts`, `shipment.e2e-spec.ts` — no relacionados con `previo`/
  `reference-documents`/`dgo`/`app.module`). `npx eslint src/previo
  src/reference-documents src/app.module.ts src/dgo/dgo.module.ts` → 0 errores (3
  warnings preexistentes en `reference-documents.service.spec.ts`, no introducidos por
  este sub-plan). `npx jest` (suite completa) → 505/505 suites, 3344/3344 tests verdes
  (incluye `previo.service.spec.ts` nuevo y `glosa-engine.service.spec.ts` existente,
  que sigue verde con el stub reemplazado).
- Frontend: `npx tsc --noEmit` sin errores. `npx eslint` sobre los 5 archivos tocados →
  0 errores (7 warnings preexistentes en `ShipmentCreationModal.tsx`, ninguno en las
  líneas tocadas por este sub-plan). `npx jest` → 88/88 tests verdes (mismas 2 suites
  preexistentes/ajenas fallando por `react-i18next` faltante, no relacionadas, ya
  anotadas en el manifiesto de SP-05).
- **Playwright no se corrió**: sin `dev_url` configurado para este cliente y sin
  entorno levantado en esta sesión (mismo motivo que SP-04/SP-05/SP-08). Flujo
  pendiente de validar en la próxima sesión de `/verify`: consultar un OS&D en solo
  lectura (desde un movimiento INBOUND con reportes), ver el estatus del Previo en el
  nuevo tab, solicitarlo y confirmar que el destino mostrado coincide con el tráfico de
  la referencia (TERRESTRE → Almacén; AEREO/MARITIMO → Trámite y Despacho); revisando
  la consola del navegador.

## Estado

✅ Implementado (2026-07-12). Diff sin commitear en `refactor/customs-operation-sp09`
(digital y odin) para revisión humana. Pendiente: validación Playwright real (sin
entorno levantado en esta sesión) y las dos deudas cosméticas anotadas arriba
(copy del botón en `ReferenceShipmentsTerrestre.tsx`, tipo de cita "PREVIO" legacy).
