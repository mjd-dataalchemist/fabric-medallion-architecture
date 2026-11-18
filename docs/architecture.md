# Diseño de arquitectura

## Decisión: un Lakehouse por capa

Se eligió separar Bronze, Silver y Gold en tres Lakehouses para demostrar aislamiento lógico, organización y la posibilidad de gestionar permisos diferenciados por capa.

| Capa | Lakehouse | Responsabilidad |
|---|---|---|
| Bronze | `LH_Bronze` | Aterrizaje y conservación de datos de origen |
| Silver | `LH_Silver` | Limpieza, tipado, deduplicación y preparación de los datos |
| Gold | `LH_Gold` | Datos estructurados y preparados para consumo analítico |

Medallion es un patrón arquitectónico, no una obligación de crear tres Lakehouses. También puede implementarse utilizando diferentes esquemas o estructuras dentro de un único Lakehouse.

## Procesamiento

El procesamiento se realiza mediante notebooks PySpark que leen tablas Delta de una capa y escriben los resultados en la siguiente.

- `01_bronze_to_silver`: transforma los datos de Bronze a Silver.
- `02_silver_to_gold`: construye las tablas analíticas de Gold.
- `03_validar_gold`: ejecuta controles de calidad sobre las tablas Gold.

Para acceder a Lakehouses distintos se utilizan referencias y rutas de OneLake específicas del entorno.

La ejecución principal está coordinada mediante el pipeline `PL_DP600_Orchestration`, compuesto por tres actividades secuenciales con dependencias de ejecución correcta (`Succeeded`).

Como alternativa al pipeline nativo, el notebook `00_orquestador` implementa una orquestación programática secuencial mediante `notebookutils.notebook.run()`. Ejecuta los notebooks de transformación y validación en orden, incorpora reintentos y controla los resultados de cada ejecución.

Esta alternativa es independiente del pipeline `PL_DP600_Orchestration` y permite demostrar la coordinación programática de procesos desde un notebook.

## Consumo

La capa Gold contiene un modelo dimensional sencillo compuesto por:

- `FactSales`: tabla de hechos de ventas.
- `DimCustomer`: dimensión de clientes.

El modelo semántico de Power BI es un artefacto independiente que puede consumir las tablas Gold; no forma parte física del Lakehouse. Su implementación y la creación de informes quedan fuera del alcance de esta PoC.