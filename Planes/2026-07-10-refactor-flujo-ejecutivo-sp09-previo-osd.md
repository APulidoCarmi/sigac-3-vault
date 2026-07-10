# Sub-plan SP-09: Previo (consulta) + Solicitar Previo + OS&D (consulta)

Parte de [[2026-07-10-refactor-flujo-ejecutivo]].

## Contexto

Pantallas #12b (Previo, 🔵 nueva), #20b (Solicitar Previo, 🔵 nueva) y #16 (Ver OS&D,
🟡 cambia de rol) del [[Inventario_Pantallas_v3]]. El **Previo** (verificación física
de la mercancía) lo ejecuta almacén / trámite y despacho, pero el ejecutivo **consulta
su resultado** y **puede solicitarlo**. Origen: glosario "Previo" del
[[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]], [[2026-07-08 - Aéreo SIGAC 3.0]]
(reconocimiento previo) y la aclaración de que OS&D lo reporta almacén, no el ejecutivo.

## D1 — punto de partida
- **Reusa / refactoriza (OS&D #16):** `components/shipment-creation/OSDReportForm.tsx`
  (hoy **formulario de captura** del ejecutivo; `features/osd/**`,
  `carmi-odin-api-v2/src/osd/controllers/osd-reports.controller.ts`, con ruta
  `/:id/public-token` útil para el portal). **Cambia de rol** → **vista de consulta**
  (solo lectura): tipo (Sobrante/Faltante/Daño), cantidad, responsable, evidencias, a
  qué movimiento afecta. Unificar nomenclatura OSND/OS&D/OSD.
- **Crea:**
  - **Tab Previo (#12b):** estatus (pendiente / solicitado / en mesa de previo /
    completado) y el **resultado de la validación de consistencia** que hace Zeus/CEUS
    contra documentos y DGO.
  - **Solicitar Previo (#20b):** acción que se dirige a **almacén** (terrestre) o a
    **trámite y despacho** (marítimo/aéreo); referencia/movimiento afectado, notas.

## Fuera de alcance
- La ejecución física del Previo (rol almacén / trámite y despacho).
- El reporte de OS&D (lo genera almacén durante el Previo).

## Pasos
- [ ] Convertir `OSDReportForm` en vista de consulta (solo lectura) y unificar nomenclatura.
- [ ] Crear el tab Previo (estatus + resultado de consistencia Zeus vs docs/DGO).
- [ ] Crear el modal Solicitar Previo (destino según tráfico).

## Riesgos y side effects
- La consistencia contra el Previo también alimenta el Expediente (SP-04) y el DGO
  (SP-05): reusar la misma fuente de resultado.

## Criterios de verificación
- Gate estático verde. Playwright: consultar un OS&D en solo lectura, ver el estatus
  del Previo y solicitarlo al destino correcto según tráfico; sin errores de consola.

## Estado
📋 Por implementar.
