# Sheets no-modales anidados y `onInteractOutside`

En `carmi-digital`, cuando un Sheet (Radix Dialog vía shadcn) se abre **encima** de otro
Sheet ya montado — patrón usado en `ReferenceDGOTab.tsx` (`DgoActionsDrawer` → "Acciones
— DGO #N") con `OperationProformaDrawer.tsx`, `InvoiceFormModal.tsx`, etc. — ambos deben
declararse `modal={false}` y el hijo con `hideOverlay` (para no sumar dos overlays
oscuros). Hasta aquí es el patrón ya documentado en el código (sesión 2026-07-28).

**Lo que faltaba y causó el bug (encontrado en la sesión 2026-07-31,
[[2026-07-30-documento-pedimento-preview-desde-dgo]]):** el contenido de cada Sheet se
renderiza vía `Portal`, así que en el DOM real el `SheetContent` del hijo es un
**hermano** de body, no un descendiente del `SheetContent` del padre — aunque en el árbol
React sí sea hijo. El fix original (sesión 2026-07-28) solo agregó
`onInteractOutside={(e) => e.preventDefault()}` + `onOpenAutoFocus preventDefault` en el
**hijo**, lo cual evita que el hijo se cierre a sí mismo y evita que el foco "se escape"
al abrir. Pero el **padre** seguía sin `onInteractOutside` prevenido: cualquier click
dentro del hijo (cambiar de tab, pulsar un botón) generaba un evento que el padre
detectaba como "fuera" de su propio contenido, cerrándose — y como el padre suele estar
condicionado a un estado (`{actionsDgo && <DgoActionsDrawer ... />}`), cerrarse significa
desmontarse, arrastrando al hijo con él.

## Regla

Todo `SheetContent` que pueda tener un Sheet hijo no-modal abierto encima **también**
necesita `onInteractOutside={(e) => e.preventDefault()}` — no solo el hijo. El botón
"Close" (X) interno y (si está cableado) Escape siguen cerrando el Sheet con normalidad,
ya que no pasan por `onInteractOutside`.

**Por qué:** un fix que solo cubre el momento de *apertura* del hijo (evitar el cierre en
cascada al hacer click en "Ver Proforma"/similar) no cubre las interacciones
*posteriores* dentro del hijo ya abierto. Una verificación que solo confirma "el Sheet
abre" puede pasar en verde mientras cualquier click posterior rompe el flujo.

**Cómo aplicar:** antes de dar por buena una verificación Playwright de un Sheet anidado,
probar explícitamente interactuar con contenido *dentro* del hijo (cambiar de tab, click
en un botón, no solo abrir) y confirmar que el padre sigue montado.

Aplicado en esta sesión a `ReferenceDGOTab.tsx` (`DgoActionsDrawer`). Si aparece un nuevo
Sheet anidado (o un tercer nivel), revisar si el mismo patrón hace falta en cada nivel del
stack.
