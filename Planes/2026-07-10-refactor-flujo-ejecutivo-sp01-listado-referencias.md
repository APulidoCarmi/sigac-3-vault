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
- [ ] Verificar la ruta y componente del listado actual (graphify).
- [ ] Tab secundario: tabla clásica con filtros (cliente/tráfico/estatus) y búsqueda.
- [ ] Tab primario: tablero de referencias en curso priorizadas.
- [ ] Paginación/virtualización (escala cientos/miles/día).

## Riesgos y side effects
- **Coordinar con Carlos** (desarrolla el listado de operaciones, M8) para no
  duplicar. Escala alta ⇒ paginar en servidor.

## Criterios de verificación
- Gate estático verde. Playwright: listado carga paginado, filtros funcionan,
  navegación a un detalle sin errores de consola.

## Estado
📋 Por implementar.
