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
