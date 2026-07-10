# Sub-plan SP-08: Recinto (solo lectura) — adaptado por tráfico

Parte de [[2026-07-10-refactor-flujo-ejecutivo]].

## Contexto

Pantalla #12 del [[Inventario_Pantallas_v3]] (🟡 renombrar + adaptar). Que el
ejecutivo sepa, sin preguntarle a almacén, **dónde está físicamente la mercancía y
en qué etapa logística**. Origen: glosario del
[[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]] (recinto fiscalizado, M7). **Decisión
de la entrevista:** nombre unificado **"Recinto"**, contenido adaptado por tráfico.

## D1 — punto de partida
- **Reusa:** `references/components/tabs/ReferenceWarehouse.tsx` (tab Almacén) y sus
  endpoints `GET /references/:id/warehouse/*`.
- **Refactoriza:** renombrar el tab a **"Recinto"**; adaptar el contenido por tráfico:
  **Almacén** (terrestre) / **Almacén Fiscalizado** (aéreo) / **Terminal** (marítimo).
  Contenido de **solo lectura**: estatus de recepción/custodia, ubicación,
  disponibilidad para despacho.
- **Crea:** nada estructural (renombre + variación por tráfico).

## Fuera de alcance
- Gestión del recinto (es rol de almacén; el ejecutivo solo consulta).

## Pasos
- [ ] Renombrar a "Recinto".
- [ ] Adaptar el contenido/etiquetas por tráfico (Almacén / Fiscalizado / Terminal).
- [ ] Asegurar que todo es solo lectura.

## Riesgos y side effects
- Confirmar qué datos expone hoy `/references/:id/warehouse/*` por tráfico.

## Criterios de verificación
- Gate estático verde. Playwright: abrir Recinto en referencias de los 3 tráficos y
  ver el nombre/contenido adaptado, sin acciones de escritura; sin errores de consola.

## Estado
📋 Por implementar.
