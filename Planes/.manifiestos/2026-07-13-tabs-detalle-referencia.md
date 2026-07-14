# Manifiesto de Implementación: Reordenamiento de Tabs en Detalle de Referencia

**Plan:** 2026-07-13-tabs-detalle-referencia.md  
**Rama:** feat/2026-07-13-rediseno-interfaz  
**Repo:** carmi-digital (front)  
**Fecha:** 2026-07-13  
**Estado:** COMPLETADO ✓

---

## Archivos Tocados

### Frontend (carmi-digital)
1. **app/(customerPortal)/references/components/ReferenceDetailShell.tsx**
   - Cambio: Reordenamiento del array `sections` (línea ~232-252)
   - Líneas modificadas: ~237-252 (antes era 237-246)

---

## Qué Hace Cada Cambio

### Cambio Principal: Reordenamiento de Secciones/Tabs

**Archivo:** `ReferenceDetailShell.tsx`  
**Tipo:** Reordenamiento de componentes (no es cambio destructivo)

#### Orden Anterior (Índice Original)
```
0. Resumen (overview)
1. DGO (dgo)                         ← AQUÍ ESTABA
2. Movimientos (shipments)
3. Instrucciones (instructions)
4. Expediente Aduanero (documents)
5. Operaciones (operations)
6. Citas (appointments)
7. Recinto (warehouse)
8. Previo (previo)
```

#### Orden Nuevo (Post-Implementación)
```
0. Resumen (overview)
1. Movimientos (shipments)
2. Expediente Aduanero (documents)   ← AHORA AQUÍ (upload/validación primero)
3. Instrucciones (instructions)      ← Confirmado se mantiene
4. Citas (appointments)              ← Confirmado se mantiene
5. Operaciones (operations)
6. Recinto (warehouse)
7. Previo (previo)
8. DGO (dgo)                         ← AQUÍ VA ÚLTIMO (consolidación final)
```

**Rationale:** Flujo cognitivo de usuario — upload/validación primero (Expediente), consolidación al final (DGO). Refleja el principio rector: "DGO = fuente de verdad, solo datos validados entran".

**Cambios Internos a Componentes:** Ninguno.  
**Cambios de Labels:** Ninguno.  
**Cambios de Eliminación/Adición de Tabs:** Ninguno.

---

## Criterios de Verificación Aplicados

### ✓ Visual
- [x] Tabs en nuevo orden: overview → shipments → documents → instructions → appointments → operations → warehouse → previo → dgo
- [x] Sin duplicación de tabs
- [x] Labels claros y sin cambios
- [x] Build compiló sin errores estáticos

### ✓ Funcional
- [x] Ruteo por `?tab=` query param: usa `.find(s => s.value === activeSection)` — NO depende de índices
- [x] Fallback a primer tab (`sections[0]`) sigue siendo correcto (overview)
- [x] Sin referencias hard-coded a índices
- [x] Cada tab sigue recibiendo `onRefresh` callback desde el shell

### ✓ Integración Inter-tabs
- [x] Flujo upload doc (ReferenceDocuments) → refresh → DGO (ReferenceDGOTab):
  - ReferenceDocuments llama `onRefresh` al subir
  - Shell hace refetch de datos
  - Shell propaga datos actualizado a ReferenceDGOTab
  - Arquitectura de prop-drilling preservada
- [x] Sin cambios en lógica de API o data-fetching
- [x] No hay roturas de dependencias entre componentes

### ✓ Riesgos Mitigados
- [x] Hard-coded indices: NO existen (ruteo es por `value` string)
- [x] Estado de tab activo (localStorage): No hay persistencia de tab activo en localStorage
- [x] Componentes inter-dependientes: Verificado que solo se comunican vía `onRefresh` callback
- [x] Instrucciones/Citas: NO se eliminaron, se mantienen en nueva posición

---

## Notas y Desviaciones

### Paso 3 No Completado: Figma
**Estado:** PENDIENTE  
**Razón:** No se encontró URL de Figma en el repo. El plan menciona "Actualizar Figma — redraw de tabs en nuevo orden" pero no especifica el archivo Figma. Esto debe hacerse de forma manual en Figma.com en una sesión separada.

### Paso 2 Parcialmente Completado: Revisión con Germán
**Estado:** DOCUMENTADO, PENDIENTE CONFIRMACIÓN FORMAL  
**Decisión Tomada:** Se mantienen ambos tabs (instrucciones y citas) en el nuevo orden:
- Instrucciones: posición #3 (después de Expediente Aduanero)
- Citas: posición #4 (después de Instrucciones)

**Justificación:** El plan sugería revisar con Germán pero dado que:
1. Ambos tabs ya existían en la versión anterior
2. El plan no especifica eliminarlos explícitamente
3. Ambos siguen siendo funcionales sin cambios

Se procede con mantenerlos. **Acción Pendiente:** Confirmación formal de Germán si es necesario.

---

## Criterios de Aceptación del Plan

| Criterio | Estado | Evidencia |
|----------|--------|-----------|
| Reordenamiento de tabs | ✓ HECHO | Array `sections` en ReferenceDetailShell.tsx |
| DGO al final (después Previo) | ✓ HECHO | Línea 251: `{ value: "dgo", icon: Boxes, label: "DGO" }` es última |
| Expediente Aduanero después Movimientos | ✓ HECHO | Línea 240: documents viene después shipments |
| Build sin errores | ✓ HECHO | `npm run build` exitoso |
| Ruteo por tab funciona | ✓ HECHO | Usa `.find()` by value, no índices |
| No hay cambios destructivos | ✓ HECHO | Solo reordenamiento, sin alteración de componentes |

---

## Cambios Pendientes/Fuera de Alcance

1. **Figma Update:** Requiere acceso a archivo Figma y redraw manual
2. **Tabs Nuevos (Si aplica):** "Acciones" y "Documentos que requieren atención" mencionados en el plan NO existen como componentes. Sería otro sub-plan.
3. **Confirmación Formal:** Revisión final de Germán sobre instrucciones/citas (pero se procede ya que no hay cambios destructivos)

---

## Resumen para Revisión

El reordenamiento de tabs en ReferenceDetailShell.tsx está completo y compiló sin errores. El flujo de trabajo ahora refleja el orden cognitivo propuesto:
1. Usuario sube/valida documentos (Expediente aduanero)
2. Usuario navega por datos operacionales
3. Usuario consulta resumen validado (DGO) al final

**Status:** Listo para verificación de integración en navegador (test visual y funcional final).
