<!--
  Generado por mi-ide-claude. No editar aquí: la fuente de verdad es
  template/Reglas/ en el repositorio del cascarón, y esta nota se
  re-siembra (sobreescribe) en cada arranque del servidor.
-->

# Regla: disciplina multi-cliente y sincronización del baúl

Cubre los casos 17, 23 y 27 del [[Mapa de flujos del cascaron]]. Escala
declarada del cascarón: 1-3 clientes y 1-2 máquinas — automatización
ligera (checklists y avisos), no orquestación.

## Varios clientes en la misma máquina (caso 17)

- **Una sesión de trabajo = un cliente.** El server inyecta las
  directrices del cliente activo en su `CLAUDE.local.md`; trabajar sobre
  el repo de un cliente con el server levantado para otro mezcla
  contextos y baúles.
- Cambiar de cliente = reiniciar el server y seleccionar el otro cliente;
  antes de hacerlo, cerrar la sesión en curso con `/sync`.
- Contexto de negocio de un cliente jamás se escribe en el baúl de otro.
  Cada baúl es un repositorio Git privado e independiente.

## El baúl alterna o migra de máquina (casos 23 y 27)

- `/hoy` y `/sync` hacen `git pull --ff-only` del baúl **antes** de
  leer/escribir, para operar siempre sobre la última versión.
- Si el pull falla porque la historia divergió (caso 27): **detenerse y
  reportarlo** — la resolución del conflicto es humana, nunca automática.
- Si el baúl no tiene remoto, o no es un repositorio Git: avisarlo en una
  línea y continuar — es el caso normal de un baúl local (como el demo) y
  no debe generar ruido.
- El baúl no es multi-usuario simultáneo: el flujo soportado es
  alternancia/migración de máquina, con `obsidian-git` (o `/sync`)
  empujando al remoto al cerrar cada jornada.

Origen: plan [[2026-07-06-cobertura-27-casos-flujo]].
