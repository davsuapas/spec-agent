# Especificación de Funcionalidad: Definición, Carga y Registro de Skills

**Creada**: 2026-05-19
**Estado**: Finalizada
**Descripción original**: "Definición y construcción de skills (Builder + Markdown), carga en conversaciones (gestor de metadatos e instrucciones), y registro de skills de core y dominio a través del agente."

---

## Historia de Usuario *(obligatoria)*

### Definir, registrar y cargar skills en un agente

Como **desarrollador** (ya sea del core de Carisa o de dominio), quiero **definir skills con una estructura común (`id`, `title`, `description`, `instructions`), registrarlos en un agente y disponer de un gestor que exponga sus metadatos y permita cargar sus instrucciones bajo demanda** durante la conversación del agente. Esto me permite dotar al agente de capacidades especializadas y reutilizables sin que el contenido completo de los skills ocupe espacio en el contexto inicial ni en el estado de Temporal.

Los skills pueden ser de dos tipos:
- **Skills de plataforma (core)**. Definidos internamente por Carisa. Algunos llevan la propiedad `always_load: true` y se cargan automáticamente al iniciar la sesión. Los desarrolladores de dominio no pueden modificar ni registrar skills de este tipo.
- **Skills de dominio**. Definidos por el desarrollador de dominio. Solo se cargan bajo demanda durante la sesión. No pueden declararse con `always_load`.

El skill es la unidad fundamental de conocimiento reutilizable en Carisa. Sin esta funcionalidad, el agente no puede disponer de capacidades más allá de su planificador base.

**Por qué es prioritaria**: Es la pieza fundacional sobre la que se construye toda la extensibilidad del agente. Define el contrato (`id`, `title`, `description`, `instructions`), los mecanismos de construcción (Builder y markdown), la carga (gestor de metadatos e instrucciones) y el registro (core vs. dominio). El resto de funcionalidades de Carisa —tools, memoria, ciclo de conversación— dependen de que los skills existan y estén correctamente definidos.

**Escenarios de Aceptación**:

1. **Dado** que el desarrollador usa el patrón Builder con todas las propiedades (`id`, `title`, `description`, `instructions`), **Cuando** invoca `.build()`, **Entonces** obtiene una instancia de `Skill` válida con todas las propiedades correctamente asignadas.
2. **Dado** una cadena markdown con frontmatter YAML válido que contiene `id`, `title`, `description` y un cuerpo con las instrucciones, **Cuando** se invoca el constructor desde markdown, **Entonces** se devuelve un `Skill` con las propiedades extraídas del frontmatter y las instrucciones del cuerpo.
3. **Dado** una cadena markdown con frontmatter mal formado (YAML inválido, propiedades obligatorias ausentes, delimitadores `---` incorrectos, o cuerpo de instrucciones vacío), **Cuando** se invoca el constructor desde markdown, **Entonces** se devuelve un error que identifica el `id` del skill (si estaba presente) y describe el problema de formato.
4. **Dado** un agente con N skills registrados, **Cuando** el gestor de skills recupera los metadatos de carga (catálogo ligero: `id` + `description`), **Entonces** devuelve los metadatos de todos los skills del agente excepto aquellos con la propiedad `always_load: true`. Si no hay skills, devuelve una lista vacía sin error.
5. **Dado** un skill registrado con un `id` concreto, **Cuando** el gestor recibe una solicitud de carga por ese `id`, **Entonces** devuelve las instrucciones completas del skill.
6. **Dado** un `id` que no corresponde a ningún skill registrado en el agente, **Cuando** el gestor recibe una solicitud de carga por ese `id`, **Entonces** devuelve un error indicando que el skill con ese `id` no existe.
7. **Dado** que el core de Carisa necesita registrar skills de plataforma, **Cuando** los registra internamente a través del agente (de 0 a N skills), **Entonces** los skills quedan disponibles para esa sesión. El desarrollador de dominio no tiene acceso a este mecanismo de registro.
8. **Dado** un desarrollador de dominio que ha definido skills, **Cuando** los registra en el agente mediante la API pública (de 0 a N skills), **Entonces** los skills quedan disponibles para carga bajo demanda en esa sesión.

### Casos Límite

- **CL-01 — Skill de dominio sin acceso a `always_load`**: Los skills de dominio no tienen acceso a la propiedad `always_load: true` en ningún caso. Esta propiedad solo existe en la definición de skills de plataforma. La estructura de datos y la API de dominio deben garantizar que un skill de dominio nunca pueda declararse con esta propiedad.
- **CL-02 — Registro con ID duplicado**: Si se intenta registrar un skill con un `id` que ya existe en el agente, el registro se ignora silenciosamente (no se registra, no se sobrescribe, no se emite error).
- **CL-03 — Markdown con cuerpo vacío**: Si el markdown tiene frontmatter correcto pero el cuerpo de instrucciones está vacío o solo contiene whitespace, el constructor desde markdown devuelve error.
- **CL-04 — Propiedades obligatorias ausentes en Builder**: Si el desarrollador omite una propiedad obligatoria al usar el Builder, el sistema debe fallar preferiblemente en tiempo de compilación. Si no es posible, debe fallar en tiempo de ejecución al invocar `.build()`.
- **CL-05 — Agente sin skills**: Un agente puede no tener skills de dominio (solo skills de plataforma con `always_load: true`). El gestor debe funcionar correctamente en ese escenario.

---

## Requisitos *(obligatorio)*

### Requisitos Funcionales

- **RF-001**: El sistema DEBE permitir definir un `Skill` de plataforma con las propiedades: `id` (String), `title` (String), `description` (String), `instructions` (String) y `always_load` (bool).
- **RF-002**: El sistema DEBE permitir definir un `Skill` de dominio con las propiedades: `id` (String), `title` (String), `description` (String) e `instructions` (String). La propiedad `always_load` NO DEBE existir en la definición de skills de dominio.
- **RF-003**: El sistema DEBE proporcionar un patrón Builder (usando `derive_builder` de Rust) para construir skills de forma programática especificando sus propiedades.
- **RF-004**: El sistema DEBE proporcionar un constructor que, a partir de una cadena markdown con frontmatter YAML delimitado por `---`, construya un `Skill` extrayendo `id`, `title` y `description` del frontmatter, y las `instructions` del cuerpo (texto libre, sin subdivisiones).
- **RF-005**: El constructor desde markdown DEBE devolver error cuando: el YAML del frontmatter sea inválido, falten propiedades obligatorias, los delimitadores `---` sean incorrectos, o el cuerpo de instrucciones esté vacío o solo contenga whitespace. El error DEBE identificar el `id` del skill si estaba presente en el frontmatter e incluir el `title` como ayuda contextual.
- **RF-006**: El sistema DEBE proporcionar un gestor de skills que recupere los metadatos de carga (`id` + `description`) de todos los skills del agente, excluyendo aquellos con `always_load: true`. La `description` es el campo que el agente usa en la conversación para decidir si necesita cargar un skill. Si no hay skills que listar, DEBE devolver una lista vacía sin error.
- **RF-007**: El gestor de skills DEBE permitir obtener las instrucciones completas de un skill a partir de su `id`. Si el `id` no corresponde a ningún skill registrado, DEBE devolver un error.
- **RF-008**: El sistema DEBE permitir al core de Carisa registrar skills de plataforma de forma interna (no accesible al desarrollador de dominio). Se admite de 0 a N skills.
- **RF-009**: El sistema DEBE proporcionar una API pública en el agente para que el desarrollador de dominio registre skills de dominio. Se admite de 0 a N skills.
- **RF-010**: Al registrar un skill con un `id` ya existente en el agente, el sistema DEBE ignorar el registro silenciosamente (sin sobrescribir ni emitir error).

### Entidades Clave

- **Skill (Plataforma)**: Unidad de conocimiento definida internamente por el core de Carisa. Propiedades: `id` (identificador único), `title` (título legible, usado como ayuda contextual en errores), `description` (descripción breve que el agente usa en la conversación para decidir si carga el skill), `instructions` (texto libre con las instrucciones para el LLM), `always_load` (bool, indica si se carga automáticamente al iniciar sesión). Solo el core puede instanciar y registrar skills de este tipo.
- **Skill (Dominio)**: Unidad de conocimiento definida por el desarrollador de dominio. Propiedades: `id` (identificador único), `title` (título legible, usado como ayuda contextual en errores), `description` (descripción breve que el agente usa en la conversación para decidir si carga el skill), `instructions` (texto libre con las instrucciones para el LLM). No tiene acceso a la propiedad `always_load`. Se registra a través de la API pública del agente y solo se carga bajo demanda.
- **Gestor de Skills**: Componente que mantiene los skills registrados para un agente y expone dos operaciones: catálogo de metadatos (excluyendo skills con `always_load: true`) y carga de instrucciones por `id`. El catálogo proporciona `id` + `description` para que el agente decida qué skills cargar bajo demanda.
- **Agente** *(entidad externa)*: Contenedor de skills sobre el que se realiza el registro. El gestor de skills opera dentro del contexto de un agente. La definición completa del agente no forma parte de esta especificación.

### No incluido (Non‑Goals)

- **NG-001**: La definición y carga de tools dentro de un skill. Las tools se abordarán en otra especificación.
- **NG-002**: El ciclo de conversación del agente (MACRO/MICRO Activities, boundaries, interacción con Temporal).
- **NG-003**: Los bindings a otros lenguajes (Python, JS, Go, .NET). Aunque las decisiones de diseño deben permitir bindings futuros, su implementación queda fuera de esta especificación.
- **NG-004**: La persistencia de skills (base de datos, sistema de ficheros, backend remoto). El gestor de skills recibe los skills ya cargados en memoria. La carga desde origen externo es responsabilidad de otros módulos.
- **NG-005**: El versionado de skills. La evolución y control de versiones de los skills queda fuera de este módulo.

---

## Unidades Demostrables *(obligatorio)*

### DU-001: Definición y construcción de Skills

**Propósito**: El desarrollador puede definir un skill (de plataforma o de dominio) usando el patrón Builder, y también puede construirlo a partir de una cadena markdown con frontmatter. Los errores de formato en el markdown se detectan y se asocian al `id` del skill.

**Requisitos cubiertos**: RF-001, RF-002, RF-003, RF-004, RF-005

**Artefacto de prueba**: Tests en Rust que demuestren:
- Construcción de un `Skill` de plataforma con todas las propiedades mediante Builder.
- Construcción de un `Skill` de dominio mediante Builder (sin acceso a `always_load`).
- Construcción de un `Skill` desde una cadena markdown válida.
- Construcción fallida desde markdown con YAML inválido, propiedades ausentes, delimitadores incorrectos y cuerpo vacío, verificando que el error identifica el `id`.

### DU-002: Gestor de Skills — Catálogo y carga

**Propósito**: Dado un conjunto de skills registrados, el gestor expone un catálogo ligero (`id` + `description`) excluyendo los skills `always_load`, y permite cargar las instrucciones completas de un skill por `id`. Si el `id` no existe, devuelve error.

**Requisitos cubiertos**: RF-006, RF-007

**Artefacto de prueba**: Tests que demuestren:
- Recuperación del catálogo con varios skills (verificando que los `always_load` están excluidos).
- Recuperación del catálogo sin skills (lista vacía, sin error).
- Carga de instrucciones de un skill existente por `id`.
- Error al cargar instrucciones de un `id` inexistente.

### DU-003: Registro de Skills en el agente

**Propósito**: Los skills de plataforma se registran internamente por el core; los de dominio se registran mediante la API pública del agente. IDs duplicados se ignoran silenciosamente.

**Requisitos cubiertos**: RF-008, RF-009, RF-010

**Artefacto de prueba**: Tests que demuestren:
- Registro interno de 0 a N skills de plataforma.
- Registro vía API pública de 0 a N skills de dominio.
- Intento de registro de un skill con `id` duplicado → se ignora sin error.
- Verificación de que un skill de dominio no puede tener `always_load`.

---

## Criterios de Éxito *(obligatorio)*

### Resultados Medibles

- **CE-001**: Un desarrollador puede definir y construir un skill completo (con todas sus propiedades) usando el Builder en menos de 10 líneas de código efectivo.
- **CE-002**: Un desarrollador puede definir y construir un skill completo a partir de un fichero markdown con frontmatter sin escribir código Rust adicional.
- **CE-003**: El 100% de los errores de formato en el markdown devuelven un mensaje que identifica el `id` del skill (si está presente) y describe el problema específico encontrado (propiedad ausente, YAML inválido, cuerpo vacío, etc.).
- **CE-004**: El gestor de skills devuelve correctamente el catálogo de metadatos y las instrucciones para cualquier combinación válida de skills registrados (desde 0 hasta N skills de cada tipo).
- **CE-005**: Un skill de dominio no puede, bajo ninguna circunstancia, declararse o comportarse como si tuviera `always_load: true`. Esta restricción queda garantizada estructuralmente (no depende de un chequeo en runtime que pueda olvidarse).

---
