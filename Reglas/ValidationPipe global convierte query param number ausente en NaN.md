Al usar `@Query('x') x?: number` en un controller de NestJS con el
`ValidationPipe` global activo (`carmi-odin-api-v2/src/main.ts`), un query
param **ausente** en la URL no llega como `undefined` al servicio — el pipe
igual intenta `Number(value)` sobre el metatipo primitivo `number` y
produce `NaN`.

**Por qué importa**: código que hace `Number(params?.page ?? 1)` para
poner un default NO cubre este caso — `NaN ?? 1` sigue siendo `NaN` (el
`??` solo trata `null`/`undefined` como nulos, no `NaN`). Encontrado en
`GET /invoices` sin `page`/`limit` explícitos: producía `skip: NaN` y
Prisma tronaba con `Argument take is missing` (500).

**Cómo aplicarlo**: usar chequeo de truthiness antes de `Number()`, no
nullish coalescing:

```ts
// mal (vulnerable a NaN):
const page = Math.max(1, Number(params?.page ?? 1));

// bien (NaN es falsy, cae al default):
const page = params?.page ? Math.max(1, Number(params.page)) : 1;
```

`GuiasService.findAll` (`carmi-odin-api-v2/src/guias/services/guias.service.ts`)
ya usaba el patrón correcto por casualidad; `InvoicesService.findAll` no,
hasta que se corrigió el 2026-08-02 (ver [[2026-08-01-facturas-en-guias-sp02-front-formulario-facturas]]).

**Al tocar cualquier endpoint paginado nuevo o existente**: grep por
`params?.page ?? ` / `params?.limit ?? ` en `carmi-odin-api-v2/src/**/*.service.ts`
antes de asumir que el default funciona sin parámetros — puede haber más
casos del mismo bug sin descubrir todavía.
