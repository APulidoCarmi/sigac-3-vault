<!--
  Generado por mi-ide-claude. No editar aquí: la fuente de verdad es
  template/Reglas/ en el repositorio del cascarón, y esta nota se
  re-siembra (sobreescribe) en cada arranque del servidor.
-->

# Playbook: migración de modo notas a Jira

Cubre el caso 14 del [[Mapa de flujos del cascaron]]: un cliente que
trabajaba en modo notas (`Tareas.md`) adquiere Jira, o al revés.

Qué hacer cuando ocurra (notas → Jira):

1. Configurar `jira_project` para el cliente en `config/clientes.json` y
   reiniciar el server: la directriz f) pasa a declarar el modo Jira y
   `/hoy`, `/reunion` y `/tickets` cambian de destino solos.
2. Migrar las tareas **abiertas** (`- [ ]`) de `Tareas.md` a Jira una a
   una, con confirmación humana por tarea (misma disciplina que
   `/reunion`): crear el issue y anotar en `Tareas.md` la línea como
   migrada con su ID (` — <ID>`), sin borrarla.
3. `Tareas.md` queda como archivo histórico: las tareas cerradas (`- [x]`)
   y las migradas no se tocan; no entran tareas nuevas ahí.

Sentido inverso (Jira → notas): quitar `jira_project`, reiniciar el
server y exportar las tareas abiertas del proyecto a `Tareas.md` como
`- [ ]` con el ID de origen como referencia.

**Deliberadamente no construido:** no hay código de migración —
construirlo sin un cliente real que migre es especulación. Si ocurre,
este playbook es el procedimiento manual; si se repite, se diseña con
`/plan`.

Origen: plan [[2026-07-06-cobertura-27-casos-flujo]] (decisión "todo lo
no especulativo").
