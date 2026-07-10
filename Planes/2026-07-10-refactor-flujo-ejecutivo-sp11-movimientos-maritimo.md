# Sub-plan SP-11: Movimientos especializados — Marítimo (Importación y Exportación)

Parte de [[2026-07-10-refactor-flujo-ejecutivo]]. Agrupa los movimientos marítimos
(justificación de agrupación en el paraguas).

## Contexto

En marítimo la unidad de seguimiento son las guías marítimas y los contenedores.
Origen: glosario "Movimiento" (marítimo import + export, ambos definidos) del
[[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]], [[2026-07-01 - Marítimo Sigac 3.0]]
y [[Inventario_Pantallas_v3]] (Fase 3 marítimo + #18/#19/#20c). **El ejecutivo SÍ
actúa** en estos procesos (resuelto).

## D1 — punto de partida
- **Reusa:** componente de **Asignación de Transporte** de SP-10 (mismo, reutilizable,
  #19); `references/components/tabs/ReferenceAppointments.tsx` + `GET /references/:id/appointments`
  como base para **Generar Cita** (#20c); patrón de `movimiento-form/**`.
- **Crea (Importación):**
  - **Revalidación Marítima (#18):** documental + pago ante la naviera.
  - **Retorno de Vacío:** solicita transporte, da seguimiento a fumigación/lavado y
    **solicita fondos** para el pago (engancha con SP-14); hasta el acuse de vacío.
  - **Recuperación de Garantía:** proceso documental; solicita el depósito/devolución.
- **Crea (Exportación):**
  - **Toma de Vacío:** solicita transporte para el contenedor vacío.
  - **Carga de Mercancía:** seguimiento del proceso de carga; genera citas si aplica.
  - **Ingreso de Mercancía:** genera la **cita** (ej. PIS en Manzanillo) para el
    ingreso a puerto/terminal.
  - **Generar Cita (#20c):** tipo de cita, puerto/recinto, fecha/hora, referencia/
    movimiento asociado.

## Fuera de alcance
- **Modulación, Liberación y Zarpe:** solo visibilidad (vive en Despacho, SP-16).
- El seguimiento del transporte (trámite y despacho, solo lectura).

## Pasos
- [ ] Reusar Asignación de Transporte (de SP-10).
- [ ] Generar Cita (#20c) sobre la base de Appointments.
- [ ] Import: Revalidación Marítima, Retorno de Vacío (con solicitud de fondos),
      Recuperación de Garantía.
- [ ] Export: Toma de Vacío, Carga de Mercancía, Ingreso de Mercancía.

## Riesgos y side effects
- Depende de SP-07 (enrutado por tráfico), SP-10 (componente de transporte) y SP-14
  (solicitud de fondos, para Retorno de Vacío).

## Criterios de verificación
- Gate estático verde. Playwright: en una referencia marítima, recorrer Retorno de
  Vacío (transporte → fumigación/lavado → solicitud de fondos → acuse) y generar una
  cita PIS; sin errores de consola.

## Estado
📋 Por implementar.
