# Sub-plan SP-05: DGO — Datos Glosados para Operación (fuente única de verdad)

Parte de [[2026-07-10-refactor-flujo-ejecutivo]]. **Nodo crítico del refactor.**

## Contexto

Pantalla #6 del [[Inventario_Pantallas_v3]] (🔵 nueva, prioridad alta) — la Función
2 de la Referencia. **Absorbe** las pantallas #7 (Mercancía), #9 (Incrementables y
Decrementables) y #10 (Identificadores), que dejan de ser tabs propios. Origen:
[[2026-07-06 - Seguimiento de refactor]] (M8: "factura glosada" como fuente de verdad
que compara y marca discrepancias), [[2026-07-07 - Revision de pantallas flujo operación]]
(M9: DGO fuente única de verdad, un DGO con partidas que concentran todos los datos,
comparación JSON vs JSON, "acciones" por factura DGO), glosario "DGO" del
[[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]] (1 DGO = 1 pedimento, estructura y
ejemplo trabajado) y el punto abierto f#6 sobre incrementables aéreos
([[2026-07-08 - Aéreo SIGAC 3.0]]).

## D1 — punto de partida
- **No existe** un tab DGO ni el concepto en D1. Lo más cercano, a **absorber**:
  - `references/components/tabs/ReferenceMerchandiseTab.tsx` (Mercancía: facturas +
    partidas, edición inline, verificación de totales). APIs `/:id/invoices`, `/:id/items`.
  - `references/components/tabs/ReferenceGlobalExpenses.tsx` (Costos Globales →
    renombrar "Gastos" con incrementables/decrementables). API `/:id/global-expenses`.
  - `references/components/tabs/ReferenceLegalConfiguration.tsx` (Configuración Legal
    → "Identificadores"). API `/:id/identifiers(+/bulk)`.
  - Lógica de comparación JSON del paso GLOSA del despacho:
    `customs-operation/components/steps/StepPedimento.tsx` (Capturado vs; endpoints
    `/dispatch/pedimento/glosa`, `/dispatch/regenerate-pedimento`) — **fuente de
    inspiración**; el DGO la sube al nivel referencia y **reemplaza** ese paso (ver SP-16).
- **Refactoriza:** migrar el contenido de esos 3 tabs a vivir **por DGO** (drawer de
  "Acciones": editar partidas, conversión de mercancías, gastos, identificadores).
- **Crea:** el modelo y la pantalla DGO:
  - Jerarquía: **Datos Globales** (heredados de la referencia o propios por DGO) →
    **Factura(s)** → **Partida(s)**. La partida concentra todos los datos sin importar
    de qué documento salieron (número de parte, fracción, descripción, TLC a nivel
    partida, valor, PROSEC, identificadores, factor de conversión, precio).
  - **1 DGO por default**; se **separa** en N DGO si hay N claves de pedimento
    distintas. **1 DGO = 1 pedimento** (misma unidad; el DGO ya trae los campos de
    pedimento: régimen, clave, aduana, destino, observaciones, identificadores G,
    incrementables/decrementables).
  - **Identificadores e incrementables/decrementables por DGO** con selección de
    uno o varios DGOs (mismos valores o distintos; para incrementables, prorrateo
    de un monto entre varios DGOs o cantidades distintas).
  - **Incrementable por guía + auto-suma** a nivel referencia/DGO (decisión f#6:
    resuelve el Excel de consolidados aéreos).
  - Comparación **JSON capturado vs factura real**, discrepancias en rojo,
    **indicador de consistencia contra el Previo**, campos de **Manifestación de
    Valor electrónica**.
  - **Firma obligatoria** antes de operar; **glosa manual de respaldo** con checklist
    y firma cuando CEUS no esté disponible (M9).

## Decisiones tomadas
- Modelo autoritativo = el del Documento (**1 DGO = 1 pedimento**), no el "un DGO por
  factura" de M9 (que era el encuadre de discovery). Anotar la reconciliación.
- Incrementable **por guía + auto-suma** (f#6).

## Fuera de alcance
- El wizard de Operación (SP-06) y el retiro del paso GLOSA del despacho (SP-16),
  aunque dependen de este DGO.
- Ligado permiso→factura/partida y mapeo docs→pedimentos complejos (M9 lo dejó para
  después del happy path; punto abierto f#8) → fase posterior.

## Pasos
- [x] Diseñar el modelo de datos del DGO en odin (Datos Globales → Factura → Partida;
      campos de pedimento embebidos; separación por clave). Decisión de diseño (ver
      manifiesto): la "Partida" YA es `InvoiceItem` en D1 (concentra ~90 campos aduaneros
      por partida, incluye TLC/PROSEC/identificadores/valor); no se crea un modelo Partida
      nuevo — se agrega `Dgo` (Datos Globales, campos de pedimento embebidos) + `dgoId`
      nullable en `Invoice`, `GlobalExpense`, `GlobalIdentifier`. Migración generada con
      `npx prisma migrate dev --name add-dgo-and-expediente-glosado`.
- [x] Migrar Mercancía (#7) al DGO (facturas/partidas, edición inline, totales). Alcance:
      la pantalla D1 (`ReferenceMerchandiseTab`) se re-monta como Acción del DGO (ver
      manifiesto); el filtrado fino por `dgoId` en `/invoice-items` queda pendiente para
      cuando haya referencias con N>1 DGO en producción (documentado como limitación).
- [x] Migrar Gastos/incrementables (#9) e Identificadores (#10) a "Acciones" por DGO, con
      selección múltiple y prorrateo. Prorrateo implementado vía `POST
      /dgo/global-expense/:id/allocate` (conserva el monto total, crea una fila de
      `GlobalExpense` por DGO).
- [x] Incrementable por guía + auto-suma a nivel referencia. `GlobalExpense.guideNumber` +
      `GET /dgo/reference/:id/incrementables-by-guide`.
- [x] Comparación JSON capturado vs real (reusar lógica de StepPedimento), rojo en
      discrepancias, indicador de consistencia vs Previo, MV electrónica. Alcance reducido
      (ver manifiesto): comparación partidas-vs-factura implementada y expuesta en UI;
      indicador de consistencia vs Previo y MV electrónica quedan fuera — no existe todavía
      una fuente de resultado del Previo en el codebase (SP-09, fuera de alcance de SP-05).
- [x] Firma obligatoria + glosa manual de respaldo con checklist/firma.
- [ ] Happy path primero (probar Ángel/German antes de casos complejos, M9). No ejecutado:
      requiere datos reales de cliente/Playwright contra un entorno con datos de Ángel/
      Germán cargados: fuera del alcance verificable en esta sesión (ver manifiesto).

## Riesgos y side effects
- **Cambio de modelo de datos** en odin: se propaga a Operación (#21), Despacho (#25)
  y Solicitud de Fondos (#23). Diseñar el modelo con cuidado antes de la UI.
- Escala: cientos/miles de partidas ⇒ paginar/virtualizar la tabla de partidas.
- Depende de SP-04 (Expediente) para el origen de documentos glosados.

## Criterios de verificación
- Gate estático verde. Playwright: crear una referencia con 1 DGO, separarlo por
  clave en 2 DGO, capturar partidas, ver discrepancias en rojo contra la factura,
  prorratear un incrementable entre DGOs, firmar; sin errores de consola.

## Estado
✅ Implementado (2026-07-12), con las reducciones de alcance anotadas arriba (Previo/MV
electrónica, filtrado por DGO en endpoints heredados, happy path Ángel/German sin correr).
Ver manifiesto. Diff sin commitear en `refactor/customs-operation-sp05` (digital y odin)
para revisión humana.
