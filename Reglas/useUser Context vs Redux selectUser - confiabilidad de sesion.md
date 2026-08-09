# `useUser()` (Context) vs Redux `selectUser` — confiabilidad de sesión en carmi-digital

En `carmi-digital` conviven dos fuentes de "usuario actual", con
confiabilidad muy distinta:

- **`UserContextProvider`** (`useUser()`, `app/context/UserContextProvider.tsx`):
  extrae el usuario **de forma asíncrona** del token de una cookie en cada
  montaje del provider. Puede quedar `null`/sin resolver aunque la sesión
  siga siendo válida (falla de extracción, timing, etc.).
- **Redux `selectUser`** (`@/store/userSlice`): se llena **una sola vez en
  el login** (`app/login/page.tsx`) y persiste vía `redux-persist`/
  `localStorage` (`persist:root`) — mucho más estable entre navegaciones y
  recargas.

**Encontrado en la sesión 2026-08-05**
([[2026-08-05 - Implementación movimiento-entrada-automatico-guia-house]]):
`SubdivisionForm.tsx` solo usaba `useUser()` y bloqueaba con "No se pudo
identificar al usuario" al intentar subdividir, mientras que en la misma
sesión `GuiaFormModal.tsx` (que ya usa `useSelector(selectUser)`) creaba
guías sin problema — mismo usuario, misma sesión, dos resultados distintos
según la fuente.

## Regla

Cualquier componente nuevo que necesite el `UserID` del usuario actual debe
usar Redux `selectUser` como fuente primaria o como fallback antes de
bloquear con un error — no confiar solo en `useUser()` Context. Patrón
mínimo:

```ts
const { user } = useUser();
const reduxUser = useSelector(selectUser);
const userId = userIdProp || user?.user?.UserID || reduxUser?.UserID || '';
```
