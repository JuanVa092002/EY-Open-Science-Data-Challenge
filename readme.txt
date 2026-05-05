Landsat_Data_Extraction_Notebook.ipynb — Guía rápida
====================================================

Qué es
-------
Cuaderno oficial del EY AI & Data Challenge 2026 (material “Desarrollo General”): extrae reflectancia de superficie Landsat Collection 2 Level 2 vía API de Microsoft Planetary Computer, agrega índices NDMI y MNDWI, y genera CSV listos para unir con el Benchmark_Model_Notebook.

Fuente remota
-------------
- Catálogo STAC: https://planetarycomputer.microsoft.com/api/stac/v1
- Colección: landsat-c2-l2
- Autenticación/firma de URLs: planetary_computer (pc.sign_inplace en el cliente STAC)

Entradas (rutas relativas al directorio de trabajo del kernel)
----------------------------------------------------------------
- water_quality_training_dataset.csv — entrenamiento (el notebook asume ~9 319 filas).
- submission_template.csv — plantilla de evaluación/envío (~200 filas en la copia del repo).

Coloca esos CSV en el mismo directorio desde el que ejecutas el notebook, o ajusta las rutas en las celdas read_csv.

Salidas
-------
- Entrenamiento (ejemplo en el notebook): landsat_features_training_200.csv — solo las primeras 200 filas como demo rápida.
- Objetivo para el benchmark completo: landsat_features_training.csv — todas las filas, tras extraer por lotes y concatenar (no se incluye en el repo; hay que generarlo).
- Validación / holdout: landsat_features_validation.csv — mismas columnas esquema que el bloque de entrenamiento.

Columnas finales de cada CSV de features
----------------------------------------
Latitude, Longitude, Sample Date, nir, green, swir16, swir22, NDMI, MNDWI

Nota: nir proviene de la banda nir08 del producto Landsat C2 L2 en el código (variable renombrada a nir en el DataFrame).

Lógica de extracción (función compute_Landsat_values)
-------------------------------------------------------
- Por fila: lat, lon, Sample Date (parseo con dayfirst=True).
- Bbox ~100 m alrededor del punto: half-size en grados 0.00089831 (aprox. 100 m / 110 km por grado en ecuador).
- Búsqueda STAC en bbox, datetime 2011-01-01/2015-12-31, eo:cloud_cover < 10.
- Si no hay items: NaN en las cuatro bandas.
- Si hay items: se ordenan por cercanía temporal a la fecha de muestreo (UTC); se usa el más cercano.
- Bandas cargadas con odc.stac stac_load: green, nir08, swir16, swir22; mediana espacial en el bbox; reflectancia 0 sustituida por NaN.
- Errores en try/except: devuelve NaN en bandas.

Índices (post-extracción)
-------------------------
- NDMI = (NIR − SWIR16) / (NIR + SWIR16 + eps) con eps = 1e-10 en el bloque de entrenamiento.
- MNDWI = (Green − SWIR16) / (Green + SWIR16 + eps).

En el bloque de validación, la celda de NDMI usa denominador (nir + swir16) sin sumar eps; el de MNDWI sí usa eps. Para resultados consistentes con el tramo de entrenamiento, conviene alinear ambas fórmulas (mismo denominador que en entrenamiento).

Rendimiento y lotes
--------------------
- El texto del notebook indica ~7+ horas para las 9 319 ubicaciones en un portátil típico; riesgo de límites de API, timeouts o fallos puntuales.
- Se recomienda extraer por lotes, guardar CSV parciales y fusionar en landsat_features_training.csv.
- La demo usa Water_Quality_df_200 (filas 0–199) y escribe landsat_features_training_200.csv.

Consumidor aguas abajo
----------------------
Benchmark_Model_Notebook.ipynb lee landsat_features_training.csv y landsat_features_validation.csv y los combina con calidad del agua / plantilla mediante Latitude, Longitude, Sample Date.

Dependencias Python (indicativas)
---------------------------------
numpy, pandas, pystac_client, planetary_computer, odc.stac, tqdm; entorno típico “geo” compatible con Planetary Computer.

Material relacionado en el repo
--------------------------------
- Landsat_Demonstration_Notebook.ipynb — exploración visual y conceptos.
- Versión Snowflake: resources/3.Material_Complementario_Desarrollo_Snowflake/LANDSAT_DATA_EXTRACTION_NOTEBOOK_SNOWFLAKE.ipynb
