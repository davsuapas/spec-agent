# T2: Construcción de skills con Builder

**Creada**: 2026-06-24
**Estado**: Implementada

---

## Instrucciones para el agente codificador

1. **Carga el skill `coder`** antes de empezar cualquier implementación. Sigue su guía.

2. **Cambia el estado** del plan en el encabezado según la fase en la que estés:
   - Al **terminar** exitosamente la codificación: cambia `**Estado**: En codificación` por `**Estado**: Implementada`.
   - Durante la codificación, si el plan necesita ajustes:
    - Si son cambios menores o locales, puedes aplicarlos directamente.
    - Si son modificaciones estructurales o de arquitectura, no las hagas tú: informa al usuario y pídele que cambie al agente `@spec/plan`, explicando los ajustes necesarios y su motivo.

3. **Sigue el plan detallado** que aparece a continuación. Es la fuente de verdad de lo que hay que implementar. Si encuentras una contradicción insalvable con el código real, resuélvela con el desarrollador. Si la solución requiere cambios en el plan, aplica el punto 2.

4. **No modifiques** esta sección de instrucciones.

---

## Plan detallado de implementación

### Contexto de la tarea

T2 es una tarea **exclusivamente de tests**. No crea ni modifica código de
producción. Verifica el comportamiento de los builders generados por
`derive_builder` en T1 (`PlatformSkillBuilder` y `DomainSkillBuilder`).

**Prerrequisito**: T1 (`Tipos de datos y errores`) implementada. Los
builders ya existen y son funcionales en `types.rs`. El `Cargo.toml` ya
incluye `derive_builder = "0.20"`.

**Archivos a modificar**:

| Archivo | Tipo de cambio |
|:--------|:---------------|
| `carisa/skill/Cargo.toml` | Añadir `[dev-dependencies] rstest = "0.22"` |
| `carisa/skill/src/types.rs` | Añadir 4 funciones de test en el módulo `#[cfg(test)] mod tests` existente |

**Casos de prueba cubiertos**: T2-CP1 a T2-CP7 (7 tests, dos de ellos
agrupados vía DDT con `rstest` en 4 casos).

---

### Paso 0 — Añadir dev-dependency `rstest`

Añadir al final de `carisa/skill/Cargo.toml`:

```toml
[dev-dependencies]
rstest = "0.22"
```

El archivo completo quedará:

```toml
[package]
name = "carisa-skill"
description = ""
authors = ["David Suárez Pascual <dav.sua.pas@gmail.com>"]
version.workspace = true
edition.workspace = true
license.workspace = true
repository.workspace = true

[dependencies]
derive_builder = "0.20"
serde = { version = "1", features = ["derive"] }
serde-saphyr = "0.0.28"
thiserror = "2"

[lints]
workspace = true

[dev-dependencies]
rstest = "0.22"
```

---

### Paso 1 — Tests de construcción exitosa (T2-CP1, T2-CP2)

Añadir al final del bloque `#[cfg(test)] mod tests { ... }` existente en
`carisa/skill/src/types.rs`, **después** de los tests de T1:

#### T2-CP1 — `PlatformSkillBuilder` completo

Accede a los campos directamente (no vía `SkillMeta`) para no duplicar
las aserciones de T1. Incluye verificación explícita de `always_load`.

```rust
/// T2-CP1: PlatformSkillBuilder with all fields produces Ok with
/// correct values.
#[test]
fn platform_skill_builder_success() {
    let skill = PlatformSkillBuilder::default()
        .id("ps-core".to_owned())
        .title("Core Skill".to_owned())
        .description("A platform-level skill".to_owned())
        .instructions("Do X, then Y".to_owned())
        .always_load(true)
        .build()
        .expect("all required fields set");

    assert_eq!(skill.id, "ps-core");
    assert_eq!(skill.title, "Core Skill");
    assert_eq!(skill.description, "A platform-level skill");
    assert_eq!(skill.instructions, "Do X, then Y");
    assert!(skill.always_load);
}
```

#### T2-CP2 — `DomainSkillBuilder` completo

Mismo patrón: acceso directo a los campos del struct resultante.

```rust
/// T2-CP2: DomainSkillBuilder with all fields produces Ok with
/// correct values.
#[test]
fn domain_skill_builder_success() {
    let skill = DomainSkillBuilder::default()
        .id("ds-custom".to_owned())
        .title("Custom Skill".to_owned())
        .description("A user-defined skill".to_owned())
        .instructions("Step 1: …".to_owned())
        .build()
        .expect("all required fields set");

    assert_eq!(skill.id, "ds-custom");
    assert_eq!(skill.title, "Custom Skill");
    assert_eq!(skill.description, "A user-defined skill");
    assert_eq!(skill.instructions, "Step 1: …");
}
```

---

### Paso 2 — Tests de campos obligatorios (T2-CP3 a T2-CP7)

Añadir `use rstest::rstest;` al inicio del bloque `#[cfg(test)]` y los
siguientes tests.

#### T2-CP3 a T2-CP6 — `DomainSkill` sin cada campo (DDT)

Test parametrizado con 4 casos. Cada caso omite un campo distinto y
verifica que `build()` devuelve `Err` con un mensaje que menciona el
nombre del campo ausente.

Los `case` ids siguen la convención `{qué_se_prueba}.{qué_se_espera}`.

```rust
/// T2-CP3..CP6: DomainSkillBuilder fails when any required field is
/// missing. The error message must mention the missing field name.
#[rstest]
#[case::missing_id_returns_error(
    DomainSkillBuilder::default()
        .title("t".into())
        .description("d".into())
        .instructions("i".into()),
    "id"
)]
#[case::missing_title_returns_error(
    DomainSkillBuilder::default()
        .id("x".into())
        .description("d".into())
        .instructions("i".into()),
    "title"
)]
#[case::missing_description_returns_error(
    DomainSkillBuilder::default()
        .id("x".into())
        .title("t".into())
        .instructions("i".into()),
    "description"
)]
#[case::missing_instructions_returns_error(
    DomainSkillBuilder::default()
        .id("x".into())
        .title("t".into())
        .description("d".into()),
    "instructions"
)]
fn domain_skill_builder_requires_all_fields(
    #[case] builder: DomainSkillBuilder,
    #[case] field_name: &str,
) {
    let err = builder.build().unwrap_err();
    let msg = err.to_string();
    assert!(
        msg.contains(field_name),
        "Error for missing field '{field_name}' should mention \
         the field name, got: {msg}"
    );
}
```

#### T2-CP7 — `PlatformSkill` sin `always_load`

Caso único (sin DDT, solo una variante):

```rust
/// T2-CP7: PlatformSkillBuilder without always_load fails.
#[test]
fn platform_skill_builder_requires_always_load() {
    let err = PlatformSkillBuilder::default()
        .id("p".to_owned())
        .title("t".to_owned())
        .description("d".to_owned())
        .instructions("i".to_owned())
        .build()
        .unwrap_err();
    let msg = err.to_string();
    assert!(
        msg.contains("always_load"),
        "Error for missing 'always_load' should mention \
         the field name, got: {msg}"
    );
}
```

---

### Paso 3 — Verificación

Tras escribir los cambios en `Cargo.toml` y `types.rs`, el codificador
debe ejecutar los tests específicos de esta tarea:

```bash
cargo test -p carisa-skill types::tests -- --test-threads 4 --no-fail-fast
```

Todos los tests deben pasar. Si alguno falla, revisar que el mensaje de
error de `derive_builder` contenga efectivamente el nombre del campo
ausente (el formato exacto puede variar ligeramente entre versiones de
`derive_builder`; la aserción usa `.contains()` para ser tolerante).

Y finalmente las validaciones completas del crate:

```bash
cargo fmt
cargo check -p carisa-skill
cargo clippy -p carisa-skill --all-targets -- -D warnings
cargo doc -p carisa-skill --no-deps
cargo test -p carisa-skill
```

Todas deben pasar sin errores ni warnings.

---

### Notas técnicas y advertencias

1. **Sin nuevo código de producción**: esta tarea solo añade tests. No
   se crean módulos nuevos ni se modifican `types.rs` (fuera de
   `#[cfg(test)]`), `error.rs` ni `lib.rs`.

2. **Los builders no necesitan imports adicionales**: `PlatformSkillBuilder`
   y `DomainSkillBuilder` ya son accesibles desde el módulo de tests de
   `types.rs` porque `use super::*;` los trae al scope.

3. **`.to_owned()` en literales**: necesario porque los campos de los
   structs son `String` (owned). El lint `str_to_string` del workspace
   puede advertir; `.to_owned()` es la forma idiomática aceptada.

4. **Formato del mensaje de error de `derive_builder`**: en la versión
   `0.20`, cuando falta un campo obligatorio el mensaje tiene el formato
   `"Field not initialized: <field_name>"`. Las aserciones usan
   `.contains(field_name)` para ser robustas ante cambios menores de
   formato entre versiones. Si `derive_builder` cambia el formato en el
   futuro y los tests fallan, actualizar la aserción.

5. **`rstest` requiere el import `use rstest::rstest;`** dentro del
   bloque `#[cfg(test)] mod tests`. Sin él, la macro `#[rstest]` no se
   resuelve.

6. **No hay solapamiento con tests de T1**: T1 verifica el contrato
   `SkillMeta`; T2 verifica el contrato del Builder (construcción
   exitosa + fallos por campos omitidos). Los tests de T2 acceden
   directamente a los campos del struct, no usan `SkillMeta`.

7. **Validación de coherencia con código existente (2026-06-24)**:
   - `PlatformSkillBuilder` y `DomainSkillBuilder` son accesibles desde
     el módulo de tests vía `use super::*;` (ya usado en T1).
   - `rstest 0.22` es una versión real publicada en crates.io.
   - El `Cargo.lock` no contiene restricciones previas sobre `rstest`.
   - El formato real del error de `derive_builder` v0.20 es
     `"Field not initialized: {field_name}"`, compatible con las
     aserciones `.contains(field_name)` del plan.
   - Sin conflictos con `lib.rs`, `error.rs`, ni el resto de `types.rs`.
