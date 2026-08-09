# Sesión 2026-08-05 — Revisión pre-implementación SP-01 (materialización de Pedimento)

Relacionado: [[2026-08-05-dgo-pedimento-en-vivo]], [[2026-08-05-dgo-pedimento-en-vivo-sp01-backend-contrato-pedimento-data]]

## Qué se trabajó

Revisión previa a implementación (fase `/plan`, sin escribir código) del
sub-plan SP-01 (`carmi-odin-api-v2`, materialización real de `Pedimento` +
contrato `PedimentoData`). Se releyó el sub-plan contra el código real
(`schema.prisma`, `operations.service.ts`, `operation-dispatch.service.ts`,
`dgo.controller.ts`) y los 3 documentos fuente en
`/Users/angelhubertopulidoburgos/Downloads/pedimento/`, usando dos
subagentes de investigación para no cargar el contexto principal con
volcados de archivos grandes.

## Decisiones (con su porqué)

- **`OperationPedimento.pedimentoType` es el campo a ampliar a catálogo
  real** (`'ORIGINAL' | 'RECTIFICACION' | 'COMPLEMENTARIO'`), no
  `Pedimento.pedimentoType`. Porqué: ese segundo campo es un enum Prisma
  distinto (`NORMAL | CONSOLIDATED`, sobre periodicidad/agrupación) y está
  muerto en la práctica — `src/pedimentos/pedimentos.service.ts` es un stub
  completo (`NotImplementedException` en cada método), ningún flujo real
  de creación de `Pedimento` lo asigna.
- **Depósito referenciado/pago electrónico va en columnas nuevas
  dedicadas en `Pedimento`, no tipando `Operation.paymentConfig`**.
  Porqué: `paymentConfig` ya tiene una forma real y viva —
  `{ IGI, IVA, DTA, PRV, CNT }` (códigos de forma de pago por contribución,
  calculados por `TaxEngineService` y consumidos solo-lectura en
  `carmi-digital`) — reutilizarlo rompería el cálculo de impuestos
  existente.
- **La resolución de e.firma/certificado (`PatentDigitalSeal`) usa la
  cadena FK real `Reference.customsPatentId → CustomsPatents →
  PatentDigitalSeal`**, replicando la query simple de
  `PatentSealsService.findActiveByPatent` — sin inyectar
  `SealResolverService` (atado al dominio VUCEM/firma de documentos, con
  excepciones propias de ese contexto ajenas a este caso de uso).
- **Los ~20 campos de materialización (tabla §3) se escriben todos dentro
  de `generatePedimento`/el `upsert` de `operation-dispatch.service.ts`**,
  no en `operations.service.ts:1363` (que queda como creación del
  borrador inicial). Porqué: es donde ya viven los cálculos reusables
  (`totalWeight`, `totalValue`, etc.) y el upsert real de `Pedimento`.
- El sub-plan se actualizó en el momento (no se esperó a una sesión de
  implementación aparte) porque las correcciones cambian directamente qué
  código hay que tocar — dejarlas sin persistir habría hecho que
  `/implementa` partiera de premisas incorrectas.

## Aprendizajes / errores a no repetir

- `Operation.paymentConfig` (Json sin tipar) es una trampa común: por ser
  Json y no tener DTO explícito, parece "libre para definir", pero ya está
  en uso real en dos repos. Siempre grepear su uso real en frontend +
  backend antes de asumir que un campo Json está vacío.
- Cuando un plan asume que existe "un patrón ya existente para reusar" (en
  este caso, resolución de sellos digitales), verificar con grep si ese
  patrón vive en el dominio correcto — `SealResolverService` existe, pero
  es de otro dominio (VUCEM) y traería complejidad/excepciones ajenas si
  se inyecta tal cual.
- Un campo de schema puede estar "vivo" en el sentido de tener lecturas
  (`saai-validation.service.ts:130` lee `pedimento.pedimentoType`) pero
  estar muerto en escritura (nunca se asigna, siempre cae al default). No
  basta con grepear lecturas para decidir si un campo es real — hay que
  confirmar también las escrituras.

## Pendientes

- Ejecutar `/implementa` sobre este sub-plan en una **sesión limpia**
  nueva (regla dura del proyecto: un sub-plan = una sesión), incluyendo la
  creación de la rama `feat/dgo-pedimento-en-vivo` en `carmi-odin-api-v2`
  antes de tocar código (si no existe ya).
- Las 5 migraciones de Prisma del sub-plan deben generarse con
  `npx prisma migrate dev --name <slug>` (regla global), nunca a mano.
- Quedan fuera de este sub-plan (documentado, no pendiente accionable
  ahora): los 24 bugs de lectura del mapeador
  (`Especificacion_Backend_Bugs_Mapeador_Historias_Usuario.md`, candidato a
  "SP-05" futuro) y el backfill de pedimentos históricos (decisión
  explícita de no hacerlo).
