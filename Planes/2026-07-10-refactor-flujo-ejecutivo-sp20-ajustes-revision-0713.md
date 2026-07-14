# Plan: SP-20 — Ajustes de bandeja, expediente, DGO, movimientos y tickets (revisión 07-13)

> Sub-plan post-cierre del paraguas [[2026-07-10-refactor-flujo-ejecutivo]] (mismo
> patrón que [[2026-07-10-refactor-flujo-ejecutivo-sp19-configuracion-pedimento-en-dgo|SP-19]]),
> agregado a partir de una revisión dirigida de producto contra
> [[2026-07-13 - Revisión de pantallas]]. Cabe en una sesión limpia de
> `/implementa`.

## Contexto

Origen: reunión [[2026-07-13 - Revisión de pantallas]] entre Ángel (desarrollo)
y German Castro (producto), continuación directa de
[[2026-07-07 - Revision de pantallas flujo operación]] (ya cubierta por SP-19).
El paraguas completo (SP00-SP19) está cerrado; este sub-plan corrige/ajusta
piezas de varios sub-planes ya cerrados (SP-01, SP-04, SP-05, SP-07, SP-17) a
la luz de la nueva revisión de pantallas.

**Hallazgo relevante al arrancar este plan:** en ambos repos
(`carmi-digital`, `carmi-odin-api-v2`, rama compartida
`feat/2026-07-13-rediseno-interfaz`) ya existe un diff sin commitear que
intentó adelantar parte de estas decisiones (p. ej. un comentario fechado
"REORDENAMIENTO (2026-07-13)" en `ReferenceDetailShell.tsx` con un nuevo
orden de tabs). El usuario confirmó que esa implementación parcial **está
mal hecha y debe revertirse** antes de reimplementar limpio según este plan
— no se reutiliza tal cual.

Área de código: front `carmi-digital/app/(customerPortal)/references/**`
(shell del detalle, tabs de Expediente/DGO/Movimientos/Instrucciones,
`InboxTabs.tsx`, listado de referencias) + navegación principal (menú
lateral de la app); back `carmi-odin-api-v2` (`reference-documents`,
`tickets`, `shipments`/tipo de movimiento por tráfico).

### Inventario de código D1 — hallazgos de la exploración para este plan

1. **Bandeja de entrada** vive hoy en una página standalone `app/(customerPortal)/inbox/page.tsx`
   con `InboxTabs.tsx` (7 secciones: Alertas, Accionable ahora, Curso
   automático, Esperando a terceros, Por identificar, Guías sin identificar,
   Documentos que requieren atención — ya renombrado). El listado de
   referencias (`ReferencesClient.tsx`, SP-01) es una pantalla aparte con sus
   propios tabs. **No están fusionados**, pese a que M9/SP-01 ya apuntaba a
   esa fusión.
2. **Shell del detalle** (`ReferenceDetailShell.tsx`, SP-03) tiene hoy (en el
   diff sin commitear, a revertir) el orden: Resumen, Movimientos, Expediente
   Aduanero, Instrucciones, Citas, Operaciones, Recinto, Previo, DGO — con
   "Instrucciones" y "Citas" marcados con nota de "pendiente confirmación con
   Germán".
3. **Crear factura manual** vive solo en `ReferenceDGOTab.tsx` /
   `ReferenceMerchandiseTab.tsx` (vía `InvoiceFormModal`). El tab de
   Expediente Aduanero (`ReferenceDocuments.tsx`) no tiene esa acción.
4. **Expediente Aduanero** (`ReferenceDocuments.tsx`) muestra hoy 2 de las 3
   partes por documento: archivo original y estado de glosado
   (`doc.glosaStatus`). **No muestra los datos extraídos por CEUS/Zeus** (no
   hay campo/UI para ello). El checklist de documentos requeridos/faltantes
   **ya existe en backend** (`ReferenceDocumentsService.getChecklist()` /
   `getPendingPanel()`, `GET .../checklist`, `GET .../pending-panel`) pero
   solo se consume desde el Inbox (`PendingPanelSection.tsx`), no desde este
   tab.
5. **Bug confirmado de tipo de movimiento**: al crear una referencia y elegir
   tráfico marítimo/aéreo/carretero, el selector de tipo de movimiento no
   muestra correctamente las opciones según el tráfico elegido (confirmado
   por el usuario). En código, `ReferenceShipments.tsx` decide el
   sub-componente por `reference.trafficType?.code`, con fallback silencioso
   a `ReferenceShipmentsTerrestre` cuando el código no matchea
   AEREO/MARITIMO.
6. **Tickets** ya es una entidad separada en backend (`prisma.Ticket`,
   `src/tickets/**`) y el tab "Instrucciones" del front ya consume ese
   endpoint — no hay entidad duplicada. Falta la regla de cardinalidad:
   **1 ticket : N referencias, pero 1 referencia : máximo 1 ticket
   simultáneo** — no está validada hoy.

### Origen (trazabilidad)

[[2026-07-13 - Revisión de pantallas]] (esta revisión), continuando
[[2026-07-07 - Revision de pantallas flujo operación]] (M9, ya resuelta en
SP-01/SP-03/SP-04/SP-05/SP-17/SP-19).

## Decisiones tomadas (entrevista de este plan)

1. **Revertir antes de reimplementar.** El diff parcial de hoy en
   `ReferenceDetailShell.tsx` (y cualquier otro archivo tocado hoy en esa
   línea) se revierte; este plan reimplementa desde el estado ya cerrado del
   paraguas (post SP-19), no sobre el intento parcial.
2. **Bandeja de entrada deja de ser pantalla propia.** Se elimina la ruta
   `/inbox`. El menú de navegación principal queda solo con **Referencias** y
   **Operaciones**. Dentro de la pantalla de **Referencias** (listado), dos
   tabs: **"Bandeja de entrada"** (las cards/tablero, reusando la lógica de
   `InboxTabs.tsx`) y **"Tabla"** (la vista tipo SIGAC 2 ya existente de
   SP-01). Se mejora la UX/UI de cómo se muestran las secciones de la bandeja
   (Alertas, Accionable ahora, En curso automático, Esperando a terceros, Por
   identificar, Documentos que requieren atención, Guías sin identificar).
3. **Orden de tabs del detalle de referencia: Expediente Aduanero antes que
   DGO** (DGO al final, como fuente de verdad ya glosada). **"Instrucciones"
   y "Citas" se mantienen ambos tabs tal cual, sin resolver la duda de
   producto** — queda como riesgo abierto (ver más abajo), no se decide en
   este plan.
4. **Mover "crear factura manual" del DGO al Expediente Aduanero.** La
   acción de dar de alta una factura manualmente pasa a vivir en
   `ReferenceDocuments.tsx`; cuando hay más de un DGO en la referencia, el
   usuario selecciona a cuál se vincula la factura recién creada/glosada.
5. **Expediente Aduanero: completar la vista a 3 partes por documento**
   (archivo / datos extraídos / resultado de glosado) y **conectar el
   checklist de documentos requeridos/faltantes** (ya existe en backend) a
   este tab, no solo al Inbox — huecos vacíos visibles para lo que falta.
6. **Corregir el bug de tipo de movimiento por tráfico.** El selector debe
   mostrar las opciones correctas según el tráfico de la referencia
   (terrestre/marítimo/aéreo) sin caer en un fallback silencioso a terrestre.
7. **Agregar restricción 1 referencia : 1 ticket.** Una referencia no puede
   asociarse a más de un ticket simultáneamente (validación en creación de
   referencia y al asociar/desasociar después); un ticket sí puede tener
   varias referencias.
8. **Fuera de este plan:** la pieza 1 de SP-17 (tabs "automático/temporal/
   avanzado" del Documento de Entendimiento original) sigue pendiente aparte,
   sin relación con la reunión de hoy — no se toca aquí.

## Fuera de alcance

- Tabs "automático / temporal / avanzado" de la bandeja (pieza 1 de SP-17,
  pendiente transversal aparte).
- Resolver si "Instrucciones" y/o "Citas" deben eliminarse o fusionarse
  (pendiente de confirmación explícita de German — riesgo abierto, no
  decisión de este plan).
- Definir nombre final del movimiento inicial de marítimo ("revalidar" u
  otro) — pendiente de contrastar contra el documento de entendimiento
  existente, fuera del bug puntual de selector que sí se corrige aquí.
- Cualquier cambio a `components/operations` / `customerPortal/operations`
  (fuera de alcance de todo el paraguas).

## Pasos

- [x] **Revertir el diff parcial de hoy** en `ReferenceDetailShell.tsx` (y
  cualquier archivo relacionado con el reordenamiento fechado
  "2026-07-13") en `carmi-digital`, dejando el repo en el estado cerrado
  post-SP-19 antes de continuar.
- [x] **Eliminar la ruta `/inbox`** y su entrada en el menú lateral; dejar el
  menú principal con solo Referencias y Operaciones.
- [x] **Fusionar bandeja + listado**: agregar tab "Bandeja de entrada" (con
  la lógica/estado de `InboxTabs.tsx` movida/reusada) y tab "Tabla" (SP-01)
  dentro de la pantalla de Referencias; mejorar la UX/UI de las secciones de
  la bandeja.
- [ ] **Reordenar el shell del detalle** de referencia: Expediente Aduanero
  antes de DGO (DGO al final); mantener Instrucciones y Citas como están.
- [ ] **Mover la acción de crear factura manual** del tab DGO/Mercancía al
  tab Expediente Aduanero, con selección de DGO destino cuando hay 2+.
- [ ] **Agregar la vista de datos extraídos** (3ra parte) por documento en
  `ReferenceDocuments.tsx` y **conectar el checklist de pendientes**
  (`getChecklist`/`getPendingPanel`, ya existentes en backend) a este tab.
- [ ] **Diagnosticar y corregir el bug de tipo de movimiento por tráfico**:
  revisar por qué el selector no filtra bien según `trafficType` al crear
  referencia/movimiento en marítimo/aéreo/carretero, y eliminar el fallback
  silencioso a terrestre en `ReferenceShipments.tsx`.
- [ ] **Agregar validación 1 referencia : 1 ticket** en backend
  (`src/tickets/**`) y reflejarla en el front (creación de referencia y
  asociación de ticket existente).

## Riesgos y side effects a vigilar

- **Revertir mal el diff de hoy** podría arrastrar cambios legítimos del
  cierre del paraguas si no se aísla bien lo tocado específicamente hoy —
  revisar con `git diff`/`git log` acotado antes de revertir, no un
  `git checkout` amplio.
- **"Instrucciones" y "Citas" quedan sin resolver**: cualquier trabajo futuro
  que dependa de esa decisión (p. ej. quitar uno de los dos tabs) debe
  esperar confirmación explícita de German — no asumir en el camino.
- **Fusión bandeja+listado** toca navegación global (menú lateral): validar
  que no rompa otros flujos que enlazan directo a `/inbox`.
- **Checklist de documentos** ya vive en backend pero pensado para el Inbox;
  confirmar que el shape de `getPendingPanel()` sirve tal cual para
  Expediente Aduanero o si necesita un endpoint/vista ligeramente distinta.

## Criterios de verificación

- Puertas estáticas (lint/typecheck/tests) verdes en ambos repos (`/verify`).
- Playwright MCP, revisando consola del navegador:
  - Navegar a Referencias → confirmar tabs "Bandeja de entrada" y "Tabla"; ya
    no existe ruta `/inbox` ni entrada de menú.
  - Detalle de referencia → confirmar orden de tabs (Expediente antes de
    DGO) y que Instrucciones/Citas siguen visibles sin cambios de contenido.
  - Expediente Aduanero → subir/ver un documento y confirmar que se ven las
    3 partes (archivo, datos extraídos, glosado) y el checklist de
    faltantes; crear una factura manual desde este tab y confirmar que
    pregunta a qué DGO se vincula si hay 2+.
  - Crear una referencia marítima y otra aérea → confirmar que "crear
    movimiento" ofrece las opciones correctas por tráfico (no cae a
    terrestre).
  - Intentar asociar una referencia ya vinculada a un ticket a un segundo
    ticket → confirmar que se bloquea.
