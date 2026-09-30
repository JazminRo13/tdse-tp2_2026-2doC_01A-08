## Depuración del modelo Actuator y Rendimiento de Tareas (Paso 04)

Se realizó la depuración del proyecto mediante STM32CubeIDE con el objetivo de verificar el correcto funcionamiento del modelo `Actuator` y medir el rendimiento temporal de las tareas mediante la estructura `task_dta_list[index]` cuando tenemos 2 leds conectados.

Para esta actividad, se configuraron y utilizaron los siguientes pines de la placa NUCLEO como entradas para los botones adicionales:
* **`LED_B`**: Pin `PB10`
* **`LED_C`**: Pin `PA8`

Luego de varias ejecuciones de `app_update()`, se leyeron y almacenaron los valores correspondientes a cada tarea del sistema:

* **`task_dta_list[0]` (Tarea Sensor):**
  * `NOE` = 5982 *(Number of Execution: número total de ejecuciones de la tarea)*
  * `LET` = 13 us *(Last Execution Time: último tiempo de ejecución medido en microsegundos)*
  * `BCET` = 13 us *(Best-Case Execution Time: mejor tiempo de ejecución histórico registrado)*
  * `WCET` = 13 us *(Worst-Case Execution Time: peor tiempo de ejecución histórico registrado)*

* **`task_dta_list[1]` (Tarea System):**
  * `NOE` = 5982 *(Number of Execution: número total de ejecuciones de la tarea)*
  * `LET` = 3 us *(Last Execution Time: último tiempo de ejecución medido en microsegundos)*
  * `BCET` = 3 us *(Best-Case Execution Time: mejor tiempo de ejecución histórico registrado)*
  * `WCET` = 4 us *(Worst-Case Execution Time: peor tiempo de ejecución histórico registrado)*

* **`task_dta_list[2]` (Tarea Actuator):**
  * `NOE` = 5982 *(Number of Execution: número total de ejecuciones de la tarea)*
  * `LET` = 4 us *(Last Execution Time: último tiempo de ejecución medido en microsegundos)*
  * `BCET` = 4 us *(Best-Case Execution Time: mejor tiempo de ejecución histórico registrado)*
  * `WCET` = 4 us *(Worst-Case Execution Time: peor tiempo de ejecución histórico registrado)*