<!--
  Generado por mi-ide-claude. No editar aquí: la fuente de verdad es
  template/Reglas/ en el repositorio del cascarón, y esta nota se
  re-siembra (sobreescribe) en cada arranque del servidor.
-->

# Regla: spikes e investigación

Cubre el caso 6 del [[Mapa de flujos del cascaron]]: trabajo cuyo
objetivo es **aprender algo**, no entregar código (evaluar una librería,
entender un sistema ajeno, medir viabilidad).

- Un spike se declara al empezar: pregunta a responder y tiempo acotado.
  No requiere `/plan` porque no produce código de producción.
- El entregable es una **nota en el baúl** (en `Sesiones/` como parte de
  la bitácora, o en `Arquitectura/` si la conclusión es duradera): qué se
  probó, qué se concluyó y con qué evidencia.
- El código exploratorio del spike es desechable: no se commitea al repo
  del cliente ni se convierte en producción "porque ya está escrito".
- Si la conclusión es "hay que construirlo", se abre `/plan` con la nota
  del spike como insumo del Contexto — el diseño empieza ahí, no en el
  prototipo.

Origen: plan [[2026-07-06-cobertura-27-casos-flujo]].
