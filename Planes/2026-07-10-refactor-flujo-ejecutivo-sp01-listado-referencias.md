# Sub-plan SP-01: Listado de Referencias

Parte de [[2026-07-10-refactor-flujo-ejecutivo]].

## Contexto

Pantalla #2 del [[Inventario_Pantallas_v3]] (🔵 nueva). La Referencia necesita
tres piezas de UI: wizard (SP-02), detalle (SP-03) y **listado**. Origen:
[[2026-07-07 - Revision de pantallas flujo operación]] (M9) — se mantienen dos
vistas en tabs: el **tablero/lista** nuevo (top de las más urgentes) como
primaria y la **tabla clásica** (estilo SIGAC 2: filtros, reportes, búsqueda)
como secundaria, mientras los ejecutivos se acostumbran. Glosario "Referencia"
del [[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]].

## D1 — punto de partida
- **Reusa:** componentes de tabla existentes del dominio referencias y el patrón
  de listado. ⚠️ El inventario de código no confirmó la ruta del listado actual
  (probablemente `references/page.tsx`) → **verificar con graphify al implementar**.
- **Refactoriza:** la tabla actual → tab secundario "tabla clásica" con filtros por
  cliente, tráfico y estatus.
- **Crea:** tablero primario (top urgentes) con las señales de M9 (referencias en
  curso, en espera de terceros, por identificar) — versión mínima; el Inbox
  completo es SP-17.

## Fuera de alcance
- La Bandeja de Entrada / Inbox completa (auditoría CEUS, guías sin identificar):
  SP-17.

## Pasos
- [x] Verificar la ruta y componente del listado actual (graphify): confirmado
      `references/page.tsx` → `ui/ReferencesClient.tsx`.
- [x] Tab secundario: tabla clásica con filtros (cliente/tráfico/estatus) y búsqueda.
      Cliente y tráfico son nuevos (`useClients`/`useTransportModes`); estatus
      amplía D1 (3→6 valores reales de `ReferenceStatus`). Backend: nuevo query
      param `trafficTypeId` en `GET /references` (odin).
- [x] Tab primario: tablero de referencias en curso priorizadas (versión mínima,
      `ReferenceBoard.tsx`, 3 columnas mapeadas a `ReferenceStatus` — ver nota).
- [x] Paginación/virtualización: ya existía server-side en D1 (page/limit); no
      se tocó, cumple el criterio de escala.

## Riesgos y side effects
- **Coordinar con Carlos** (desarrolla el listado de operaciones, M8) para no
  duplicar. Escala alta ⇒ paginar en servidor.

## Criterios de verificación
- Gate estático verde. Playwright: listado carga paginado, filtros funcionan,
  navegación a un detalle sin errores de consola.

## Nota de implementación — mapeo de columnas del tablero

El sub-plan no fija a qué campo mapean las 3 categorías de M9 ("en curso" /
"en espera de terceros" / "por identificar"); no existe un campo dedicado.
Decisión tomada en la implementación (documentada, no improvisada a ciegas):
usar `ReferenceStatus` (prisma/schema.prisma) como proxy —
`DRAFT`="por identificar", `PENDING_QUOTE`="en espera de terceros",
`QUOTED`+`APPROVED`="en curso". Sin orden por urgencia real (ETA/prioridad):
el endpoint no expone `orderBy` custom hoy; se usa el orden por defecto
(`createdAt desc`). Una priorización real queda para el Inbox completo (SP-17).

## Estado
✅ Cerrado (2026-07-10). Ramas: `refactor/customs-operation-sp01` en
`carmi-odin-api-v2` (commit `64402b45`, filtro `trafficTypeId` +
cierre de spec coverage preexistente del controller) y en `carmi-digital`
(commits `db4f2678`, `0354dd90`, sobre `refactor/customs-operation-sp03`).
Gates estáticos verdes (lint, tsc, jest). Playwright NO ejecutado: el
entorno usa credenciales reales (`.env`) contra infraestructura viva sin
supervisión humana — no se navegó el flujo en vivo, ver guía de pruebas.
