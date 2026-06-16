# 🛒 Análisis de Ventas — Supermercado de Verduras
### Pipeline completo: Python → Modelo Estrella → Power BI

![Dashboard](dashboard_preview.png)

---

## 📌 Descripción del Proyecto

Proyecto de análisis de datos end-to-end sobre **878,042 transacciones** de ventas de un supermercado de verduras en China. El proceso incluye depuración de datos con Python, construcción de un modelo estrella y visualización en Power BI.

**Período analizado:** Julio 2020 – Junio 2023  
**Moneda:** RMB (Yuan Chino) convertido a USD (1 RMB = 0.14 USD)  
**Dataset:** [Supermarket Sales Data — Kaggle (yapwh1208)](https://www.kaggle.com/datasets/yapwh1208/supermarket-sales-data)

---

## 🎯 Objetivos

- Depurar y transformar datos crudos con Python/Pandas
- Construir un modelo estrella optimizado para Power BI
- Identificar patrones de ventas, rentabilidad y merma
- Visualizar KPIs ejecutivos en un dashboard interactivo

---

## 🔍 Hallazgos Clave

| Hallazgo | Detalle |
|---|---|
| 💰 Ingresos totales | $472,083 USD (3 años) |
| 📦 Categoría líder | Vegetales de Hoja — 32% de ingresos |
| ⏰ Hora pico | 10:00 – 11:00 hrs |
| 📅 Días de mayor venta | Sábado y Domingo (18% y 17%) |
| 🏷️ Ventas sin descuento | 95.22% |
| 📉 Merma total | 2,366 kg (0.5% del total vendido) |
| 📈 Mes pico | Febrero 2021 — $25,043 USD |

---

## 🛠️ Stack Tecnológico

| Herramienta | Uso |
|---|---|
| Python 3 | Depuración y transformación de datos |
| Pandas | Manipulación de DataFrames |
| Power BI Desktop | Modelado y visualización |
| DAX | Medidas y columnas calculadas |
| GitHub | Control de versiones |

---

## 📁 Estructura del Proyecto

```
supermarket_sales/
│
├── 📓 analisis_supermercado_final.ipynb  # Notebook de depuración
│
├── 📂 data_raw/                          # Datos originales de Kaggle
│   ├── annex1.csv                        # Catálogo de productos
│   ├── annex2.csv                        # Transacciones de ventas
│   ├── annex3.csv                        # Precios mayoristas
│   └── annex4.csv                        # Tasa de pérdida por producto
│
├── 📂 data_clean/                        # Tablas depuradas para Power BI
│   ├── fact_ventas.csv                   # Tabla de hechos (878K filas)
│   ├── dim_producto.csv                  # Dimensión producto (251)
│   ├── dim_fecha.csv                     # Dimensión fecha (1,095 días)
│   ├── dim_precio_mayorista.csv          # Dimensión precios (55,982)
│   └── dim_perdidas.csv                  # Dimensión merma (251)
│
└── 📊 supermarket.pbix                   # Archivo Power BI
```

---

## 📊 Dataset Original

| Archivo | Descripción | Filas |
|---|---|---|
| `annex1.csv` | Catálogo de 251 productos con categoría | 251 |
| `annex2.csv` | Transacciones individuales de venta por peso (kg) | 878,503 |
| `annex3.csv` | Precio de compra mayorista por producto y fecha | 55,982 |
| `annex4.csv` | Tasa de merma (pérdida) por producto en % | 251 |

---

## ⭐ Modelo Estrella

```
                    dim_fecha
                        │ Date (1)
                        │
fact_ventas[Date*] ─────┘
fact_ventas[Item Code*] ──────► dim_producto[Item_Code (1)]
                                        │
                                dim_perdidas[Item_Code*]

dim_precio_mayorista → usado con LOOKUPVALUE en DAX
```

### Relaciones en Power BI
| Desde (*) | Campo | Hacia (1) | Campo | Tipo |
|---|---|---|---|---|
| fact_ventas | Item Code | dim_producto | Item_Code | Muchos a uno |
| fact_ventas | Date | dim_fecha | Date | Muchos a uno |
| dim_perdidas | Item_Code | dim_producto | Item_Code | Muchos a uno |

---

## 🧹 Proceso de Depuración (Python)

### fact_ventas
- Conversión de `Date` a datetime y extracción de componentes temporales
- Filtro de devoluciones — solo conservar `sale`
- Filtro de cantidades negativas
- Cálculo de `Total_Rev_RMB = Quantity_Kg × Unit_Price_RMB`
- Conversión a USD (`× 0.14`)
- Conversión de descuento a flag numérico (0/1)

### dim_fecha
- Generación de calendario continuo sin huecos
- Atributos: Year, Quarter, Month, Week, Day, Day_Name, Is_Weekend

### dim_precio_mayorista
- Conversión de fecha a datetime
- Agregado de columna en USD

### dim_perdidas
- Limpieza de BOM en nombres de columnas
- Conversión de Loss_Rate_Pct a numérico

---

## 📐 Medidas DAX Principales

```dax
-- KPIs
Total Ingresos USD = SUM(fact_ventas[Total_Rev_USD])
Total Kg Vendidos = SUM(fact_ventas[Quantity_Kg])
Total Transacciones = COUNTROWS(fact_ventas)
% Con Descuento = DIVIDE(COUNTROWS(FILTER(fact_ventas, fact_ventas[Discount_Flag]=1)), COUNTROWS(fact_ventas)) * 100

-- Temporal
Ingresos Mes Anterior = CALCULATE([Total Ingresos USD], DATEADD(dim_fecha[Date], -1, MONTH))
Crecimiento MoM % = DIVIDE([Total Ingresos USD] - [Ingresos Mes Anterior], [Ingresos Mes Anterior]) * 100

-- Rentabilidad
Costo Total USD = SUMX(fact_ventas, fact_ventas[Quantity_Kg] * LOOKUPVALUE(dim_precio_mayorista[Wholesale_Price_USD], dim_precio_mayorista[Item_Code], fact_ventas[Item Code], dim_precio_mayorista[Date], fact_ventas[Date]))
Margen Bruto USD = [Total Ingresos USD] - [Costo Total USD]
Margen % = DIVIDE([Margen Bruto USD], [Total Ingresos USD]) * 100

-- Pérdidas
Kg Perdidos Estimados = SUMX(fact_ventas, fact_ventas[Quantity_Kg] * LOOKUPVALUE(dim_perdidas[Loss_Rate_Pct], dim_perdidas[Item_Code], fact_ventas[Item Code]) / 100)
% Merma = DIVIDE([Kg Perdidos Estimados], [Total Kg Vendidos]) * 100
```

---

## 🚀 Cómo Reproducir el Proyecto

1. Descarga el dataset de [Kaggle](https://www.kaggle.com/datasets/yapwh1208/supermarket-sales-data)
2. Coloca los 4 archivos CSV en la carpeta `data_raw/`
3. Ejecuta `analisis_supermercado_final.ipynb` celda por celda
4. Los 5 CSVs limpios se generarán automáticamente
5. Abre `supermarket.pbix` en Power BI Desktop
6. Actualiza las rutas de los archivos en Power Query

> ⚠️ **Nota de formato:** Los CSVs usan `;` como separador y `,` como decimal (formato español). En Power Query no requieren configuración adicional si tu Windows está en español.

---

## 👤 Autor

**Diego Encalada**  
Ingeniero en Ciencias de la Computación  
Analista BI Jr. — Ambiensa S.A.  
Maestría en Visual Analytics & Big Data — UNIR  

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Diego_Encalada-0077B6?style=flat&logo=linkedin)](https://www.linkedin.com/in/diego-encalada-garcia-b13322271/)
[![GitHub](https://img.shields.io/badge/GitHub-Daeg1702-181717?style=flat&logo=github)](https://github.com/Daeg1702)
