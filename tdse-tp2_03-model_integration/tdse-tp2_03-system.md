# Depuración del modelo System y Rendimiento de Tareas (Paso 04)

Se realizó la depuración del proyecto mediante **STM32CubeIDE** con el objetivo de verificar el correcto funcionamiento del modelo System y medir el rendimiento temporal de las tareas mediante la estructura `task_dta_list[index]`.

---

## Modificaciones en la inicialización

Dado que en el diagrama `task_system.jpg` no se menciona un evento inicial como `EV_SYS_IDLE`, decidimos quitar la inicialización de dicha variable en la función `void task_system_init(void *parameters)`:

```c
// event = EV_SYS_IDLE;
// p_task_system_dta->event = event;
```

Dado que inicialmente `p_task_system_dta->flag = false;`, no debería ser un inconveniente que tenga un valor indefinido, porque siempre consultamos si el flag es verdadero antes de leer el evento, como en la siguiente condición:

```c
if ((true == p_task_system_dta->flag) && (EV_SYS_CAMERA == p_task_system_dta->event))
```

---

Luego de varias ejecuciones de `app_update()`, se leyeron y almacenaron los valores correspondientes a cada tarea del sistema:

* **`task_dta_list[0]` (Tarea Sensor):**
  * `NOE` = 7592 *(Number of Execution: número total de ejecuciones de la tarea)*
  * `LET` = 12 us *(Last Execution Time: último tiempo de ejecución medido en microsegundos)*
  * `BCET` = 12 us *(Best-Case Execution Time: mejor tiempo de ejecución histórico registrado)*
  * `WCET` = 12 us *(Worst-Case Execution Time: peor tiempo de ejecución histórico registrado)*

* **`task_dta_list[1]` (Tarea System):**
  * `NOE` = 7592 *(Number of Execution: número total de ejecuciones de la tarea)*
  * `LET` = 3 us *(Last Execution Time: último tiempo de ejecución medido en microsegundos)*
  * `BCET` = 3 us *(Best-Case Execution Time: mejor tiempo de ejecución histórico registrado)*
  * `WCET` = 4 us *(Worst-Case Execution Time: peor tiempo de ejecución histórico registrado)*

* **`task_dta_list[2]` (Tarea Actuator):**
  * `NOE` = 7592 *(Number of Execution: número total de ejecuciones de la tarea)*
  * `LET` = 2 us *(Last Execution Time: último tiempo de ejecución medido en microsegundos)*
  * `BCET` = 2 us *(Best-Case Execution Time: mejor tiempo de ejecución histórico registrado)*
  * `WCET` = 2 us *(Worst-Case Execution Time: peor tiempo de ejecución histórico registrado)*
