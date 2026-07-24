# Sesión 2026-07-21 — Análisis de flujo de referencias (visita Querétaro)

Relacionado: [[2026-07-16 - Flujo diario de manifiestos, régimen y candados (Querétaro)]], [[2026-07-20 - Recap visita Querétaro]], [[2026-07-10-refactor-flujo-ejecutivo]], [[2026-07-15-movimientos-dinamicos-aereo-maritimo]]

## Qué se trabajó

- **Snapshot as-is** del código actual (front `carmi-digital` + back `carmi-odin-api-v2`, rama `feat/movimientos-dinamicos-por-trafico`, working tree sin commitear) para 4 flujos: creación de referencia, movimiento/detalle de referencia, operaciones, pedimento.
- **Reconciliación de 3 fuentes**: el flujo objetivo dictado por el usuario (10 puntos, basado en la visita a Querétaro), el snapshot as-is, y los sub-planes ya existentes del paraguas `refactor-flujo-ejecutivo` (sp01–sp20) + el plan `2026-07-15-movimientos-dinamicos-aereo-maritimo`.
- Resolución conversacional de 15 preguntas abiertas de esa reconciliación, más 4 de seguimiento sobre puntos ambiguos.
- Generación de **14 tareas nuevas en `Tareas.md`**.
- **Deep-dive técnico** sobre el mecanismo de "Subdivisión" de Shipment (SP-07) para evaluar si aplica al caso de régimen mixto dentro de una guía aérea — conclusión: NO se debe reusar; el split por régimen ya tiene hogar natural en `Dgo`/`Invoice`.
- Discusión de UX del tablero de "Guías" nuevo vs. la sección "Por identificar" del Inbox existente — el usuario confirmó que debe ser pantalla separada, pero queda abierta la duda de si absorbe o convive con "Por identificar".
- **7 tareas adicionales propuestas en conversación pero NO escritas aún a `Tareas.md`** — pendientes de confirmación del usuario antes de persistir (ver Aprendizajes).

## Commits relevantes

Ninguno — sesión 100% de análisis, sin tocar código. Ambos repos (`carmi-digital`, `carmi-odin-api-v2`) siguen en `feat/movimientos-dinamicos-por-trafico` con el mismo WIP sin commitear que tenían al iniciar la sesión.

## Decisiones (con su porqué)

### 1. Alcance de esta ronda: solo tráfico aéreo (Querétaro)
Marítimo queda sin definir — no se documentó (o no ocurrió) la reunión del 21-jul que iba a tratarlo.

### 2. Prealerta se generaliza a todos los clientes
Hoy es exclusiva de Tyco en la operación real; se generaliza por si algún otro cliente lo requiere a futuro.

### 3. Tablero de "Guías" (pre-referencia) es pantalla nueva y separada
No es una evolución de "Por identificar" del Inbox (`InboxDashboard.tsx`, `UnidentifiedWaybill`) ni el `air-manifest` ya existente (SP-10, que vive DENTRO de una Referencia ya creada). Es una tercera superficie: ingesta del manifiesto diario, control de qué guías se trabajan hoy, comparación con prealerta, disparo de creación de referencia. **Pendiente:** si absorbe o convive con "Por identificar".

### 4. Guía como entidad: usar `AirManifest` (ya existente) + `UnidentifiedWaybill` como dos etapas del mismo ciclo de vida
`AirManifest` ya trae etapas de previo (`assignedToPrevioTableAt`, `readyForPrevioAt`) y vive dentro de una Referencia. `UnidentifiedWaybill` cubre la etapa previa a identificación. Se necesita un puente explícito entre ambos, no una entidad nueva desde cero.

### 5. Régimen mixto en una guía se resuelve vía DGO/Invoice, NO vía Subdivisión de Shipment
La investigación técnica confirmó: el motor de Subdivisión (SP-07) no tiene noción de "motivo" (solo un string libre en metadata), y el régimen ya vive de forma dedicada en `Dgo`/`Invoice` (comentario del propio modelo: "se separa en N DGO si hay N claves de pedimento distintas"). Además, aéreo usa `AirManifest` como unidad de previo, no `Shipment` — no existe el puente `AirManifest↔Shipment` que sí existe en terrestre, así que reusar Subdivisión exigiría construir ese puente solo para un caso que ya tiene mejor solución. La detección del régimen mixto la hace el ejecutivo por experiencia (no el equipo de previo), revisando el manifiesto o la página de la paquetería.

### 6. Llegada parcial (OS&D, mismo régimen) se maneja directo a nivel DGO/factura/partida
Ejemplo trabajado: guía de 20 partidas, llegan 15 hoy y 5 mañana. Si se saca todo junto mañana, no hay subdivisión, solo tracking de faltante (`MovementInvoice.cantidadEntrada`/`cantidadPrevista`, ya soportado). Si se saca en 2 momentos, son 2 DGOs de la misma Referencia — ya soportado conceptualmente por el modelo.

### 7. Previo se remodela de 1:1-Referencia a 1:1-guía
Una referencia puede tener N guías; el previo se hace por guía, no por referencia completa.

### 8. Validación de guía máster + régimen homogéneo también a nivel Referencia
Hoy solo se valida al crear la Operación (DGO); se agrega también al crear/editar la Referencia.

### 9. "Citas" se quita solo del menú aéreo, no globalmente

### 10. Diferidas explícitamente (no prioridad de esta ronda)
Candado de catálogo (autorización de clasificación por cliente), documentos obligatorios reales por cliente vía MOMP (gap ya señalado por SP-04), ancla del previo en marítimo (guía vs. contenedor), equivalente marítimo del tablero de guías. Generadas como tareas en `Tareas.md` con nota explícita de "pendiente de análisis".

## Aprendizajes / errores a no repetir

1. **No persistir conclusiones a `Tareas.md`/`Planes/` apenas se formulan durante el análisis.** El usuario rechazó una edición a mitad de discusión ("vamos a platicarlo primero"). Regla aplicada desde entonces: proponer en el chat, escribir solo tras confirmación explícita o cuando el usuario pide directamente "genera las tareas".

2. **"Manifiesto" y "Subdivisión" son términos sobrecargados en este dominio.** Cada uno ya significa algo distinto y ya construido en código (air-manifest post-referencia, Subdivisión de Shipment terrestre) además de lo que el negocio quiere decir ahora. Cualquier conversación de diseño nueva debe desambiguar explícitamente antes de asumir que se habla de lo mismo.

3. **Antes de asumir que hace falta una entidad/mecanismo nuevo, revisar si el dominio ya tiene un "hogar natural".** El caso de régimen mixto casi se diseña sobre Shipment/Subdivisión por instinto, pero ya existía un mecanismo dedicado (`Dgo`) mejor alineado. Vale la pena un paso explícito de "¿esto ya vive en algún lado?" antes de proponer construir algo nuevo.

## Pendientes

### Para próxima sesión
1. **Resolver si el tablero de "Guías" absorbe o convive con "Por identificar" del Inbox** — sigue abierta; no se retomó en la sesión del 2026-07-22.
2. ~~Confirmar y escribir a `Tareas.md` las 7 tareas propuestas~~ — **Resuelto 2026-07-22**: revisadas, corregidas (varias con cambios sustanciales tras validar contra código real) y escritas, ver [[2026-07-22 - Refinamiento de tareas Querétaro]].
3. ~~Decidir estructura de plan formal~~ — **Resuelto 2026-07-22**: el usuario confirmó que es **iniciativa aparte, no extiende el paraguas `refactor-flujo-ejecutivo`**. Falta usar `/plan` cuando se decida pasar de tareas sueltas a plan formal.
4. Las tareas marcadas "pendiente de análisis, no prioridad actual" en `Tareas.md` (candado, MOMP, previo marítimo, equivalente marítimo del tablero de guías, catálogo de movimientos aéreo) siguen sin dueño ni fecha — quedan ahí hasta que el negocio las priorice. Ver también nota de posible inconsistencia en la tarea de catálogo de movimientos aéreo, señalada en [[2026-07-22 - Refinamiento de tareas Querétaro]].

### Notas en el baúl que ganaron contexto
- [[2026-07-16 - Flujo diario de manifiestos, régimen y candados (Querétaro)]]
- [[2026-07-20 - Recap visita Querétaro]]
- `Tareas.md` — 14 tareas nuevas agregadas, trazables a estas dos reuniones (o al sub-plan SP-04 en el caso de MOMP)

## Resumen para la próxima sesión

Sesión de análisis puro (sin tocar código): se construyó un snapshot as-is, se reconcilió contra las reuniones de Querétaro y los planes existentes, se resolvieron ~19 preguntas de diseño en conversación, y se llegó a una arquitectura clara para el caso de régimen mixto (Dgo/Invoice, no Shipment). Quedan 7 tareas por confirmar y escribir, 1 pregunta de UX abierta (Inbox vs. tablero de Guías), y la decisión de cómo estructurar el/los plan(es) formales antes de pasar a `/plan` → `/implementa`.

Contexto principal consolidado en baúl. Puedes descargar contexto principal con `/clear` si la sesión se agota.
