# Sub-plan SP-04: Expediente Aduanero + glosado (Zeus/CEUS)

Parte de [[2026-07-10-refactor-flujo-ejecutivo]]. Brecha grande (D1 vs propuesta).

## Contexto

Pantalla #5 del [[Inventario_Pantallas_v3]] (🟡 modificar, brecha grande) — la
Función 1 de la Referencia: expediente digital aduanero. Origen:
[[2026-07-06 - Seguimiento de refactor]] (M8: fuente de verdad documental, portal,
mapeo docs↔perfil, historial), [[2026-07-07 - Revision de pantallas flujo operación]]
(M9: glosar TODO el expediente con trigger en cada carga, UI dinámica donde CEUS
clasifica, selección primaria/secundaria, historial a nivel expediente, borrado
lógico) y glosario "Expediente aduanero" del [[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]].
Consume (solo lectura) las reglas de **MOMP** (checklist por perfil de cliente).

## D1 — punto de partida
- **Reusa:** `references/components/tabs/ReferenceDocuments.tsx` (grid de cards,
  badge de categoría Facturas/Packing/BL/Otros, badge de estado
  Pendiente/Subido/Verificado/Rechazado, toggle E-Document VUCEM, badge rojo error
  VUCEM). APIs `GET /references/:id/documents`, `/:id/documents/checklist`;
  controller `carmi-odin-api-v2/src/reference-documents/controllers/reference-documents.controller.ts`.
- **Refactoriza:** renombrar el tab a **"Expediente Aduanero"**; convertir el grid
  de cards fijas en **UI dinámica** (N documentos en una bolsa; CEUS clasifica/tipifica);
  añadir por documento: tipo, por qué se pide (fundamento legal / perfil cliente /
  perfil número de parte), requisitos de validación, vigencia, historial, estatus
  de glosa (pendiente/glosado, fases 1-4, verde/rojo con motivo), timestamp y la
  representación digital (JSON) extraída.
- **Crea:**
  - **Checklist dinámico** armado con perfil de cliente (MOMP) + perfil del número
    de parte + operación; marca requerido/opcional y si está cargado.
  - **Motor de re-glosado con Zeus/CEUS:** cada vez que se sube o se **genera** un
    documento (COVE, MV, pedimento), se re-glosa **todo** el expediente sobre la
    representación JSON (no los PDF), validando: (a) validez legal propia, (b)
    consistencia con los demás documentos y el DGO, (c) consistencia con el **Previo**.
  - **Selección primaria/secundaria** y **ligado** de documentos del mismo tipo.
  - **Historial a nivel expediente** (logs: subido, glosado, marcado primaria,
    desactivado) y **borrado lógico** (queda para auditoría).
  - Recepción de documentos del **Portal del Cliente** (SP-18), marcados "subido por
    el cliente".

## Fuera de alcance
- Configuración de MOMP (solo se consumen sus reglas).
- Ejecución del Previo (solo se consume su resultado).
- Portal del cliente en sí (SP-18); aquí solo el punto de recepción.

## Pasos
- [ ] Renombrar tab y montar la UI dinámica (bolsa + clasificación CEUS).
- [ ] Metadatos por documento (fundamento, requisitos, vigencia, JSON, estatus glosa).
- [ ] Checklist dinámico por perfil (consumir MOMP).
- [ ] Motor de re-glosado (trigger en carga/generación) sobre JSON, 3 validaciones.
- [ ] Primaria/secundaria + ligado + historial a nivel expediente + borrado lógico.
- [ ] Bloqueo: no iniciar operación si la factura enviada tiene problema de glosa.

## Riesgos y side effects
- **Re-glosar todo en cada carga** tiene coste (CEUS) y latencia: vigilar a escala.
- Confirmar la versión del endpoint de extracción/glosa (v1 `/api/workflow/process`
  vs v3) antes de cablear.
- Depende de SP-05 (DGO) para la validación (b) contra el DGO.

## Criterios de verificación
- Gate estático verde. Playwright: subir un documento dispara re-glosado, el
  estatus de glosa se refleja (verde/rojo con motivo), primaria/secundaria y
  borrado lógico funcionan; sin errores de consola.

## Estado
📋 Por implementar.
