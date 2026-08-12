# Plan: Nuevos estatus de guía aérea (En camino, Revalidado, Peso certificado, Digitalizada)

## Contexto

Origen: ticket del panel de tickets — línea de `Tareas.md`:
`- [ ] Definir nuevos estatus de guía aérea: Revalidado, Peso certificado, Digitalizada ([[2026-07-24 - Prueba operación real desde ticket a facturación]]) — SCRUM-504`

La reunión de origen ([[2026-07-24 - Prueba operación real desde ticket a facturación]], tarea #15) solo nombra los tres estatus sin definir su significado operativo ni su posición — quedó explícitamente pendiente de definir con Germán/Taico. Esta sesión de `/plan` cerró esa definición pendiente directamente con el usuario (Ángel).

Área del código (confirmada por exploración de graphify + grep, ver Contrato):
- Back (`carmi-odin-api-v2`): enum Prisma `GuiaStatus` en `prisma/schema.prisma`, modelo `Guia` (campo `status`), historial `GuiaStatusHistory`. El modelo `Guia` es hoy exclusivo del "Flujo Querétaro (aéreo)" — no existe modelo separado de guía marítima/terrestre, así que no hace falta acotar el enum por tipo de tráfico.
- Front (`carmi-digital`): `GUIA_STATUS_OPTIONS` en `app/(customerPortal)/guias/components/guia-status-schema.ts`, tipo espejo `GuiaStatus` en `lib/api/modules/guias.ts`, consumido por `GuiaStatusModal.tsx` y `BulkGuiaStatusModal.tsx`.

El estatus es hoy un campo libre: `GuiasService.updateStatus()` acepta cualquier valor del enum sin validar transición desde el estatus anterior, y cada cambio queda en `GuiaStatusHistory`. No hay lógica automática ni side effects (notificaciones, glosa digital, bloqueo de edición) atados al cambio de estatus, salvo la auto-transición ya existente a `REFERENCIA_CREADA` al vincular la guía a una referencia/DGO.

## Decisiones tomadas

- **Los 4 estatus son parte del mismo select lineal existente**, no flags independientes. Se agregan como valores más del enum `GuiaStatus` actual. *Por qué:* el usuario confirmó explícitamente mantenerlo como "estatus lineal en el select que ya existe" en vez de convertirlo en múltiples flags booleanos.
- **Se agrega un cuarto estatus no mencionado en el acta original: "En camino"**. *Por qué:* pedido directo del usuario durante la entrevista, para cubrir el tramo antes de que la guía sea recibida.
- **Secuencia final completa** (valor de enum → label mostrado):
  1. `EN_CAMINO` → "En camino"
  2. `RECIBIDA` → "Recibida" *(existente)*
  3. `EN_REVISION` → "En revisión" *(existente)*
  4. `REGIMEN_ASIGNADO` → "Régimen asignado" *(existente)*
  5. `REVALIDADO` → "Revalidado"
  6. `PESO_CERTIFICADO` → "Peso certificado"
  7. `DIGITALIZADA` → "Digitalizada"
  8. `REFERENCIA_CREADA` → "Referencia creada" *(existente)*

  *Por qué:* el usuario ubicó "En camino" antes de "Recibida" (tramo previo a la recepción física), y los tres estatus del acta ("Revalidado", "Peso certificado", "Digitalizada") juntos e inmediatamente antes de "Referencia creada" — es decir, como el tramo final de preparación documental antes de generar la referencia.
- **Los estatus se asignan manualmente**, igual que los existentes — el usuario los selecciona a mano desde el select/modal de estatus. *Por qué:* confirmado explícitamente; no hay automatización (ej. Zeus disparando "Digitalizada" solo) en el alcance de este plan.
- **Sin significado operativo definido más allá del nombre** ("Revalidado", "Peso certificado", "Digitalizada" quedan sin una definición formal de qué condición de negocio representan). *Por qué:* decisión explícita del usuario de no cerrar esa definición ahora; ver Riesgos.
- **Nombres de valor de enum**: `EN_CAMINO`, `REVALIDADO`, `PESO_CERTIFICADO`, `DIGITALIZADA` — siguiendo el estilo `SCREAMING_SNAKE_CASE` ya usado por los valores existentes. Confirmado por el usuario.
- **Alcance puramente aditivo al catálogo/select**: sin lógica automática nueva, sin cambios a `GuiaStatusHistory`, sin validación de transición (se mantiene como campo libre, igual que hoy). Confirmado explícitamente por el usuario.

## Fuera de alcance

- Definir qué condición de negocio dispara cada uno de los tres estatus del acta (ej. qué significa exactamente "Peso certificado" u origina "Digitalizada" desde Zeus) — queda sin definir, ver Riesgos.
- Cualquier automatización de cambio de estatus (ej. que Zeus marque "Digitalizada" automáticamente al digitalizar el documento).
- Validación de máquina de estados / transiciones permitidas entre estatus — el campo sigue siendo libre.
- Diferenciar estatus por tipo de tráfico (marítimo/terrestre) — no aplica porque el modelo `Guia` hoy solo cubre aéreo.
- Cualquier cambio a guía master vs. house (no hay diferencia de estatus entre ambas, no se toca).

## Contrato

Este requerimiento cruza capas (back define el enum, front lo consume). Contrato de solo lectura — quien implemente no lo cambia; si algo no calza, se detiene y reporta.

**Back — enum Prisma** (`carmi-odin-api-v2/prisma/schema.prisma`, enum `GuiaStatus`, hoy L7171-7178):

Agregar, preservando el orden existente y respetando la secuencia completa de Decisiones tomadas:

```prisma
enum GuiaStatus {
  EN_CAMINO
  RECIBIDA
  EN_REVISION
  REGIMEN_ASIGNADO
  REVALIDADO
  PESO_CERTIFICADO
  DIGITALIZADA
  REFERENCIA_CREADA
}
```

Migración obligatoria por CLI: `npx prisma migrate dev --name add-en-camino-revalidado-peso-certificado-digitalizada-to-guia-status` (regla global de CLAUDE.md: nunca escribir el SQL de migración a mano; si el entorno no permite correr la CLI, detente y pregunta).

**Front — catálogo de labels** (`carmi-digital/app/(customerPortal)/guias/components/guia-status-schema.ts`, `GUIA_STATUS_OPTIONS`):

Agregar las 4 entradas nuevas con value/label exactos, en la posición correspondiente a la secuencia de Decisiones tomadas:

```ts
{ value: "EN_CAMINO", label: "En camino" }
{ value: "REVALIDADO", label: "Revalidado" }
{ value: "PESO_CERTIFICADO", label: "Peso certificado" }
{ value: "DIGITALIZADA", label: "Digitalizada" }
```

**Front — tipo espejo** (`carmi-digital/lib/api/modules/guias.ts`, tipo `GuiaStatus`, L3): agregar los mismos 4 valores como union members, en el mismo orden.

**Front — schema de validación del formulario** (`carmi-digital/app/(customerPortal)/guias/components/guia-status-schema.ts`, `guiaStatusFormSchema`, L12): el `z.enum([...])` está hardcodeado por separado de `GUIA_STATUS_OPTIONS` — agregar ahí también los mismos 4 valores nuevos, en el mismo orden, o el formulario rechazará los estatus nuevos aunque aparezcan en el select. *Hallazgo de la ronda de dudas post-plan, no visto en la redacción original.*

**Front — labels y color de badge** (`carmi-digital/app/(customerPortal)/guias/page.tsx`, `STATUS_LABELS` y `STATUS_VARIANTS`, L49-61): son dos `Record<GuiaStatus, ...>` exhaustivos sin `default` — al ampliar el union type, TypeScript exige las 4 claves nuevas o no compila. Agregar:

```ts
// STATUS_LABELS
EN_CAMINO: 'En camino',
REVALIDADO: 'Revalidado',
PESO_CERTIFICADO: 'Peso certificado',
DIGITALIZADA: 'Digitalizada',

// STATUS_VARIANTS
EN_CAMINO: 'secondary',
REVALIDADO: 'default',
PESO_CERTIFICADO: 'default',
DIGITALIZADA: 'warning',
```

*Por qué esas variantes:* el usuario confirmó mantener el patrón visual existente ("más avanzado en el flujo = más cálido"), sin implicar significado de negocio no validado (ver Riesgos): `EN_CAMINO` arranca en `secondary` igual que `RECIBIDA` hoy; `DIGITALIZADA`, al ser el último paso antes de `REFERENCIA_CREADA`, queda en `warning` igual que `REGIMEN_ASIGNADO` hoy. *Hallazgo de la ronda de dudas post-plan; el plan original declaraba el alcance "puramente aditivo al catálogo/select" sin listar este archivo — se amplía el contrato para que el build no se rompa.*

Este archivo (`page.tsx`) se agrega al alcance de este plan pese a no estar en la redacción original, porque es una consecuencia mecánica y obligatoria de ampliar el union type `GuiaStatus`, no una funcionalidad nueva.

## Pasos

- [x] **Back**: agregar los 4 valores al enum `GuiaStatus` en `prisma/schema.prisma` en la posición exacta del Contrato y generar la migración con `npx prisma migrate dev --name add-en-camino-revalidado-peso-certificado-digitalizada-to-guia-status`.
  - Archivos: `carmi-odin-api-v2/prisma/schema.prisma` (enum `GuiaStatus`), migración nueva `prisma/migrations/20260811194838_add_en_camino_revalidado_peso_certificado_digitalizada_to_guia_status/migration.sql`.
  - `migrate dev` falló primero por entorno no interactivo + drift severo de la DB local (`carmi_dev` muy desincronizada del historial de migraciones); el usuario resolvió el drift y se reintentó con éxito. La migración generada es limpia: solo 4 `ALTER TYPE ... ADD VALUE`, sin ruido de otras tablas.
  - Prisma Client regenerado (`node_modules/.pnpm/@prisma+client@5.22.0`). Rama de trabajo `feat/nuevos-estatus-guia-aerea` creada en ambos repos (back y front) antes de empezar.
  - Pendiente: las 3 tareas de front (catálogo/labels, tipo espejo, `guiaStatusFormSchema`; `STATUS_LABELS`/`STATUS_VARIANTS` en `page.tsx`; verificación visual de los modales).
- [x] **Front**: agregar las 4 entradas a `GUIA_STATUS_OPTIONS` (`guia-status-schema.ts`), al `z.enum` de `guiaStatusFormSchema` en el mismo archivo, y al tipo `GuiaStatus` (`lib/api/modules/guias.ts`), en el orden del Contrato.
  - Archivos: `carmi-digital/app/(customerPortal)/guias/components/guia-status-schema.ts` (`GUIA_STATUS_OPTIONS` y `guiaStatusFormSchema.status`), `carmi-digital/lib/api/modules/guias.ts` (tipo `GuiaStatus`). Los 4 valores nuevos insertados en el orden exacto del Contrato.
  - No se corrió typecheck todavía: `page.tsx` (`STATUS_LABELS`/`STATUS_VARIANTS`) sigue sin las 4 claves nuevas, así que el build fallará hasta la tarea 3 — es el estado esperado, ya anticipado en el plan.
  - Pendiente: tarea 3 (`STATUS_LABELS`/`STATUS_VARIANTS` en `page.tsx`) y tarea 4 (verificación visual de los modales).
- [x] **Front**: agregar las 4 claves nuevas a `STATUS_LABELS` y `STATUS_VARIANTS` en `page.tsx` con los valores exactos del Contrato, para que compile con el union type ampliado.
  - Archivo: `carmi-digital/app/(customerPortal)/guias/page.tsx` (`STATUS_LABELS`, `STATUS_VARIANTS`). Las 4 claves nuevas agregadas con los valores exactos del Contrato (`EN_CAMINO`→secondary, `REVALIDADO`→default, `PESO_CERTIFICADO`→default, `DIGITALIZADA`→warning).
  - `npx tsc --noEmit -p tsconfig.json` corre sin errores: los dos `Record<GuiaStatus, ...>` exhaustivos compilan con el union type ampliado.
  - Pendiente: tarea 4 (verificación visual en navegador de `GuiaStatusModal.tsx` y `BulkGuiaStatusModal.tsx` vía subagente `verificador-front`).
- [x] **Front**: confirmar visualmente que `GuiaStatusModal.tsx` y `BulkGuiaStatusModal.tsx` muestran los 4 valores nuevos en el select sin cambios de código adicionales (ambos consumen `GUIA_STATUS_OPTIONS`) — si alguno tiene una whitelist o filtro propio de estatus, reportarlo como desviación del contrato antes de continuar.
  - Verificado vía subagente `verificador-front` sobre `localhost:3100` (front) + `localhost:8090` (back), ambos levantados en background para esta verificación — no quedan corriendo por defecto entre sesiones, hay que relanzarlos si se retoma este flujo. Ninguno tenía `PORT` propio en `.env`; heredaban `PORT=47821` del entorno de mi-ide-claude y colisionaban — se forzó `PORT=3100` (front) y `PORT=8090` (back, coincide con la API URL que el front espera).
  - Ambos modales (`GuiaStatusModal` y `BulkGuiaStatusModal`, sobre guías `okokww`/`okok` del Flujo Querétaro) mostraron las 8 opciones en el orden exacto del Contrato, sin whitelist propia divergente de `GUIA_STATUS_OPTIONS`.
  - Guardado real de un cambio a "Peso certificado" confirmado: el campo "Comentario" es requerido (validación existente, no relacionada a este plan) y el badge en `page.tsx` reflejó el label/color correctos tras guardar. Cero errores de consola nuevos; solo warnings preexistentes (logo sin width/height, `DialogContent` sin `Description` en Radix).
  - Evidencia en `carmi-digital/../.playwright-mcp/` (`estado-actual.png`, `estado-post-save.png`, `final-listado.png`).

## Riesgos y side effects a vigilar

- **Los tres estatus del acta no tienen definición de negocio formal.** Si más adelante Taico/Germán aclaran que alguno de estos debería ser automático (ej. "Digitalizada" disparado por Zeus) o que representa una condición verificable, este plan no lo cubre — sería un requerimiento nuevo, no una corrección de este.
- El comentario existente en `schema.prisma` (L7168-7170) ya advierte que el set de estatus es "hipótesis a validar" — este cambio no resuelve esa incertidumbre de fondo, solo añade 4 valores más sobre la misma base sin validar.
- Revisar si `BulkGuiaStatusModal.tsx` u otro consumidor de `GuiaStatus` tiene lógica propia (ej. un `switch` que no cubra los nuevos valores con un `default` seguro) que rompa silenciosamente con los 4 valores nuevos.
- **`linkGuiasToReference` (back) fuerza `status: REFERENCIA_CREADA` sin importar el estatus previo al vincular una guía a una referencia** — ya existe hoy para los estatus actuales; con más estatus intermedios en el select, ese salto puede pisar un estatus recién asignado (ej. `PESO_CERTIFICADO`). Usuario confirmó explícitamente mantener este comportamiento sin cambios: *"Está bien el comportamiento de referencia creada cuando se crea"* — queda fuera de alcance, no es una regresión de este plan.

## Criterios de verificación

- `npx prisma migrate dev` corre sin error y el nuevo enum queda reflejado en el cliente Prisma generado.
- Typecheck del front pasa con los 4 valores nuevos agregados al tipo `GuiaStatus`, al `z.enum` de `guiaStatusFormSchema`, y a `STATUS_LABELS`/`STATUS_VARIANTS` en `page.tsx` (sin este último paso el build falla por los `Record` exhaustivos).
- Flujo manual en navegador (vía subagente `verificador-front`, nunca en este hilo): abrir el modal de cambio de estatus de una guía aérea existente y confirmar que el select muestra la secuencia completa en el orden del Contrato, incluyendo los 4 valores nuevos; repetir en `BulkGuiaStatusModal` con selección múltiple de guías.
- Guardar un cambio de estatus a cada uno de los 4 valores nuevos y confirmar que se refleja en `GuiaStatusHistory` (estatus anterior/nuevo correctos), que el badge en `page.tsx` muestra el label y color correctos, y que el submit del formulario no es rechazado por `guiaStatusFormSchema`.
