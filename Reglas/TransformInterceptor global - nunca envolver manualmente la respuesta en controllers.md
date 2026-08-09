# `TransformInterceptor` global — nunca envolver manualmente la respuesta en controllers

En `carmi-odin-api-v2`, `core.module.ts` registra `TransformInterceptor` como
`APP_INTERCEPTOR` global: **toda** respuesta de **todo** controller queda
envuelta automáticamente en `{ success, data, timestamp, messageId, traceId }`
(ver `src/common/interceptors/transform.interceptor.ts`).

Si un controller además envuelve manualmente su propio `return` en
`{ success: true, data, timestamp: ... }`, la respuesta queda **doble
envuelta**: `{ success, data: { success, data, timestamp }, ... }`. El front
lee `res.data.data` esperando el array/objeto real, pero recibe ese objeto
interno — rompe en silencio (`items.map is not a function`, tablas vacías,
etc.) sin que el backend registre ningún error.

**Encontrado en la sesión 2026-08-05**
([[2026-08-05 - Implementación movimiento-entrada-automatico-guia-house]]):
`AirManifestController`, `TransportAssignmentController`,
`AirRevalidationController` y `PrevioController` tenían este wrap manual —
nunca se había detectado porque los flujos que los consumen (Movimientos
aéreo) nunca se habían ejercitado de punta a punta antes de este plan.

**Se encontraron otros 14 controllers con el mismo patrón, sin arreglar**
(fuera de alcance del plan, decisión explícita de no ampliar el diff):
`maritime-*`, `forklift`, `device-tokens`, `dispatch`, `task-integrations`,
`reference-portal`, `dgo`, `unidentified-waybills`, `company-contacts`.
Candidato a un sub-plan de limpieza aparte.

## Regla

Un controller de este repo **nunca** debe hacer
`return { success: true, data, timestamp: ... }` — el interceptor global ya
lo hace. El patrón correcto es devolver el dato crudo del service:

```ts
async findByReference(@Param('referenceId') referenceId: string) {
  return await this.service.findByReference(referenceId);
}
```

(ver `ShipmentsController.findByReference` como referencia correcta desde el
principio). Si al tocar un controller nuevo aparece un wrap manual, es señal
de este mismo bug — quitarlo, no solo en el método que se esté tocando.
