# Diseño de arquitectura

## Decisión: un Lakehouse por capa

Se eligió separar Bronze, Silver y Gold en tres Lakehouses para demostrar aislamiento lógico, organización y permisos diferenciados por capa.

| Capa | Lakehouse | Responsabilidad |
|---|---|---|
| Bronze | `LH_DP600_Lab` | Aterrizaje y conservación de datos de origen |
| Silver | `LH_Silver` | Limpieza, tipado, deduplicación y validación |
| Gold | `LH_Gold` | Datos preparados para consumo analítico |

Medallion es un patrón arquitectónico, no una obligación de crear tres Lakehouses. También puede implementarse en uno solo.

## Procesamiento

Los notebooks PySpark leen tablas Delta de la capa anterior y escriben resultados en la siguiente. Para acceder a Lakehouses distintos se utilizan referencias o rutas de OneLake específicas del entorno.

## Consumo

Gold contiene un modelo dimensional sencillo. El modelo semántico de Power BI es un artefacto independiente que consume las tablas Gold; no forma parte física del Lakehouse.
