# Sesión 2026-07-22 — Refinamiento de tareas Querétaro

Relacionado: [[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]],
[[2026-07-16 - Flujo diario de manifiestos, régimen y candados (Querétaro)]],
[[2026-07-20 - Recap visita Querétaro]], [[Discutir antes de persistir tareas]]

## Qué se trabajó

- Se retomaron los 3 pendientes que dejó abiertos la sesión del 21-jul: la
  pregunta de UX (tablero de Guías vs. "Por identificar" del Inbox) **no se
  retomó**; las 7 tareas propuestas se revisaron una por una y se
  confirmaron/reescribieron; y se decidió la estructura de plan formal
  (ver Decisiones).
- Validación técnica profunda contra el schema y código real
  (`carmi-odin-api-v2`, rama `feat/movimientos-dinamicos-por-trafico`, WIP
  sin commitear) de las dos tareas de modelo de datos propuestas en la
  sesión anterior — se corrigieron ambas al leer el código real en vez de
  confiar en la nota de sesión:
  - `UnidentifiedWaybill` es genérica (BOL+AWB+OTHER, no exclusiva de
    aéreo) — se descartó fusionarla con `AirManifest`.
  - La FK del puente vive en `UnidentifiedWaybill.linkedAirManifestId`
    (no al revés), porque `AirManifest.referenceId` es `NOT NULL` hoy.
  - El vínculo `AirManifest↔Invoice` debe ser M:N directo
    (`InvoiceAirManifestLink`, espejeando `InvoiceShipmentLink` ya
    existente en terrestre), no `AirManifest↔Dgo`.
  - Corrección de dato: el régimen vive en `Reference.regimen`/`Dgo.regimen`
    (columna plana), no en `Invoice` como se asumía.
- El usuario compartió un Excel real de ejemplo del manifiesto de correo
  (1 Guía Master → 10 Guías House, con `Destinatario`/`Domicilio` en texto
  libre repetido entre house guides del mismo cliente, sin ningún ID de
  cliente ni régimen). Esto reveló 3 piezas de scope que no existían en
  ninguna tarea previa:
  - No hay ingesta línea-por-línea del manifiesto (ni `AirManifest` ni
    `UnidentifiedWaybill` capturan esos campos hoy).
  - No existe ningún matcher automático `Destinatario→Company`
    (`bol-reconciliation` solo matchea número de guía exacto;
    `company.service` solo tiene búsqueda manual `ILIKE`).
  - No existe ningún concepto de "Team"/equipo de trabajo que agrupe
    ejecutivos y los vincule a compañías — lo más cercano
    (`UserCompany`/`UserCompanyRole`) es asignación individual 1 a 1; el
    componente "Carmi Team" del front es mock decorativo sin datos reales.
- Se confirmó (documentación ya extensa en `Planes/`/`Arquitectura/`) que
  Zeus es el motor de extracción/glosado ya conceptualizado en el
  proyecto, pero sin integración HTTP real cableada hoy — insumo para una
  tarea nueva de UI de Expediente Aduanero.
- Se redactaron, corrigieron y confirmaron **13 tareas nuevas** para la
  sección Flujo Querétaro de `Tareas.md`, con varias rondas de ajuste por
  tarea (ver [[Discutir antes de persistir tareas]]).
- Se reestructuró `Tareas.md` completo: de lista plana cronológica a
  secciones temáticas (Bandeja de entrada, Referencia/DGO/Operaciones
  core, Detalle de Referencia y UI, Expediente Aduanero, Recinto y Citas,
  Flujo Querétaro con 5 sub-secciones). Se fusionó una tarea duplicada de
  Expediente Aduanero.

## Commits relevantes

Ninguno — sesión 100% de análisis y actualización del baúl (`Tareas.md`),
sin tocar código. Ambos repos (`carmi-digital`, `carmi-odin-api-v2`) siguen
en `feat/movimientos-dinamicos-por-trafico` con el mismo WIP sin commitear
que tenían al iniciar la sesión (137 y 143 archivos respectivamente, sin
cambios).

## Decisiones (con su porqué)

1. **`UnidentifiedWaybill` se mantiene como tabla genérica** (BOL+AWB+OTHER);
   no se fusiona con `AirManifest` — evita romper la reutilización para
   marítimo.
2. **FK del puente en `UnidentifiedWaybill.linkedAirManifestId`**, mismo
   patrón que `linkedShipmentId`/`linkedReferenceId`.
3. **Vínculo `AirManifest↔Invoice` es M:N directo**, no vía `Dgo` — evita
   doble fuente de verdad al subdividir DGOs.
4. **El régimen se captura por guía en el tablero de Guías, antes de crear
   la Referencia**, y se hereda al crearla; selección de guías con
   régimen mixto → 1 DGO por régimen distinto presente en la selección.
5. **No hay pantalla de asignación de facturas en la guía** — crear la
   Referencia desde el tablero es el único gesto necesario; el flujo
   normal de Referencia (crear facturas, subdividir DGO) cubre el resto.
6. **Se construye una entidad `Team` nueva** (grupo de ejecutivos ↔
   `Company`) en vez de reusar `UserCompany` individual — el negocio
   pidió trabajar por equipos, no asignación 1 a 1.
7. **La reconciliación Destinatario→Company ocurre en el momento de la
   ingesta**, no como paso manual posterior — para que la guía "nazca" ya
   con su cliente candidato visible (`UnidentifiedWaybill` sigue
   significando "sin Referencia vinculada", no "cliente desconocido").
8. **`Tareas.md` se reestructura por secciones temáticas** en vez de lista
   cronológica — facilita consultar por flujo (ej. "todo lo de Querétaro")
   en vez de por fecha de reunión.
9. **Iniciativa Querétaro queda como plan aparte, NO extiende el paraguas
   `refactor-flujo-ejecutivo`** — decisión explícita del usuario, cierra
   el pendiente #3 de la sesión del 21-jul.

## Aprendizajes / errores a no repetir

1. No asumir que una etiqueta corta de una nota anterior ("visibilidad de
   régimen en la vista de previo") captura la intención real — al pedir
   que se explicara, la intención real era distinta y más amplia (asignar
   régimen, no solo mostrarlo). Vale la pena pedir/dar el "por qué" antes
   de dar una tarea por entendida.
2. Validar contra el schema y código real con un subagente antes de
   proponer un diseño de datos, en vez de razonar solo desde la nota de
   sesión — se corrigieron 2 de 2 propuestas iniciales (dirección de FK,
   ubicación del campo régimen) al verificar contra código.
3. Un archivo de ejemplo real (el Excel del manifiesto) cambió el
   entendimiento de forma sustancial — reveló ausencia total de ID de
   cliente/régimen en la fuente y la relación 1 master→N house. Pedir
   datos de muestra reales antes de diseñar, no solo actas de reunión.
4. [[Discutir antes de persistir tareas]] se sostuvo bien en una sesión
   larga con muchas rondas de iteración por tarea — promovida de
   aprendizaje puntual a regla propia del baúl en esta sesión.

## Pendientes

### Para próxima sesión
1. Decidir si la tarea "Definir catálogo de tipos de movimiento por
   tráfico para aéreo y encaje de la subdivisión de guía incompleta"
   (sección Diferidas de Flujo Querétaro) se cierra o se reescribe —
   quedó señalada como posiblemente resuelta por las decisiones de esta
   sesión (aéreo no necesita Subdivisión).
2. Resolver si el tablero de "Guías" absorbe o convive con "Por
   identificar" del Inbox — sigue abierta desde la sesión del 21-jul, no
   se retomó en esta sesión.
3. Marítimo sigue sin definir (tablero equivalente, dato ancla de previo,
   escalabilidad del patrón guía↔DGO/régimen) — pendiente de una reunión
   de marítimo que no se ha documentado.
4. La integración HTTP real con Zeus sigue sin cablear — la tarea nueva de
   esta sesión (UI de Expediente Aduanero) es explícitamente solo la
   UI/sincronización con el DGO; la integración real queda como tarea
   aparte a futuro.
5. Cuando se decida pasar de tareas sueltas a plan formal, usar `/plan`
   con todo este contexto ya reunido (esta nota + la del 21-jul +
   `Tareas.md` sección Flujo Querétaro, 22 tareas).

### Notas en el baúl que ganaron contexto
- [[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]] —
  todas las tareas nuevas de esta sesión llevan wikilink hacia aquí
- `Tareas.md` — reestructurado por secciones; 13 tareas nuevas en Flujo
  Querétaro, 1 fusión en Expediente Aduanero, 1 reformulación (movimientos
  dinámicos)
- [[Discutir antes de persistir tareas]] — regla nueva

## Resumen para la próxima sesión

Se cerraron 2 de los 3 pendientes de la sesión del 21-jul (las 7 tareas
confirmadas y escritas, la decisión de plan aparte tomada); la pregunta de
UX del Inbox sigue abierta. `Tareas.md` ya no es lista plana — está por
secciones, con "Flujo Querétaro" como sección propia de 22 tareas listas
para trabajarse. El siguiente paso natural es `/plan` cuando el usuario
quiera pasar de tareas sueltas a plan de implementación formal; hasta
entonces, todo lo demás sigue en `Tareas.md`.
