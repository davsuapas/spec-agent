# coder: Guía completa y reglas para la codificación de software.

Cubre reglas de código, refactorización, estilo de código, documentación, testing y buenas prácticas.

---

# INSTRUCCIONES DE LOS PASOS A SEGUIR POR EL AGENTE CUANDO ABORDE LA TAREA DE CODIFICACIÓN

**Seguir los siguientes pasos teniendo en cuenta todos las REGLAS especificadas por puntos # 1, # 2, etc, así con todas**

## Codificar siguiendo los siguientes pasos:

### Razonar antes de escribir código

Antes de escribir cualquier línea de código, razona explícitamente sobre los siguientes puntos. No avances al paso 2 hasta haberlos resuelto todos.

#### Entender la tarea

- Sigue la instrucciones del plan detallado de implementación.
- Si algo no está claro, pide aclaración antes de continuar.

#### Verificar el contexto antes de tocar código

- Lee los ficheros existentes que vas a modificar. No asumas su contenido.
- Identifica las dependencias directas del código que vas a escribir o modificar.

### Ejecuta los tests de cada módulo editado o añadido

Ejecuta el comando `cargo test <fichero_rs> -- --test-threads 4 --no-fail-fast`. Si se produce algún error en los test, detente, corrige los errores y vuelve a lanzar los test para ver que se ha solucionado.

<fichero_rs>: Ejemplo: Si los test se encuentran físicamente en src/infra/dominio.rs, sería infra::dominio.

### Realizar validaciones una vez concluida la sesión de trabajo

- Antes de dar por finalizada la tarea, **debes** ejecutar los siguientes comandos en la terminal de forma secuencial. 

#### Formatear el código

Ejecuta el comando `cargo fmt` en el terminal desde la ruta principal del proyecto y verifica que el formato es correcto. Si no es así, corrígelo.

#### Analiza el código

- Ejecuta el comando `cargo check` en el terminal desde la ruta principal del proyecto. Si este comando falla, detente, corrige los errores y vuelve a lanzar el comando para ver que se ha solucionado.
- Ejecuta el comando `cargo clippy --all-targets -- -D warnings` en el terminal desde la ruta principal del proyecto. Si este comando devuelve warning o error, detente, corrige los errores y vuelve a lanzar el comando para ver que se ha solucionado.

#### Analiza la documentación

Ejecuta el comando `cargo doc --no-deps` en el terminal desde la ruta principal del proyecto. Si este comando falla, detente, corrige los errores y vuelve a lanzar el comando para ver que se ha solucionado.

# 1. REGLAS PARA REFACTORIZAR

> Filosofía: inmutabilidad por defecto, errores como valores, estados inválidos imposibles, explicitez sobre magia.
> En Rust: el compilador es tu aliado. Si el borrow checker o el type system te fuerzan a un diseño, escúchalos antes de añadir indirección.
>
> **Prioridad:** [!!] crítico · [!] importante

## Reglas Universales

- Refactoriza **antes** de añadir funcionalidad nueva, nunca durante.
- Refactoriza también al corregir errores, revisar código o detectar señales de alarma.
- Si el comportamiento externo cambia, no es refactorización, es un cambio funcional.
- Refactorización local → codificador. Refactorización estructural (jerarquía, patrón arquitectónico) → decisión del arquitecto.

## Checklist antes de entregar

- [ ] El comportamiento externo no ha cambiado.
- [ ] Los tests existentes siguen pasando.
- [ ] Los comentarios están actualizados.
- [ ] No se ha introducido deuda técnica nueva.
- [ ] Los nombres reflejan la nueva estructura.

## Parte I — Composición y Estructura

### [!!] Extraer función

**Aplica cuando:** un bloque hace más de una cosa o necesita un comentario para entenderse.
**Señal nuevo:** vas a escribir un comentario que describe el siguiente bloque.
**Señal existente:** función larga con secciones separadas por comentarios o saltos de línea con semántica distinta.

### [!!] Inline de función

**Aplica cuando:** la indirección oscurece más de lo que clarifica.
**Señal existente:** el cuerpo es más claro que el nombre, o la función tiene un solo llamador sin semántica propia.
**No apliques** por rendimiento especulativo.

### [!!] Extraer módulo

**Aplica cuando:** un módulo acumula responsabilidades que cambian por razones distintas (SRP).
**Señal existente:** nombre genérico (`manager`, `helper`, `utils`) o dos grupos de funciones que podrían vivir de forma independiente.

### [!] Inline de módulo

**Aplica cuando:** el módulo ya no justifica su existencia.
**Señal existente:** un solo tipo, un solo llamador, o su abstracción no aporta nada que un tipo simple no resuelva.

### [!] Reemplazar struct por tipo de datos puro

**Aplica cuando:** un struct solo agrupa datos sin comportamiento real.
**Señal existente:** todos los métodos son getters/setters sin validación ni transformación.
**Beneficio en Rust:** deriva `Copy`, `Clone`, `PartialEq` libremente; más fácil de testear.

## Parte II — Movimiento de Responsabilidades

### [!!] Mover función

**Aplica cuando:** una función usa más datos de otro módulo o struct que del propio.
**Señal existente:** recibe como parámetro el objeto al que realmente pertenece, o accede constantemente a los internos de otro tipo (LoD).

### [!!] Mover atributo

**Aplica cuando:** un campo es usado principalmente por otro struct.
**Señal existente:** para leer o escribir el campo siempre se navega desde otro objeto.

### [!] Ocultar delegado

**Aplica cuando:** el cliente navega por la estructura interna de un objeto para llegar a lo que necesita.
**Señal existente:** cadenas `a.b().c()`. Encapsula la navegación en un método del objeto intermedio.

### [!] Eliminar intermediario

**Aplica cuando:** un struct solo delega sin añadir valor.
**Señal existente:** la mayoría de sus métodos son delegaciones directas sin lógica propia.
**No elimines** si existe para desacoplar o cumplir DIP.

## Parte III — Simplificación de Expresiones

### [!!] Descomponer condicionales

**Aplica cuando:** una condición requiere esfuerzo para entender su intención.
**Señal nuevo:** `if` con más de dos condiciones encadenadas.
**Señal existente:** operadores lógicos, negaciones o comparaciones no inmediatamente legibles.
**Acción:** extrae a una función cuyo nombre exprese la intención, no la mecánica.

### [!!] Consolidar condicionales duplicados

**Aplica cuando:** varias ramas llevan a la misma acción.
**Señal existente:** el mismo bloque aparece en múltiples ramas `if/else`.

### [!!] Reemplazar condicional por polimorfismo

**Aplica cuando:** un `if/else` o `match` selecciona comportamiento según tipo o estado y tiende a crecer.
**Señal existente:** añadir un caso nuevo obliga a buscar y modificar todos los puntos que discriminan por ese tipo.
**Acción en Rust:** enum con `match` exhaustivo si las variantes son cerradas; trait si son abiertas. Ver Parte V.

### [!!] Reemplazar `panic!` en errores recuperables por `Result`

**Aplica cuando:** una función falla de forma predecible y el llamador puede recuperarse.
**Señal existente:** `panic!`, `unwrap()` o `expect()` en rutas de parseo, I/O o validación de entrada.
**Acción:** devuelve `Result<T, E>`. El error es un valor; el llamador decide qué hacer.
**Mantén** `expect()` con mensaje que explique el invariante cuando el panic es genuinamente correcto.

### [!] Eliminar variables temporales innecesarias

**Aplica cuando:** una variable se usa una vez y no aporta nombre semántico.
**Señal existente:** nombres como `temp`, `result`, `aux`, `data` que viven una sola línea.
**No elimines** si el nombre clarifica la intención.

### [!] Introducir variable explicativa

**Aplica cuando:** una expresión compleja no es legible.
**Señal existente:** expresión larga en una condición o retorno que requiere esfuerzo para entender.

## Parte IV — Gestión de Parámetros

### [!!] Introducir struct parámetro

**Aplica cuando:** una función recibe más de 4 parámetros relacionados conceptualmente.
**Señal nuevo:** firma con parámetros del mismo tipo primitivo que podrían confundirse.
**Señal existente:** varios parámetros siempre viajan juntos entre funciones.

### [!!] Eliminar banderas booleanas

**Aplica cuando:** un booleano cambia el comportamiento de la función de forma significativa.
**Señal novo:** parámetro `is_urgent`, `include_inactive`, `force`, `dry_run`.
**Señal existente:** `if` en el primer nivel que bifurca por el valor del booleano.
**Acción:** separa en dos funciones con nombres que expresen la intención de cada variante.

### [!] Reemplazar parámetro por consulta

**Aplica cuando:** un parámetro puede obtenerse desde el propio objeto o desde otro parámetro disponible.
**Señal existente:** el llamador calcula un valor solo para pasárselo a la función, que podría calcularlo sola.

### [!] Preservar inmutabilidad en parámetros

**Aplica cuando:** una función modifica directamente una colección o estructura recibida.
**Señal existente:** parámetro mutado dentro de la función en lugar de devolver una versión transformada.
**En Rust:** recibir `&mut T` cuando `T → T` o `&T → NewT` es la firma correcta es una señal de alarma.

## Parte V — Tipos y Estados

### [!!] Modelar estados inválidos como imposibles

**Aplica cuando:** el código tiene validaciones defensivas repetidas para comprobar consistencia.
**Señal existente:** `if !self.initialized`, validaciones repetidas antes de usar un valor, o flags booleanos que combinados crean estados imposibles.
**Acción en Rust:** constructor privado o `TryFrom` que garantiza el invariante. El tipo válido es el único representable. Mueve la validación a la construcción, no al uso.

### [!!] Reemplazar flags booleanos combinados por enum

**Señal:** dos o más `bool` en un struct donde combinados crean estados imposibles (`is_open: true, is_closed: true`).
**Acción:** un enum que hace imposibles los estados inválidos. El compilador valida exhaustividad con `match`.

### [!!] Reemplazar `Option<bool>` por enum con tres variantes

**Señal:** `Option<bool>` donde `None`, `Some(false)` y `Some(true)` tienen semánticas distintas no relacionadas con ausencia.
**Acción:** enum nombrado con variantes que expresen cada caso.

### [!!] Reemplazar campo tipo `String`/constante por enum

**Aplica cuando:** un string o entero representa el tipo y se usa para bifurcar comportamiento en múltiples lugares.
**Señal existente:** constantes `TYPE_A: &str` en `if/match` a lo largo del código.
**Acción en Rust:** enum con variantes. `match` exhaustivo garantiza que todos los casos están cubiertos.

### [!!] Subir validación a la construcción del tipo

**Señal:** `if !self.initialized` o validaciones defensivas repetidas antes de usar un valor.
**Acción:** constructor privado o `TryFrom` que garantiza el invariante en construcción.

### [!] Reemplazar `unwrap()`/`expect()` en lógica de negocio

**Señal:** `unwrap()` fuera de tests o de contextos donde el panic es genuinamente imposible y documentado.
**Acción:** propaga con `?`, convierte con `ok_or`, o maneja el caso explícitamente.

### [!] Consolidar tipos de error en un enum

**Señal:** función que devuelve `Box<dyn Error>` cuando los errores posibles son conocidos y finitos.
**Acción:** enum de error con variantes. Permite `match` exhaustivo al llamador. Implementa `From` para conversión automática con `?`.

## Parte VI — Ownership y Borrows

### [!!] Eliminar `.clone()` defensivo

**Señal:** `.clone()` existe para callar al compilador, no por necesidad semántica.
**Acción por orden de preferencia:** reorganizar el orden de operaciones · cambiar a borrow `&` · `mem::take()` / `mem::replace()` · descomponer el struct. Clonar es la última opción.

### [!!] Descomponer struct cuando el borrow checker lo pide

**Señal:** el compilador rechaza borrows de campos independientes en el mismo scope.
**Acción:** extrae campos relacionados en structs más pequeños que el struct original componga. No añadas `.clone()`.

### [!] Reducir lifetimes explícitos

**Señal:** anotaciones `'a` proliferan y hacen ilegible una firma.
**Acción:** comprueba si `impl Trait` en el retorno, owned types, o reestructurar la propiedad eliminan la necesidad.


## Parte VII — Abstracción y Polimorfismo

### [!!] Decidir entre genérico y trait object al refactorizar polimorfismo

- Variantes conocidas en compilación, rendimiento crítico → genérico `fn foo<T: Trait>(t: T)`.
- Variantes abiertas o colección heterogénea → `Box<dyn Trait>`.
- No mezcles sin razón: `Box<dyn Trait>` donde un genérico resuelve el problema añade heap allocation innecesaria.

### [!!] Composición en lugar de delegación con Deref

**Señal:** `impl Deref for Bar { type Target = Foo }` donde Bar y Foo no tienen relación de puntero, o struct que reimplementa métodos de otro struct solo para redirigirlos.
**Acción:** contén el struct como campo y delega explícitamente, o extrae un trait compartido si el comportamiento es genuinamente común. `Deref` es solo para smart pointers.

### [!!] Extraer trait cuando varias implementaciones divergen

**Aplica cuando:** varias structs tienen comportamiento similar sin relación formal, o necesitas desacoplar de implementaciones concretas (DIP).
**Señal existente:** el llamador depende de un struct concreto cuando solo necesita un subconjunto de su comportamiento.

### [!] Eliminar trait con una sola implementación permanente

**Señal:** trait que nunca tendrá más de un implementador real y no existe para testing ni para DIP.
**Acción:** elimina el trait, usa el tipo concreto directamente. YAGNI.


## Parte VIII — Nomenclatura y Legibilidad

### [!!] Renombrar para expresar intención

**Aplica cuando:** un nombre describe la mecánica en lugar de la intención, o usa abreviaturas ambiguas.
**Señal existente:** necesitas leer la implementación para entender qué hace o representa un símbolo.
**En Rust:** sigue convenciones — `new()` constructores, `into_*` conversiones que consumen, `as_*` conversiones baratas, `to_*` conversiones costosas. Romperlas requiere documentación explícita.

### [!!] Hacer explícita la mutabilidad

**Aplica cuando:** defines variables o parámetros que podrían ser inmutables.
**Señal existente:** `let mut` donde el valor nunca se muta, o colecciones modificadas en lugar de transformadas.
**En Rust:** `let mut` solo cuando sea necesario. El compilador avisa, pero la señal de diseño es más sutil: preferir `let data = { let mut d = ...; d.sort(); d }` sobre mutar y reusar.

### [!] Eliminar comentarios que describen el qué

**Señal existente:** comentarios como `// increment counter`, `// call service`.
**Acción:** si el código necesita ese comentario, está mal nombrado. Renombra o extrae una función.
**En Rust:** los comentarios `// SAFETY:` y `// PANICS:` son obligatorios cuando aplican; no son candidatos a eliminar.

### [!] Reemplazar literales mágicos por constantes con nombre

**Señal existente:** `0.15`, `86400`, `"active"` directamente en la lógica sin nombre que exprese su significado.
**En Rust:** `const` con tipo explícito en el nivel de módulo.


## Parte IX — Manejo de Errores y Robustez

### [!!] Hacer los efectos visibles en la firma

**Aplica cuando:** una función tiene efectos secundarios no evidentes.
**Señal existente:** devuelve `()` pero modifica estado externo sin que el nombre lo indique.
**Acción:** devuelve un tipo que exprese el resultado posible incluyendo el fallo (`Result`). El nombre debe indicar el efecto.

### [!!] Fallar rápido y explícitamente

**Aplica cuando:** una función recibe entrada inválida o está en un estado que no puede manejar.
**Señal existente:** continúa con datos inválidos produciendo errores más adelante difíciles de rastrear.
**Acción:** valida en el punto de entrada con guard clauses. El error debe ocurrir lo más cerca posible de su causa.

### [!] Separar validación de ejecución

**Aplica cuando:** validación y lógica principal están mezcladas.
**Señal existente:** lógica principal anidada en múltiples niveles de validación (pyramid of doom).
**Acción:** guard clauses al inicio con retorno temprano. La lógica principal queda al nivel más bajo de indentación.

## Parte X — Concurrencia y Estado Compartido

### [!!] Minimizar estado mutable compartido

**Aplica cuando:** componentes se ejecutan concurrentemente o comparten estado.
**Señal existente:** campos mutables accedidos desde múltiples tareas sin sincronización explícita.
**Acción:** pasa datos inmutables entre tareas. Si el estado compartido es inevitable, hazlo explícito y localizado.

### [!!] Reemplazar `Arc<Mutex<T>>` innecesario por paso de mensajes

**Señal:** `Arc<Mutex<T>>` compartido entre tareas donde la comunicación es unidireccional o por eventos.
**Acción:** canal `mpsc` o `broadcast`. El estado vive en un solo propietario; los demás envían mensajes.

### [!!] Hacer explícitas las operaciones asíncronas

**Aplica cuando:** una operación puede bloquearse o ejecutarse de forma asíncrona.
**Señal existente:** I/O síncrono en contextos async, o funciones async mezcladas con síncronas sin distinción clara.
**Acción:** la asincronía debe ser visible en la firma (`async fn`) y en el nombre cuando ayude.

### [!] Anotar `Send + Sync` en abstracciones concurrentes

**Señal:** trait object usado en contexto async o multi-hilo sin bounds de `Send`.
**Acción:** `Box<dyn Trait + Send + Sync>` cuando el contexto lo requiere. Falla en compilación, no en runtime.

### [!] Separar lógica pura de efectos secundarios

**Aplica cuando:** lógica de negocio está mezclada con I/O, logging o modificación de estado externo.
**Señal existente:** imposible testear la lógica de negocio sin mockear servicios externos.
**Acción:** extrae la lógica pura como funciones que reciben datos y devuelven datos. Empuja los efectos hacia los bordes del sistema.

---

# 2. REGLAS DE ESTILO DE CÓDIGO

## Formato
- **Usa** un máximo de 80 caracteres por línea en todo el código, incluyendo docstrings, comentarios y documentación. Nunca excedas este límite.
- **Usa SIEMPRE** 2 espacios por nivel de indentación, NUNCA tabs.

## Convenciones de nombres

Usa la conveción para el lenguaje rust

## Estructura y organización

### Reducir anidación

Validaciones y control de errores al principio del bloque con return temprano o `continue`. La lógica principal queda al nivel más bajo de indentación.

### Ámbito de variables

Declara las variables lo más cerca posible de donde se usan.
**Excepción:** si entra en conflicto con la regla de reducir anidación, tiene precedencia la anidación.

### Agrupar declaraciones similares

- Agrupa por funcionalidad, tipo de operación, dependencia lógica o contexto de ejecución.
- Constantes siempre en la cabecera del módulo.
- No fuerces agrupaciones artificiales que perjudiquen la legibilidad.

### Orden de funciones

- Funciones **públicas** al principio del fichero.
- Funciones **privadas** intercaladas junto a las públicas que las llaman (proximidad).
- Funciones **helper** al final del fichero.

### Líneas en blanco

- Una línea en blanco entre secciones lógicas dentro de una función.
- Una línea en blanco antes de un comentario que introduce un nuevo bloque.
- Nunca línea en blanco al inicio de una estructura de control (`if`, `for`, etc.).

### Imports

- Añadir siempre los imports al principio del fichero, **nunca** dentro de clases y funciones

## Patrones de implementación

### Constructores estáticos

Si la inicialización requiere lógica compleja, usa constructores estáticos en lugar de sobrecargar constructor. Inclúyelos en sus propios impl.

### Evitar else innecesario

Si una variable tiene valor por defecto y solo cambia bajo una condición, asigna el valor por defecto antes del `if` y elimina el `else`.

### Enumerados frente a constantes

Usa `enum` en lugar de constantes sueltas cuando los valores están relacionados entre sí.

### Funciones de módulo frente a métodos estáticos

Si las funciones no usan propiedades de una clase ni tienen cohesión entre sí, defínelas como funciones de módulo. Nunca crees una clase solo para agrupar métodos estáticos sin cohesión.

## Calidad del código

### Sin valores mágicos

Nunca uses literales numéricos, strings u otros valores sin contexto directamente en la lógica. Define constantes descriptivas en la cabecera del módulo.

### Sin efectos secundarios ocultos

Si una función modifica estado, debe ser evidente en su nombre. Las funciones deben ser predecibles y fáciles de testear.

### Reducir complejidad cognitiva

Divide condiciones complejas en variables intermedias descriptivas. Evita condiciones excesivamente largas en un solo `if`.

### Inmutabilidad por defecto

Prefiere estructuras inmutables siempre que sea posible. Especialmente para configuraciones y datos que no deben modificarse.

---

# 3. REGLAS PARA LA DOCUMENTACIÓN (Rust / rustdoc)

## Principios fundamentales

- **Documentar antes de implementar.**
- **Actualizar en cada refactorización.** Cualquier cambio en el código implica revisar y adaptar su documentación.
- **Documentar elementos privados** solo si su lógica es compleja o no evidente.
- **Documentar todos los elementos públicos** sin excepción.
- **Aprovechar y adaptar la documentación de los módulos, structs, traits, funciones, etc. que se crean o se editan**.

## Flujo para la documentación

1. Documentar el código: structs, enums, traits, funciones, métodos, constantes, etc.
2. Documentar los módulos y el crate raíz.

## Comentarios de código (`//` y `/* */`)

### Principio fundamental

- Los comentarios de código (no de documentación) explican el **por qué**, nunca el qué ni el cómo.
- El código debe ser suficientemente claro para explicar qué hace y cómo lo hace.

### Cuándo comentar

Comentar solo cuando:

- Se explica una decisión de diseño no obvia.
- Se documentan efectos secundarios importantes.
- Se advierte sobre limitaciones o casos especiales.
- Se clarifica código complejo que no puede simplificarse más.
- Se proporciona contexto sobre decisiones de negocio.
- Se referencia un ticket, documentación o discusión relevante.
- Variables locales cuyo propósito no sea obvio por el nombre y el contexto.

### Cuándo NO comentar

- El comentario repite lo que el código ya dice claramente.
- El código podría reescribirse para ser más claro.
- La información está desactualizada o es incorrecta.
- El comentario es obvio o redundante.

## Actualización de documentación en refactorizaciones

Cualquier cambio en el código implica:

1. Revisar y actualizar los doc comments del código modificado.
2. Actualizar la cabecera del módulo si cambia su responsabilidad o sus dependencias.
3. Ejecutar `cargo doc` para verificar que no hay enlaces rotos ni advertencias nuevas.

## Formato por tipo

### Crate (raíz del crate — `lib.rs` / `main.rs`)

Usar `//!` (inner doc comments) en la primera línea del archivo:

```rust
//! Primera línea: descripción concisa de para qué sirve este crate.
//!
//! Explicación más completa: qué problema resuelve, cuándo usarlo,
//! ventajas frente a alternativas. Incluir un ejemplo de uso real
//! que el usuario pueda copiar y pegar.
//!
//! # Ejemplos
//!
//! ```rust
//! use mi_crate::MiStruct;
//!
//! let resultado = MiStruct::new().operacion();
//! ```
//!
//! # Features
//!
//! Listar aquí las features opcionales del crate si las hay.
```

### Módulo

```rust
//! Primera línea: qué es este módulo.
//!
//! Explicación completa del módulo: funcionalidades, casos especiales
//! y ejemplos si es necesario.
```

### Constantes y variantes de enumerados

```rust
/// La constante X define | sirve para | es ...
pub const MI_CONSTANTE: u32 = 42;

pub enum Estado {
    /// Descripción de la variante Activo.
    Activo,
    /// Descripción de la variante Inactivo.
    Inactivo,
}
```

### Structs e interfaces (traits)

```rust
/// Primera línea: qué es esta struct / trait.
///
/// Más detalles si aplica (ver "Cuándo comentar").
pub struct MiStruct { ... }

/// Primera línea: qué contrato establece este trait.
pub trait MiTrait { ... }
```

### Implementaciones de traits (`impl Trait for Type`)

```rust
impl MiTrait for MiStruct {
    /// Ver [`MiTrait::mi_metodo`].
    fn mi_metodo(&self) { ... }
}
```

> Usar intra-doc links con la sintaxis `` [`Tipo::metodo`] `` para que rustdoc genere el hipervínculo automáticamente.

### Funciones y métodos

Estructura recomendada por rustdoc:

```rust
/// Primera línea: qué hace (frase corta, sin punto final).
///
/// Descripción detallada incluyendo el comportamiento de cada parámetro
/// y el valor de retorno si no es obvio por la firma.
///
/// # Ejemplos
///
/// ```rust
/// let resultado = mi_funcion(param1, param2);
/// assert_eq!(resultado, valor_esperado);
/// ```
///
/// # Panics
///
/// Describe cuándo esta función puede entrar en pánico, si aplica.
///
/// # Errors
///
/// Si devuelve `Result`, describe los posibles errores.
///
/// # Safety
///
/// Si es `unsafe`, explica los invariantes que el llamador debe garantizar.
pub fn mi_funcion(param1: T, param2: U) -> V { ... }
```

## Secciones estándar en doc comments

| Sección      | Cuándo usarla                                               |
| ------------ | ----------------------------------------------------------- |
| `# Ejemplos` | Siempre que sea posible; incluir código compilable          |
| `# Panics`   | Cuando la función puede entrar en pánico en casos concretos |
| `# Errors`   | Cuando devuelve `Result<_, E>`                              |
| `# Safety`   | Cuando la función o el bloque es `unsafe`                   |

## Ejemplos en documentación (doctests)

Los bloques de código en los doc comments son **tests automáticos** ejecutados con `cargo test`. Seguir estas reglas:

- **Preferir código que compile y se pueda ejecutar** frente a pseudocódigo.
- **Ocultar boilerplate** con `#` al inicio de la línea (visible en el test, invisible en la doc):

```rust
/// ```rust
/// # fn main() -> Result<(), std::num::ParseIntError> {
/// let n = "42".parse::<u32>()?;
/// println!("{} + 10 = {}", n, n + 10);
/// # Ok(())
/// # }
/// ```
```

- Evitar `unwrap()` en ejemplos; ocultar el manejo de errores si complica la lectura.
- Usar `no_run` cuando el ejemplo requiere efectos externos (red, ficheros, etc.):

```rust
/// ```no_run
/// conectar_a_base_de_datos("postgres://localhost/db");
/// ```
```

- Usar `compile_fail` para documentar código que **no** debe compilar:

```rust
/// ```compile_fail
/// let x: u32 = "no soy un número"; // error de tipos
/// ```
```

## Intra-doc links

Enlazar a otros ítems del crate (o de dependencias) directamente en el texto:

```rust
/// Usa [`MiStruct`] internamente. Ver también [`otro_modulo::OtraFuncion`].
/// Para listas, consulta [`Vec`] o [`std::collections::HashMap`].
```

## Elementos que NO deben aparecer en la documentación pública

Usar `#[doc(hidden)]` para ocultar detalles de implementación que forman parte de la API pública por razones técnicas pero no deberían ser usados directamente:

```rust
#[doc(hidden)]
pub fn __internal_helper() { ... }
```

---

# 4. REGLAS PARA TESTING

## Obligatoriedad
Todo código requiere tests. Sin excepciones salvo aprobación del usuario para:
- Infraestructura CI/CD (Jenkins, orquestación, hardware físico)
- Requiere documentación del motivo en código
- Testear partes aislables sigue siendo obligatorio

## Cobertura
**Objetivo: 80-90%** medido por comportamientos, no líneas.

Prioridad:
1. Lógica de negocio crítica
2. Casos de error y fronteras
3. Caminos secundarios

## Tipos
- **Unit**: sin I/O, rápido, determinista
- **Integration**: con dependencias reales (DB, red, FS)

## Qué testear
- Lógica de negocio y validaciones
- Errores y fronteras (vacíos, nulos, límites)
- Contratos públicos
- Bugs corregidos (test primero)

## No se testea
- Getters/setters triviales
- Código generado
- Librerías de terceros

## Tests no son código de Producción
- Claridad contra patrones de diseño
- Funcional contra orientado a objetos
- Duplicar datos OK, duplicar lógica NO
- Sin jerarquías, herencias, ni indirecciones

## Señales de Alarma
- Código sin tests
- Solo camino feliz cubierto
- Tests que nunca fallan
- Dependencia de orden de ejecución

## Fronteras Obligatorias
- Colecciones vacías
- Nulos/ausentes
- Límites exactos de rangos
- Strings vacíos vs espacios
- Overflow si aplica
- Concurrencia si tiene estado compartido

### Estructura
**Arrange / Act / Assert**: sin mezclar
**Independencia total**: orden aleatorio OK
**Sin lógica compleja**: Si necesitas lógica compleja y repetitiva en los test mover a un módulo compartido

## Cuándo usar o no DDT (Data-Driven Testing)

| Situación                              | Patrón                                 |
| -------------------------------------- | -------------------------------------- |
| Un único caso o escenario              | Sin DDT — función de test simple       |
| Dos o más casos, entradas o escenarios | Con DDT — tabla de casos parametrizada |

**Regla:** si estás copiando y pegando un test cambiando solo los datos, usa DDT.

**Utiliza el crate rstest** para crear test DDT. Cada caso siempre identificarlo con el siguiente formato id: "Que se prueba.Que se espera" 

## Diseño de Tests

### Separar lógica pura de efectos secundarios

La lógica de negocio debe poder testearse sin levantar infraestructura. Si un test unitario necesita una base de datos real o una conexión de red, es señal de que la lógica y los efectos están mezclados.

**Acción:** extrae la lógica pura a funciones sin efectos. Los tests unitarios prueban esa lógica. Los tests de integración prueban los adaptadores de infraestructura de forma separada.

### Inyección de dependencias como herramienta de testabilidad

Las dependencias externas (repositorios, servicios, clientes HTTP...) deben poder sustituirse en tests. Si una clase instancia sus dependencias internamente, no puede testearse de forma aislada.

**Señal:** para testear una clase necesitas mockear métodos estáticos o parchear módulos globales. Eso indica que la dependencia no está inyectada.

### Evitar mocks en exceso

Los mocks verifican implementación, no comportamiento. Un test que mockea todo excepto la línea que está probando es frágil y de poco valor.

**Regla:** mockea solo las dependencias que cruzan fronteras del sistema (red, disco, tiempo, servicios externos). La lógica interna se testea sin mocks.
