# ETL Process – Ask A Manager Salary Survey

## Project Overview

This project implements a complete ETL (Extract, Transform, Load) pipeline for the Ask A Manager Salary Survey dataset.  
The objective is to standardize salary information globally and enable cross-country comparisons in a unified currency (COP and USD).

# 1. Extract

## Data Source

The dataset originates from the public salary survey conducted by *Ask a Manager*.

Original survey form:
https://www.askamanager.org/2021/04/how-much-money-do-you-make-4.html

Data extraction method:

- Programmatic CSV export from Google Sheets
- Accessed using a constructed export URL with spreadsheet ID and GID
- Loaded into Python using `pandas.read_csv()`
- A static copy of the dataset is saved locally in the project repository to ensure reproducibility.

# 2. Transform

## 2.1 Column Standardization

- Original long column names renamed to snake_case identifiers.
- Ensures consistency and modeling clarity.


## 2.2 Text Normalization

Applied `normalize_text()` to:

- country
- city
- job_title

Normalization steps:

- Lowercase conversion
- Accent removal
- Whitespace trimming

Output columns:

- `country_norm`
- `city_norm`
- `job_title_norm`


## 2.3 Country Standardization

Implemented rule-based regex matching:

- Pattern-based country mapping
- Manual variant mapping 
- Phrase-length filtering for invalid country entries

Output column:

- `country_clean`


## 2.4 Salary Parsing

The survey form enforces numeric-only input for the salary field.  
It does not allow:

- Decimal values  
- Currency symbols  
- Commas  
- Letters  
- Special characters  


Transformation applied:

- Converted `annual_salary_raw` to numeric using `pandas.to_numeric()`
- Cast to nullable integer (`Int64`) to preserve potential missing values

Output column:

- `annual_salary`

## 2.5 Currency Resolution

Standardized currency codes:

- Used `currency`
- If "Other", cleaned `currency_other`
- Created unified column:

Output column:

- `currency_final`

---

## 2.6 FX Enrichment

### FX API

- USD base FX rates retrieved via public API
- Used for conversion to USD

### TRM (Colombia)

- Official USD → COP rate retrieved via Colombian public data source
- Stored date and rate for reproducibility

Generated columns:

- `salary_USD`
- `salary_COP`

FX metadata stored in:

```
/docs/fx_metadata.json
```

---

# 3. Load

Final cleaned dataset stored in:

```
/data/processed/salary_clean_global.csv
/data/processed/salary_clean_global.xlsx
```

This dataset is used for:

- Dashboard creation
- Cross-country salary analysis
- Storytelling and visualization

