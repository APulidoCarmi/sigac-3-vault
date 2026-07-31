# Auditar código antes de confiar en el texto del plan

En `/implementa`, el texto de un plan (`Planes/*.md`) puede haberse redactado semanas
antes de ejecutarse, o incluso dentro de la misma sesión pero basado en una lectura
parcial del código. No asumir que un archivo/línea/decisión que el plan nombra sigue
siendo correcto — confirmarlo contra el código real antes de tocarlo.

**Por qué:** en [[2026-07-28-unificar-wizard-operacion-dgo-y-proforma-por-dgo]] esto pasó
tres veces en una sola sesión de `/implementa`:
1. La tarea 3 nombraba "el step de agrupar por clave" como `Step2OperationConfig` — ese
   archivo no agrupa nada; la lógica real vivía en otro componente
   (`Step1SourceData` → `MultiShipmentSummary`). Seguir el texto literal habría borrado
   por error el step correcto (TC/fechas/agente) y dejado vivo el que sí debía eliminarse.
2. La tarea 8 pedía adaptar un componente para un endpoint que la tarea 12 (más adelante
   en el mismo plan) todavía no había construido — dependencia cruzada no marcada en el
   texto.
3. La Decisión 7 (auditoría de "todos los entry points" de un componente) no había
   encontrado 2 entry points reales que sí navegaban al flujo viejo.

En los tres casos, grep/lectura directa del código —no el texto del plan— fue lo que
reveló la discrepancia, y en los tres casos la sesión se detuvo a preguntarle al usuario
en vez de decidir unilateralmente cómo resolverla.

**Cómo aplicarlo:**
- Antes de editar un archivo que el plan nombra por su rol ("el step que hace X"),
  leerlo y confirmar que efectivamente hace X.
- Antes de asumir que una lista de "entry points"/"consumidores"/"llamadores" de una
  auditoría previa está completa, correr el grep de verificación de nuevo.
- Si una tarea depende de algo que otra tarea del mismo plan construye más adelante,
  marcarlo y decidir con el usuario si conviene reordenar o dividir el alcance — no
  intentar completar la tarea contra un endpoint/función que aún no existe.
- Al encontrar una discrepancia real (no solo una duda menor), parar y preguntar antes de
  improvisar un diseño distinto — ver también [[Discutir antes de persistir tareas]] para
  el mismo principio aplicado a memoria/tareas.
