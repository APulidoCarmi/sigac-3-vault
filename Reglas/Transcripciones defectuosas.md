<!--
  Generado por mi-ide-claude. No editar aquí: la fuente de verdad es
  template/Reglas/ en el repositorio del cascarón, y esta nota se
  re-siembra (sobreescribe) en cada arranque del servidor.
-->

# Regla: transcripciones defectuosas

Cubre el caso 25 del [[Mapa de flujos del cascaron]]: la transcripción
que llega a `/reunion` es inservible (vacía, cortada a la mitad, idioma
mal detectado, hablantes irreconocibles, puro ruido).

- **Nunca inventar contenido.** Una nota de reunión con contenido
  reconstruido de la imaginación es peor que no tener nota: envenena el
  baúl y el grafo.
- Reportar al usuario qué la hace inservible (vacía, cortada, idioma,
  ruido) y pedir el re-export de la transcripción o las notas manuales
  del asistente.
- Si algo se rescata (por ejemplo, la mitad buena de una transcripción
  cortada): guardar la nota en `Reuniones/` marcada como **"parcial"** en
  el título y con una línea al inicio que diga qué tramo falta y por qué.
- Las tareas solo se dan de alta desde los tramos rescatados y con la
  confirmación humana de siempre; de un tramo perdido no se infiere nada.

Origen: plan [[2026-07-06-cobertura-27-casos-flujo]].
