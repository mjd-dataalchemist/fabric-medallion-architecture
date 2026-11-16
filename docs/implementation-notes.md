# Notas de implementación

## Capacidad Spark y alta simultaneidad

Durante la ejecución del Pipeline se observó `TooManyRequestsForCapacity` (HTTP 430) al crear una sesión Livy. El mensaje indica que la capacidad no podía admitir el nuevo trabajo Spark en ese momento; no debe interpretarse automáticamente como un defecto del código.

Se investigaron:
- Supervisión de ejecuciones.
- Trabajos Spark activos y en cola.
- Capacidad Trial compartida.
- Configuración de alta simultaneidad.
- Requisitos de compatibilidad para compartir sesiones: Lakehouse predeterminado, configuración Spark y bibliotecas.

La ejecución final del Pipeline se completó correctamente tras ajustar la configuración del laboratorio. Añadir capturas y detalles técnicos no sensibles en una futura revisión.

## Reproducción del laboratorio en Microsoft Fabric

### Configuración de Lakehouses

La arquitectura utiliza tres Lakehouses independientes:

- `LH_Bronze`: almacenamiento de datos originales.
- `LH_Silver`: almacenamiento de datos depurados y transformados.
- `LH_Gold`: almacenamiento de tablas preparadas para el análisis.

### Configuración de los notebooks

Cada notebook utiliza como Lakehouse predeterminado el destino de sus escrituras:

| Notebook | Lakehouse predeterminado | Origen | Destino |
|---|---|---|---|
| `01_bronze_to_silver.ipynb` | `LH_Silver` | `LH_Bronze` | `LH_Silver` |
| `02_silver_to_gold.ipynb` | `LH_Gold` | `LH_Silver` | `LH_Gold` |

Es importante configurar correctamente el Lakehouse predeterminado antes de ejecutar cada notebook. Las operaciones `saveAsTable()` sin una ruta explícita escriben las tablas en el Lakehouse predeterminado.

### Lectura entre Lakehouses

Para acceder a tablas de un Lakehouse distinto del predeterminado se utilizan rutas ABFSS a las carpetas Delta de origen.

En el notebook `02_silver_to_gold.ipynb`, las rutas de `Sales_Bronze` y `Customers_Bronze` se representan mediante marcadores genéricos. Cada usuario debe sustituirlos por las rutas de su propio entorno Fabric.

No se deben publicar rutas ABFSS que contengan identificadores específicos del workspace o del entorno personal.

### Resultados de validación

La ejecución de los dos notebooks ha generado los siguientes resultados:

| Tabla | Registros |
|---|---:|
| `Bronze_Sales` | 10 |
| `Bronze_Customers` | 6 |
| `Sales_Bronze` | 10 |
| `Customers_Bronze` | 6 |
| `FactSales` | 10 |
| `DimCustomer` | 6 |

### Incidencia durante la reproducción

Durante la reproducción del laboratorio, las tablas de salida del primer notebook se crearon inicialmente en `LH_Bronze`.

La causa fue que `LH_Bronze` estaba configurado como Lakehouse predeterminado. Al establecer `LH_Silver` como predeterminado y volver a ejecutar el notebook, las tablas se generaron en la capa correcta.

Esta experiencia pone de manifiesto la importancia de comprobar el Lakehouse predeterminado antes de ejecutar operaciones de escritura.