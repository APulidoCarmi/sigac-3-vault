# useEffect dependencies: índices vs identidades

**Regla**: Cuando un useEffect depende de un valor que es un índice o contador (ej. `activePedimentoGroupIndex`), agregar también una propiedad identificadora única del objeto en cuestión (ej. `activeGroup?.id`) para garantizar que el efecto se re-ejecute cuando la entidad cambia de identidad, no solo cuando el índice cambia.

**Por qué**: En componentes que montan de forma eager (antes de que los datos reales se establezcan), un índice puede quedar en su valor "inicial" incluso tras cargarse datos nuevos. Si el índice no cambia en el caso normal (ej. 1 solo grupo → siempre índice 0), el useEffect nunca se re-ejecutará post-mount, quedando congelado en el estado inicial.

**Cuándo aplica**:
- El componente monta antes de que los datos reales se resuelvan (wizard con tabs montados de entrada, lazy lists, etc.)
- El efecto lee o modifica estado derivado de esos datos (prefill de forms, cálculos, etc.)
- El índice por sí solo no captura cambios de entidad (caso normal: 1 grupo → índice 0 siempre)

**Cómo aplicar**: En el array de dependencias, incluye tanto el índice como `entity?.id` (o la propiedad única que identifique la entidad).

```tsx
// ❌ Incorrecto: efecto congelado si activePedimentoGroupIndex never changes
useEffect(() => {
  setValue("regimeId", customsData.regimeId);
}, [activePedimentoGroupIndex]);

// ✅ Correcto: efecto re-ejecuta cuando el grupo cambia de identidad
useEffect(() => {
  setValue("regimeId", customsData.regimeId);
}, [activePedimentoGroupIndex, activeGroup?.id]);
```

**Ejemplo real**: [[2026-07-23 - SP-01 paso 10 fix regimeId preselection]]

---

**Descubierto**: 2026-07-23 durante verificación de paso 10 (operation-creation wizard, step 4, tab "Aduanas").  
**Bug encontrado**: El selector de régimen aduanero no mostraba preseleccionado el regimeId del DGO, pese a que el store tenía el valor. Root cause: useEffect que siembra el form dependía solo de activePedimentoGroupIndex (nunca cambia en caso normal de 1 grupo).  
**Fix**: Agregar activeGroup?.id a las deps → efecto ahora se re-ejecuta cuando initializeGroupsFromDgos() crea el grupo.
