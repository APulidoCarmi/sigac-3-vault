# Cómo leer este contexto (léeme primero)

Orientación para las skills (`/plan`, `/implementa`) al trabajar el refactor
del **flujo del ejecutivo de cliente** (importación/exportación) en SIGAC 3.

## Las 3 fuentes tienen roles distintos — no las mezcles

1. **`Reuniones/`** = cómo trabaja **HOY** el ejecutivo en **SIGAC 2** (legacy).
   Son **requisitos y el porqué / dolores a resolver**, NO decisiones de diseño
   de SIGAC 3.
2. **El código de `sigac-3`** = estado **ACTUAL** de SIGAC 3 (parcialmente
   construido). Es el **punto de partida** del refactor (el "as-is"). Explóralo
   con graphify.
3. **`Arquitectura/`** ([[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]] e
   [[Inventario_Pantallas_v3]]) = cómo **DEBE quedar** el flujo nuevo (el
   "to-be", el **destino**). Ojo: lo marcado como propuesta **NO decidida**
   (M8, M9, M10, rediseño de German) no se asume — se pregunta.

**El refactor = llevar el código actual (as-is) al objetivo de Arquitectura
(to-be), resolviendo los dolores que muestran las reuniones.**

## Alcance en el código

- **DENTRO:** el flujo del ejecutivo vive en el front
  `carmi-digital/app/(customerPortal)/customs-operation` (+ las APIs que
  consume, sobre todo `carmi-odin-api-v2`).
- **FUERA:** todo lo llamado **`operations`** (`customerPortal/operations`,
  `dashboard/operations`, `components/operations`, `actions/operations`, y los
  módulos `operations` de las APIs) es **otro flujo que NO nos compete**.
