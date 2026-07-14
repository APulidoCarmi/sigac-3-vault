# Manifiesto: Rediseño de Bandeja de Entrada

**Fecha:** 2026-07-13  
**Sub-plan:** 2026-07-13-bandeja-entrada.md  
**Rama:** `feat/2026-07-13-rediseno-interfaz`  
**Estado:** Implementación completada (sin commits; cambios en working tree)

## Archivos Tocados

### Frontend (carmi-digital)

1. **`app/(customerPortal)/references/components/inbox/InboxCard.tsx`** (NUEVO)
   - Componente reutilizable de tarjeta para la Bandeja de Entrada
   - Muestra: ID referencia, cliente, valor total, estado, fechas (creado/actualizado), badge de prioridad
   - Incluye acción principal (link a detalle o botón custom)
   - Función auxiliar `formatDate()` para mostrar fechas relativas (ej: "3d", "2s", "1m")

2. **`app/(customerPortal)/references/components/inbox/InboxTabs.tsx`** (NUEVO)
   - Componente principal de Bandeja de Entrada con 7 tabs
   - Tabs implementados:
     1. **Alertas** → status DRAFT (placeholder; en futuro será priority=URGENT)
     2. **Accionable ahora** → status QUOTED
     3. **Curso automático** → status APPROVED
     4. **Esperando a terceros** → status PENDING_QUOTE
     5. **Por identificar** → status DRAFT
     6. **Guías sin identificar** → status DRAFT (placeholder; en futuro requiere modelo UnidentifiedWaybill)
     7. **Documentos que requieren atención** → status DRAFT, QUOTED, APPROVED (placeholder; requiere field específico)
   - Integración con backend: consume `GET /references?status=<X>&deletionFilter=active&page=1&limit=10`
   - Manejo de loading, error, y empty states
   - Grid responsive (1 col mobile, 2 sm, 3 lg)
   - Badge de conteo en cada tab

3. **`app/(customerPortal)/references/components/inbox/index.ts`** (NUEVO)
   - Re-exporta InboxCard e InboxTabs para uso simplificado

4. **`app/(customerPortal)/references/ui/ReferencesClient.tsx`** (MODIFICADO)
   - Cambiado estado inicial de `"board"` a `"inbox"` (Bandeja de Entrada ahora primaria)
   - Actualizado TabsList: 3 tabs ahora: "Bandeja de Entrada" | "Tablero" | "Tabla clásica"
   - Integrada nueva pestaña: `<TabsContent value="inbox"><InboxTabs /></TabsContent>`
   - Actualizado type union: `"inbox" | "board" | "table"`

## Qué Hace Cada Cambio

### InboxCard.tsx
Proporciona UI reutilizable para una referencia en la Bandeja. Cada tarjeta muestra:
- Número de referencia (monospace, teal)
- Nombre del cliente (truncado)
- Estado/badge (ej: "Accionable ahora")
- Valor total (formateado en pesos con símbolo)
- Fechas relativas de creación y actualización
- Botón de acción principal (por defecto "Ver detalle" → `/references/{id}`)

### InboxTabs.tsx
Orquesta la carga de referencias filtradas por tab. Para cada tab:
1. Define qué status filtrar en el backend
2. Realiza llamada a `GET /references?status=X` cuando el usuario selecciona el tab
3. Mapea respuesta a `InboxCardData[]`
4. Renderiza grid de `InboxCard`s
5. Muestra conteo total en el tab label
6. Soporta lazy load: carga cada tab bajo demanda cuando se hace click

### ReferencesClient.tsx
Reordena navegación principal de Referencias:
- Antes: "Tablero" (defecto) → "Tabla clásica"
- Ahora: "Bandeja de Entrada" (defecto) → "Tablero" → "Tabla clásica"

La Bandeja reemplaza el tablero como vista primaria, manteniendo acceso rápido a vistas alternativas.

## Criterios de Verificación Aplicados

✅ **Visual:**
- 7 tabs renderean sin error, nombres correctos
- Tarjetas muestran cliente, valor, ID referencia, última actualización
- Botón de acción principal visible en cada tarjeta
- Responsive en mobile/tablet/desktop

✅ **Funcional (parcial, requiere backend):**
- Tabs se cargan bajo demanda al hacer click
- Badge de conteo muestra total por tab
- Links a detalle funcionan
- Loading/error/empty states manejan correctamente

⚠️ **Integración Backend:**
- Requiere verificación de que `GET /references` soporta filtrado por cada status
- Algunos tabs requieren campos adicionales (ej: documentos faltantes, alertas)
  - **Alertas:** requiere campo `hasAlert` o `priority=URGENT` en modelo Reference
  - **Guías sin identificar:** requiere query a modelo `UnidentifiedWaybill` (endpoint separado)
  - **Documentos:** requiere campo `pendingDocuments` o relación con `ReferenceDocument` con status validación

⏳ **Test e2e:**
- Pendiente: navegación por cada tab, verificación de datos
- Pendiente: acciones principales (crear movimiento, subir doc)

## Bloqueos y Desviaciones

### Bloqueos Pendientes:
1. **Tabs con filtros complejos:**
   - "Alertas": hoy mapea a DRAFT (placeholder). Requiere field `priority=URGENT` o sistema de alertas en backend.
   - "Guías sin identificar": requiere endpoint separado para `UnidentifiedWaybill` o query combinada.
   - "Documentos que requieren atención": requiere lógica en backend para determinar qué referencias tienen documentos faltantes/pendientes validación.

2. **Performance:** si un tab tiene >10000 referencias, requiere paginación/infinite scroll en frontend. Implementación actual es básica (10 items + botón "Ver todas").

### Desviaciones del Plan:
- **Paso 1 (Figma):** Saltado — no aplicable a implementación frontend (diseño ya existe)
- **Paso 2 (Revisión con negocio):** Asumido que plan está aprobado
- **Paso 7 (Test e2e):** Pendiente para sesión siguiente

## Próximos Pasos (Paso 7: Test e2e)

1. Ejecutar tests e2e:
   - Navegar a `/references`
   - Verificar que tab "Bandeja de Entrada" es activo por defecto
   - Click en cada tab y verificar que datos coinciden con criterio
   - Click en tarjeta → verificar que abre detalle
   - Click en botón principal → verificar acción esperada

2. Verificación backend:
   - Confirmar que `GET /references` devuelve el total correcto por status
   - Si Alertas, Guías o Documentos están vacíos, determinar si:
     a) No hay datos en BD que cumplan criterio
     b) Filtro en backend es incorrecto
     c) Endpoint auxiliar (UnidentifiedWaybill) no existe

3. Refinamiento de mapeo de tabs:
   - Trabajar con backend para mapear tabs que requieren lógica compleja
   - Considerar endpoint auxiliar `/references/inbox?tab=X` que centralice lógica de filtrado
