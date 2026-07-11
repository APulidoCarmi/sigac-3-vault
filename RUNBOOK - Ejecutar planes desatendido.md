# RUNBOOK — Poner a Claude a trabajar un plan (desatendido)

Guía para dejar a Claude implementando una iniciativa mientras no estás, sin
quemar el presupuesto de tokens. Aplica la disciplina de
[[Presupuesto de tokens y sesiones limpias]] y
[[Planes paraguas y replaneo]].

## Paso 0 — Preparar el terreno (una vez)

```
cd /Users/angelhubertopulidoburgos/Desktop/proyects/carmi/sigac-3
git -C carmi-digital status        # front (rama base: test)
git -C carmi-odin-api-v2 status    # back  (rama base: staging)
```

- Git limpio en front y back (el flujo acumula sin commits; así el diff final
  es solo de esta corrida). Si hay algo pendiente ajeno: `git -C <repo> stash`.

## Paso 1 — Sesión limpia

- Abre Claude Code **en `carmi/sigac-3`** y haz `/clear` (o sesión nueva). El
  hilo principal debe arrancar vacío.

## Paso 2 — Modo sin restricciones

- Enciende auto-accept / bypass (CLI: `--dangerously-skip-permissions`; IDE:
  el toggle). Solo evita que pida permiso mientras no estás; NO afecta el
  gasto de tokens.

## Paso 3 — Lanzar la orquestación

```
/implementa-paraguas 2026-07-10-refactor-flujo-ejecutivo
```

**Quédate para este momento.** Claude mostrará:
- la **rama** `feat/<slug-del-paraguas>` que creará (una sola, compartida) en
  front y back al iniciar — los subagentes NO crean ramas propias;
- el **grafo de niveles** (qué sub-planes van en paralelo y cuáles en
  secuencia por dependencia o por tocar los mismos archivos).

Revisa el orden, **confírmalo**, y ahí sí déjalo trabajando solo.

## Paso 4 — Al volver, revisar

```
git -C carmi-digital branch --show-current
git -C carmi-odin-api-v2 branch --show-current
ls "/Users/angelhubertopulidoburgos/Desktop/proyects/bauls/sigac-3/Planes/.manifiestos/"
git -C carmi-digital diff
git -C carmi-odin-api-v2 diff
```

- Los **manifiestos** (`Planes/.manifiestos/<hijo>.md`) permiten revisar el
  diff grande por partes.
- Cuando lo apruebes, corre `/sync` (refresca el grafo una sola vez y
  consolida en el baúl).

## Qué te protege automáticamente

- **Tokens:** cada sub-plan corre en su propio subagente → el contexto
  principal no crece de forma cuadrática. `GRAPH_REPORT.md` y `*graph*.json`
  están bloqueados por permisos.
- **Ramas:** trabajo aislado en `feat/…` en front y back, sin ensuciar
  `test`/`staging`. La rama se crea al iniciar el plan, una sola vez.
- **Dependencias:** un sub-plan que depende de otro (o toca los mismos
  archivos) espera; no se pisan.
- **Prisma:** si un sub-plan toca `schema.prisma`, la migración se genera con
  `npx prisma migrate dev --name …`, nunca a mano (regla global).

## Avisos

- La confirmación del Paso 3 es el único punto que necesita tu criterio: 30
  segundos que evitan arrancar con un orden malo.
- Si un sub-plan se **bloquea o invalida**, la cadena se detiene ahí y no
  lanza a los que dependían de él. Lo verás en el reporte — es lo correcto.

---
Para un solo plan (no paraguas): mismos pasos, pero usa `/implementa <plan>`.
La rama `feat/<slug>` se crea igual al iniciar.
