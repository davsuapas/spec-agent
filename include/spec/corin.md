# Corin: Análisis de Coherencia y Extracción de Conocimiento

Compara reglas y conceptos extraídos de un fichero contra la base de conocimiento global del proyecto, resuelve conflictos con el usuario y actualiza los ficheros globales según el ámbito (negocio o técnico).

**Sigue las instrucciones paso por paso**.

---

## Parámetros

Estos parámetros son definidos por el agente primario:

| Parámetro            | Valores                   | Descripción                                                                                                            |
| -------------------- | ------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `CORIN_AMBITO`       | `"negocio"` o `"tecnico"` | Ámbito de validación. Determina qué extrae `@claron`, qué conflictos se procesan y qué ficheros globales se modifican. |
| `CORIN_RUTA_FICHERO` | Ruta absoluta             | Fichero que se va a validar (especificación, plan de tareas, etc.).                                                    |

---

## Paso 1: Invocación del sub-agente @claron

1. Invoca a `@claron` con estos parámetros:
   - `modo`: `{CORIN_AMBITO}`
   - `ruta_fichero`: `{CORIN_RUTA_FICHERO}`
   - `ruta_reglas`: `"doc/reglas-globales-negocio.json"`
   - `ruta_reglas_tecnicas`: `"doc/reglas-globales-tecnicas.json"`
   - `ruta_conceptos`: `"doc/conceptos.json"`

2. Espera el JSON de respuesta.

3. **Si el sub-agente falla** (timeout, error de invocación, JSON mal formado):
   - Informa al usuario:
   > "❌ El validador de coherencia no ha podido completar el análisis en el ámbito {CORIN_AMBITO}. Esto puede deberse a un problema temporal con el índice semántico o con la invocación del sub-agente. La validación **no** se ha dado por finalizada. Por favor, vuelve a intentarlo escribiendo 'validar coherencia' o 'continuar'. Si el problema persiste, revisa la conectividad con el índice de Kilocode."
   - Detén la ejecución inmediatamente. No continúes.

4. **Si responde correctamente**, continúa al Paso 2.

---

## Paso 2: Filtrado del JSON por ámbito

Del JSON devuelto por `@claron`, céntrate exclusivamente en los elementos del ámbito `{CORIN_AMBITO}`:

- **Si `CORIN_AMBITO = "negocio"`**:
  - Arrays a procesar: `reglas_extraidas`, `conceptos_extraidos`.
  - Conflictos a procesar: solo aquellos cuyo `tipo` sea `regla_vs_regla`, `concepto_vs_concepto`, `regla_no_formalizada`, `concepto_no_formalizado`.
  - Ignora `reglas_tecnicas_extraidas` y conflictos de tipo `regla_tecnica_*`.

- **Si `CORIN_AMBITO = "tecnico"`**:
  - Arrays a procesar: `reglas_tecnicas_extraidas`.
  - Conflictos a procesar: solo aquellos cuyo `tipo` sea `regla_tecnica_vs_regla_tecnica`, `regla_tecnica_no_formalizada`.
  - Ignora `reglas_extraidas`, `conceptos_extraidos` y conflictos de ámbito negocio.

---

## Paso 3: Detección de elementos nuevos sin conflicto

Revisa los arrays que estás procesando para detectar elementos marcados como `es_nueva: true` / `es_nuevo: true` que **no** aparezcan referenciados en ningún conflicto del ámbito actual.

Si encuentras alguno, preséntalo al usuario:

> "📋 Además de los conflictos, he detectado [N] elementos nuevos en el ámbito {CORIN_AMBITO} que no existen en los ficheros globales y no presentan conflictos con nada existente:"
>
> - **[Tipo de elemento]**: [enunciado o definición] — [contexto]
>
> "¿Quieres que los añada a los ficheros globales al finalizar? [Sí, añadir todos] [Revisar uno a uno] [No añadir ninguno]"

- Si elige **"Sí, añadir todos"**: añádelos a la lista de cambios pendientes para el Paso 6.
- Si elige **"Revisar uno a uno"**: preséntalos secuencialmente con el mismo formato de opciones que los conflictos.
- Si elige **"No añadir ninguno"**: no los incluyas en los cambios del Paso 6. Déjalo registrado como "descartado por el usuario".

---

## Paso 4: Resolución de conflictos

### 4a. Sin conflictos

Si el array de conflictos (filtrado por ámbito) está vacío:

> "✅ He contrastado el fichero con las reglas globales de {CORIN_AMBITO}. No se ha detectado ningún conflicto. Todo coherente."

Salta directamente al Paso 6.

### 4b. Con conflictos — Resumen inicial

Agrupa los conflictos por tipo y gravedad, y presenta un resumen:

> "🔍 He analizado el fichero con el conocimiento global del proyecto en el ámbito {CORIN_AMBITO}. Esto es lo que he encontrado:"
>
> - **[N] conflictos**: [desglose por tipo y gravedad: alta, media, baja]
>
> "¿Quieres que los revisemos juntos? [Sí, uno a uno] [Ver la lista completa primero] [Omitir todos]"

- Si elige **"Omitir todos"**: salta al Paso 5 advirtiendo que los conflictos quedan sin resolver y se finaliza sin modificar los ficheros globales.
- Si elige **"Ver la lista completa"**: muestra un resumen numerado de todos los conflictos con su descripción y gravedad, luego vuelve a preguntar si quiere revisarlos uno a uno u omitir.
- Si elige **"Sí, uno a uno"**: procede al Paso 4c.

### 4c. Revisión secuencial por gravedad

Recorre los conflictos **ordenados por gravedad** (altos → medios → bajos). Para cada conflicto:

1. Localiza en los arrays procesados el elemento relacionado con este conflicto para obtener su `contexto`:
   - Para `regla_vs_regla`: busca en `reglas_extraidas` por coincidencia de `enunciado`.
   - Para `concepto_vs_concepto`: busca en `conceptos_extraidos` por coincidencia de `termino`.
   - Para `regla_tecnica_vs_regla_tecnica`: busca en `reglas_tecnicas_extraidas` por coincidencia de `enunciado`.
   - Para `regla_no_formalizada`, `concepto_no_formalizado`, `regla_tecnica_no_formalizada`: el contexto está en el propio elemento extraído.

2. Presenta el conflicto:

```markdown
## Conflicto [N] de [M]: {tipo} — Gravedad: {gravedad}

**Descripción**: {descripcion del JSON del conflicto}

|                   | Enunciado / Definición                                          | Ubicación   |
| ----------------- | --------------------------------------------------------------- | ----------- |
| **Origen**        | {regla_origen.enunciado o concepto_origen.definicion}           | {ubicacion} |
| **Conflicto con** | {regla_conflictiva.enunciado o concepto_conflictivo.definicion} | {ubicacion} |

**Contexto en el fichero**: {contexto del elemento extraído}

**Opciones**:

| Opción | Acción                                                            | Consecuencia                                                  |
| ------ | ----------------------------------------------------------------- | ------------------------------------------------------------- |
| A      | {sugerencia del JSON}                                             | Se actualiza el fichero global correspondiente                |
| B      | Mantener la versión del fichero actual y documentar la diferencia | La regla/concepto del fichero actual se trata como excepción  |
| C      | Mantener ambas y documentar la diferencia                         | Ambas versiones coexisten; se registra en decisiones.md       |
| Omitir | No resolver ahora, dejar marcado como pendiente                   | El conflicto queda sin resolver; se registra en decisiones.md |

**Tu elección**: _[Esperar respuesta]_
```

3. Tras cada respuesta del usuario, **no actualices los ficheros globales todavía**. Solo:
   - Registra la decisión tomada en una lista temporal en memoria.
   - Pasa al siguiente conflicto.

4. Al llegar a conflictos de **gravedad baja**, ofrece la opción de aprobación masiva:

> "Quedan [N] conflictos de gravedad baja (principalmente formalizaciones pendientes). ¿Quieres revisarlos uno a uno o prefieres [A: Añadir todas las reglas/conceptos sugeridos a los ficheros globales] [B: Revisarlos uno a uno] [C: Omitir todos]?"

---

## Paso 5: Resumen de decisiones y confirmación

Antes de modificar ningún fichero global, presenta un resumen de lo acordado:

> "📋 Resumen de decisiones de coherencia en ámbito {CORIN_AMBITO}:"
>
> - Conflicto 1: [acción acordada]
> - Conflicto 2: [acción acordada]
> - ...
> - Conflicto N: [omitido — pendiente para revisión futura]
> - Elementos nuevos sin conflicto: [N] añadidos, [M] descartados
>
> "Esto implica los siguientes cambios en los ficheros globales:"
>
> - [Lista de ficheros y cambios: añadir X reglas, modificar Y conceptos, etc.]
>
> "¿Confirmas que quieres aplicar estos cambios? [Sí, aplicar] [No, revisar de nuevo]"

**Si el usuario confirma**, procede al Paso 6.

**Si el usuario rechaza o pide revisar**, vuelve al Paso 4c con los conflictos que quiera revisar.

---

## Paso 6: Actualización de ficheros globales

Solo si el usuario ha confirmado en el Paso 5 y hay cambios que aplicar.

**Nota**: La estructura detallada de los ficheros globales con el sistema de versionado de reglas y conceptos se describe en el Anexo A. Consúltalo si tienes dudas sobre el formato durante la actualización

### 6a. Lectura de los ficheros actuales

Lee los ficheros globales que correspondan al ámbito `{CORIN_AMBITO}` justo antes de escribir, para evitar sobrescritura de cambios concurrentes:

- **Ámbito `"negocio"`**: `doc/reglas-globales-negocio.json` y `doc/conceptos.json`.
- **Ámbito `"tecnico"`**: `doc/reglas-globales-tecnicas.json`.

### 6b. Aplicación de cambios

#### Para `doc/reglas-globales-negocio.json` y `doc/reglas-globales-tecnicas.json`

- Incrementa el campo `version` semánticamente (cambios nuevos → `minor++`).
- Actualiza `ultima_actualizacion` con la fecha actual.
- **Regla nueva**: crea una entrada con `id` incremental (BR-NNN para negocio, RT-NNN para técnico), `version: 1`, `historial` con la entrada inicial, `fecha_creacion` actual, y `especificaciones_origen` como array incluyendo `{CORIN_RUTA_FICHERO}`.
- **Modificación de regla existente**:
  - Añade `hasta` con la fecha actual a la entrada actual del `historial`.
  - Incrementa `version`.
  - Actualiza `enunciado` / `excepciones` según lo acordado.
  - Añade `{CORIN_RUTA_FICHERO}` al array `especificaciones_origen` si no estaba ya.
  - Añade una nueva entrada en `historial` con los nuevos valores y sin `hasta`.
- **Regla detectada en varios ficheros pero no en globales**: añádela como nueva, incluyendo en `especificaciones_origen` todos los ficheros donde `@claron` la detectó, no solo el actual.

#### Para `doc/conceptos.json`

- Incrementa `version` (`minor++`), actualiza `ultima_actualizacion`.
- **Concepto nuevo**: crea entrada con `id` incremental (C-NNN), `version: 1`, `definicion` consensuada, `sinonimos` si se acordaron, y `especificaciones_que_usan` incluyendo `{CORIN_RUTA_FICHERO}`.
- **Modificación de concepto existente**:
  - Añade `hasta` a la versión actual del `historial`.
  - Incrementa `version`.
  - Actualiza `definicion`.
  - Añade `{CORIN_RUTA_FICHERO}` a `especificaciones_que_usan` si no estaba.
  - Registra la versión anterior en `historial`.

### 6c. Escritura

Escribe los ficheros con los cambios aplicados.

---

## Paso 7: Registro de decisiones

Registra las decisiones en el fichero `SPECIFY_FEATURE_DIRECTORY/checklists/decisiones.md`. Si no existe, créalo con este formato:

```markdown
# Registro de Decisiones de Coherencia

**Creado**: [FECHA]
**Fichero validado**: {CORIN_RUTA_FICHERO}
**Ámbito**: {CORIN_AMBITO}
**Informe de validación**: version {version_informe} generado el {timestamp}

## Decisiones tomadas

- **[FECHA] Conflicto [ID]**: [Descripción breve del conflicto] → [Decisión tomada] — [Justificación]
- ...
```

Si el fichero ya existe, añade las nuevas decisiones al final.

---

## Paso 8: Finalización

1. Informa al usuario:

> "✅ Validación de coherencia completada en ámbito {CORIN_AMBITO}"
>
> "📄 Fichero validado: `{CORIN_RUTA_FICHERO}`"
> "🔍 Coherencia global: [sin conflictos / [N] conflictos resueltos / [N] conflictos omitidos]"

2. Si hubo conflictos omitidos, añade:

> "⚠️ Quedan [N] conflictos sin resolver. Se han registrado en `checklists/decisiones.md` para revisarlos más adelante."


## Anexo A — Estructura de ficheros globales con versionado de reglas y conceptos

### `doc/reglas-globales-negocio.json` y `doc/reglas-globales-tecnicas.json`

```json
{
  "version": "2.3",
  "ultima_actualizacion": "2026-05-09",
  "descripcion": "Reglas de negocio globales del proyecto.",
  "reglas": [
    {
      "id": "BR-001",
      "version": 2,
      "fecha_creacion": "2025-11-01",
      "enunciado": "El email de cada usuario debe ser único en el sistema",
      "excepciones": [
        {
          "tipo": "cuenta_servicio",
          "condicion": "cuentas de servicio",
          "descripcion": "pueden compartir email con una cuenta humana principal"
        }
      ],
      "especificaciones_origen": [
        "specs/001-autenticacion/spec.md",
        "specs/003-integraciones/spec.md"
      ],
      "historial": [
        {
          "version": 1,
          "enunciado": "El email de cada usuario debe ser único en el sistema",
          "excepciones": [],
          "fecha_creacion": "2025-11-01",
          "hasta": "2026-05-09"
        }
      ]
    }
  ]
}
```

### `doc/conceptos.json`

```json
{
  "version": "1.8",
  "ultima_actualizacion": "2026-05-09",
  "descripcion": "Diccionario de conceptos de dominio del proyecto.",
  "conceptos": [
    {
      "id": "C-001",
      "version": 3,
      "termino": "cuenta de servicio",
      "definicion": "Cuenta generada por API, sin usuario humano asociado, utilizada exclusivamente para integraciones entre sistemas. No tiene interfaz gráfica ni requiere autenticación interactiva. Puede pertenecer a múltiples organizaciones simultáneamente.",
      "sinonimos": ["service account", "cuenta de integración"],
      "especificaciones_que_usan": [
        "specs/001-autenticacion/spec.md",
        "specs/003-integraciones/spec.md",
        "specs/007-organizaciones/spec.md"
      ],
      "fecha_creacion": "2025-11-05",
      "historial": [
        {
          "version": 1,
          "definicion": "Cuenta generada por API, sin usuario humano asociado.",
          "fecha_creacion": "2025-11-05",
          "hasta": "2026-01-15"
        },
        {
          "version": 2,
          "definicion": "Cuenta generada por API, sin usuario humano asociado, utilizada exclusivamente para integraciones entre sistemas. No tiene interfaz gráfica.",
          "fecha_creacion": "2026-01-15",
          "hasta": "2026-05-09"
        }
      ]
    }
  ]
}
```

### Reglas de mantenimiento del historial

- El campo `historial` es **inmutable**. Las entradas pasadas no se modifican nunca.
- La versión actual de la regla/concepto es la de mayor número en `historial`.
- Los campos `enunciado` / `definicion` y `excepciones` en la raíz del objeto reflejan **siempre** la versión vigente (la más reciente).
- Al modificar una regla o concepto:
  1. Se añade `hasta` con la fecha actual a la última entrada del `historial`.
  2. Se incrementa `version`.
  3. Se actualizan los campos raíz (`enunciado`, `excepciones`, `definicion`) con los nuevos valores.
  4. Se añade una nueva entrada al `historial` con los nuevos valores y sin `hasta`.
  5. Se añade la especificación activa al array de specs correspondiente si no estaba ya presente.
- `especificaciones_origen` y `especificaciones_que_usan` son arrays acumulativos: una spec no se elimina aunque la regla evolucione, porque esa spec sigue siendo parte del origen histórico.