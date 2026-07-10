# SP-00 — Conclusiones del spike: esquema de operación virtual / B1

Cierra [[2026-07-10-refactor-flujo-ejecutivo-sp00-spike-b1-virtual]], parte de
[[2026-07-10-refactor-flujo-ejecutivo]]. Responde la Brecha #5 del
[[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]].

## Pregunta

¿Cómo se representa —o cómo habría que representar— una operación **virtual
(B1)** en el modelo de datos actual, y dónde deben vivir sus identificadores y
las alertas de folios?

## Respuesta corta

**No se representa hoy.** No existe ningún flag/enum "virtual" en `Reference`,
`Operation` ni `TransportMode`. Es una **migración de esquema** en
`carmi-odin-api-v2` (Prisma), no un simple valor de enum. `carmi-db-api` es un
esquema legado sin relación con `Reference`/`Operation`/`TransportMode` (0
hits) — la migración futura **no lo involucra**.

## Evidencia (verificada línea por línea contra `carmi-odin-api-v2/prisma/schema.prisma`)

- **`TransportMode`** — tabla catálogo (`model TransportMode`, L6814),
  filas MARITIME/AIR/LAND vinculadas vía `Reference.trafficTypeId`. Confirmado:
  no es un enum, no tiene campo de tipo booleano ni de "virtual".
- **`MompConfiguration`** (L8803): `immex` (L8814) e `ic` (L8816) ya existen
  como `String?` a nivel cliente.
- **`MompSupplier`** (L8905): `companyId` es **opcional** (`String?`, L8917,
  relación `company` hacia `Company`). Este es el punto abierto (ver abajo).
- **`ReferenceIdentifier`** (L3387) y **`GlobalIdentifier`** (L3334): ambas
  con forma clave/código + 3 complementos (`c1/c2/c3` o
  `complement1/2/3`) — corrección menor sobre la numeración citada en el
  sub-plan (los números de línea estaban invertidos entre ambos modelos; la
  forma y la conclusión de reutilización no cambian).
- **Folios**: se revisaron los 4 campos `*Folio` existentes —
  `coveFolio` (L7501), `eDocumentFolio` (L7535), `manifestationFolio` (L7587),
  `referenceFolio` (L8447) — ninguno aplica a folios B1 mensuales.

## Decisiones (quedan documentadas para el `/plan` de implementación de B1)

1. **`isVirtual`**: nuevo campo boolean en `Reference` (y/o `Operation`),
   ortogonal a `trafficTypeId` — una operación virtual puede darse con
   cualquier tráfico (MAR/AER/TER), no solo Aéreo.
2. **Identificadores MS/IM/B1/IC**: reutilizar `ReferenceIdentifier` /
   `GlobalIdentifier` (ya tienen la forma clave+complementos) en vez de crear
   tabla nueva.
   - **MS** (Modalidad de Servicios): complemento constante "1".
   - **IM** (IMMEX propio): fuente = `MompConfiguration.immex` del cliente.
   - **IC** (identificador OEA): fuente = `MompConfiguration.ic` del cliente.
     Es string libre — falta decidir si su sola presencia basta como señal de
     "cliente OEA" o si hace falta un flag explícito (afecta también la regla
     de cierre 10 vs 20 días hábiles).
   - **B1** (IMMEX de la contraparte): leer `MompConfiguration.immex` de la
     `Company` de la contraparte, vía `MompSupplier.companyId → Company`.
3. **Folios B1 mensuales + alertas**: no existe modelo hoy. Propuesta: nueva
   tabla `VirtualOperationFolio` (mes, fecha límite, si aplica extensión OEA,
   estado) ligada a `Reference`; alertas de vencimiento sobre el módulo
   `notifications` ya existente (mismo patrón que `momp-notification.service.ts`).

## Punto abierto — NO resuelto (bloquea diseño de detalle de B1)

`MompSupplier.companyId` es opcional. Si la contraparte **no** es cliente del
sistema (proveedor puramente externo, sin `Company`/`MompConfiguration`
propios), no hay fuente de la que leer su IMMEX para el identificador B1.

El `/plan` de implementación de las pantallas B1 debe decidir: ¿campo manual
de respaldo en `MompSupplier`? ¿bloquear la marca de "virtual" si no hay
match de `Company`? Este spike no lo resuelve — solo lo señala con evidencia.

## Conclusión y alcance de la migración futura

- Afecta **solo `carmi-odin-api-v2`** (Prisma + DTOs + controllers de
  `references`/`operations`). `carmi-db-api` no participa (esquema legado
  distinto).
- No se tocó ni se necesita tocar `carmi-digital` (front) en este spike: no
  hay pantallas ni flujo que probar todavía — eso es la fase posterior
  (`/plan` de pantallas B1), explícitamente fuera de alcance de SP-00.
- Este spike no generó código de producción ni cambios en ningún repositorio;
  es exploratorio y desechable por definición ([[Spikes e investigacion]]).

## Estado
✅ Cerrado. Bloqueaba cualquier sub-plan de pantallas B1/virtual — ya puede
diseñarse el `/plan` de esas pantallas usando esta nota como insumo de
Contexto, resolviendo primero el punto abierto de `MompSupplier.companyId`.
