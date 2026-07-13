# Checklist de Calidad del Plan de Tareas: [NOMBRE DE LA FUNCIONALIDAD]

**Propósito**: Validar la completitud y calidad del plan de tareas antes de pasar a la implementación
**Creado**: [FECHA]
**Plan**: [Enlace a TASKS_FILE]

## Calidad del contenido

- [ ] Sin detalles de implementación en el plan global (clases, funciones, APIs concretas)
- [ ] Los diagramas usan notación UML adecuada
- [ ] Los diagramas no contienen detalles de implementación
- [ ] Todas las secciones obligatorias completadas
- [ ] Todos los puntos del documento son coherentes y tienen relación

## Completitud

- [ ] No quedan marcadores [NECESITA ACLARACIÓN]
- [ ] La sección de Suposiciones está vacía
- [ ] Todos los RF del `spec.md` están cubiertos por al menos una tarea
- [ ] Cada tarea tiene un plan específico suficiente para que otro agente/desarrollador cree un plan de implementación sin ambigüedad
- [ ] Cada tarea tiene asignado un nivel de profundidad (N1, N2 o N3) en su plan específico
- [ ] Los casos de prueba automatizados de los sub-grupos cubren todos los casos límite identificados en el `spec.md`
- [ ] Los casos de prueba automatizados de los sub-grupos cubren todos los criterios de éxito del `spec.md`
- [ ] Los artefactos de prueba definidos en las DU de `spec.md` están reflejados en los casos de prueba automatizados de alguna tarea
- [ ] Las reglas de negocio del `spec.md` no se violan en el plan ni en las tareas
- [ ] Cada tarea que requiere pruebas de aceptación las incluye en la sección opcional correspondiente
- [ ] Todas las solicitudes de cambio en `plan-tasks.md` están resueltos
- [ ] Todos los cambios en `spec-tasks.md` están resueltos

## Consistencia arquitectónica

- [ ] Los patrones de diseño de `bluesprint` se respetan en el plan global y en los planes específicos
- [ ] El diagrama de dependencias cumple estrictamente el formato exigido (único grupo de paralelismo) y no tiene ciclos
- [ ] Las tareas declaradas paralelas no tienen dependencias entre sí (verificado explícitamente)
- [ ] Las tareas paralelas son realmente independientes (no comparten estado mutable sin coordinación explícita)
- [ ] El plan arquitectónico global es internamente coherente con los planes específicos de cada tarea
- [ ] No hay RT contradictorios entre sí

## Sincronización

- [ ] La estructura de carpetas `TASKS_DIR` está sincronizada: existe exactamente una carpeta por cada tarea del documento

## Notas

- Los elementos marcados como incompletos requieren actualización del plan antes de la implementación
- La sección de Suposiciones debe ser eliminada del documento final
