# Reunión 2026-07-13 — Revisión de pantallas (operaciones, recinto, citas)

**Hora:** 16:39 MST

## Asistentes
- Angel Huberto Pulido Burgos (Ángel)
- German Castro (Germán)
- Carlos Alexis Galaviz Rosas (Carlos)

## Decisiones

### Operaciones — Botón de selección de múltiples DGOs
**Cambio importante:** El botón "Crear operación" debe estar **fuera de un DGO individual**.

Actualmente: Botón dentro de cada DGO
**Correcto:** Botón fuera, permitiendo seleccionar uno o varios DGOs para crear la operación

**Flujo:** Seleccionar DGOs → Clic en "Crear operación" → Sistema trae datos de los DGOs seleccionados

**Razón:** Operación se crea a partir de uno o varios DGOs (pedimentos). Deben seleccionarse primero.

### Orden de pestañas en detalle de referencia — CONFIRMADO
Orden final decidido:
1. **Resumen** (con timeline a un lado)
2. **Ticket** (crear o vincular aquí, o desde crear referencia)
3. **Expediente Aduanero** (donde subes documentos, ZEUS extrae, glosa digital)
4. **Movimientos** (creación y monitoreo, dinámicos por tráfico)
5. **Operaciones** (desde DGOs seleccionados)
6. **Citas** (pendiente definición de flujo)
7. **Recinto** (ubicación física + etapa logística)
8. **Previo** (resultado de verificación física)

### Recinto — Propósito y diferencia con Previo
**Propósito:** Permitir que ejecutivo consulte ubicación física y estatus logístico sin solicitar a almacén

**Adaptabilidad por tráfico:**
- **Terrestre:** Almacén (ej. Herada, Bronville)
- **Aéreo:** Almacén fiscalizado (ej. Querétaro)
- **Marítimo:** Terminal portuaria (ej. Manzanillo, Lázaro Cárdenas, Veracruz, Tampico)

**Diferencia:**
- **Recinto:** Ubicación física + etapa logística (dónde está, en qué proceso está)
- **Previo:** Resultado de verificación física + inspección de mercancía

Se mantienen **separadas** (no se unifican).

### Recinto — Datos de locations
Necesita migración de locations a interfaz. Datos disponibles en catálogo:

**Terrestre (Almacén):**
- Herada
- Bronville
- (Posiblemente más)

**Aéreo (Almacén fiscalizado):**
- Querétaro
- (Otros por confirmar)

**Marítimo (Terminales):**
- Manzanillo
- Lázaro Cárdenas
- Veracruz
- Tampico
- (Otros según catálogo de locations)

### Tickets — Generación y vínculo
- Se puede **generar ticket** desde detalle de referencia O desde crear referencia
- Se puede **vincular a ticket existente** (mismo cliente)
- Un ticket puede vincularse a **varias referencias**, pero una referencia a **máximo 1 ticket**

### Citas — Flujo SIN DEFINIR (Revisión pendiente)
**Problema:** No está claro cómo se generan las citas ni cuándo el ejecutivo interviene.

**Propuestas en discusión:**

1. **Generación automática al crear movimiento de entrada**
   - Problema: No tenemos toda la información (cliente, transportista, placas, B/L, etc.)

2. **Enviar link al transportista/cliente para llenar la cita**
   - Ejecutor llena información conocida
   - Transportista/cliente completa información adicional
   - Carlos sugiere esto como viable

3. **Programación semanal de citas**
   - Caso GKN: Envía programación semanal de entregas
   - Sistema genera citas en batch
   - Ejecutivos confirman en dashboard
   - Almacén tiene citas pre-programadas

**Restricciones:** Almacén necesita información completa de la cita para que sea válida (cliente, B/L, transportista, placas, etc.)

**Acción decidida:** Reunión con ejecutivos y almacén juntos (Enrique López y Yolanda) para definir flujo de citas. Demo con GKN mañana a las 2 pm (hora Germán) / 12 pm (hora Carlos).

**Nota importante:** La cita es un tema **legal y logístico**, no solo de conveniencia. Deber ser > comodidad.

### DGO — Confirmado
- Equivalente a un **pedimento**
- Contendrá: Configuración de aduana, clave de pedimento, patente, régimen, destino
- Contendrá: Facturas, mercancía, partidas
- Contendrá: Incrementables y identificadores a nivel pedimento
- Una referencia puede tener **múltiples DGOs**
- Operación se crea a partir de **uno o varios DGOs**

### Componente de documentos — Refactorización
Carlos está refactorizando el componente de documentos en crear referencia:
- Debe ser **reutilizable** en Expediente Aduanero también
- No debe duplicar funcionalidad de ZEUS (que ya extrae datos automáticamente)
- **Acuerdo:** Carlos lidera el desarrollo, Germán lo integra después
- Comportamiento se mantiene igual, solo refactorización de UI/componente

## Lógica de Negocio

### DGO como "fuente de verdad"
- En **Expediente Aduanero:** Se crean facturas manualmente, se suben documentos, ZEUS extrae datos
- Una vez validados los datos → Se llevan a **DGO**
- **DGO** contiene solo datos verificados y glosados
- De ahí se genera la operación

### Comparación de datos (Glosa digital)
- Datos extraídos por ZEUS se comparan contra datos en DGO
- Si hay inconsistencias, se marcan como errores
- Ejecutivo revisa y valida antes de pasar a operación

## Problemas/Pendientes

### Citas — Flujo no definido
Necesita sesión de trabajo con ejecutivos (Enrique, Yolanda) y almacén para definir:
1. ¿Quién genera la cita? (ejecutivo, transportista, sistema automático)
2. ¿Cuándo se genera? (al crear movimiento, al crear referencia, semanal)
3. ¿Qué información es obligatoria? (cliente, transportista, B/L, placas, etc.)
4. ¿Cómo se integra con el sistema de warehouse?

Demo con GKN está programada para validar una propuesta.

### Movimientos por tráfico — Aéreo pendiente
**Definido:**
- **Terrestre:** Entrada, Subdivisión, Salida, Previo
- **Marítimo:** Según Excel (documento disponible con Ángel)

**Pendiente:**
- **Aéreo:** No está definido aún. Necesita:
  - Revisar documento de entendimiento (que ya define qué movimientos van en cada tráfico)
  - Confirmar con Ángel qué movimientos van en aéreo
  - Implementar dinámicamente según tráfico

**Acción:** Germán pide a Ángel que use documento de entendimiento y Excel de movimientos para hacer las modificaciones necesarias en la UI, asegurándose que aéreo muestre los movimientos correctos.

### Integración de ZEUS con componente de documentos
**Pendiente aclaración:** ¿El componente de documentos refactorizado debe integrar ZEUS directamente o ya viene separado?
- Carlos menciona que el componente anterior fue hecho antes de la interfaz de chat de ZEUS
- Necesita revisar si se sigue necesitando el componente antiguo o si solo va ZEUS

### Context decay en LLM
**Observación:** Germán nota que el contexto de Ángel es muy grande y puede estar confundiendo al LLM (no se consideró el documento de entendimiento que define movimientos). 
- **Sugerencia:** Mantener contexto más acotado y enfocado
- Priorizar documentos de especificación (Excel, documento de entendimiento) como "fuente de verdad"

## Temas descartados por irrelevantes
- Problemas de conexión (Germán no veía pantalla)
- Saludos, bienvenidas
- Discusión sobre INE/identificación (off-topic)
- Charla sobre grafo de conocimiento (tangencial)
- Planes internos de Germán
- Small talk
