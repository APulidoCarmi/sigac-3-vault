# Sesión 2026-07-14 — Procesamiento de reuniones y refinamiento de refactor

Relacionado: [[2026-07-14 - Prueba operación real desde ticket a facturación]], [[2026-07-13 - Revisión de pantallas (bandeja entrada, detalle referencia)]], [[2026-07-13 - Revisión de pantallas (operaciones, recinto, citas)]]

## Qué se trabajó

### Ingesta y depuración de 3 reuniones
Procesé secuencialmente 3 transcripciones de reuniones sobre el refactor de pantallas:

1. **2026-07-14 16:00** — "Prueba operación real desde ticket a facturación"
   - 1 hora 8 min, 13 asistentes
   - Revisión de bandeja de entrada, DGO, expediente aduanero, citas, operaciones
   - Decisiones: Estructura de 7 secciones en bandeja, límite de 3-5 referencias, orden de pestañas, DGO como source of truth

2. **2026-07-13 13:04** — "Revisión de pantallas (bandeja entrada, detalle referencia)"
   - 55 min, 2 asistentes (Ángel, Germán)
   - Redefinición de cards en bandeja, workflow Expediente → DGO, movimientos vinculados a DGO
   - Decisión crítica: Orden de pestañas (Expediente primero, DGO al final)

3. **2026-07-13 16:39** — "Revisión de pantallas (operaciones, recinto, citas)"
   - 45 min, 3 asistentes (Ángel, Germán, Carlos)
   - Botón de operaciones, nomenclatura recinto/previo, flujo de citas, locations
   - Bloqueador identificado: Citas necesita reunión con ejecutivos + almacén

### Creación de notas y reconciliación con backlog
- Creé 3 notas limpias en `Reuniones/` (depuradas conservadoramente, preservando decisiones y lógica de negocio)
- Reconcilié propuestas de tareas contra backlog existente (Tareas.md, que usa checkbox no Jira)
- Identificadas y documentadas 15 tareas de implementación

### Actualización de backlog
Tareas.md ahora contiene:
- 2 tareas antiguas completadas (bandeja de entrada, pedimentos)
- 13 tareas nuevas abiertas:
  - 3 de bandeja/límites (implementar límite visualización)
  - 5 de refactor de pantallas (reordenar pestañas, mover botón, migrar locations, movimientos dinámicos, citas)
  - 5 de DGO y operaciones (consolidación DGO, generación desde DGO, nomenclatura dinámica)
  - 2 de Expediente Aduanero (checklist automático, comparación trilateral)
  - 1 de tickets (pestaña Tickets)
  - 1 de validación (ETA + arribo)

### Grafo del baúl
Refrescado con `graphify update` + merge con grafo del repo. 59K+ nodos, 139K+ edges.

## Decisiones (con su porqué)

### 1. Bandeja de entrada — Límite de visualización (3-5 referencias)
**Por qué:** Evitar saturación. Objetivo: herramienta de priorización rápida, no listado largo. Ejecutivo entra, ve limpio/verde → día va bien.
**Cómo:** Mostrar TOP 3-5 más críticas, indicador de cantidad total, filtrar por prioridad.

### 2. Orden de pestañas en detalle de referencia
**Decidido:** Resumen → Ticket → **Expediente Aduanero** → Movimientos → Operaciones → Citas → Recinto → Previo → DGO **al final**
**Por qué:** DGO es "source of truth" solo para datos validados. Workflow: crear/subir en Expediente → validar → pasar a DGO → generar operación.

### 3. DGO como consolidación de 3 funcionalidades
**Decidido:** Unificar Factura + Packing List + Mercancía + Config. Pedimento en un solo lugar.
**Por qué:** Simplificar, centralizar config. de pedimento, tener una "fuente de verdad" con glosa digital automática.
**Equivalencia:** 1 DGO = 1 Pedimento.

### 4. Operaciones desde DGOs, no desde movimientos
**Decidido:** Cambiar lógica: antes operación ← movimiento; ahora operación ← DGO(s).
**Por qué:** DGO contiene datos ya validados y glosados, es más seguro para generar pedimento.

### 5. Movimientos dinámicos por tipo de tráfico
**Decidido:** UI muestra movimientos específicos de cada tráfico.
- Terrestre: Entrada, Subdivisión, Salida, Previo
- Aéreo: Manifiesto, Revalidación aérea, Asignación transporte, Importación, Exportación
- Marítimo: Revalidación marítima, Asignación transporte, Retorno vacío, Recuperación garantía, Importación (con sub-procesos), Exportación
**Referencia:** Documento de entendimiento "Revisión de flujo – 2026-07-09" define taxonomía de movimientos.

### 6. Nomenclatura dinámica Recinto/Previo por tráfico
**Decidido:** 
- Terrestre: "Verificación" (ubicación física + etapa logística)
- Marítimo/Aéreo: "Previo" (resultado de verificación física)
**Por qué:** Simplificar terminología para ejecutivos, adaptar a cada tráfico.

### 7. Citas — Necesita reunión con ejecutivos + almacén
**Bloqueador:** No está definido quién genera cita, cuándo, información obligatoria, integración con warehouse.
**Propuestas en discusión:** Generación automática al movimiento, enviar link al transportista/cliente, programación semanal (caso GKN).
**Acción:** Demo con GKN mañana (Germán ya coordinó).

### 8. Migración de locations desde SIGAC 2
**Decidido:** Traer warehouse, almacén fiscalizado, terminales portuarias a interfaz.
**Nota:** Necesita análisis de dónde sacar datos (SIGAC 2 vs catálogo actual).

### 9. Checklist automático en Expediente Aduanero
**Decidido:** Sistema lee perfiles (cliente, número de parte, aduana, agente) y genera checklist de documentos requeridos.
**Referencia:** Algoritmo: leer MOMP, perfil de cliente, determinar qué documentos van.

### 10. Comparación trilateral en Expediente Aduanero
**Decidido:** Cada documento muestra 3 vistas: PDF original, datos extraídos (ZEUS), datos glosados (validación).
**Por qué:** Asegurar coherencia, cumplir Anexo 22, detectar discrepancias.

## Aprendizajes / errores a no repetir

1. **Depuración conservadora es correcta:** Al procesar reuniones, mantuve señal y corté solo ruido evidente (saludos, small talk, problemas técnicos). Esto preservó decisiones clave que de otra forma se hubieran perdido.

2. **Documento de entendimiento es fuente de verdad:** La reunión 2026-07-13 16:39 mencionó que hay un documento "Revisión de flujo – 2026-07-09" que define movimientos por tráfico. Este es **authoritative** — debe usarse como referencia antes de implementar.

3. **No todas las decisiones se formalizan automáticamente como tareas:** Se tomaron muchas decisiones en las reuniones, pero no todas se convirtieron a tareas implementables. Necesité hacer un segundo pase para formalizar cada decisión de negocio como tarea de ingeniería.

4. **Límite de visualización es medida rápida, no final:** La bandeja de entrada con límite 3-5 referencias es temporal. Necesita validación con Enrique + Excel de estructura antes de llamarla "hecha".

5. **Bloqueadores claros:** Citas es un bloqueador explicito (reunión needed), locations es bloqueador medio (requiere análisis), comparación trilateral requiere integración ZEUS confirmada.

## Pendientes

### Para próxima sesión
1. **Crear Excel de estructura de bandeja de entrada** — Con campos y validaciones de cada sección (Alertas, Accionables, Curso automático, Esperando a terceros, Por identificar, Consolidados, Pedimentos pagados).

2. **Validar bandeja de entrada con Enrique** — Mostrar Excel + UI, obtener feedback, iterar.

3. **Completar flujo Expediente → DGO** — Sesión de diseño con Germán para clarifcar:
   - Crear factura en Expediente (con o sin documento)
   - Subir documento, ZEUS extrae
   - Validar datos glosados
   - Factura pasa a DGO
   - Si múltiples DGOs, usuario selecciona cuál

4. **Reunión de citas** — Con Enrique López y Yolanda (warehouse) para definir flujo. Demo con GKN está coordinada para mañana.

5. **Análisis de migración locations** — Dónde sacar datos de SIGAC 2, catálogo actual, estructura destino.

6. **Definir movimientos aéreo** — Usar documento de entendimiento para formalizar qué movimientos van en aéreo (marítimo ya está en documento).

### Notas en el baúl que ganaron contexto
- [[2026-07-14 - Prueba operación real desde ticket a facturación]] — Completa, con 10 decisiones clave
- [[2026-07-13 - Revisión de pantallas (bandeja entrada, detalle referencia)]] — Completa, con flujo Expediente-DGO
- [[2026-07-13 - Revisión de pantallas (operaciones, recinto, citas)]] — Completa, con bloqueadores identificados

Todas enlazadas a Tareas.md para trazabilidad.

## Resumen para la próxima sesión

Sesión productiva:
- ✓ 3 reuniones procesadas y documentadas
- ✓ 15 tareas identificadas, reconciliadas y priorizadas
- ✓ Bloqueadores claros (citas, locations, flujo Expediente-DGO)
- ✓ Grafo refrescado

El refactor está en etapa de **formalización de decisiones → tareas de implementación**. Próximas acciones son más tácticas: Excel, validaciones, reuniones de alineación con stakeholders (Enrique, warehouse, ejecutivos).

Contexto principal consolidado en baúl. Puedes descargar contexto principal con `/clear` si la sesión se agota.
