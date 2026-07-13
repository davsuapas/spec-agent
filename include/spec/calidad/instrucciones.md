# Validación de calidad

Valida la calidad siguiendo estos pasos:

## Crear el checklist de calidad

Genera el fichero `FICHERO_CALIDAD` con el contenido definido en la plantilla `FICHERO_PLANTILLA_CALIDAD`. **Solamente se genera el `FICHERO_CALIDAD`** si se ha definido el parámetro `FICHERO_CALIDAD`. Si no se ha definido mantener en memoria la plantilla.

## Ejecutar la validación

- Antes de revisar el documento ten en cuenta las siguientes reglas:
  - Todo parte de `SPECIFY_FEATURE_DIRECTORY`. Si en algún elemento de la plantilla te indican:
    -  `spec.md` se refieren a `SPEC_FILE`
    -  `tasks.md` se refieren a `TASKS_FILE`.
    -  `spec-tasks.md` se refieren a `CAMBIOS_S_T_FILE`.
    -  `tasks-spec.md` se refieren a `CAMBIOS_T_S_FILE`.
    -  `tasks-plan.md` se refieren a `CAMBIOS_T_P_FILE`.
    -  `plan-tasks.md` se refieren a `CAMBIOS_P_T_FILE`.
- Revisa `FICHERO_CALIDAD_DOCUMENTO` contra cada elemento del checklist que has copiado en `FICHERO_CALIDAD` o la plantilla en memoria.
- Para cada elemento:
  - **Si todo pasa**: marca el checklist con una `x`.
  - **Si hay fallos**: deja el checklist desmarcado.
- Si necesitas añadir alguna nota aclaratoria, añádelas en la sección "## Notas".
