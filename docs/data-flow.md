# Flujo de datos y reglas

## Bronze

Entradas: `fabric_poc_sales.csv` y `fabric_poc_customers.csv`.

Tablas Delta: `Bronze_Sales` y `Bronze_Customers`.

## Silver

Ventas:
- Conversión de `SaleId`, `CustomerId` y `Quantity` a enteros.
- Conversión de `SaleDate` a fecha y `UnitPrice` a numérico.
- Deduplicación por `SaleId`.
- Exclusión de identificadores/fechas nulos, cantidades no positivas y precios negativos.
- Cálculo de `TotalAmount = Quantity * UnitPrice`.

Clientes:
- Conversión de `CustomerId` a entero.
- Deduplicación por `CustomerId`.
- Exclusión de identificadores o nombres nulos.

Tablas: `Sales_Clean` y `Customers_Clean`.

## Gold

- `FactSales`: granularidad de una fila por venta.
- `DimCustomer`: granularidad de una fila por cliente.

`CustomerId` relaciona ambas tablas. En Power BI se modela habitualmente como relación 1:* desde la dimensión hacia la tabla de hechos.

## Ejecución del flujo Medallion

El flujo de transformación se divide en dos notebooks de Microsoft Fabric:

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

### 3. Orquestación

El pipeline `PL_Medallion_ETL` coordina la ejecución secuencial de los notebooks:

1. Ejecuta `NB_Bronze_to_Silver`.
2. Si finaliza correctamente, ejecuta `NB_Silver_to_Gold`.

Las transformaciones se realizan en los notebooks PySpark; el pipeline se encarga de la orquestación y las dependencias entre actividades.

### 4. Validación

La ejecución se considera correcta cuando se generan las seis tablas esperadas y sus recuentos coinciden con los datos de prueba:

- Bronze: 10 ventas y 6 clientes.
- Silver: 10 ventas y 6 clientes.
- Gold: 10 registros en `FactSales` y 6 en `DimCustomer`.
