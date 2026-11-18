# Detección automática de personas en imágenes aéreas

Proyecto de **Inteligencia Artificial II** orientado al apoyo de operaciones de búsqueda y rescate mediante detección automática de personas en imágenes aéreas capturadas desde drones.

**Autores:** Miguel Murillo · Lucero Gómez  
**Curso:** Inteligencia Artificial II — B2 · 2026-2

---

## Descripción

El proyecto estudia el uso de modelos de Deep Learning para detectar y localizar personas mediante *bounding boxes* en imágenes aéreas.

El problema presenta una dificultad importante debido al tamaño reducido de las personas dentro de las imágenes. Durante el análisis exploratorio se encontró que el **63,44 % de las personas ocupa menos del 1 % del área total de la imagen**, por lo que la detección de objetos pequeños se convirtió en uno de los principales retos del proyecto.

Inicialmente se estudiaron tres arquitecturas:

- **YOLOv8n**, detector de una etapa.
- **Faster R-CNN**, detector de dos etapas basado en propuestas de regiones.
- **Mask R-CNN**, analizado como posible alternativa con segmentación por instancia.

Mask R-CNN no fue implementado debido a que los conjuntos de datos utilizados contienen únicamente anotaciones mediante *bounding boxes* y no máscaras de segmentación por instancia.

Después de comparar YOLOv8n y Faster R-CNN, se seleccionó **YOLOv8n como modelo final** por ofrecer el mejor equilibrio entre calidad de detección, localización y eficiencia computacional.

---

## Conjunto de datos

El experimento principal utiliza **SARD — Search And Rescue Dataset**.

- **Train:** 4.041 imágenes
- **Validation:** 1.144 imágenes
- **Test:** 570 imágenes
- **Total:** 5.755 imágenes
- **Clase:** `Human`
- **Resolución:** 640 × 640 píxeles
- **Anotaciones:** formato YOLO  
  `class x_center y_center width height`

Durante el análisis exploratorio se contabilizaron **7.424 bounding boxes** correspondientes a personas.

### Dataset principal

SARD:  
https://www.kaggle.com/datasets/nikolasgegenava/sard-search-and-rescue

### Dataset complementario analizado

SARD 2:  
https://www.kaggle.com/datasets/nikolasgegenava/sard-2-search-and-rescue-dataset-extra-classes

SARD 2 fue estudiado como posible complemento, pero no se incorporó al experimento final debido al alto solapamiento encontrado entre los identificadores de origen de ambos conjuntos:

- SARD: 1.980 IDs
- SARD 2: 1.980 IDs
- IDs compartidos: 1.979

Por esta razón se decidió conservar SARD como conjunto principal y evitar introducir redundancia entre escenas.

---

## Comparación de modelos

Durante la segunda etapa del proyecto se compararon YOLOv8n y Faster R-CNN sobre el conjunto de test.

| Métrica | YOLOv8n | Faster R-CNN |
|---|---:|---:|
| AP@50 | **0.937** | 0.921 |
| AP@75 | **0.717** | 0.680 |
| AP@50-95 | **0.627** | 0.600 |
| F1 máximo | 0.920 | **0.927** |
| Precision / Recall con conf. 0.25 | 0.880 / 0.914 | 0.840 / 0.918 |
| Falsos positivos con conf. 0.25 | **91** | 128 |
| Latencia por imagen | **≈ 5.3 ms** | ≈ 85.8 ms |
| Parámetros | **≈ 3.1 M** | ≈ 41 M |

Faster R-CNN presentó un Recall ligeramente superior en algunos puntos de operación, mientras que YOLOv8n obtuvo mejores valores de Average Precision y una diferencia considerable en velocidad y cantidad de parámetros.

A partir de estos resultados se seleccionó **YOLOv8n como detector principal para la etapa final del proyecto**.

---

## Modelo final: YOLOv8n

YOLOv8n fue entrenado mediante transferencia de aprendizaje utilizando pesos preentrenados y posteriormente ajustado sobre SARD.

Configuración principal:

- Entrada: 640 × 640
- Épocas: 50
- Batch size: 16
- Clase: `Human`
- Dataset principal: SARD
- Transfer learning
- Selección de resultados mediante validación durante el entrenamiento

La variante Nano fue utilizada debido a su bajo costo computacional y su capacidad para mantener un buen desempeño sobre objetos pequeños.

---

## Evaluación final

Después de seleccionar YOLOv8n se realizó un análisis más detallado de su comportamiento.

Además de las métricas estándar de detección, se utilizó un criterio complementario con:

- **IoU mínimo:** 0.10
- **AP@0.10:** 0.956
- **Threshold de confianza analizado:** 0.374
- **Recall objetivo:** ≥ 0.90

El uso de IoU 0.10 permite analizar si el modelo logra identificar aproximadamente la región donde se encuentra la persona, incluso cuando el bounding box no queda ajustado con precisión.

Este criterio no reemplaza las métricas estándar AP@50 y AP@50-95, sino que se utiliza como análisis complementario orientado a presencia.

### Resultados en el punto de operación analizado

| Métrica | Resultado |
|---|---:|
| True Positives | 659 |
| False Positives | 21 |
| False Negatives | 73 |
| Precision | 0.969 |
| Recall | 0.900 |
| F1 | 0.933 |
| AP@0.10 | 0.956 |

El análisis del threshold mostró el compromiso existente entre Precision y Recall:

- un threshold bajo permite encontrar más personas, pero aumenta las falsas alarmas;
- un threshold alto reduce los falsos positivos, pero aumenta las personas omitidas.

En un escenario de búsqueda y rescate se consideró especialmente importante conservar un Recall alto, debido al costo asociado a un falso negativo.

---

## Análisis de falsos positivos

Se analizaron manualmente los **21 falsos positivos** obtenidos en el punto de operación seleccionado.

El desglose mostró:

- **15 detecciones duplicadas**
- **6 detecciones asociadas inicialmente a fondo**

Dentro de las detecciones de fondo se encontraron casos visualmente ambiguos, incluyendo regiones con posibles personas parcialmente visibles o elementos sin una anotación clara en el ground truth.

Aproximadamente **2 de estos casos se identificaron como falsas alarmas visualmente claras**.

Este análisis mostró que una parte importante de los falsos positivos no corresponde necesariamente a una confusión completa entre una persona y el entorno, sino a detecciones duplicadas o casos ambiguos de anotación.

---

## Análisis de falsos negativos

También se estudiaron los **73 falsos negativos**.

El análisis mostró:

- **55** personas sí generaron una detección, pero con confianza inferior al threshold.
- **13** no fueron detectadas.
- **5** compartían una caja o región con otra persona.
- **49 de 73** se encontraban cerca del borde del tile.
- **69 de 73** correspondían a personas pequeñas.

El tamaño del objeto resultó ser uno de los factores más importantes en el desempeño del detector.

### Recall según tamaño

| Tamaño de la persona | Recall |
|---|---:|
| Pequeña — menos del 1 % de la imagen | **0.847** |
| Mediana — entre 1 % y 5 % | **0.984** |
| Grande — 5 % o más | **1.000** |

Estos resultados muestran que la principal dificultad de YOLOv8n se encuentra en la detección de personas pequeñas, especialmente cuando aparecen parcialmente cortadas en los bordes de los tiles.

También se identificaron anotaciones con cajas inferiores a 8 píxeles, que representan casos especialmente difíciles y que podrían requerir una revisión adicional del etiquetado.

---

## Mask R-CNN

Mask R-CNN fue considerado inicialmente como una posible tercera arquitectura.

Sin embargo, la revisión de SARD y SARD 2 mostró que ninguno de los dos conjuntos contiene máscaras ni polígonos de segmentación.

- SARD: 7.424 anotaciones mediante bounding boxes.
- SARD 2: 6.354 anotaciones mediante bounding boxes.
- Máscaras de segmentación disponibles: 0.

Crear máscaras rectangulares directamente a partir de los bounding boxes no representaría correctamente la silueta real de las personas.

Por esta razón, Mask R-CNN se mantuvo únicamente como referencia dentro del análisis y no fue incluido en la comparación experimental.

---

## Principales conclusiones

YOLOv8n presentó el mejor equilibrio global entre calidad de detección y eficiencia computacional frente a Faster R-CNN.

El análisis final permitió identificar que:

- YOLOv8n alcanza un **AP@0.10 de 0.956** cuando se utiliza un criterio orientado a presencia.
- Un valor alto de AP@0.10 no implica necesariamente una localización precisa de la caja.
- Las métricas AP@50 y AP@50-95 siguen siendo necesarias para evaluar la calidad espacial de las detecciones.
- La principal dificultad del modelo se concentra en las personas pequeñas.
- Cerca del 95 % de los falsos negativos analizados corresponden a objetos que ocupan menos del 1 % de la imagen.
- Una parte importante de los errores aparece cerca de los bordes de los tiles.
- La mayoría de los falsos positivos detectados en el análisis final corresponden a cajas duplicadas.

Como posibles mejoras futuras se plantea estudiar el uso de tiles con solapamiento, mayor resolución de entrada y una revisión específica de las anotaciones extremadamente pequeñas.

---

## Estructura del repositorio

```text
Sard-Person-Detection/
├── README.md
├── requirements.txt
├── .gitignore
│
├── notebooks/
│   └── IA2_SARD_Detection.ipynb
│
├── data/
│   └── README.md
│
├── models/
│   └── README.md
│
├── results/
│   ├── figures/
│   │   ├── faster_rcnn_training_curves.png
│   │   ├── model_comparison.png
│   │   ├── threshold_analysis.png
│   │   ├── true_positives.png
│   │   ├── false_positives.png
│   │   ├── false_negatives.png
│   │   └── recall_by_object_size.png
│   │
│   └── metrics/
│       ├── model_comparison.csv
│       └── final_yolov8n_analysis.csv
│
└── docs/
    └── presentacion_final.pdf
```

## Autores

**Miguel Murillo**  
**Lucero Gómez**

Universidad Industrial de Santander  
Inteligencia Artificial II — 2026-2
