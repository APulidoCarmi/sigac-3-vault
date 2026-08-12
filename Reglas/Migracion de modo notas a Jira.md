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
2. Migrar las tareas de `Tareas.md` a Jira **una a una, a discreción del
   usuario** (no todas de golpe) desde el panel de Tareas del IDE: el
   panel muestra sub-pestañas **Pendientes** / **Terminadas** (ambas
   leídas de `Tareas.md`, que sigue siendo la fuente incluso en modo
   Jira) y, junto a cada tarea sin ` — <ID>` todavía, un botón
   "Enviar a Jira". El botón dispara la skill `/tarea-jira` en la
   terminal, que confirma con el usuario antes de crear el issue.
   - Pestaña **Pendientes** → el ticket se crea en la columna inicial
     (To Do / Backlog).
   - Pestaña **Terminadas** → el ticket se crea y además se transiciona
     a Done — a diferencia de la versión anterior de este playbook, las
     tareas cerradas **sí se migran** cuando el usuario lo pide (decisión
     explícita: el histórico de trabajo terminado también debe verse en
     el tablero de Jira del cliente, no solo lo pendiente).
   - Cada migración anota en `Tareas.md` la línea como migrada con su ID
     (` — <ID>`), sin borrarla ni tocar su checkbox — así queda como
     marca de idempotencia y el botón deja de ofrecerse para esa tarea.
3. `Tareas.md` sigue siendo el archivo de referencia local: las tareas
   sin migrar (pendientes o terminadas) quedan disponibles indefinidamente
   para migrarse después, una por una, sin fecha límite; no es necesario
   vaciarlo tras activar Jira.

Sentido inverso (Jira → notas): quitar `jira_project`, reiniciar el
server y exportar las tareas abiertas del proyecto a `Tareas.md` como
`- [ ]` con el ID de origen como referencia. Esta dirección sigue siendo
manual — no hay skill equivalente a `/tarea-jira` para este sentido
porque no ha surgido un caso real que la valide; si ocurre, se diseña con
`/plan` usando este playbook como contexto.

Construido: panel del IDE (`public/js/tareas.js`, endpoint
`GET /api/tareas`) y skill `/tarea-jira` (`.claude/skills/tarea-jira`).

Origen: plan [[2026-07-06-cobertura-27-casos-flujo]] (decisión "todo lo
no especulativo").
