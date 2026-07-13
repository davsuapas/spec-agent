---
description: Solicitar registro de cambios detectados en el documento predecesor (plan → tasks, tasks → spec).
model: deepseek/deepseek-v4-flash
agent: general
subtask: true
---

# Entrada del usuario

```text
$ARGUMENTS
```

El argumento **es** una descripción del cambio a registrar. Si `$ARGUMENTS` está vacío, responde con:

> "Por favor, proporcione una descripción del cambio a registrar."

No continues con el esquema de ejecución hasta que el usuario te proporcione la descripción.

---

# Esquema de ejecución

## Paso 1 — Variables de trabajo

- Asigna `DESCRIPCION` = `$ARGUMENTS`.
- `SPECIFY_FEATURE_DIRECTORY`: se obtiene de `.spec/feature.json` (campo `feature_directory`). Ejemplo: `specs/001-mi-funcionalidad`.

## Paso 2 — Identificar el agente activo

- Pregunta al agente activo de qué agente se trata. Para ello, puedes buscar en su system prompt de su memoria (no en ficheros), la cadena `- Nombre agente:` y extraer el valor que aparece a continuación.
- Almacena el valor obtenido en la variable `AGENTE` (ej. `@spec/plan`, `@spec/tasks`).

## Paso 3 — Validar que el agente sea el esperado

- Si `AGENTE` **no es** `@spec/plan` ni `@spec/tasks`, responde con el siguiente mensaje y termina la ejecución:

> "Este comando solo puede ser ejecutado por los agentes @spec/plan o @spec/tasks."

## Paso 4 — Registrar el cambio

- Determina la ruta del fichero de cambios según el agente:
  - Si `AGENTE` = `@spec/plan`, la ruta es `{SPECIFY_FEATURE_DIRECTORY}/cambios/plan_tasks.md`
  - Si `AGENTE` = `@spec/tasks`, la ruta es `{SPECIFY_FEATURE_DIRECTORY}/cambios/tasks_spec.md`
- Genera la entrada de cambio utilizando el siguiente formato (adaptando la línea de acción según el agente detectado y el flujo de precedencias):
  ```markdown
  ### [FECHA] - [HORA (HH:MM:SS)]
  - **Descripción del cambio**: [DESCRIPCION]
  - **Estado**: Pendiente
  ```
- Añade esta entrada al **final** del fichero correspondiente. Si el fichero no existe, créalos.
- **No elimines** entradas anteriores. El fichero es acumulativo.

## Paso 5 — Resumen

- Confirma la operación mostrando un resumen de lo registrado y la ruta utilizada.