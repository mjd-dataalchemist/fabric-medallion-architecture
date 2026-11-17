# Orquestación de procesos en Microsoft Fabric

## 1. Objetivo

Este documento describe las dos estrategias de orquestación implementadas en el laboratorio de arquitectura Medallion de Microsoft Fabric:

- Orquestación visual mediante Fabric Data Factory Pipelines.
- Orquestación programática mediante Notebooks y `notebookutils.notebook.runMultiple()`.

El objetivo es comparar ambas alternativas y comprender sus posibilidades de aplicación en arquitecturas de datos empresariales.

## 2. Arquitectura del laboratorio

El laboratorio utiliza tres capas de datos:

- Bronze: almacenamiento de datos originales.
- Silver: datos transformados y depurados.
- Gold: datos preparados para el análisis.

Se han desarrollado cuatro Notebooks:

| Notebook | Responsabilidad |
|---|---|
| `00_orquestador` | Coordina la ejecución de los procesos mediante código |
| `01_bronze_to_silver` | Transforma datos de Bronze a Silver |
| `02_silver_to_gold` | Transforma datos de Silver a Gold |
| `03_validar_gold` | Comprueba la integridad de la capa Gold |

Además, se ha creado un Pipeline denominado `PL_DP600_Orchestration`, que coordina los mismos procesos mediante actividades visuales.

## 3. Orquestación mediante Pipeline

Se ha implementado un Pipeline utilizando actividades de tipo Notebook.

**Pipeline:** `PL_DP600_Orchestration`

El flujo contiene tres actividades:

1. `ACT_01_Bronze_to_Silver`
2. `ACT_02_Silver_to_Gold`
3. `ACT_03_Validar_Gold`

Las actividades están conectadas mediante dependencias *On success*, de manera que cada proceso comienza únicamente cuando el anterior finaliza correctamente.

Esta implementación permite:

- Diseñar visualmente los flujos de ejecución.
- Establecer dependencias entre actividades.
- Ejecutar y monitorizar los procesos.
- Consultar el historial de ejecuciones.
- Gestionar la programación y los errores de ejecución.

## 4. Orquestación mediante Notebook

Se ha desarrollado el Notebook `00_orquestador`, que coordina los procesos mediante `notebookutils.notebook.runMultiple()`.

La orquestación utiliza una estructura DAG (*Directed Acyclic Graph*) para definir las dependencias entre los procesos.

Se utilizan dos funciones principales:

- `notebookutils.notebook.validateDAG()`: valida la estructura del grafo.
- `notebookutils.notebook.runMultiple()`: ejecuta los Notebooks respetando las dependencias definidas.

La secuencia implementada es:

1. Bronze → Silver.
2. Silver → Gold, una vez finalizado correctamente el primer proceso.
3. Validación de Gold, una vez finalizado correctamente el segundo proceso.

La concurrencia se establece en 1 para ejecutar los procesos secuencialmente y limitar el consumo de recursos Spark.

## 5. Gestión de errores

En la implementación mediante Notebook, los resultados devueltos por `runMultiple()` se inspeccionan para identificar las actividades que han generado excepciones.

En la implementación mediante Pipeline, los errores se pueden consultar desde el historial de ejecuciones y la supervisión de actividades.

Ambas alternativas permiten identificar los procesos que no han finalizado correctamente, aunque utilizan mecanismos de gestión y monitorización diferentes.

## 6. Comparación entre las dos estrategias

| Característica | Pipeline | Notebook |
|---|---|---|
| Definición | Visual | Python |
| Dependencias | Configuración visual | DAG |
| Ejecución de Notebooks | Sí | Sí |
| Ejecución paralela | Sí | Sí |
| Gestión de errores | Actividades y dependencias | Código Python |
| Monitorización | Integrada en Fabric | Resultados y monitorización Spark |
| Actividades distintas de Notebook | Sí | No directamente |
| Programación | Integrada | Requiere un mecanismo externo o un Pipeline |

## 7. Consideraciones técnicas

La orquestación mediante Notebook es especialmente adecuada para procesos centrados en Spark y PySpark.

La reutilización de sesiones Spark requiere cumplir los requisitos de compatibilidad de High concurrency, incluyendo el mismo workspace, usuario, Lakehouse predeterminado, configuración Spark y bibliotecas.

La orquestación mediante Pipeline resulta más flexible cuando se necesita integrar diferentes tipos de actividades, programación, monitorización centralizada y gestión operativa.

## 8. Conclusiones

El laboratorio demuestra dos estrategias nativas de orquestación en Microsoft Fabric:

- **Pipeline:** orquestación visual de procesos de datos mediante actividades y dependencias.
- **Notebook:** orquestación programática mediante DAG y `runMultiple()`.

Ambas alternativas permiten coordinar el procesamiento de la arquitectura Medallion, pero responden a necesidades operativas diferentes.

Para escenarios empresariales con procesos heterogéneos, programación y monitorización centralizada, Pipeline constituye una opción especialmente adecuada. Para procesos centrados en Spark, la orquestación mediante Notebook proporciona flexibilidad y control programático.

## 9. Estructura de documentación relacionada

- `docs/architecture.md`: arquitectura general del laboratorio.
- `docs/data-flow.md`: flujo de datos entre las capas Medallion.
- `docs/implementation-notes.md`: notas técnicas y decisiones de implementación.
- `docs/notebook-orchestration.md`: orquestación mediante Notebook.
- `pipelines/pipeline-design.md`: diseño y configuración del Pipeline.