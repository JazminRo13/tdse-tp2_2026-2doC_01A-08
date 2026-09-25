# Resolución del Paso 21


# Análisis del Subsistema "Task Actuator"

### 1. Evolución de Variables en `task_actuator_dta_list`
Durante la inicialización (`task_actuator_init()`) y la ejecución repetitiva (`task_actuator_update()`), las variables de la tarea evolucionan de la siguiente manera:

*   **`index`**: En la inicialización y actualización, itera desde 0 hasta `ACTUATOR_DTA_QTY - 1`. Como en este caso hay un solo actuador configurado, el índice toma únicamente el valor 0.
*   **`tick`**: Su unidad de medida implícita es en milisegundos (ms). Si bien no se inicializa con un valor explícito en la etapa de `init()`, la máquina de estados le asigna por defecto el valor `DEL_LED_MIN` (0) si cae por error en la condición `default`.
*   **`state`**: Comienza obligatoriamente en el estado de reposo `ST_LED_IDLE`. Evoluciona a `ST_LED_ACTIVE` cuando recibe y procesa la orden de encendido, y retorna a `ST_LED_IDLE` al procesar la orden de apagado.
*   **`event`**: Se inicializa en `EV_LED_IDLE`. A diferencia del sistema anterior, esta variable es modificada asincrónicamente por la interfaz cuando se le ordena un nuevo evento (ej. `EV_LED_ACTIVE`).
*   **`flag`**: Inicia en `false`. Pasa a `true` cuando la interfaz externa inyecta un evento nuevo. Una vez que el *statechart* del actuador detecta esta bandera en `true`, consume el evento y la devuelve inmediatamente a `false`.

### 2. Comportamiento de `task_actuator_statechart(uint32_t index)`
Esta función implementa la máquina de estados finitos (FSM) que controla físicamente el actuador (LED):

1.  **Carga de Contexto**: Accede a los punteros de configuración de hardware (puerto, pin, estados lógicos) y de datos dinámicos correspondientes al `index` provisto.
2.  **Estado IDLE (`ST_LED_IDLE`)**: Si el actuador está apagado, evalúa si la bandera está levantada (`flag == true`) y si el evento solicitado es `EV_LED_ACTIVE`. De cumplirse, baja la bandera, enciende el LED utilizando `HAL_GPIO_WritePin` con la configuración de encendido (`led_on`), y transita al estado `ST_LED_ACTIVE`.
3.  **Estado ACTIVE (`ST_LED_ACTIVE`)**: Si el LED está encendido, espera a que la bandera sea `true` y el evento sea `EV_LED_IDLE`. Al ocurrir, baja la bandera, apaga el LED escribiendo el valor `led_off` en el hardware, y regresa al estado `ST_LED_IDLE`.
4.  **Estado por Defecto**: Existe un caso `default` de seguridad que fuerza a la máquina al estado `ST_LED_IDLE`, borra la bandera y resetea las variables a sus condiciones iniciales.

### 3. Evolución de las Variables de Interfaz
La comunicación hacia el actuador se materializa al llamar a la función `put_event_task_actuator(event, identifier)`. Cuando el sistema invoca esto, las variables evolucionan así:

*   **`identifier`**: Representa el ID del actuador destino (como `ID_LED_A`). Se utiliza como índice directo para acceder al elemento correcto dentro del arreglo global `task_actuator_dta_list`.
*   **`event`**: Es actualizada de forma directa con el valor del parámetro `event` recibido (por ejemplo, encender o apagar), sobreescribiendo cualquier evento previo.
*   **`flag`**: Es forzada incondicionalmente al valor booleano `true`. Esto funciona como un semáforo que le avisa al `task_actuator_statechart` en su próxima ejecución periódica que hay una acción nueva lista para ser efectuada.
