# Sesión 2026-08-04 — Análisis causa raíz: documento de factura no aparece en Expediente Aduanero

Relacionado: [[2026-08-02 - Implementación facturas-en-guías sp02 y hallazgo documento de factura]], [[2026-08-02-documento-oficial-factura-pdf]]

## Qué se pidió

El usuario reportó que al subir la Factura Comercial (vía `InvoiceDocumentUpload`,
implementado en el sub-plan 03 de la sesión 2026-08-02), el documento debería
aparecer en Expediente Aduanero clasificado con el tipo de documento **94 —
"Factura o lista de empaque comercial"**, pero no aparece. Pidió solo análisis,
sin tocar código.

## Confirmación: no es un bug nuevo

Este punto ya estaba identificado como fuera de alcance en la sesión del
2026-08-02 (ver nota relacionada, punto 1 de las observaciones del usuario) y
corresponde a las tareas ya existentes en `Tareas.md` sección "Expediente
Aduanero":
- "Permitir cargar el documento de una factura en el DGO de forma que quede
  vinculado a esa factura... y aparezca también en Expediente Aduanero..."
- "**PRIORIDAD.** Al subir una factura en el DGO, integrar la factura al mismo
  objeto de expediente asociado al movimiento/referencia..."

Lo nuevo de hoy es la **causa raíz técnica concreta**, investigada a fondo.

## Causa raíz (con evidencia file:line)

Son dos tablas completamente separadas — no falta un campo, falta el paso
completo de creación del registro cruzado:

- **Lo que escribe `InvoiceDocumentUpload`**: `documents` (vía `POST
  /storage/upload`, `storage.controller.ts:55`) + `invoice_images` con
  `category: MAIN` (`InvoiceImagesService.syncInvoiceImages`, invocado desde
  `invoices.service.ts:820`). Cero referencia a expediente.
- **Lo que lista Expediente Aduanero**: tabla `reference_documents`
  (`schema.prisma:3567-3625`), con su propio enum `documentType`
  (`Referencedocumenttype`: `FACTURA`, `LISTA_EMPAQUE`, etc.) y FK a
  `referenceId`/`operationId`/`shipmentId`. La consulta que alimenta la UI es
  `reference-documents.service.ts` método `findByReference` (líneas 129-161):
  simplemente lee lo que ya exista ahí.
- El catálogo numérico de tipos de documento (código 94 = "FACTURA O LISTA DE
  EMPAQUE COMERCIAL") vive aparte en `document_types`
  (seed: `prisma/seeds/document-types.seed.ts:234`).
- **No existe ninguna FK ni llamado entre los dos mundos** (confirmado por
  grep: `invoice-images.service.ts` / `invoices.service.ts` no referencian
  `ReferenceDocument` en ningún punto).
- **Precedente encontrado pero muerto**: `references.service.ts:5612`
  (`createProforma`) tiene un bloque que sí crea el documento y luego
  `tx.referenceDocument.create({ documentType: 'PROFORMA', ... })` — pero está
  **comentado**, no es código vivo. No hay ningún path activo en el sistema
  que auto-registre un upload de archivo (factura ni packing list) como fila
  de `reference_documents`.

## Ambigüedad sin resolver

El código 94 dice "Factura **o** lista de empaque comercial", pero el enum
interno `Referencedocumenttype` los separa en `FACTURA` vs `LISTA_EMPAQUE`.
Ningún código actual resuelve a cuál mapear un upload de factura — habrá que
decidirlo antes de implementar (probablemente `FACTURA`, a confirmar con
negocio si el catálogo de 94 en Expediente Aduanero usa ese enum directamente
o un campo/código numérico distinto).

## Pendientes

- Decidir con negocio/Germán si esto se implementa junto con las dos tareas
  ya existentes de "Expediente Aduanero" en `Tareas.md`, o si se separa en un
  sub-plan propio dentro del paraguas de facturas.
- Si se retoma: usar `/plan` para diseñar el flujo antes de tocar código —
  incluye decidir el mapeo `FACTURA` vs `LISTA_EMPAQUE`, y si el registro de
  `reference_documents` se crea en el mismo momento del upload (`invoice
  document upload`) o de forma diferida/reconciliable.
