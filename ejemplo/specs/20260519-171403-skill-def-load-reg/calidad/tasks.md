# Checklist de Calidad del Plan de Tareas: Definición, Carga y Registro de Skills

**Propósito**: Validar la completitud y calidad del plan de tareas antes de pasar a la implementación
**Creado**: 2026-06-29
**Plan**: [tasks.md](../tasks.md)

## Calidad del contenido

- [x] Sin detalles de implementación en el plan global (lenguajes, APIs concretas)
- [x] Los diagramas usan notación UML adecuada
- [x] Los diagramas no contienen detalles de implementación
- [x] Todas las secciones obligatorias completadas
- [x] Todos los puntos del documento son coherentes y tienen relación

## Completitud

- [x] No quedan marcadores [NECESITA ACLARACIÓN]
- [x] La sección de Suposiciones está vacía
- [x] Todos los RF del spec están cubiertos por al menos una tarea
- [x] Cada tarea tiene un plan específico suficiente para que otro agente/desarrollador cree un plan de implementación sin ambigüedad
- [x] Cada tarea tiene asignado un nivel de profundidad (N1, N2 o N3) en su plan específico
- [x] Los casos de prueba automatizados de los sub-grupos cubren todos los casos límite identificados en el spec
- [x] Los casos de prueba automatizados de los sub-grupos cubren todos los criterios de éxito del spec
- [x] Los artefactos de prueba definidos en las DU están reflejados en los casos de prueba automatizados de alguna tarea
- [x] Las reglas de negocio del spec no se violan en el plan ni en las tareas
- [x] Cada tarea que requiere pruebas de aceptación las incluye en la sección opcional correspondiente
- [x] Todos los cambios en `spec-tasks.md` están resueltos

## Consistencia arquitectónica

- [x] Los patrones de diseño de `bluesprint` se respetan en el plan global y en los planes específicos
- [x] El diagrama de dependencias cumple estrictamente el formato exigido (único grupo de paralelismo) y no tiene ciclos
- [x] Las tareas declaradas paralelas no tienen dependencias entre sí (verificado explícitamente)
- [x] Las tareas paralelas son realmente independientes (no comparten estado mutable sin coordinación explícita)
- [x] El plan arquitectónico global es internamente coherente con los planes específicos de cada tarea
- [x] No hay RT contradictorios entre sí

## Sincronización

- [x] La estructura de carpetas `tasks/` está sincronizada: existe exactamente una carpeta por cada tarea del documento

## Coherencia técnica

- [x] Los RT no contradicen `doc/reglas-globales-tecnicas.json` (validado con `@claron`)

## Notas

- Los elementos marcados como incompletos requieren actualización del plan antes de la implementación
- La sección de Suposiciones debe ser eliminada del documento final
