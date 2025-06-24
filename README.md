# Análisis y Conclusiones - Clasificación de Audio con Redes Neuronales

## Resumen del Proyecto

El presente trabajo implementa y compara dos arquitecturas de redes neuronales para la clasificación de dígitos hablados utilizando el dataset AudioMNIST. Se evalúan redes convolucionales (CNN) y redes recurrentes (LSTM) aplicadas sobre representaciones espectrográficas de las señales de audio.

## Metodología Implementada

### Preprocesamiento de Datos

- **Normalización**: Aplicación de normalización min-max para estandarizar las amplitudes de audio en el rango [-1, 1]
- **Padding/Truncado**: Uniformización de longitud a 8000 muestras mediante padding simétrico y truncado central. Esta función se ha desarrollado y podría utilizarse en applicaciones futuras, pero se optó por eliminación de outliers.
- **Eliminación de outliers**: Filtrado de audios con duración superior a 8000 muestras
- **Transformación espectral**: Conversión a espectrogramas mediante Short-Time Fourier Transform (STFT)

### Balance Dataset

El dataset presenta un balance perfecto (250 ejemplos por clase). Solo se han eliminado 4 muestras que representaban outliers, pero asimismo se implementó una función de preprocesamiento para cortar por ambos extremos audios que superen la longitud deseada de 8000 samples.

### Arquitecturas Evaluadas

#### Red Convolucional (CNN)
- Redimensionamiento a 32×32 píxeles
- Dos capas convolucionales (32 y 64 filtros)
- MaxPooling y Dropout para regularización
- Capa densa final de 128 neuronas

#### Red Recurrente (LSTM)
- Capa LSTM con 128 unidades
- Capa densa intermedia de 64 neuronas
- Arquitectura más simple y directa

## Resultados Experimentales

### Análisis del Parámetro frame_step

#### CNN - Impacto del frame_step
- **frame_step = 128**: Accuracy = 97%
- **frame_step = 64**: Accuracy = 98%

**Interpretación**: La reducción del frame_step mejora el rendimiento de la CNN debido al incremento en la resolución temporal del espectrograma, proporcionando mayor información visual que la arquitectura convolucional puede explotar efectivamente.

#### LSTM - Impacto del frame_step
- **frame_step = 128**: Accuracy = 98%
- **frame_step = 64**: Accuracy = 91%

**Interpretación**: El comportamiento inverso en LSTM se explica por el aumento excesivo en la longitud de secuencia, lo que introduce redundancia y dificulta el aprendizaje de patrones temporales relevantes.

## Conclusiones

### Rendimiento Comparativo

Las arquitecturas CNN y LSTM demuestran capacidades similares para la tarea de clasificación de dígitos hablados, alcanzando accuracy superior al 97% en condiciones óptimas. Sin embargo, presentan sensibilidades diferentes a los parámetros de preprocesamiento y características arquitectónicas distintivas.

### Análisis de Complejidad Arquitectónica

Se observa una diferencia significativa en la complejidad de los modelos:

- **LSTM**: 141,002 parámetros (550.79 KB) - Arquitectura más eficiente
- **CNN**: 1,625,869 parámetros (6.20 MB) - Arquitectura más compleja

A pesar de su mayor simplicidad, la red LSTM alcanza un accuracy del 99%, superando ligeramente a la CNN (97%). Esto sugiere que para tareas de clasificación de audio secuencial, las representaciones temporales pueden ser más eficientes que las espaciales redimensionadas.

### Impacto de la Resolución Temporal

Se observa una relación inversa entre el beneficio de alta resolución temporal y el tipo de arquitectura:

- **CNN**: Beneficiada por mayor resolución (menor frame_step) debido a su capacidad de procesamiento espacial de patrones locales en espectrogramas
- **LSTM**: Penalizada por secuencias excesivamente largas que introducen redundancia y dificultan el aprendizaje de patrones temporales relevantes

### Análisis de Confusión Fonética

Los classification reports muestran que ambas arquitecturas mantienen métricas superiores a 0.97 para la mayoría de las clases. Sin embargo, se identificaron patrones de confusión específicos entre dígitos fonéticamente similares:

- **Dígitos 3 y 6**: Presentan características acústicas similares en su pronunciación, especialmente en las transiciones consonánticas y la estructura silábica
- **Implicaciones**: Esta confusión refleja los desafíos inherentes del reconocimiento de voz, donde la similitud fonética puede introducir ambigüedad incluso en modelos de alto rendimiento

### Arquitecturas Implementadas - Detalles Técnicos

#### Red Recurrente (LSTM)
```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━┓
┃ Layer (type)                    ┃ Output Shape           ┃       Param # ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━┩
│ lstm (LSTM)                     │ (None, 128)            │       132,096 │
├─────────────────────────────────┼────────────────────────┼───────────────┤
│ dense_2 (Dense)                 │ (None, 64)             │         8,256 │
├─────────────────────────────────┼────────────────────────┼───────────────┤
│ dense_3 (Dense)                 │ (None, 10)             │           650 │
└─────────────────────────────────┴────────────────────────┴───────────────┘
```

**Total params**: 141,002 (550.79 KB)  
**Trainable params**: 141,002 (550.79 KB)  
**Non-trainable params**: 0 (0.00 B)

#### Red Convolucional (CNN)
**Input shape**: (61, 129, 1)

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━┓
┃ Layer (type)                    ┃ Output Shape           ┃       Param # ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━┩
│ resizing (Resizing)             │ (None, 32, 32, 1)      │             0 │
├─────────────────────────────────┼────────────────────────┼───────────────┤
│ normalization (Normalization)   │ (None, 32, 32, 1)      │             3 │
├─────────────────────────────────┼────────────────────────┼───────────────┤
│ conv2d (Conv2D)                 │ (None, 30, 30, 32)     │           320 │
├─────────────────────────────────┼────────────────────────┼───────────────┤
│ conv2d_1 (Conv2D)               │ (None, 28, 28, 64)     │        18,496 │
├─────────────────────────────────┼────────────────────────┼───────────────┤
│ max_pooling2d (MaxPooling2D)    │ (None, 14, 14, 64)     │             0 │
├─────────────────────────────────┼────────────────────────┼───────────────┤
│ dropout (Dropout)               │ (None, 14, 14, 64)     │             0 │
├─────────────────────────────────┼────────────────────────┼───────────────┤
│ flatten (Flatten)               │ (None, 12544)          │             0 │
├─────────────────────────────────┼────────────────────────┼───────────────┤
│ dense (Dense)                   │ (None, 128)            │     1,605,760 │
├─────────────────────────────────┼────────────────────────┼───────────────┤
│ dropout_1 (Dropout)             │ (None, 128)            │             0 │
├─────────────────────────────────┼────────────────────────┼───────────────┤
│ dense_1 (Dense)                 │ (None, 10)             │         1,290 │
└─────────────────────────────────┴────────────────────────┴───────────────┘
```

**Total params**: 1,625,869 (6.20 MB)  
**Trainable params**: 1,625,866 (6.20 MB)  
**Non-trainable params**: 3 (16.00 B)

### Recomendaciones Técnicas

1. **Para aplicaciones de producción escalables**: Preferir arquitecturas CNN debido a su capacidad de paralelización durante el entrenamiento e inferencia
2. **Para máxima velocidad de procesamiento**: Las redes convolucionales permiten procesamiento paralelo de lotes, crítico en aplicaciones en tiempo real
3. **Optimización de hiperparámetros**: El frame_step debe ajustarse considerando el tipo de arquitectura utilizada

### Conclusión Final

Aunque la LSTM alcanza un rendimiento ligeramente superior (99% vs 97% de accuracy), la diferencia no justifica las limitaciones de procesamiento secuencial. Las arquitecturas CNN ofrecen ventajas en términos de escalabilidad y eficiencia computacional, especialmente considerando su capacidad de paralelización tanto en entrenamiento como en inferencia.
