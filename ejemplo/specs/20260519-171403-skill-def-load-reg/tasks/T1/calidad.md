# Checklist de Calidad del Plan de Implementación

**Propósito**: Validar la completitud y calidad del plan detallado antes de pasarlo al agente codificador.
**Creado**: 2026-06-24
**Actualizado**: 2026-06-26 (cambio SkillMeta → métodos inherentes)
**Plan**: `specs/20260519-171403-skill-def-load-reg/tasks/T1/plan.md`
**Plan de tareas**: `specs/20260519-171403-skill-def-load-reg/tasks.md`

## Completitud

- [x] Todos los casos de prueba del `tasks.md` para esta tarea están reflejados en el plan detallado con suficiente detalle
- [x] Todos los archivos a crear/modificar están listados con rutas concretas
- [x] Los pasos de implementación son accionables y no dejan ambigüedad sobre qué hacer
- [x] Los diagramas (si los hay) son correctos y usan notación Mermaid válida
- [x] Un ingeniero junior sería capaz de implementarlo
- [x] Todos los cambios en `tasks-plan.md` están resueltos:
  - ✅ `serde_yml` → `serde-saphyr` (6 referencias)
  - ✅ `YamlParse.source` → `msg: String` (5 ubicaciones + nota técnica)
  - ✅ `SkillMeta` → métodos inherentes (trait eliminado, 3 bloques `impl` añadidos, tests renombrados, `lib.rs` actualizado)

## Coherencia con el plan de tareas

- [x] El plan detallado respeta el plan específico de la tarea definido en `tasks.md`
- [x] El plan detallado no contradice el plan arquitectónico global (métodos inherentes como API pública, sin trait)
- [x] El plan detallado cumple los requisitos técnicos aplicables (RT-002 v2: `serde-saphyr`, RT-004 v2: sin trait en API pública)
- [x] Las dependencias con otras tareas están identificadas y respetadas (T3 impactada, documentado en nota 10)

## Coherencia con el código existente

- [x] Se ha verificado que los archivos y funciones referenciados existen realmente
- [x] Las firmas, interfaces y tipos propuestos son compatibles con el código existente
- [x] Los patrones de `coder` y `blueprint` se respetan (métodos inherentes en lugar de trait para acceso zero-cost, enum Skill con match exhaustivo)
- [x] No se introducen conflictos con otros módulos del proyecto (salvo T3, documentado)
- [x] `MarkdownError` deriva `PartialEq` porque `YamlParse` usa `msg: String` — consistente con el código real

## Calidad del contenido

- [x] El plan no contiene estimaciones de esfuerzo temporal
- [x] Los diagramas Mermaid no contienen comillas dobles ni paréntesis dentro de corchetes
- [x] El lenguaje es claro y orientado a la acción
- [x] Las notas técnicas y advertencias están incluidas donde aplica (nota 10 añadida sobre impacto en T3)

## Notas

- Validación tras cambio `serde_yml` → `serde-saphyr` (6 referencias reemplazadas en `plan.md`).
- Validación tras cambio `YamlParse.source: serde_saphyr::Error` → `msg: String` (5 ubicaciones en `plan.md`, 2 referencias en tests, 1 nota técnica).
- Validación tras cambio `SkillMeta` → métodos inherentes (trait + 2 impls eliminados; 3 bloques `impl` nuevos; tests renombrados; `lib.rs` actualizado; diagrama corregido).
- La regla global RT-002 v2 ya refleja `serde-saphyr`. La regla RT-004 v2 elimina la mención a `SkillMeta`.
- `DomainSkill` usa `pub(crate)` en sus campos para acceso directo desde `markdown.rs` (T3).
- El código real (`types.rs`, `lib.rs`) aún contiene `SkillMeta`. El codificador debe aplicar los cambios de este plan para alinear el código.
