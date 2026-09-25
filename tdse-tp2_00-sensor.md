# Resolución del Paso 15

### 1. Funcionalidad de los Archivos
* **`task_sensor_attribute.h` y `task_system_attribute.h`**: Definen las estructuras de datos, estados (ej. `ST_BTN_IDLE`, `ST_BTN_ACTIVE`) y eventos (ej. `EV_BTN_UP`, `EV_BTN_DOWN`) para las tareas del sensor y del sistema respectivamente.
* **`task_sensor.c`**: Implementa la lógica de la tarea encargada de leer el estado físico de un botón (sensor) y actualizar su máquina de estados (FSM). 
* **`task_system_interface.c`**: Implementa una estructura de datos tipo cola circular (FIFO) que actúa como puente de comunicación para enviar eventos desde el sensor hacia el sistema.

### 2. Evolución de Variables en `task_sensor.c`
Al arrancar la aplicación (`task_sensor_init()`) y durante el bucle principal (`task_sensor_update()`):

* **`index`**: Comienza en `0`. Dado que `SENSOR_DTA_QTY` es 1 (solo hay configurado un botón), el ciclo `for` itera una única vez (`index = 0`) en cada llamada.
* **`task_sensor_dta_list[index].state`**: 
  * *Inicio:* Se inicializa forzosamente en `ST_BTN_IDLE`.
  * *Evolución:* Cambia a `ST_BTN_ACTIVE` cuando se detecta la pulsación del botón y vuelve a `ST_BTN_IDLE` cuando se suelta.
* **`task_sensor_dta_list[index].event`**:
  * *Inicio:* Se inicializa en `EV_BTN_UP`.
  * *Evolución:* En cada iteración del bucle, lee el pin del hardware. Si está presionado adopta `EV_BTN_DOWN`, y si está liberado adopta `EV_BTN_UP`.
* **`task_sensor_dta_list[index].tick` (Unidad: milisegundos / *ticks* del sistema)**:
  * En esta implementación específica del estado, no se está utilizando activamente para generar retardos antirrebote. Por defecto arranca en `0` (al ser variable global) y solo tomaría el valor `DEL_BTN_MIN` (`0ul`) si la máquina de estados cayera accidentalmente en el caso `default`.

### 3. Comportamiento de `task_sensor_statechart(uint32_t index)`
Es la máquina de estados finitos (FSM) que procesa la lógica del botón:
1. **Lectura de Hardware**: Consulta el estado del pin a través de la HAL y actualiza el evento actual (`EV_BTN_DOWN` o `EV_BTN_UP`).
2. **Transiciones**:
   * Si el estado es `ST_BTN_IDLE` y el evento es `EV_BTN_DOWN` (el usuario presionó el botón), envía la señal configurada (`signal_down`) a la cola del sistema mediante `put_event_task_system()` y cambia el estado a `ST_BTN_ACTIVE`.
   * Si el estado es `ST_BTN_ACTIVE` y el evento es `EV_BTN_UP` (el usuario soltó el botón), envía la señal (`signal_up`) a la cola y regresa al estado `ST_BTN_IDLE`.

### 4. Evolución de la Cola de Eventos (`event_task_system_queue`)
Asumiendo que se llamó previamente a `init_event_task_system()`, la cola evoluciona según la actividad del sensor:

* **Estado Inicial**: 
  * `head` (cabeza), `tail` (cola) y `count` (cantidad) inician en `0`. 
  * Todas las posiciones `queue[i]` (de 0 a 15) se rellenan con el valor `EMPTY` (`255`).
* **En sucesivas ejecuciones (al presionar/soltar el botón)**:
  * Cuando el *statechart* del sensor detecta un cambio válido, invoca a `put_event_task_system()`.
  * **`count`**: Se incrementa en +1 por cada evento añadido.
  * **`queue[head]`**: Se sobreescribe la posición actual de `head` con el evento enviado (ej. `EV_SYS_ACTIVE` o `EV_SYS_IDLE`).
  * **`head`**: Se incrementa en +1 apuntando al próximo espacio libre. Si llega al límite de la cola (`QUEUE_LENGTH`), vuelve a `0` (comportamiento circular).
  * **`tail`**: Permanece estática hasta que otra tarea consuma los eventos utilizando la función `get_event_task_system()`.