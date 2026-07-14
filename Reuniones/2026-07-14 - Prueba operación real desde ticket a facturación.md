# Reunión 2026-07-14 — Prueba operación real desde ticket a facturación

## Asistentes
- Angel Huberto Pulido Burgos (Ángel)
- German Castro (Germán)
- Enrique Lopez
- Elian Shair Armendariz Puch
- Fernando Angel Lopez Soto
- Carlos Alexis Galaviz Rosas
- Enrique Garza
- Perla Lopez

## Decisiones y Acuerdos

### Refactor de pantallas completado
Se reorganizó la interfaz de manera más lógica tras la adición de campos de referencia. El objetivo era reducir navegación innecesaria entre pantallas.

### Estructura de bandeja de entrada del ejecutivo
Se definió una propuesta de 7 secciones para la bandeja de entrada (dashboard de ejecutivo):

1. **Alertas** — Referencias con situaciones críticas:
   - ETA vencida: rojo (≤5 días)
   - ETA próxima: amarillo (5-10 días)
   - ETA lejano: verde (>10 días)
   - Propuesta mejorada: validar no solo fecha ETA, sino fecha de **arribo real** cuando existe. Si ETA venció pero no arribó, permanece en rojo. Una vez que arriba, pasa a otro proceso (verificación/clasificación).

2. **Accionables** — Referencias en proceso sin alertas pero con trabajo pendiente:
   - Operación inicializada pero incompleta
   - Criterios pendientes de definir formalmente (Enrique debe proporcionar lista completa)
   - Facilita visibilidad de lo atrasado que se olvidó

3. **Curso automático** — Operaciones en flujo automatizado:
   - Operaciones que iniciaron con botón "Iniciar operación automática"
   - Muestra progreso visual (porcentaje o stepper) sin necesidad de entrar una por una
   - Visualizar dónde va cada operación en el proceso automático

4. **Esperando terceros** — Procesos atrasados por dependencias externas:
   - Falta revisión/clasificación de almacén
   - Falta permiso del cliente
   - Ejemplo: "clasificación pendiente hace 3 días"

5. **Por identificar** — Movimientos generados por almacén sin vincular a referencia:
   - Permite detectar movimientos huérfanos y vincularlos o crear referencia

6. **Consolidados** — Consolidados semanales/mensuales pendientes por cerrar:
   - Cada uno taggueado con tipo (semanal/mensual)
   - Recordatorio de cierre pendiente

7. **Pedimentos pagados sin modular** — Alerta de pedimentos próximos a vencer:
   - Pedimento está pagado pero no se moduló en sistema
   - A punto de cumplir vigencia (3 días es crítico en exportación)
   - Recordatorio urgente antes de vencer

**Decisión**: Crear Excel con estructura formal (como se hace en marítimo/aéreo) definiendo campos y validaciones de cada sección, para validación con Enrique.

### Validación mejorada de ETA y fecha de arribo
Necesidad: El sistema no debe marcar alerta roja solo por ETA vencida si la mercancía aún no ha arribado. Debe:
- Revisar combinación de ETA + fecha de arribo real
- Si ETA ≤5 días Y no ha arribado = rojo (mercancía atrasada)
- Si ya arribó = pasa a verificación/clasificación (otro proceso)
- Definir explícitamente los procesos posteriores al arribo (verificación en frontera, captura, clasificación, etc.) para validar en alertas

### Expediente dinámico por tipo de tráfico
Los movimientos varían según el tipo de tráfico (terrestre, aéreo, marítimo). Cada referencia mostrará dinámicamente los tipos de movimiento correspondientes: entrada, subdivisión, salida, previo (terrestre), o los específicos de aéreo/marítimo.

### DGO — Datos Glosados para Operación
Cambio fundamental en cómo se generan operaciones:

**Antes**: Operación se generaba desde un movimiento.

**Ahora**: Operación se genera desde uno o varios **DGO** (Datos Glosados para Operación).

**Qué es un DGO**:
- Consolidación de 3 funcionalidades antiguas en una sola:
  1. Alta de factura (que estaba en formulario de referencia)
  2. Pestaña de mercancía
  3. Configuración de pedimento
- Equivalencia: **1 DGO = 1 Pedimento**
- Si una referencia tiene dos facturas con claves de pedimento diferentes, el sistema detecta y puede crear 2 DGO/2 pedimentos automáticamente (o permitir hacerlo manualmente)
- Validación en tiempo real: al editar cualquier dato, se corre glosa digital automáticamente y marca errores (rojo si datos no cuadran con documentos, reglas MOMP, etc.)
- Cuando todos los DGO están en verde = glosa digital pasó, datos verificados contra expediente y reglas
- Source of truth: los datos en DGO son lo que se usa para generar el pedimento

**Decisión**: Simplificar formulario de crear referencia quitando campo de facturas (ya no se suben ahí). Se suben desde dentro de DGO cuando se está rellenando el expediente.

### Verificación vs. Previo — Nomenclatura
- **Terrestre**: Llamar "Verificación" a la inspección física
- **Marítimo**: Llamar "Previo" a la inspección física
- Ambos muestran resultado digitalizado del previo/inspección (formato con foto, etapas, detalles)

### Citas — Pendiente de revisión con warehouse
Propuesta: permitir que ejecutivos programen citas desde referencias.

**Estado actual**: Pocas citas, se usan más notificaciones. PL hace citas pero se comunican por correo con almacén (copia a ejecutivos), no en sistema.

**Propuesta mejorada**:
- Eliminar emails, generar citas en sistema
- Algunos clientes envían programación semanal (ej. GKN): idealmente se carga un Excel con múltiples citas y se generan de una vez
- Ejecutivos deben poder ver citas vinculadas a referencias, cuándo son, confirmación de almacén
- Investigar si QR de cita permite hacer cita desde sistema externo de warehouse o si hay que usar formulario dentro del sistema

**Bloqueador**: Necesita reunión con warehouse para definir flujo de generación, confirmación y visualización. Carlos mencionó que Yoli (warehouse) ha comentado sobre esto.

## Lógica de Negocio

### Procesos después del arribo de mercancía
Enrique describió secuencia de procesos que ocurren después de que mercancía arriba a almacén:
1. Recepción/arribo
2. Descarga
3. Captura (registro en sistema)
4. Verificación/inspección
5. Clasificación
6. Expediente completo (documentos validados)

Estos procesos son lo que **debe validar el campo de alertas** para determinar cuándo hay atraso en alguno de ellos.

### Criterios para accionables — Pendiente detalle
Enrique mencionó que envió al chat lista de procesos/criterios que definen "accionable ahora", pero Angel requiere una documentación formal y completa de todos ellos para implementar la lógica de tagging.

## Problemas y Bloqueadores

### Opciones faltantes en menú
Faltan varias opciones del menú que antes aparecían:
- `warehouse`
- `operations`
- `movimientos`

Nadie reconoce haberlas quitado explícitamente. Pueden haber sido removidas en cambios de código reciente. **Necesita revisar logs de Git** para saber quién las quitó y por qué.

### Menú de referencias desaparecido del acceso principal
Cuando Enrique intenta acceder al módulo de referencias desde el menú, no aparece. Carlos dice que algo se movió para producción.

**Solución temporal**: Usar link directo para acceder a referencias sin pasar por el menú.

**Necesario**: Restaurar acceso desde menú o investigar qué sucedió.

### Módulo de referencias en desarrollo
La estructura refactorizada (bandeja, DGO, expediente, etc.) está solo en **rama de desarrollo**. La rama de producción sigue con interfaz anterior. Esto hace que Enrique no pueda ver ni validar los cambios en tiempo real en producción.

## Tareas

*(Pendientes de proponer tras reconciliación con backlog)*

## Temas descartados por irrelevantes
- Problemas técnicos del meet (audio cortado entre Ángel y Enrique)
- Saludos de asistentes nuevos
- Confirmaciones genéricas ("okay", "sí", "ajá")
- Contexto vago de "esta mañana estaba trabajando"
