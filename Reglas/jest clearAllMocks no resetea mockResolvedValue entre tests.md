`jest.clearAllMocks()` (típicamente en un `beforeEach`) limpia el historial
de llamadas (`mock.calls`, `mock.results`) de todos los mocks, pero **no**
resetea la implementación fijada con `.mockResolvedValue(...)`/
`.mockReturnValue(...)` en un test anterior del mismo archivo. Un test
nuevo puede heredar silenciosamente el valor resuelto que dejó configurado
otro test que corrió antes en el mismo `describe`/archivo, si no lo
sobreescribe explícitamente.

**Por qué**: pasó al escribir el test de exclusión de credenciales para
`buildPedimentoDataFromDgo` en `carmi-odin-api-v2`
(`operation-dispatch.service.spec.ts`) — `mockPrisma.dgo.findUnique` traía
el `mockResolvedValue` de un test anterior (`getPedimentoPreviewFromDgo`),
lo que hizo que `computeDgoTotalBultos` (que hace su propio
`prisma.dgo.findUnique` interno) creyera que el DGO existía y siguiera a
`prisma.guia.findMany`, un modelo no mockeado — `TypeError: Cannot read
properties of undefined (reading 'findMany')`. Ver
[[2026-08-06 - Implementación SP-01 pedimento-en-vivo backend]].

**Cómo aplicar**: si un mock de Prisma (o cualquier dependencia) participa
en más de un `it()` del archivo, fija su valor explícitamente al inicio de
cada test que lo use (no confíes en que `clearAllMocks()` en el
`beforeEach` lo deja "limpio"). Si el valor por defecto debe ser el mismo
en casi todos los tests, ponlo en el `beforeEach` compartido (como ya se
hace con `mockPrisma.invoiceItem.findMany.mockResolvedValue([])`), pero
recuerda que un test que necesite un valor distinto debe sobreescribirlo,
y uno que necesite volver al "caso vacío" también debe fijarlo, no asumirlo.
Usa `jest.resetAllMocks()` en vez de `clearAllMocks()` si de verdad quieres
que cada test arranque sin ninguna implementación previa (aunque eso
obliga a re-mockear todo en cada test que lo necesite).
