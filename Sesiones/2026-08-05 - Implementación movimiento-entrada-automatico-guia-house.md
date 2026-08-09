# Sesión 2026-08-05 — Movimiento de entrada automático por Guía House

Relacionado: [[Planes/2026-08-05-movimiento-entrada-automatico-guia-house]],
[[2026-08-01 - Ingesta reunión Revisión de Movimientos - Guía]],
[[TransformInterceptor global - nunca envolver manualmente la respuesta en controllers]],
[[useUser Context vs Redux selectUser - confiabilidad de sesion]]

## Qué se trabajó

Ejecución completa (vía `/implementa`) del plan **Movimiento de entrada
automático por Guía House** (tarea 2 de la reunión de revisión de
Movimientos-Guía, marcada PRIORIDAD): al dar de alta una guía House
(Querétaro, flujo aéreo), se genera automáticamente su Shipment esqueleto
(movimiento de entrada), sin intervención manual.

Las 8 tareas del plan quedaron completas, pero la sesión se extendió mucho
más allá del plan original porque cada tarea de verificación destapó un bug
real, nunca antes ejercitado de punta a punta:

1. **Migración Prisma** `guiaId` único en `Shipment` (CLI, no SQL a mano —
   el entorno no tenía TTY para `migrate dev`, el usuario lo corrió él
   mismo en su terminal).
2. Helper `createShipmentSkeletonForGuia` + enganche en `create()`/
   `createBulk()` de `GuiasService`, con rollback si no se resuelven
   `TransportMode`/`ShipmentStatus`.
3. Propagación de `referenceId` en `linkGuiasToReference()` sin duplicar
   Shipment.
4. `GET /shipments/reference/:id` ampliado con la relación `Guia`.
5. Frontend: `ReferenceShipmentsAereo.tsx` reescrito para mostrar el listado
   real de guías con sus movimientos.

A partir de ahí, cada vez que el usuario probaba el flujo real aparecía un
bug preexistente nunca detectado (ver Aprendizajes).

## Commits relevantes

- `carmi-odin-api-v2` — `607650ea` *"feat(guias): generar movimiento de
  entrada automático al crear guía House"* → pusheado a
  `feat/movimiento-entrada-automatico-guia-house` (sin PR abierto).
- `carmi-digital` — `fedd2f76` *"feat(guias): cablear movimientos de entrada
  por Guía House en Referencias"* → pusheado a la misma rama (sin PR
  abierto).

## Decisiones (con su porqué)

- **Guia y Shipment se mantienen como tablas separadas**, no se fusionan.
  El usuario preguntó explícitamente si convenía eliminar `Guia` y usar
  `Shipment` directamente. Análisis: `Guia` es un flujo de clasificación
  aduanal (alta masiva, régimen/clave de pedimento, `GuiaStatus`,
  `GuiaStatusHistory`) ortogonal al seguimiento físico de almacén
  (`Shipment`). Fusionar mete ~15 columnas y un enum de status nuevos en un
  modelo que ya comparten terrestre/marítimo. El riesgo real
  ("dos tablas con campos duplicados sin garantía de sincronía") se resolvió
  con sincronía automática, no fusionando modelos.
- **`Shipment` esqueleto se llena con datos reales de la Guía** (pieces,
  grossWeight/Unit, eta/arrivalDate) en vez de dejarlo vacío ("sin snapshot"
  del diseño original). Se reabrió esa decisión porque se encontraron ~10
  consumidores reales (`receipt-creation.service.ts`, Control Tower,
  Verification, `ShipmentCard`) que leen esos campos crudos sin ningún
  fallback a Guía — dejarlo vacío rompía silenciosamente medio sistema.
  `GuiasService.update()` resincroniza esos campos en cada edición de la
  guía.
- **Editar un movimiento de guía edita la Guía, no el Shipment** (mismo
  criterio para subdivisiones: heredan los datos del Shipment padre).
- **Vinculación de DGO por clave de pedimento, incremental y sin
  sobreescribir**: `linkGuiasToReference` se corrigió para no asumir que la
  primera clave de cada llamada es "la primera de la referencia" — ahora
  reutiliza el DGO existente con esa clave o crea uno nuevo, nunca
  sobreescribe el default ya asignado.
- **"Crear Operación" se quita de la vista de Guías/Movimientos aéreo**
  (siempre, no condicional): las operaciones solo se crean desde DGO.
- **Reutilizar componentes existentes en vez de construir ad-hoc**:
  `GuiaFormModal` (edición y creación, con cliente bloqueado al de la
  referencia y clave de pedimento auto-vinculada a los DGO existentes) y
  `GuiaInvoicesModal` (facturas) se reutilizan tal cual desde
  `ReferenceShipmentsAereo.tsx`, en vez de duplicar UI.

## Aprendizajes / errores a no repetir

- **Doble-wrap de respuesta por `TransformInterceptor` global** — ver
  [[TransformInterceptor global - nunca envolver manualmente la respuesta en controllers]].
  Encontrado en 4 controllers (arreglados) + 14 más sin tocar (documentado
  como deuda).
- **`useUser()` Context menos confiable que Redux `selectUser`** — ver
  [[useUser Context vs Redux selectUser - confiabilidad de sesion]].
- **`packageUnits` obligatorio en `CreateSubdivisionShipmentDto` sin ningún
  input en `SubdivisionForm.tsx`** — bloqueaba TODA subdivisión (terrestre
  incluida) con 400, invisible hasta que se pudo completar una subdivisión
  por primera vez en esta sesión.
- **Patrón recurrente de la sesión**: casi todos los bugs encontrados eran
  de la misma naturaleza — código que "existe" pero nunca se ejercitó de
  punta a punta con datos reales (rutas nunca alcanzadas, campos nunca
  poblados, formularios con campos faltantes). El plan de guías fue la
  primera vez que el flujo aéreo se probó real desde alta hasta subdivisión.
- **`npx prisma migrate dev` requiere TTY interactivo** — en un entorno sin
  TTY (como esta sesión) falla con "non-interactive environment detected";
  no hay forma de forzarlo por stdin, hay que pedirle al usuario que lo
  corra en su propia terminal.

## Pendientes

- Abrir PRs de ambos repos (el usuario no lo pidió aún explícitamente).
- Sub-plan aparte para el doble-wrap en los 14 controllers restantes.
- Considerar si "Recinto" (`ReferenceWarehouse.tsx`) y "Solicitar Previo"
  (`RequestPrevioModal.tsx`) — que tenían el mismo bug de comparación de
  código de tráfico SAT, ya corregido esta sesión — necesitan verificación
  end-to-end aparte.
