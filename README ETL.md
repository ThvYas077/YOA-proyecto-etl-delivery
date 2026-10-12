# 🍔 Pipeline ETL & Inteligencia de Mercado – Food Delivery

Trabajo Práctico Integrador del módulo **Python para Análisis de Datos (ETL)**.
Instructor: Christian Condori.

**Autor:** [Tu nombre completo]

---

## 📌 Contexto

Una startup de Food Delivery busca expandirse en Estados Unidos y necesita entender cómo operan sus competidores: en qué ciudades se concentran, cómo se distribuyen los precios y si un precio más alto se traduce en mejor calidad.

Este proyecto construye un pipeline ETL completo sobre datos obtenidos por web scraping y responde a cuatro preguntas del CEO mediante consultas SQL y visualizaciones.

## 🏗️ Arquitectura del pipeline

```
CSV crudos ──► EXTRACT ──► TRANSFORM ──► LOAD ──► BUSINESS INTELLIGENCE
              (Pandas)    (limpieza,    (SQLite)   (SQL + Seaborn)
                           filtros,
                           merge)
```

| Fase | Qué se hace |
|---|---|
| 📥 **Extract** | Lectura de `restaurants.csv` y de los menús con Pandas + Data Profiling (`shape`, `dtypes`, nulos). |
| 🧹 **Transform** | Mapeo de rangos de precio, split de la dirección, filtro de calidad, limpieza y casteo del precio, filtro de restaurantes consolidados e INNER JOIN. |
| 💾 **Load** | Carga de `df_master` en la base SQLite `delivery_insights.db`, tabla `master_food_data`. |
| 📊 **BI** | 4 consultas SQL con `pd.read_sql(text(query), conn)` y gráficos para el directorio. |

## 📂 Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `Proyecto_final.ipynb` | Notebook con todo el pipeline, comentado y organizado por fases. |
| `README.md` | Este documento. |

Los archivos CSV no se incluyen en el repositorio porque son los datos entregados por el instructor.

## ⚙️ Entorno de ejecución

El proyecto se desarrolló en **Databricks (Serverless)** en lugar de Google Colab. Esto implica tres diferencias:

1. Los CSV se leen desde un Volume de Unity Catalog: `/Volumes/workspace/default/delivery_raw/`.
2. Se usa `dbutils.fs.ls()` para listar los archivos (función exclusiva de Databricks).
3. La base SQLite se crea en `/tmp/delivery_insights.db`, porque el almacenamiento del Workspace no admite el bloqueo de archivos que requiere SQLite.

**Para ejecutarlo en Google Colab**, basta con cambiar la variable `ruta` por la carpeta donde estén los CSV (por ejemplo, `"data/"`), eliminar la celda de `dbutils.fs.ls()` y usar `sqlite:///delivery_insights.db` como ruta de la base.

## 🧹 Decisiones de limpieza

Durante el diagnóstico se detectaron problemas que no estaban en la consigna y se resolvieron así:

| Hallazgo | Decisión |
|---|---|
| El dataset de menús llegó **particionado en 10 archivos**. | Se leyeron y unieron con `pd.concat`. |
| `restaurant_menus_parte_6 (1).csv` tenía el mismo tamaño exacto que `parte_6.csv`. | Se excluyó por ser una copia duplicada. |
| 2,543 filas duplicadas en los menús. | Se eliminaron con `drop_duplicates()`. |
| Precios **negativos** en los menús (mínimo −7.05 USD). | Se eliminaron junto con los precios en 0 y nulos (`price > 0`). |
| Un valor de `price_range` con 17 signos `$`. | Error de scraping: se dejó como nulo al no corresponder a ninguna categoría. |
| Direcciones con más de 2 comas (`calle, ciudad, estado, zip`). | Se usó `split(",", n=2)` para obtener exactamente 3 columnas. |

## 📉 Reducción del dataset

| Etapa | Registros |
|---|---|
| Restaurantes cargados | 63,469 |
| Tras filtro de calidad (score válido) | 35,302 |
| Restaurantes consolidados (> 100 ratings) | 8,702 |
| Menús cargados | 836,350 |
| **`df_master` (INNER JOIN)** | **63,186 platos** |

## 📊 Resultados para el CEO

| Pregunta | Hallazgo |
|---|---|
| **1. Penetración geográfica** | Milwaukee (122) y Seattle (119) concentran unas 3 veces más restaurantes consolidados que el resto del top 5 (Lynnwood, Everett, Bellevue). |
| **2. Oportunidad de precio** | El 73.6% de los restaurantes es Económico y solo el 0.7% es Caro; ningún restaurante "Muy caro" superó los filtros. El mercado no es mayoritariamente caro. |
| **3. Menú por ciudad** | Seattle es la ciudad más cara del top 3 ($11.97 por plato), frente a Lynnwood ($10.82) y Milwaukee ($9.53). |
| **4. Precio vs. calidad** | No hay evidencia de que un precio más alto garantice mejor experiencia: Económico (4.62) y Moderadamente caro (4.64) tienen puntajes casi iguales, y Caro (4.45) se basa en solo 4 restaurantes. |

## 🛠️ Tecnologías

Python · Pandas · SQLAlchemy · SQLite · Matplotlib · Seaborn · Databricks
