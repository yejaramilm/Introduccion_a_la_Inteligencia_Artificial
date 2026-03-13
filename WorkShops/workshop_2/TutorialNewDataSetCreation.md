# Tutorial: Cómo construir un nuevo dataset a partir de señales EMG

> Este tutorial explica de forma sencilla cómo transformar las señales crudas del dataset **Muscle_Fatigue_Cycling** en una nueva base de datos lista para entrenar un modelo de clasificación.

---

## ¿Cómo luce el dataset original?

Imagina que tienes una tabla gigante. Cada fila es una medición tomada en un instante de tiempo muy preciso, y cada columna es un músculo diferente (canal). Se ve algo así:

| Time (s) | Muscle_1 | Muscle_2 | ... | Muscle_8 | Target |
|---|---|---|---|---|---|
| 0.000 | 0.012 | -0.034 | ... | 0.007 | 0 |
| 0.001 | 0.015 | -0.031 | ... | 0.009 | 0 |
| 0.002 | 0.011 | -0.029 | ... | 0.006 | 0 |
| ... | ... | ... | ... | ... | ... |

El equipo que midió las señales lo hizo a **1000 muestras por segundo** (1 kHz). Eso significa que por cada segundo de experimento, hay **1000 filas** en la tabla. Si el experimento duró varios minutos, la tabla puede tener cientos de miles de filas.

Ese volumen de datos es muy difícil de manejar directamente. Además, una sola fila (un instante de 0.001 segundos) no dice mucho sobre si el músculo está fatigado o no. La fatiga se ve en el **comportamiento de la señal a lo largo del tiempo**, no en un único valor aislado.

La solución: **agrupar las filas en ventanas de 1 segundo y resumir cada ventana con unas pocas características.**

---

## Paso 1: Entender qué es una ventana

Imagina que estás leyendo un libro y en lugar de leerlo letra por letra, lo lees párrafo por párrafo. Cada párrafo te da una idea completa. Las **ventanas** funcionan igual.

Una ventana de 1 segundo es simplemente un grupo de **1000 filas consecutivas** (porque la frecuencia de muestreo es 1000 Hz → 1000 muestras = 1 segundo).

```
Señal original (cada guión es una muestra):

|---...---||---...---||---...---||---...---|
 Ventana 1   Ventana 2   Ventana 3   Ventana 4
 (1000 muestras) (1000 muestras) ...
```

Las ventanas **no se solapan** (sin overlapping). Cuando termina una, empieza la siguiente. No comparten filas entre sí.

---

## Paso 2: Construir las ventanas en código

```python
import numpy as np
import pandas as pd

# Cargar el dataset
df = pd.read_csv("muscle_fatigue_cycling.csv")

fs = 1000          # Frecuencia de muestreo: 1000 Hz
window_size = fs   # 1 segundo = 1000 muestras

channels = ['Muscle_1', 'Muscle_2', 'Muscle_3', 'Muscle_4',
            'Muscle_5', 'Muscle_6', 'Muscle_7', 'Muscle_8']

# Calcular cuántas ventanas completas caben
n_windows = len(df) // window_size

print(f"Total de muestras: {len(df)}")
print(f"Ventanas de 1 segundo: {n_windows}")
```

Si el dataset tiene, por ejemplo, 500,000 filas:
- `500,000 / 1,000 = 500 ventanas`
- El nuevo dataset tendrá solo **500 filas** en lugar de 500,000. ¡Se redujo 1000 veces!

---

## Paso 3: ¿Qué es una característica (feature)?

En lugar de guardar las 1000 muestras de cada ventana, queremos guardar un **resumen** de esa ventana. Ese resumen son las características.

Piénsalo así: en lugar de guardar todas las notas de un examen (1000 preguntas), guardas el promedio, la nota más alta, la nota más baja y la desviación. Eso resume el comportamiento.

Extraemos características de **dos dominios**:

### Dominio del tiempo
Describen la forma "cruda" de la señal.

| Característica | Qué mide | Código Python |
|---|---|---|
| **RMS** (Root Mean Square) | La energía o fuerza promedio de la señal | `np.sqrt(np.mean(ventana**2))` |
| **Varianza** | Qué tan dispersos están los valores | `np.var(ventana)` |
| **Zero Crossing Rate** | Cuántas veces la señal cruza el cero | `np.sum(np.diff(np.sign(ventana)) != 0)` |
| **MAV** (Mean Absolute Value) | Promedio del valor absoluto | `np.mean(np.abs(ventana))` |

### Dominio de la frecuencia
Para extraer estas, primero se aplica la **Transformada de Fourier (FFT)**, que convierte la señal del tiempo a frecuencias. Es como descomponer un sonido en sus notas musicales.

| Característica | Qué mide | Código Python |
|---|---|---|
| **Frecuencia mediana** | La frecuencia que divide la potencia espectral en dos mitades iguales | Ver código abajo |
| **Frecuencia media** | Promedio ponderado de las frecuencias | Ver código abajo |
| **Potencia espectral total** | Energía total de la señal en frecuencia | `np.sum(psd)` |

```python
from scipy.signal import welch

def frecuencia_mediana(ventana, fs):
    freqs, psd = welch(ventana, fs=fs)
    potencia_acumulada = np.cumsum(psd)
    frecuencia_media_idx = np.searchsorted(potencia_acumulada,
                                            potencia_acumulada[-1] / 2)
    return freqs[frecuencia_media_idx]

def frecuencia_media(ventana, fs):
    freqs, psd = welch(ventana, fs=fs)
    return np.sum(freqs * psd) / np.sum(psd)
```

---

## Paso 4: Extraer todas las características de todas las ventanas

Ahora combinamos todo: recorremos ventana por ventana, canal por canal, y extraemos las características.

```python
from scipy.signal import welch

def extraer_caracteristicas(ventana, fs=1000):
    """Extrae características de tiempo y frecuencia de una ventana 1D."""
    # --- Dominio del tiempo ---
    rms     = np.sqrt(np.mean(ventana**2))
    varianza = np.var(ventana)
    zcr     = np.sum(np.diff(np.sign(ventana)) != 0)
    mav     = np.mean(np.abs(ventana))

    # --- Dominio de la frecuencia ---
    freqs, psd = welch(ventana, fs=fs)
    pot_total  = np.sum(psd)
    frec_media = np.sum(freqs * psd) / np.sum(psd)
    pot_acum   = np.cumsum(psd)
    idx_median = np.searchsorted(pot_acum, pot_acum[-1] / 2)
    frec_mediana = freqs[idx_median]

    return [rms, varianza, zcr, mav, pot_total, frec_media, frec_mediana]


# Construir el nuevo dataset
filas = []

for i in range(n_windows):
    inicio = i * window_size
    fin    = inicio + window_size

    ventana_df = df.iloc[inicio:fin]

    fila = {}

    for canal in channels:
        ventana = ventana_df[canal].values
        feats   = extraer_caracteristicas(ventana, fs=fs)

        nombres = ['rms', 'var', 'zcr', 'mav', 'pot', 'f_media', 'f_mediana']
        for nombre, valor in zip(nombres, feats):
            fila[f'{canal}_{nombre}'] = valor

    # El target de la ventana es el más frecuente en esas 1000 muestras
    fila['target'] = ventana_df['Target'].mode()[0]

    filas.append(fila)

nuevo_df = pd.DataFrame(filas)
print(nuevo_df.shape)
print(nuevo_df.head())
```

---

## Paso 5: ¿Cómo queda el nuevo dataset?

El resultado es una tabla mucho más pequeña y manejable. Cada fila ya no es un instante de tiempo sino **un resumen de 1 segundo de señal**. Se ve así:

| Muscle_1_rms | Muscle_1_var | ... | Muscle_8_f_mediana | target |
|---|---|---|---|---|
| 0.045 | 0.002 | ... | 68.4 | 0 |
| 0.089 | 0.008 | ... | 52.1 | 1 |
| ... | ... | ... | ... | ... |

Con 8 músculos y 7 características por músculo, cada fila tiene **56 columnas de entrada** + la columna `target`.

> **Importante:** La columna `Time` del dataset original **no se incluye** en el nuevo dataset. El tiempo aumenta de forma lineal y tiene una correlación directa con el target (a más tiempo, más fatiga), lo que haría que el modelo "hiciera trampa" usando el tiempo en lugar de aprender de las señales musculares reales.

---

## Resumen visual del proceso completo

```
Dataset original
(500,000 filas × 10 columnas)
          |
          | Dividir en ventanas de 1000 muestras (1 seg)
          ↓
   [W1: filas 0–999]   [W2: filas 1000–1999]   ...   [W500: filas 499000–499999]
          |
          | Extraer 7 características × 8 músculos por ventana
          ↓
Nuevo dataset
(500 filas × 57 columnas)   ← ¡1000 veces más pequeño!
```

Este nuevo dataset es el que se usa para entrenar los modelos de clasificación.

---

## ¿Por qué esto funciona?

Porque la **fatiga muscular no ocurre en un instante**, ocurre gradualmente a lo largo del tiempo. Las características como la frecuencia mediana de la señal EMG son conocidas por **disminuir progresivamente con la fatiga muscular**, lo que las hace muy informativas para distinguir entre condición normal y desgaste. Al resumir cada segundo en esas características, le damos al modelo información útil, compacta y representativa para aprender.
