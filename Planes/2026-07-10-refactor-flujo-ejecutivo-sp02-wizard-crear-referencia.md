# Sub-plan SP-02: Wizard de Crear Referencia (3 pasos)

Parte de [[2026-07-10-refactor-flujo-ejecutivo]].

## Contexto

Pantalla #3 del [[Inventario_Pantallas_v3]] (🟡 modificar) — **decisión ya tomada**
(M9): el wizard baja de 4 a 3 pasos, sin forzar documentos comerciales al crear;
esa sección se mueve al detalle (al DGO, SP-05). Origen:
[[2026-07-07 - Revision de pantallas flujo operación]] (M9) y glosario "Referencia"
del [[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]]. Relacionado con
[[2026-07-02 - Aéreo Sigac 3.0]] (referencia sin facturas obligatorias, CEUS).

## D1 — punto de partida
- **Reusa:** `references/createReference/page.tsx:43-68` (StepperModal, 4 pasos);
  cuerpos de paso `components/stepBasicDataReference/`, `stepDocumentsReference/`
  (extracción vía `POST /api/workflow/process` — **v1**, ver discrepancia),
  `stepAcceptedDocuments/`, `stepCommercialDocsReference/`. APIs `POST /references`,
  `GET /references/:id`, `GET /references/:id/invoices`.
- **Refactoriza:** 4 → 3 pasos; **sacar "Documentos Comerciales" del wizard** y
  moverlo al detalle (DGO, SP-05). Añadir campo **"orden de compra"** a la referencia
  (M9: el cliente busca por su OC). Soportar los **dos sabores de creación** (M9):
  de cara al cliente (arrastra documentos → audita → crea) y de cara al ejecutivo
  (cliente + tráfico + instrucciones → sube documentos → audita). Opciones de
  arranque: con documentos / anticipada / virtual.
- **Crea:** nada estructural nuevo (reordenación + campo OC).
- **Limpia (código muerto confirmado):** `references/reference-stepper/index.tsx`
  (wizard alterno con placeholders). Revisar `components/stepChecklistGlosa/` y
  `stepInvoicesReference/` (existen pero no cableados) — decidir integrar o eliminar.

## Decisión abierta a resolver en este sub-plan
- **Orden de los 3 pasos:** M9 dice *datos básicos → subir a CEUS → auditoría de
  extracción*; el [[Inventario_Pantallas_v3]] dice *Documentos → Documentos
  Aceptados → Datos Básicos*. Resolver aquí (recomendado: seguir M9, que además
  argumenta que la auditoría exige datos básicos primero) y anotar el porqué.

## Fuera de alcance
- La sección de documentos comerciales en sí (vive ahora en el DGO, SP-05).
- El portal de carga del cliente (SP-18).

## Pasos
- [x] Resolver y aplicar el orden de los 3 pasos. (datos básicos → documentos → auditoría, per M9)
- [x] Reducir el wizard a 3 pasos; retirar el paso "Documentos Comerciales". (componente no borrado, queda para SP-05)
- [x] Añadir campo "orden de compra". (código completo; migración Prisma bloqueada, ver manifiesto)
- [x] Soportar los dos sabores (cliente / ejecutivo). (vía `user.IsInternalUser`; validar con negocio)
- [x] Eliminar `reference-stepper/index.tsx`; resolver stepChecklistGlosa/stepInvoicesReference. (stepper eliminado; los otros dos quedan como decisión abierta, no cableados)

## Riesgos y side effects
- Confirmar la versión de extracción correcta (v1 `/api/workflow/process` vs v3)
  antes de tocar el paso de documentos. — confirmado v1, sin tocar.

## Criterios de verificación
- Gate estático verde. Playwright: crear una referencia por ambos sabores en 3
  pasos, sin exigir documentos comerciales, con OC; sin errores de consola.
- Estático: verde (back tsc/eslint/jest; front tsc/eslint sobre archivos tocados).
- Playwright: NO ejecutado (bloqueo de migración impide levantar el flujo completo).

## Estado
🟡 Parcialmente implementado — bloqueo de infraestructura en la migración Prisma
(historial de migraciones inconsistente en el entorno, ver manifiesto en
`.manifiestos/2026-07-10-refactor-flujo-ejecutivo-sp02-wizard-crear-referencia.md`).
Decisiones abiertas: distinción cliente/ejecutivo sin validar con negocio;
destino de `stepChecklistGlosa`/`stepInvoicesReference` sin resolver.
