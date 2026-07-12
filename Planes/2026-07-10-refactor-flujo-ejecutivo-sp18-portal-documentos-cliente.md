# Sub-plan SP-18: Portal de Documentos del Cliente

Parte de [[2026-07-10-refactor-flujo-ejecutivo]]. Feature externo/público.

## Contexto

Pantalla #26 del [[Inventario_Pantallas_v3]] (🔵 nueva, ✅ confirmada). Que el cliente
suba documentos directamente, sin pasar por el ejecutivo, y reciba retroalimentación.
Origen: [[2026-07-06 - Seguimiento de refactor]] (M8: portal por URL pública, totales
requeridos/validados/pendientes/rechazados) y
[[2026-07-07 - Revision de pantallas flujo operación]] (M9: documentos de embarque vs
internos, búsqueda por orden de compra, descarga "expediente completo" ZIP).
**Decisión de la entrevista:** dentro del refactor, **después** de Expediente + glosado.

## D1 — punto de partida
- **No existe.** Reusa como referencia: el patrón de **public-token** de OSD
  (`carmi-odin-api-v2/src/osd/controllers/osd-reports.controller.ts` `/:id/public-token`)
  para la URL pública, y los **Signed URL de GCP** (preview/descarga, expiran ~10 min).
- **Crea:**
  - Portal por **URL pública** donde el cliente sube documentos que **caen directo al
    Expediente Aduanero** (#5, SP-04), marcados "subido por el cliente".
  - **Feedback del glosado (Zeus/CEUS) en ambos lados:** en el Expediente (ejecutivo) y
    en el portal (cliente: válido / con error / pendiente).
  - Totales requeridos / validados / pendientes / rechazados; **búsqueda por orden de
    compra**; descarga de **"expediente completo" (ZIP)** (XML del pedimento,
    comprobantes, factura Carmi, hoja SAT, COVEs, BL, etc.).

## Fuera de alcance
- La notificación al cliente por URL para comprobante de **fondos** (fase posterior de SP-14).
- El motor de glosado en sí (SP-04); aquí se consume su resultado.

## Pasos
- [x] URL pública + carga de documentos → Expediente (marcado "subido por el cliente").
- [x] Reflejar el resultado del glosado al cliente (válido/error/pendiente) y totales.
- [x] Búsqueda por orden de compra.
- [x] Descarga de "expediente completo" (ZIP).

## Riesgos y side effects
- **Externo/público (outward-facing):** confirmar seguridad de la URL pública (tokens,
  expiración) antes de exponer. **Depende de SP-04** (Expediente + glosado).

## Criterios de verificación
- Gate estático verde. Playwright: subir un documento desde el portal por URL pública,
  verlo aparecer en el Expediente marcado "subido por el cliente" con su feedback de
  glosado, y descargar el ZIP; sin errores de consola.

### Estado de verificación (2026-07-12)
- Gate estático: ✅ verde en ambos repos (tsc, eslint, jest — ver manifiesto).
- Playwright E2E: ⏸️ pendiente — no se ejecutó en esta sesión por falta de datos
  sembrados (compañía cliente + referencia con OC de prueba) para recorrer el flujo
  real; el flujo se validó por inspección de código y tests unitarios/build. Ver
  manifiesto para detalle.

## Estado
✅ Implementado (backend + frontend), pendiente validación Playwright E2E con datos
de prueba reales. Ver manifiesto para detalle de diseño y desviaciones.
