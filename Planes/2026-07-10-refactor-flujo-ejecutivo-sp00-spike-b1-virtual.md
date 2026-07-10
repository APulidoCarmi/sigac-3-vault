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

## Decisiones tomadas (entrevista previa a `/implementa`, con evidencia de esquema)

Hallazgos verificados en `carmi-odin-api-v2/prisma/schema.prisma` (NO en
`carmi-db-api`, que es un esquema legado distinto sin ningún hit de
`Reference`/`Operation`/`TransportMode`):

1. **`TransportMode` es tabla catálogo** (`model TransportMode`, L6814; filas
   MARITIME/AIR/LAND vía `Reference.trafficTypeId`), no un enum. Confirma la
   Brecha #5: no existe ningún flag/valor de "virtual" en `Reference`,
   `Operation` ni `TransportMode`.
2. **`isVirtual` es independiente del Tipo de Tráfico** (decisión tomada): una
   referencia/operación puede ser virtual con cualquier `trafficType`
   (MAR/AER/TER), no solo Aéreo. Aunque el origen del requisito (M6) es un meet
   de Aéreo, no se restringe la Brecha en el esquema. Implementación futura:
   nuevo campo boolean `isVirtual` en `Reference` (y/o `Operation`), ortogonal a
   `trafficTypeId`.
3. **Estructura de identificadores MS/IM/B1/IC:** `ReferenceIdentifier`/
   `GlobalIdentifier` (L3334, L3387) ya tienen la forma `clave` + `c1/c2/c3`
   (clave+complementos) — reutilizar esa tabla para las 4 claves en vez de
   crear una nueva.
   - **IM (IMMEX del propio cliente):** reutilizar `MompConfiguration.immex`
     (L8814, ya existe a nivel cliente) como fuente; no se agrega campo nuevo.
   - **IC (identificador OEA):** reutilizar `MompConfiguration.ic` (L8816, ya
     existe). Nota: es un string libre, no un boolean de certificación — la
     fase posterior deberá decidir si su sola presencia (no-nulo) basta como
     señal de "cliente certificado OEA" o si hace falta un flag explícito
     (relevante también para la regla de cierre 10 vs 20 días hábiles).
   - **B1 (IMMEX de la contraparte):** **NO** se agrega campo a `MompSupplier`.
     Se resuelve leyendo el `MompConfiguration.immex` de la Company de la
     contraparte, cuando esa contraparte es también cliente del sistema (tiene
     su propio `MompConfiguration` vía `MompSupplier.companyId` → `Company`).
     **Punto abierto sin resolver:** `MompSupplier.companyId` es opcional
     (`String?`) — si la contraparte NO es cliente del sistema (proveedor
     puramente externo, sin `Company`/`MompConfiguration` propios), no hay
     fuente de la que leer su IMMEX. La fase de implementación de pantallas B1
     debe decidir qué pasa en ese caso (¿campo manual de respaldo? ¿bloquear
     virtual si no hay match?).
   - **MS (Modalidad de Servicios):** complemento siempre "1" — valor
     constante, no requiere fuente de datos.
4. **Folios B1 mensuales + alertas de vencimiento:** confirmado que no existe
   ningún modelo hoy (se revisaron los 4 campos `*Folio` del esquema —
   `coveFolio`, `eDocumentFolio`, `manifestationFolio`, `referenceFolio` — y
   ninguno aplica). Decisión: nueva tabla dedicada (p. ej.
   `VirtualOperationFolio`: mes, fecha límite, si aplica extensión OEA,
   estado), ligada a `Reference`; las alertas de vencimiento se apoyan en el
   módulo `notifications` ya existente en `carmi-odin-api-v2/src/notifications`
   (mismo patrón que `momp-notification.service.ts`).

## D1 — punto de partida
- **Reusa (para investigar):** `carmi-db-api` (prisma/entidades) para el catálogo
  `TransportMode` y el esquema de Reference/Operation; DTOs de odin ya inspeccionados.
- **Refactoriza/crea:** nada en este spike (código exploratorio desechable).

## Pasos
- [x] Localizar en `carmi-db-api` el modelo/tabla de `TransportMode` y el esquema
      de Reference/Operation (graphify primero; luego prisma/entidades). →
      confirmado legado, 0 hits; el esquema real vive en
      `carmi-odin-api-v2/prisma/schema.prisma`.
- [x] Determinar si hay algún flag/enum/campo que marque "virtual" hoy, o si es
      ausencia total (⇒ requeriría migración de esquema, no solo un valor de enum).
      → ausencia total, confirmada línea por línea.
- [x] Mapear dónde configurar los identificadores MS/IM/B1/IC (Documento: MS y IM
      a nivel relación cliente-proveedor/comprador; IC a nivel cliente si OEA).
- [x] Ubicar dónde viven hoy los folios y qué haría falta para alertas de vencimiento
      (mensual, 10 días hábiles / 20 si OEA).
- [x] Escribir la nota de conclusión en `Arquitectura/` (conclusión duradera) con
      evidencia (rutas de esquema, qué se probó, qué se concluyó). →
      [[SP-00 - Spike esquema B1-virtual - conclusiones]].

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
✅ Cerrado (2026-07-10). Ver [[SP-00 - Spike esquema B1-virtual - conclusiones]]
en Arquitectura/ — evidencia verificada contra
`carmi-odin-api-v2/prisma/schema.prisma`. No se generó código de producción ni
cambios en ningún repositorio (spike exploratorio, según su propio alcance).
Queda **un punto abierto sin resolver** (`MompSupplier.companyId` opcional)
que el `/plan` de pantallas B1 deberá decidir antes de diseñar el flujo de
identificador B1.
