# Diseño del Pipeline

Nombre: `PL_Medallion_ETL`

Secuencia:
1. `ACT_Bronze_to_Silver` → `NB_Bronze_to_Silver`.
2. Dependencia **On success**.
3. `ACT_Silver_to_Gold` → `NB_Silver_to_Gold`.

El Pipeline orquesta las actividades. Las transformaciones se implementan en notebooks PySpark.

La programación recurrente (Schedule) es opcional y se configura por separado de una ejecución manual (**Run**).
