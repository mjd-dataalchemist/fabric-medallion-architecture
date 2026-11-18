# Pipeline de Orquestación Medallion

## 1. Objetivo

La canalización `PL_DP600_Orchestration` implementa la orquestación de extremo a extremo de la arquitectura Medallion en Microsoft Fabric.

Su responsabilidad es coordinar la ejecución secuencial de los procesos de transformación y validación de datos, desde la capa Bronze hasta Gold.

Las transformaciones se implementan mediante notebooks PySpark y la canalización controla el orden de ejecución y las dependencias entre actividades.

## 2. Flujo de ejecución

La canalización está formada por tres actividades:

```text
ACT_01_Bronze_to_Silver
          |
          | On success (Succeeded)
          v
ACT_02_Silver_to_Gold
          |
          | On success (Succeeded)
          v
ACT_03_Validar_Gold
```

### Actividades

| Actividad | Notebook asociado | Responsabilidad |
|---|---|---|
| ACT_01_Bronze_to_Silver | 01_bronze_to_silver | Transformación de Bronze a Silver |
| ACT_02_Silver_to_Gold | 02_silver_to_gold | Construcción de las tablas FactSales y DimCustomer |
| ACT_03_Validar_Gold | 03_validar_gold | Validaciones de calidad de la capa Gold |

## 3. Dependencias y control de ejecución

Las actividades están conectadas mediante dependencias de tipo `Succeeded`.

Esto garantiza que:

- Silver solo se ejecuta si Bronze ha finalizado correctamente.
- La validación de Gold solo se ejecuta si la transformación Silver → Gold ha finalizado correctamente.
- Si una actividad falla, las actividades posteriores dependientes no se ejecutan.

Este comportamiento evita continuar el procesamiento cuando una etapa anterior no ha finalizado satisfactoriamente.

## 4. Configuración de ejecución

Configuración aplicada a las tres actividades:

| Parámetro | Valor |
|---|---|
| Tipo de actividad | TridentNotebook |
| Timeout | 12 horas |
| Reintentos | 0 |
| Intervalo de reintento | 30 segundos |
| Session tag | DP600_MEDALLION |
| Secure input | false |
| Secure output | false |

Las tres actividades utilizan la misma etiqueta de sesión `DP600_MEDALLION`.

## 5. Validación de calidad

La última actividad ejecuta el notebook `03_validar_gold`, que comprueba la integridad y calidad de las tablas de la capa Gold.

Se realizan nueve comprobaciones:

1. FactSales no está vacía.
2. DimCustomer no está vacía.
3. FactSales no contiene claves nulas.
4. DimCustomer no contiene CustomerId nulos.
5. SaleId no presenta duplicados.
6. CustomerId no presenta duplicados en DimCustomer.
7. No existen ventas sin cliente asociado.
8. Quantity es mayor que cero y UnitPrice es mayor o igual que cero.
9. TotalAmount coincide con el resultado de Quantity multiplicado por UnitPrice.

Si alguna validación falla, el notebook genera una excepción y la actividad se considera fallida.

## 6. Ejecución y programación

La canalización puede ejecutarse manualmente mediante la opción Run de Microsoft Fabric.

La programación recurrente (Schedule) es opcional y se configura por separado de una ejecución manual.

## 7. Evidencia de ejecución

El 18 de noviembre de 2026 se realizó una ejecución satisfactoria de `PL_DP600_Orchestration`.

| Actividad | Hora de inicio | Estado |
|---|---|---|
| ACT_01_Bronze_to_Silver | 12:50:24 | Correcto |
| ACT_02_Silver_to_Gold | 12:51:56 | Correcto |
| ACT_03_Validar_Gold | 12:52:48 | Correcto |

**Resultado:** las tres actividades finalizaron correctamente y la canalización alcanzó el estado Correcto.

## 8. Relación con el orquestador mediante DAG

El repositorio también incluye el notebook `00_orquestador.ipynb`, que implementa una alternativa de orquestación mediante DAG utilizando `notebookutils.notebook.runMultiple()`.

Este notebook se ha ejecutado de forma independiente, fuera de la canalización `PL_DP600_Orchestration`.

Por tanto, se mantienen dos mecanismos diferenciados:

- Orquestación nativa mediante Pipeline de Microsoft Fabric.
- Orquestación mediante DAG desde un notebook PySpark.

La canalización `PL_DP600_Orchestration` es el mecanismo principal de ejecución del flujo de trabajo documentado en este proyecto.

## 9. Consideraciones

Los identificadores de workspace y de notebooks son específicos del entorno de Microsoft Fabric.

No se incluyen identificadores reales en esta documentación para facilitar la reutilización del proyecto y evitar publicar referencias específicas del entorno.

La configuración de actividades, dependencias y parámetros debe adaptarse al workspace donde se despliegue la solución.

Nota: El archivo PL_DP600_Orchestration.template.json es una plantilla de referencia, no un JSON listo para importar directamente en Fabric. Los identificadores son marcadores y pueden requerirse metadatos adicionales según el método de despliegue.