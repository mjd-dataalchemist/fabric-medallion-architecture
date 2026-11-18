# Flujo de datos y reglas

## Bronze

Entradas:
- `fabric_poc_sales.csv`
- `fabric_poc_customers.csv`

Tablas Delta:
- `Bronze_Sales`
- `Bronze_Customers`

Esta capa conserva los datos de origen para su posterior procesamiento.

## Silver

**Ventas**

- Conversión de `SaleId`, `CustomerId` y `Quantity` a enteros.
- Conversión de `SaleDate` a fecha y `UnitPrice` a numérico.
- Deduplicación por `SaleId`.
- Exclusión de identificadores y fechas nulos, cantidades no positivas y precios negativos.
- Cálculo de `TotalAmount = Quantity * UnitPrice`.

**Clientes**

- Conversión de `CustomerId` a entero.
- Deduplicación por `CustomerId`.
- Exclusión de identificadores o nombres nulos.

Tablas Delta:
- `Sales_Bronze`
- `Customers_Bronze`

## Gold

La capa Gold contiene un modelo dimensional sencillo:

- `FactSales`: granularidad de una fila por venta.
- `DimCustomer`: granularidad de una fila por cliente.

`CustomerId` relaciona ambas tablas. En un modelo semántico de Power BI, la relación habitual sería de uno a muchos (1:*) desde `DimCustomer` hacia `FactSales`.

## Ejecución del flujo Medallion

El flujo de transformación se divide en dos notebooks de Microsoft Fabric.

### 1. Bronze → Silver

Notebook: `01_bronze_to_silver.ipynb`

- Lee `Bronze_Sales` y `Bronze_Customers` desde `LH_Bronze`.
- Convierte los tipos de datos a los formatos adecuados.
- Elimina duplicados.
- Filtra registros no válidos.
- Calcula el importe total de cada venta (`TotalAmount`).
- Escribe las tablas resultantes en `LH_Silver`.

Tablas de salida:
- `Sales_Bronze`
- `Customers_Bronze`

### 2. Silver → Gold

Notebook: `02_silver_to_gold.ipynb`

- Lee `Sales_Bronze` y `Customers_Bronze` desde `LH_Silver` mediante rutas ABFSS.
- Prepara las tablas analíticas.
- Genera la tabla de hechos `FactSales`.
- Genera la dimensión `DimCustomer`.
- Escribe ambas tablas en `LH_Gold`.

### 3. Orquestación mediante pipeline

El pipeline `PL_DP600_Orchestration` coordina la ejecución secuencial de tres actividades:

1. `ACT_01_Bronze_to_Silver`: ejecuta `01_bronze_to_silver`.
2. `ACT_02_Silver_to_Gold`: ejecuta `02_silver_to_gold` cuando la primera actividad finaliza correctamente (`Succeeded`).
3. `ACT_03_Validar_Gold`: ejecuta `03_validar_gold` cuando la segunda actividad finaliza correctamente (`Succeeded`).

Las transformaciones se realizan en los notebooks PySpark; el pipeline se encarga de la orquestación y de las dependencias entre actividades.

### 4. Orquestación alternativa mediante notebook

El notebook `00_orquestador` demuestra un mecanismo alternativo de orquestación mediante `notebookutils.notebook.runMultiple()`.

Esta ejecución permite coordinar los notebooks mediante un DAG (Directed Acyclic Graph), de forma independiente al pipeline nativo de Fabric.

En esta PoC, el pipeline nativo constituye el mecanismo principal de orquestación.

### 5. Validación de calidad en Gold

El notebook `03_validar_gold` comprueba la calidad de las tablas Gold mediante nueve controles:

1. `FactSales` no está vacía.
2. `DimCustomer` no está vacía.
3. No existen claves nulas en `FactSales`.
4. No existen valores nulos en `DimCustomer.CustomerId`.
5. `SaleId` es único en `FactSales`.
6. `CustomerId` es único en `DimCustomer`.
7. No existen ventas asociadas a clientes inexistentes.
8. `Quantity` es mayor que cero y `UnitPrice` es mayor o igual que cero.
9. `TotalAmount` coincide con el producto de `Quantity` por `UnitPrice`.

Si alguna comprobación falla, el notebook genera una excepción y la ejecución se considera fallida.

### 6. Resultados de la PoC

La ejecución satisfactoria del flujo genera seis tablas Delta con los siguientes recuentos:

| Capa | Tabla | Registros |
|---|---|---:|
| Bronze | `Bronze_Sales` | 10 |
| Bronze | `Bronze_Customers` | 6 |
| Silver | `Sales_Bronze` | 10 |
| Silver | `Customers_Bronze` | 6 |
| Gold | `FactSales` | 10 |
| Gold | `DimCustomer` | 6 |

Los nueve controles de calidad de Gold finalizaron correctamente en la ejecución validada.