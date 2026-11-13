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
