# Entrega Proyecto Python EDA

## 📋 Descripción del proyecto

Análisis exploratorio de datos (EDA) sobre campañas de marketing directo de una institución bancaria. El objetivo es identificar los factores que influyen en que un cliente suscriba un depósito a plazo bancario (`y = yes/no`), a partir de variables demográficas, financieras, de campaña, macroeconómicas y de perfil de cliente.

---

## 📊 Datasets utilizados

### 1. `bank-additional.csv`

Dataset principal con 43.000 registros y 23 variables sobre clientes contactados en campañas telefónicas (2015–2019).

| Tipo | Variables |
|------|-----------|
| Demográficas | `age`, `job`, `marital`, `education` |
| Financieras | `default`, `housing`, `loan` |
| Campaña | `contact`, `duration`, `campaign`, `pdays`, `previous`, `poutcome` |
| Macroeconómicas | `emp.var.rate`, `cons.price.idx`, `cons.conf.idx`, `euribor3m`, `nr.employed` |
| Geo/Tiempo | `date`, `latitude`, `longitude` |
| Target | `y` (suscribió: yes/no) |

### 2. `customer-details.xlsx`

Archivo Excel con 3 hojas (clientes 2012, 2013 y 2014) combinadas en un único DataFrame de 43.170 registros con variables de perfil del cliente: `Income`, `Kidhome`, `Teenhome`, `Dt_Customer`, `NumWebVisitsMonth`, `ID`.

---

## 🗂️ Estructura del repositorio

```
├── README.md
├── DataProyecto/
│   └── Bruto/
│       ├── bank-additional.csv
│       └── customer-details.xlsx
└── EDA/
    ├── EDA_bank-additional.ipynb
    └── EDA_Clientes.ipynb
```

---

## 🔧 Pasos seguidos

### Notebook 1 — `EDA_bank-additional.ipynb`

**Carga de datos**

- Carga del CSV principal con `pd.read_csv()`

**Limpieza y transformación**

- Eliminación de la columna `Unnamed: 0` (índice residual)
- Conversión de columnas numéricas almacenadas como `object` (`cons.price.idx`, `cons.conf.idx`, `euribor3m`, `nr.employed`) — problema causado por el uso de comas como separador decimal, resuelto con `.str.replace(',', '.')` y `pd.to_numeric()`
- Parseo de fechas en español (ej. `2-agosto-2019`) mediante diccionario de traducción de meses y conversión con `pd.to_datetime()`
- Conversión de columnas categóricas (`job`, `marital`, `education`, `contact`, `poutcome`) a dtype `category`
- Conversión de `age` a `Int64` (acepta nulos)

**Análisis descriptivo**

- `df.describe().T` para estadísticas de tendencia central y dispersión
- Identificación de valores nulos: ~5.000 en `age`, ~9.000 en `euribor3m`, ~500 en `cons.price.idx`
- Detección de valor especial `pdays = 999` (indica cliente no contactado previamente)

---

### Notebook 2 — `EDA_Clientes.ipynb`

**Carga de datos**

- Carga de las 3 hojas del Excel con `pd.read_excel(sheet_name=...)` y combinación con `pd.concat()`

**Limpieza y transformación**

- Eliminación de `Unnamed: 0`
- Verificación de nulos (ninguno en todo el dataset)
- Verificación y eliminación de duplicados por `ID`
- Revisión de outliers en `Income` mediante método IQR
- Conversión de `Kidhome` y `Teenhome` a dtype `category`
- `Dt_Customer` ya estaba en `datetime64` correctamente desde la carga

---

## 📊 Visualizaciones realizadas

### `EDA_bank-additional.ipynb`

1. Distribución del target (`y`) — desbalance de clases
2. Boxplots de variables numéricas vs target
3. Tasas de conversión por variables categóricas
4. Mapa de correlaciones entre variables numéricas
5. Evolución temporal de la tasa de conversión mensual (con anotación de máximo y mínimo)
6. Distribución geográfica de clientes (scatter latitud/longitud)

### `EDA_Clientes.ipynb`

1. Histogramas de `Income` y `NumWebVisitsMonth` con KDE
2. Countplots de `Kidhome` y `Teenhome`
3. Comparativa `Kidhome` vs `Teenhome`
4. Evolución temporal de nuevas altas de clientes por mes (con anotación de bajada)
5. Boxplots de `Income` por `Kidhome` y `Teenhome`
6. Scatter plot `Income` vs `NumWebVisitsMonth`

---

## 📈 Principales hallazgos

### Dataset `bank-additional.csv`

**Variables numéricas**

- **`duration`** es la variable más discriminante: las llamadas más largas se asocian consistentemente con suscripción. Puede contener *data leakage* ya que la duración se conoce solo al finalizar la llamada.
- **`euribor3m`**, **`emp.var.rate`** y **`nr.employed`** muestran que los clientes suscriben más cuando los tipos de interés son bajos
- **`previous`**: los clientes con historial de contacto previo tienen mayor propensión a suscribir.
- **`age`** y **`campaign`** no muestran diferencias significativas entre grupos.

**Correlaciones entre variables numéricas**

- `emp.var.rate`, `euribor3m` y `nr.employed` están altamente correlacionadas entre sí (r > 0.90) — prácticamente redundantes, miden el mismo estado.
- `pdays` y `previous` muestran correlación negativa (-0.59), coherente con su significado.
- `duration`, `age` y `campaign` son independientes del resto.

**Variables categóricas**

- **`poutcome`** es la variable más potente: los clientes con resultado exitoso en la campaña anterior convierten al, frente al ~8% sin historial.
- **`job`**: los **estudiantes** y **jubilados** presentan las tasas de conversión más altas. Los trabajadores *blue-collar* son el perfil con menor conversión (~5%).
- **`contact`**: el contacto por **móvil dobla** la tasa de conversión respecto al teléfono fijo.
- **`education`**: a mayor nivel educativo universitario, mayor conversión.
- **`marital`**: los solteros convierten ligeramente más, pero las diferencias entre grupos son pequeñas.

**Evolución temporal**

- La tasa de conversión se mantiene estable entre el durante todo el período 2015–2019, sin tendencia clara ni estacionalidad fuerte. El pico máximo se registra en septiembre 2016 (~14.5%) y el mínimo en julio 2017.

**Distribución geográfica**

- Las coordenadas cubren el territorio de EEUU pero la distribución de conversiones es completamente homogénea. La ubicación geográfica no aporta valor predictivo y las variables `latitude` y `longitude` son sintéticas.

---

### Dataset `customer-details.xlsx`

El análisis exploratorio reveló que este dataset es **completamente sintético**, generado con distribuciones artificiales. Los indicadores que lo confirman son:

- **`Income`** presenta una distribución perfectamente uniforme entre 5.841 y 180.802 — imposible en datos reales de ingresos.
- **`NumWebVisitsMonth`** muestra una distribución bimodal anómala con un pico artificial en el valor máximo (32).
- **`Kidhome`** y **`Teenhome`** tienen exactamente el mismo número de registros para los valores 0, 1 y 2 (~14.300 cada uno) — distribución uniforme perfecta.
- El scatter plot `Income` vs `NumWebVisitsMonth` forma una **cuadrícula perfecta**, confirmando que ambas variables son totalmente independientes y generadas aleatoriamente.
- **`Income` no varía** en absoluto según el número de hijos o adolescentes en casa.

La única variable con comportamiento real es **`Dt_Customer`**, que muestra una caída significativa de nuevas altas de clientes entre noviembre de 2012 y abril de 2013.

---

## 🏁 Conclusiones generales

Los factores con mayor influencia en la suscripción de un depósito son, por orden de importancia:

1. **Resultado de campaña anterior** (`poutcome = success`) — clientes con historial positivo convierten al 65%
2. **Duración de la llamada** (`duration`) — mayor duración implica mayor conversión
3. **Contexto macroeconómico** (`euribor3m` bajo) — tipos de interés bajos favorecen la suscripción
4. **Perfil del cliente** (`job`: estudiantes y jubilados; `contact`: móvil)

Las variables geográficas y temporales no aportan poder predictivo relevante. El dataset `customer-details.xlsx`, al ser sintético, no permite extraer conclusiones de negocio fiables más allá de la evolución temporal de altas de clientes.

---

## ▶️ Cómo ejecutar el proyecto

1. Clonar el repositorio
2. Instalar dependencias: `pip install pandas matplotlib seaborn openpyxl`
3. Ejecutar los notebooks en orden:
   - `EDA/EDA_Clientes.ipynb`
   - `EDA/EDA_bank-additional.ipynb`
4. Los datos en bruto deben estar en `DataProyecto/Bruto/`
