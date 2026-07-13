# blueprint: Guía completa y reglas para la arquitectura y diseño estratégico de software

Cubre principios fundamentales, patrones de diseño idiomáticos, anti-patrones a evitar, y el uso de los lenguajes para controlar la complejidad del código.

Al diseñar, o revisar código, sigue estas reglas y patrones para controlar la complejidad y garantizar software idiomático y mantenible.

---

# 1. RELGAS DE DISEÑO DE SOFTWARE

## Principios fundamentales

- El objetivo principal es **controlar la complejidad**, no solo hacer que el código funcione.
- Usa **desarrollo incremental**: el diseño evoluciona con la implementación, no se cierra al inicio.
- Adopta siempre **programación estratégica** sobre táctica.

## Programación estratégica vs. táctica

**Táctico (prohibido):**
- Copiar/pegar código para cumplir plazos.
- Crear funciones específicas por caso (`categorizarZARMadrid()`, `categorizarZARValencia()`).

**Estratégico (obligatorio):**
- Invertir tiempo en diseño limpio desde el principio.
- Crear funciones genéricas y reutilizables (`categorizarZAR(provincia)`).
- Refactorizar antes de duplicar.

## Consistencia

- Nombres de variables, funciones, clases y módulos deben seguir siempre el mismo patrón.
- Funciones relacionadas deben tener firmas similares (mismos nombres de parámetros, mismos tipos de retorno).
- Usa los mismos patrones de diseño para problemas similares en todo el sistema.

## Señales de mal diseño (evitar)

| Síntoma                      | Descripción                                                           | Solución                       |
| ---------------------------- | --------------------------------------------------------------------- | ------------------------------ |
| **Amplificación del cambio** | Un cambio pequeño obliga a modificar múltiples sitios                 | Centralizar la lógica afectada |
| **Carga cognitiva alta**     | El desarrollador necesita recordar demasiado para completar una tarea | Abstraer y documentar          |
| **Incógnitas desconocidas**  | Componentes no documentados que fallan al cambiar otros               | Hacer dependencias explícitas  |

## Dependencias

- Las dependencias deben ser **simples y obvias**.
- Evita que información crítica esté oculta o sea difícil de encontrar.
- Nombra funciones y módulos por lo que hacen, no por cómo lo hacen.

### Gestión de la complejidad

#### Eliminar complejidad innecesaria

- Identificadores consistentes y descriptivos. `processCustomerOrders()` no `processData()`.
- Encapsula lógica repetida en funciones o módulos reutilizables.
- Consolida valores o configuraciones duplicadas en un único lugar.

#### Encapsular complejidad

Diseña módulos que oculten detalles internos y expongan interfaces simples. El llamador no debe gestionar lo que el módulo puede gestionar él mismo.

## Diseño de módulos

### Módulos profundos vs. superficiales

- Un módulo es **profundo** si ofrece mucha funcionalidad a través de una interfaz simple y oculta su complejidad interna.
- Un módulo es **superficial** si obliga al usuario a gestionar detalles internos manualmente.
- Preferir siempre módulos profundos.
- Evitar referencias circulares.

### Ocultación de información

- Los detalles internos de un módulo **nunca deben ser visibles** para otros componentes.
- Las interfaces públicas deben estar claramente definidas para evitar dependencias ocultas entre módulos.

### Propósito general antes que especializado

- Diseñar módulos reutilizables en diferentes contextos antes que especializados en un caso concreto.
- Prefiere interfaces de propósito general cuando el coste es bajo. Una función que soporta varios formatos es más reutilizable que una función por formato.
- Checklist al definir un módulo:
  - ¿Es reutilizable en otros contextos?
  - ¿Oculta detalles innecesarios al usuario?
  - ¿La interfaz es clara y genérica?

### Juntar vs. separar

**Agrupa** componentes cuando comparten responsabilidad o combinarlos simplifica la interfaz.
**Separa** componentes cuando tienen objetivos distintos o evolucionan a ritmos diferentes.

- Prioriza la agrupación siempre que se pueda

## Tirar la complejidad hacia abajo

Las capas superiores deben ser simples. Validaciones, configuraciones y manejo de errores van en los módulos de bajo nivel, no en el llamador.

## Reglas de código

### Comentarios

- Comenta intenciones y decisiones de diseño, nunca lo que el código ya dice.
- Escribe los comentarios antes que el código: fuerza a pensar en el diseño antes de implementar.
- Si un nombre necesita un comentario para aclararse, cambia el nombre.

### Definir errores fuera de existencia

Elimina errores mediante diseño: usa valores por defecto, valida al inicio, o diseña para que el caso de error no sea alcanzable. Una excepción evitada por diseño es mejor que una excepción manejada.

### El código debe ser obvio

El desarrollador debe poder adivinar qué hace el código sin documentación adicional. Señales de que no es obvio: parámetros posicionales sin contexto, nombres ambiguos, comportamiento ante casos edge no evidente.

### Diseño para el rendimiento

- Mide antes de optimizar. No optimices sin un cuello de botella identificado.
- Concentra las optimizaciones en la ruta crítica.
- Elige estructuras de datos adecuadas al uso: búsquedas frecuentes ? diccionario, no lista.

### Modificación de código existente

- Cada modificación es una oportunidad de mejorar el diseño o eliminar duplicados.
- Actualiza siempre los comentarios al modificar código. Un comentario desactualizado es peor que no tener comentario.
- Si detectas deuda técnica, resuélvela en el momento.
- Antes de entregar, revisa que todos los comentarios afectados siguen siendo válidos.

---

# 2. REGLAS PARA ELEGIR Y APLICAR PATRONES DE DISEÑO

> **Prioridad:** [!!] crítico · [!] importante · [ref] referencia · [no] desaconsejado
Ten en cuenta la prioridad a la hora de elegir el patrón

## Reglas de Patrones — Rust

> **Filosofía Rust primero:** Rust tiene un sistema de tipos, ownership, traits y pattern matching que hacen obsoletos muchos patrones OOP clásicos. Antes de aplicar cualquier patrón, pregúntate: *¿el compilador ya resuelve esto?* Si la respuesta es sí, no añadas indirección.

## Reglas Universales

- Identifica el problema concreto antes de aplicar un patrón. Sin problema claro, no hay patrón.
- KISS tiene precedencia sobre cualquier patrón. El mejor código Rust es el más directo.
- **En Rust, los traits son el mecanismo de abstracción principal.** Herencia OOP no existe; composición y traits son el camino por defecto.
- El compilador es tu aliado: si el pattern matching o el borrow checker te fuerzan a un diseño, escúchalos antes de añadir indirección.

## Parte I — Patrones de Código

### [!!] DRY — Don't Repeat Yourself

**Aplica cuando:** la misma lógica aparece en dos o más lugares.
**No apliques si:** el parecido es superficial pero las razones de cambio son distintas.

**En Rust:** extrae a una función libre, un método de trait, o un macro (`macro_rules!`) según el nivel de generalización necesario. Preferir funciones libres sobre macros salvo que la generalización requiera variabilidad sintáctica.

**Señales de violación:**
- Estás copiando un bloque para adaptarlo ligeramente.
- Un bug aparece corregido en un sitio pero no en otro.
- Un cambio de lógica obliga a tocar el mismo algoritmo en varios módulos.

### [!!] SOLID

#### S — Single Responsibility Principle

**Aplica cuando:** defines o revisas un struct, enum o módulo.

**En Rust:** un struct o módulo debe tener una sola razón de cambio. Si un struct acumula métodos que cambian por razones distintas, separa en structs más pequeños (ver *Struct decomposition* en Parte II). Los módulos son la unidad de encapsulación en Rust; úsalos para acotar responsabilidades.

**Señales de violación:**
- El nombre del struct/módulo contiene "y", "también" o es genérico sin acotación (`Manager`, `Helper`, `Utils`).
- Los métodos del struct cambian por razones completamente distintas entre sí.

#### O — Open/Closed Principle

**Aplica cuando:** una nueva variante de comportamiento exige modificar código ya estable.

**En Rust — decisión de implementación:**
- Sin estado → define un trait e implementa una función genérica `fn foo<T: MiTrait>(...)`. No necesitas Strategy con clase.
- Con estado → usa un trait object `Box<dyn MiTrait>` o un enum con variantes.
- Si las variantes son cerradas y conocidas en compilación → **usa un enum con `match` exhaustivo**. El compilador garantiza que no olvidas casos.
- Si las variantes son abiertas o vienen de crates externos → **usa traits** (dyn o genérico).

**Señales de violación:**
- Añadir una variante requiere un `if/else if` en código que ya funciona.
- El struct central crece cada vez que el negocio añade un caso nuevo.

#### L — Liskov Substitution Principle

**Aplica cuando:** defines traits o implementas traits para múltiples tipos.

**En Rust:** LSP se aplica a nivel de traits. Si implementas un trait para un tipo, todas las garantías del trait deben cumplirse. No implementes un trait dejando métodos que hacen `panic!()` o `unimplemented!()` de forma silenciosa.

**Señales de violación:**
- Una implementación de trait hace `panic!()` o `todo!()` en métodos que el caller espera funcionar.
- El código cliente comprueba el tipo concreto con `downcast` antes de llamar a un método polimórfico.

**Acción:** Divide el trait en traits más pequeños (ISP) o usa enums en lugar de herencia de traits.

#### I — Interface Segregation Principle

**Aplica cuando:** defines un trait implementado por varios tipos distintos.

**En Rust:** los traits deben ser pequeños y cohesivos. Rust ya favorece esto (`Iterator`, `Display`, `From`, etc. son traits de un solo método o muy acotados). Componer traits con `where T: TraitA + TraitB` es preferible a un supertrait monolítico.

**Señales de violación:**
- Una implementación concreta deja métodos con `unimplemented!()` o cuerpo vacío.
- Cambios en una parte del trait obligan a modificar implementaciones que no usan esa parte.

#### D — Dependency Inversion Principle

**Aplica cuando:** un módulo de alto nivel instancia directamente un servicio concreto (BD, HTTP, sistema de archivos).

**En Rust — implementación:**
- Inyecta dependencias como parámetros genéricos con trait bounds: `fn process<S: Storage>(store: &S)`.
- Para dependencias en runtime donde el tipo varía: usa `Box<dyn Trait>` o `Arc<dyn Trait>`.
- Evita instanciar servicios concretos dentro de lógica de negocio; recíbelos como parámetros.

**Señales de violación:**
- Para testear una función necesitas instanciar servicios externos reales.
- Cambiar una implementación concreta obliga a modificar el módulo que la usa.

### [!!] KISS — Keep It Simple

**Aplica cuando:** siempre, como filtro antes de introducir cualquier patrón.

**En Rust:** el lenguaje ya es expresivo. Un `match` exhaustivo sobre un enum suele ser más claro que un sistema de objetos polimórficos. Una función libre es más simple que un trait con una sola implementación.

**Señales de violación:**
- Necesitas explicar por qué el diseño es así.
- Hay más indirección (`Box<dyn>`, traits intermedios) que lógica de negocio real.
- Se aplica un patrón "por si acaso" sin problema concreto que lo justifique.

### [!!] POLA — Principio del Menor Asombro

**Aplica cuando:** nombras funciones, métodos, tipos o módulos.

**En Rust:** sigue las convenciones de la comunidad: `new()` para constructores, `into_*` para conversiones que consumen, `as_*` para conversiones baratas que preservan, `to_*` para conversiones costosas. Si rompes estas convenciones, documenta el motivo.

**Señales de violación:**
- Un `get_` modifica estado; un `build_` lanza efectos de red; un `validate_` persiste datos.
- El comportamiento requiere leer la implementación para entenderse dado su nombre.

### [!!] Self-Documentation

**Aplica cuando:** nombras cualquier símbolo: módulo, struct, función, variable, tipo genérico.

**En Rust:** los parámetros de tipo genérico deben ser descriptivos (`T: Formatter` mejor que `T`; `E: Error` es aceptable por convención). Los nombres de variables de un solo carácter solo son aceptables en cierres cortos (`|x| x + 1`) e iteradores convencionales.

**Señales de violación:**
- Abreviaturas ambiguas (`mgr`, `proc`, `d`, `f`) fuera de contexto obvio.
- Un comentario encima de una línea que describe *qué* hace esa línea en lugar del *porqué*.
- Parámetros de tipo genérico llamados `T` cuando tienen un rol semántico claro.

### [!] CRP — Composite Reuse Principle

**Aplica cuando:** eliges cómo reutilizar comportamiento.

**En Rust:** composición por defecto. Incluye structs dentro de otros structs. La herencia OOP no existe; si necesitas compartir comportamiento, usa traits con implementaciones por defecto, no una jerarquía de tipos.

**Señales de aplicación:**
- Necesitas reutilizar lógica de un tipo en otro tipo diferente → contén el primero como campo del segundo.
- Quieres compartir comportamiento entre tipos no relacionados → define un trait con implementación por defecto.

### [!] LoD — Ley de Deméter

**Aplica cuando:** un método accede a la estructura interna de otro objeto.

**En Rust:** respeta los límites de visibilidad (`pub`, `pub(crate)`, privado). Si te encuentras navegando `a.b.c.d`, probablemente `b` debería exponer un método que encapsule esa navegación. El borrow checker además penaliza el acceso profundo en cadena.

**Señales de violación:**
- Cadenas `a.b().c().d()` donde `b`, `c`, `d` no son tipos de `a`.
- Acceder a campos privados o `pub(crate)` desde módulos no relacionados.

### [!] DbC — Diseño por Contratos

**Aplica cuando:** defines funciones públicas con restricciones de uso no evidentes.

**En Rust:** usa el sistema de tipos para hacer imposibles los estados inválidos en lugar de documentar precondiciones en texto. Cuando el tipo no pueda expresarlo, documenta en el docstring con sección `# Panics` (para panics esperados) y `# Safety` (para `unsafe`). Usa `debug_assert!` para verificar invariantes en desarrollo sin coste en producción.

**Preferencia:** un tipo que hace imposible el estado inválido es mejor que un `assert!` en runtime.

## Parte II — Patrones de Diseño en Rust

> **Nota para el agente:** en Rust, muchos patrones GOF clásicos se expresan de forma diferente o directamente con características del lenguaje. La columna "Rust idiomático" es la guía principal.

### Patrones Creacionales

#### [!!] Constructor (`new`)

**Rust idiomático:** usa una función asociada `new()` como convención de constructor. Implementa `Default` cuando todos los campos tienen un valor por defecto razonable. Implementa ambos cuando corresponda.

**Cuándo usar `Default`:** cuando el valor cero/vacío es un estado válido y útil. Permite usar `..Default::default()` para inicialización parcial.

#### [!!] Builder

**Usa cuando:** un struct tiene más de 4-5 campos, especialmente si son opcionales, o cuando la construcción requiere validación.

**Rust idiomático:** el Builder toma y devuelve `self` por valor (encadenamiento fluido). El método `build()` devuelve `Result<T, E>` si hay validación.

**No uses Builder si:** el struct tiene pocos campos y Rust permite inicialización directa con campos nombrados `MyStruct { field: value }` — esa ya es una forma segura y ergonómica sin boilerplate adicional.

**Usa el crate:** derive_builder en vez de implementarlo tu mismo.

#### [!!] Factory Method

**Usa cuando:** el tipo concreto a crear depende de configuración o contexto en runtime, y el cliente no debe conocer el tipo concreto.

**Rust idiomático:** una función que devuelve `Box<dyn Trait>` o un enum. Si las variantes son conocidas en compilación, el enum es preferible porque evita heap allocation y permite pattern matching exhaustivo.

```rust
// Preferible si variantes son conocidas:
enum Formatter { Json(JsonFormatter), Text(TextFormatter) }

// Si variantes son abiertas (plugins, runtime):
fn create_formatter(kind: &str) -> Box<dyn Formatter> { ... }
```

**Señal de necesidad:** el cliente tiene un `match` o `if/else` para instanciar tipos concretos.

#### [!] Abstract Factory

**Usa cuando:** necesitas garantizar compatibilidad entre objetos de una misma familia en runtime.

**Rust idiomático:** un trait que devuelve tipos asociados o `Box<dyn Trait>` para cada producto de la familia. En Rust esto es menos frecuente que en OOP clásico porque los generics resuelven muchos casos en compilación. Intenta usar mejor genéricos con trait bounds.

#### [no] Singleton

#### [no] Prototype

### Patrones Estructurales

#### [!!] Newtype

**Usa cuando:** quieres type safety sobre un tipo primitivo, ocultar un tipo interno, o añadir implementaciones de traits sin conflicto de coherencia.

**Rust idiomático:** tuple struct de un campo. Abstracción zero-cost.

```rust
struct Meters(f64);
struct Seconds(u64);
```

**Úsalo para:** distinguir unidades, restringir la API pública de un tipo, implementar traits de crates externos sobre tipos de crates externos (newtype como solución a la regla de coherencia).

#### [!!] RAII con Guards

**Usa cuando:** un recurso debe liberarse al salir de un scope, independientemente de cómo (retorno normal, `?`, panic).

**Rust idiomático:** implementa `Drop` para el tipo guard. El compilador garantiza que `drop()` se llama al salir del scope. Es el patrón nativo de Rust para manejo de recursos (`MutexGuard`, `File`, conexiones de BD).

**Señal de necesidad:** tienes código de limpieza que debe ejecutarse en múltiples puntos de salida de una función.

#### [!!] Facade

**Usa cuando:** el cliente repite la misma secuencia de llamadas para coordinar un subsistema complejo.

**Rust idiomático:** un módulo público con funciones de alto nivel que orquestan módulos internos (`pub(crate)` o privados). La fachada es el API pública del crate/módulo; los subsistemas son detalles de implementación.

**Señal de necesidad:** el cliente necesita conocer y coordinar múltiples structs/funciones internas para realizar una operación conceptualmente simple.

#### [!!] Adapter

**Usa cuando:** integras una librería externa o código legacy con una interfaz incompatible.

**Rust idiomático:** implementa el trait requerido por tu sistema para el tipo externo. Si no puedes implementar directamente (regla de coherencia), usa Newtype como wrapper y luego implementa el trait.

#### [!] Struct Decomposition (para borrow checker)

**Usa cuando:** un struct grande causa problemas con el borrow checker porque el compilador trata el struct como una unidad aunque solo necesitas borrows de campos independientes.

**Rust idiomático:** descompón en structs más pequeños. El compilador puede entonces prestar campos de structs distintos de forma independiente.

**Señal de necesidad:** el borrow checker rechaza borrows de campos independientes de un mismo struct en el mismo scope.

#### [!] Bridge

**Usa cuando:** una jerarquía crece exponencialmente por mezclar dos dimensiones de variación independientes.

**Rust idiomático:** en lugar de jerarquías de tipos, usa dos traits independientes y composición. O usa un tipo genérico con dos parámetros de tipo.

**Señal de necesidad:** estás creando tipos como `ReportePdfComprimido`, `ReporteHtmlComprimido`, `ReportePdfSinComprimir`...

#### [!] Prefer Small Crates / Módulos

**En Rust:** favorece módulos y crates pequeños con una responsabilidad clara. Cargo hace trivial añadir dependencias con garantías de reproducibilidad. Separar en múltiples crates también permite compilación paralela.

**Señal de necesidad:** un módulo o crate está creciendo con responsabilidades no relacionadas.

#### [!] Contain Unsafety in Small Modules

**Usa cuando:** necesitas código `unsafe` para implementar una abstracción segura.

**Rust idiomático:** encapsula todo el código `unsafe` en el módulo más pequeño posible. Ese módulo expone una API completamente segura. El módulo exterior solo usa código safe.

**Regla:** todo `unsafe` debe tener un comentario `// SAFETY:` explicando por qué es correcto.

#### [ref] Deref para Smart Pointers

**Solo para implementaciones de smart pointers propios.** No uses `Deref` para emular herencia entre structs no relacionados (ver anti-patrón *Deref polymorphism*).

### Patrones de Comportamiento

#### [!!] Strategy

**Usa cuando:** una familia de algoritmos es intercambiable y debe poder variar en runtime o compilación.

**Rust idiomático — decisión:**
- Sin estado, variación en compilación → parámetro de tipo genérico con trait bound: `fn process<F: Formatter>(f: F)`
- Sin estado, variación en runtime → closure inyectada: `fn process(f: impl Fn(&Data) -> String)`
- Con estado complejo, variación en runtime → `Box<dyn Trait>`
- Variantes cerradas y conocidas → enum con `match`

**Señal de necesidad:** un `match` sobre una variable de "tipo de algoritmo" aparece en múltiples métodos.

#### [!!] State

**Usa cuando:** el comportamiento de un tipo cambia radicalmente según su estado y los condicionales para gestionarlo se vuelven complejos.

**Rust idiomático — decisión:**
- **Primera opción:** enum con variantes que llevan datos asociados + `match` exhaustivo. El compilador garantiza que todos los estados están cubiertos. Hace imposibles los estados inválidos.
- Alternativa (estados muy complejos con mucha lógica): structs separados que implementan un trait común.

**Señal de necesidad:** `match` sobre una variable de estado aparece en múltiples métodos de un struct.
**Señal de alarma:** si añadir un estado requiere modificar múltiples métodos.

#### [!!] Observer

**Usa cuando:** un cambio en un objeto debe notificar a otros sin conocerlos directamente.

**Rust idiomático:** usa canales (`std::sync::mpsc`, `tokio::sync::broadcast`) en lugar de listas de callbacks con referencias mutables compartidas. Los canales son más explícitos, seguros en concurrencia, y evitan el problema de notificar a suscriptores en estado inválido.

#### [!!] Command

**Usa cuando:** necesitas undo/redo, encolar operaciones, auditarlas o ejecutarlas de forma diferida.

**Rust idiomático:** en Rust, un Command es simplemente un closure o función almacenada. No necesita una clase propia.
- Sin undo: `Box<dyn Fn()>` o `Vec<fn()>`
- Con undo: un trait `Command` con métodos `execute` y `rollback`, implementado por structs.
- Con estado capturado: `Box<dyn Fn() -> &str>` o closures que capturan su entorno.

#### [!!] Visitor

**Usa cuando:** necesitas añadir operaciones a una estructura de datos sin modificarla, especialmente en ASTs o estructuras de árbol.

**Rust idiomático — decisión:**
- Si los tipos de nodo son un enum → **usa `match` directamente**. No necesitas Visitor.
- Si la jerarquía es abierta (tipos definidos por el usuario) → implementa un trait `Visitor` con métodos `visit_*`.
- Proporciona funciones `walk_*` para el recorrido estándar, separado de las operaciones.

```rust
// Con enum (preferido):
fn interpret(expr: &Expr) -> i64 {
    match expr {
        Expr::IntLit(n) => *n,
        Expr::Add(l, r) => interpret(l) + interpret(r),
        Expr::Sub(l, r) => interpret(l) - interpret(r),
    }
}
```

#### [!] Iterator personalizado

**Usa cuando:** necesitas recorrer una colección con lógica compleja o personalizada sin exponer su estructura interna.

**Rust idiomático:** implementa el trait `Iterator` con `fn next(&mut self) -> Option<Self::Item>`. Una vez implementado, obtienes gratis `map`, `filter`, `take`, `zip`, `chain`, `collect`, etc.

**No implementes Iterator manualmente si:** las primitivas del lenguaje (`iter()`, `iter_mut()`, `into_iter()` sobre colecciones estándar, más adaptadores) ya resuelven tu caso.

#### [!] Mediator

**Usa cuando:** varios componentes se conocen mutuamente y el acoplamiento hace el sistema difícil de modificar.

**Rust idiomático:** usa canales de mensajes para desacoplar. El canal actúa como mediador implícito. Alternativa: un struct mediador que posee las referencias y coordina sin que los componentes se conozcan entre sí.


#### [!] Fold (transformación de estructuras)

**Usa cuando:** necesitas transformar una estructura de datos en otra estructura similar aplicando operaciones en cada nodo.

**Rust idiomático:** un trait `Folder` con métodos `fold_*` que tienen implementaciones por defecto (identidad). Solo sobreescribes los nodos que te interesan. Similar a Visitor pero produce una nueva estructura en lugar de solo observar.


#### [no] Template Method


#### [ref] Chain of Responsibility

**Referencia:** cadena de handlers donde uno procesa la petición. En Rust, implementable con iteradores de closures o un Vec de `Box<dyn Handler>`. Considera si un `match` con múltiples casos es más simple.

#### [ref] Interpreter

**Referencia:** define un DSL y un intérprete. En Rust, considera `macro_rules!` para DSLs que se resuelven en compilación. Para DSLs en runtime, un enum recursivo con `match` es el enfoque idiomático.

## Parte III — Anti-patrones (NO HACER)

### [!!] Clonar para satisfacer al borrow checker

**No hagas esto.** Si `.clone()` existe solo para evitar un error del compilador, el diseño de ownership es incorrecto. Soluciones correctas:

- `mem::take()` / `mem::swap()` — intercambia o extrae el valor sin clonar ni mover el contenedor. Útil cuando necesitas el valor de un campo `mut` sin mover el struct entero.
- `mem::replace()` — sustituye un valor dejando un placeholder en su lugar.
- Reestructurar el código o usar referencias con lifetime explícito.
- Descomponer el struct (ver *Struct Decomposition*).
- **Acortar el lifetime del borrow**: en lugar de tomar prestado un resultado compuesto (`&String`), extrae solo el dato escalar que necesitas (`usize`, `bool`) antes de la mutación. El borrow termina antes y el compilador lo acepta sin clone.

```rust
// MAL: el borrow de dst vive hasta el push
let largest: &String = dst.iter().max_by_key(|s| s.len()).unwrap();
for s in src { if s.len() > largest.len() { dst.push(s.clone()); } }

// BIEN: el borrow termina al extraer el usize
let largest_len: usize = dst.iter().max_by_key(|s| s.len()).unwrap().len();
for s in src { if s.len() > largest_len { dst.push(s.clone()); } }
```

- **Usar un bloque `{}`** para delimitar explícitamente el scope de una referencia mutable y liberar el borrow antes de volver a usar el dato original:

```rust
{
    let tmp = &mut data.field;
    tmp.modify();
} // borrow termina aquí
data.other_field.use(); // ahora es válido
```

### [!!] `#![deny(warnings)]` en código de librería

**No pongas `#![deny(warnings)]` en el código fuente de librerías.** Rompe builds cuando el compilador añade nuevos lints. Usa `RUSTFLAGS="-D warnings"` en CI, o declara lints específicos con `#![deny(...)]` de forma selectiva.


### [!!] Deref Polymorphism (emular herencia con Deref)

**No uses `Deref` para emular herencia entre structs no relacionados.** `Deref` es para smart pointers. Usarlo para herencia es sorprendente, no funciona con trait bounds, rompe con `self`, y es confuso para otros desarrolladores.

```rust
// MAL: usar Deref para heredar de Foo en Bar
impl Deref for Bar { type Target = Foo; ... } // No hagas esto

// BIEN: composición explícita o trait compartido
impl Bar {
    fn m(&self) { self.foo.m() } // explícito y claro
}
```

## Parte IV — Uso Funcional de Rust

Rust soporta programación funcional de forma nativa. Estos patrones son idiomáticos y deben preferirse sobre equivalentes imperativos cuando aumenten la claridad.

### [!!] Iteradores y adaptadores

Prefiere la cadena de adaptadores sobre bucles imperativos cuando exprese mejor la intención. Cuando iteres para construir un valor de retorno derivado de un argumento, **recupera el valor desde el iterador en lugar de clonar** — `args.next()` sobre un `impl Iterator<Item = String>` transfiere ownership sin copia.

### [!!] Option y Result como contenedores

Usa los métodos de `Option` y `Result` (`map`, `and_then`, `unwrap_or`, `ok_or`) en lugar de `match` explícito cuando simplifiquen la expresión. Usa `?` para propagación de errores.

### [!!] Generics como Type Classes

Usa parámetros de tipo genérico con trait bounds para dividir APIs por estado o capacidad en compilación. Esto detecta errores en compilación en lugar de runtime.

### [!] Temporally Mutable → Immutable

Usa rebinding de variable para señalar que un dato ya no debe modificarse tras su inicialización:

```rust
let mut data = prepare();
data.sort();
let data = data; // ahora inmutable, el compilador lo garantiza
```

### [!] `Cow<B>` — clone on write

**Usa cuando:** una función puede devolver datos prestados o propios según una condición, y quieres evitar clonar en el caso frecuente.

**Rust idiomático:** `Cow<str>` o `Cow<[T]>` como tipo de retorno o parámetro. Si el dato no necesita modificación, se devuelve `Borrowed` sin copia; si necesita modificación, `into_owned()` clona solo entonces.

```rust
fn maybe_escape(s: &str) -> Cow<str> {
    if s.contains('<') { Cow::Owned(s.replace('<', "&lt;")) }
    else               { Cow::Borrowed(s) }
}
```

**Señal de necesidad:** tienes un `if` que devuelve `s.to_string()` en un caso y `s` en otro, forzando siempre una copia.

### [!] `MaybeUninit<T>` — inicialización diferida

**Usa cuando:** necesitas reservar un buffer grande o un array sin inicializar para sobreescribirlo inmediatamente (FFI, parsers de bajo nivel, pools de memoria). Evita la inicialización por defecto y mejora el rendimiento cuando solo se rellena una fracción del buffer.

**Regla:** nunca leer de un `MaybeUninit` sin haber llamado antes a `write()`. Todo acceso a `assume_init()` requiere un comentario `// SAFETY:` que argumente que cada byte está inicializado.

```rust
let mut buf: MaybeUninit<[u8; 4096]> = MaybeUninit::uninit();
// SAFETY: llenamos todos los bytes antes de asumir init
let slice = unsafe { (*buf.as_mut_ptr()).as_mut_slice() };
fill(slice);
let buf = unsafe { buf.assume_init() };
```

**No uses si:** el tipo implementa `Default` y la inicialización cero no tiene coste significativo — `MaybeUninit` añade complejidad y riesgo de UB.

### [!] `thread_local!` — estado aislado por hilo

**Usa cuando:** necesitas un estado mutable global por hilo sin sincronización (cachés por hilo, contadores de métricas, buffers de scratch).

**Rust idiomático:** declara con `thread_local! { static X: RefCell<T> = RefCell::new(...); }` y accede con `X.with(|x| ...)`. Cada hilo tiene su propia copia; los cambios no se propagan entre hilos.

**No uses si:** necesitas que los hilos compartan o agreguen el estado — usa `Arc<Mutex<T>>` o canales.

## Parte V — Herramientas y Convenciones

- Usa `thiserror` para definir tipos de error en librerías; evita implementar `Display` y `Error` a mano.
- Usa `crossbeam::channel` en lugar de `std::sync::mpsc` cuando necesites `select!` sobre múltiples canales, timeouts o canales bidireccionales.
- Usa `include_str!()` e `include_bytes!()` para embeber assets en el binario en lugar de leerlos en runtime.
- Usa `dbg!()` durante el desarrollo en lugar de `println!("{:?}", ...)` — imprime fichero, línea y expresión, y devuelve el valor.
- Anota con `#[inline]` funciones pequeñas y críticas en el camino caliente cuando el profiler lo justifique; no lo apliques por defecto.
- Usa `split_at_mut()` para dividir un slice en dos partes mutables independientes sin violar el borrow checker — alternativa a clonar cuando necesitas paralelismo sobre un mismo buffer.
- Prefiere `&str` sobre `&String`, `&[T]` sobre `&Vec<T>`, `&T` sobre `&Box<T>` en parámetros de función. En structs, usa `String` / `Vec<T>` (owned). Devuelve `&str` solo si el slice es parte del parámetro recibido; devuelve `String` si es nuevo.

## Referencia Rápida — Decisiones Clave en Rust

| Situación                                            | Solución Rust idiomática                 |
| ---------------------------------------------------- | ---------------------------------------- |
| Variantes de comportamiento conocidas en compilación | `enum` + `match` exhaustivo              |
| Variantes abiertas / plugins                         | `Box<dyn Trait>`                         |
| Algoritmo intercambiable sin estado                  | closure / fn pointer                     |
| Algoritmo intercambiable con estado                  | `impl Trait` genérico o `Box<dyn Trait>` |
| Recurso que debe liberarse                           | `impl Drop` (RAII)                       |
| Constructor complejo                                 | Builder pattern                          |
| Type safety sobre primitivos                         | Newtype                                  |
| Borrow checker con struct grande                     | Struct decomposition                     |
| Notificación a múltiples consumidores                | Canales (`mpsc`, `broadcast`)            |
| Código unsafe inevitable                             | Encapsula en módulo mínimo con API safe  |
| Herencia OOP                                         | Composición + traits                     |
| Retorno borrowed o owned según condición             | `Cow<B>`                                 |
| Buffer grande sin inicialización por defecto         | `MaybeUninit<T>`                         |
| Estado mutable sin sincronización entre hilos        | `thread_local!`                          |
| Intercambiar valores sin clonar                      | `mem::swap` / `mem::replace`             |
| Borrow demasiado largo bloqueando mutación           | Extrae escalar antes de mutar            |
