# `$transaction([...])` en forma de arreglo con modelos auditados rompe Prisma

En `carmi-odin-api-v2`, `AuditedPrismaService` (el cliente Prisma que se inyecta por
defecto) envuelve **automáticamente** cualquier escritura (`create`/`update`/`updateMany`/
etc.) a un modelo con trigger de auditoría (`AUDITED_MODELS`, poblado consultando
`pg_trigger` al boot) en `runAudited(...)` **cuando se invoca fuera de un tx activo** — ver
`src/prisma/audited-prisma.service.ts`.

`runAudited` devuelve una `Promise` normal (ya ejecutándose), no el `PrismaPromise`
perezoso que `$transaction([...])` (forma de arreglo) exige de **todos** sus elementos.
Si algún elemento del arreglo es una llamada a un modelo auditado hecha fuera de un
`$transaction(fn)`/`runAudited(fn)` callback, Prisma rechaza el arreglo **completo** con:

```
Error: All elements of the array need to be Prisma Client promises. Hint: Please make
sure you are not awaiting the Prisma client calls you intended to pass in the
$transaction function.
```

**Encontrado en la sesión 2026-08-03**
([[2026-08-02-facturas-en-guias-sp05-auto-vinculo-dgo]]): `GuiasService
.linkGuiasToReference` pasaba `this.prisma.invoice.updateMany(...)` (Invoice es
auditado) como uno de los elementos de un `$transaction([...])` en forma de arreglo. Bug
preexistente (verificado por aislamiento: reproducido con y sin el cambio de esta
sesión) — nadie lo había disparado antes porque el flujo exacto (vincular guías con al
menos un grupo de pedimento) no se había probado E2E recientemente.

**Fix aplicado:** convertir el `$transaction([...])` (arreglo) a la forma callback
`runAudited(this.prisma, async tx => { ...awaits secuenciales sobre tx... })` — mismo
patrón que ya usan `GuiasService.remove()`/`restore()` en el mismo archivo. `tx` (un
`Prisma.TransactionClient`) sí soporta escrituras a modelos auditados sin el problema,
porque `runAudited` ya proyecta el actor/GUC de auditoría en esa transacción.

## Regla

Antes de escribir un `$transaction([...])` en forma de arreglo en este repo, revisar si
alguno de sus elementos escribe a un modelo auditado (`Invoice`, `Reference`,
`Operation`, `PackingList`, etc. — la lista completa está en `pg_trigger` con nombre
`audit_%`, consultable con
`SELECT c.relname FROM pg_trigger t JOIN pg_class c ON c.oid = t.tgrelid WHERE
t.tgname LIKE 'audit\_%' AND NOT t.tgisinternal;`). Si sí, usar la forma callback
`runAudited(this.prisma, async tx => {...})` con awaits secuenciales en vez del arreglo.
