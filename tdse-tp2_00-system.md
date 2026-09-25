# Resolución del Paso 18

# Análisis del Subsistema "Task System" y Actuador

### 1. Evolución de Variables de `task_system_dta_list`
Durante la inicialización (`task_system_init()`) y la ejecución repetitiva (`task_system_update()`), las variables evolucionan de la siguiente manera:

*   **`index`**: En `task_system_init()`, se utiliza en un ciclo para recorrer las configuraciones. Como `SYSTEM_DTA_QTY` equivale a la cantidad de modos (`MODE_QTY` = 1), el índice solo toma el valor `0` correspondiente al modo `NORMAL`.
*   **`tick`**: Su unidad de medida implícita es en milisegundos (ms). En este código base no se inicializa explícitamente al principio, pero la máquina de estados le asigna `DEL_SYS_MIN` (0) si cae en la condición por defecto. Sirve para llevar la cuenta de retardos no bloqueantes.
*   **`state`**: Comienza obligatoriamente en `ST_SYS_IDLE`. Evoluciona a `ST_SYS_ACTIVE` cuando recibe la orden de encendido y retorna a `ST_SYS_IDLE` cuando recibe la orden de apagado.
*   **`event`**: Inicia en `EV_SYS_IDLE`. Durante el bucle `update`, se sobreescribe constantemente con los eventos que se extraen de la cola.
*   **`flag`**: Inicia en `false`. Pasa a `true` al detectar y leer un evento válido desde la cola. Una vez que el *statechart* procesa dicho evento (transición de estado), se vuelve a poner en `false`.

### 2. Comportamiento del Statechart del Sistema
*Nota aclaratoria: En el código proporcionado, la función se denomina `task_system_normal_statechart(void)` y opera sobre el índice estático `NORMAL`, en lugar de recibir `index` por parámetro.*

El comportamiento de la máquina de estados es el siguiente:
1.  **Lectura de eventos**: Consulta `any_event_task_system()`. Si hay eventos en la cola, levanta su bandera (`flag = true`) y lee el evento con `get_event_task_system()`.
2.  **Estado IDLE**: Si la máquina está en `ST_SYS_IDLE` y procesa un evento `EV_SYS_ACTIVE`, despacha una señal de encendido al actuador (`EV_LED_ACTIVE`) y cambia su estado a `ST_SYS_ACTIVE`.
3.  **Estado ACTIVE**: Si la máquina está en `ST_SYS_ACTIVE` y procesa un evento `EV_SYS_IDLE`, manda a apagar el actuador (`EV_LED_IDLE`) y retorna al estado `ST_SYS_IDLE`.

### 3. Evolución de la Cola de Eventos (Queue)
Las variables de `event_task_system_queue` gestionan el búfer circular (FIFO):

*   **`i`**: Es solo un iterador temporal en `init_event_task_system()`. Evoluciona de 0 a 15 para limpiar la memoria.
*   **`queue[i]`**: Inician todas sus posiciones en `EMPTY` (255). Durante la ejecución normal, almacenan de forma transitoria los valores `EV_SYS_ACTIVE` o `EV_SYS_IDLE` enviados por el sensor.
*   **`head`**: Inicia en 0. Se incrementa en +1 cada vez que entra un nuevo evento (escritura).
*   **`tail`**: Inicia en 0. Se incrementa en +1 cada vez que el sistema extrae un evento (lectura).
*   **`count`**: Inicia en 0. Sube al escribir y baja al leer. Indica la cantidad de tareas pendientes. Si `head` o `tail` llegan a 16 (`QUEUE_LENGTH`), vuelven a 0.

### 4. Evolución de las Variables del Actuador
Cuando el sistema invoca `put_event_task_actuator()` desde su statechart, estas variables de destino se modifican:

*   **`identifier`**: Toma el valor fijo `ID_LED_A` (0), apuntando a la configuración del primer LED en el arreglo.
*   **`event`**: Toma el valor ordenado por el sistema, ya sea `EV_LED_ACTIVE` (para encender) o `EV_LED_IDLE` (para apagar).
*   **`flag`**: Es forzada directamente a `true`. Esto deja señalizado para la posterior ejecución de la tarea del actuador que tiene un evento nuevo y legítimo listo para ser ejecutado.
