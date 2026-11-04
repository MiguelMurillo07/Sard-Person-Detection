# Guía rápida para publicar en GitHub

## 1. Crea el repositorio

Nombre recomendado:

`sard-aerial-person-detection`

Descripción sugerida:

`Detección de personas en imágenes aéreas de búsqueda y rescate con YOLOv8n y Faster R-CNN sobre SARD.`

Crea el repositorio vacío, preferiblemente **sin README ni .gitignore**, porque este paquete ya los incluye.

## 2. Abre una terminal en esta carpeta

```bash
git init
git branch -M main
git add .
git commit -m "Initial commit: SARD aerial person detection project"
```

## 3. Conecta GitHub

Cambia `TU_USUARIO` por tu cuenta:

```bash
git remote add origin https://github.com/TU_USUARIO/sard-aerial-person-detection.git
git push -u origin main
```

## 4. Revisa antes de subir

```bash
git status
```

No deberían aparecer:

- datasets completos;
- archivos `.pt` o `.pth`;
- `kaggle.json`;
- carpetas de caché;
- `runs_sard/`.

## 5. Después del primer push

En GitHub:

- añade una breve descripción;
- agrega los topics `deep-learning`, `object-detection`, `yolov8`, `faster-rcnn`, `search-and-rescue`, `computer-vision`;
- fija una imagen social si quieren;
- si van a compartir checkpoints, publíquenlos como Release o mediante Git LFS.
