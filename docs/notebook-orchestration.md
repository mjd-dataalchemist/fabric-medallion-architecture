# Orquestación de procesos en Microsoft Fabric

## 1. Objetivo

Este documento describe las dos estrategias de orquestación implementadas en el laboratorio de arquitectura Medallion de Microsoft Fabric:

- Orquestación visual mediante Fabric Data Factory Pipelines.
- Orquestación programática mediante Notebooks y `notebookutils.notebook.run()`.

El objetivo es comparar ambas alternativas y comprender sus posibilidades de aplicación en arquitecturas de datos.

## 2. Arquitectura del laboratorio

El laboratorio utiliza tres capas de datos, implementadas mediante Lakehouses independientes:

- Bronze: `LH_Bronze`, almacenamiento de datos originales.
- Silver: `LH_Silver`, datos transformados y depurados.
- Gold: `LH_Gold`, datos preparados para el análisis.

Se han desarrollado cuatro Notebooks:

| Notebook | Responsabilidad |
|---|---|
| `00_orquestador` | Coordina la ejecución secuencial de los procesos mediante código, reintentos y control de resultados |
| `01_bronze_to_silver` | Transforma datos de Bronze a Silver |
| `02_silver_to_gold` | Transforma datos de Silver a Gold |
| `03_validar_gold` | Comprueba la integridad y calidad de la capa Gold |

Además, se ha creado un Pipeline denominado `PL_DP600_Orchestration`, que coordina los procesos mediante actividades visuales.

## 3. Orquestación mediante Pipeline

Se ha implementado un Pipeline utilizando actividades de tipo Notebook.

**Pipeline:** `PL_DP600_Orchestration`

El flujo contiene tres actividades:

1. `ACT_01_Bronze_to_Silver`: ejecuta `01_bronze_to_silver`.
2. `ACT_02_Silver_to_Gold`: ejecuta `02_silver_to_gold`.
3. `ACT_03_Validar_Gold`: ejecuta `03_validar_gold`.

Las actividades están conectadas mediante dependencias `Succeeded`, de manera que cada proceso comienza únicamente cuando el anterior finaliza correctamente.

Esta implementación permite:

- Diseñar visualmente los flujos de ejecución.
- Establecer dependencias entre actividades.
- Ejecutar y monitorizar los procesos.
- Consultar el historial de ejecuciones.
- Gestionar la programación y los errores de ejecución.

## 4. Orquestación mediante Notebook

Se ha desarrollado el Notebook `00_orquestador`, que coordina los procesos mediante `notebookutils.notebook.run()`.

La implementación ejecuta los notebooks de forma secuencial, respetando el orden de las capas Medallion y la validación final.

La secuencia implementada es:

1. Bronze → Silver.
2. Silver → Gold, una vez finalizado correctamente el primer proceso.
3. Validación de Gold, una vez finalizado correctamente el segundo proceso.

El orquestador incorpora mecanismos de reintento y comprueba los resultados devueltos por las ejecuciones para identificar posibles errores.

Este mecanismo permite centralizar en un notebook la coordinación de los procesos y el control de sus resultados.

## 5. Gestión de errores

En la implementación mediante Notebook, los resultados de `notebookutils.notebook.run()` se inspeccionan para identificar las ejecuciones que han generado excepciones.

El mecanismo de reintentos permite volver a ejecutar un notebook cuando se produce un fallo, según los parámetros configurados en el orquestador.

En la implementación mediante Pipeline, los errores se pueden consultar desde el historial de ejecuciones y la supervisión de actividades.

Las dependencias `Succeeded` impiden que una actividad posterior se ejecute si la anterior no finaliza correctamente.

Ambas alternativas permiten identificar procesos que no han finalizado correctamente, aunque utilizan mecanismos de gestión y monitorización diferentes.

## 6. Comparación entre las dos estrategias

| Característica | Pipeline | Notebook |
|---|---|---|
| Definición | Visual | Python |
| Dependencias | Configuración visual | Control secuencial mediante código |
| Ejecución de Notebooks | Sí | Sí |
| Ejecución paralela | Sí, según configuración | No en la implementación de esta PoC |
| Gestión de reintentos | Configuración de actividades | Código Python |
| Gestión de errores | Actividades y dependencias | Excepciones y resultados de ejecución |
| Monitorización | Integrada en Fabric | Resultados y monitorización Spark |
| Actividades distintas de Notebook | Sí | No directamente |
| Programación | Integrada | Requiere un mecanismo externo o un Pipeline |

## 7. Consideraciones técnicas

La orquestación mediante Notebook es especialmente adecuada para procesos centrados en Spark y PySpark cuando se necesita controlar la secuencia de ejecución desde código.

La reutilización de sesiones Spark requiere cumplir los requisitos de compatibilidad de High concurrency, incluyendo las condiciones relativas al workspace, usuario, Lakehouse predeterminado, configuración Spark y bibliotecas.

La orquestación mediante Pipeline resulta más flexible cuando se necesita integrar diferentes tipos de actividades, programación, monitorización centralizada y gestión operativa.

Durante las pruebas de este laboratorio también se observó que la disponibilidad de recursos de la capacidad Trial puede condicionar la creación de sesiones Spark y la ejecución de los procesos.

## 8. Conclusiones

El laboratorio demuestra dos estrategias de orquestación en Microsoft Fabric:

- **Pipeline:** orquestación visual de procesos de datos mediante actividades y dependencias.
- **Notebook:** orquestación programática secuencial mediante `notebookutils.notebook.run()`, con reintentos y control de errores.

Ambas alternativas permiten coordinar el procesamiento de la arquitectura Medallion, pero responden a necesidades operativas diferentes.

Para escenarios empresariales con procesos heterogéneos, programación y monitorización centralizada, Pipeline constituye una opción especialmente adecuada. Para procesos centrados en Spark, la orquestación mediante Notebook proporciona flexibilidad y control programático.

En esta PoC, el Pipeline constituye el mecanismo principal de orquestación, mientras que `00_orquestador` demuestra una alternativa programática.

## 9. Estructura de documentación relacionada

- `docs/architecture.md`: arquitectura general del laboratorio.
- `docs/data-flow.md`: flujo de datos entre las capas Medallion.
- `docs/implementation-notes.md`: notas técnicas y decisiones de implementación.
- `docs/notebook-orchestration.md`: orquestación mediante Notebook.
- `pipelines/pipeline-design.md`: diseño y configuración del Pipeline.