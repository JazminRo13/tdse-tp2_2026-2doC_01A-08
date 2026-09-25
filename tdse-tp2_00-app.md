# Resolución de los Pasos 12 y 13

# 1. Análisis y Explicación del Código Fuente

A continuación se detalla el funcionamiento de los archivos que componen la lógica de control principal y medición de tiempos (profiling) en la aplicación *Bare Metal - Event-Triggered Systems (ETS)*.

### A. `app.c` y `app_it.c` (Control de Aplicación e Interrupciones)
* **`app_init()` en `app.c`**: Es la función de inicialización de la lógica del usuario. Primero imprime mensajes de depuración (usando la macro `LOGGER_INFO`) para indicar que la aplicación arrancó. Luego inicializa a cero el contador principal de la aplicación (`g_app_cnt`) y activa el registro DWT del procesador Cortex-M mediante `cycle_counter_init()` para habilitar un contador de ciclos de reloj de alta precisión. Posteriormente, recorre la lista de tareas `task_cfg_list` (sensor, sistema, actuador) llamando a sus rutinas de inicialización (`task_init`). Además, inicializa las estructuras de datos `task_dta_list` (que guardan la telemetría de ejecución de cada tarea) a sus valores por defecto (ej. NOE=0, LET=0, BCET=1000, WCET=0). Finalmente, invoca `app_it_init()` para poner a 0 el contador de *ticks* `g_app_tick_cnt` de manera atómica, deshabilitando previamente las interrupciones mediante el código ensamblador `__asm("CPSID i")` y volviéndolas a habilitar con `__asm("CPSIE i")`.
* **`HAL_SYSTICK_Callback()` en `app_it.c`**: Esta función es llamada automáticamente cada 1 ms (o según el periodo del SysTick configurado) desde el vector de interrupción en hardware. Su única labor es incrementar `g_app_tick_cnt++`.
* **`app_update()` en `app.c`**: Es la máquina de estados principal que corre dentro del `while(1)`. Revisa de manera atómica (protegido contra interrupciones) si `g_app_tick_cnt` es mayor a 0 (lo que indica que ha pasado al menos un milisegundo o *tick* de hardware). Si es así, decremente el contador y establece una bandera `b_time_update_required`. A continuación, se entra a un ciclo `while (b_time_update_required)` que ejecuta secuencialmente todas las tareas (`task_sensor`, `task_system`, `task_actuator`) reseteando el contador de ciclos del procesador antes de cada tarea y midiendo el tiempo que tardó cada una después.
* **`HAL_GPIO_EXTI_Callback()` en `app_it.c`**: Captura los eventos físicos (ej. presionar un botón). Por el momento, el evento del pin `BTN_A_PIN` está vacío y pendiente de ser implementado.

### B. `dwt.h` (Data Watchpoint and Trace)
El archivo expone una interfaz de bajo nivel para controlar la unidad de *Data Watchpoint and Trace* (DWT), exclusiva de la arquitectura ARM Cortex-M, con el fin de perfilar la ejecución en tiempo real. Al llamar a `cycle_counter_init()`, se habilita globalmente el *Trace* (activando el bit `TRCENA` en el registro `DEMCR` del núcleo) y se enciende el contador de 32 bits de hardware `CYCCNT` activando `DWT_CTRL_CYCCNTENA_Msk`. La macro `cycle_counter_get_time_us()` convierte de manera veloz los ciclos de reloj transcurridos en microsegundos dividiendo el registro de ciclos entre la frecuencia en MHz del procesador (obtenida de `SystemCoreClock`). Las funciones se declaran estáticas e integradas (`always_inline`) para no introducir latencia (overhead) adicional al medir el código.

### C. `logger.c` y `logger.h` (Registro de Información / Debugging)
Este módulo implementa el envío de texto plano hacia el exterior (comúnmente la terminal UART del puerto de depuración). `logger.h` define macros sofisticadas (`LOGGER_LOG` y `LOGGER_INFO`) que bloquean las interrupciones del microcontrolador (`CPSID i`) justo antes de formatear los datos de entrada usando `snprintf()` (lo que asegura que ningún evento concurrente corrompa el búfer). Luego llama a `logger_log_print_()` y vuelve a habilitar las interrupciones (`CPSIE i`). Dependiendo de si la macro `LOGGER_CONFIG_USE_SEMIHOSTING` está activa o no, la función en `logger.c` despachará el mensaje mediante la librería estándar (`printf()` sobre *Semihosting*) u otra vía.

### D. `systick.c` (Generación de Retardos Bloqueantes)
Implementa la función `systick_delay_us(uint32_t delay_us)` para pausar la ejecución del microcontrolador de forma precisa una determinada cantidad de microsegundos, sin usar interrupciones. Para ello, accede a los registros nativos del temporizador *SysTick* (específicamente a su valor actual en cuenta regresiva `SysTick->VAL` y su límite superior `SysTick->LOAD`). Mediante un ciclo continuo, mide cuántos ciclos (y por ende microsegundos) han pasado calculando la resta entre el valor de inicio y el actual, manejando correctamente los "desbordamientos" en los que el contador baja a cero y vuelve a cargar su valor máximo.

---

# 2. Evolución de Variables de Control y Tiempos de Ejecución

A continuación se traza la evolución de las variables durante la ejecución del programa, asumiendo un ciclo regular del bucle principal:

* **`g_app_tick_cnt` (Variable atómica tipo entero sin signo):** 
  * En `app_init()`: Inicia en 0.
  * Mientras ocurre la ejecución normal: Al ocurrir una interrupción de hardware cada milisegundo (SysTick), la ISR la incrementa en +1. Al inicio del ciclo principal en `app_update()`, el programa desactiva temporalmente las interrupciones, verifica que sea mayor a 0, y en ese caso le resta -1. Al final del bucle de `app_update()` vuelve a repetirse este control. Permite encolar trabajos pendientes si las tareas tardaron más de 1 ms.
* **`g_app_runtime_us` (Microsegundos - us):** 
  * Al inicio de cada ciclo efectivo dentro del bucle de `app_update()`, se reinicia a 0.
  * Por cada tarea ejecutada (sensor, sistema, actuador) acumula el valor medido del tiempo de la última ejecución (LET) de cada una (`g_app_runtime_us += task_dta_list[index].LET`). Indica cuánto tiempo global le tomó a todo el arreglo de tareas ejecutarse durante un *tick*.
* **`index`:** 
  * Comienza en `0` y va iterando (0, 1, 2) según `TASK_QTY`. Sirve de puntero secuencial al arreglo bidimensional de funciones (tareas) y telemetría de las mismas.
* **`task_dta_list[index].NOE` (Número de ejecución):**
  * En `app_init()`: Comienza en `0` (vía macro `TASK_X_NOE_INI`).
  * En `app_update()`: Por cada vez que una tarea en particular se termina de ejecutar, se suma +1.
* **`task_dta_list[index].LET` (Último tiempo de ejecución en microsegundos - us):**
  * En `app_init()`: Inicia en `0` (`TASK_X_LET_INI`).
  * En `app_update()`: Inmediatamente después de ejecutar cada tarea, adopta el valor de `cycle_counter_get_time_us()`, marcando el tiempo instantáneo exacto que costó ejecutar la tarea en esa iteración específica.
* **`task_dta_list[index].BCET` (Mejor tiempo de ejecución en microsegundos - us):**
  * En `app_init()`: Se setea en un número artificialmente alto por diseño para la primera vez: `1000` (`TASK_X_BCET_INI`).
  * En `app_update()`: Si el valor actual de `LET` es menor que el `BCET` guardado, se sobreescribe con este nuevo mínimo histórico, registrando la iteración en la que el código tardó la menor cantidad de tiempo de toda la historia de ejecución del microcontrolador.
* **`task_dta_list[index].WCET` (Peor tiempo de ejecución en microsegundos - us):**
  * En `app_init()`: Arranca lógicamente en `0` (`TASK_X_WCET_INI`).
  * En `app_update()`: Si en cualquier iteración el valor medido `LET` de una tarea excede el de `WCET`, se sobreescribe el `WCET` con este nuevo máximo, registrando el peor tiempo de ejecución o cuello de botella.

---

# 3. Impacto de usar `LOGGER_INFO()` en los Tiempos (WCET)

Insertar instrucciones como `LOGGER_INFO()` **dentro de las rutinas de las tareas que componen `task_cfg_list`** tiene un impacto masivo y drástico en las mediciones de rendimiento:

1. **Aumento brutal de Latencia (WCET):** Al llamar a `LOGGER_INFO`, el microcontrolador bloquea por completo las interrupciones del hardware (`CPSID i`), se detiene a formatear una cadena de caracteres usando la pesada función de librería `snprintf()` y finalmente aguarda a que todos esos caracteres viajen y se impriman (frecuentemente a velocidades seriales lentas o por depurador *Semihosting*), todo de forma *bloqueante*. El tiempo `task_dta_list[index].LET` medido a la salida de dicha tarea se disparará, registrándose automáticamente un nuevo y masivo `task_dta_list[index].WCET`.
2. **Corrupción del Tiempo Total (`g_app_runtime_us`):** Todo el lapso de inactividad que causó la impresión bloqueante de la cadena se sumará directamente a la variable global `g_app_runtime_us`.
3. **Colapso del Modelo Basado en Eventos:** Si `g_app_runtime_us` excede 1 milisegundo (el *tick* del sistema SysTick), la variable `g_app_tick_cnt` comenzará a acumular iteraciones atrasadas y las tareas de tiempo real no lograrán cumplir con su meta operativa de ejecutarse, corrompiendo la arquitectura de eventos en tiempo real. 

Por estas razones, el *logging* y rutinas de impresión **jamás deben ejecutarse** dentro de bloques medidos por *profiling* ni dentro de tareas deterministas de lazos cerrados, a menos que se desee explícitamente evaluar el costo (generalmente prohibitivo) de imprimir en la consola serial.

## Valores de la práctica.
Después de varias ejecuciones (4 segundos) valor de la variable:
 - task_dta_list[0].NOE = 7678 
 - task_dta_list[0].LET = 4 us
 - task_dta_list[0].BCET = 4 us
 - task_dta_list[0].WCET = 4 us
 - task_dta_list[1].NOE = 7678
 - task_dta_list[1].LET = 3 us
 - task_dta_list[1].BCET = 3 us
 - task_dta_list[1].WCET = 3 us
 - task_dta_list[2].NOE = 7678
 - task_dta_list[2].LET = 2 us
 - task_dta_list[2].BCET = 2 us
 - task_dta_list[2].WCET = 2 us