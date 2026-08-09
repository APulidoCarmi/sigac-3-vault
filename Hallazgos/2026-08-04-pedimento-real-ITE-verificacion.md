# Pedimento real de referencia — IMP / ITE (importación temporal) — verificación campo por campo

**Origen:** pedimento real proporcionado por el usuario (`Archivo (6).pdf`, 3 páginas). Referencia `0661651`, pedimento `26 64 3739 6002152`, régimen **ITE** (importación temporal para elaboración — IMMEX), clave **IN**, 4 partidas, misma fracción arancelaria (85389001), proveedor único con 4 facturas, operación triangulada (proveedor ≠ exportador legal).

Este documento NO repite los 189 campos genéricos del Anexo 22 (ver `2026-08-04-pedimento-campos-cadena-completa.md`). Aquí se listan **solo los campos que este pedimento real trae con dato**, con su valor real, y se marca si el sistema hoy lo puede reproducir tal cual, siguiendo la cadena `Reference → Dgo → Guia → Invoice/InvoiceItem → Operation → Pedimento`.

**Leyenda:** ✅ Cumple (se calcula/lee y se emite hoy) · ⚠️ Cumple parcial (el dato existe en algún modelo de la cadena pero el render no lo emite — gap de mapeo, no de schema) · ❌ No cumple (no hay dónde ponerlo o hay un bloqueo de schema real)

---

## 1. Encabezado principal (página 1)

| Campo | Valor real en el pedimento | Estado | Evidencia |
|---|---|---|---|
| NÚM. PEDIMENTO | `26 64 3739 6002152` | ❌ No cumple | Se compone (`generatePedimento`) pero **nunca se persiste**: la creación del `Pedimento` fija `numeroPedimento: null` explícitamente. Bloqueante para imprimir el documento tal cual. |
| T. OPER. | IMP | ✅ Cumple | `Reference.serviceType`. |
| CVE. PEDIMENTO | IN | ✅ Cumple | `Reference.pedimentoTypeId → CustomsDeclarationCode`; seed confirma `{code:'IN', regimeCode:'ITE'}`. |
| RÉGIMEN | ITE | ✅ Cumple | `Regime.code='ITE'` existe en catálogo (seed `seed-regimes.ts:22`) y el motor de impuestos (`tax-engine.service.ts:183`) ya trata `ITE` como régimen temporal (afecta forma de pago del IGI). |
| DESTINO | 9 | ⚠️ Cumple parcial | El builder **hardcodea** `destino: '9'` siempre — en este pedimento coincide por casualidad, pero no es un valor calculado desde `MerchandiseDestination`. |
| TIPO DE CAMBIO | 17.51300 | ✅ Cumple | `Operation.exchangeRate`. |
| PESO BRUTO | 74.320 | ✅ Cumple | Calculado real desde `Shipment`/`InvoiceItem`/`Guia`. |
| ADUANA E/S | 640 | ✅ Cumple | `Operation.customsOfficeId → CustomsOffice`. |
| MEDIOS DE TRANSPORTE (4/4/7) | Entrada 4, Arribo 4, Salida 7 | ✅ Cumple | `Operation.operationMetadata.customs.*`. |
| VALOR DÓLARES | 2,415.92 | ✅ Cumple | `Operation.totalValue`. |
| VALOR ADUANA | 42,310 | ✅ Cumple | `Operation.customsValue`. |
| PRECIO PAGADO/VALOR COMERCIAL | 25,602.00 | ✅ Cumple | Suma calculada de `Invoice`/`InvoiceItem`. |
| RFC / NOMBRE del importador | TEM6006038H0 / TYCO ELECTRONICS MEXICO, S. DE R.L. DE C.V. | ✅ Cumple | `Reference.clientCompanyId → Company`. |
| CURP del importador | (vacío — persona moral) | ✅ No aplica | Correcto: personas morales no llevan CURP. |
| DOMICILIO del importador | Av. Vía Dr. Gustavo Baz Prada... Tlalnepantla... | ⚠️ Cumple parcial | El modelo existe completo (`Company → EntityAddress → Address`), pero el builder emite `domicilio: ''`. Gap de mapeo, no de schema. |
| VAL. SEGUROS / SEGUROS / FLETES / EMBALAJES / OTROS INCR. | 25,602.00 / 8 / 16,701 / 0 / 0 | ✅ Cumple | `Invoice` + `Adjustment`/`GlobalExpense`. |
| VALOR DECREMENTABLES (5 columnas, todas 0) | 0 / 0 / 0 / 0 / 0 | ✅ Cumple | `Adjustment`/`GlobalExpense` decrementables. |
| CÓDIGO DE ACEPTACIÓN | DPGT2IG8 | ❌ No cumple | `Pedimento.codigoAceptacion` existe y se **lee**, pero ningún servicio lo **escribe** — este valor lo genera el validador/SAAI externo; hoy no hay integración que lo reciba y persista. |
| CLAVE SECCIÓN ADUANERA DE DESPACHO | 640 | ✅ Cumple | `Operation.customsOfficeId → CustomsOffice.code`. |
| MARCAS, NÚMEROS | S/M | ⚠️ Cumple parcial | Hardcodeado a `'S/M'` siempre — coincide aquí, pero no es capturado. |
| TOTAL DE BULTOS | 13 | ✅ Cumple | Calculado real desde `Shipment`/`Guia.bultos`. |
| FECHAS (Entrada / Pago) | 25/07/2026 / 27/07/2026 | ✅ Cumple | `Operation.entryExitDate` / `Operation.paymentDate`. |
| TASAS A NIVEL PEDIMENTO (DTA 4/462, PRV 2/330, IVA PRV 1/16) | ver tabla | ✅ Cumple | `OperationTax` relacional; el motor de impuestos genera literalmente los códigos `DTA`, `PRV`, `IVA_PRV` (`tax-engine.service.ts:309-353`). |
| CUADRO DE LIQUIDACIÓN (DTA 462, IVA PRV 53, IVA 6843, PRV 330; Efectivo 7,688 / Otros 0 / Total 7,688) | ver tabla | ✅ Cumple | Derivado de `OperationTax`. |

## 2. Depósito referenciado / pago electrónico

| Campo | Valor real | Estado | Evidencia |
|---|---|---|---|
| Leyenda "\*\*\* PAGO ELECTRONICO \*\*\*" + código de barras de línea de captura | presente | ⚠️/❌ Mixto | La línea de captura como dato existe (`lineaCaptura`, `operation-dispatch.service.ts:3714`), pero el **código de barras gráfico no existe** (confirmado: 0 resultados de barcode/QR para pedimento en todo el schema). |
| PATENTE / PEDIMENTO / ADUANA (repetidos) | 3739 / 6002152 / 640 | ✅/❌ Mixto | Patente y aduana sí (ver arriba); pedimento no (ver NÚM. PEDIMENTO). |
| BANCO | HSBC | ❌ No cumple | No hay columna tipada; `operation-dispatch.service.ts` emite `banco: ''` fijo. |
| IMPORTE PAGADO / FECHA Y HORA DE PAGO | $7,688 / 27/07/2026 16:52:01 | ❌ No cumple | Mismos puntos de emisión vacíos (`:5586-5587`, `:5817-5818`). |
| NÚMERO DE OPERACIÓN BANCARIA / NÚMERO DE TRANSACCIÓN SAT | 00000620883170 / 4002127... | ❌ No cumple | Sin columna en schema; hardcodeado vacío. |
| MEDIO DE PRESENTACIÓN | "Otros medios electronicos (Pago Electronico)" | ❌ No cumple | Sin columna; en `reports.service.ts` solo existe como literal de **datos de muestra**, no como dato real. |
| MEDIO DE RECEPCIÓN/COBRO | "Efectivo (cargo a cuenta)" | ❌ No cumple | Ídem. |
| Código QR | presente (imagen) | ❌ No cumple | No existe generador para pedimento (solo hay QR en recibos/inventario, modelo distinto). |

**Esto es el bloque más débil hoy**: de 10 subcampos, solo patente/aduana/línea de captura tienen dato real; el resto (banco, importes, fechas de pago, medios, código de barras, QR) no están modelados como datos tipados — hoy solo existen como strings de muestra en código de reportes.

## 3. Agente aduanal / mandatario / firma

| Campo | Valor real | Estado | Evidencia |
|---|---|---|---|
| NOMBRE / RFC / CURP del agente (M. YOLANDA LEON MAGAÑA / LEMM660223BC4 / LEMY660223MGTNGL06) | presente | ✅ Cumple | Cascada `MompCustomsAgent` / `Reference.customsPatent→CustomsPatents`. |
| Agencia (CS CARGA FORWARDERS, S.A. DE C.V. / CCF061026E66) | presente | ✅ Cumple | `Company` vía `brokerCompany`. |
| PATENTE O AUTORIZACIÓN | 3739 | ✅ Cumple | `Reference.customsPatentId → CustomsPatents.code`. |
| NÚMERO DE SERIE DEL CERTIFICADO | 00001000000713087546 | ⚠️ Cumple parcial | `PatentDigitalSeal.certificateSerial` existe con el dato, pero el builder emite `''` en vez de leerlo. |
| e.firma (blob de firma) | presente | ⚠️ Cumple parcial | Cadena real hasta `PatentDigitalSeal`/`CompanyPatentSignatureConfig`, pero el builder emite `'BORRADOR'` fijo. |
| MANDATARIO/PERSONA AUTORIZADA (nombre/RFC/CURP) | vacío en este pedimento | ✅ No aplica | No se usó en esta operación; el modelo (`CustomsAgent`) existe y funciona cuando aplica. |

## 4. Proveedor, facturas y COVE (página 2)

| Campo | Valor real | Estado | Evidencia |
|---|---|---|---|
| ID. FISCAL / NOMBRE proveedor (TE CONNECTIVITY SOLUTIONS GMBH, CHE116347444MWST) | presente | ✅ Cumple | `Invoice.supplierId → Company`. |
| DOMICILIO proveedor | Muehlenstrasse... Schaffhausen, CHE | ⚠️ Cumple parcial | Modelo completo (`supplierAddressId → Address`), builder emite `''`. |
| 4 facturas (No., fecha, INCOTERM FCA, moneda USD, val. mon. fact., factor 1.0, val. dólares, VINC. SI) | presente x4 | ✅ Cumple | `PedimentoInvoice` se escribe realmente en `operations.service.ts:1404`. |
| COVE por factura (COVE2687SQGW3, etc.) | presente x4 | ⚠️ Cumple parcial | `Invoice.cove`/`PedimentoInvoice.coveEdocument` existen, builder emite `cove: ''`. |
| EXPORTADOR distinto del proveedor (TYCO ELECTRONICS CZECH S.R.O., triangulación) | presente (dentro de Observaciones) | ❌ No cumple como campo estructurado | **Hallazgo nuevo**: `Invoice` solo tiene `supplierId`; no existe `exporterId`/`legalExporter` en `Invoice` ni `Reference`. Lo más cercano es `Shipment.shipperId` (rol logístico, no exportador legal). Hoy esto solo podría capturarse como texto libre dentro de observaciones — que es, de hecho, cómo aparece impreso en el pedimento real (el Anexo 22 no tiene bloque estructurado "exportador" cuando difiere del proveedor; se declara en observaciones). **No es un gap bloqueante**, pero confirma que el dato no viaja como relación, solo como texto. |

## 5. Guías (página 2)

| Campo | Valor real | Estado | Evidencia |
|---|---|---|---|
| GUÍA House 7954998763 / Master 783-50184643 | presente | ⚠️ Cumple parcial | `Guia` es un modelo completo y enganchado a la cadena (`Invoice.guiaId`, `Guia.dgoId`, `Guia.referenceId`), pero `buildHbsFromPedimento` **no emite** el bloque de guías en el documento impreso. Gap de mapeo puro. |

## 6. Identificadores a nivel pedimento (página 2)

| Campo | Valor real | Estado | Evidencia |
|---|---|---|---|
| CR 210, IM 54162006, IC O, MS 1, y **5 identificadores con la misma clave "ED"** (0170261EB0CA3, 0438261HO29C2, 0192261BCTFE3, 0192261BCTHG3, 0192261BCTGC2) | presente | ✅ Cumple | Verificado directamente: `GlobalIdentifier`/`OperationIdentifier`/`ReferenceIdentifier` **no tienen** `@@unique` sobre `[referenceId, code]` — solo índices no únicos. Los 5 "ED" caben sin conflicto. El único `@@unique` real es en el catálogo (`IdentifierCatalog @@unique([code, level])`), que es correcto (1 definición, N aplicaciones). |

## 7. Observaciones (página 2)

| Campo | Valor real | Estado | Evidencia |
|---|---|---|---|
| Texto libre (art. 108 fracc. I LA, arts. 36/36A/65 + regla 3.1.8 RGCE, + bloque exportador) | presente | ✅ Cumple | `buildHbsFromPedimento` emite `observaciones_generales: p.notes \|\| fo.notes`; es texto libre, cabe completo incluyendo el bloque de exportador manual. |

## 8. Partidas (páginas 2-3, 4 partidas idénticas en estructura)

| Campo | Valor real (partida 1 de ejemplo) | Estado | Evidencia |
|---|---|---|---|
| FRACCIÓN / SUBD / VINC / MET.VAL / UMC / CANT. UMC / UMT / CANT. UMT / P.V/C / P.O/D | 85389001 / 00 / 1 / 1 / 6 / 3,000.000 / 6 / 3,000.00000 / CHE / HUN | ✅ Cumple | Todo desde `InvoiceItem`, confirmado en el análisis previo. Nota: P.O/D = HUN (Hungría) ≠ país del proveedor (Suiza) — otra señal de triangulación, pero es un campo normal de país de origen por partida, sin problema. |
| DESCRIPCIÓN | PARTE MOLDEADA PARA CONECTOR | ✅ Cumple | `InvoiceItem.pedimentDescription`. |
| VAL.ADU/USD / IMP.PRECIO PAG. / PRECIO UNIT. | 11,018.00 / 6,667 / 2.22233 | ✅ Cumple | `InvoiceItem.totalValue`/`unitValue`. |
| VAL. AGREG. / VAL. DE RETORNOS | ambos vacíos | ✅ Cumple (no aplica en este caso) | **Hallazgo nuevo**: `VAL. DE RETORNOS` **sí existe** en schema — `InvoiceItem.valorRetornos` (Decimal?) — y se mapea en `operation-dispatch.service.ts:3330`. No estaba en la auditoría genérica de 189 campos porque es un campo adicional de este formato de pedimento. Ojo: aunque el modelo existe, el payload de impresión **no emite** `val_de_retornos` (sí emite `val_agreg: ''` fijo) — en este pedimento no importa porque ambos van vacíos, pero quedaría igual de vacío aunque tuviera dato. |
| MARCA / MODELO / CÓDIGO PRODUCTO | vacíos | ✅ No aplica | Sin dato en este pedimento; el modelo soporta los 3 campos cuando se necesiten. |
| CON. IVA (tt=1, fp=0, importe 1,781) | presente | ✅ Cumple | `OperationItemTax` se escribe realmente. |
| CON. IGI/IGE (tt=1, **fp=5**, importe 0) | presente | ❌ No cumple (parcial) | **Hallazgo nuevo / blocker de schema menor**: `OperationItemTax` **no tiene columna `paymentForm`** (a diferencia de `OperationTax`, que sí la tiene). El `fp=5` (forma de pago "Garantía", típico de IGI en regímenes temporales como ITE) **no se puede persistir a nivel partida** hoy. Además, "IGI/IGE" como concepto combinado solo existe como string de impresión (`reports.service.ts:974,1257`), no como código de catálogo formal — no bloquea, pero es deuda técnica. |
| OBSERVACIONES A NIVEL PARTIDA (texto "Fact(Item) 2881864841(1)(N.P. 1-2289319-1)(DESC...)(CANT. 3000.000 Pza)") | presente x4 | ✅ Cumple — **corrección al análisis previo** | El análisis genérico anterior marcó esto como "repite datos ya declarados, prohibido por Anexo 22". Viendo el pedimento REAL, este texto sintético generado por el sistema (factura+ítem+número de parte+descripción+cantidad) **es exactamente el formato que SAT acepta e imprime** en pedimentos reales — no es un bug, es el comportamiento correcto para este tipo de observación. Retiro esa alerta. |

## 9. Fin de pedimento

| Campo | Valor real | Estado | Evidencia |
|---|---|---|---|
| NUM. TOTAL DE PARTIDAS | 4 | ✅ Cumple | `COUNT` de partidas renderizadas. |
| CLAVE PREVALIDADOR | 010 | ❌ No cumple | Confirmado: no existe en el schema; el builder emite `clave_prevalidador: ''`. |

---

## Veredicto: ¿podemos reproducir ESTE pedimento hoy?

**No completo, pero el núcleo operativo (encabezado, tasas, cuadro de liquidación, partidas, identificadores) sí funciona.** Los bloqueos reales para imprimir este documento tal cual son:

### Bloqueantes duros (sin esto, el documento impreso queda incompleto o inválido)
1. **NÚM. PEDIMENTO no se persiste** — se calcula pero se descarta; sin esto no hay número real que imprimir.
2. **Código de aceptación no se genera/persiste** — depende de integración con el validador externo (SAAI), no es solo un gap interno.
3. **Bloque de pago electrónico** (banco, importe pagado, fecha/hora de pago, núm. operación bancaria, núm. transacción SAT, medio de presentación, medio de recepción/cobro, código de barras, QR) — **9 de 10 subcampos sin dato tipado**, todo hardcodeado vacío o solo como sample data.
4. **Clave del prevalidador** — no existe en el schema.

### Gaps de mapeo (el dato YA existe en la cadena, solo falta conectarlo al render — arreglo rápido, sin tocar schema)
5. Domicilio del importador y del proveedor.
6. COVE por factura.
7. Bloque de guías (Guía completo, nunca emitido).
8. Número de serie del certificado y e.firma real (en vez de `'BORRADOR'`).

### Gap de schema menor (requiere migración, pero acotado)
9. `OperationItemTax` necesita columna `paymentForm` para soportar formas de pago distintas por contribución a nivel partida (caso real: IGI con FP=5 en regímenes temporales como ITE).

### No es gap, es diseño correcto del sistema (confirmado con este ejemplo real)
- Régimen ITE, identificadores repetidos con la misma clave, y el texto sintético de observaciones a nivel partida **ya funcionan como deberían** — el análisis genérico anterior marcaba incorrectamente el último como un riesgo.

### Confirmado que no aplica en este caso pero sigue siendo un gap conceptual
- **Exportador distinto del proveedor** (triangulación): no hay campo relacional (`exporterId`); solo se puede declarar como texto en observaciones. Funciona para este pedimento (porque así se imprime en la realidad), pero significa que el sistema no puede *reportar* o *validar* quién es el exportador legal más que como texto libre.
