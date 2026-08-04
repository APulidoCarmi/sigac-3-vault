# `AccordionTrigger` compartido envuelve todo su `children` en un `<button>`

En `carmi-digital`, el `AccordionTrigger` de `components/ui/accordion.tsx` es un wrapper
de conveniencia sobre `AccordionPrimitive.Header` + `AccordionPrimitive.Trigger` — **todo**
lo que se le pase como `children` termina dentro del `<button>` real de Radix
(`AccordionPrimitive.Trigger` renderiza un `<button>`).

**Encontrado en la sesión 2026-08-03**
([[2026-08-02-facturas-en-guias-sp05-auto-vinculo-dgo]]): `InvoiceMerchandiseAccordion.tsx`
pasaba botones de acción ("Agregar Partida", editar factura) como children del
`AccordionTrigger`, junto al label/badges. Resultado: `<button>` anidado dentro de otro
`<button>` — HTML inválido, Next.js lo marca como error de hidratación ("`<button>` cannot
contain a nested `<button>`").

## Regla

Si un header de acordeón necesita **botones de acción** además del label/badges
(togglear el acordeón no debe disparar la acción del botón), **no** usar el
`AccordionTrigger` compartido con esos botones como children. En su lugar, importar
`AccordionPrimitive` directo (`import * as AccordionPrimitive from
"@radix-ui/react-accordion"`) y construir el header a mano:

```tsx
<AccordionPrimitive.Header className="flex items-center ...">
  <AccordionPrimitive.Trigger className="flex flex-1 ...">
    {/* solo label/badges/chevron — esto sí es el <button> real */}
  </AccordionPrimitive.Trigger>
  <div className="flex shrink-0 items-center gap-1">
    {/* botones de acción, hermanos del Trigger, NO sus hijos */}
  </div>
</AccordionPrimitive.Header>
```

Los botones de acción, al quedar fuera del `<button>` del trigger, ya no necesitan
`e.stopPropagation()` en su `onClick` (no hay bubbling hacia el toggle del acordeón).

**Cómo detectarlo:** el error de hidratación de React/Next solo aparece en la consola del
navegador (no lo marca `tsc`/ESLint) — parte de la verificación Playwright de cualquier
acordeón con botones de acción en su header debe incluir revisar
`browser_console_messages` en busca de "cannot contain a nested".
