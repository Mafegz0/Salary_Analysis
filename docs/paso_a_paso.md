# Script funcional — Limpieza y estandarización (cleaning_data.ipynb)

> **Objetivo:** descargar data (Google Sheets), estandarizar columnas, limpiar país/moneda/salario/compensación, convertir a COP y USD, y exportar dataset + diccionario de datos.

---

## 0) Configuración de rutas y librerías

- Importar librerías: `pandas`, `re`, `unicodedata`, `requests`, `json`, `pathlib`.
- Definir rutas de salida:
  - `../data/raw/`
  - `../data/processed/`
  - `../docs/`

---

## 1) Descargar la data desde Google Sheets

1. Definir:
   - `SPREADSHEET_ID`
   - `GID`
2. Construir URL de exportación:
   - `https://docs.google.com/spreadsheets/d/{SPREADSHEET_ID}/export?format=csv&gid={GID}`
3. Leer a DataFrame:
   - `df_raw = pd.read_csv(url)`

---

## 2) Guardar copia RAW

- Exportar `df_raw` a:
  - `../data/raw/data_raw.csv`

---

## 3) Estandarizar nombres de columnas

1. Definir lista `short_names` (nombres cortos estándar).
2. Crear `rename_dict = dict(zip(df_raw.columns, short_names))`.
3. Renombrar:
   - `df = df_raw.rename(columns=rename_dict)`

---

## 4) Diccionario de datos v1 (mapeo columnas)

1. Crear `data_dictionary_v1` con:
   - `original_name` (nombre original)
   - `new_name` (nombre corto)
   - `input_type` (dtype detectado)
2. Agregar `descripcion_es` (descripción por variable).
3. Exportar a:
   - `../docs/data_dictionary_v1.xlsx`

---

## 5) Función de normalización de texto

Definir `normalize_text(x)` para:
- minúsculas
- remover tildes (NFKD)
- eliminar caracteres especiales
- normalizar espacios

---

## 6) Crear columnas normalizadas (para reglas)

Generar:
- `country_norm = normalize_text(country_raw)`
- (opcional, si existen) `city_norm`, `job_title_norm`

---

## 7) Limpieza de país con patrones (prioridad por orden)

### 7.1 Definir reglas de patrones
Definir `pattern_rules` como lista de tuplas:
- `("USA", regex_usa)`
- `("UK", regex_uk)`
- `("Ireland", regex_ireland)`
- `("Canada", regex_canada)`

### 7.2 Definir función `apply_country_patterns(df, norm_column, out_column)`
Lógica:
1. Inicializar `out_column` como `NaN`.
2. Recorrer `pattern_rules` en orden:
   - Identificar filas donde `out_column` está vacío **y** `norm_column` hace match con el regex
   - Asignar el país estándar
3. Para filas no asignadas:
   - dejar temporalmente `country_norm` (o `NaN` para completar después)

### 7.3 Ejecutar limpieza por patrones
- `df = apply_country_patterns(df, norm_column="country_norm", out_column="country_clean")`

---

## 8) Marcar países como Unknown (ruido / inválidos)

Definir y aplicar `clean_unknown_countries(df, col="country_clean")`:

Reglas funcionales:
- Si no contiene letras → `Unknown`
- Si tiene demasiadas palabras (`max_words=4`) → `Unknown`
- Si contiene keywords de ruido (ej.: `salary`, `bonus`, `work`, `job`, `company`, `international`, etc.) → `Unknown`

Aplicar:
- `df = clean_unknown_countries(df, col="country_clean")`

---

## 9) Limpieza de salario anual

1. Crear `annual_salary` desde `annual_salary_raw`:
   - convertir a string
   - remover comas/espacios
   - `pd.to_numeric(errors="coerce")`
2. Convertir a entero nullable:
   - `astype("Int64")`

---

## 10) Estandarización de moneda

### 10.1 Preparar recursos
- `VALID_CURRENCIES` (ej.: `USD`, `COP`, `EUR`, `GBP`, etc.)
- `CURRENCY_NAME_KEYWORDS`: mapear palabras clave a códigos (ej.: `"peso colombiano" -> "COP"`)
- `TOKEN_NORMALIZATION`: normalizar tokens (ej.: `"us dollars"`, `"usd"`, `"u.s. dollars"` -> `USD`)

### 10.2 Resolver campo `currency_other`
Definir `resolve_currency_other(text)`:
- normalizar texto
- buscar keywords/tokens
- retornar código estándar o `None`

### 10.3 Función `standardize_currencies(df)`
- Crear `currency_other_clean = resolve_currency_other(currency_other)`
- Crear `currency_final`:
  - si `currency` es válida → usar `currency`
  - si `currency == "OTHER"` → usar `currency_other_clean`
  - si es no monetario (ej. contiene “equity”) → `NON_MONETARY`
  - si no se puede resolver → `UNKNOWN`

Ejecutar:
- `df = standardize_currencies(df)`

---

## 11) Obtener tasas de cambio (CUR → COP)

### 11.1 Función `fetch_rates_to_cop(symbols, base_for_api="USD")`
1. Consultar API:
   - `https://open.er-api.com/v6/latest/USD`
2. Con rates obtenidas (USD→X):
   - calcular `CUR→COP = (USD→COP) / (USD→CUR)`
3. Retornar:
   - `rates_to_cop` (dict)
   - `fx_metadata` (fecha, fuente, base, etc.)

Guardar metadata:
- `../docs/fx_metadata.json`

---

## 12) Convertir salario a COP y USD

1. Construir lista de monedas monetarias:
   - excluir `UNKNOWN` y `NON_MONETARY`
2. Asignar tasa por fila:
   - `rate_to_cop = df["currency_final"].map(rates_to_cop)`
3. Calcular:
   - `salary_COP = annual_salary * rate_to_cop`
   - `salary_USD = salary_COP / (USD_to_COP)` (tomado del API: `rates["COP"]`)

---

## 13) Limpieza de compensación adicional y total compensación

1. Crear `additional_compensation` desde `additional_compensation_raw`:
   - limpiar separadores
   - `pd.to_numeric(errors="coerce")`
2. Convertir a COP:
   - `compensation_COP = additional_compensation * rate_to_cop`
3. Total:
   - `total_compensation_COP = salary_COP + compensation_COP`

---

## 14) Exportar dataset final (processed)

Exportar `df` a:
- `../data/processed/salary_clean_global.xlsx`

---

## 15) Diccionario de datos v2 (modelado)

1. Construir `data_dictionary_v2` con:
   - `column_name`
   - `data_type`
   - `source_columns` (origen por variable derivada)
   - `transformations` (regla aplicada)
2. Exportar a:
   - `../docs/data_dictionary_v2_modeled.xlsx`
   - `../docs/data_dictionary_v2_modeled.csv`
