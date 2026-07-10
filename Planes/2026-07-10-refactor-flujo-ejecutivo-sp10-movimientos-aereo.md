# Sub-plan SP-10: Movimientos especializados — Aéreo (Importación)

Parte de [[2026-07-10-refactor-flujo-ejecutivo]]. Agrupa las pantallas de movimientos
aéreos de importación (justificación de agrupación en el paraguas).

## Contexto

Cubre del [[Inventario_Pantallas_v3]]: #20 (Tablero de Manifiestos de Carga), #18
(Revalidación Aérea) y #19 (Asignación de Transporte) — todas 🔵 nuevas. En aéreo la
unidad de seguimiento son los manifiestos y las guías master/house. Origen: glosario
"Movimiento" (aéreo importación) del [[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]],
[[2026-07-02 - Aéreo Sigac 3.0]] y [[2026-07-08 - Aéreo SIGAC 3.0]] (guías por
longitud 12/11/10, master/house, peso llega después de la báscula del almacén).

## D1 — punto de partida
- **Reusa:** el patrón de `components/movimientos/movimiento-form/**` y el
  StepperModal como armazón; el catálogo de tráfico de la referencia.
- **Crea:**
  - **Manifiesto de Carga + Tablero de planeación diaria (#20):** recepción por correo
    de manifiestos (almacén, FedEx, DHL); etapas llega → ETA → confirma llegada →
    descarga → desconsolida → asigna a mesa de previo → listo para previo.
  - **Revalidación Aérea (#18):** proceso documental + pago ante la aerolínea
    (comprobante de pago). El ejecutivo actúa.
  - **Asignación de Transporte (#19):** el ejecutivo asigna transportista (datos de
    contacto/ruta); el **semáforo verde/rojo** (llega, carga correcta, sale) lo
    actualiza **trámite y despacho** → **solo lectura** para el ejecutivo. Diseñar
    como **componente reutilizable** (mismo que marítimo, SP-11).

## Fuera de alcance
- **Export Aéreo** (bloqueado — falta sesión de discovery).
- El seguimiento del transporte (rol trámite y despacho, solo lectura).
- El incrementable por guía + auto-suma vive en el DGO (SP-05); aquí solo el tracking.

## Pasos
- [ ] Componente reutilizable de Asignación de Transporte (semáforo solo lectura).
- [ ] Revalidación Aérea (documental + pago).
- [ ] Manifiesto de Carga + Tablero de planeación diaria con las etapas.

## Riesgos y side effects
- Depende de SP-07 (el tab Movimientos debe enrutar a estos según tráfico aéreo).
- El componente de Asignación de Transporte se comparte con SP-11: definirlo aquí bien.

## Criterios de verificación
- Gate estático verde. Playwright: en una referencia aérea de importación, ver el
  tablero de manifiestos avanzar de etapa, registrar una revalidación con pago y
  asignar transporte (semáforo solo lectura); sin errores de consola.

## Estado
📋 Por implementar.
