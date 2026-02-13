# Evaluación módulo 1

Este documento describe el **primer entregable obligatorio del curso**, el cual consta de **dos ejercicios prácticos** orientados a la formulación y solución de problemas de búsqueda y optimización usando técnicas de Inteligencia Artificial.

---

## Objetivo del entregable

Que el estudiante:
- Comprenda cómo **modelar un problema** en términos de estados, acciones, costos y objetivos.
- Implemente y analice algoritmos clásicos de IA (**A\*** y **Algoritmos Genéticos**).
- Trabaje a partir de **descripciones formales** y **código base**, completando y extendiendo las soluciones propuestas.

---

## Ejercicio 1: Búsqueda A\* en un campus universitario

### Descripción

En este ejercicio se debe resolver un problema de navegación dentro de un campus universitario utilizando el algoritmo **A\*** (A-Star).  
El problema incluye:
- Un mapa tipo grilla (estilo *pacman*)
- Estados con posición y piso
- Acciones con diferentes costos
- Una heurística basada en distancia euclídea

### Archivo del ejercicio

- **Descripción del problema**:  
  `ejercicio_astar_campus.md`

Este archivo contiene:
- Definición del espacio de estados
- Acciones y costos
- Heurística
- Condición de objetivo
- Tareas a desarrollar

### Entregable esperado

El estudiante debe:
- Implementar el algoritmo **A\*** para el problema descrito
- Mostrar la ruta encontrada
- Calcular el costo total
- Probar al menos **dos configuraciones distintas** de estado inicial y objetivo

---

## Ejercicio 2: Algoritmo Genético para planificación de cursos

Este ejercicio corresponde a un **único problema**, pero se entrega en **dos archivos complementarios**.

### Descripción

El objetivo es resolver un problema de **planificación de horarios de cursos** usando un **Algoritmo Genético**, teniendo en cuenta:
- Restricciones duras (capacidad, solapamientos, profesores)
- Restricciones blandas (preferencias de horario y uso de salones)
- Diseño de cromosomas, función de fitness y operadores genéticos

### Archivos del ejercicio

- **Descripción detallada del problema y la metodología**:  
  `TUTORIAL_AG_Course_Scheduling.md`

- **Código base (starter notebook)**:  
  `ejercicio_AG_Course_Scheduling.ipynb`

El notebook incluye la estructura inicial del algoritmo y funciones que deben ser completadas o ajustadas.

### Entregable esperado

El estudiante debe:
- Completar la implementación del algoritmo genético
- Explicar brevemente la representación del cromosoma
- Mostrar la evolución del fitness
- Reportar la mejor solución encontrada
- Justificar al menos **una decisión de diseño** (parámetros, mutación, selección, etc.)

---

## Forma de entrega

- Subir los archivos solicitados en el buzón de interactiva
- Incluir:
  - Código funcional
  - Resultados obtenidos
  - Comentarios claros en el código
  - Informe

---

## Importante

- Este entregable es **obligatorio** y constituye la **primera evidencia evaluable del curso**.
- Ambos ejercicios deben resolverse para que el entregable sea considerado completo.

---

## Checklist antes de entregar

- [ ] Leí completamente los archivos `.md`
- [ ] Implementé A\* correctamente
- [ ] Corrí y verifiqué el algoritmo genético
- [ ] Documenté mis decisiones
- [ ] Probé distintos escenarios

---

**Cualquier ambigüedad debe resolverse justificando las decisiones tomadas.**  
La claridad del razonamiento es tan importante como el resultado final.

---

## Criterios de Calificación

El primer entregable tiene una calificación total de **100 puntos**, distribuidos entre los dos ejercicios de la siguiente manera:

---

## Ejercicio 1 – Búsqueda A\* en el campus (40 puntos)

| Criterio | Descripción | Puntaje |
|--------|-------------|---------|
| Modelado del problema | Definición clara de estados, acciones, costos y condición de objetivo, acorde al enunciado | 10 pts |
| Implementación de A\* | Uso correcto de g(n), h(n) y f(n), manejo de frontera y nodos explorados | 15 pts |
| Heurística | Implementación correcta de la heurística (distancia euclídea) y uso consistente | 5 pts |
| Resultados y pruebas | Ejecución correcta con al menos dos escenarios distintos (start/goal) | 5 pts |
| Claridad del código | Código legible, comentado y organizado | 5 pts |
| **Total Ejercicio 1** |  | **40 pts** |

---

## Ejercicio 2 – Algoritmo Genético para planificación de cursos (60 puntos)

| Criterio | Descripción | Puntaje |
|--------|-------------|---------|
| Representación del cromosoma | Definición correcta y coherente con el problema (genes, estructura, significado) | 10 pts |
| Función de fitness | Implementación completa de restricciones duras y blandas según el enunciado | 15 pts |
| Operadores genéticos | Implementación correcta de selección, cruce y mutación | 15 pts |
| Ejecución del algoritmo | Evolución funcional del algoritmo (generaciones, mejora del fitness) | 10 pts |
| Análisis de resultados | Reporte de la mejor solución y breve interpretación del resultado obtenido | 5 pts |
| Claridad y comentarios | Código organizado, comentado y fácil de seguir | 5 pts |
| **Total Ejercicio 2** |  | **60 pts** |

---

## Consideraciones importantes

- Un ejercicio **no entregado o no ejecutable** obtiene **0 puntos** en ese ejercicio.
- Soluciones que **no respeten las restricciones duras** del problema serán penalizadas significativamente.
- El código debe **ejecutar sin errores** usando los archivos entregados.
- Se valora la **coherencia del razonamiento**, incluso si la solución no es óptima.
- No se exige la mejor solución posible, sino una **implementación correcta y bien argumentada**.

---

## Bonificación (hasta +5 pts)

Se podrá otorgar una bonificación adicional por:
- Visualización de resultados
- Comparación de parámetros (ej. tasa de mutación)
- Discusión crítica sobre limitaciones del enfoque

La bonificación **no reemplaza** los requisitos mínimos del entregable.

---
