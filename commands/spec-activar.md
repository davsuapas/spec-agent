---
description: Activa una especificación existente para trabajar con ella dado su nombre corto.
model: deepseek/deepseek-v4-flash
agent: general
subtask: true
---

# Entrada del usuario

```text
$ARGUMENTS
```

El argumento **es** el nombre corto de la especificación. Si `$ARGUMENTS` está vacío, responde con:

> "Por favor, proporcióname un nombre corto para activar la especificación, por ejemplo `autenticacion-usuario`."

No continues con el esquema de ejecución hasta que el usuario te proporcione el nombre corto.

---

# Esquema de ejecución

## Paso 1 — Variables de trabajo

- Asigna `SHORT_NAME` = `$ARGUMENTS`.

## Paso 2 — Comprobar si ya existe la especificación

- **Comprueba si existe un sub-directorio que su nombre termina en `-SHORT_NAME` en la carpeta `specs/`.**
- Si **NO existe**, detente y responde con este mensaje, sin modificar nada:

  > "La especificación `[SHORT_NAME]` no tiene un directorio creado en `specs/`.
  > No se puede activar.
  > Si lo que necesitas es crear una funcionalidad nueva, indícame el nuevo nombre, por ejemplo `/spec-crear nuevo-nombre`."

  El flujo termina aquí.

- Si **existe**, asigna el nombre del sub-directorio como `specs/[sub-directorio]` en `SPEC_ACTIVAR` y continúa al Paso 3.

## Paso 3 — Activar la especificación

- Persiste `SPEC_ACTIVAR` como valor en la clave `feature_directory` dentro del fichero `.spec/feature.json`:

## Paso 4 — Resumen y próximos pasos

Muestra un resumen muy parecido (se profesional) a el usuario, el cual te propongo a continuación. 
> ✅ Especificación activada correctamente
>
> 📁 Directorio de la funcionalidad: ``SPEC_ACTIVAR``
>
> Próximos pasos
>
> Ahora indica a el agente que trabajo quieres que realice sobre la especificación activa
