---
description: Crea el fichero de especificación para una funcionalidad a partir de un nombre corto.
model: deepseek/deepseek-v4-flash
agent: general
subtask: true
---

# Entrada del usuario

```text
$ARGUMENTS
```

El argumento **es** el nombre corto de la funcionalidad (formato `accion-sustantivo`, minúsculas y guiones). Si `$ARGUMENTS` está vacío, responde con:

> "Por favor, proporcióname un nombre corto para la funcionalidad, por ejemplo `autenticacion-usuario`."

No continues con el esquema de ejecución hasta que el usuario te proporcione el nombre corto.

---

# Esquema de ejecución

## Paso 1 — Validar el nombre corto

- Verifica que `$ARGUMENTS` tenga entre 2 y 4 palabras separadas por guiones, sin espacios ni caracteres especiales, y que esté en minúsculas.
- Si no cumple el formato, sugiere normalizarlo (p. ej., "Autenticación de Usuario" → `autenticacion-usuario`) y pide confirmación antes de continuar.
- Asigna `SHORT_NAME` = `$ARGUMENTS`.

## Paso 2 — Determinar el directorio de la funcionalidad

**Orden de resolución de `SPECIFY_FEATURE_DIRECTORY`**:

1. Genera el directorio automáticamente bajo `specs/`:
   - El prefijo es `YYYYMMDD-HHMMSS` (timestamp actual).
   - Construye el nombre del directorio: `<prefijo>-<SHORT_NAME>` (p. ej., `20260506-122010-autenticacion-usuario`).
   - Asigna `SPECIFY_FEATURE_DIRECTORY` = `specs/<nombre-directorio>`.

## Paso 3 — Lee el fichero `.spec/feature.json`
- Si *existe*, asigna el valor de la clave "feature_directory" en `FEATURE_DIRECTORY`.
- Si *no existe*, asigna "ninguna" en `FEATURE_DIRECTORY`.

## Paso 3 — Comprobar si ya existe

- **Comprueba si existe un sub-directorio que su nombre termina en `-SHORT_NAME` en la carpeta `specs/`.**
- Si **existe**, detente y responde con este mensaje, sin modificar nada:

  > "La funcionalidad `[SHORT_NAME]` ya tiene un directorio creado en `[SPECIFY_FEATURE_DIRECTORY]`.
  > La feature activa es `FEATURE_DIRECTORY`.
  > Si quieres trabajar con esa especificación existente y es la feature activa, usa el agente de especificaciones directamente sobre ese fichero.
  > Si quieres trabajar con otra especificación que no es la activa, actívala (usa el comando /spec-activar `[SHORT_NAME]`) antes de trabajar con ella.
  > Si lo que necesitas es crear una funcionalidad nueva con otro nombre, indícame el nuevo nombre, por ejemplo `/spec-crear nuevo-nombre`."

  No se creará ningún directorio ni fichero nuevo. El flujo termina aquí.

- Si **no existe**, continúa al Paso 4.

## Paso 4 — Crear directorio y fichero de especificación

- Ejecuta `mkdir -p SPECIFY_FEATURE_DIRECTORY`.
- Crea el fichero `SPECIFY_FEATURE_DIRECTORY/spec.md` como punto de partida con el contenido del Anexo A.
- Asigna `SPEC_FILE` = `SPECIFY_FEATURE_DIRECTORY/spec.md`.
- Persiste la ruta resuelta en `.spec/feature.json`:

```json
{
  "feature_directory": "<directorio resuelto>"
}
```

Escribe el valor real resuelto (p. ej., `specs/003-autenticacion-usuario`), nunca el literal `SPECIFY_FEATURE_DIRECTORY`.

## Paso 5 — Resumen y próximos pasos

Muestra un resumen muy parecido (se profesional) a el usuario, el cual te propongo a continuación. **Es obligatorio** que el directorio de la funcionalidad y el fichero de especificación, salgan en el resumen.

> ✅ Fichero de especificación creado correctamente
>
> 📁 Directorio de la funcionalidad: `SPECIFY_FEATURE_DIRECTORY`
> 📄 Fichero de especificación:      `SPEC_FILE`
>
> ¿Qué acaba de ocurrir?
>
> Has creado la "carpeta contenedora" y el documento base donde se definirá tu funcionalidad. 
> En este momento el fichero está casi vacío; solo contiene la estructura que rellenaremos.
>
> Próximos pasos
>
> Ahora necesitas un **asesor que te guíe** para completar ese documento con lo que realmente necesitas.  
> Ese asesor soy yo en modo agente de especificaciones.
>
>👉 **Acción recomendada**: Invócame como agente de especificaciones, si no es así ya, y cuéntame qué quieres construir.  
> Por ejemplo:
>
> "Quiero crear un pipeline que integre las ventas de los clientes en mi CRM"
>
> Yo iré preguntándote paso a paso y rellenaré el fichero contigo hasta que quede completo y validado.
>
> ¿Continuamos? 😊


# Anexo A

```markdown
# Especificación de Funcionalidad: [NOMBRE DE LA FUNCIONALIDAD]

**Creada**: [FECHA]
**Estado**: Borrador
**Descripción original**: "$ARGUMENTS"

---

## Historia de Usuario *(obligatoria)*

<!--
  Una sola historia de usuario por especificación.
  Debe describir el flujo principal desde la perspectiva del usuario.
  Los escenarios alternativos y casos límite se cubren en las secciones
  de Escenarios de Aceptación y Casos Límite.
-->

### [Título breve de la historia]

[Describe el recorrido del usuario en lenguaje llano. Explica qué quiere hacer, en qué contexto y cuál es el valor que obtiene.]

**Por qué es prioritaria**: [Explica el valor que aporta y por qué es el núcleo de esta funcionalidad]

**Escenarios de Aceptación**:

1. **Dado** [estado inicial], **Cuando** [acción], **Entonces** [resultado esperado]
2. **Dado** [estado inicial], **Cuando** [acción], **Entonces** [resultado esperado]
3. **Dado** [estado inicial], **Cuando** [acción], **Entonces** [resultado esperado]

### Casos Límite

- ¿Qué ocurre cuando [condición de borde]?
- ¿Cómo gestiona el sistema [escenario de error]?
- ¿Qué pasa si [situación excepcional]?

---

## Requisitos *(obligatorio)*

### Requisitos Funcionales *(obligatorio)*

- **RF-001**: El sistema DEBE [capacidad específica, p. ej., "permitir a los usuarios crear una cuenta"]
- **RF-002**: El sistema DEBE [capacidad específica, p. ej., "validar las direcciones de correo electrónico"]
- **RF-003**: Los usuarios DEBEN poder [interacción clave, p. ej., "restablecer su contraseña"]
- **RF-004**: El sistema DEBE [requisito de datos, p. ej., "persistir las preferencias del usuario"]
- **RF-005**: El sistema DEBE [comportamiento, p. ej., "registrar todos los eventos de seguridad"]

<!--
  Ejemplo de cómo marcar requisitos poco claros (máximo 3 en todo el documento):
  - **RF-006**: El sistema DEBE autenticar a los usuarios mediante [NECESITA ACLARACIÓN: método no especificado — email/contraseña, SSO, OAuth?]
-->

### Entidades Clave *(incluir solo si la funcionalidad implica datos)*

- **[Entidad 1]**: [Qué representa, atributos clave sin detalles de implementación]
- **[Entidad 2]**: [Qué representa, relaciones con otras entidades]

### No incluido (Non‑Goals) *(se añade si el usuario explicita exclusiones)*

<!--
   Lista de lo que queda fuera del alcance de esta especificación.
   Se escribe solo si el usuario mencionó algo explícitamente tras la
   pregunta de la Fase 3.
-->

- **NG-001**: [Descripción de la exclusión, p. ej., “no incluye migración de datos históricos”]
- **NG-002**: …

---

## Unidades Demostrables *(obligatorio)*

<!--
  Agrupación de los requisitos funcionales independientes
  que se pueden implementar y demostrar por separado.
-->

### DU-001: [Título de la unidad]

**Propósito**: [Qué logra esta unidad y para quién]

**Requisitos cubiertos**: RF-001, RF-003, ...

**Artefacto de prueba**: [Descripción concreta de lo que se mostrará para
demostrar que esta unidad funciona]

### DU-002: [Título de la unidad]

**Propósito**: [Qué logra esta unidad y para quién]

**Requisitos cubiertos**: RF-002, RF-004, ...

**Artefacto de prueba**: [Descripción concreta de lo que se mostrará para
demostrar que esta unidad funciona]

---

## Criterios de Éxito *(obligatorio)*

### Resultados Medibles

- **CE-001**: [Métrica medible, p. ej., "Los usuarios pueden completar el alta en menos de 2 minutos"]
- **CE-002**: [Métrica medible, p. ej., "El sistema gestiona 1.000 usuarios concurrentes sin degradación"]
- **CE-003**: [Métrica de satisfacción, p. ej., "El 90% de los usuarios completa la tarea principal en el primer intento"]
- **CE-004**: [Métrica de negocio, p. ej., "Se reducen en un 50% los tickets de soporte relacionados con [X]"]

---

## Suposiciones *(obligatorio)*

<!--
  Esta sección recoge únicamente las suposiciones tomadas por el agente
  cuando la descripción de la funcionalidad no especificaba ciertos detalles.
  El usuario debe revisarlas antes de confirmar el spec.
  Si no se modifican, el plan se generará asumiendo que son correctas.
-->

- **[Suposición del agente]**: [Descripción de la suposición] — *Motivo: [por qué el agente tomó esta decisión por defecto]*
- **[Suposición del agente]**: [Descripción de la suposición] — *Motivo: [por qué el agente tomó esta decisión por defecto]*
- **[Suposición del agente]**: [Descripción de la suposición] — *Motivo: [por qué el agente tomó esta decisión por defecto]*
```