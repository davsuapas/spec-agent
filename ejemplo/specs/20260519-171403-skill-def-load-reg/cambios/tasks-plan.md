### 2026-06-23 - 19:50:10
- **Tareas afectadas**: T1
- **Resumen del cambio en tasks.md**: `serde_yml` (deprecada v0.0.13) reemplazada por `serde-saphyr` (v0.0.27) en RT-002, dependencias de T1 y descripción de T3. La regla global `doc/reglas-globales-tecnicas.json` RT-002 se actualizó a v2 con `serde-saphyr`.
- **Lo que @plan debe hacer en cada tarea afectada**: 
  - **T1**: Reemplazar las 6 referencias a `serde_yml` en `tasks/T1/plan.md` por `serde-saphyr`. Esto incluye: mención en dependencias del crate, tipo `serde_yml::Error` → `serde_saphyr::Error` en `MarkdownError::YamlParse`, llamadas `serde_yml::from_str` → `serde_saphyr::from_str` en ejemplos de tests, y la nota de diseño sobre `PartialEq`/`Clone` que referencia `serde_yml::Error`. Posteriormente, el campo `source: serde_saphyr::Error` se reemplazó por `msg: String` en `YamlParse` (ver cambio tasks-plan del 2026-06-24 más abajo).
- **Estado**: ✅ Resuelto por @plan el 2026-06-04 — Se cambio a serde-saphyr = "0.0.28"

### 2026-06-24 - 17:20:13
- **Tareas afectadas**: T1, T3
- **Resumen del cambio en tasks.md**: `MarkdownError::YamlParse` migra de `source: serde_saphyr::Error` (~152 bytes) a `msg: String` para resolver `clippy::result_large_err`. El campo `msg` almacena `error.to_string()`. Esto permite que `MarkdownError` derive `PartialEq`.
- **Lo que @plan debe hacer en cada tarea afectada**: 
  - **T1 (`tasks/T1/plan.md`)**:
    1. **Paso 2.2 — Definición de `MarkdownError`**: En la variante `YamlParse`, eliminar el campo `#[source] source: serde_saphyr::Error` y añadir `msg: String` con doc comment `/// Human-readable YAML parse error.`
    2. **Paso 2.3 — `Display` impl**: Cambiar el patrón `Self::YamlParse { id, title: _, source }` a `Self::YamlParse { id, title: _, msg }`. En los dos `write!`, reemplazar `{source}` por `{msg}`.
    3. **Tests — `yaml_parse_with_id_includes_id_in_display`**: Reemplazar `source: yaml_err` por `msg: yaml_err.to_string()`.
    4. **Tests — `yaml_parse_without_id_is_readable`**: Reemplazar `source: yaml_err` por `msg: yaml_err.to_string()`.
    5. **Nota técnica 6** (actualmente dice: "No se deriva `PartialEq` en `MarkdownError`: la variante `YamlParse` contiene `serde_saphyr::Error` que no implementa `PartialEq`..."): Reemplazar por: "`MarkdownError` deriva `PartialEq`: la variante `YamlParse` ahora almacena `msg: String` (que implementa `PartialEq`), por lo que el enum puede derivar `PartialEq` sin restricciones. Esto simplifica los tests permitiendo `assert_eq!` en lugar de `matches!()`."
  - **T1 (`tasks/T1/checklist.md`)**:
    1. **Nota técnica 6 en checklist**: Actualizar la referencia de `serde_saphyr::Error no implementa PartialEq` a `MarkdownError ya puede derivar PartialEq porque YamlParse usa msg: String`.
- **Estado**: ✅ Resuelto por @plan el 2026-06-24 — YamlParse migrado a msg: String en plan.md (5 ubicaciones) y checklist.md

### 2026-06-24 - 17:20:14
- **Tareas afectadas**: T3
- **Resumen del cambio en tasks.md**: `MarkdownError::YamlParse` migra de `source: serde_saphyr::Error` (~152 bytes) a `msg: String` para resolver `clippy::result_large_err`. El campo `msg` almacena `error.to_string()`. Esto permite que `MarkdownError` derive `PartialEq`.
- **Lo que @plan debe hacer en cada tarea afectada**: 
  - **T3 (`tasks/T3/plan.md`)**:
    1. **Algoritmo paso 3**: Cambiar `MarkdownError::YamlParse { id: scanned_id, title: scanned_title, source: error }` a `MarkdownError::YamlParse { id: scanned_id, title: scanned_title, msg: error.to_string() }`.
    2. **Validación de coherencia** (línea con `serde_saphyr::Error`): Cambiar `source: serde_saphyr::Error` a `source: String` (o simplemente `msg: String` reflejando el nuevo campo).
- **Estado**: ✅ Resuelto por @plan el 2026-06-24 — YamlParse migrado a msg: String en plan.md (2 ubicaciones: algoritmo paso 3 y validación de coherencia)

### 2026-06-26 - 18:26:00
- **Tareas afectadas**: T1
- **Resumen del cambio en tasks.md**: Se eliminó el trait `SkillMeta` y se reemplazó por métodos inherentes en `PlatformSkill`, `DomainSkill` y `enum Skill`. `SkillMeta` se elimina de los re-exports de `lib.rs`; `Skill` se añade al re-export. Los casos de prueba T1-CP1 y T1-CP2 se reformularon con "métodos inherentes" en lugar de "implementa SkillMeta".
- **Lo que @plan debe hacer en cada tarea afectada**:
  - **T1 (`tasks/T1/plan.md`)**:
    1. Eliminar toda referencia al `trait SkillMeta` y sus `impl` blocks para `PlatformSkill` y `DomainSkill`.
    2. Añadir bloque `impl PlatformSkill` (separado del Builder) con métodos inherentes: `pub fn id(&self) -> &str`, `pub fn title(&self) -> &str`, `pub fn description(&self) -> &str`, `pub fn instructions(&self) -> &str`, `pub(crate) fn always_load(&self) -> bool`.
    3. Añadir bloque `impl DomainSkill` (separado) con: `pub fn id(&self) -> &str`, `pub fn title(&self) -> &str`, `pub fn description(&self) -> &str`, `pub fn instructions(&self) -> &str`.
    4. Añadir bloque `impl Skill` con `pub fn id(&self) -> &str`, `pub fn title(&self) -> &str`, `pub fn description(&self) -> &str`, `pub fn instructions(&self) -> &str` — cada método hace `match self { Skill::Platform(p) => p.id(), Skill::Domain(d) => d.id() }` etc.
    5. En `lib.rs`: eliminar `SkillMeta` del `pub use`, añadir `Skill` al `pub use` si no está.
    6. Actualizar tests: renombrar `platform_skill_implements_skill_meta` a algo como `platform_skill_inherent_methods_return_values`, ídem para `domain_skill`. Eliminar `use super::SkillMeta` o `use crate::types::SkillMeta` de los imports de test.
    7. En el test `skill_enum_accepts_both_variants`, la closure `desc` que llama `ps.description()` y `ds.description()` debe usar los métodos inherentes (ya lo hace, pero verificar que no dependa de `SkillMeta`).
- **Estado**: ✅ Resuelto por @plan el 2026-06-26 — SkillMeta eliminado, métodos inherentes añadidos a PlatformSkill (5), DomainSkill (4) y enum Skill (4). Tests renombrados. lib.rs actualizado (re-exports y doc). Diagrama y notas corregidos.
- **Tareas afectadas**: T3
- **Resumen del cambio en tasks.md**: Los tests de `markdown.rs` importan `use crate::types::SkillMeta`. Al eliminarse el trait, deben cambiarse a usar los métodos inherentes de `DomainSkill`.
- **Lo que @plan debe hacer en cada tarea afectada**:
   - **T3 (`tasks/T3/plan.md`)**:
     1. En la sección de tests: eliminar `use crate::types::SkillMeta;` del bloque de imports.
     2. Verificar que todas las llamadas como `skill.id()`, `skill.title()`, `skill.description()`, `skill.instructions()` funcionan con los métodos inherentes de `DomainSkill` (deberían funcionar sin cambios, ya que los nombres de método son idénticos).
- **Estado**: ✅ Resuelto por @plan el 2026-06-26 — Eliminada referencia a SkillMeta de imports de tests, verificados métodos inherentes en DomainSkill. Plan alineado con código real (extract_field con parseo string, IgnoredAny para validación, PartialEq en MarkdownError).

### 2026-06-26 - 18:26:02
- **Tareas afectadas**: T4
- **Resumen del cambio en tasks.md**: `load()` cambió de `-> Result<String, LoadError>` a `-> Result<&Skill, LoadError>`. El consumidor obtiene instrucciones vía `load("x")?.instructions()` sin clon. `catalog()` usa métodos inherentes de `Skill` en lugar del trait `SkillMeta`. El `always_load` se accede vía `s.always_load()` (método inherente `pub(crate)`).
- **Lo que @plan debe hacer en cada tarea afectada**:
  - **T4 (`tasks/T4/plan.md`)**:
    1. Cambiar la firma de `load()`: de `fn load(&self, id: &str) -> Result<String, LoadError>` a `fn load(&self, id: &str) -> Result<&Skill, LoadError>`.
    2. En la implementación de `load()`: en lugar de clonar/devolver `String`, devolver `Ok(skill)` (referencia al valor en el `HashMap`).
    3. En la implementación de `catalog()`: usar `skill.id()` y match sobre variantes para extraer `id`/`description` (métodos inherentes del enum `Skill`), en lugar del trait `SkillMeta`.
    4. Cambiar `s.always_load` (acceso a campo) por `s.always_load()` (llamada al método inherente `pub(crate)`).
    5. Actualizar tests de `load()`: en lugar de comparar `String`, verificar `load("x")?.instructions()` o `load("x")?.id()`.
- **Estado**: ✅ Resuelto por @plan el 2026-06-29 — load() → Result<&Skill, LoadError>, catalog() con métodos inherentes, always_load() integrados en plan.md
- 
### 2026-06-29 - 18:16:00
- **Tareas afectadas**: T4
- **Resumen del cambio en tasks.md**: `SkillManager` ahora recibe los skills en el constructor `new(platform_skills: Vec<PlatformSkill>, domain_skills: Vec<DomainSkill>) -> Self` en lugar de tener métodos separados `register_platform` y `register_domain`. El constructor preasigna capacidad exacta en el `HashMap` y resuelve duplicados silenciosamente (RF-010). Todos los métodos pasan a tomar `&self` (sin `&mut self`). Se eliminan los métodos de registro.
- **Lo que @plan debe hacer en cada tarea afectada**: T4: reescribir el plan de implementación de `manager.rs` para reflejar el nuevo constructor único. Los tests deben usar `SkillManager::new(platform_vec, domain_vec)` en lugar de construir vacío y luego llamar a `register_platform`/`register_domain`. El test de `Arc` ya no necesita `RwLock` para solo lectura.
- **Estado**: ✅ Resuelto por @plan el 2026-06-29 — Constructor único `new(platform_skills, domain_skills)` integrado en plan T4. Eliminados `register_platform` y `register_domain`. Tests reescritos para constructor único. `Arc` sin `RwLock`.
