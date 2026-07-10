<!--
  Generado por mi-ide-claude. No editar aquí: la fuente de verdad es
  template/Reglas/ en el repositorio del cascarón, y esta nota se
  re-siembra (sobreescribe) en cada arranque del servidor.
-->

# Regla: MCPs de Claude Code se registran a scope user

Cubre el caso 16 del [[Mapa de flujos del cascaron]].

Los MCP del cascarón (Playwright, `jira-atlassian`) se registran SIEMPRE a
scope **user** (`claude mcp add --scope user …`), nunca a scope local:

- El scope local es privado al proyecto y a la máquina — otros
  proyectos/clientes de la misma máquina no lo ven, y da la ilusión de
  "ya está registrado" sin ser portable.
- La detección de `setup.sh` (`claude mcp list | grep`) no distingue
  scopes: un registro local previo hace que el script lo dé por
  registrado y nunca lo migre solo. `setup.sh` detecta este estado y
  avisa; la corrección es manual y explícita:
  `claude mcp remove <nombre> -s local` y re-correr `./setup.sh`.
- Tras migrar de local a user, el estado queda "Needs authentication": el
  OAuth no se hereda entre scopes; hay que re-autenticar con `/mcp` en
  una sesión. Es comportamiento esperado, no un fallo.

Origen: sesión [[2026-07-06 - Implementacion plan MCP Jira y skill tickets]],
plan [[2026-07-06-mcp-jira-setup-y-skill-tickets]].
