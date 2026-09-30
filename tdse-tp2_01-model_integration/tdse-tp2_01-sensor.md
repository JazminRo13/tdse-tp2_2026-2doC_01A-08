## Implementación del modelo Sensor

Se modificó el código fuente del modelo Sensor para implementar el
diagrama de estados correspondiente al botón `BTN_A`, asociado al
pulsador `B1 USER (Blue)` de la placa.

El modelo Sensor utiliza los eventos `EV_BTN_UP` y `EV_BTN_DOWN` para
representar, respectivamente, el botón liberado y presionado.

La máquina de estados está compuesta por cuatro estados:

- `ST_BTN_UP`: el botón se encuentra liberado.
- `ST_BTN_FALLING`: estado de transición utilizado para verificar la
  pulsación del botón.
- `ST_BTN_DOWN`: el botón se encuentra presionado.
- `ST_BTN_RISING`: estado de transición utilizado para verificar la
  liberación del botón.

Se modificó el archivo `task_sensor_attribute.h` para incorporar los
cuatro estados del modelo Sensor.

Además, se modificó `task_sensor.c` para inicializar el modelo en
`ST_BTN_UP` e implementar las transiciones del diagrama de estados.

Los estados `ST_BTN_FALLING` y `ST_BTN_RISING` utilizan la variable
`tick` para temporizar la validación de los cambios de estado del
pulsador. Una vez confirmada una pulsación o liberación, el modelo
Sensor envía al modelo System los eventos correspondientes mediante
`put_event_task_system()`.

## Depuración del modelo Sensor y Rendimiento de Tareas (Paso 04)

Se realizó la depuración del proyecto mediante STM32CubeIDE con el objetivo de verificar el correcto funcionamiento del modelo `Sensor` y medir el rendimiento temporal de las tareas mediante la estructura de telemetría `task_dta_list[index]`.

Luego de varias ejecuciones de `app_update()`, se leyeron y almacenaron los valores correspondientes a cada tarea del sistema:

* **`task_dta_list[0]` (Tarea Sensor):**
  * `NOE` = `6784` (número de ejecuciones)
  * `LET` = `4 us` (último tiempo de ejecución en microsegundos)
  * `BCET` = `4 us` (mejor tiempo de ejecución en microsegundos)
  * `WCET` = `4 us` (peor tiempo de ejecución en microsegundos)

* **`task_dta_list[1]` (Tarea System):**
  * `NOE` = `6784` (número de ejecuciones)
  * `LET` = `3 us` (último tiempo de ejecución en microsegundos)
  * `BCET` = `3 us` (mejor tiempo de ejecución en microsegundos)
  * `WCET` = `3 us` (peor tiempo de ejecución en microsegundos)

* **`task_dta_list[2]` (Tarea Actuator):**
  * `NOE` = `6784` (número de ejecuciones)
  * `LET` = `2 us` (último tiempo de ejecución en microsegundos)
  * `BCET` = `2 us` (mejor tiempo de ejecución en microsegundos)
  * `WCET` = `2 us` (peor tiempo de ejecución en microsegundos)

De esta manera, se comprueba la correcta sincronización, el número de ejecuciones acumuladas y la eficiencia temporal no bloqueante de cada una de las capas de la aplicación.
