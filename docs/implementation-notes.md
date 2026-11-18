# Notas de implementación

## Capacidad Spark y alta simultaneidad

Durante la ejecución del pipeline se observó el error `TooManyRequestsForCapacity` (HTTP 430) al intentar crear una sesión Livy.

El mensaje indicaba que la capacidad no podía admitir el nuevo trabajo Spark en ese momento. Por tanto, no debía interpretarse automáticamente como un defecto del código de los notebooks.

Durante el diagnóstico se revisaron los siguientes aspectos:

- Supervisión de ejecuciones.
- Trabajos Spark activos y en cola.
- Limitaciones de la capacidad Trial compartida.
- Configuración de alta simultaneidad.
- Requisitos de compatibilidad para compartir sesiones, como el Lakehouse predeterminado, la configuración Spark y las bibliotecas.

Para facilitar la ejecución en el entorno Trial se redujo la configuración de recursos del Starter Pool, limitando los nodos y ejecutores a uno.

Tras este ajuste, el pipeline se ejecutó correctamente y las tres actividades finalizaron satisfactoriamente.

Esta incidencia pone de manifiesto que la disponibilidad de recursos de la capacidad puede condicionar la ejecución de trabajos Spark en Microsoft Fabric, incluso cuando el código es correcto.

## Reproducción del laboratorio en Microsoft Fabric

### Configuración de Lakehouses

La arquitectura utiliza tres Lakehouses independientes:

- `LH_Bronze`: almacenamiento de los datos originales.
- `LH_Silver`: almacenamiento de los datos depurados y transformados.
- `LH_Gold`: almacenamiento de las tablas preparadas para el análisis.

### Configuración de los notebooks

Los notebooks de transformación utilizan como Lakehouse predeterminado el destino de sus escrituras.

| Notebook | Lakehouse predeterminado | Origen | Destino |
|---|---|---|---|
| `01_bronze_to_silver.ipynb` | `LH_Silver` | `LH_Bronze` | `LH_Silver` |
| `02_silver_to_gold.ipynb` | `LH_Gold` | `LH_Silver` | `LH_Gold` |
| `03_validar_gold.ipynb` | `LH_Gold` | `LH_Gold` | No escribe tablas de salida |

Es importante configurar correctamente el Lakehouse predeterminado antes de ejecutar cada notebook. Las operaciones `saveAsTable()` sin una ruta explícita escriben las tablas en el Lakehouse predeterminado.

El notebook `00_orquestador` constituye una alternativa de orquestación mediante `notebookutils.notebook.runMultiple()`, independiente del pipeline nativo.

### Lectura entre Lakehouses

Para acceder a tablas de un Lakehouse distinto del predeterminado se utilizan rutas ABFSS a las carpetas Delta de origen.

En el notebook `02_silver_to_gold.ipynb`, las rutas de `Sales_Bronze` y `Customers_Bronze` dependen del entorno Fabric.

No se deben publicar rutas ABFSS que contengan identificadores específicos del workspace o del entorno personal. Para reproducir el laboratorio, cada usuario debe adaptar las referencias a su propio entorno.

### Resultados de ejecución y validación

La ejecución satisfactoria del pipeline ha generado las siguientes tablas y recuentos:

| Capa | Tabla | Registros |
|---|---|---:|
| Bronze | `Bronze_Sales` | 10 |
| Bronze | `Bronze_Customers` | 6 |
| Silver | `Sales_Bronze` | 10 |
| Silver | `Customers_Bronze` | 6 |
| Gold | `FactSales` | 10 |
| Gold | `DimCustomer` | 6 |

El notebook `03_validar_gold` ejecuta nueve controles de calidad sobre las tablas Gold. En la ejecución validada, todos los controles finalizaron correctamente.

La validación se realiza mediante comprobaciones en el notebook y excepciones cuando alguna condición no se cumple. No se genera una tabla de auditoría persistente.

### Incidencia durante la reproducción

Durante la reproducción del laboratorio, las tablas de salida del primer notebook se crearon inicialmente en `LH_Bronze`.

La causa fue que `LH_Bronze` estaba configurado como Lakehouse predeterminado. Al establecer `LH_Silver` como predeterminado y volver a ejecutar el notebook, las tablas se generaron en la capa correcta.

Esta experiencia pone de manifiesto la importancia de comprobar el Lakehouse predeterminado antes de ejecutar operaciones de escritura.