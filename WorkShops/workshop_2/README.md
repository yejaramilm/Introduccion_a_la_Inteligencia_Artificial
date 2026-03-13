# Workshop 2 – Machine Learning & Deep Learning Aplicado

**Universidad EAFIT – Introducción a la Inteligencia Artificial (2026-01)**

---

## Descripción General

Este workshop integra dos problemas supervisados independientes: uno de **clasificación** y uno de **regresión**, aplicando el ciclo completo de un proyecto de Machine Learning y Deep Learning: análisis del problema, exploración de datos, preprocesamiento, entrenamiento, evaluación y análisis crítico de resultados.

---

## Problema 1 – Clasificación: Detección de Fatiga Muscular en Ciclismo

### Dataset

- **Nombre:** Muscle Fatigue Cycling
- **Fuente:** HuggingFace – [YominE/Muscle_Fatigue_Cycling](https://huggingface.co/datasets/YominE/Muscle_Fatigue_Cycling)
- **Descripción:** Señales de electromiografía (EMG) registradas en 8 músculos de la pierna dominante de sujetos realizando sprints en bicicleta. El target indica el estado muscular del sujeto durante la prueba (condición normal vs. desgaste muscular).

### Instrucciones

#### 1. Análisis Preliminar del Problema

a. Preprocese el target estableciendo únicamente **2 etiquetas**:
   - `0` = Condición normal
   - `1` = Desgaste muscular

   Para lo anterior, las etiquetas estimadas en 2 se cambiaran por 1

b. Clasifique las características presentes en el dataset en tipos de variables (numéricas, categóricas, binarias, ordinales, etc.).


#### 2. Extracción de Características (Feature Engineering)

a. Diseñe un algoritmo que tome **ventanas de 1 segundo** sobre los 8 canales de señal, teniendo en cuenta la frecuencia de muestreo determinada previamente.

b. Extraiga **mínimo 4 características** por ventana por canal, combinando características en el **dominio del tiempo** (ej. valor RMS, varianza, cruce por cero, pendiente media de la señal) y en el **dominio de la frecuencia** (ej. frecuencia mediana, frecuencia media, potencia espectral). Estas características conformarán la nueva base de datos. Para la nueva base de datos no tenga en cuenta la variable tiempo, ya que su dependencia lineal con el target sobresimplificaria el problema

c. Documente y justifique cada característica seleccionada en términos de su relevancia.

#### 3. Análisis Exploratorio de Datos (EDA)

Realice un EDA completo sobre la nueva base de datos de características:

- Distribuciones de variables y estadísticos descriptivos.
- Correlaciones entre características.
- Relación entre características y el target (boxplots por clase, separabilidad).
- Análisis de balance de clases.

Cada gráfico o estadística debe ir acompañado de una **interpretación detallada** que explique qué información aporta al problema de clasificación.
Utilice librerías como `pandas`, `numpy`, `matplotlib` y `seaborn`.

a. Grafique una porción de las señales en el tiempo y haga una identificación inicial de los datos. Basado en la informacion disponible, cuales son sus conclusiones acerca de este dataset?


#### 4. Procesamiento de Datos

- Manejo de valores nulos.
- Normalización o estandarización de características.
- Implemente un **pipeline de procesamiento con scikit-learn**.
- Divida los datos en `X_train`, `X_val` y `X_test` con proporciones justificadas (ej. 70/15/15).

#### 5. Entrenamiento y Comparación de Modelos

a. Entrene y evalúe los siguientes clasificadores:
   - k-Nearest Neighbors (kNN)
   - Decision Tree
   - Random Forest
   - Gradient Boosting
   - Deep Neural Network (DNN) *(mínimo 3 capas ocultas, con funciones de activación y regularización)*

b. Para cada modelo, realice **ajuste de hiperparámetros** utilizando alguno de los siguientes algoritmos de optimización:
   - Grid Search
   - Random Search
   - Algoritmos Genéticos

c. Muestre los resultados en una **tabla comparativa** con métricas de clasificación (Accuracy, Precision, Recall, F1-Score) sobre `X_train`, `X_val` y `X_test`.

d. Grafique las curvas de entrenamiento y validación de cada modelo para detectar y justificar **overfitting o underfitting**.

e. Responda:
   - ¿Cuál modelo tuvo mejor desempeño?
   - ¿Alguno presentó overfitting o underfitting? ¿Cómo lo detectó?
   - ¿Cuál seleccionaría para producción y por qué?

#### 6. Evaluación Final del Mejor Modelo

a. Seleccione el mejor modelo, **reentrénelo** con la combinación de datos de entrenamiento y validación (`X_train + X_val`), y realice la predicción final sobre `X_test`.

b. Analice los resultados mediante:
   - **Tabla de métricas de clasificación** final.
   - **Boxplots** de características representativas diferenciando muestras clasificadas en fatiga vs. condición normal, para generar conclusiones.
   - Matriz de confusión.

c. Responda: ¿Considera que es un buen clasificador? ¿Cómo podría mejorarse?

#### 7. Prueba con Muestra Artificial

Genere una muestra aleatoria con valores aproximados a los reales e ingrésela al modelo seleccionado. Analice si el clasificador la define como un sujeto fatigado o no fatigado. ¿El resultado tiene sentido?


## Problema 2 – Regresión: Estimación de Edad a partir de Imágenes Faciales

### Dataset

- **Nombre:** Faces: Age Detection from Images
- **Fuente:** Kaggle – [arashnic/faces-age-detection-dataset](https://www.kaggle.com/datasets/arashnic/faces-age-detection-dataset)
- **Descripción:** Conjunto de imágenes faciales etiquetadas con la edad del sujeto. El objetivo es entrenar un modelo que estime la edad a partir de los píxeles de la imagen.

### Instrucciones

#### 1. Análisis Preliminar del Problema

a. Justifique por qué este es un problema de **regresión** e identifique claramente el target (variable objetivo).

b. Describa las características de entrada (imágenes): dimensiones, espacio de color, distribución de edades, etc.

c. Investigue y explique el protocolo de adquisición y/o generación de los datos.

#### 2. Análisis Exploratorio de Datos (EDA)

- Distribución de edades en el dataset (histograma, estadísticos descriptivos).
- Análisis de balance y posibles sesgos en la distribución del target.
- Visualización de muestras representativas del dataset.
- Análisis de la calidad y variabilidad de las imágenes.

Cada visualización debe ir acompañada de una **interpretación detallada** orientada al problema de regresión.

#### 3. Procesamiento de Datos

- Redimensionamiento y normalización de imágenes.
- Estrategia de data augmentation si se justifica.
- División en `X_train`, `X_val` y `X_test` con proporciones justificadas (ej. 70/15/15).
- Implemente un **pipeline de preprocesamiento** reproducible.

#### 4. Entrenamiento del Modelo CNN para Regresión

Entrene **un modelo de regresión basado en Redes Neuronales Convolucionales (CNN)**:

- Defina y justifique la arquitectura (capas convolucionales, pooling, capas densas, función de activación de salida).
- Incluya técnicas de regularización (Dropout, Batch Normalization, etc.).
- Utilice una función de pérdida apropiada para regresión (ej. MSE, MAE o Huber Loss).

Reporte las métricas de evaluación sobre `X_train`, `X_val` y `X_test`:

| Métrica | Descripción |
|---|---|
| MAE | Mean Absolute Error |
| RMSE | Root Mean Squared Error |
| R² | Coeficiente de determinación |

Grafique las curvas de pérdida de entrenamiento y validación a lo largo de las épocas. ¿Hay overfitting o underfitting?

#### 5. Prueba con Muestra Artificial

Tome una imagen de prueba e ingrésela al modelo CNN entrenado (Debe tener las mismas caracteristicas en size y channels). Analice la predicción:
- ¿El resultado tiene sentido?
- ¿Qué pasaría si modificara características visuales de la imagen (iluminación, escala, orientación)?



---

## Entregables

El proyecto completo debe entregarse en un **repositorio de GitHub** con la siguiente estructura mínima:

```
workshop_2/
├── README.md                  ← Este archivo (introducción al proyecto)
├── clasificacion/
│   ├── clasificacion.ipynb    ← Notebook con desarrollo completo del problema de clasificación
│   └── ...
├── regresion/
│   ├── regresion.ipynb        ← Notebook con desarrollo completo del problema de regresión
│   └── ...
```

Cada notebook debe incluir:
- Código ejecutable y documentado.
- **Tabla comparativa** de desempeño de modelos (clasificación) o métricas del modelo CNN (regresión).
- Respuestas a todas las preguntas de análisis planteadas.

El desarrollo detallado de cada numeral se documentará en los notebooks, y el README servirá como introducción y guía de navegación del proyecto.

---

## Criterios de Evaluación

### Problema de Clasificación

| Criterio | Peso |
|---|---|
| Claridad en la justificación teórica y análisis preliminar | 20% |
| Calidad del EDA, extracción de características e interpretaciones | 20% |
| Correcto preprocesamiento de datos y pipeline | 20% |
| Implementación, ajuste de hiperparámetros y comparación de modelos | 25% |
| Prueba con muestra artificial y análisis  | 15% |

### Problema de Regresión

| Criterio | Peso |
|---|---|
| Claridad en la justificación teórica y análisis preliminar | 20% |
| Calidad del EDA e interpretaciones | 20% |
| Correcto preprocesamiento de datos y pipeline | 20% |
| Implementación y evaluación del modelo CNN de regresión | 25% |
| Prueba con muestra artificial y análisis  | 15% |

---

## Nota Importante

> Este workshop será sustentado por un miembro del equipo seleccionado de forma aleatoria, con el fin de verificar el entendimiento del mismo. **El trabajo colaborativo y la estandarización del conocimiento dentro de los equipos es fundamental.**
