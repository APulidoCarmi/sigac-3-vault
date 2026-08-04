# Plan: Front — Modal "Facturas de la guía" + generalización de InvoiceFormModal (sp02 de facturas en Guías)

> Sub-plan hijo de [[2026-08-01-facturas-en-guias]]. Repo: `carmi-digital`.
> Depende de [[2026-08-01-facturas-en-guias-sp01-backend-guiaid-invoice]]
> (necesita el campo `guiaId` y el endpoint de listado reales, no se mockea).

## Contexto

Ver paraguas [[2026-08-01-facturas-en-guias]]. Investigación de código ya
hecha (sesión 2026-08-01): módulo de Guías en
`app/(customerPortal)/guias/` (tabla + modales:
`GuiaFormModal.tsx`, `BulkAssignGuiasModal.tsx`,
`GuiaStatusModal.tsx`/`BulkGuiaStatusModal.tsx`,
`CreateReferencesFromGuiasModal.tsx`). Formulario de factura a reutilizar:
`app/(customerPortal)/references/components/invoices/InvoiceFormModal.tsx`
(1668 líneas), con `InvoiceFormModalProps` en líneas 196-224 y
`buildInvoiceDto` en líneas 682-829. Componente de documento reutilizable
sin cambios: `InvoiceImageDrawer.tsx` (sube como base64 embebido, agnóstico
de la entidad padre).

## Decisiones tomadas

1. Acción nueva por fila en `guias/page.tsx` (no dentro de
   `GuiaFormModal`), que abre un modal nuevo "Facturas de la guía".
2. El modal nuevo lista las facturas ya vinculadas a esa guía (múltiples,
   v1 con soporte de llegadas parciales) y tiene botón "Agregar factura"
   que abre `InvoiceFormModal` en un modo nuevo asociado a `guiaId`.
3. Se reutiliza `InvoiceFormModal` completo (mismos campos: partidas,
   montos, incoterm, vinculación, valoración) — no se recorta a un
   subconjunto.
4. `InvoiceFormModalProps.referenceId` pasa de obligatorio a opcional;
   se agrega `guiaId?: string`. Debe venir exactamente uno de
   `referenceId` o `guiaId` al abrir el modal desde cada contexto (DGO/
   Referencia vs. Guías).
5. Sin restricción de rol nueva para esta acción.

## Fuera de alcance

- Igual que el paraguas: tareas 2-5 de la reunión de origen, matching Zeus,
  validación explícita para tráfico marítimo.
- Cualquier cambio al wizard de creación de Reference o a
  `CreateReferencesFromGuiasModal` más allá de lo que ya haga el sub-plan
  01 en backend (la migración automática es transparente para el front).

## Pasos

- [x] Crear el hook `use-guia-invoices.ts` (o extender `use-guias.ts`) con
      `useGuiaInvoices(guiaId)` (GET del endpoint nuevo del sub-plan 01) y
      reusar el patrón de mutation existente para crear/editar factura,
      apuntando al mismo endpoint `POST/PATCH {API_ODIN}/invoices` pero
      con `guiaId` en el DTO.
- [x] Generalizar `InvoiceFormModalProps` (líneas 196-224): `referenceId`
      opcional, agregar `guiaId?: string`. Ajustar los tipos y cualquier
      uso interno que asuma `referenceId` siempre presente (queries,
      keys de react-query, invalidación de caché) — auditar antes de
      tocar, es un componente de 1668 líneas usado hoy en producción por
      DGO/Referencia; no romper ese flujo.
- [x] Ajustar `buildInvoiceDto` (líneas 682-829) para incluir `guiaId`
      cuando esté presente y omitir `referenceId` cuando no lo esté (mismo
      patrón condicional que ya usa para `dgoId`, línea 785).
- [x] Crear el componente `GuiaInvoicesModal.tsx` (o nombre equivalente,
      siguiendo convención de `GuiaStatusModal.tsx`): lista de facturas de
      la guía (usa `useGuiaInvoices`), botón "Agregar factura" que abre
      `InvoiceFormModal` con `guiaId` y sin `referenceId`, y permite editar
      una factura existente de la lista (modo `edit`, igual que hoy).
- [x] Agregar la acción/ícono nuevo en la fila de la tabla de
      `guias/page.tsx` que abre `GuiaInvoicesModal`, siguiendo el patrón
      visual de las acciones existentes (status, bulk assign, etc.).
- [x] Verificar que `InvoiceImageDrawer.tsx` no requiere cambios (ya es
      agnóstico de la entidad padre) — confirmar en la prueba manual que
      subir/comprimir/adjuntar imagen funciona igual dentro de este nuevo
      flujo.

## Riesgos y side effects a vigilar

- `InvoiceFormModal` es el componente más grande y compartido del flujo de
  facturación (usado por `ReferenceDGOTab.tsx`, `ReferenceMerchandiseTab.tsx`,
  `OperationProformaDrawer.tsx`); cualquier cambio a sus props/lógica de
  guardado debe probarse también en esos tres puntos de uso existentes, no
  solo en el nuevo flujo de Guías.
- Mientras la tarea 2/3 (movimiento/expediente por guía) no exista, la
  factura subida desde este modal no aparecerá en Expediente Aduanero —
  confirmar que el modal deja claro (copy/UX) que es solo el registro de
  factura, no el expediente completo, para no generar expectativa
  equivocada en el usuario final.

## Criterios de verificación

- `/verify`: lint/typecheck del front.
- Flujo manual con Playwright MCP (revisando consola del navegador):
  1. Abrir módulo de Guías, seleccionar una guía sin Referencia.
  2. Abrir "Facturas de la guía" (lista vacía), agregar factura con
     documento + datos completos.
  3. Confirmar que aparece en la lista y que el documento se ve
     correctamente (thumbnail/preview vía `InvoiceImageDrawer`).
  4. Editar la factura recién creada, confirmar que los cambios persisten.
  5. Regresión: abrir el flujo normal de DGO/Referencia y confirmar que
     `InvoiceFormModal` sigue funcionando sin cambios de comportamiento
     visibles (crear/editar factura ligada a `dgoId`/`referenceId` como
     hoy).
