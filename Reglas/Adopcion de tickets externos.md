<!--
  Generado por mi-ide-claude. No editar aquí: la fuente de verdad es
  template/Reglas/ en el repositorio del cascarón, y esta nota se
  re-siembra (sobreescribe) en cada arranque del servidor.
-->

# Playbook: adopción de tickets externos

Cubre el caso 4 del [[Mapa de flujos del cascaron]]: el requerimiento no
nace en una reunión ni de un pedido directo, sino que llega como ticket
ya creado en el sistema del cliente (su Jira u otro tracker).

Qué hacer cuando ocurra:

1. El ticket externo es el **origen** del requerimiento: `/plan` lo
   registra en Contexto con su ID y enlace (si el cliente tiene Jira
   configurado, se puede leer el ticket vía el MCP `jira-atlassian` para
   alimentar la entrevista).
2. El plan vive igual que siempre en `Planes/` del baúl; el ticket
   externo no sustituye al plan — es insumo, no diseño.
3. Al cerrar el trabajo, la traza vuelve al ticket externo (comentario,
   transición de estado) según el proceso del cliente; hoy ese paso es
   manual y `/sync` lo deja anotado como pendiente si no se hizo.

**Deliberadamente no construido:** una semilla automática ticket→plan
(precargar la entrevista de `/plan` desde el issue) es especulación sin
un usuario real que la valide. Si el caso se vuelve frecuente con un
cliente real, se diseña entonces con `/plan` usando este playbook como
contexto.

Origen: plan [[2026-07-06-cobertura-27-casos-flujo]] (decisión "todo lo
no especulativo").
