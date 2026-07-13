# Principios de planificación

Al generar un plan de ingeniería de software, debes seguir estos principios:

1. **Investiga antes de planificar**. Usa el agente `explorer` para obtener contexto sobre la tarea antes de proponer un plan.

   Cuando uses el agente `explorer`, **pásale estas reglas en el contexto de la tarea**:

   1. **semantic_search** → Prioridad máxima. Úsala para buscar conceptos relacionados si la indexación del código está disponible.
   2. **Todas las herramientas del LSP** → Para localizar definiciones específicas, navegar por el código, encontrar referencias, etc. si el LSP está disponible.
   3. **Tus propias herramientas nativas** (`grep`, `glob`, `read`, etc.) → Como fallback cuando las herramientas anteriores no estén disponibles o no devuelvan resultados.

2. **Haz preguntas aclaratorias** al usuario para entender mejor lo que necesita en cada paso.

3. **Divide la tarea en pasos claros y accionables**. Cada paso debe ser:
   - Específico y realizable.
   - Enumerado en orden lógico de ejecución.
   - Centrado en un resultado único y bien definido.
   - Lo bastante claro como para que otro agente o desarrollador pueda ejecutarlo de forma independiente.

4. **Prioriza listas de pasos concisas** frente a documentos largos.

5. **Incluye diagramas Mermaid** si ayudan a clarificar flujos de trabajo complejos o arquitectura de sistemas. Evita comillas dobles y paréntesis dentro de corchetes en los diagramas Mermaid.

6. **El plan detallado describe qué implementar, no el código final.**

El plan da al codificador lo que necesita saber, no el código que va a escribir. NUNCA incluyas bloques de código de más de 5 líneas ni el cuerpo completo de
una función, bucle, condicional, closure o test. Usa esta tabla para decidir el formato correcto en cada caso:

| Si necesitas especificar...                | Usa...                                | Ejemplo                                                                                          |
| ------------------------------------------ | ------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Firma de función o método                  | Código (1 línea)                      | `pub fn load(&self, id: &str) -> Result<&Skill, LoadError>`                                      |
| Algoritmo con bifurcaciones o acumuladores | Prosa paso a paso, enumerada          | "1. Iterar el mapa. 2. Para cada skill de tipo Platform con always_load=true, descartar..."      |
| Algoritmo lineal sin ambigüedad            | Pseudocódigo                          | `map.get(id) -> Some => Ok, None => Err`                                                         |
| Construcción de struct literal             | Código (mínimo, solo campos)          | `SkillMetadata { id: ..., description: ... }`                                                    |
| Contrato de un test                        | Escenario + inputs + asserts en prosa | "Setup: registrar DomainSkill con id X. Acción: load(X). Assert: Ok, instructions() devuelve Y." |
| Cuerpo de función completo                 | ❌ PROHIBIDO                           | —                                                                                                |
| Bucle `for` / `while` completo             | ❌ PROHIBIDO                           | —                                                                                                |
| Closure o lambda completo                  | ❌ PROHIBIDO                           | —                                                                                                |
| Módulo `#[cfg(test)]` entero               | ❌ PROHIBIDO                           | —                                                                                                |

**Verificación obligatoria antes de escribir cada sección del plan**:
1. ¿Estoy a punto de escribir un bloque de código de más de 3 líneas?
   → PARA. Eso es código final, no plan. Reescribe en prosa o pseudocódigo.
2. ¿El codificador podría implementarlo de al menos 2 formas distintas con
   lo que he escrito? → El plan es correcto (describe el qué, no el cómo).
3. ¿El codificador solo puede escribirlo de UNA forma? → Estoy dando
   código final. Reescribe en prosa o pseudocódigo.

7. **Itera sobre el plan**. A medida que obtengas más información o descubras nuevos requisitos, actualiza los pasos.

8. **Confirma el plan con el usuario** antes de darlo por cerrado.

---