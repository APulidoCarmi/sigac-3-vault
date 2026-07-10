<!--
  Generado por mi-ide-claude. No editar aquí: la fuente de verdad es
  template/Reglas/ en el repositorio del cascarón, y esta nota se
  re-siembra (sobreescribe) en cada arranque del servidor.
-->

# Regla: planes paraguas y replaneo

Cubre los casos 10 y 20 del [[Mapa de flujos del cascaron]]: iniciativas
que no caben en un plan ejecutable, y planes que la realidad invalida a
medias.

## Plan paraguas (caso 10)

- Una iniciativa grande (varias features relacionadas, un refactor por
  etapas) se organiza como un **plan paraguas** en `Planes/`: contexto y
  decisiones de la iniciativa completa, y una lista de sub-planes hijos
  enlazados con wikilinks, cada uno con su estado.
- Cada sub-plan hijo es un plan normal: cabe en sesiones limpias de
  `/implementa` y tiene sus propios criterios de verificación.
- El paraguas no se "implementa": se actualiza al cerrar cada hijo (marcar
  el hijo, anotar decisiones que afecten a los siguientes). `/sync` lo
  trata como cualquier plan activo.

## Replaneo (caso 20)

- Si durante `/implementa` se descubre algo que invalida el plan (una
  decisión equivocada, una incógnita nueva), la regla dura de la skill
  aplica: **detenerse y reportarlo**, nunca improvisar un diseño distinto
  sobre la marcha.
- El replaneo se hace sobre el mismo archivo del plan: se corrige la
  decisión afectada registrando el porqué y la fecha del cambio, y se
  ajustan solo los pasos pendientes — lo ya ejecutado no se reescribe; si
  un paso hecho quedó invalidado, se anota como tal y el ajuste entra
  como paso nuevo.
- Ningún replaneo continúa sin validación humana del plan corregido.

Origen: plan [[2026-07-06-cobertura-27-casos-flujo]].
