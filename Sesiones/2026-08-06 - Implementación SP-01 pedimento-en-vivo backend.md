# Sesión 2026-08-06 — Implementación SP-01 pedimento-en-vivo (backend)

Relacionado: [[2026-08-05-dgo-pedimento-en-vivo-sp01-backend-contrato-pedimento-data]], [[2026-08-05-dgo-pedimento-en-vivo]], [[2026-08-05 - Revisión pre-implementación SP-01 pedimento-en-vivo]]

## Qué se trabajó

Ejecución completa vía `/implementa` (sesión limpia) de las 15 tareas del
sub-plan SP-01 en `carmi-odin-api-v2`, rama `feat/dgo-pedimento-en-vivo-sp01`:

1. **5 migraciones de Prisma** (más 2 adicionales descubiertas en el camino):
   - Migración 1: quitó el `as any` y las columnas fantasma
     (`clavePedimento`/`regimen`/`tipoCambio`) del upsert de `Pedimento` en
     `generatePedimento`; agregó columnas reales `pesoBruto`/`valorDolares`/
     `valorAduana`/`precioPagado`.
   - Migración 2: `PedimentoRectificacion` + `PedimentoDiferenciaContribucion`
     + catálogo real `OperationPedimentoType` (enum `ORIGINAL|RECTIFICACION|
     COMPLEMENTARIO`, antes string libre).
   - Migración 3: `PedimentoCuentaAduanera`, `PartidaCuentaAduanera`,
     `PedimentoCompensacion`, `PedimentoDocumentoFormaPago`.
   - Migración 4: `PedimentoPruebaSuficiente`, `PartidaContribucionComplementario`,
     `DeterminacionTmecArt25`, `DeterminacionUeAelcAcc`.
   - Migración 5: campos sueltos (código de barras/QR, kilometraje, clave
     prevalidador), 3 fechas nuevas, depósito referenciado/pago electrónico.
   - **Adicional** (no contemplada en los pasos originales): `domicilioImportador`/
     `domicilioExportador`/`certificadoNumeroSerie` en `Pedimento` — hueco
     encontrado al escribir el servicio de materialización.
   - **Adicional**: `paisDestino` en `DeterminacionTmecArt25` — el contrato
     Zod lo exige obligatorio y el modelo de Migración 4 no lo tenía.
2. **Servicio de materialización** en `generatePedimento`: resuelve CURP
   (importador/agente), domicilios reales (`EntityAddress`→`Address`),
   número de serie del certificado FIEL (`PatentDigitalSeal` activo), y deja
   explícitas en `null` las 3 fechas sin fuente.
3. **Corrección T. OPER.** y **deprecación en código** de
   `pedimentoNumber`/`medioTransporteEntrada` (comentarios `@deprecated`,
   sin migración destructiva).
4. **Contrato Zod `PedimentoDataSchema`** (28 bloques, `src/pedimentos/contracts/pedimento-data.types.ts`)
   — transcripción fiel de `PEDIMENTO_PRISMA_ZOD.md` §1. Se agregó `zod`
   como dependencia directa (antes solo transitiva vía Puppeteer/chromium-bidi).
5. **`buildPedimentoDataFromDgo`** y **`buildPedimentoDataFromPedimento`**
   (funciones hermanas de `buildHbsFromDgo`/`buildHbsFromPedimento`, mismo
   patrón de duplicación intencional que el resto de la codebase).
6. **Endpoint `GET /dgo/:id/pedimento-data`** (`dgo.controller.ts` +
   `OperationDispatchService.getPedimentoDataFromDgo`) — resuelve un único
   `Pedimento` por `dgoId` (no el `findMany` sin filtro de
   `getOperationPedimento`).
7. **2 tests de regresión**: exclusión de credenciales (FIEL) en ambos
   builders + derivación de `tieneFirmaElectronicaAvanzada`; T. OPER. en
   `generatePedimento` (3 casos).

## Commits relevantes

Ninguno — todo el trabajo quedó sin commitear en el working tree de
`carmi-odin-api-v2` (rama `feat/dgo-pedimento-en-vivo-sp01`), a la espera
de revisión humana, según la regla dura de `/implementa`.

## Decisiones (con su porqué)

- **`pedimento-subdivision.service.ts` (marca/modelo/valorAgregado de
  `PedimentoPartida`) queda fuera de alcance** — tocar ese archivo hubiera
  expandido el paso "servicio de materialización" más allá de lo declarado
  (`generatePedimento`/upsert de `Pedimento`). Queda como hallazgo para un
  sub-plan aparte.
- **Los bugs de lectura de `pedimentoNumber`** en `saai-validation.service.ts`
  (7 lugares, generación del M-file SAT) y `timeline-hydration.service.ts`
  quedan documentados pero sin corregir — mismo motivo: fuera del alcance
  declarado de "deprecar en código", que solo pedía dejar de escribir/leer
  en código *nuevo*.
- **`kilometraje` se agregó igual como columna en `Pedimento`** pese a ser
  conceptualmente redundante con `PartidaMercancia.kilometraje` (nivel
  partida, ya existente) — el usuario prefirió cumplir el paso del plan
  literalmente en vez de reinterpretar el diseño.
- **No se construyó un test de regresión de T.OPER. para
  `operations.service.ts:create()`** — ese método (creación real de DGO-only)
  no tiene ningún test hoy y es una transacción enorme; montar un harness
  solo para esta línea es desproporcionado y se dejó fuera, documentado como
  hallazgo (deuda de testing preexistente, no introducida por este sub-plan).
- **`buildProveedoresFromInvoices` usa `Company.metadata.address`, que está
  roto** (`Company` no tiene columna `metadata` en el schema real — el
  código siempre cae al fallback). Se descubrió al escribir
  `resolveCompanyDomicilio`, que sí usa la fuente real
  (`EntityAddress`→`Address`). No se corrigió la función existente (fuera
  de alcance), solo se documentó.

## Aprendizajes / errores a no repetir

- Al construir `buildPedimentoDataFromPedimento` pasé un `fmtDate` de
  relleno (`() => ''`) a `buildProveedoresFromInvoices`, lo que habría
  dejado `fechaFactura` siempre vacío en el contrato — detectado y
  corregido antes de cerrar la tarea (revisar siempre las funciones
  helper reusadas con los parámetros reales, no placeholders).
- `jest.clearAllMocks()` limpia el historial de llamadas pero **no** resetea
  `mockResolvedValue` configurado en tests anteriores del mismo archivo —
  un test nuevo puede heredar silenciosamente el mock de un test previo si
  no fija explícitamente su propio valor. Pasó con `mockPrisma.dgo.findUnique`.
- El contrato Zod (`PEDIMENTO_PRISMA_ZOD.md` §1) tenía al menos 2 huecos
  reales contra el modelo Prisma que se estaba diseñando en paralelo
  (campo `paisDestino` obligatorio sin columna; domicilios/certificado sin
  columna) — vale la pena cruzar el contrato de salida contra el schema
  real antes de dar por buena una migración "final".

## Pendientes

- Revisión humana del diff completo (nada commiteado).
- Sub-plan aparte candidato para: fix de `pedimento-subdivision.service.ts`
  (valorAgregado), bugs de `pedimentoNumber` en `saai-validation.service.ts`/
  `timeline-hydration.service.ts`, y arreglo de
  `buildProveedoresFromInvoices` (domicilio de proveedor roto).
- SP-02 (frontend, componentes de bloque/validación) sigue en su propia
  sesión limpia, como corresponde a la regla de "un sub-plan = una sesión".
