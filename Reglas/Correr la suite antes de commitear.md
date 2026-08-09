# Correr la suite antes de commitear

Si el repo tiene tests (`npm test`, `node --test`, jest…), **córrelos antes de
crear el commit**, no después ni "cuando toque". Si el commit se va a empujar,
con más razón.

## Por qué

El 2026-08-08, en `mi-ide-claude`, el commit `483a2ba` se creó y se empujó a
`origin/main` con la suite en rojo. El test `provisionTemplate: un settings.json
ajeno no se pisa` llevaba roto desde que se escribió `ensureTokenBudgetSettings`
—que fusiona el presupuesto de contexto en el settings del cliente a propósito,
mientras el test exigía que el archivo quedara byte a byte igual—. Nadie corrió
`npm test`; se detectó por casualidad en el commit siguiente, ya publicado.

El costo no es el test roto: es que un rojo publicado deja de ser señal. La
siguiente persona que corra la suite ve un fallo y no sabe si lo rompió ella.

## Cómo aplicarlo

- Antes de `git commit`, corre la suite. Si no hay suite, dilo explícitamente
  en vez de asumir que no hacía falta.
- **Un test en rojo no significa que el código esté mal.** Puede estar
  describiendo un contrato viejo. Lee qué afirma el test antes de decidir de
  qué lado está el error: en el caso de arriba lo correcto era reescribir el
  test contra el contrato nuevo (la config propia del cliente sobrevive, las
  claves del presupuesto se fusionan), no "arreglar" el código.
- Si la suite ya estaba roja antes de tus cambios, dilo al reportar en vez de
  arrastrarla en silencio.
