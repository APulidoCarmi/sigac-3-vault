# Sub-plan SP-00 (spike): Esquema de operación virtual / B1

Parte de [[2026-07-10-refactor-flujo-ejecutivo]]. Spike según [[Spikes e investigacion]]:
el objetivo es **aprender**, no entregar código de producción.

## Contexto

Brecha #5 del [[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]]: D1 muestra "Tipo
de Tráfico" con MAR/AER/TER pero **no B1**, y no está claro cómo (o si) el modelo
de datos actual identifica una operación **virtual**. Origen del requisito:
[[2026-07-01 - Aéreo Sigac 3.0 - operaciones virtuales (B1)]] (M6 — identificadores
MS/IM/B1/IC, folios mensuales, cierre 10 días hábiles / 20 si OEA) y punto abierto
f#9 (alertas automáticas de vencimiento de folios B1).

Hallazgo del inventario D1 (odin): el tipo de tráfico es `trafficTypeId →
TransportMode`, **no un enum con B1** (`carmi-odin-api-v2/src/references/dtos/create-reference.dto.ts:105-112`;
el path XML usa enum inline `['MARITIME','AIR','LAND']` en
`create-reference-from-xml.dto.ts:181-182`). No hay concepto B1/virtual en la
capa DTO. Falta el nivel de BD.

## Pregunta a responder (acotada)

¿Cómo se representa —o cómo habría que representar— una operación **virtual (B1)**
en el modelo de datos actual, y dónde deben vivir sus identificadores y las
alertas de folios?

## D1 — punto de partida
- **Reusa (para investigar):** `carmi-db-api` (prisma/entidades) para el catálogo
  `TransportMode` y el esquema de Reference/Operation; DTOs de odin ya inspeccionados.
- **Refactoriza/crea:** nada en este spike (código exploratorio desechable).

## Pasos
- [ ] Localizar en `carmi-db-api` el modelo/tabla de `TransportMode` y el esquema
      de Reference/Operation (graphify primero; luego prisma/entidades).
- [ ] Determinar si hay algún flag/enum/campo que marque "virtual" hoy, o si es
      ausencia total (⇒ requeriría migración de esquema, no solo un valor de enum).
- [ ] Mapear dónde configurar los identificadores MS/IM/B1/IC (Documento: MS y IM
      a nivel relación cliente-proveedor/comprador; IC a nivel cliente si OEA).
- [ ] Ubicar dónde viven hoy los folios y qué haría falta para alertas de vencimiento
      (mensual, 10 días hábiles / 20 si OEA).
- [ ] Escribir la nota de conclusión en `Arquitectura/` (conclusión duradera) con
      evidencia (rutas de esquema, qué se probó, qué se concluyó).

## Fuera de alcance
- Implementar el soporte B1 (pantallas, identificadores, alertas): fase posterior,
  con esta nota como insumo del Contexto de su `/plan`.

## Riesgos y side effects
- Si el modelo no contempla lo virtual, es **migración de esquema** (afecta
  `carmi-db-api` + odin + front), no un simple valor de enum: dimensionarlo aquí.

## Criterios de verificación
- Nota en `Arquitectura/` que responde la pregunta con evidencia, suficiente para
  abrir el `/plan` de las pantallas B1 sin re-investigar. No hay gate estático
  (no produce código de producción).

## Estado
📋 Por ejecutar. **Bloquea** cualquier sub-plan de pantallas B1/virtual.
