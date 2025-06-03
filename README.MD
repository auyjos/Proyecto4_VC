
# Proyecto: Estimación de Peso de Pollos con Visión por Computadora

## Información General
Este proyecto desarrolla un sistema capaz de estimar el peso de pollos a partir de imágenes mediante técnicas de visión por computadora y aprendizaje profundo. Se utiliza un modelo de regresión basado en una red neuronal convolucional (CNN) preentrenada, complementada con metadatos de edad y sexo para mejorar la precisión. Adicionalmente, se integra un detector genérico de aves (YOLOv5) para aislar la región del pollo en cada fotograma de una cámara en tiempo real, y luego pasar ese recorte al regresor para obtener el peso estimado.

### Componentes Principales
- **Dataset de imágenes**: Carpeta que contiene fotografías de pollos junto a un archivo CSV con metadatos (nombre de archivo, peso real, edad en días, sexo).
- **Modelo de regresión (CNN + metadatos)**:  
  - Backbone CNN preentrenado (ResNet50) para extraer características visuales.  
  - Rama secundaria para procesar metadatos (edad normalizada y bandera de sexo).  
  - Cabeza de regresión que combina ambas representaciones y produce un valor continuo (peso en kilogramos).
- **Detector en tiempo real (YOLOv5)**:  
  - Se emplea la versión preentrenada en COCO para la clase “bird” como proxy de “pollo”.  
  - Filtra la detección por confianza y tamaño de caja para recortar únicamente la región correspondiente al ave.
- **Integración y pipeline de inferencia**:  
  - Cada fotograma capturado por la cámara se procesa con el detector.  
  - El recorte resultante (solo el pollo) se preprocesa con las mismas transformaciones utilizadas en entrenamiento (redimensionado, normalización).  
  - Se crea un tensor con metadatos (edad normalizada, sexo binarizado) y ambos se pasan al regresor exportado (TorchScript) para obtener el peso.  
  - Se dibujan recuadros y etiquetas superpuestas con el peso estimado en tiempo real sobre el video.

## Propósito del Proyecto
El objetivo principal es ofrecer una solución automática, no invasiva y en tiempo real para estimar el peso de pollos sin necesidad de pesarlos físicamente en cada operación. Entre los beneficios se incluyen:
- **Reducción de estrés animal**: Evitar manipular constantemente a los pollos para pesarlos, disminuyendo el estrés y el riesgo de lesiones.
- **Optimización de procesos en granjas avícolas**: Permitir un monitoreo continuo y automático del peso en ciclos de crecimiento, facilitando la toma de decisiones sobre alimentación, salud y comercialización.
- **Escalabilidad y velocidad**: Con la integración de modelos exportados (TorchScript, ONNX), se habilita la inferencia rápida en dispositivos con o sin GPU, apta tanto para laboratorios como para instalaciones de producción.
- **Medición estandarizada**: Al usar una cámara fija y un algoritmo consistente, se obtiene una referencia uniforme y repetible, reduciendo la variabilidad asociada a pesajes manuales.
- **Fácil despliegue**: El uso de librerías populares (PyTorch, YOLOv5, OpenCV) y la exportación a formatos optimizados permiten incorporar el sistema en aplicaciones móviles, sistemas embebidos o estaciones fijas en granjas.

## Flujo de Trabajo Resumido
1. **Preparación de datos**  
   - Normalizar metadatos (edad en días → escala [0,1], convertir sexo a bandera binaria).  
   - Dividir el conjunto en entrenamiento, validación y prueba.  
2. **Entrenamiento del regresor**  
   - Utilizar transfer learning con ResNet50 para extraer características visuales.  
   - Añadir dos capas densas para combinar las características visuales y tabulares.  
   - Entrenar con pérdida MSE, validación periódica, early stopping y scheduler de tasa de aprendizaje.  
3. **Exportación del modelo**  
   - Guardar el mejor checkpoint y exportar a TorchScript (formato `pollo_regressor_scripted_float.pt`).  
   - (Opcional) Exportar a ONNX para usar con ONNX Runtime en CPU.  
4. **Demo de inferencia en tiempo real**  
   - Capturar video con OpenCV a una resolución predeterminada.  
   - Detectar “birds” (pollos) en cada fotograma con YOLOv5.  
   - Recortar la región del pollo y preprocesarla para el regresor.  
   - Construir el vector de metadatos (edad normalizada, sexo) y realizar la inferencia del peso.  
   - Mostrar la salida gráficamente con recuadros y texto superpuesto.  

## Ventajas y Aplicaciones
- **No requiere infraestructura costosa**: Solo es necesario un computador con GPU opcional y una cámara digital estándar.  
- **Versatilidad**: Puede adaptarse a distintas razas o edades de pollos ajustando únicamente la recolección de datos y el entrenamiento.  
- **Monitoreo continuo**: Ideal para instalaciones donde se requiere seguimiento diario o incluso horarios específicos de peso.  
- **Escalabilidad futura**: Se puede extender para incluir detección de múltiples animales por fotograma, análisis de comportamiento o estimación de otras características (talla, salud visual, etc.).

## Consideraciones Futuras
- **Detector específico de pollos**: Entrenar un YOLOv5 personalizado con anotaciones de bounding boxes de pollos para reducir falsos positivos y negativos.  
- **Ingreso de metadatos en tiempo real**: Diseñar una interfaz para que el operario ingrese edad y sexo antes de colocar el pollo frente a la cámara, o desarrollar un modelo secundario que clasifique sexo automáticamente.  
- **Control de escala**: Incluir un marcador físico (por ejemplo, una regla o rejilla) en el encuadre para mejorar la estimación de tamaño y, por ende, la precisión del peso.  
- **Optimización de CPU**: Exportar los modelos a ONNX y TensorRT para entornos de CPU-constrained en dispositivos embebidos o computadoras industriales.  
- **Mantenimiento y reentrenamiento**: Periodicamente recolectar nuevas imágenes para retrain/update del regresor y del detector, garantizando robustez ante cambios de iluminación, fondo o variabilidad genética de los pollos.

