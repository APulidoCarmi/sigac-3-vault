# Manifiesto SP-17: Bandeja de Entrada (Inbox)

Sub-plan: [[2026-07-10-refactor-flujo-ejecutivo-sp17-bandeja-entrada]].

## Estado: 🚧 BLOQUEADO en fase de diseño (2026-07-12)

No se escribió código. Ramas creadas por convención (`refactor/customs-operation-sp17`,
encadenadas desde `refactor/customs-operation-sp16` en ambos repos) pero sin
diff propio — solo cargan el diff acumulado de Fase 1-3.

## Diagnóstico: D1 del sub-plan está desactualizado/inválido en 3 de sus 4 piezas

El D1 dice "reusa el tablero primario de SP-01" + "crea" tres secciones dando
a entender que son extensiones de datos ya existentes. La investigación en
código (ambos repos, graphify + lectura de fuente) muestra que **la mayoría
de las señales de negocio que el Inbox necesita mostrar no existen como datos
persistidos ni consultables hoy** — no es un gap de UI, es ausencia de modelo
de dominio. Detalle por pieza:

### 1. Tabs "Referencias en curso": automático / temporal / avanzado
**No existe ningún campo/enum en ningún repo** que represente esta
clasificación de 3 valores. Búsqueda exhaustiva en `prisma/schema.prisma`,
DTOs y servicios de `operations`/`references` (es/en): nada. Lo único
parecido es `CustomsRegime` (catálogo genérico `code/name/category/type`,
sin valores poblados que correspondan) y los campos legacy `regimen`/
`customsRegimeOld` (clave SAT de régimen, no un "tipo de despacho").

**Riesgo de falso amigo detectado:** `DispatchMonitor.tsx`
(`carmi-digital/app/(customerPortal)/customs-operation/components/DispatchMonitor.tsx`)
dice *"Seguimiento en tiempo real del flujo de despacho con Temporal"* —
esto es **Temporal.io** (motor de workflows), sin relación con un "régimen
temporal" aduanero. Si el sub-plan asumió esa lectura, la premisa es errónea.

No hay decisión de qué significan estos 3 valores (¿tipos de tramitación
Anexo 22? ¿modo de captura? ¿algo del Documento de Entendimiento no
levantado en el inventario de código?) — requiere entrevista de producto,
no una implementación a ciegas.

### 2. "Cartas sin firmar N días del cliente"
No existe ningún tipo de documento "carta de instrucciones/autorización" con
firma del cliente. `Referencedocumenttype` (enum Prisma) tiene `CARTA_PORTE`
(bill of lading terrestre — documento distinto). El único rastro de "firma"
es un ejemplo de forma libre en `create-reference-document.dto.ts:127`
(`requirements: { requiresSignature: true, ... }`, JSON sin estructura,
no queryable). No hay campos `sentAt`/`signedAt` en ningún modelo.
Construir esto requiere: decidir el tipo de documento, agregar columnas
Prisma (migración CLI) y decidir la regla de "N días" (¿desde cuándo? ¿quién
la parametriza?) — decisión de producto, no de implementación.

### 3. "Guías llegadas sin identificar" (ligar o crear referencia)
**No existe absolutamente nada.** `Shipment.referenceId` es nullable pero
ningún flujo crea/lista shipments sin referencia intencionalmente. El único
punto de entrada real es `src/email-documents/**` →
`BolReconciliationService.reconcileBol`
(`src/email-documents/bol-reconciliation.service.ts:118`): cuando el BOL
extraído por Zeus no matchea ningún shipment existente, el código hace
`logger.warn(...)` **y descarta el dato** — no persiste nada, no hay tabla
ni endpoint para reconocerlo después. Construir la funcionalidad que pide el
sub-plan (mostrar la guía a todos, reconocerla, ligarla o crear referencia)
exige diseñar un modelo nuevo (`UnidentifiedWaybill` o similar) que capture
ese caso `no match` en vez de perderlo, más el flujo de vinculación/creación
— esto es una feature greenfield completa, no una vista sobre datos
existentes.

### 4. "Auditoría de CEUS" (folios faltantes)
"CEUS" no existe como módulo en ningún repo (solo aparece como sinónimo
textual de "Zeus" en comentarios de SP-04). Lo que sí existe y es
consultable: `GET /references/:id/documents/checklist` (SP-04, checklist de
documentos por referencia individual) y campos de folio dispersos por
modelo (`coveFolio`, `eDocumentFolio`, `manifestationFolio`,
`referenceFolio` — cada uno en su propio modelo, sin relación entre sí).
**No hay ningún endpoint agregado tipo dashboard** que junte "lo que falta"
de muchas referencias a la vez — habría que construir esa capa de
agregación desde cero cruzando checklist + `glosaStatus` + los folios
sueltos. Es factible sin decisión de producto adicional (a diferencia de
los 3 puntos previos), pero es trabajo de diseño técnico no trivial que no
estaba dimensionado en el D1 ("lo que falta: folios, etc." asume que el dato
agregado ya existe).

## Lo que SÍ es viable tal cual (piezas confirmadas)

- **Tablero primario de SP-01** (`ReferenceBoard.tsx`) es reusable/extensible
  como base — ya deja anotado en su propio código (comentario líneas 18-21)
  que "priorización real queda para el Inbox completo (SP-17)".
- **"Por identificar"** → `ReferenceStatus.DRAFT`, patrón ya usado en SP-01.
- **"Previo"** → modelo `Previo` (SP-09) totalmente consultable
  (`Previo.status IN (REQUESTED, IN_TABLE)` = esperando previo). Solo falta
  un endpoint agregado (listar referencias con Previo pendiente), trivial
  dado el modelo existente.
- **"Arribo"** → ya existe `GET /shipments/pre-arrival`
  (`src/shipments/controllers/shipment-pre-arrival.controller.ts`),
  directamente reusable para esta columna.
- **"Clasificación"**: existe el módulo `classification` (no explorado en
  profundidad porque el bloqueo ya está confirmado por los otros 3 puntos;
  no cambia la conclusión).

## Por qué se bloquea y no se improvisa

Instrucción explícita de mi tarea: "Si al leer el sub-plan y el código real
encuentras que su D1 está desactualizado o inválido, NO improvises: detente,
documenta el diagnóstico... y repórtalo como tal." 3 de las 4 piezas del
Inbox (tabs de régimen, cartas sin firmar, guías sin identificar) requieren
decisiones de producto no tomadas (qué significan, qué reglas de negocio
llevan, qué modelo de datos exacto) antes de poder escribir una migración
Prisma o un endpoint — construirlas a ciegas violaría también el estándar
del repo de "nada de atajos/parches" y probablemente generaría rework como
ya pasó con SP-16 (bloqueado por la misma razón: diseño insuficiente).

## Recomendación para desbloquear (para la próxima entrevista `/plan`)

Antes de reabrir este sub-plan, decidir con el usuario:
1. Qué son exactamente "automático/temporal/avanzado" (fuente: revisar
   Documento_Entendimiento / Inventario_Pantallas_v3 con más detalle, o
   preguntar directo — la reunión de origen M9 no lo especifica en el texto
   del sub-plan).
2. Si "cartas sin firmar" es una feature nueva completa (tipo de documento +
   flujo de firma) o si en realidad ya existe en SIGAC 2 con otro nombre que
   no se mapeó.
3. El alcance real de "guías sin identificar": ¿es solo BOL vía
   `email-documents`, o también AWB aéreo / otros? ¿Quién crea la referencia
   nueva desde ahí (mismo wizard de SP-02)?
4. Si la "Auditoría de CEUS" agregada es aceptable como capa nueva construida
   sobre checklist+folios existentes (viable sin decisión adicional), o si
   el usuario esperaba una integración Zeus/CEUS en vivo (no existe, ver
   manifiesto SP-04).

## Archivos tocados
Ninguno (código). Solo este manifiesto y las ramas creadas por convención
(sin commits, sin diff).

## Gates
No aplica — no se escribió código.
