# Pedimento (Anexo 22 / RGCE 2026) — origen real de cada campo siguiendo la cadena completa

**Cadena de datos verificada en `prisma/schema.prisma` + `src/`:**

```
Reference ──> Dgo ──> Guia ──> Invoice ──> InvoiceItem
   │           │                  │              │
   │           └──> GlobalIdentifier             └──> InvoiceItemProduct (serialNumber)
   │
   ├──> Shipment ──> ShipmentContainer / candados / carrier(Company)
   │
   └──> Operation ──> OperationInvoice ──> OperationItem ──> OperationItemTax
                 ├──> OperationTax  (cuadro de liquidación nivel pedimento)
                 ├──> OperationIdentifier (hereda de GlobalIdentifier)
                 └──> OperationPedimento ──> Pedimento ──> PedimentoInvoice / PedimentoPartida
                                                       └──> PedimentoShipment ──> Shipment
```

**Evidencia de código clave (leída, no asumida):**

- `src/operations/services/operation-dispatch.service.ts:1325-1345` — comentario literal *"Reference relations (source of truth for customs config)"*. Patente, aduana, régimen y clave de pedimento se resuelven **desde `Reference`** (`customsPatent`, `customsOffice`, `customsRegime`, `customsDeclarationCode`) con fallback a las columnas planas `Reference.patente/aduana/regimen/clavePedimento`. **No se leen de `Pedimento`.**
- `src/operations/services/operation-dispatch.service.ts:5251-5620` (`buildHbsFromPedimento`) — es el mapeador real Anexo 22 → documento. Confirma qué se llena y qué sale como `''`/`'BORRADOR'`.
- `src/operations/services/operation-dispatch.service.ts:5606+` (`buildHbsFromDgo`) — el mismo contrato se puede armar **solo desde el DGO**, sin Operation ni Pedimento. Prueba de que la cadena, no `Pedimento`, es la fuente.
- `src/operations/services/operations.service.ts:1363-1382` — al crear el `Pedimento` real solo se escriben: `tipoOperacion`, `status`, `reference`, `clientCompany`, `brokerCompany`, `fechaEntrada`, `fechaPago`, `notes`, `createdBy`. `patentNumber`, `numeroPedimento`, `acuses`, `fechaValidacion`, `snapshotData` se fijan **explícitamente a `null`**.
- **Bug confirmado**: `operation-dispatch.service.ts:1600-1640` construye `pedimentoStructured` con `clavePedimento`, `regimen`, `tipoCambio`, `pesoBruto`, `valorDolares`, `valorAduana`, `precioPagado` y hace `prisma.pedimento.upsert({...} as any)`. **Ninguna de esas 7 columnas existe en `model Pedimento`** (verificado). El `as any` oculta el error de tipos; esos valores calculados no tienen dónde aterrizar.
- `PedimentoPartida` solo se escribe en `src/pedimentos/pedimento-subdivision.service.ts:436` (flujo de subdivisión), con `metodoValoracion:'1'` y `umc:'PZA'` hardcodeados (`// TODO: Obtener de clasificación`). `PartidaGravamen`, `PartidaRegulacion` y `PartidaIdentificador` **nunca se escriben en ningún servicio**. El render de partidas se hace al vuelo desde `InvoiceItem` (`buildPartidasFromInvoices`, línea 6043).

**Leyenda de Estado:**

- **Disponible** — el dato existe en la cadena y hoy se lee/escribe de verdad.
- **Disponible sin usar** — la columna/modelo existe, pero el código no la llena o no la lee (placeholder, `''`, TODO).
- **No existe** — no hay dónde ponerlo; hay que construirlo.

**Conteo (sobre las 191 filas listadas; el doc base titula "189 campos" porque colapsa filas de encabezado/reservadas):**

| Estado | Campos |
|---|---|
| Disponible | 79 |
| Disponible sin usar | 41 |
| No existe | 66 |
| No aplica (autoridad / reservado) | 5 |

---

## A. Encabezado del pedimento (página 1)

### A.1 — Encabezado principal del pedimento (45 campos)

| # | Campo | Fuente real hoy (modelo.campo, siguiendo la cadena) | Estado | Nota breve |
|---|---|---|---|---|
| 1 | NÚM. PEDIMENTO | `Pedimento.numeroPedimento` (@unique); componentes desde `Reference.aduana` + `Reference.customsPatent→CustomsPatents.code` | Disponible sin usar | Se compone en `generatePedimento` (`${año} ${aduana} ${patente} ${random}`) pero **no se persiste**: la creación lo fija a `null` y el upsert dice "NO incluir numeroPedimento". `pedimentoNumber` (VarChar 50) es la columna legacy duplicada. |
| 2 | T. OPER. | `Reference.serviceType` → `Pedimento.tipoOperacion` | Disponible | `operations.service.ts:1341` deriva IMP/EXP de `dto.serviceType`; `generatePedimento` lo hardcodea a `'IMP'`. No duplicar: la verdad está en `Reference.serviceType`. |
| 3 | CVE. PEDIMENTO | `Reference.pedimentoTypeId → CustomsDeclarationCode.code` (FK real), fallback `Dgo.clavePedimento` / `Reference.clavePedimento` / `Operation.pedimentoCode` | Disponible | **Corrección al doc base**: no es gap. `Pedimento` no necesita la columna — el catálogo real cuelga de `Reference` y el DGO puede sobrescribirlo por clave. |
| 4 | RÉGIMEN | `Reference.customsRegimeId → CustomsRegime` / `Dgo.regimen` / `Operation.regimeId → Regime.code` | Disponible | Cascada resuelta en código. `Regime` (code/description) es el catálogo Apéndice 16. `Reference.customsRegimeOld`/`regimen` son legacy planos. |
| 5 | DESTINO/ORIGEN | `Operation.merchandiseDestinationId → MerchandiseDestination.code`; también `Dgo.destino` | Disponible sin usar | FK a catálogo real, pero `buildHbsFromPedimento` hardcodea `destino: '9'`. |
| 6 | TIPO CAMBIO | `Operation.exchangeRate` (Decimal 10,4) | Disponible | Se lee (`exchangeRate = Number(p.tipoCambio ?? fo.exchangeRate ?? 1)`; `p.tipoCambio` no existe → cae siempre a Operation). No hay copia congelada en `Pedimento`. |
| 7 | PESO BRUTO | `PedimentoShipment → Shipment.grossWeight`; fallback `InvoiceItem.grossWeight`/`netWeight`; en aéreo `Guia.peso`/`pesoUnidad` | Disponible | **Corrección al doc base ("Falta")**: se calcula real en `buildHbsFromPedimento` y `computePesoBrutoFromInvoiceItems`. |
| 8 | ADUANA E/S | `Operation.customsEntryExitOfficeId → CustomsOffice.aduana`; despacho: `Operation.customsOfficeId`; fallback `Reference.customsOfficeId`/`Reference.aduana`/`Dgo.aduana` | Disponible | Las 4 columnas `Pedimento.aduana*` existen pero **no se escriben ni se leen**; el builder consulta `CustomsOffice` directamente. |
| 9 | MEDIO DE TRANSPORTE (entrada/salida) | `Operation.operationMetadata.customs.transportModeEntry` → `Pedimento.medioTransporte`; catálogo relacional en `Reference.trafficTypeId → TransportMode` | Disponible | Duplicado real (`medioTransporte` vs `medioTransporteEntrada`): el código solo usa `medioTransporte`; `medioTransporteEntrada` es huérfano. |
| 10 | MEDIO DE TRANSPORTE DE ARRIBO | `Operation.operationMetadata.customs.transportModeArrival` → `Pedimento.medioTransporteArribo` | Disponible | Default `'4'`. |
| 11 | MEDIO DE TRANSPORTE DE SALIDA | `Operation.operationMetadata.customs.transportModeExit` → `Pedimento.medioTransporteSalida` | Disponible | Default `'7'`. |
| 12 | VALOR DÓLARES | `Operation.totalValue` (NOT NULL) | Disponible | Se lee directo de Operation; `Pedimento.totalValue` existe pero no se llena. |
| 13 | VALOR ADUANA | `Operation.customsValue` (ya en MXN) | Disponible | Escrito en `operations.service.ts:1324`. |
| 14 | PRECIO PAGADO / VALOR COMERCIAL | `SUM(getInvoiceNetTotal(Invoice))` sobre `Invoice`/`InvoiceItem.totalValue` (neto de descuentos) | Disponible | Calculado al vuelo, nunca persistido a nivel Pedimento. |
| 15 | RFC DEL IMPORTADOR/EXPORTADOR | **`Reference.clientCompanyId → Company.taxId`** (`Company.rfc` como alterno) | Disponible | **NO duplicar en `Pedimento`**. `Pedimento.clientCompanyId` y `Operation.clientCompanyId` son copias de `Reference.clientCompanyId`; el builder ya lee `p.clientCompany?.taxId \|\| fo.clientCompany?.taxId`. |
| 16 | CURP DEL IMPORTADOR/EXPORTADOR | `Reference.clientCompanyId → Company.curp` | Disponible sin usar | El builder emite `curp_importador: ''`. `Pedimento.curpImportador` tiene **0 usos en `src/`**. |
| 17 | NOMBRE / RAZÓN SOCIAL | `Reference.clientCompanyId → Company.legalName` | Disponible | Igual que #15: vive en Reference, no hay que duplicarlo. |
| 18 | DOMICILIO DEL IMPORTADOR/EXPORTADOR | `Reference.clientCompanyId → Company → EntityAddress(entityOrigin, entityOriginId) → Address` (street1/2, exteriorNumber, interiorNumber, neighborhood, municipality, city, state, postalCode, country) | Disponible sin usar | Modelo completo y correcto, pero el builder emite `domicilio: ''`. |
| 19 | VAL. SEGUROS | `Invoice.insurance` + `Adjustment(concept.code='INC_VAL_SEG')` + `Reference.globalExpenses→GlobalExpense` | Disponible | `computeIncrementablesDecrementables`. Fuente = Factura + Bolsa de Gastos de la Referencia, no `Pedimento.valSeguros`. |
| 20 | SEGUROS | `Invoice.insurance` / `insuranceIncludable` + `Adjustment 'INC_SEG'` + `GlobalExpense` | Disponible | Ídem. |
| 21 | FLETES | `Invoice.freight` / `freightIncludable` + `Adjustment 'INC_FLE'` + `GlobalExpense` | Disponible | Ídem. |
| 22 | EMBALAJES | `Invoice.packaging` / `packagingIncludable` + `Adjustment 'INC_EMB'` | Disponible | Ídem. |
| 23 | OTROS INCREMENTABLES | `Invoice.otherCharges` / `otherChargesIncludable` + `Adjustment 'INC_OTR'` | Disponible | Ídem. |
| 24 | TRANSPORTE DECREMENTABLES | `Adjustment/GlobalExpense` con `concept.code='DEC_TRA'`, `kind='DECREMENTABLE'` | Disponible | `Pedimento.transporteDecr` existe como copia no usada. |
| 25 | SEGURO DECREMENTABLES | `Adjustment/GlobalExpense 'DEC_SEG'` | Disponible | Ídem. |
| 26 | CARGA DECREMENTABLES | `Adjustment/GlobalExpense 'DEC_CAR'` → `Pedimento.carga` | Disponible | **Nombre confirmado**: la columna `carga` existe y el flujo la calcula. |
| 27 | DESCARGA DECREMENTABLES | `Adjustment/GlobalExpense 'DEC_DES'` → `Pedimento.descarga` | Disponible | **Nombre confirmado**. |
| 28 | OTROS DECREMENTABLES | `Adjustment/GlobalExpense` decrementable sin código específico | Disponible | Cajón por defecto en el `else`. |
| 29 | ACUSE ELECTRÓNICO DE VALIDACIÓN | `Pedimento.acuses` (Json) | Disponible sin usar | 1 sola aparición en `src/`: `acuses: null` en la creación. Nadie lo escribe ni lo lee. |
| 30 | CÓDIGO DE BARRAS | (ninguno) — derivable de `Pedimento.numeroPedimento` | No existe | Apéndice 17; no hay campo ni generador. |
| 31 | CLAVE DE LA SECCIÓN ADUANERA DE DESPACHO | `Operation.customsOfficeId → CustomsOffice.code`; catálogo `CustomsSection`; fallback `Reference.seccion` (default `'0'`) | Disponible | Se emite como `seccion`/`aduanera_de_despacho`. `Pedimento.aduanaSeccionDespacho` no se usa. |
| 32 | MARCAS, NÚMEROS Y TOTAL DE BULTOS | Bultos: `PedimentoShipment→Shipment.packageUnits(ShipmentPackageUnit.quantity)` \|\| `Shipment.pieces` \|\| `computeDgoTotalBultos(Guia.bultos)`. Marcas: `Pedimento.marcasNumeros` | Disponible sin usar | Bultos: **cálculo real y bueno** (cadena Shipment/Guía). Marcas: hardcodeado a `'S/M'`, nunca se captura. |
| 33 | FECHAS (ENTRADA / PAGO / EXTRACCIÓN / PRESENTACIÓN / IMP.EUA-CAN / ORIGINAL) | ENTRADA: `Operation.entryExitDate` → `Pedimento.fechaEntrada`. PAGO: `Operation.paymentDate` → `Pedimento.fechaPago` | Disponible sin usar | Solo 2 de 6 tipos de fecha. `Pedimento.fechasJson` tiene **0 usos**; EXTRACCIÓN/PRESENTACIÓN/ORIGINAL no existen. |
| 34 | CONTRIB. | **`Operation.operationTaxes → OperationTax.contributionCode`** | Disponible | **Corrección mayor al doc base ("Backend pendiente / tasas JSON")**: existe tabla relacional `operation_taxes` y `buildTasasYLiquidacion` la consume. Escrita en `operations.service.ts:1296`. |
| 35 | CVE. T. TASA | `OperationTax.rateCode` ('1' ad-valorem, '2' cuota fija, '7' al millar) | Disponible | Ídem, con conversión de formato SAT (×100 / ×1000). |
| 36 | TASA | `OperationTax.rateValue` (Decimal 10,5) | Disponible | Formateado a 5 decimales. |
| 37 | CONCEPTO | `OperationTax.contributionCode` | Disponible | El `// TODO: Calculate from partidas` con `contribuciones: []` es el **camino legacy** (`operation-dispatch.service.ts:3309`); el camino vivo usa `OperationTax`. |
| 38 | F.P. | `OperationTax.paymentForm` | Disponible | Apéndice 13, viene de DB. |
| 39 | IMPORTE | `OperationTax.amount` | Disponible | Redondeo a cero decimales (regla Anexo 22). |
| 40 | EFECTIVO | `SUM(OperationTax.amount WHERE paymentForm='0')` | Disponible | Calculado; `Pedimento.liquidacionEfectivo` es copia no usada. |
| 41 | OTROS | `SUM(OperationTax.amount WHERE paymentForm<>'0')` | Disponible | Ídem `liquidacionOtros`. |
| 42 | TOTAL | efectivo + otros | Disponible | Ídem `liquidacionTotal`. |
| 43 | CERTIFICACIÓN | `Pedimento.codigoAceptacion` | Disponible sin usar | Se **lee** (`p.codigoAceptacion \|\| 'BORRADOR'`) pero ningún servicio lo escribe. |
| 44 | DEPÓSITO REFERENCIADO / PAGO ELECTRÓNICO | `Operation.paymentConfig` (Json) + `Operation.taxesCalculated`; formas de pago vía `PaymentMethod` | Disponible sin usar | El builder emite `banco`, `linea_de_captura`, `importe_pagado`, `numero_de_operacion_bancaria`, `numero_de_transaccion_sat` todos `''`. No hay columnas tipadas para la línea de captura. |
| 45 | CÓDIGO QR / VERIFICADOR | (ninguno) | No existe | Sin campo ni generador. |

### A.2 — Encabezado para páginas secundarias (5 campos)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | NÚM. PEDIMENTO | `Pedimento.numeroPedimento` (ver A.1 #1) | Disponible sin usar | Se reinyecta en `anexo_pages[].num_pedimento`, pero llega vacío porque nunca se persiste. |
| 2 | TIPO OPER. | `Reference.serviceType` → `Pedimento.tipoOperacion` | Disponible | `anexo_pages[].t_oper`. |
| 3 | CVE. PEDIM. | `Reference.pedimentoTypeId → CustomsDeclarationCode.code` / `Dgo.clavePedimento` | Disponible | `anexo_pages[].cve_pedimento`. |
| 4 | RFC | `Reference.clientCompanyId → Company.taxId` | Disponible | `anexo_pages[].rfc_importador`. |
| 5 | CURP | `Reference.clientCompanyId → Company.curp` | Disponible sin usar | No se emite en las páginas del anexo. |

---

## B. Pie de página / cuerpo a nivel pedimento

### B.1 — Agente aduanal / agencia / representante / apoderado (9 campos)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | NOMBRE, DENOM. O RAZ. SOC. | `Operation.mompCustomsAgentId → MompCustomsAgent.name`; fallback `Pedimento.brokerCompany`/`Operation.brokerCompany → Company.legalName`; alterno `Reference.customsPatentId → CustomsPatents.name` | Disponible | Cascada real de 3 niveles ya implementada. `Pedimento.nombreAgente`/`razonSocialAgencia` tienen **0 usos** — son columnas muertas. |
| 2 | RFC | `MompCustomsAgent.rfc` \|\| `Reference.customsPatent→CustomsPatents.rfc` \|\| `Company.taxId` del broker | Disponible | Misma cascada; `Pedimento.rfcAgente`/`rfcAgencia` no se escriben. |
| 3 | CURP | `Reference.customsPatentId → CustomsPatents.curp` (o `CustomsAgent.curp`) | Disponible sin usar | El builder emite `curp: ''`. `Pedimento.curpAgente` con 0 usos. |
| — | MANDATARIO / PERSONA AUTORIZADA (encabezado) | — | No aplica | Encabezado de sub-bloque, sin dato propio. |
| 4 | NOMBRE (mandatario) | **`Operation.customsAgentId → CustomsAgent.nombre`** | Disponible | **Corrección al doc base ("Backend pendiente / no existe modelo")**: `CustomsAgent(personaId, nombre, rfc, curp)` existe, tiene FK real desde `Operation` y el builder ya emite `nombre_mandatario`. |
| 5 | RFC (mandatario) | `Operation.customsAgentId → CustomsAgent.rfc` | Disponible | `rfc_mandatario`. |
| 6 | CURP (mandatario) | `Operation.customsAgentId → CustomsAgent.curp` | Disponible | `curp_mandatario`. |
| 7 | PATENTE O AUTORIZACIÓN | **`Reference.customsPatentId → CustomsPatents.code`** (única FK con integridad real); fallbacks `MompCustomsAgent.patent`, `Dgo.patente`, `Reference.patente`, `Pedimento.patentNumber` | Disponible | **El doc base marcaba "Discrepancia confirmada"**: sí hay 3 fuentes, pero el código ya define un orden de precedencia estable y `Reference.customsPatentId` es la fuente autoritativa. `Operation.customsPatentId` (VarChar sin FK) está muerto en este flujo. |
| 8 | FIRMA ELECTRÓNICA AVANZADA | `Reference.customsPatent → PatentDigitalSeal` (`cerHash`, `status`, `certificateExpiry`) + `CompanyPatentSignatureConfig`; legacy `Pedimento.fielCertificate/fielPrivateKey/fielPassword` | Disponible sin usar | El builder emite `efirma: 'BORRADOR'`. **Riesgo de seguridad real**: `fielPrivateKey`/`fielPassword` son columnas planas en `Pedimento` sin exclusión en los DTO de lectura. |
| 9 | NÚM. DE SERIE DEL CERTIFICADO | **`PatentDigitalSeal.certificateSerial`** (VarChar 64) | Disponible sin usar | **Corrección al doc base ("Falta")**: el campo existe con `certificateExpiry`/`certificateNotBefore`; el builder emite `numero_de_serie_del_certificado: ''`. |
| — | FIN DEL PEDIMENTO | `total_partidas` = COUNT de `InvoiceItem` renderizados; clave prevalidador: (ninguno) | No existe | El total sí se calcula; la **clave del prevalidador** no existe en el schema (`clave_prevalidador: ''`). |

### B.2 — Datos del proveedor/comprador (11 campos, repetible por CFDI)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | ID. FISCAL | `Invoice.supplierId → Company.taxId` | Disponible | `buildProveedoresFromInvoices`; fallback `'S/D'`. |
| 2 | NOMBRE / RAZÓN SOCIAL | `Invoice.supplierId → Company.legalName` | Disponible | Fallback `'PROVEEDOR NO CONFIGURADO'`. |
| 3 | DOMICILIO | `Invoice.supplierAddressId → Address` (FK directa) o `Company → EntityAddress → Address`; sucursal vía `Invoice.supplierBranchId → Branch` | Disponible sin usar | Modelo correcto y con 3 rutas, pero el builder emite `domicilio: ''`. |
| 4 | VINCULACIÓN | `Invoice.vinculacion` (VarChar 30) | Disponible | Default `'NO'` en el render. |
| 5 | NÚM. CFDI O DOC. EQUIVALENTE | `Invoice.invoiceNumber` → snapshot `PedimentoInvoice.numeroFactura` | Disponible | `PedimentoInvoice` sí se escribe en `operations.service.ts:1404`. |
| 6 | FECHA | `Invoice.invoiceDate` → `PedimentoInvoice.fechaFactura` | Disponible | Ídem. |
| 7 | INCOTERM | **Fuente: `Invoice.incoterm`**; copia congelada: `PedimentoInvoice.incoterm` | Disponible | No hay conflicto: `PedimentoInvoice.incoterm` se llena *desde* `Invoice.incoterm` (`inv.incoterm ?? null`). La factura manda. |
| 8 | MONEDA FACT. | `Invoice.currency` (VarChar 3, NOT NULL); catálogo `Currency` | Disponible | **Corrección al doc base ("Falta")**: existe, es obligatorio y se emite como `moneda`. |
| 9 | VAL. MON. FACT. | `Invoice.totalAmount` (neto vía `getInvoiceNetTotal`) → `PedimentoInvoice.valorMonedaFactura` | Disponible | — |
| 10 | FACTOR MON. FACT. | Calculado (`currency==='USD' ? 1.0 : Operation.exchangeRate`) → `PedimentoInvoice.factorMonedaFactura` | Disponible | — |
| 11 | VAL. DÓLARES | Calculado → `PedimentoInvoice.valorDolares` | Disponible | Bonus: `Invoice.cove` / `PedimentoInvoice.coveEdocument` existen para el COVE, pero el builder emite `cove: ''`. |

### B.3 — Datos del destinatario (3 campos) — exclusivo de exportación

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | ID. FISCAL | `Invoice.recipientId → Company.taxId` | Disponible sin usar | La FK existe (`invoices_recipient_idTocompanies`), pero `buildHbsFromPedimento` no arma bloque destinatario. |
| 2 | NOMBRE / RAZÓN SOCIAL | `Invoice.recipientId → Company.legalName` | Disponible sin usar | Ídem. |
| 3 | DOMICILIO | `Invoice.recipientId → Company → EntityAddress → Address` | Disponible sin usar | Ídem. Alterno: `Shipment.consigneeId → Company`. |

### B.4 — Datos del transporte y transportista (6 campos)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | IDENTIFICACIÓN | **`PedimentoShipment → Shipment.transportIdentification`** (+ `Shipment.caat` para carriles con tecnología) | Disponible | **Corrección al doc base ("Falta / ninguno")**: existen ambas columnas y el builder las emite en `transporte_anexo`. |
| 2 | PAÍS | `Shipment.transportCountry` (VarChar 3) | Disponible | Emitido con default `'MEX'`. |
| 3 | TRANSPORTISTA | `Shipment.carrierName`; relacional `Shipment.carrierId → Company.legalName` | Disponible | **Corrección al doc base**: dos fuentes reales (denormalizada + FK). |
| 4 | RFC | `Shipment.carrierId → Company.taxId` | Disponible sin usar | La ruta correcta existe, pero el código emite `rfc: fo.brokerCompany?.taxId` — **bug: usa el RFC del agente aduanal, no el del transportista**. |
| 5 | CURP | `Shipment.carrierId → Company.curp`; legacy `Pedimento.curpTransportista` | Disponible sin usar | `curpTransportista` tiene **0 usos** en `src/`. |
| 6 | DOMICILIO/CIUDAD/ESTADO | `Shipment.carrierId → Company → EntityAddress → Address`; o `Shipment.carrierBranchId → Branch` | Disponible sin usar | **Corrección al doc base**: la ruta existe completa; no se mapea. |

### B.5 — Candados (3 campos)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | NÚMERO DE CANDADO | `PedimentoShipment → Shipment.shipmentContainers → ShipmentContainer.sealNumber` (relacional, prioritario); fallback `Shipment.candados` (Json) | Disponible | **Corrección al doc base ("Backend pendiente, existe solo a nivel Shipment")**: `PedimentoShipment` es el puente Pedimento↔Shipment y el builder ya lo recorre. |
| 2 | 1RA. REVISIÓN | — | No aplica | Uso exclusivo de la autoridad aduanera. |
| 3 | 2DA. REVISIÓN | — | No aplica | Uso exclusivo de la autoridad aduanera. |

### B.6 — Guías / conocimientos de embarque (2 campos, repetible)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | NÚMERO (GUÍA / DOC. TRANSPORTE) | **`Invoice.guiaId → Guia.guiaMaster` / `Guia.guiaHouse`**; también `Guia.dgoId → Dgo` y `Guia.referenceId → Reference` | Disponible sin usar | `Guia` es un modelo completo (master/house, `numeroVuelo`, `peso`, `bultos`, `piezas`, `remitente`, `destinatario`, domicilios) y ya está enganchado a la cadena en 3 puntos. **Pero `buildHbsFromPedimento` no emite bloque de guías.** |
| 2 | ID. (M / H) | Derivable: `Guia.guiaMaster` presente → `'M'`; `Guia.guiaHouse` → `'H'` | Disponible sin usar | No hay columna explícita de tipo, pero es derivable sin construir nada. |

### B.7 — Contenedores / equipo ferrocarril / núm. económico (2 campos, repetible)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | NÚMERO DE CONTENEDOR/EQUIPO/VEHÍCULO | `PedimentoShipment → Shipment → ShipmentContainer.containerNumber` | Disponible | **Corrección al doc base**: el builder lo emite en `contenedores_anexo`, con fallback a `Shipment.candados`. |
| 2 | TIPO | `ShipmentContainer.containerTypeId → ContainerType.code` | Disponible | Catálogo real (Apéndice 10); fallback `'N/A'`. |

### B.8 — Identificadores (nivel pedimento) (4 campos, repetible)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | CLAVE | **`Reference.globalIdentifiers → GlobalIdentifier.code`** (con `dgoId` para acotar al DGO) → heredado a `Operation.identifiers → OperationIdentifier.code`; alterno `ReferenceIdentifier.clave` | Disponible | **Corrección mayor al doc base ("PedimentoIdentificador no modelado")**: no hace falta un `PedimentoIdentificador` — el nivel "G" vive en `GlobalIdentifier` (por Referencia y por DGO) y se materializa en `OperationIdentifier`. El builder lo emite como `identificadores_pedimento`. |
| 2–4 | COMPL. IDENTIFICADOR 1/2/3 | `GlobalIdentifier.complement1/2/3` → `OperationIdentifier.complement1/2/3`; alterno `ReferenceIdentifier.c1/c2/c3` | Disponible | Ídem, con `isInherited` y `sourceGlobalIdentifierId` para trazabilidad. |

### B.9 — Cuentas aduaneras / garantía / carta de crédito (nivel pedimento) (7 campos)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | TIPO | (ninguno en toda la cadena) | No existe | Grep global: 0 coincidencias de `cuentaAduanera`/`cartaCredito`/`garantia` en `schema.prisma`. |
| 2 | CLAVE DE GARANTÍA | (ninguno) | No existe | Los 7 hits de `guarantee` son `MaritimeGuaranteeRecovery` (garantía de contenedor marítimo), concepto distinto. |
| 3 | INSTITUCIÓN EMISORA | (ninguno) | No existe | — |
| 4 | NÚM. DE CONTRATO O CARTA DE CRÉDITO | (ninguno) | No existe | — |
| 5 | FOLIO CONSTANCIA | (ninguno) | No existe | — |
| 6 | IMPORTE TOTAL | (ninguno) | No existe | — |
| 7 | FECHA | (ninguno) | No existe | — |

### B.10 — Descargos (3 campos, repetible)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | NÚM. PEDIMENTO ORIGINAL | (ninguno) | No existe | `OperationPedimento.pedimentoType` (default `'ORIGINAL'`) es el único gancho conceptual, sin campos de descargo. |
| 2 | FECHA DE OPERACIÓN ORIGINAL | (ninguno) | No existe | — |
| 3 | CVE. PEDIMENTO ORIGINAL | (ninguno) | No existe | — |

### B.11 — Compensaciones (4 campos, repetible)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | NÚM. PEDIMENTO ORIGINAL | (ninguno) | No existe | Confirmado con grep global. |
| 2 | FECHA DE OPERACIÓN ORIGINAL | (ninguno) | No existe | — |
| 3 | CLAVE DE GRAVAMEN | (ninguno) | No existe | `PartidaGravamen.claveContribucion` es a nivel partida, no compensación. |
| 4 | IMPORTE DEL GRAVAMEN | (ninguno) | No existe | — |

### B.12 — Documentos que amparan formas de pago distintas a efectivo (7 campos)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | FORMAS DE PAGO | Parcial: catálogo `PaymentMethod` + `OperationTax.paymentForm` | No existe | Existe la *clave* de forma de pago, no el **documento** que la ampara. |
| 2 | DEPENDENCIA O INSTITUCIÓN EMISORA | (ninguno) | No existe | — |
| 3 | NÚMERO DEL DOCUMENTO | (ninguno) | No existe | — |
| 4 | FECHA DEL DOCUMENTO | (ninguno) | No existe | — |
| 5 | IMPORTE DEL DOCUMENTO | (ninguno) | No existe | — |
| 6 | SALDO DISPONIBLE | (ninguno) | No existe | — |
| 7 | IMPORTE A PAGAR | (ninguno) | No existe | — |

### B.13 — Observaciones (nivel pedimento) (1 campo)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | OBSERVACIONES | `Pedimento.notes` \|\| `Operation.notes`; captura natural en la cadena: **`Dgo.observations`** y `Reference.observations`/`Reference.notes`/`clientNotes` | Disponible | **Corrección al doc base ("no se encontró código que lo lea/escriba")**: `buildHbsFromPedimento` emite `observaciones_generales: p.notes \|\| fo.notes`, y `operations.service.ts:1379` escribe `notes: group.notes` por grupo/DGO. `draft_data` y `snapshotData` sí son ruido (0-2 usos). |

### B.14 — Rectificaciones (4 campos)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | PEDIMENTO ORIGINAL | (ninguno) | No existe | Solo `OperationPedimento.pedimentoType` con valor fijo `'ORIGINAL'`; no hay auto-referencia pedimento→pedimento original. |
| 2 | CVE. PEDIM. ORIGINAL | (ninguno) | No existe | — |
| 3 | CVE. PEDIM. RECT. | (ninguno) | No existe | — |
| 4 | FECHA PAGO RECT. | (ninguno) | No existe | — |

### B.15 — Diferencias de contribuciones (nivel pedimento) (6 campos)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | CONCEPTO | (ninguno) | No existe | `OperationTax` cubre el cuadro de liquidación normal, no el de diferencias por rectificación. |
| 2 | F.P. | (ninguno) | No existe | — |
| 3 | DIFERENCIA | (ninguno) | No existe | — |
| 4 | EFECTIVO | (ninguno) | No existe | — |
| 5 | OTROS | (ninguno) | No existe | — |
| 6 | DIF. TOTALES | (ninguno) | No existe | — |

### B.16 — Prueba suficiente (3 campos) — exportación con retorno T-MEC/Canadá

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | PAÍS DESTINO | Parcial: `Reference.destinationCountry` (Char 2) existe, pero sin semántica de prueba suficiente | No existe | El bloque como tal no está modelado. |
| 2 | NÚM. PEDIMENTO EUA/CAN | (ninguno) | No existe | — |
| 3 | PRUEBA SUFICIENTE (catálogo I–V) | (ninguno) | No existe | — |

---

## C. Cuerpo de partidas (páginas "ANEXO DEL PEDIMENTO")

> **Nota transversal C.1–C.4:** el render real de partidas (`buildPartidasFromInvoices`, `operation-dispatch.service.ts:6043`) se hace **directo desde `Invoice.items → InvoiceItem`**, sin pasar por `PedimentoPartida`. `PedimentoPartida` solo se materializa en el flujo de subdivisión (`pedimento-subdivision.service.ts:436`). Por eso la "fuente real" de casi toda la sección es `InvoiceItem`, y `PedimentoPartida.*` es el snapshot opcional.

### C.1 — Partidas (25 campos, repetible)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | SEC. | Contador secuencial sobre `Invoice.items`; snapshot `PedimentoPartida.secuencia` | Disponible | `@@unique([pedimentoId, secuencia])`. |
| 2 | FRACCIÓN | `InvoiceItem.fullTariffCode` \|\| `tariffFraction` \|\| `htsCode` → `PedimentoPartida.fraccion` | Disponible | Fallback `'S/D'`; enriquecida vía `InvoiceItem.enrichedTariffId → TariffFractionEnriched`. |
| 3 | SUBD. / NICO | `InvoiceItem.nico` (VarChar 2) → `PedimentoPartida.nico` | Disponible | **Corrección al doc base ("Parcial/Ambiguo")**: el campo existe en ambos modelos y se emite (`subd`, default `'00'`). |
| 4 | VINC. | `InvoiceItem.vinculacion` (Boolean) \|\| `Invoice.vinculacion` → `PedimentoPartida.vinculacion` | Disponible | Se convierte a `'1'`/`'0'`. |
| 5 | MET. VAL. | `InvoiceItem.metodoValoracion` (SmallInt) \|\| `Invoice.valoracion` | Disponible | En el flujo de subdivisión está **hardcodeado a `'1'`**. |
| 6 | UMC | `InvoiceItem.commercialUnit`/`unitOfMeasure` → `CoveUnit.code → PedimentoUnit.appendixCode` (Apéndice 7) | Disponible | Traducción de catálogo real (`resolveUnit`). En subdivisión está hardcodeado a `'PZA'` (TODO). |
| 7 | CANTIDAD UMC | `InvoiceItem.quantity` (Decimal 14,3) | Disponible | — |
| 8 | UMT | `InvoiceItem.tariffUnit` → `CoveUnit/PedimentoUnit`; catálogo `TariffUnit` | Disponible | — |
| 9 | CANTIDAD UMT | `InvoiceItem.tariffQuantityCalc` \|\| `tariffQuantity` \|\| `quantity` | Disponible | Conversión vía `InvoiceItem.conversionFactor` / `OperationItemConversion`. |
| 10 | P.V/C | `InvoiceItem.paisVendedor` (Char 2); fallback `Invoice.supplier.country` | Disponible | — |
| 11 | P.O/D | `InvoiceItem.originCountry` \|\| `procedenceCountry` (+ `originCountry3`) | Disponible | Catálogo `Country`/`Country2`. |
| 12 | DESCRIPCIÓN | `InvoiceItem.pedimentDescription` \|\| `description` \|\| `partNumber` | Disponible | `pedimentDescription` es el campo específico de pedimento. |
| 13 | VAL. ADU / VAL. USD | `InvoiceItem.totalValue`; con prorrateo de incrementables → `PedimentoPartida.valorAduana` | Disponible | El prorrateo real solo ocurre en el flujo de subdivisión; el render directo usa `totalValue` sin prorratear. |
| 14 | IMP. PRECIO PAG. / VALOR COMERCIAL | `InvoiceItem.totalValue` (neto) / `grossValue` (bruto pre-descuento) | Disponible | `OperationItem.proratedIncrementables` guarda el prorrateo por item. |
| 15 | PRECIO UNIT. | `InvoiceItem.unitValue` \|\| `unitPrice` | Disponible | — |
| 16 | VAL. AGREG. | `InvoiceItem.valorAgregado`; total por factura en `Invoice.valorAgregadoTotal`/`valorAgregadoUsd`/`valorAgregadoExchangeRate` | Disponible sin usar | Modelo completo (SPEC-VA-001), pero el builder emite `val_agreg: ''`. |
| 17 | (VACÍO) | — | No aplica | Campo reservado, sin dato. |
| 18 | MARCA | `InvoiceItem.marca` (VarChar 100) → `PedimentoPartida.marca` | Disponible sin usar | Existe en ambos modelos; el builder emite `marca: ''`. |
| 19 | MODELO | `InvoiceItem.modelo` (VarChar 100) → `PedimentoPartida.modelo` | Disponible sin usar | Ídem, `modelo: ''`. |
| 20 | CÓDIGO PRODUCTO | `InvoiceItem.partNumber` → `PedimentoPartida.codigoProducto` | Disponible | Se emite como `codigo_producto`. |
| 21 | CON. | **`OperationItem.operationItemTaxes → OperationItemTax.contributionCode`**; snapshot `PartidaGravamen.claveContribucion` | Disponible sin usar | `OperationItemTax` **sí se escribe** (`operations.service.ts:1307`), pero el builder lee `item.taxes.breakdown` — propiedad que **no existe en `InvoiceItem`** — así que cae al prorrateo de la factura o a `[]`. |
| 22 | TASA | `OperationItemTax.rateValue`; snapshot `PartidaGravamen.tasa` | Disponible sin usar | El builder emite `tasa: ''`. |
| 23 | T.T. | `OperationItemTax.rateCode`; snapshot `PartidaGravamen.tipoTasa` | Disponible sin usar | El builder emite `tt: ''`. |
| 24 | F.P. | `PartidaGravamen.formaPago`; a nivel pedimento `OperationTax.paymentForm` | Disponible sin usar | El builder **hardcodea `fp: '0'`** (efectivo). `OperationItemTax` no tiene columna `paymentForm`. |
| 25 | IMPORTE | `OperationItemTax.amount`; snapshot `PartidaGravamen.importe` | Disponible sin usar | Hoy se aproxima prorrateando el breakdown de la factura. `PartidaGravamen` nunca se escribe en ningún servicio. |

### C.2 — Mercancías (2 campos) — sub-bloque de C.1

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | NIV / NÚM. SERIE | **`InvoiceItem.products → InvoiceItemProduct.serialNumber`** → `PartidaMercancia.vinOSerie` | Disponible sin usar | Cadena real confirmada (`pedimento-subdivision.service.ts:468-473`), pero solo se materializa en el flujo de subdivisión; el render directo de partidas no emite bloque de mercancías. |
| 2 | KILOMETRAJE | `PartidaMercancia.kilometraje` (Int) | Disponible sin usar | Se escribe siempre `null` con `// TODO: Agregar si aplica para vehículos`. No hay campo origen en `InvoiceItem`/`InvoiceItemProduct`. |

### C.3 — Regulaciones y restricciones no arancelarias (5 campos, repetible)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | PERMISO | `InvoiceItem.permisos` (Json) / `permitType` / `requiresPermit` / `nomCodes[]`; snapshot `PartidaRegulacion.clavePermiso` | Disponible sin usar | `PartidaRegulacion` **nunca se escribe** (0 referencias en `src/`) y `buildPartidasFromInvoices` no mapea `permisos`. |
| 2 | NÚMERO DE PERMISO | `InvoiceItem.permisos` (Json); Regla Octava vía `InvoiceItem.eighthRulePermitId → EighthRulePermit`; snapshot `PartidaRegulacion.numeroPermiso` | Disponible sin usar | La R8 sí tiene modelo relacional real, pero no alimenta el bloque C.3. |
| 3 | FIRMA DESCARGO | `PartidaRegulacion.firmaDescargo` | Disponible sin usar | Columna existe; sin origen ni escritura. |
| 4 | VAL. COM. DLS. | `PartidaRegulacion.valorComercialDls`; derivable de `InvoiceItem.totalValue` | Disponible sin usar | Ídem. |
| 5 | CANTIDAD UMT/C | `PartidaRegulacion.cantidadUmt`; derivable de `InvoiceItem.tariffQuantity` | Disponible sin usar | Ídem. |

### C.4 — Identificadores (nivel partida) (4 campos, repetible)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | IDENTIF. | **`InvoiceItem.identificadores` (Json: `code`)**; snapshot relacional `PartidaIdentificador.clave` | Disponible | El builder **sí lo emite** (`identificadores_partida`). Catálogo en `IdentifierCatalog` / plantillas en `PartNumberIdentifierTemplate` / `AppliedIdentifier`. `PartidaIdentificador` nunca se escribe. |
| 2–4 | COMPLEMENTO 1/2/3 | `InvoiceItem.identificadores[].complement1/2/3` (acepta `comp1/2/3`); snapshot `PartidaIdentificador.complemento1/2/3` | Disponible | Ídem. |

### C.5 — Cuentas aduaneras de garantía y carta de crédito (nivel partida) (8 campos)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | CVE. GAR. | (ninguno) | No existe | Grep global negativo. |
| 2 | INSTITUCIÓN EMISORA | (ninguno) | No existe | — |
| 3 | FECHA | (ninguno) | No existe | — |
| 4 | NÚMERO DE CUENTA O CARTA DE CRÉDITO | (ninguno) | No existe | — |
| 5 | FOLIO CONSTANCIA | (ninguno) | No existe | — |
| 6 | IMPORTE TOTAL | (ninguno) | No existe | — |
| 7 | PRECIO ESTIMADO | (ninguno) | No existe | — |
| 8 | CANT. U.M. PRECIO EST. | (ninguno) | No existe | — |

### C.6 — Determinación de contribuciones T-MEC 2.5 / Decisión / TLCAELC / ACC (nivel partida)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | VALOR MERCANCÍAS NO ORIGINARIAS | (ninguno) | No existe | `InvoiceItem.hasCertificateOrigin`/`treatyCode`/`preferentialTariff` son insumos de origen, no de este cálculo. |
| 2 | MONTO IGI | `InvoiceItem.igiAmount` existe, pero **no** segregado por mercancía no originaria | No existe | Falta el concepto de "no originaria" en el modelo. |

### C.7 — Observaciones a nivel partida (1 campo)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | OBSERVACIONES A NIVEL PARTIDA | **`InvoiceItem.observaciones` (Json)** y `InvoiceItem.notes` | Disponible sin usar | **Corrección al doc base ("Falta")**: el campo existe. Hoy el builder **ignora** ambos y genera un texto sintético `"{invoiceNumber} - {descripción}"`, que además **repite datos ya declarados** (prohibido por el Anexo 22). |

---

## D. Pedimento complementario (nivel partida)

### D.1 — Contribuciones nivel partida, complementarios art. 2.5 T-MEC (13 campos)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | SEC. | (ninguno) | No existe | Todo el bloque complementario está sin modelar. |
| 2 | FRACCIÓN | (ninguno) | No existe | — |
| 3 | VALOR MERC. NO ORIG. | (ninguno) | No existe | — |
| 4 | MONTO IGI | (ninguno) | No existe | — |
| 5 | TOTAL ARAN. EUA/CAN | (ninguno) | No existe | — |
| 6 | MONTO EXENTO | (ninguno) | No existe | — |
| 7 | F.P. | (ninguno) | No existe | — |
| 8 | IMPORTE | (ninguno) | No existe | — |
| 9 | UMT | (ninguno) | No existe | — |
| 10 | CANTIDAD UMT | (ninguno) | No existe | — |
| 11 | FRACC. EUA/CAN | (ninguno) | No existe | — |
| 12 | TASA EUA/CAN | (ninguno) | No existe | — |
| 13 | ARAN. EUA/CAN | (ninguno) | No existe | — |

### D.2 — Contribuciones nivel partida, complementarios Decisión UE / TLCAELC / ACC (6 campos)

| # | Campo | Fuente real hoy | Estado | Nota breve |
|---|---|---|---|---|
| 1 | SEC. | (ninguno) | No existe | Ídem D.1. |
| 2 | FRACCIÓN | (ninguno) | No existe | — |
| 3 | VALOR MERC. NO ORIG. | (ninguno) | No existe | — |
| 4 | MONTO IGI | (ninguno) | No existe | — |
| 5 | F.P. | (ninguno) | No existe | — |
| 6 | IMPORTE | (ninguno) | No existe | — |

---

## Conclusiones sobre la cadena (qué NO hay que duplicar)

1. **Identidad del importador/exportador (A.1 #15-18, A.2 #4-5)** vive en `Reference.clientCompanyId → Company` (+ `EntityAddress → Address`). `Pedimento.clientCompanyId` y `Operation.clientCompanyId` son copias redundantes. No construir campos nuevos: mapear el domicilio y el CURP que ya existen.
2. **Configuración aduanera del pedimento (A.1 #3, #4, #8, #31; B.1 #7)** vive en `Reference` (`pedimentoTypeId`, `customsRegimeId`, `customsOfficeId`, `customsPatentId`) y se sobrescribe por DGO (`Dgo.clavePedimento/regimen/aduana/patente`). El código ya lo declara "source of truth". Las columnas equivalentes en `Pedimento` son legacy muerto.
3. **Cuadro de liquidación (A.1 #34-42)** ya es relacional: `OperationTax`. El `tasas` Json y el `// TODO: Calculate from partidas` son el camino viejo. A nivel partida existe `OperationItemTax`, pero el render no lo lee (lee una propiedad inexistente).
4. **Transporte y contenedores (B.4, B.5, B.7)** cuelgan de `Shipment` y llegan a `Pedimento` por `PedimentoShipment`. No es un gap de modelo, es un gap de mapeo (y un bug: el RFC del transportista se toma del broker).
5. **Identificadores nivel "G" (B.8)** no requieren un `PedimentoIdentificador`: `GlobalIdentifier` (con `dgoId`) → `OperationIdentifier` ya cubre clave + 3 complementos con trazabilidad de herencia.
6. **Guías (B.6)** están completamente modeladas (`Guia`, con triple anclaje `referenceId`/`dgoId`/`Invoice.guiaId`) y sin usar en el documento.
7. **Bug de esquema**: `operation-dispatch.service.ts` escribe 7 campos (`clavePedimento`, `regimen`, `tipoCambio`, `pesoBruto`, `valorDolares`, `valorAduana`, `precioPagado`) que no existen en `model Pedimento`, enmascarado por `as any`.
