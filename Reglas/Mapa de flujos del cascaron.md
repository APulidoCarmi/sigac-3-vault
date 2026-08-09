<!--
  Generado por mi-ide-claude. No editar aquí: la fuente de verdad es
  template/Reglas/ en el repositorio del cascarón, y esta nota se
  re-siembra (sobreescribe) en cada arranque del servidor.
-->

# Mapa de flujos del cascarón

Índice de los 27 casos de trabajo que el cascarón puede enfrentar,
agrupados en 6 familias. Cada caso apunta a su dueño: la skill,
directriz, regla o playbook que lo cubre. Si aparece un caso nuevo que no
encaja aquí, se añade a este mapa (editando `template/Reglas/` en el
repo del cascarón) antes de improvisar una solución.

## A. Origen del requerimiento

1. **Nace en una reunión** → `/reunion` depura la transcripción y da de
   alta las tareas aprobadas (Jira o `Tareas.md` según el modo).
2. **Pedido directo del usuario en sesión** → `/plan` (entrevista de
   diseño) y después `/implementa`.
3. **Trazabilidad del origen** → `/plan` registra en Contexto de dónde
   salió el requerimiento: meet, ticket externo o pedido directo.
4. **Ticket externo ya existente** → playbook
   [[Adopcion de tickets externos]].

## B. Tamaño del trabajo

5. **Micro-fix cosmético** → [[Umbral micro-fix y plan expres]]: solo
   typos, textos visibles, comentarios y formato van sin plan.
6. **Spike / investigación** → [[Spikes e investigacion]]: tiempo
   acotado, resultado en nota, sin código de producción.
7. **Feature normal** → flujo completo `/plan` → `/implementa` →
   `/verify` (directrices b y c).
8. **Urgencia real** → plan exprés, documentado en `/plan` y en
   [[Umbral micro-fix y plan expres]].
9. **Refactor grande multi-sesión** → plan normal con pasos ejecutables
   uno por sesión limpia; si crece a varios planes, ver
   [[Planes paraguas y replaneo]].
10. **Iniciativa paraguas (varios planes)** →
    [[Planes paraguas y replaneo]].

## C. Modo del cliente

11. **Cliente con Jira** → `jira_project` en `config/clientes.json`;
    `/hoy`, `/reunion` y `/tickets` operan contra ese proyecto.
12. **Cliente sin Jira (modo notas)** → tareas en la nota central
    `Tareas.md` del baúl, como `- [ ]` con wikilink de origen.
13. **Onboarding de cliente nuevo** → checklist del README del cascarón:
    alta en `clientes.json` → `setup-baul.sh` → arranque del server →
    verificación (estructura, reglas sembradas, directrices, grafo).
14. **Migración de modo notas a Jira (o inversa)** → playbook
    [[Migracion de modo notas a Jira]].

## D. Máquina y entorno

15. **Máquina nueva** → `./setup.sh` (bootstrap idempotente: Node,
    graphify, MCPs, config local).
16. **MCP registrado a scope local** →
    [[MCPs de Claude Code a scope user]]; `setup.sh` lo detecta y avisa
    con los comandos de migración.
17. **Varios clientes en la misma máquina** →
    [[Disciplina multi-cliente y sincronizacion del baul]].
18. **El cliente tiene skills propias** → los 8 nombres de skills del
    cascarón son reservados (declarados en el README); las skills del
    cliente usan otros nombres y el aprovisionamiento no las toca.

## E. Ciclo de vida de la sesión

19. **Arranque del día** → `/hoy` lista las tareas vigentes y trae el
    contexto de origen de la elegida.
20. **El plan se invalida a medias** → replaneo según
    [[Planes paraguas y replaneo]]: detenerse, actualizar el plan,
    validación humana antes de seguir.
21. **Contexto alto o cambio de tema** → directriz e): persistir en baúl
    o plan y `/compact` o `/clear` (la statusline lleva la cuenta).
22. **Cierre de sesión** → `/sync` consolida la sesión en el baúl y
    refresca el grafo.
23. **El baúl alterna o migra de máquina** →
    [[Disciplina multi-cliente y sincronizacion del baul]]: `/hoy` y
    `/sync` hacen `git pull --ff-only` antes de leer/escribir.

## F. Fallos

24. **Lint/tests fallan al cerrar** → `/verify` es puerta obligatoria:
    se corrige antes de dar por terminado, sin excepciones.
25. **Transcripción de reunión inservible** →
    [[Transcripciones defectuosas]] y la regla correspondiente en
    `/reunion`.
26. **`/tickets` falla a media creación** → `/tickets` escribe cada ID
    en el plan inmediatamente después de crear cada issue: lo ya creado
    queda trazado y la re-ejecución no duplica.
27. **La historia del baúl divergió entre máquinas** → `/hoy` y `/sync`
    se detienen y lo reportan para resolución humana
    ([[Disciplina multi-cliente y sincronizacion del baul]]).

Origen: inventario de la sesión
[[2026-07-06 - Implementacion plan MCP Jira y skill tickets]], plan
[[2026-07-06-cobertura-27-casos-flujo]].
