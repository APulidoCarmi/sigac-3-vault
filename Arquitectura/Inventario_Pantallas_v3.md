# Inventario de Pantallas — Flujo v2 del Ejecutivo (SIGAC 3.0)
## v3 — actualizado con el Documento de Entendimiento revisado por German

**Alcance:** solo pantallas que usa el **ejecutivo de cliente contento**. No se diseñan pantallas propias de almacén, clasificación, tesorería, glosa **ni trámite y despacho** (este último confirmado como rol aparte, aplica principalmente en marítimo/aéreo) — cuando el ejecutivo necesita ver algo que viene de esos roles, se marca **solo lectura**.

**Qué cambió respecto a la v2 de este inventario** (por los ajustes al Documento de Entendimiento):
1. **Referencia no es solo un wizard** — tiene 3 funciones (Expediente, DGO, punto de partida) y 3 piezas de UI (wizard, detalle, listado). Se agrega **Listado de Referencias** como pantalla nueva.
2. **DGO ahora muestra consistencia contra el Previo** (el glosado con Zeus valida contra los datos de la verificación física), no solo contra otros documentos.
3. **Movimiento se reorganiza por tipo de tráfico** — Entrada/Salida ya no son genéricos, son específicos de Importación Terrestre. Aéreo y marítimo tienen sus propios movimientos especializados. Esto cambia radicalmente el contenido de la Fase 3.
4. **Mercancía se fusiona con DGO** — deja de ser un tab separado; la captura de facturas/partidas vive dentro de Datos Glosados.
5. **Se agrega el tab "Previo"**, separado de "Recinto" — muestran cosas distintas (ver nota en Fase 2).
6. **Modelo definitivo de 3 planos (el cambio más profundo):** Referencia/DGO/Pedimento = plano documental-legal (1 DGO = 1 pedimento, misma unidad, no "se generan" uno al otro). Movimiento = plano físico-logístico, vinculado al DGO de forma **flexible, no 1 a 1**. Operación = nivel que **agrupa** DGO/pedimentos — no los crea ni transforma. Esto cambia cómo describo DGO (#6), Movimientos (#8), Operaciones (#11) y el wizard de Crear Operación (#21).
7. **Incrementables y Decrementables (#9) e Identificadores (#10) dejan de ser tabs de la referencia** — pasan a ser campos configurables **por DGO** (#6), pudiendo seleccionar varios DGOs para darles los mismos valores o valores distintos.

**Cómo leer la columna Estado:** 🟢 Ya existe en D1 · 🟡 Existe en D1, con cambios pendientes · 🔵 Nueva.
**Columna Quién actúa** (nueva, importante para no salirnos de alcance): **Ejecutivo** (diseñamos esto) · **Solo visibilidad** (el ejecutivo solo lo consulta, otro rol actúa) · **⚠️ Confirmar** (no está claro todavía).

---

## Fase 0 — Entrada al trabajo

| # | Pantalla | Para qué se usa | Contiene | Justificación | Estado |
|---|---|---|---|---|---|
| 1 | **Bandeja de entrada (Inbox)** | Punto de arranque del día | En curso / en espera (clasificación, almacén, arribos) / pendientes por identificar | Propuesta M9 | 🔵 Nueva |

---

## Fase 1 — Referencia (wizard + detalle + listado)

| # | Pantalla | Para qué se usa | Contiene | Justificación | Estado |
|---|---|---|---|---|---|
| 2 | **Listado de Referencias** | Ver e ir a cualquier referencia existente | Tabla/índice de referencias (filtros por cliente, tráfico, estatus) | Aclarado por ti: la Referencia necesita wizard + detalle + **listado**, no solo el wizard | 🔵 Nueva — no estaba en el inventario v2 |
| 3 | **Crear Referencia (wizard, 3 pasos)** | Abrir una referencia rápido, a partir del disparador | Paso 1: Documentos (opcional); Paso 2: Documentos Aceptados; Paso 3: Datos Básicos. Ya no obliga documentos comerciales | Decidido — M9. D1 tiene 4 pasos hoy | 🟡 Modificar |

---

## Fase 2 — Detalle de Referencia (las 3 funciones)

Recordatorio de las 3 funciones que vive aquí: **(1) Expediente digital aduanero, (2) Datos Glosados para Operación (DGO), (3) punto de partida para Operaciones/Movimientos/Pedimentos/Timeline**.

| # | Pantalla (tab) | Para qué se usa | Contiene | Justificación | Estado |
|---|---|---|---|---|---|
| 4 | **Resumen** | Vista general | Datos del cliente, transporte, estatus | M8 | 🟢 Ya existe |
| 5 | **Expediente aduanero** (Función 1) | Checklist de documentos + su validación legal | Grid de documentos, estado (Pendiente/Subido/Verificado/Rechazado), estado de glosa verde/rojo, documento "primario" + versiones. **Cada carga dispara glosado con Zeus** contra: (a) validez legal propia, (b) consistencia con otros documentos/DGO, (c) consistencia con el **Previo** | Propuesta M9 + decisión de re-glosado completo con German | 🟡 Modificar (brecha grande) |
| 6 | **Datos Glosados (DGO)** (Función 2) | Fuente única de verdad, nivel partida. **1 DGO = 1 pedimento** — no son pasos distintos, es la misma unidad. **Absorbe la captura de facturas y partidas** (antes en Mercancía) **y la configuración de Identificadores e Incrementables/Decrementables** (antes tabs propios de la referencia) | Jerarquía: Datos Globales (heredados de la referencia, o propios) → Factura(s) → Partida(s). Una referencia trae **1 DGO por default**; se separa en más si hay claves de pedimento distintas. Ya incluye campos de pedimento (régimen, clave, aduana, destino, observaciones, identificadores, incrementables/decrementables) — no se capturan aparte. **Identificadores e Incrementables/Decrementables se configuran por DGO**: se puede seleccionar varios DGOs y ponerles los mismos, o dejarlos con valores distintos por DGO (y para incrementables, prorratear un monto entre varios DGOs o asignar cantidades distintas a cada uno). Comparación JSON capturado vs factura real, discrepancias en rojo, **indicador de consistencia contra el Previo**, campos de Manifestación de Valor Electrónica | Propuesta M9/M8, modelo definitivo confirmado por ti — Identificadores/Incrementables dejan de ser tabs de referencia y pasan a ser campos del DGO | 🔵 Nueva — prioridad alta |
| 7 | ~~Mercancía~~ | ✅ **Resuelto — se fusiona con DGO (#6).** Ya no es un tab separado; su contenido (facturas, partidas, edición inline, verificación de totales) vive dentro de Datos Glosados. | — | D1 tenía "Mercancía" como tab propio; decisión: se integra a DGO | ✅ Fusionada — ver #6 |
| 8 | **Movimientos** (tab contenedor) | Ver el inventario/tracking físico-logístico de la referencia (plano separado del documental-legal); el contenido varía según el tráfico | Ver catálogo completo en **Fase 3** — este tab debe adaptar qué muestra según tráfico. Se vincula a los DGO de forma **flexible, no 1 a 1** (un DGO puede cubrir varios movimientos, o viceversa) — el tab debería dejar ver ese vínculo, no asumirlo fijo | M0, M0b, y la nueva taxonomía de German; relación flexible con DGO confirmada por ti | 🟡 **Rediseño de fondo** — D1 lo trata igual para todos los tráficos y asume vínculo fijo con la operación, ya no aplica |
| 9 | ~~Incrementables y Decrementables~~ | ✅ **Resuelto — deja de ser tab de la referencia.** Pasa a ser campo configurable **por DGO** (ver #6): selección de uno o varios DGOs, mismo monto o prorrateado, o cantidades distintas por DGO. | — | Aclarado por ti | ✅ Absorbida — ver #6 |
| 10 | ~~Identificadores~~ | ✅ **Resuelto — deja de ser tab de la referencia.** Pasa a ser campo configurable **por DGO** (ver #6): se pueden seleccionar varios DGOs y ponerles los mismos identificadores, o dejarlos distintos por DGO. | — | Aclarado por ti | ✅ Absorbida — ver #6 |
| 11 | **Operaciones** (Función 3) | Ver **todas las operaciones que agrupan DGO/pedimentos vinculados a esta referencia** — no solo las creadas exclusivamente desde ella. La Operación **agrupa, no crea** DGO/pedimentos; puede agrupar varias referencias, y una referencia puede tener DGO/pedimentos repartidos en varias operaciones | Tabla folio/status/pedimento/valor; acciones Inicio de Operación/Editar/Proforma/Fondos | D1 (tabla base) + modelo definitivo de Operación como agrupador (no creador) de DGO/pedimentos | 🟡 **Ajustar consulta** — debe traer operaciones vinculadas vía DGO, aunque no hayan nacido solo de esta referencia |
| 12 | **Recinto** (solo lectura) — 💡 *propuesta mía de nombre unificado; ver nota de naming al pie* | Que el ejecutivo sepa, sin tener que preguntarle a almacén, **dónde está físicamente la mercancía y en qué etapa logística** (recibida, en custodia, lista para salir) — se adapta según el tráfico: **Almacén** (terrestre) / **Almacén Fiscalizado** (aéreo) / **Terminal** (marítimo) | Estatus de recepción/custodia, ubicación, disponibilidad para despacho | Fuera de alcance como rol (almacén) — el ejecutivo solo consulta, no gestiona el recinto | 🟡 **Renombrar + adaptar por tráfico** |
| 12b | **Previo** | Ver el resultado de la verificación previa de la mercancía (inspección física antes del despacho) y si es consistente con el Expediente/DGO; **el ejecutivo también puede solicitar que se haga** (ver #20b) | Estatus del previo (pendiente / solicitado / en mesa de previo / completado), resultado de la validación de consistencia que hace Zeus contra documentos y DGO | Previo lo ejecuta almacén/trámite (fuera de alcance), pero el ejecutivo consulta el resultado **y puede solicitarlo** — Documento de Entendimiento, entrada "Previo" | 🔵 Nueva |
| 13 | **Línea de tiempo** (Función 3) | Trazabilidad completa | Timeline cross-entidad: entradas, salidas, glosa, pagos | M0 | 🟢 Ya existe |

> **Recinto vs. Previo — por qué son dos tabs distintos, no uno:** **Recinto** responde "¿dónde está físicamente la mercancía y en qué estatus logístico general?" (llegó, está en almacén/terminal, salió). **Previo** responde una pregunta distinta y más específica: "¿ya se verificó la mercancía contra lo declarado, y ese resultado cuadra con el Expediente/DGO?" — es la etapa de inspección y su resultado de consistencia, no la ubicación física. Un embarque puede estar "en Recinto" sin que su Previo esté completo todavía, y viceversa (previo completado, pero la mercancía sigue en el recinto esperando despacho).

> 💡 **Nota de naming (#12):** propongo "Recinto" como nombre unificado del tab porque ya usamos el término legal "recinto fiscalizado" en el documento (M7, identificador SR 210) — es el concepto paraguas que cubre almacén, almacén fiscalizado y terminal por igual. Es propuesta mía, no de los meets — dime si prefieres otro nombre o dejarlo adaptado por tráfico sin unificar.

---

## Fase 3 — Catálogo de Movimientos (material de referencia para diseñar el tab #8, no son 26 pantallas nuevas)

El tab **Movimientos** (#8) debe adaptar su contenido según el tráfico de la referencia. Esta tabla es la base para ese diseño — varios renglones comparten UI (ver Fase 3b para las pantallas/modales concretos que sí hay que diseñar).

> **Recordatorio del modelo:** Movimiento vive en el plano **físico-logístico** (almacén); es distinto del DGO/pedimento (plano **documental-legal**). Se vinculan de forma **flexible, no 1 a 1** — un DGO puede cubrir varios movimientos, o viceversa. Por eso ningún renglón de esta tabla "crea" una operación ni un pedimento por sí solo; solo describen el tracking físico.

### Estándar (los 3 tráficos)

| Movimiento | Qué es | Quién actúa | Estado |
|---|---|---|---|
| **Subdivisión** | Segmenta mercancía de una factura DGO/entrada original (remanentes, separaciones, ajustes de régimen) | Ejecutivo | 🟢 Ya existe (D1) |
| **Previo** | Inspección de la carga antes del despacho | Solo visibilidad (almacén ejecuta; ejecutivo consulta consistencia vía DGO) | 🔵 Nueva vista de consulta |

### Terrestre

| Movimiento | Import/Export | Quién actúa | Estado |
|---|---|---|---|
| **Entrada** | Importación | Ejecutivo (avisa a almacén) | 🟢 Ya existe |
| **Salida** | Importación | Ejecutivo (vía "Solicitar Salida" desde Operación, #22) | 🟡 Ajustar (M0b) |
| *(sin definir)* | Exportación | — | ⚠️ **Falta sesión con ejecutivos de terrestre** |

### Aéreo

| Movimiento | Import/Export | Quién actúa | Estado |
|---|---|---|---|
| **Manifiesto de Carga** | Importación | Ejecutivo (recibe y da seguimiento) | 🔵 Nueva |
| **Revalidación Aérea** | Importación | Ejecutivo (documental + pago ante aerolínea) | 🔵 Nueva |
| **Asignación de Transporte** | Importación | Ejecutivo asigna; seguimiento es de trámite y despacho (fuera de alcance, solo visibilidad) | 🔵 Nueva |
| *(sin definir)* | Exportación | — | ⚠️ **Falta sesión con ejecutivos de aéreo** |

### Marítimo

| Movimiento | Import/Export | Quién actúa | Estado |
|---|---|---|---|
| **Revalidación Marítima** | Importación | Ejecutivo (documental + pago ante naviera) | 🔵 Nueva |
| **Asignación de Transporte** | Importación | Ejecutivo asigna; seguimiento es de trámite y despacho (fuera de alcance, solo visibilidad) — mismo componente que aéreo | 🔵 Nueva (reutilizable) |
| **Retorno de Vacío** | Importación | ✅ **Ejecutivo** — solicita transporte, da seguimiento a fumigación/lavado, solicita fondos para el pago | 🔵 Nueva |
| **Recuperación de Garantía** | Importación | ✅ **Ejecutivo** — proceso documental, solicita el depósito/devolución | 🔵 Nueva |
| **Toma de Vacío** | Exportación | ✅ **Ejecutivo** — solicita transporte para el contenedor vacío | 🔵 Nueva |
| **Carga de Mercancía** | Exportación | ✅ **Ejecutivo** — da seguimiento, genera citas si aplica | 🔵 Nueva |
| **Ingreso de Mercancía** | Exportación | ✅ **Ejecutivo** — genera cita (ej. PIS en Manzanillo) | 🔵 Nueva |
| **Modulación, Liberación y Zarpe** | Exportación | Solo visibilidad (modulación ya vive en Despacho, #25) | Solo visibilidad |

---

## Fase 3b — Modales/pantallas concretas de acciones sobre Movimientos

| # | Pantalla | Para qué se usa | Contiene | Justificación | Estado |
|---|---|---|---|---|---|
| 14 | **Crear Movimiento de Entrada** (terrestre) | Avisar a almacén qué llega y cuándo | Facturas/BL, transporte, guías, bultos, contenedores | M0 | 🟢 Ya existe |
| 15 | **Crear Subdivisión** | Dividir inventario de un movimiento | Original/Subdividido/Disponible/A Asignar; no se puede subdividir el 100% | M0 | 🟢 Ya existe |
| 16 | **Ver OS&D** (Sobrante/Faltante/Daño) — ⚠️ *corregido: no lo reporta el ejecutivo* | Consultar la discrepancia detectada durante el **Previo** (tipo, cantidad, evidencia) y a qué movimiento afecta, para poder decidir qué acción tomar | Tipo (Sobrante/Faltante/Daño), cantidad, responsable, evidencias — **solo lectura**; el reporte lo genera almacén/Previo, no el ejecutivo | M2 (SIGAC 2: OSND) — aclarado por ti: el ejecutivo consulta, no reporta | 🟡 **Cambia de rol** — D1 lo trata como formulario de captura del ejecutivo (`OSDReportForm`); pasa a ser vista de consulta |
| 17 | **Transferir ítems entre Movimientos** | Mover mercancía entre movimientos | Selector de destino, cantidad | D1 — acción sobre un movimiento, no tipo propio | 🟢 Ya existe |
| 18 | **Revalidación (Aérea/Marítima)** | Proceso documental + pago ante aerolínea/naviera | Datos de la revalidación, comprobante de pago | Nuevo, taxonomía de German | 🔵 Nueva |
| 19 | **Asignación de Transporte** (aéreo/marítimo) | El ejecutivo asigna la línea de transporte | Selección de transportista, datos de contacto/ruta; estatus semáforo verde/rojo (llega, carga correcta, sale) **actualizado por trámite y despacho, solo lectura para el ejecutivo** | Nuevo, taxonomía de German — probablemente un solo componente para ambos tráficos; rol de trámite y despacho confirmado fuera de alcance | 🔵 Nueva |
| 20 | **Tablero de Manifiestos de Carga** (aéreo) | Planeación diaria de manifiestos recibidos | Manifiestos del día, etapas (llega → ETA → confirma llegada → descarga → desconsolida → asigna a previo) | Nuevo, taxonomía de German | 🔵 Nueva |
| 20b | **Solicitar Previo** | El ejecutivo pide que se haga la verificación previa de la mercancía | Se dirige a **almacén** (terrestre) o a **trámite y despacho** (marítimo/aéreo); referencia/movimiento afectado, notas | Aclarado por ti: el ejecutivo sí puede solicitar el previo, aunque no lo ejecute | 🔵 Nueva |
| 20c | **Generar Cita** (marítimo/aéreo) | El ejecutivo agenda citas necesarias para el proceso (ej. PIS en Manzanillo para ingreso de mercancía) | Tipo de cita, puerto/recinto, fecha/hora, referencia/movimiento asociado | Aclarado por ti: el ejecutivo es responsable de generar citas dentro de Retorno de Vacío/Carga/Ingreso de Mercancía | 🔵 Nueva |

---

## Fase 4 — Operación

| # | Pantalla | Para qué se usa | Contiene | Justificación | Estado |
|---|---|---|---|---|---|
| 21 | **Crear/Editar Operación (wizard)** | **Agrupar** uno o más DGO/pedimentos ya existentes bajo un mismo expediente (no los crea ni transforma) | **Step 0 pasa a ser "Seleccionar DGO(s)"** (ya no "Seleccionar Movimientos") — la operación se arma directamente a partir de los DGO elegidos, no de los movimientos. El número de DGO seleccionados determina el número de pedimentos. Como el DGO ya trae los campos de pedimento, **Step 3 (Config Fiscal) y Step 4 (Encabezado del Pedimento) llegarían prellenados** — el wizard se simplifica a confirmar/ajustar, no capturar desde cero | D1 (base del wizard, Step 0 hoy selecciona Movimientos) + modelo definitivo: la Operación se crea a partir de los DGO | 🟡 **Cambiar el Step 0** (de Movimientos a DGOs) + ajustar fuente de datos + renombres pendientes |
| 22 | **Solicitar Salida (mini-modal)** | Avisar a almacén que algo va a salir | Sello, contenedor, tipo de contenedor — resto heredado | Decidido en M0b | 🔵 Nueva |
| 23 | **Solicitud de Fondos (rediseño)** | Solicitar fondos para pagar el pedimento **y otros procesos** (ej. garantía de contenedor, fumigación, lavado — ver movimientos marítimos en Fase 3) | 3 categorías: gastos comprobados, impuestos, honorarios | M10, platicado con Miguel — D1 es solo un botón vacío; alcance ampliado con los movimientos marítimos confirmados | 🔵 Nueva — brecha grande |
| 24 | **Ver Proforma / Pedimento (Anexo 22)** | Revisar el pedimento | Resumen + documento Anexo 22 | D1 | 🟢 Ya existe |

---

## Fase 5 — Despacho

| # | Pantalla | Para qué se usa | Contiene | Justificación | Estado |
|---|---|---|---|---|---|
| 25 | **Inicio de Operación / Despacho (stepper)** | Disparar y monitorear el despacho | EDOCUMENTS → COVE → MANIFESTACION → VALIDACION → PAGO → DODA → MODULACION (**GLOSA se retira**, la cubre el DGO) | D1 — decisión de dejar solo DGO; falta validar el resto del stepper (pendiente) | 🟡 Modificar |

---

## Fase 6 — Portal de Documentos del Cliente (confirmada, integrada al Expediente)

| # | Pantalla | Para qué se usa | Contiene | Justificación | Estado |
|---|---|---|---|---|---|
| 26 | **Portal de Documentos del Cliente** ✅ *confirmado, no solo propuesta* | Que el cliente suba documentos directamente, sin pasar por el ejecutivo, y reciba retroalimentación sobre su estatus | Los documentos subidos caen **directo al Expediente Aduanero** (#5), marcados como "subido por el cliente". El resultado del glosado digital (Zeus) se refleja **en ambos lados**: en el Expediente Aduanero (para el ejecutivo) y en este portal (para que el cliente vea el feedback de lo que subió — válido/con error/pendiente) | Propuesta M8, mecánica confirmada por ti | 🔵 Nueva |

---

## Notas sobre lo que NO incluí (y por qué)

- **Previo, la ejecución física** — sigue fuera de alcance (la hace almacén/trámite y despacho); el ejecutivo sí participa **consultando el resultado** (#12b) y **solicitándolo** (#20b), pero no lo ejecuta.
- **Configuración de MOMP** — la pantalla de configuración en sí (dar de alta/editar las reglas de un cliente) queda fuera de este inventario, parece config de cliente/onboarding, no parte del flujo operativo. Pero sus reglas **sí se consumen** (solo lectura) dentro del flujo del ejecutivo — sobre todo en el **Expediente aduanero (#5)**, cuyo checklist depende del perfil de cliente (proforma, cartas firmadas, etc., M1/M2), y probablemente en el **DGO (#6)** para reglas específicas de captura. No es una pantalla nueva, es una dependencia de datos a tener presente al diseñar esas dos.
- **Pasos de solo lectura del despacho sin cambios** (COVE, MANIFESTACION, PAGO, DODA) — agrupados dentro de #25, no como pantallas propias.

## ⚠️ Preguntas de alcance que quedaron abiertas con este rediseño

1. ✅ **Resuelto — "Trámite y despacho" como rol:** es otro rol fuera de alcance, como almacén/clasificación/tesorería, aplica principalmente en marítimo y aéreo. El ejecutivo asigna transporte y solicita previo, pero el seguimiento/ejecución en sí es de trámite y despacho — solo visibilidad para el ejecutivo.
2. ✅ **Resuelto — Retorno de Vacío, Recuperación de Garantía, Toma de Vacío, Carga de Mercancía, Ingreso de Mercancía** (marítimo): el ejecutivo **sí es responsable** de estos procesos — genera citas, solicita transporte, solicita fondos, etc. No son solo tracking/visibilidad. Ver catálogo actualizado en Fase 3 y las nuevas pantallas #20b/#20c.
3. **Exportación Terrestre y Exportación Aéreo** — sin definir; toca agendar esa sesión antes de poder completar la Fase 3 para esos dos tráficos.
4. ✅ **Resuelto — Regla M0b aplicada a aéreo/marítimo:** con el modelo definitivo, la Operación se crea seleccionando **DGOs directamente** (#21, Step 0 = "Seleccionar DGO(s)"), no movimientos — así que la pregunta de "¿qué movimiento equivale a Entrada en aéreo/marítimo?" ya no aplica para este punto. El vínculo Movimiento↔DGO (flexible, no 1 a 1) queda como un dato de trazabilidad dentro del DGO/tab Movimientos, no como mecanismo para crear la Operación.
5. ✅ **Resuelto — Variantes de relación Referencia↔Operación:** no es 1 a 1. Una operación agrupa uno o más DGO/pedimentos, que pueden venir de varias referencias (consolidado); y una referencia puede tener sus DGO/pedimentos repartidos en varias operaciones distintas. El tab Operaciones (#11) y el wizard de Crear Operación (#21) ya quedaron ajustados para reflejar esto.

## Resumen numérico

- **29 pantallas numeradas** (28 + la nueva 20c), de las cuales **28 son distintas** — la #7 (Mercancía) quedó fusionada dentro de la #6 (DGO) — + 1 tabla de referencia (catálogo de Movimientos, Fase 3).
- 🟢 Ya existen: **6** (baja 1 más — Ver OS&D cambia de rol, de captura del ejecutivo a consulta)
- 🟡 Existen con cambios pendientes: **7**
- 🔵 Nuevas: **13**
- ✅ Fusionadas/absorbidas: **3** (Mercancía, Incrementables y Decrementables, Identificadores — todas dentro de DGO)
- ⚠️ Con pregunta de alcance sin resolver antes de diseñar: **1 sin definir por falta de sesión (#3)**, 4 resueltas (#1, #2, #4, #5)

## Siguiente paso

De las 5 preguntas de alcance originales, solo queda pendiente la **#3** (exportación terrestre/aéreo), que necesita su propia sesión con esos ejecutivos — el resto ya quedó resuelto. Las nuevas 🔵 de mayor riesgo/novedad para empezar a diseñar siguen siendo, en orden: **#6 Datos Glosados (DGO)**, **#22 Solicitar Salida**, **#23 Solicitud de Fondos**, y luego las de Fase 3b (movimientos especializados de aéreo/marítimo).
