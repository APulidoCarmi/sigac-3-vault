<!--
  Generado por mi-ide-claude. No editar aquí: la fuente de verdad es
  template/Reglas/ en el repositorio del cascarón, y esta nota se
  re-siembra (sobreescribe) en cada arranque del servidor.
-->

# Regla: presupuesto de tokens y sesiones limpias

Cómo trabajar iniciativas grandes sobre monorepos grandes sin quemar el
presupuesto. Complementa a [[Planes paraguas y replaneo]] (que define el
troceo) con la disciplina de ejecución.

## El principio

**Mantén el contexto principal pequeño y desechable.** El contexto de una
sesión no se borra entre turnos: cada turno reenvía todo lo acumulado. Sobre
un monorepo grande, cualquier lectura o `grep` sin acotar se queda en el
contexto el resto de la sesión y encarece cada turno siguiente. El costo
crece de forma **cuadrática** con la longitud de la sesión.

## Regla dura: un sub-plan = una sesión limpia

- Cada sub-plan hijo de un plan paraguas se implementa en **su propia
  sesión** y se cierra con `/sync` + `/clear` antes de abrir el siguiente.
- **NUNCA** encadenes varios sub-planes en la misma sesión ni ejecutes
  "todos los planes de corrido". Es el error más caro posible y anula el
  diseño mismo de los sub-planes (que existen justo para caber en sesiones
  limpias).

## Reglas de exploración

- **Delega la exploración a subagentes** (`Explore` / `Task`). El subagente
  lee muchos archivos, quema *sus* tokens y devuelve solo la conclusión; el
  contexto principal nunca ve los volcados. Usa el hilo principal para
  editar y decidir, no para leer decenas de archivos.
- **graphify primero, acotado.** Orienta con `graphify query` / `explain` /
  `path` (subgrafo pequeño). **No leas `graphify-out/GRAPH_REPORT.md` ni
  `graphify-out/*graph*.json`**: pesan cientos de KB / decenas de MB y
  revientan el presupuesto en una sola lectura. Están bloqueados por
  permisos en `.claude/settings.json` para evitar accidentes.
- **No corras `graphify update` tras cada edición**: re-escanea todo el
  repo. Se hace una sola vez al cerrar el sub-plan (lo hace `/sync`).
- **Apaga los MCP que no uses** en la sesión (Figma, Canva, Playwright
  cargan mucho esquema de herramientas); reactívalos solo cuando el plan
  los pida.

Origen: incidente 2026-07-11 — se corrieron los 18 sub-planes de
[[2026-07-10-refactor-flujo-ejecutivo]] en una sola sesión sin restricción
y se agotó el presupuesto de tokens de un golpe.
