# Tutorial: Algoritmo Genético para Planificación de Horarios de Cursos

## Introducción

Este tutorial te guiará paso a paso para implementar un algoritmo genético que resuelva el problema de **asignar cursos a bloques de tiempo y salones**, respetando múltiples restricciones.

---

## 📋 Entendiendo el Problema

### Datos del Problema

Tenemos:
- **8 cursos** que necesitan ser programados
- **5 bloques de tiempo** disponibles (ej: Lunes 8-10, Lunes 10-12, etc.)
- **3 salones** con diferentes capacidades

**Ejemplo de datos:**

```
Cursos:
  MAT: 45 estudiantes
  FIS: 30 estudiantes
  QUI: 35 estudiantes
  PRG: 40 estudiantes
  LIT: 25 estudiantes
  HIS: 30 estudiantes
  ING: 35 estudiantes
  EDF: 50 estudiantes

Bloques de tiempo:
  1: Lunes 8:00-10:00
  2: Lunes 10:00-12:00
  3: Miércoles 8:00-10:00
  4: Miércoles 10:00-12:00
  5: Viernes 8:00-10:00

Salones:
  A: Capacidad 40 estudiantes
  B: Capacidad 30 estudiantes
  C: Capacidad 50 estudiantes
```

### Restricciones

#### **Restricciones DURAS (deben cumplirse obligatoriamente):**

1. **No solapamiento de salones:** Un salón no puede tener dos cursos al mismo tiempo
   - ✗ Incorrecto: MAT y FIS ambos en Bloque 1, Salón A
   - ✓ Correcto: MAT en Bloque 1, Salón A y FIS en Bloque 2, Salón A

2. **Capacidad suficiente:** El salón debe tener capacidad para los estudiantes
   - ✗ Incorrecto: EDF (50 estudiantes) en Salón B (capacidad 30)
   - ✓ Correcto: EDF (50 estudiantes) en Salón C (capacidad 50)

3. **Profesores compartidos:** Cursos del mismo profesor no pueden ser simultáneos
   - MAT y FIS tienen el mismo profesor
   - QUI y PRG tienen el mismo profesor
   - ✗ Incorrecto: MAT en Bloque 1 y FIS en Bloque 1
   - ✓ Correcto: MAT en Bloque 1 y FIS en Bloque 2

#### **Restricciones BLANDAS (preferibles pero no obligatorias):**

1. **Preferir horarios tempranos:** Bloques 1, 2, 3 son mejores que 4 y 5
2. **Uso eficiente de salones:** Evitar usar salones grandes para cursos pequeños
   - ✗ Poco eficiente: LIT (25 estudiantes) en Salón C (capacidad 50)
   - ✓ Eficiente: LIT (25 estudiantes) en Salón B (capacidad 30)

---

## 🧬 Representación del Cromosoma

### ¿Qué es un Cromosoma en este Problema?

Un **cromosoma** representa una **solución completa**: un horario para todos los cursos.

### Estructura del Cromosoma

El cromosoma es una **lista de 8 tuplas**, donde cada tupla es un **gen** que contiene:
- El **bloque de tiempo** asignado al curso
- El **salón** asignado al curso

```
Cromosoma = [
    Gen_0,  # Información para MAT (curso índice 0)
    Gen_1,  # Información para FIS (curso índice 1)
    Gen_2,  # Información para QUI (curso índice 2)
    Gen_3,  # Información para PRG (curso índice 3)
    Gen_4,  # Información para LIT (curso índice 4)
    Gen_5,  # Información para HIS (curso índice 5)
    Gen_6,  # Información para ING (curso índice 6)
    Gen_7   # Información para EDF (curso índice 7)
]
```

Cada **gen** tiene la forma: `(bloque_tiempo, salón)`

### Ejemplo Concreto

```
Cromosoma ejemplo:
[
    (1, 'A'),  # MAT: Bloque 1, Salón A
    (2, 'B'),  # FIS: Bloque 2, Salón B
    (3, 'C'),  # QUI: Bloque 3, Salón C
    (4, 'A'),  # PRG: Bloque 4, Salón A
    (5, 'B'),  # LIT: Bloque 5, Salón B
    (1, 'C'),  # HIS: Bloque 1, Salón C
    (2, 'A'),  # ING: Bloque 2, Salón A
    (3, 'B')   # EDF: Bloque 3, Salón B
]
```

**Interpretación:**
- MAT se dicta el Lunes 8-10 en el Salón A
- FIS se dicta el Lunes 10-12 en el Salón B
- QUI se dicta el Miércoles 8-10 en el Salón C
- etc.

### Relación Índice-Curso

**MUY IMPORTANTE:** El índice en el cromosoma corresponde directamente al índice del curso en la lista de cursos.

```
Índice 0 → Gen para MAT
Índice 1 → Gen para FIS
Índice 2 → Gen para QUI
Índice 3 → Gen para PRG
...
```

---

## 1️⃣ Crear Individuos (Población Inicial)

### Función: `crear_individuo_aleatorio()`

**Objetivo:** Generar un cromosoma completamente aleatorio.

**Pseudocódigo:**

```
FUNCIÓN crear_individuo_aleatorio():
    CREAR lista_vacía llamada individuo
    
    PARA cada curso desde 0 hasta 7:
        bloque_aleatorio = elegir_aleatoriamente entre [1, 2, 3, 4, 5]
        salón_aleatorio = elegir_aleatoriamente entre ['A', 'B', 'C']
        
        gen = (bloque_aleatorio, salón_aleatorio)
        AGREGAR gen a individuo
    FIN PARA
    
    RETORNAR individuo
FIN FUNCIÓN
```

**Ejemplo de salida:**
```
[(4, 'C'), (1, 'A'), (5, 'B'), (2, 'C'), (3, 'A'), (1, 'B'), (4, 'A'), (2, 'B')]
```

### Función: `crear_poblacion()`

**Objetivo:** Generar múltiples cromosomas aleatorios (población inicial).

**Pseudocódigo:**

```
FUNCIÓN crear_poblacion(tamaño):
    CREAR lista_vacía llamada población
    
    PARA i desde 1 hasta tamaño:
        individuo = crear_individuo_aleatorio()
        AGREGAR individuo a población
    FIN PARA
    
    RETORNAR población
FIN FUNCIÓN
```

**Explicación:**
- Si `tamaño = 20`, generamos 20 cromosomas aleatorios
- Cada cromosoma es una solución potencial diferente
- Esta diversidad inicial es clave para la exploración

---

## 2️⃣ Función de Aptitud (Fitness)

### Objetivo

Evaluar **qué tan buena** es una solución (cromosoma). Asignar un puntaje numérico donde:
- **Mayor puntaje = Mejor solución**
- **Menor puntaje = Peor solución**

### Sistema de Puntuación

```
Puntaje Base: 200 puntos

PENALIZACIONES DURAS (violaciones graves):
  - Conflicto de salón: -50 puntos
  - Capacidad insuficiente: -50 puntos
  - Profesores simultáneos: -50 puntos

PENALIZACIONES BLANDAS (preferencias):
  - Bloque tardío: -5 puntos
  - Salón muy grande: -3 puntos
```

### Pseudocódigo Completo

```
FUNCIÓN fitness(individuo):
    puntaje = 200  // Empezar con puntaje base
    
    // ============================================================
    // RESTRICCIÓN DURA 1: Conflictos de salón
    // ============================================================
    PARA i desde 0 hasta 7:
        PARA j desde i+1 hasta 7:
            bloque_i, salón_i = individuo[i]
            bloque_j, salón_j = individuo[j]
            
            SI bloque_i == bloque_j Y salón_i == salón_j:
                puntaje = puntaje - 50  // Dos cursos en mismo bloque y salón
            FIN SI
        FIN PARA
    FIN PARA
    
    // ============================================================
    // RESTRICCIÓN DURA 2: Capacidad de salones
    // ============================================================
    PARA i desde 0 hasta 7:
        bloque, salón = individuo[i]
        nombre_curso, num_estudiantes = cursos[i]
        capacidad_salón = capacidades[salón]
        
        SI num_estudiantes > capacidad_salón:
            puntaje = puntaje - 50  // Salón muy pequeño
        FIN SI
    FIN PARA
    
    // ============================================================
    // RESTRICCIÓN DURA 3: Profesores compartidos
    // ============================================================
    // MAT (índice 0) y FIS (índice 1) comparten profesor
    bloque_MAT, salón_MAT = individuo[0]
    bloque_FIS, salón_FIS = individuo[1]
    SI bloque_MAT == bloque_FIS:
        puntaje = puntaje - 50  // Mismo profesor en dos lugares
    FIN SI
    
    // QUI (índice 2) y PRG (índice 3) comparten profesor
    bloque_QUI, salón_QUI = individuo[2]
    bloque_PRG, salón_PRG = individuo[3]
    SI bloque_QUI == bloque_PRG:
        puntaje = puntaje - 50  // Mismo profesor en dos lugares
    FIN SI
    
    // ============================================================
    // RESTRICCIÓN BLANDA 1: Bloques tardíos
    // ============================================================
    PARA cada (bloque, salón) en individuo:
        SI bloque >= 4:  // Bloques 4 o 5 son tardíos
            puntaje = puntaje - 5
        FIN SI
    FIN PARA
    
    // ============================================================
    // RESTRICCIÓN BLANDA 2: Uso eficiente de salones
    // ============================================================
    PARA i desde 0 hasta 7:
        bloque, salón = individuo[i]
        nombre_curso, num_estudiantes = cursos[i]
        capacidad_salón = capacidades[salón]
        
        SI num_estudiantes < 35 Y capacidad_salón > 45:
            puntaje = puntaje - 3  // Desperdicio de espacio
        FIN SI
    FIN PARA
    
    RETORNAR puntaje
FIN FUNCIÓN
```

### Ejemplos de Evaluación

**Ejemplo 1: Cromosoma con conflicto**

```
Individuo: [(1, 'A'), (1, 'A'), (3, 'C'), ...]
                      ^^^^^^^ MAT y FIS en mismo bloque y salón

Cálculo:
  Puntaje base: 200
  Conflicto salón (MAT-FIS): -50
  ... otras penalizaciones ...
  Puntaje final: ~100 (bajo, mala solución)
```

**Ejemplo 2: Cromosoma casi perfecto**

```
Individuo: [(1, 'A'), (2, 'B'), (3, 'C'), (1, 'B'), (2, 'C'), (3, 'A'), (1, 'C'), (2, 'A')]

Cálculo:
  Puntaje base: 200
  Sin conflictos: 0
  Sin violaciones capacidad: 0
  Sin profesores simultáneos: 0
  Sin bloques tardíos: 0
  Sin desperdicios: 0
  Puntaje final: 200 (¡perfecto!)
```

---

## 3️⃣ Operador de Cruce (Crossover)

### Objetivo

Combinar dos cromosomas padres para crear un cromosoma hijo que herede características de ambos.

### Estrategia: Cruce de Un Punto

El cruce de un punto funciona así:
1. Elegir un punto de corte aleatorio en el cromosoma
2. Tomar la primera parte del Padre 1
3. Tomar la segunda parte del Padre 2
4. Unir ambas partes para formar el hijo

### Pseudocódigo

```
FUNCIÓN crossover(padre1, padre2):
    // Elegir punto de corte entre 1 y 7
    punto_corte = número_aleatorio_entre(1, 7)
    
    // Tomar genes 0 hasta punto_corte-1 del padre 1
    parte1 = padre1[0 : punto_corte]
    
    // Tomar genes desde punto_corte hasta el final del padre 2
    parte2 = padre2[punto_corte : 8]
    
    // Combinar ambas partes
    hijo = concatenar(parte1, parte2)
    
    RETORNAR hijo
FIN FUNCIÓN
```

### Ejemplo Visual Detallado

```
Padre 1: [(1,'A'), (2,'B'), (3,'C'), (4,'A'), (5,'B'), (1,'C'), (2,'A'), (3,'B')]
          Gen 0    Gen 1    Gen 2    Gen 3    Gen 4    Gen 5    Gen 6    Gen 7

Padre 2: [(2,'C'), (3,'A'), (4,'B'), (5,'C'), (1,'A'), (2,'B'), (3,'C'), (4,'A')]
          Gen 0    Gen 1    Gen 2    Gen 3    Gen 4    Gen 5    Gen 6    Gen 7

Punto de corte = 4

┌─────────────────────────────┬───────────────────────────────┐
│  Tomar de Padre 1 (0-3)     │  Tomar de Padre 2 (4-7)       │
└─────────────────────────────┴───────────────────────────────┘

Hijo:    [(1,'A'), (2,'B'), (3,'C'), (4,'A'), (1,'A'), (2,'B'), (3,'C'), (4,'A')]
          └──────────── De P1 ──────────┘  └──────────── De P2 ──────────┘
          
          MAT    FIS    QUI    PRG     LIT    HIS    ING    EDF
```

### ¿Qué hereda el hijo?

- **Genes 0-3 (MAT, FIS, QUI, PRG):** Del Padre 1
  - MAT: Bloque 1, Salón A
  - FIS: Bloque 2, Salón B
  - QUI: Bloque 3, Salón C
  - PRG: Bloque 4, Salón A

- **Genes 4-7 (LIT, HIS, ING, EDF):** Del Padre 2
  - LIT: Bloque 1, Salón A
  - HIS: Bloque 2, Salón B
  - ING: Bloque 3, Salón C
  - EDF: Bloque 4, Salón A

### ¿Por qué funciona?

✅ **Hereda bloques buenos:** Si el Padre 1 tiene una buena asignación para los primeros cursos, el hijo la mantiene.

✅ **Combina estrategias:** El Padre 2 puede tener buenas asignaciones para los últimos cursos.

✅ **Explora nuevas combinaciones:** El hijo es diferente a ambos padres, permitiendo explorar el espacio de soluciones.

---

## 4️⃣ Operador de Mutación

### Objetivo

Introducir **variación aleatoria** en el cromosoma para:
- Explorar nuevas soluciones
- Evitar quedarse atrapado en soluciones subóptimas
- Mantener diversidad en la población

### Estrategia: Mutación por Gen

Para cada gen del cromosoma, con cierta probabilidad (ej: 20%), cambiar aleatoriamente su bloque, salón, o ambos.

### Pseudocódigo

```
FUNCIÓN mutate(individuo, probabilidad_mutación):
    // Para cada gen del cromosoma
    PARA i desde 0 hasta 7:
        // Generar número aleatorio entre 0 y 1
        aleatorio = número_aleatorio()
        
        // Solo mutar si el número es menor que la probabilidad
        SI aleatorio < probabilidad_mutación:
            bloque_actual, salón_actual = individuo[i]
            
            // Decidir qué mutar
            tipo_mutación = elegir_aleatoriamente([1, 2, 3])
            
            SI tipo_mutación == 1:
                // OPCIÓN 1: Cambiar solo el bloque
                nuevo_bloque = elegir_aleatoriamente([1, 2, 3, 4, 5])
                individuo[i] = (nuevo_bloque, salón_actual)
                
            SINO SI tipo_mutación == 2:
                // OPCIÓN 2: Cambiar solo el salón
                nuevo_salón = elegir_aleatoriamente(['A', 'B', 'C'])
                individuo[i] = (bloque_actual, nuevo_salón)
                
            SINO:
                // OPCIÓN 3: Cambiar ambos
                nuevo_bloque = elegir_aleatoriamente([1, 2, 3, 4, 5])
                nuevo_salón = elegir_aleatoriamente(['A', 'B', 'C'])
                individuo[i] = (nuevo_bloque, nuevo_salón)
            FIN SI
        FIN SI
    FIN PARA
    
    RETORNAR individuo
FIN FUNCIÓN
```

### Ejemplo de Mutación

```
Original: [(1,'A'), (2,'B'), (3,'C'), (4,'A'), (5,'B'), (1,'C'), (2,'A'), (3,'B')]

Probabilidad de mutación: 20%

Proceso:
  Gen 0 (MAT): aleatorio=0.75 > 0.20 → No mutar
  Gen 1 (FIS): aleatorio=0.15 < 0.20 → SÍ MUTAR
    - Tipo mutación: 1 (cambiar bloque)
    - Cambiar de (2, 'B') a (5, 'B')
  
  Gen 2 (QUI): aleatorio=0.89 > 0.20 → No mutar
  Gen 3 (PRG): aleatorio=0.05 < 0.20 → SÍ MUTAR
    - Tipo mutación: 2 (cambiar salón)
    - Cambiar de (4, 'A') a (4, 'C')
  
  Gen 4-7: No mutan

Mutado:  [(1,'A'), (5,'B'), (3,'C'), (4,'C'), (5,'B'), (1,'C'), (2,'A'), (3,'B')]
                    ^^^^^^             ^^^^^^
                    cambió             cambió
```

### Efecto de la Tasa de Mutación

**Probabilidad baja (5%):**
- Pocas mutaciones
- Cambios pequeños
- Mayor explotación de soluciones actuales
- ⚠️ Riesgo: Quedarse atrapado en óptimos locales

**Probabilidad media (20%):**
- Balance entre exploración y explotación
- Suficiente variación sin destruir buenas soluciones
- ✅ Recomendado para la mayoría de casos

**Probabilidad alta (50%):**
- Muchas mutaciones
- Cambios drásticos
- Mayor exploración
- ⚠️ Riesgo: Destruir buenas soluciones, búsqueda casi aleatoria

---

## 5️⃣ Selección

### Objetivo

Elegir qué cromosomas pueden "reproducirse" (ser padres) basándose en su fitness.

### Estrategia: Selección por Torneo

**Idea:** 
1. Elegir aleatoriamente un pequeño grupo de cromosomas
2. El mejor de ese grupo gana el torneo
3. El ganador se convierte en padre

**Ventaja:** Simple y eficiente, da más oportunidades a los mejores sin ignorar completamente a los peores.

### Pseudocódigo

```
FUNCIÓN selección_torneo(población, tamaño_torneo):
    // Elegir aleatoriamente 'tamaño_torneo' cromosomas
    participantes = elegir_aleatoriamente(población, tamaño_torneo)
    
    // Encontrar el mejor de los participantes
    mejor = NULO
    mejor_fitness = -INFINITO
    
    PARA cada participante en participantes:
        fit = fitness(participante)
        SI fit > mejor_fitness:
            mejor = participante
            mejor_fitness = fit
        FIN SI
    FIN PARA
    
    RETORNAR mejor
FIN FUNCIÓN
```

**Ejemplo con tamaño de torneo = 3:**

```
Población: [Ind1, Ind2, Ind3, Ind4, Ind5, ...]
            Fit:  120   145   100   180   150

Torneo 1: Elegimos aleatoriamente Ind2, Ind4, Ind5
          Fitness: 145, 180, 150
          Ganador: Ind4 (fitness 180)

Torneo 2: Elegimos aleatoriamente Ind1, Ind3, Ind5
          Fitness: 120, 100, 150
          Ganador: Ind5 (fitness 150)
```

---

## 6️⃣ Ciclo Evolutivo Completo

### Algoritmo Genético General

```
FUNCIÓN algoritmo_genético(tamaño_población, número_generaciones):
    // Paso 1: Crear población inicial
    población = crear_poblacion(tamaño_población)
    
    // Paso 2: Evolucionar por múltiples generaciones
    PARA generación desde 1 hasta número_generaciones:
        
        // Crear nueva población vacía
        nueva_población = []
        
        // Generar nueva generación
        MIENTRAS tamaño(nueva_población) < tamaño_población:
            
            // Seleccionar dos padres
            padre1 = selección_torneo(población, 3)
            padre2 = selección_torneo(población, 3)
            
            // Crear hijo mediante cruce
            hijo = crossover(padre1, padre2)
            
            // Aplicar mutación
            hijo = mutate(hijo, 0.20)
            
            // Agregar hijo a nueva población
            AGREGAR hijo a nueva_población
        FIN MIENTRAS
        
        // Reemplazar población antigua con nueva
        población = nueva_población
        
        // Mostrar progreso
        mejor = encontrar_mejor(población)
        IMPRIMIR "Generación", generación, "Mejor fitness:", fitness(mejor)
    FIN PARA
    
    // Paso 3: Retornar mejor solución encontrada
    mejor_final = encontrar_mejor(población)
    RETORNAR mejor_final
FIN FUNCIÓN
```

### Flujo Visual del Algoritmo

```
┌─────────────────────────────────────────┐
│  1. POBLACIÓN INICIAL                   │
│  Crear 20-30 cromosomas aleatorios      │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  2. EVALUACIÓN                          │
│  Calcular fitness de cada cromosoma     │
└──────────────┬──────────────────────────┘
               │
               ▼
       ┌───────────────┐
       │ BUCLE POR     │
       │ GENERACIONES  │
       └───────┬───────┘
               │
        ┌──────▼──────────────────────────┐
        │  2.1 SELECCIÓN                  │
        │  Torneo para elegir padres      │
        └──────┬──────────────────────────┘
               │
        ┌──────▼──────────────────────────┐
        │  2.2 CRUCE                      │
        │  Combinar padres → hijo         │
        └──────┬──────────────────────────┘
               │
        ┌──────▼──────────────────────────┐
        │  2.3 MUTACIÓN                   │
        │  Introducir variación           │
        └──────┬──────────────────────────┘
               │
        ┌──────▼──────────────────────────┐
        │  2.4 NUEVA GENERACIÓN           │
        │  Reemplazar población           │
        └──────┬──────────────────────────┘
               │
               │ ¿Más generaciones?
               │ Sí → Volver a 2.1
               │ No ↓
               ▼
┌─────────────────────────────────────────┐
│  3. RESULTADO FINAL                     │
│  Retornar mejor cromosoma encontrado    │
└─────────────────────────────────────────┘
```

---

## 7️⃣ Mejoras Opcionales

### Elitismo

**Idea:** Preservar los mejores cromosomas de una generación a la siguiente.

```
FUNCIÓN evolución_con_elitismo(población, elite_tamaño):
    // Ordenar población por fitness (mejor primero)
    población_ordenada = ordenar_por_fitness(población)
    
    // Guardar los mejores
    élite = población_ordenada[0:elite_tamaño]
    
    // Crear nueva generación
    nueva_población = élite  // Empezar con la élite
    
    MIENTRAS tamaño(nueva_población) < tamaño_población:
        // ... proceso normal de selección, cruce, mutación ...
    FIN MIENTRAS
    
    RETORNAR nueva_población
FIN FUNCIÓN
```

**Ventaja:** Garantiza que las mejores soluciones no se pierdan.

### Mutación Inteligente

**Idea:** En lugar de mutar aleatoriamente, intentar corregir violaciones específicas.

```
FUNCIÓN mutación_inteligente(individuo):
    // Detectar violaciones
    SI tiene_conflicto_salón(individuo):
        // Cambiar salón del curso que causa conflicto
        cambiar_salón_conflictivo(individuo)
    FIN SI
    
    SI tiene_violación_capacidad(individuo):
        // Cambiar a un salón más grande
        cambiar_a_salón_mayor(individuo)
    FIN SI
    
    SI tiene_profesor_simultáneo(individuo):
        // Cambiar bloque de uno de los cursos
        cambiar_bloque_conflictivo(individuo)
    FIN SI
    
    RETORNAR individuo
FIN FUNCIÓN
```

---

## 📊 Resumen: Diferencias con Job Shop Scheduling

| Aspecto | Job Shop Scheduling | Course Scheduling |
|---------|---------------------|-------------------|
| **Problema** | Ordenar operaciones en máquinas | Asignar cursos a bloques/salones |
| **Cromosoma** | Orden de ejecución | Asignaciones directas |
| **Gen** | `(job, operación)` | `(bloque, salón)` |
| **Precedencias** | ✓ Sí (op1 antes op2) | ✗ No |
| **Crossover** | Order Crossover complejo | Cruce de un punto simple |
| **Complejidad** | Alta | Media |
| **Validación** | Difícil (verificar orden) | Fácil (cualquier asignación) |

---

## 🎯 Checklist de Implementación

Cuando implementes el código, asegúrate de:

- [ ] **Representación correcta:**
  - Cromosoma = lista de 8 tuplas
  - Cada tupla = (bloque, salón)
  - Índice corresponde al curso

- [ ] **Fitness completa:**
  - Puntaje base 200
  - Penalizar conflictos de salón (-50)
  - Penalizar capacidad insuficiente (-50)
  - Penalizar profesores simultáneos (-50)
  - Penalizar bloques tardíos (-5)
  - Penalizar desperdicio de salones (-3)

- [ ] **Crossover funcionando:**
  - Elegir punto de corte aleatorio
  - Combinar partes de ambos padres
  - Hijo tiene exactamente 8 genes

- [ ] **Mutación apropiada:**
  - Probabilidad configurable (20%)
  - Cambiar bloque, salón, o ambos
  - No destruir estructura del cromosoma

- [ ] **Selección por torneo:**
  - Elegir 2-3 individuos
  - Retornar el mejor

- [ ] **Ciclo evolutivo:**
  - Crear población inicial
  - Iterar por generaciones
  - Selección → Cruce → Mutación
  - Reemplazar población

---

## 💡 Consejos Finales

1. **Empieza simple:** Implementa primero la versión básica, luego añade mejoras.

2. **Prueba incrementalmente:** Verifica cada función antes de pasar a la siguiente.

3. **Visualiza resultados:** Imprime el mejor cromosoma cada 10 generaciones para ver el progreso.

4. **Ajusta parámetros:** 
   - Tamaño de población: 20-50
   - Generaciones: 50-200
   - Probabilidad de mutación: 0.15-0.30

5. **Valida soluciones:** Verifica que las restricciones duras se cumplan.

6. **Experimenta:** Prueba diferentes configuraciones y observa qué funciona mejor.

---

**¡Buena suerte implementando tu algoritmo genético!** 🚀
