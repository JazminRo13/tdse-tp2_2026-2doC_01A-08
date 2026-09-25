## Depuración del modelo Sensor y Rendimiento de Tareas (Paso 04)

Se realizó la depuración del proyecto mediante STM32CubeIDE con el objetivo de verificar el correcto funcionamiento del modelo `Sensor` y medir el rendimiento temporal de las tareas mediante la estructura de telemetría `task_dta_list[index]` cuando tenemos 3 botones conectados.

Luego de varias ejecuciones de `app_update()`, se leyeron y almacenaron los valores correspondientes a cada tarea del sistema:

* **`task_dta_list[0]` (Tarea Sensor):**
  * `NOE` = `14374` (número de ejecuciones)
  * `LET` = `12 us` (último tiempo de ejecución en microsegundos)
  * `BCET` = `12 us` (mejor tiempo de ejecución en microsegundos)
  * `WCET` = `12 us` (peor tiempo de ejecución en microsegundos)

* **`task_dta_list[1]` (Tarea System):**
  * `NOE` = `14374` (número de ejecuciones)
  * `LET` = `3 us` (último tiempo de ejecución en microsegundos)
  * `BCET` = `3 us` (mejor tiempo de ejecución en microsegundos)
  * `WCET` = `4 us` (peor tiempo de ejecución en microsegundos)

* **`task_dta_list[2]` (Tarea Actuator):**
  * `NOE` = `14374` (número de ejecuciones)
  * `LET` = `2 us` (último tiempo de ejecución en microsegundos)
  * `BCET` = `2 us` (mejor tiempo de ejecución en microsegundos)
  * `WCET` = `2 us` (peor tiempo de ejecución en microsegundos)
