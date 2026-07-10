# Sub-plan SP-03: Shell del detalle de Referencia (menú lateral) + Resumen

Parte de [[2026-07-10-refactor-flujo-ejecutivo]]. **Transversal — va primero en Fase 1**:
todas las demás pantallas de detalle cuelgan de este shell.

## Contexto

Decisión de la entrevista: **migrar la navegación del detalle de Tabs a menú
lateral (M9)**. Cubre las pantallas #4 (Resumen) y #13 (Línea de tiempo) del
[[Inventario_Pantallas_v3]]. Origen: [[2026-07-07 - Revision de pantallas flujo operación]]
(estructura de pestañas, breadcrumbs a corregir, línea de tiempo dentro de
Resumen) y el glosario "Referencia" (3 funciones) del
[[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]].

## D1 — punto de partida
- **Reusa:** `references/[id]/page.tsx` (ruta de detalle); `references/components/tabs/ReferenceOverview.tsx`
  (= Resumen); `references/components/tabs/ReferenceTimeline.tsx` (API `GET /references/:id/timeline`).
- **Refactoriza:** `references/components/ReferenceTabs.tsx:214-281` (hoy Tabs de
  shadcn con valores `overview/inventory/shipments/globalExpenses/legalConfiguration/instructions/documents/appointments/warehouse/operations/timeline`)
  → **shell de menú lateral** que enmarca las mismas secciones. Corregir breadcrumbs
  ("referencias › GKN Drive › referencia" → "referencias › referencia").
- **Crea:** componente de shell lateral reutilizable (navegación + área de contenido)
  que los sub-planes de cada sección consuman.

## Decisiones tomadas
- Navegación = menú lateral (M9). Los renombres de secciones (documentos →
  Expediente aduanero; mercancía → DGO; costos globales → Gastos; configuración
  legal → Identificadores) se aplican en sus sub-planes respectivos; aquí solo el
  contenedor.

## Decisión abierta a resolver en este sub-plan
- **Línea de tiempo:** M9 la mueve *dentro de Resumen*; el Inventario #13 la deja
  como sección propia. Resolver aquí (recomendado seguir M9 por ser la fuente de
  diseño más reciente) y anotar el porqué.

## Pasos
- [ ] Definir la estructura del menú lateral (secciones primarias vs secundarias)
      con el volumen real de secciones de D1.
- [ ] Construir el shell lateral reutilizable y migrar `ReferenceTabs.tsx` a él,
      preservando el ruteo por sección.
- [ ] Resolver Resumen (#4): incorporar (o no) la Línea de tiempo dentro de Resumen.
- [ ] Corregir breadcrumbs.

## Fuera de alcance
- El contenido/rediseño de cada sección (Expediente, DGO, Movimientos, etc.): sus
  propios sub-planes.

## Riesgos y side effects
- Cambio de shell transversal: si no va primero, se re-trabaja la UI de cada
  sección. Coordinar el orden con el resto de Fase 1.

## Criterios de verificación
- Gate estático verde. Playwright: navegar el detalle por el menú lateral, todas
  las secciones cargan sin errores de consola; breadcrumbs correctos.

## Estado
📋 Por implementar.
