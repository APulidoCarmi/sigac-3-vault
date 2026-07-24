# Plan: Movimientos dinámicos por tráfico — Implementar aéreo/marítimo

## Contexto

**Origen:** Tarea #2 de [[Tareas.md]] ("Implementar movimientos dinámicos por tipo de tráfico"), de reunión [[2026-07-14 - Prueba operación real desde ticket a facturación]].

**Problema:** El tab Movimientos en detalle de referencia muestra dinámicamente para TERRESTRE (Entrada, Subdivisión), pero para AÉREO y MARÍTIMO solo expone un input manual de `shipmentId` sin lista real.

**Estado actual:**
- ✅ TERRESTRE: `ReferenceShipmentsTerrestre.tsx` renderiza tabs por tipo de movimiento (INBOUND, SUBDIVISION)
- ❌ AÉREO: `ReferenceShipmentsAereo.tsx` solo tiene shell vacío (línea 37-87)
- ❌ MARÍTIMO: `ReferenceShipmentsMaritimo.tsx` solo tiene shell vacío (línea 41-113)
- ❌ `FlowTypeSelector.tsx` no filtra qué movimientos son permitidos por tráfico
- ❌ No hay enumeración centralizada que mapee tráfico → tipos de movimiento permitidos

**Módulos afectados:**
- Frontend: `carmi-digital/app/(customerPortal)/references/components/tabs/`
  - `ReferenceShipments.tsx` (router por tráfico)
  - `shipments/ReferenceShipmentsTerrestre.tsx` (modelo a replicar)
  - `shipments/aereo/ReferenceShipmentsAereo.tsx` (por implementar)
  - `shipments/maritimo/ReferenceShipmentsMaritimo.tsx` (por implementar)
- Frontend: `carmi-digital/components/shipment-creation/FlowTypeSelector.tsx` (por actualizar filtrado)

**Especificaciones de movimientos:**
- SP-10 (Aéreo, importación): **Manifiesto de Carga, Revalidación Aérea, Asignación de Transporte**
- SP-11 (Marítimo, import/export): 
  - **Import:** Revalidación Marítima, Retorno de Vacío, Recuperación de Garantía
  - **Export:** Toma de Vacío, Carga de Mercancía, Ingreso de Mercancía, Generar Cita

## Decisiones tomadas

1. **Estructura UI idéntica a terrestre:** Usar tabs por tipo de movimiento, no input manual de `shipmentId`.
2. **Mapeo centralizado:** Crear enum/constante `ALLOWED_SHIPMENT_FLOWS_BY_TRAFFIC` que defina qué `ShipmentFlowType` es permitido para cada tráfico.
3. **FlowTypeSelector filtrado:** Actualizar para que solo muestre opciones permitidas según tráfico de la referencia.
4. **Flujos de creación:** Reusar `ShipmentCreationModal.tsx` y `SubdivisionCreationModal.tsx` si aplican; crear nuevos modales si los movimientos aéreo/marítimo requieren formas específicas (fuera de esta tarea si ya existen en SP-10/SP-11).
5. **Backend:** Confiar en que `GET /references/:id/shipments` filtra por tráfico automáticamente y que endpoints de creación validan.

## Fuera de alcance

- Implementación de las **formas específicas** de creación para movimientos aéreo/marítimo (Manifiesto, Revalidación, etc.). Eso ya existe en SP-10/SP-11 como componentes independientes.
- Validación backend de qué `ShipmentFlowType` es permitido por tráfico (asumir que ya existe o se implementa en backend).
- Movimientos de **exportación terrestre** (bloqueado per plan anterior).

## Pasos

- [ ] **1. Crear enum centralizado `ALLOWED_SHIPMENT_FLOWS_BY_TRAFFIC`**
  - Archivo: `carmi-digital/types/shipment-flows.constants.ts` (nueva)
  - Mapeo: `TERRESTRE` → `['INBOUND', 'SUBDIVISION', 'OUTBOUND', 'OSD']`; `AEREO` → `['MANIFIESTO', 'REVALIDACION_AEREA', 'ASIGNACION_TRANSPORTE']`; `MARITIMO_IMPORT` → `['REVALIDACION_MARITIMA', 'RETORNO_VACIO', 'RECUPERACION_GARANTIA']`; `MARITIMO_EXPORT` → `['TOMA_VACIO', 'CARGA_MERCANCIA', 'INGRESO_MERCANCIA', 'GENERAR_CITA']`
  - Nota: Si los flowType en BD para aéreo/marítimo usan nombre diferente (p. ej. códigos internos), sincronizar con backend primero

- [ ] **2. Actualizar `shipment.types.ts`**
  - Extender `ShipmentFlowType` con nuevos tipos si no existen: `'MANIFIESTO' | 'REVALIDACION_AEREA' | ... | 'GENERAR_CITA'`
  - O bien, validar que ya están definidos en backend y traerlos del API

- [ ] **3. Refactorizar `ReferenceShipmentsTerrestre.tsx` → componente genérico**
  - Crear `ReferenceShipmentsGeneric.tsx` (o mantener en terrestre si es más simple)
  - Acepta: `reference`, `allowedFlowTypes` (del enum centralizado), `onUpdate`
  - Renderiza tabs dinámicamente para cada `flowType` permitido
  - Filtra: `shipments.filter(s => allowedFlowTypes.includes(s.flowType))`
  - Reutilizar en terrestre, aéreo y marítimo

- [ ] **4. Implementar `ReferenceShipmentsAereo.tsx` real**
  - Usar componente genérico o replicar estructura de terrestre
  - Pasar `allowedFlowTypes = ALLOWED_SHIPMENT_FLOWS_BY_TRAFFIC['AEREO']`
  - Renderizar tabs: "Manifiesto de Carga", "Revalidación Aérea", "Asignación de Transporte"
  - Quitar input manual de `shipmentId`

- [ ] **5. Implementar `ReferenceShipmentsMaritimo.tsx` real**
  - Similar a aéreo, pero con dos casos:
    - Si `reference.flowDirection === 'IMPORT'` → mostrar tabs de import
    - Si `reference.flowDirection === 'EXPORT'` → mostrar tabs de export
  - Pasar `allowedFlowTypes` dinámicamente según dirección

- [ ] **6. Actualizar `FlowTypeSelector.tsx`**
  - Recibir `trafficType` como prop
  - Filtrar opciones: `Object.keys(ALLOWED_SHIPMENT_FLOWS_BY_TRAFFIC[trafficType])`
  - Solo mostrar botones/opciones permitidas

- [ ] **7. Gate estático y Playwright**
  - `tsc --noEmit` sin errores en archivos tocados
  - `eslint` sin errores
  - Playwright: 
    1. Crear referencia TERRESTRE → ver tabs Entrada, Subdivisión; crear movimiento INBOUND
    2. Crear referencia AÉREO → ver tabs Manifiesto, Revalidación, Asignación; verificar que no aparece input manual de `shipmentId`
    3. Crear referencia MARÍTIMO IMPORT → ver tabs de import; crear un movimiento
    4. Crear referencia MARÍTIMO EXPORT → ver tabs de export
    5. Sin errores de consola en ningún caso

## Riesgos y side effects a vigilar

1. **Cambio de nombre de `ShipmentFlowType`:** Si aéreo/marítimo usan nombres diferentes en BD vs. frontend, habrá mismatch. Sincronizar con backend ANTES de implementar.
2. **Componentes de creación:** SP-10/SP-11 probablemente ya tienen modales específicos (ej. `ManifiestoModal.tsx`). Asegurarse de que se enrutan correctamente desde los tabs.
3. **Endpoints backend:** Si `/references/:id/shipments` NO filtra automáticamente por tráfico, la lista mostrará movimientos de tráficos ajenos. Validar antes.
4. **Dirección de referencia (import/export):** MARÍTIMO necesita saber si es import o export para mostrar movimientos diferentes. Asegurarse de que `reference.flowDirection` existe y tiene valores consistentes.
5. **Side effect en FlowTypeSelector:** Cambiar de filtrado podría afectar a otras pantallas que lo usen. Buscar dónde más se importa y validar que no rompe.

## Criterios de verificación

| Paso | Criterio |
|------|----------|
| 1-2 | Archivo creado, enum compilable, tipos TypeScript consistentes |
| 3-5 | Componentes renderizen sin errores, tabs dinámicos aparecen según tráfico |
| 6 | FlowTypeSelector filtra opciones; solo muestra las permitidas para el tráfico de la referencia |
| 7 (Gate) | `tsc` y `eslint` 0 errores en archivos nuevos/tocados |
| 7 (Playwright) | Flujo completo en 3 tráficos sin errores de consola; tabs correctos por tipo |

## Notas de implementación

- **No crear nuevos endpoints backend:** Confiar en que ya existen o se agregan en paralelo (SP-10/SP-11).
- **Reusar patrones existentes:** Los modales de creación para cada movimiento aéreo/marítimo ya están en SP-10/SP-11; solo enrutar desde los tabs.
- **Validación de dirección marítima:** Investigar primero cómo viene `reference.flowDirection` del API; podría ser un campo booleano, enum o absent.
