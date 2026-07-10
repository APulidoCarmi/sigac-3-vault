# Sub-plan SP-17: Bandeja de Entrada (Inbox)

Parte de [[2026-07-10-refactor-flujo-ejecutivo]].

## Contexto

Pantalla #1 del [[Inventario_Pantallas_v3]] (🔵 nueva, propuesta M9 — "le gustó a
German"). Punto de arranque del día del ejecutivo. Origen:
[[2026-07-07 - Revision de pantallas flujo operación]] (M9: tablero / bandeja de entrada).

## D1 — punto de partida
- **Reusa:** el tablero primario del Listado de Referencias (SP-01) como base.
- **Crea:** la Bandeja de Entrada con:
  - **Referencias en curso** (tabs: automático / temporal / avanzado) y las que
    **esperan a terceros**: clasificación, previo/almacén, arribo, cartas sin firmar N
    días del cliente, "por identificar".
  - **Auditoría de CEUS** arriba (lo que falta: folios, etc.).
  - **Guías llegadas sin identificar** que se muestran a todos para reconocerlas y
    **ligarlas o crear una referencia**.

## Fuera de alcance
- El listado/tabla clásica de referencias (SP-01).

## Pasos
- [ ] Sección de referencias en curso y en espera de terceros.
- [ ] Panel de auditoría CEUS (pendientes/folios).
- [ ] Guías sin identificar: reconocer → ligar o crear referencia.

## Riesgos y side effects
- Escala alta ⇒ paginar/priorizar en servidor. Solapa con SP-01: definir qué vive
  en cada uno para no duplicar.

## Criterios de verificación
- Gate estático verde. Playwright: abrir la bandeja, ver referencias en espera por
  categoría y ligar una guía sin identificar; sin errores de consola.

## Estado
📋 Por implementar.
