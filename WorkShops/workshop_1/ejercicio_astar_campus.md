# EJERCICIO DE CLASE: Navegación en un campus universitario usando A\* (A-Star)

Un agente (robot/estudiante) se encuentra en un campus universitario con **bloques 30–38**, **Cafetería** y **Biblioteca**, conectados por caminos tipo *pacman* (corredores en una grilla 2D).  
Su tarea es ir desde un **estado inicial** hasta un **estado objetivo** con el **menor costo posible**, usando **A\*** con una **heurística de distancia euclídea**.

---

## 1) Plano 2D estilo “pacman” (grilla con caminos)

La grilla usa coordenadas *(x, y)*. Los edificios están ubicados en nodos específicos del mapa y los pasillos conectan esos puntos.

**Coordenadas de edificios (nodos especiales):**

| Lugar | Coordenada (x, y) |
|------|-------------------|
| B30 | (2, 2) |
| B31 | (5, 2) |
| B32 | (8, 2) |
| B33 | (11, 2) |
| B34 | (14, 2) |
| B35 | (4, 6) |
| B36 | (8, 6) |
| B37 | (12, 6) |
| B38 | (16, 6) |
| Cafetería | (6, 10) |
| Biblioteca | (14, 10) |

> Nota: El mapa tiene pasillos “abiertos” que conectan estos puntos; los demás espacios se consideran “pared” (no transitables).

---

## 2) Definición de estado

El estado del agente está dado por:

```python
state = {
  "edificio": str,      # 'B30'...'B38', 'Cafetería', 'Biblioteca' o 'Pasillo'
  "piso": int,          # 1..8
  "coordenada": (x, y)  # tupla con la posición en la grilla
}
```

Regla: el valor **edificio** se determina así:
- Si (x, y) coincide con la coordenada de un edificio → edificio = nombre del edificio
- En otro caso → edificio = 'Pasillo'

---

## 3) Ascensores y pisos
Los edificios **B32, B33, B35, B37 y B38** tienen ascensor con pisos **1 a 3**.

Los demás edificios no tienen ascensor (pero sí escaleras).

---

## 4) Acciones disponibles

Las acciones permiten movimientos en el plano y cambios de piso:

1. **Moverse en el plano (costo 1):**
   - `arriba` / `abajo` / `izquierda` / `derecha`  
   *(solo si la celda destino es transitable)*

2. **Ascensor (solo en B32, B33, B35, B37, B38):**
   - `subir_ascensor` costo **3**
   - `bajar_ascensor` costo **3**

3. **Escaleras (en cualquier edificio):**
   - `subir_escaleras` costo **7 + p**, donde **p** es el numero de pisos a subir
   - `bajar_escaleras` costo **5**

---

## 5) Costos (Action Costs)

- Movimientos simples (plano): **1**
- Subir / bajar en ascensor: **3**
- Bajar escaleras: **5**
- Subir escaleras: **7 + p** (p = numero de pisos a subir)

---

## 6) Heurística (A\*)

La heurística está dada por la **distancia euclídea** entre la posición actual y la posición objetivo en el plano:

```python
h(n) = sqrt((x - x_goal)^2 + (y - y_goal)^2)
```

> Importante: para este ejercicio, **la heurística NO incluye el piso**. (Esto hace que A\* tenga que “descubrir” los costos de cambios de piso vía g(n)).

---

## 7) Nodo inicial y nodo objetivo (Goal)

- **Nodo inicial**: cualquier estado válido:
  ```python
  start = {"edificio": "...", "piso": 1..3, "coordenada": (x, y)}
  ```

- **Nodo objetivo**: cualquier estado objetivo válido:
  ```python
  goal = {"edificio": "...", "piso": 1..3, "coordenada": (x_goal, y_goal)}
  ```

**Condición de objetivo (goal test):**

El estado actual es objetivo si:
- `coordenada == goal["coordenada"]` **y**
- `piso == goal["piso"]`

---

## 8) Tarea del estudiante

Construir un agente que, usando **A\***, encuentre el **mejor path** (secuencia de acciones) que minimice el costo total.

Las pruebas deben permitir variar:
- el **nodo inicial**
- el **nodo objetivo**
- el criterio de evaluación del path (se mantiene la misma función de costo; solo cambian start/goal)

---

## 9) Salida esperada

El programa debe imprimir al menos:
- Ruta (lista de acciones)
- Secuencia de estados (opcional pero recomendado)
- Costo total acumulado

Ejemplo (formato sugerido):

```
Ruta: ['derecha','derecha','subir_escaleras', ...]
Costo total: 47
```
## 10) Mapa de referencia

Campus = [
# x:  0    1    2     3    4     5     6    7    8     9    10   11    12   13   14    15   16
    [" ", "#", " ",  " ", "#",  " ",  " ", "#", " ",  " ", "#", " ",  " ", "#", " ",  " ", " "],  # y=0
    [" ", "#", " ",  " ", "#",  " ",  " ", "#", " ",  " ", "#", " ",  " ", "#", " ",  " ", " "],  # y=1

    [" ", " ", "B30"," ", " ", "B31"," ", " ", "B32"," ", " ", "B33"," ", " ", "B34"," ", " "],  # y=2

    [" ", "#", " ",  "#", "#", " ",  "#", " ", "#",  "#", " ", "#",  " ", "#", " ",  "#", " "],  # y=3
    [" ", "#", " ",  " ", " ", " ",  "#", " ", "#",  " ", " ", "#",  " ", "#", " ",  "#", " "],  # y=4

    [" ", " ", " ",  "#", " ", " ",  " ", " ", " ",  "#", " ", " ",  " ", " ", " ",  "#", " "],  # y=5

    [" ", "#", " ",  " ", "B35"," ",  "#", " ", "B36"," ", "#", " ",  "B37"," ", "#", " ", "B38"], # y=6

    [" ", "#", "#",  " ", "#", " ",  "#", " ", "#",  " ", "#", " ",  "#", " ", "#",  " ", " "],  # y=7
    [" ", " ", " ",  " ", "#", " ",  " ", " ", "#",  " ", " ", " ",  "#", " ", " ",  " ", " "],  # y=8

    ["#", "#", " ",  "#", "#", " ",  "#", " ", " ",  " ", "#", " ",  "#", " ", "#",  "#", " "],  # y=9

    [" ", " ", " ",  " ", "#", " ",  "C", " ", "#",  " ", "#", " ",  " ", " ", "B",  " ", " "],  # y=10

    [" ", "#", "#",  " ", "#", " ",  "#", " ", "#",  " ", "#", " ",  "#", " ", "#",  " ", " "],  # y=11
    [" ", " ", " ",  " ", " ", " ",  " ", " ", " ",  " ", " ", " ",  " ", " ", " ",  " ", " "],  # y=12

    [" ", "#", " ",  "#", " ", "#",  " ", "#", " ",  "#", " ", "#",  " ", "#", " ",  "#", " "],  # y=13
    [" ", "#", " ",  " ", " ", " ",  " ", " ", " ",  " ", " ", " ",  " ", " ", " ",  "#", " "],  # y=14

    [" ", " ", " ",  "#", "#", "#",  " ", "#", "#",  "#", " ", "#",  "#", "#", " ",  " ", " "],  # y=15
]


