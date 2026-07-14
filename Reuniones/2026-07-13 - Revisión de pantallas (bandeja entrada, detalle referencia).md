# Reunión 2026-07-13 — Revisión de pantallas (bandeja entrada, detalle referencia)

**Hora:** 13:04 MST

## Asistentes
- Angel Huberto Pulido Burgos (Ángel)
- German Castro (Germán)

## Decisiones

### Bandeja de Entrada — Redefinición de cards
**Problema identificado:** Las 3 cards iniciales ("por identificar", "cartas sin firmar", "arribos") muestran referencias de forma poco clara.

**Decisión:**
- **Card 1**: Referencias esperando información del cliente o transportista (cualquier dependencia externa)
- **Card 2**: Referencias con documentos faltantes (renombrando "cartas sin firmar" a algo como "Documentos que requieren atención")
- **Card 3**: Referencias que ya tienen cita y están por llegar (arribos)
- **Sección abajo**: Guías sin identificar (movimientos que llegaron al almacén sin saber de quién son)

### Estructura de Listado de Referencias
Se elimina la pantalla separada de "Bandeja de entrada" y se integra en "Listado de referencias" con 2 tabs:
- **Tab 1 "Bandeja de entrada"**: Vista de dashboard/cards (la que se acaba de redefinir)
- **Tab 2 "Tabla"**: Vista clásica de tabla con todas las referencias

**Razón:** Evitar duplicación de visualización de lo mismo.

### Detalle de Referencia — Orden de pestañas (CRÍTICO)
**Problema identificado:** El orden propuesto era incorrecto.

**Orden INCORRECTO (propuesto inicialmente):**
1. DGO
2. Movimientos
3. Resto de pestañas

**Orden CORRECTO (decidido):**
1. **Expediente Aduanero** (primero - es donde subes documentos)
2. [Otras pestañas]
3. **DGO** (último - es la fuente de verdad, solo va lo validado y glosado)

**Porqué:** DGO es la "fuente de verdad" para el pedimento. Solo debe contener datos ya validados. El workflow debe ser: 1) Crear factura en Expediente, 2) Subir documento, 3) Validar, 4) Pasar a DGO.

### Detalle de Referencia — Pestañas definidas
- Resumen (línea de tiempo, info de referencia)
- **Expediente Aduanero** (primero)
- DGO
- Movimientos
- Operaciones
- Citas
- Recinto
- Previo
- **NO incluir**: Instrucciones/Tickets como tab aquí

**Razón para NO incluir Instrucciones:** Son configurables a nivel formulario de crear referencia (donde creas o vinculas un ticket). En detalle de referencia solo mostrarías el ticket vinculado, no los generarías ahí.

### Crear Referencia — Simplificación
- **Step 1**: Datos básicos (cliente, tipo de operación, tipo de tráfico, ETA, orden de compra, etc.) + Opción de vincular/crear ticket
- **Step 2**: Subir documentos
- **Eliminado:** El campo de upload de facturas del formulario inicial (se maneja desde Expediente Aduanero)

### DGO — Ubicación en workflow
**Cambio en lógica de entrada de datos:**
- **Crear factura MANUALMENTE**: Esto va en **Expediente Aduanero**, no en DGO
- **Subir documento de factura**: Va en Expediente Aduanero
- **ZEUS extrae datos**: Muestra datos extraídos + resultado de glosa digital
- **Factura validada pasa a DGO**: Solo cuando está todo correcto y glosado
- **Múltiples DGOs**: Si una factura puede ir a varios DGOs (diferentes claves de pedimento), el usuario selecciona a cuál

**Comparación de datos:** Los datos extraídos por ZEUS se comparan contra lo que hay en DGO. Si ambos existen en la BD, se comparan para detectar inconsistencias (glosa digital).

### Expediente Aduanero — Estructura
Debe mostrar checklist automático de documentos requeridos basado en:
- Perfil de cliente
- Perfil de número de parte
- Perfil de aduanas
- Perfil de agente aduanal

**Cada documento debe mostrar 3 vistas/estados:**
1. **PDF original**: El archivo subido
2. **Datos extraídos** (por ZEUS): Los datos que el sistema automático extrajo
3. **Datos glosados**: La validación/resultado de glosa digital (si está bien, si hay inconsistencias)

**Ejemplo visual:** Cada documento como una "tarjeta" donde puedes ver: ✓ PDF | ✓ Datos extraídos | ✗ Datos glosados (falta revisión). Una factura completa tendría los 3. Una NOM incompleta tendría solo el PDF.

**Múltiples versiones:** Puedes tener varias NOMs, varias facturas. El sistema permite múltiples versiones del mismo tipo de documento.

### Movimientos — Vinculación a DGO
- Los movimientos quedan **vinculados a un DGO por default**
- Al crear movimiento: seleccionar DGO → indicar qué **facturas** se da entrada (no el DGO completo)
- **Ejemplo:** Un DGO con 1 factura, pero 2 movimientos de entrada (si la mercancía llega en dos entregas)

### Movimientos — Dinámicos por tráfico
**Terrestre:**
- Tabs: Entrada, Subdivisión, Salida, Previo
- Se generan principalmente Entrada y Previo, pero se monitorean/agrupan todos

**Marítimo:**
- Movimientos diferentes (revalidar, etc.)
- **Pendiente:** Revisar documentación de qué movimientos existen en marítimo vs aéreo, cuáles son seriales, cuáles paralelos

**Aéreo:**
- Movimientos específicos a definir

### Tickets/Instrucciones
- **En crear referencia**: Opción de crear nuevo ticket o vincular uno existente (del mismo cliente)
- **En detalle de referencia**: Solo mostrar que está vinculado a un ticket, con opción de cambiar el vínculo si no se creó en el formulario
- **Regla:** Una referencia = máximo 1 ticket. Un ticket = puede estar vinculado a varias referencias.

### Componente reutilizable de documentos
El mecanismo para subir y gestionar documentos debe ser **idéntico** en:
1. Crear referencia (Step 2)
2. Expediente Aduanero (dentro de detalle de referencia)

Ambos usan el mismo componente para que el usuario pueda editar/subir documentos desde cualquier lugar.

## Lógica de Negocio

### Workflow correcto de Expediente → DGO
1. **Usuario crea factura manualmente en Expediente**: Llena número de factura, partidas, datos que ya tiene (sin documento si aún no llegó)
2. **Usuario sube documento de esa factura a Expediente**: ZEUS extrae automáticamente los datos
3. **Sistema muestra 3 vistas**: PDF original, datos extraídos, datos glosados (validación)
4. **Una vez validado todo**: Esa factura "pasa" o se vincula al DGO
5. **En DGO**: Aparecen solo las facturas validadas, con datos ya confirmados
6. **Glosa digital automática**: Al editar algo en DGO, se compara contra los datos extraídos para detectar inconsistencias

### Documentos requeridos — Algoritmo
El sistema debe leer:
- MOMP (reglas aduanales)
- Perfil del cliente
- Perfil del número de parte
- Perfil de aduanas
- Perfil de agente aduanal

Y generar automáticamente una lista de documentos obligatorios. Ej: Si perfil de número de parte requiere NOM, aparece NOM en el checklist.

## Problemas/Pendientes

### Documentación de movimientos
**Falta revisar:** Qué movimientos existen específicamente en marítimo vs aéreo, cuáles son seriales (uno debe terminar antes de que empiece el siguiente), cuáles son paralelos.

### Workflow de Expediente-DGO
**En construcción:** Germán comenzó a dibujar el flujo exacto pero la reunión terminó. Necesita sesión de follow-up.

### Definición de "cartas sin firmar"
**Duda:** ¿Son proformas? ¿Son documentos que el cliente debe firmar? Necesita aclaración con el equipo operativo.

## Temas descartados por irrelevantes
- Problemas técnicos de conexión
- Saludos/despedidas
- Comentario sobre otra reunión de Germán
- Small talk
