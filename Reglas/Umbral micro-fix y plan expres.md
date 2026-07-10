<!--
  Generado por mi-ide-claude. No editar aquí: la fuente de verdad es
  template/Reglas/ en el repositorio del cascarón, y esta nota se
  re-siembra (sobreescribe) en cada arranque del servidor.
-->

# Regla: umbral micro-fix y plan exprés

Línea nítida para decidir cuánta ceremonia lleva un cambio, sin juicio en
caliente. Cubre los casos 5 y 8 del [[Mapa de flujos del cascaron]].

## Micro-fix (sin plan): SOLO cosmético

Un cambio va sin plan únicamente si es cosmético: typos, textos visibles,
comentarios, formato. **Cualquier cambio de lógica — aunque sea 1 línea —
lleva plan** (exprés si es chico). Es la única defensa contra la
pendiente resbaladiza de "esto es chiquito, lo meto directo".

Aun en micro-fix:

- `/verify` aplica siempre antes de darlo por terminado.
- El `/sync` del día lo menciona en la bitácora.

## Plan exprés (urgencia real)

Cuando hay urgencia real que no admite la entrevista completa de `/plan`,
se usa el modo exprés documentado dentro de la propia skill `/plan`:

- Contexto de 3 líneas, 1-2 pasos, criterio de verificación.
- Mismas reglas duras que un plan normal: persistido en `Planes/` del
  baúl, sin commits durante `/implementa`, `/verify` obligatorio.

La urgencia justifica acortar el diseño, nunca saltárselo.

Origen: plan [[2026-07-06-cobertura-27-casos-flujo]].
