# Detección automática de personas en imágenes aéreas

Proyecto de **Inteligencia Artificial II** orientado al apoyo de operaciones de búsqueda y rescate mediante detección de personas en imágenes aéreas capturadas desde drones.

**Autores:** Miguel Murillo · Lucero Gómez  
**Curso:** Inteligencia Artificial II — B2 · 2026-2

---

## Descripción

El proyecto estudia modelos de *Deep Learning* para **detectar y localizar personas mediante bounding boxes** en imágenes aéreas. El problema es especialmente desafiante porque gran parte de las personas ocupa una región muy pequeña de la imagen.

Se implementaron y compararon:

- **YOLOv8n** — detector de una etapa utilizado como *baseline*.
- **Faster R-CNN** — detector de dos etapas basado en propuestas de regiones.

También se analizó **Mask R-CNN**, pero no se implementó porque los datasets disponibles contienen cajas delimitadoras y no máscaras de segmentación por instancia.

## Dataset

El experimento principal utiliza **SARD — Search And Rescue Dataset**.

- Train: **4.041** imágenes
- Validation: **1.144** imágenes
- Test: **570** imágenes
- Total: **5.755** imágenes
- Clase: `Human`
- Resolución de la exportación utilizada: **640 × 640**
- Anotaciones: formato YOLO (`class x_center y_center width height`)

Durante el EDA se contabilizaron **7.424 bounding boxes**. El **63,44 %** de las personas ocupa menos del **1 % del área total de la imagen**, lo que confirma la presencia predominante de objetos pequeños.

Datasets de referencia:

- SARD: https://www.kaggle.com/datasets/nikolasgegenava/sard-search-and-rescue
- SARD 2: https://www.kaggle.com/datasets/nikolasgegenava/sard-2-search-and-rescue-dataset-extra-classes

> Los datasets no se incluyen en este repositorio por tamaño y procedencia. El notebook los descarga desde Kaggle.

## Resultados

### Conjunto de prueba

| Modelo | Precision | Recall | mAP@50 | mAP@50-95 | Velocidad |
|---|---:|---:|---:|---:|---:|
| **YOLOv8n** | **0.9654** | 0.8743 | **0.9343** | **0.6203** | ≈ 4.5 ms/imagen* |
| **Faster R-CNN** | 0.8146 | **0.9126** | 0.9084 | 0.5900 | 85.06 ms/imagen · 11.76 FPS |

\*La cifra de YOLO corresponde a la inferencia reportada por Ultralytics. Para comparar velocidad de forma estricta, ambos modelos deben medirse con el mismo protocolo.

### Interpretación

- **YOLOv8n** alcanzó la mayor Precision, el mejor mAP y una inferencia considerablemente más rápida.
- **Faster R-CNN** consiguió el mayor Recall, detectando una mayor proporción de personas reales.
- En el escenario evaluado, **YOLOv8n presenta el mejor equilibrio global entre calidad de detección y velocidad**.
- El Recall de Faster R-CNN es relevante en búsqueda y rescate porque un falso negativo implica una persona no detectada.

## Viabilidad de Mask R-CNN

La inspección de SARD y SARD 2 mostró:

- 0 máscaras o polígonos de segmentación.
- SARD: **7.424** anotaciones tipo caja.
- SARD 2: **6.354** anotaciones tipo caja.

Mask R-CNN requiere una máscara por instancia para entrenar y evaluar su rama de segmentación. Generar máscaras rectangulares a partir de los bounding boxes no representaría la silueta real de las personas.

**Conclusión:** Mask R-CNN se conserva como referencia del estado del arte, pero **no es viable con las anotaciones actuales sin construir nuevas máscaras de segmentación**.

## SARD 2

SARD 2 fue analizado como posible complemento, pero se encontró un solapamiento muy alto a nivel de identificadores de origen:

- SARD: 1.980 IDs
- SARD 2: 1.980 IDs
- Compartidos: **1.979**

Por esta razón no se incorporó al experimento final de esta entrega, evitando introducir posibles escenas derivadas o redundancia entre particiones.

## Estructura del repositorio

```text
sard-aerial-person-detection/
├── README.md
├── requirements.txt
├── .gitignore
├── notebooks/
│   └── IA2_SARD_Detection.ipynb
├── data/
│   └── README.md
├── models/
│   └── README.md
├── results/
│   ├── figures/
│   │   ├── faster_rcnn_training_curves.png
│   │   └── model_comparison.png
│   └── metrics/
│       └── model_comparison.csv
└── docs/
    └── presentacion_segunda_entrega.pdf
```

## Ejecución

### Opción recomendada: Google Colab

1. Abrir `notebooks/IA2_SARD_Detection.ipynb`.
2. Activar un entorno con GPU.
3. Ejecutar las celdas de instalación.
4. Descargar SARD mediante `kagglehub`.
5. Ejecutar el EDA o saltar directamente a las secciones de entrenamiento si se dispone de checkpoints.

El notebook utiliza Google Drive para conservar checkpoints de los entrenamientos.

### Entorno local

```bash
python -m venv .venv
```

Windows:

```bash
.venv\Scripts\activate
```

Linux/macOS:

```bash
source .venv/bin/activate
```

Luego:

```bash
pip install -r requirements.txt
```

> Para entrenamiento con GPU local, instala PyTorch y Torchvision con la versión CUDA adecuada para tu equipo.

## Configuración principal

### YOLOv8n

- Transfer learning
- Input: 640 × 640
- 50 épocas
- Dataset: SARD
- Clase: `Human`

### Faster R-CNN

- Backbone: ResNet50-FPN
- Transfer learning
- 15 épocas máximas
- Mejor checkpoint: **época 12**
- Optimizer: SGD
- Data augmentation: horizontal flip
- Selección del mejor modelo mediante mAP de validación
- Checkpoints persistentes en Google Drive

## Archivos pesados

Este repositorio **no versiona**:

- datasets completos;
- checkpoints `.pt` / `.pth`;
- carpetas de ejecución completas de Ultralytics;
- cachés de Kaggle/Colab.

Esto evita exceder los límites de GitHub y mantiene el historial del repositorio limpio.

Si se desea compartir los pesos entrenados, se recomienda utilizar **GitHub Releases, Git LFS o un enlace externo**.

## Presentación

La presentación de la segunda entrega está disponible en:

`docs/presentacion_segunda_entrega.pdf`

## Estado del proyecto

- [x] Análisis exploratorio de SARD
- [x] Verificación de etiquetas y particiones
- [x] Análisis de objetos pequeños
- [x] YOLOv8n
- [x] Faster R-CNN
- [x] Comparación preliminar
- [x] Análisis de viabilidad de Mask R-CNN
- [x] Análisis de SARD 2
- [ ] Análisis detallado de falsos positivos/falsos negativos
- [ ] Medición de velocidad con protocolo unificado
- [ ] Consolidación de entrega final

---

Proyecto académico — Universidad Industrial de Santander.
