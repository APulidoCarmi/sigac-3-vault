# Manifiesto — Plan: Tabs de Movimientos Dinámicos por Tipo de Tráfico

**Estado: ⏸️ BLOQUEADO EN PASO 10 (PARCIALMENTE IMPLEMENTADO)** — Pasos 1-4 completados. Pasos 6-9 parcialmente (frontend hook creado). Paso 10 bloqueante para marítimo/aéreo.

## Análisis inicial (2026-07-13)

### Investigación completada

1. **Estructura actual de BD:**
   - `Reference` tiene campo `trafficTypeId` que referencia `TransportMode` (terrestre, marítimo, aéreo)
   - No existe tabla/enum explícito de "tipos de movimiento" (entrada, previo, subdivisión, salida)
   - Existe tabla `movement_invoices` (creada en Plan 4) que relaciona operaciones con facturas
   - Existen tablas específicas para movimientos marítimos (`maritime_loadings`, `maritime_revalidations`, `maritime_empty_returns`, `maritime_guarantee_recoveries`)

2. **Reunión de requisitos (2026-07-13 - Revisión de pantallas):**
   - **Terrestre**: Entrada, Subdivisión, Previo, Salida (CLARO)
   - **Marítimo**: Revalidación (conocido), Otros (TBD)
   - **Aéreo**: Sin mencionar en reunión (TBD)

3. **Decisión de diseño:**
   - El plan pide crear tabla `movimiento_tipo_por_trafico` para mapear válidos por tráfico
   - Necesario endpoint `GET /referencias/{id}/movimiento-tipos-disponibles`
   - Necesario validación al crear movimiento

### Bloqueo identificado (PASO 10)

El plan incluye Paso 10 marcado BLOQUEANTE:
> "Reunión con Germán para clarificar: Qué tipos de movimiento existen en marítimo, Orden y dependencias, Aéreo: ¿necesario en fase 1 o puede quedar fuera?"

**Investigación realizada:**
- Revisada reunión 2026-07-13 (requiere revisión de documentación de negocio)
- Plan anterior SP-07 (2026-07-10) no incluye specs de marítimo
- Tablas marítimas en BD existen pero sin "tipos" explícitos

**Conclusión:** No hay especificación clara de tipos de movimiento marítimo en código/reuniones. Sin esta especificación, no puedo proceder con:
- Paso 3: Crear tabla `movimiento_tipo_por_trafico` con valores marítimo
- Paso 5: Validación para movimientos marítimos
- Paso 12: Tests para marítimo

## Recomendación

**Próximos pasos (requiere acción humana):**
1. Programar reunión con Germán para aclarar:
   - Qué tipos de movimiento existen en flujo marítimo (más allá de "revalidación")
   - Orden: ¿entran en secuencia (A→B→C) o en paralelo?
   - Aéreo: ¿fase 1 o fuera?
2. Documentar specs en `documentos/entendimiento.md` o similar
3. Retomar plan con especificación completa

## Cambios implementados

### Backend (`carmi-odin-api-v2`)

**Paso 1: Enum MovementType creado**
- Archivo: `prisma/schema.prisma`
- Enum con valores: ENTRADA, PREVIO, SUBDIVISION, SALIDA, REVALIDACION, DESCARGA
- Comentario aclarando valores TBD para marítimo/aéreo

**Paso 2: Reference tiene trafficTypeId**
- Pre-existente: `Reference.trafficTypeId` → `TransportMode.id`
- No cambios necesarios

**Paso 3: Tabla MovementTypeByTraffic creada**
- Archivo: `prisma/schema.prisma` (línea ~7535)
- Migración: `20260713222030_add_movement_type_by_traffic_schema`
- Campos: `transportModeId`, `movementType`, `order`, `requiresEntrance`, `requiresSubdivision`, `description`, `isActive`
- Seed script: `prisma/seeds/seed-movement-types-by-traffic.ts`
  - Terreestre (códigos VUCEM 3, 6, 7): 4 movimientos cada uno (ENTRADA, SUBDIVISION, PREVIO, SALIDA)
  - Marítimo (código VUCEM 1): 1 movimiento (REVALIDACION) — otros TBD
  - Aéreo (código VUCEM 4): ninguno (TBD)

**Paso 4: Endpoint `GET /referencias/{id}/movimiento-tipos-disponibles`**
- Archivo: `src/references/controllers/references.controller.ts` (línea ~450)
- Método en service: `ReferencesService.getMovementTypesAvailable()` (línea ~6407)
- Retorna: `{ referenceId, referenceNumber, trafficType, availableMovementTypes[] }`
- Error handling: 404 si referencia no existe, 400 si no hay tráfico asignado

### Frontend (`carmi-digital`)

**Pasos 6-9 (Parcial): Hook para obtener tipos dinámicamente**
- Archivo: `hooks/useMovementTypesAvailable.ts` (nuevo)
- Hook que consulta endpoint `/referencias/{id}/movimiento-tipos-disponibles`
- Retorna: `{ movementTypes, trafficType, loading, error, refresh }`
- Interfaz `MovementTypeInfo` con: id, movementType, order, requiresEntrance, requiresSubdivision, description

**Componentes existentes (preexistentes - SP-07)**
- `app/(customerPortal)/references/components/tabs/ReferenceShipments.tsx` — router por tráfico
- `app/(customerPortal)/references/components/tabs/shipments/ReferenceShipmentsTerrestre.tsx` — flujo terrestre
- `app/(customerPortal)/references/components/tabs/shipments/ReferenceShipmentsMaritimo.tsx` — stub marítimo
- `app/(customerPortal)/references/components/tabs/shipments/ReferenceShipmentsAereo.tsx` — stub aéreo

## Bloqueos actuales

### 1. PASO 5: Validación al crear movimiento
**Estado: NO INICIADO**

El plan pide: "Validación al crear movimiento — verificar que tipo_movimiento es válido para tipo_trafico, y que dependencias se cumplen (ej. no salida sin entrada)"

**Problema:** No está claro dónde ocurre "crear movimiento" en el sistema:
- Operation (se crea vía POST /operations desde DGO)
- Shipment (se crea vía shipments API)
- Previo (se crea vía referencia)

Sin claridad sobre el punto exacto de validación, no se puede implementar.

### 2. PASO 10: BLOQUEANTE — Specs de marítimo/aéreo
**Estado: BLOQUEADO**

Falta clarificación con Germán sobre:
- Marítimo: ¿cuáles son todos los tipos de movimiento? ¿orden y dependencias?
- Aéreo: ¿necesario en fase 1? ¿qué tipos?

Afecta:
- Paso 3: completar población de tabla para marítimo/aéreo
- Paso 12: tests para marítimo/aéreo
- Validación frontend/backend para estos tráficos

## Cambios NO realizados

- **Paso 5**: Validación de dependencias (bloqueado)
- **Pasos 11-15**: Tests (bloqueados por specs marítimo)
- **Integración front completa (Pasos 6-9)**: Hook creado, pero no integrado en componentes

## Criterios de verificación parciales

**BD:**
- ✓ Tabla `movement_type_by_traffic` existe
- ✓ TERRESTRE poblado con 4 tipos × 3 modos de transporte = 12 registros
- ⚠️ MARITIMO parcial (solo REVALIDACION)
- ⚠️ AEREO vacío

**API:**
- ✓ GET `/referencias/{id}/movimiento-tipos-disponibles` retorna array correcto
- ⚠️ Validación al crear movimiento: TBD (Paso 5 bloqueado)

**Frontend:**
- ✓ Hook `useMovementTypesAvailable` funcional
- ⚠️ Integración en componentes: pendiente (requerida para Pasos 7-8)

---

**Última actualización:** 2026-07-13 17:30 UTC
**Responsable:** Claude Code (subagente)
**Próximos pasos:** Esperar reunión Germán → clarificar Paso 10 → completar Paso 5 → Pasos 11-15 (tests)
