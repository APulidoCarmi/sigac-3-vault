# Tareas — sigac-3

Nota central de tareas del cliente (modo sin Jira). Gestionada por las
skills /hoy y /reunion: las tareas nuevas entran como `- [ ]` con
wikilink a la reunión de origen; al completarse se marcan `- [x]`.

- [ ] Implementar límite de visualización en bandeja de entrada (máx 3-5 referencias/movimientos top) ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; nota: medida temporal hasta validar Excel con Enrique)
- [x] Validar bandeja de entrada con Enrique en pantalla ([[2026-07-14 - Prueba operación real desde ticket a facturación]])
- [x] Definir búsquedas y visualización del módulo de Pedimentos en dashboard ([[2026-07-14 - Prueba operación real desde ticket a facturación]])
- [ ] Implementar movimientos dinámicos por tipo de tráfico ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; terrestre: entrada, subdivisión, salida, previo; aéreo/marítimo: otros específicos)
- [ ] Implementar consolidación de DGO (unificar Factura + Packing List + Mercancía + Config. Pedimento) ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; nota: 1 DGO = 1 Pedimento, actúa como source of truth)
- [ ] Implementar generación de operaciones desde DGOs (no desde movimientos) ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; nota: cambio fundamental en lógica)
- [ ] Implementar nomenclatura dinámica (Recinto/Previo) por tráfico ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; terrestre: "Verificación", marítimo/aéreo: "Previo")
- [ ] Implementar pestaña Tickets en detalle de referencia (reemplazar Instrucciones) ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; permitir crear o vincular ticket existente)
- [ ] Implementar checklist automático en Expediente Aduanero ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; basado en perfiles: cliente, número de parte, aduana, agente)
- [ ] Implementar comparación trilateral en Expediente Aduanero ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; PDF original vs datos extraídos vs datos glosados, Anexo 22)
- [ ] Implementar validación mejorada ETA + fecha de arribo ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; no solo ETA, sino validar arribo real y proceso posterior)
- [ ] Reordenar pestañas en detalle de referencia ([[2026-07-13 - Revisión de pantallas (bandeja entrada, detalle referencia)]])
- [ ] Mover botón "Crear operación" fuera de DGO individual ([[2026-07-13 - Revisión de pantallas (operaciones, recinto, citas)]])
- [ ] Migrar locations (recintos) a interfaz dinámicos por tráfico ([[2026-07-13 - Revisión de pantallas (operaciones, recinto, citas)]]; nota: migrar desde SIGAC 2 — analizar de dónde sacar los datos para la migración)
- [ ] Definir flujo de citas con ejecutivos y almacén ([[2026-07-13 - Revisión de pantallas (operaciones, recinto, citas)]]; nota: es NECESARIO tener esta reunión antes de implementar)
