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
- [x] Definir la estructura del menú lateral (secciones primarias vs secundarias)
      con el volumen real de secciones de D1: D1 tenía 11 tabs (`overview`,
      `inventory`, `shipments`, `globalExpenses`, `legalConfig`, `instructions`,
      `documents`, `appointments`, `warehouse`, `operations`, `timeline`); se
      listan las 10 restantes (`timeline` absorbida en Resumen) en una sola
      columna — el lateral tiene espacio de sobra, ya no hace falta el split
      primary/dropdown de las Tabs.
- [x] Construir el shell lateral reutilizable y migrar `ReferenceTabs.tsx` a él
      (renombrado `ReferenceDetailShell.tsx` vía `git mv`, único importador
      actualizado en `references/[id]/page.tsx`), preservando el ruteo por
      `?tab=` querystring (mismo mecanismo `useSearchParams`/`router.replace`
      que las Tabs; si la URL trae un valor no navegable como `timeline` cae a
      `overview` por defecto).
- [x] Resolver Resumen (#4): Línea de tiempo incorporada DENTRO de Resumen
      (se sigue M9, fuente de diseño más reciente) en la 3ª columna del grid
      (`lg:col-span-1`) que en D1 estaba sin usar, reusando
      `ReferenceTimeline.tsx` (API `GET /references/:id/timeline`) tal cual.
- [x] Corregir breadcrumbs: se quita el segmento intermedio del cliente
      (enlazaba a la misma ruta que "Referencias", redundante) en
      `ReferenceHeroHeader.tsx` → "Referencias › referencia".

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
✅ Cerrado (2026-07-11, rehecho desde cero tras el descarte del 2026-07-11).
Rama `refactor/customs-operation-sp03` en `carmi-digital` (primer sub-plan en
tocar el repo tras el reinicio, partió de `test`; se auditó el código antes de
empezar y se confirmó que no quedaba ningún residuo de la implementación
descartada — D1 seguía con `ReferenceTabs.tsx`/Tabs de shadcn intactas). Sin
commits (working tree queda para revisión humana, por instrucción del flujo
`/implementa`). Archivos tocados: `references/components/ReferenceDetailShell.tsx`
(nuevo, renombrado de `ReferenceTabs.tsx` vía `git mv`), `references/[id]/page.tsx`
(importador actualizado), `references/components/tabs/ReferenceOverview.tsx`
(línea de tiempo embebida), `references/components/ReferenceHeroHeader.tsx`
(breadcrumb corregido). Gates estáticos verdes: `tsc --noEmit` sin errores
nuevos (los 4 errores restantes —`three`, `mammoth`— son preexistentes y
ajenos); `eslint` sobre los 4 archivos tocados sin errores (5 warnings, todos
preexistentes, verificado con `git stash`/diff). Playwright NO ejecutado: el
entorno usa credenciales reales (`.env`) contra infraestructura viva sin
supervisión humana — no se navegó el flujo en vivo, ver guía de pruebas.
