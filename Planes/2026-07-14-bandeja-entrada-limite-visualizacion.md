# Plan: Límite de visualización en bandeja de entrada (máx 5 referencias/movimientos por sección)

## Contexto

Reunión **2026-07-14** — Prueba operación real desde ticket a facturación. [[2026-07-14 - Prueba operación real desde ticket a facturación]].

El Inbox (SP-17) propone 7 secciones (Alertas, Accionables, Curso automático, Esperando terceros, Por identificar, Consolidados, Pedimentos pagados sin modular). Sin límite de visualización, la bandeja se satura: el ejecutivo ve decenas de referencias aunque solo 3-5 requieren su atención inmediata.

**Decisión de la reunión:** Mostrar máximo 5 referencias/movimientos por sección (las más relevantes), con indicador "5 de 20" para que sepa que hay más. La bandeja actúa como **filtro de prioridades**, no listado exhaustivo.

> **Nota de verificación previa a la implementación (2026-07-14):** Se exploró `carmi-odin-api-v2` antes de implementar y se detectaron varias discrepancias entre este plan y el estado real del código. Quedan resueltas abajo (ver "Decisiones tomadas" actualizadas y "Fuera de alcance"). Resumen de lo encontrado:
> - Solo existen 5 de las 7 secciones en `inbox.service.ts` (Alertas, Accionables, Curso automático, Esperando terceros, Por identificar). **Consolidados y Pedimentos pagados sin modular no existen** (ni modelo, ni query, ni DTO, ni UI) → excluidas de este plan.
> - La respuesta de `GET /inbox` **ya es** `{ items, total, page, limit, totalPages }` por sección (Prisma `skip/take` en 3 de ellas, paginación en memoria en las otras 2). No hay cambio de contrato: es solo ajustar `limit` a 5. Único consumidor: `InboxDashboard.tsx`.
> - "Curso automático" no tiene `progreso` persistido (se calcula on-the-fly desde `dispatchSteps`, `inbox.service.ts:263`) ni "fecha de inicio de operación" (solo `DispatchStep.startedAt` a nivel de paso). Se usa el progreso calculado.
> - Los umbrales de severidad de Alertas en código (rojo ≤1 día, amarillo ≤3, naranja >3, ventana `ALERT_MAX_DAYS_AHEAD=7`) no coincidían con los del plan (5/10) → se actualizan en la implementación.
> - El frontend ya tiene un botón "Cargar más" en `InboxDashboard.tsx` que usa `total` → se elimina, queda solo el indicador "5 de 20".

## Decisiones tomadas

1. **Límite duro:** 5 items máximo por sección (fijo, no configurable por cliente ni usuario).

2. **Criterio de priorización:**
   - **Alertas:** por severidad (rojo ≤5 días ETA vencido → amarillo 5-10 días → verde >10 días). Requiere actualizar `ALERT_MAX_DAYS_AHEAD` y `severityFor()` en `inbox.service.ts` (hoy son 1/3/7 días, no 5/10).
   - **Accionables, Esperando terceros:** por fecha de creación descendente (más recientes primero, el ejecutivo atiende lo urgente).
   - **Curso automático:** por **progreso calculado ascendente** (reutilizar el cálculo on-the-fly ya existente `completedCount/totalSteps` en `inbox.service.ts:263`, ordenar asc = las más atrasadas primero). No existe campo de fecha de inicio a nivel de operación, solo `DispatchStep.startedAt` por paso — no se usa.
   - **Por identificar:** por fecha de llegada descendente (más recientes primero); son movimientos/guías, no referencias. Ya existe (`getUnidentified`, fusiona `UnidentifiedWaybillsService` + `Shipment` sin `referenceId`).

3. **Indicador de cantidad:** Mostrar "5 de 20" debajo del título de cada sección (o en un badge) si hay más de 5. Si hay ≤5, no mostrar contador (está todo visible).

4. **Alcance de "referencias/movimientos":** De las secciones en alcance, 4 muestran **referencias** (Alertas, Accionables, Curso automático, Esperando terceros). Solo **"Por identificar"** muestra **movimientos** (guías sin identificar: BOL, AWB, etc.) — ya implementado en `getUnidentified`.

5. **"Consolidados" y "Pedimentos pagados sin modular" quedan fuera de este plan**: no existen todavía en el backend (ni modelo, ni query, ni DTO, ni UI de SP-17). Este plan solo limita a 5 las secciones que ya existen (Alertas, Accionables, Curso automático, Esperando terceros, Por identificar). Construir esas dos secciones nuevas requiere un plan separado.

6. **Botón "Cargar más" existente:** `InboxDashboard.tsx` ya tiene un botón "Cargar más" por sección que usa `total` para paginar. Se **elimina** ese botón; queda solo el indicador "5 de 20", sin forma de ver más desde el Inbox (concuerda con "Ver todas fuera de alcance").

## Fuera de alcance

- Rediseño de las 7 secciones propuestas (ya implementadas en SP-17).
- **Construir las secciones "Consolidados" y "Pedimentos pagados sin modular"** (no existen en el backend hoy; requieren modelo, query, DTO y UI nuevos — plan separado).
- Orden por campos MOMP específicos de "Accionables" (se usa fecha, el criterio formal lo define Enrique fuera de este plan).
- Link "Ver todas", drill-down a sección expandida, o el botón "Cargar más" que ya existía en `InboxDashboard.tsx` (se elimina, no se reemplaza por otra forma de expandir).
- Configuración dinámica de límite (siempre 5, no 3 ni 10).

## Pasos

- [ ] **Backend (carmi-odin-api-v2)** — el shape `{ items, total, page, limit, totalPages }` ya existe en `inbox.service.ts`; el cambio es de `limit` default + orden, no de contrato:
  - [x] Fijar `limit` default = 5 (hard-coded, no configurable) en las 5 secciones existentes: Alertas, Accionables, Curso automático, Esperando terceros, Por identificar.
  - [ ] Alertas: actualizar `ALERT_MAX_DAYS_AHEAD` y `severityFor()` a los umbrales del plan (rojo ≤5 días vencido, amarillo 5-10, verde >10) y ordenar por severidad (rojo → amarillo → verde).
  - [ ] Accionables, Esperando terceros: orden por fecha de creación desc (más recientes).
  - [ ] Curso automático: orden por progreso calculado asc (reutilizar cálculo `completedCount/totalSteps` de `inbox.service.ts:263`, más atrasadas primero).
  - [ ] Por identificar (movimientos): orden por fecha de llegada desc (más recientes) — ya implementado en `getUnidentified`, solo ajustar límite/orden si hace falta.

- [ ] **Frontend (carmi-digital):**
  - [ ] Renderizar el contador "5 de 20" debajo de cada título de sección (si `total > 5`) en `InboxDashboard.tsx`.
  - [ ] Eliminar el botón "Cargar más" existente por sección.
  - [ ] Asegurar que la UI no se corta ni cambia layout si hay 3 items vs 20.

- [ ] **Test:**
  - [ ] Backend: endpoint devuelve máx 5 items en cada sección (de las 5 en alcance), `total` es cantidad real.
  - [ ] Backend: orden por severidad en Alertas con umbrales 5/10 (rojo antes que amarillo).
  - [ ] Backend: orden por fecha en Accionables/Esperando terceros/Por identificar.
  - [ ] Backend: orden por progreso asc en Curso automático.
  - [ ] Frontend: contador visible cuando `total > 5`, invisible cuando `total ≤ 5`; botón "Cargar más" ya no existe.
  - [ ] Playwright: navegar a Inbox, verificar máx 5 refs por sección, contador visible, sin errores de consola.

## Riesgos y side effects a vigilar

- **Escala:** El `total` ya se calcula hoy (Prisma `$transaction([findMany, count])` en Alertas/Accionables/Curso automático; en memoria en Esperando terceros/Por identificar). No es un cálculo nuevo, así que no hay riesgo de performance adicional por este cambio — el riesgo preexistente en la paginación en memoria queda igual.

- **Contrato de la API:** El shape `{ items, total, page, limit, totalPages }` ya existe; este plan solo cambia el `limit` default a 5 (hard-coded) y el orden de cada sección. Único consumidor confirmado: `InboxDashboard.tsx` — se actualiza en el mismo plan (elimina "Cargar más"), no hay otros consumidores en el monorepo.

- **Orden en Accionables:** Se usa fecha de creación, pero Enrique en la reunión mencionó "criterios pendientes de definir formalmente". Si durante implementación se requiere un orden diferente (ej. por prioridad MOMP), reparametrizar sin reabrir el plan.

- **Movimientos vs Referencias:** "Por identificar" es la única sección con movimientos. Verificar que el endpoint de unidentified waybills devuelve waybills (no referencias) y que la UI renderiza correctamente (estructura diferente a las otras secciones).

## Criterios de verificación

- ✅ Cada sección del Inbox muestra máx 5 items.
- ✅ Contador "X de Y" aparece solo cuando `Y > 5`.
- ✅ Alertas están ordenadas por severidad (rojo primero).
- ✅ Otras secciones (excepto Alertas) ordenadas por fecha descendente (más recientes primero).
- ✅ Curso automático ordenado por progreso ascendente (más atrasadas primero).
- ✅ "Por identificar" muestra movimientos (no referencias), máx 5, ordenados por fecha descendente.
- ✅ Gate estático verde (tsc, lint, tests).
- ✅ Playwright: navegar Inbox, ver secciones con límite 5, contadores visibles/ocultos según corresponda, sin errores de consola.
