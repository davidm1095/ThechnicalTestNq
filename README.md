# ThechnicalTestNq
Datos históricos NBA

# Documentación del Proyecto: Arquitectura de Procesamiento de Datos con Azure y Power BI

## Descripción General
Esta solución fue desarrollada utilizando tecnologías de **Azure Data Lake**, **Azure Data Factory**, y **Jupyter Notebook**, emulando un comportamiento similar al que podría desarrollarse en **Databricks**. Además, **Power BI Desktop** se utilizó para disponibilizar las fuentes de datos y construir el modelo analítico. 

El proyecto emplea **Python** como lenguaje de programación principal, junto con librerías específicas de **PySpark** para el procesamiento distribuido.

---

## Fuente de Datos
El conjunto de datos fue extraído del repositorio público de **Kaggle Datasets** y consta de 5 archivos en formato CSV que contienen información sobre juegos de la **NBA (National Basketball Association)**. Estos archivos se almacenaron en un contenedor de **Azure Data Lake** denominado `nba`.

---

## ¿Qué problemas podría resolver esta solución?

La solución resuelve varios problemas relacionados con el análisis y manejo de datos históricos y en tiempo real de la NBA:

- **Consolidación y análisis de datos históricos:** Integra múltiples conjuntos de datos (partidos, estadísticas de jugadores, rankings, equipos, etc.) para permitir un análisis completo y contextualizado de las temporadas de la NBA desde 2004.
- **Seguimiento del rendimiento de jugadores y equipos:** Facilita el análisis detallado de las estadísticas individuales y grupales para evaluar desempeño, identificar patrones, y realizar comparaciones históricas.
- **Exploración de tendencias y correlaciones:** Permite analizar cómo variables como los rankings, los puntajes de juegos, y el rendimiento de los jugadores afectan los resultados de los equipos a lo largo de las temporadas.
- **Acceso eficiente a datos complejos:** Organiza grandes volúmenes de datos en una estructura manejable y optimizada, asegurando que las consultas y análisis puedan realizarse de manera eficiente incluso con datasets grandes.
- **Toma de decisiones basada en datos:** Responde a las necesidades de usuarios finales que buscan información relevante para predicciones, estrategias deportivas, o investigaciones relacionadas con la NBA.

## ¿Qué beneficios aporta a la organización o usuarios finales?

- **Acceso a datos ricos y relevantes:** Los usuarios tienen acceso a datos limpios, estructurados y enriquecidos sobre la NBA, lo que facilita realizar análisis desde el nivel más básico hasta insights avanzados.
- **Información estratégica:** Los equipos técnicos, analistas deportivos, y fanáticos pueden tomar decisiones mejor informadas, ya sea para diseñar estrategias de juego, pronósticos de rendimiento o estudios históricos.
- **Eficiencia en el análisis:** La solución está diseñada para manejar grandes volúmenes de datos y realizar consultas complejas de manera eficiente, reduciendo tiempos de espera para obtener resultados.
- **Personalización y profundidad del análisis:** Los usuarios pueden analizar información específica, como estadísticas individuales de jugadores, impacto del ranking en los resultados de los juegos, o comparaciones entre equipos.
- **Predicciones y análisis de tendencias:** Al consolidar datos históricos y actuales, la solución permite identificar patrones y tendencias que podrían usarse para predicciones futuras, como desempeño de equipos o rankings esperados.
- **Mejora de experiencias para audiencias amplias:** Ya sea que los usuarios sean aficionados, medios de comunicación, equipos de la NBA o empresas, la solución entrega información valiosa que puede ser usada para narrativas, reportajes o estrategias comerciales.

---

## Arquitectura de la Solución
La arquitectura sigue el enfoque **Medallion** (Medallón), ampliamente adoptado en diseños **Lakehouse**. 
Más información sobre esta arquitectura se puede consultar en [Documentación Oficial de Microsoft](https://learn.microsoft.com/es-es/azure/databricks/lakehouse/medallion-architecture).

---


## Configuración de Azure Data Factory

### 1. Configuración Inicial
- **Linked Service:** Se creó un servicio vinculado para conectar **Data Factory** con **Blob Storage**.
- **Datasets:** Se implementaron cuatro datasets:
  - Dos para la fuente inicial (`nba`).
  - Dos para la capa `landing`.

> **Nota:** Se utilizó el formato **Parquet** para los datasets, aprovechando ventajas como compresión integrada, rendimiento optimizado y eficiencia en el procesamiento, especialmente útil en entornos **Big Data**.

---

### 2. Flujo de Datos (Dataflow)
Se diseñó un **Dataflow** para transformar la fuente `games`:
1. **Filtrado:** Filtrar la columna `GAME_DATA_SET`.
2. **Ordenamiento:** Ordenar los datos de manera ascendente por la misma columna.
3. **Carga:** Almacenar los datos transformados en el contenedor `landing`.

---

### 3. Pipelines
Se implementaron dos **pipelines**:
- Uno para ejecutar el **Dataflow**.
- Otro para mover datos desde el contenedor `nba` hacia la capa `landing`.

El pipeline `ingestión_nba_data` está parametrizado para recibir el nombre de la fuente deseada, asegurando flexibilidad en la ejecución.

---

## Procesamiento con Jupyter Notebook
La herramienta **Jupyter Notebook** se utilizó para simular operaciones propias de **Databricks**, organizando el proyecto en carpetas según la estructura **Medallion**:

### Estructura de Carpetas
```plaintext
├── ingest_layer/
│   ├── conformance/
│   ├── curated/
├── business_layer/
│   ├── work/
├── catalogo/
│   ├── bronze/
│   ├── silver/
│   ├── gold/
```

## Proceso de Conformance y Curated

### Conformance Layer
En esta fase, el propósito principal de los notebooks es:

1. **Lectura de Datos:** Leer los archivos directamente del Data Lake.
2. **Definición de Esquema:** Establecer un esquema fijo, definiendo los tipos de datos más adecuados para cada variable.
3. **Parsing y Validación:** Realizar un parsing para garantizar la consistencia y validez de los datos.

Ejemplo del procesamiento en la capa **Conformance**:

- Se lee el archivo del contenedor correspondiente.
- Se define un esquema fijo para la tabla.
- Se registran logs con detalles de las ejecuciones exitosas y errores.
  
A continuación, los datos procesados se escriben en la capa **Curated** del Data Lake.

```plaintext
Estructura de Logs:
- Hora de ejecución
- Resultado del proceso
- Detalle de errores
```
    
---

## Curated Layer
El propósito de los notebooks en la capa **Curated** es garantizar la limpieza y calidad de los datos. Se aplican reglas específicas para transformar y depurar las fuentes antes de moverlas a la siguiente capa (**Work**).

### Ejemplo de Transformaciones:

1. **Transformaciones Generales:**
   - Para todas las fuentes, se convierten las cadenas de texto a mayúsculas.
   - Eliminación de duplicados.

2. **Tabla `games`:**
   - Se eliminan duplicados en la columna `GAME_ID`.
   - Se extraen las columnas `YEAR`, `MONTH` y `DAY` de la columna `GAME_DATE_EST`.

3. **Tabla `games_details`:**
   - Limpieza de la columna `MIN` (minutos de juego):
     - Valores no numéricos se reemplazan por `NaN` utilizando `numpy`.
     - Conversión de la columna a tipo `float`.
     - Creación de una nueva columna derivada llamada `MINFLOAT`, que será utilizada para construir medidas calculadas.

4. **Otras Transformaciones:**
   - Las transformaciones específicas de las demás fuentes están documentadas en los notebooks correspondientes.

### Resultado del Proceso
- Los datos procesados se escriben tanto en la **zona Silver** del catálogo como en el Data Lake, dentro de la carpeta **Curated**.

---

## Work Layer
En la capa **Work**, se aplican reglas de negocio y operaciones necesarias para consolidar los datos en tablas de dimensiones y hechos listas para el consumo.

### Actividades Principales:
1. **Definición de Reglas de Negocio:**
   - Se crean columnas calculadas adicionales según los requerimientos del modelo de datos.

2. **Uniones y Consolidación:**
   - Se relacionan datos entre tablas para formar dimensiones y tablas de hechos.

### Resultado del Proceso:
- Las tablas procesadas son escritas en la **capa Gold** del catálogo y en el Data Lake, dentro de la carpeta **Work**.

---

## Consumo en Power BI Desktop
En esta última etapa, se conectan las tablas procesadas en la capa **Work** a Power BI Desktop para construir el modelo de datos y facilitar su análisis.

### Modelo de Datos en Power BI
- **Estructura del Modelo:**
  - Tablas de hechos: `games_details`.
  - Dimensiones: `teams`, `players`, `coaches`, entre otras.

- **Optimización del Modelo:**
  - Se establecen relaciones entre las tablas.
  - Se crean medidas calculadas para facilitar el análisis.

### Visualización
- El modelo diseñado permite obtener insights clave sobre los datos de la NBA mediante dashboards y reportes interactivos.

---

## Conclusión
Esta solución demuestra cómo aplicar una arquitectura de medallones para procesar y consumir datos de manera eficiente, utilizando tecnologías modernas como **Azure Data Factory**, **Azure Data Lake**, y **Jupyter Notebook**. La integración final con **Power BI Desktop** asegura que los datos sean confiables, escalables y estén listos para análisis avanzados.

### Recursos Adicionales
- [Documentación de Arquitectura de Medallones - Microsoft Learn](https://learn.microsoft.com/es-es/azure/databricks/lakehouse/medallion-architecture)
- Capturas y logs detallados están disponibles en el repositorio del proyecto para referencias adicionales.

