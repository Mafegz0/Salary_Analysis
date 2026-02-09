#  Salary Analysis – Ask a Manager Survey 2021

Proyecto de modelado, limpieza y visualización de datos basado en la base pública **Ask a Manager Salary Survey 2021**, con el objetivo de transformar datos crudos en insumos analíticos listos para análisis y visualización interna.

---

##  Objetivo del Proyecto

Estandarizar, modelar y visualizar información salarial proveniente de una encuesta pública, resolviendo problemas de:

- Inconsistencias en campos 
- Errores en valores salariales 
- Conversión de moneda a Pesos Colombianos (COP) a través de una API
- Generación de métricas agregadas para análisis comparativo por industria y ubicación.

El resultado final es un dataset limpio y un dashboard funcional que permite analizar tendencias salariales por industria y ubicación geográfica.

---

##  Estructura del Repositorio
Salary_Analysis/
│
├── dashboard/ # Archivos del dashboard
│
├── data/
│ ├── raw/ # Base original descargada
│ │ └── data_raw.csv
│ │
│ └── processed/ # Datos modelados y limpios
│ ├── salary_clean_global.csv
│ └── salary_clean_global.xlsx
│
├── docs/
│ ├── data_dictionary_v1.csv
│ ├── data_dictionary_v2_modeled.csv
│ ├── ETL_process.md
│ └── fx_metadata.json
│
├── notebooks/
│ └── cleaning_data.ipynb
│
└── README.md

## Fuente de Datos

Formulario original:  
https://www.askamanager.org/2021/04/how-much-money-do-you-make-4.html

Google Sheets público:  
https://docs.google.com/spreadsheets/d/1IPS5dBSGtwYVbjsfbaMCYIWnOuRmJcbequohNxCyGVw

---

# Proceso de Modelado

##  Extracción

Método de extracción de datos:

- Exportación programática en formato CSV desde Google Sheets.
- Acceso mediante una URL de exportación construida con el ID del documento y el GID de la hoja específica.
- Carga de los datos en Python utilizando `pandas.read_csv()`.
- Se guardó una copia estática del dataset dentro del repositorio del proyecto para garantizar la reproducibilidad del análisis.

## Transformación

### Estandarización de Columnas
- Renombrado de columnas a formato `snake_case`.
- Homogeneización de nombres para facilitar el modelado.

### Normalización de Texto
Aplicado a `country`, `city` y `job_title`:
- Conversión a minúsculas.
- Eliminación de acentos.
- Limpieza de espacios.

Columnas generadas:
- `country_norm`
- `city_norm`
- `job_title_norm`

### Estandarización Geográfica
- Reglas basadas en expresiones regulares (regex).
- Mapeo manual de variantes.
- Filtrado de entradas inválidas.

Columnas finales:
- `country_clean`
- `city_clean`

### Procesamiento Salarial
- Conversión de `annual_salary_raw` y `additional_compensation_raw` a numérico.
- Uso de tipo entero nullable (`Int64`) para preservar valores faltantes.

Columnas generadas:
- `annual_salary`
- `additional_compensation`

### Resolución de Moneda
- Normalización de códigos de moneda.
- Consolidación en `currency_final`.

### Enriquecimiento Cambiario
- Conversión a USD mediante tasas FX públicas.
- Conversión de USD a COP usando TRM oficial de Colombia.
- Registro de metadatos en: docs/fx_metadata.json en el que se reporta la fecha en que se extraen la TRM

### Columnas generadas:
- `salary_USD`
- `salary_COP`
- `additional_compensation`
- `compensation_COP`
- `total_compensation_COP`

## Carga 
El dataset limpio final se almacena en:
data/processed/salary_clean_global.csv
data/processed/salary_clean_global.xlsx

Este archivo es la fuente oficial para:
- Construcción del dashboard.
- Análisis comparativo internacional.
- Visualización y storytelling.

#  Dashboard

El dashboard incluye los siguientes módulos:

##  Módulo 1 – Contador de respuestas
Total de registros válidos.

##  Módulo 2 – Mapa geográfico
Visualización por país mostrando la distribución de respuestas y el max salario entre paises 

##  Módulo 3 – Industria vs Salario Promedio
Promedio de salario por industria.

##  Módulo 4 – Análisis adicional
Participación el salario total (COP) según los años de experiencia

---

# 📘 Documentación

## Diccionario de datos (base original)
Archivo: `docs/data_dictionary_v1.xlsx`

- **Variables** para facilitar trazabilidad.
- Incluye por cada variable:
  - **Tipo de variable** (número, texto, fecha, etc.).
  - **Descripción en español** (qué representa y cómo interpretarla).

## Diccionario de datos (base modelada)
Archivo: `docs/data_dictionary_v2_modeled.xlsx`

- **Variables posteriores al modelado** (nombres estandarizados de forma consistente en todo el proyecto).
- Incluye por cada variable:
  - **Tipo de variable**.
  - **Descripción corta en español** (máx. 1 párrafo).
- Contiene también los **campos creados** 

## Guía de implementación (paso a paso)
Para replicar la actualización del dataset y aplicar el modelado sobre nuevas versiones, ver:

- `docs/ETL_process.md`
- `docs/paso_a_paso.md`