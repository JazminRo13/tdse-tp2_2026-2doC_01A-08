# 1. Análisis y Explicación del Código Fuente

### A. `startup_stm32f103rbtx.s` (Código de Arranque en Ensamblador)
Este archivo es el script de arranque (startup) del microcontrolador[cite: 3].
* **Tabla de Vectores de Interrupción (`g_pfnVectors`)**: Define la estructura de memoria de las interrupciones[cite: 3]. Carga la dirección inicial del puntero de pila (`_estack`), la rutina de Reset (`Reset_Handler`) y los manejadores de excepciones e interrupciones de periféricos[cite: 3].
* **Rutina `Reset_Handler`**:
  1. **Llamada a `SystemInit`**: Invoca la configuración inicial del sistema de reloj[cite: 3].
  2. **Inicialización de `.data`**: Copia los valores iniciales de las variables desde la memoria Flash hacia la memoria RAM[cite: 3].
  3. **Limpieza de `.bss`**: Pone a cero la sección de RAM reservada para variables globales no inicializadas[cite: 3].
  4. **Llamada a `__libc_init_array`**: Ejecuta las inicializaciones necesarias de la librería[cite: 3].
  5. **Salto a `main`**: Cede el control a la función principal `main()` de la aplicación[cite: 3].

### B. `stm32f1xx_it.c` (Manejadores de Interrupciones / ISR)
Contiene las Rutinas de Servicio de Interrupción para excepciones y periféricos[cite: 2].
* **Excepciones del Sistema**: Manejadores de fallos como `NMI_Handler`, `HardFault_Handler`, `MemManage_Handler`, `BusFault_Handler` y `UsageFault_Handler` están configurados como bucles infinitos (`while (1)`) para atrapar errores fatales[cite: 2].
* **`SysTick_Handler`**: Es la interrupción periódica del temporizador del sistema[cite: 2]. Llama a `HAL_IncTick()` para incrementar el contador de tiempo y a `HAL_SYSTICK_IRQHandler()`[cite: 2].
* **`EXTI15_10_IRQHandler`**: Maneja las interrupciones externas, delegando el evento del pin `B1_Pin` a la función de la HAL `HAL_GPIO_EXTI_IRQHandler(B1_Pin)`[cite: 2].

### C. `main.c` (Aplicación Principal)
Es el archivo donde se configura el hardware y se ejecuta la lógica del programa[cite: 1].
* **Ejecución de `main()`**:
  1. **`HAL_Init()`**: Inicializa la capa de abstracción de hardware, resetea periféricos, inicializa la interfaz Flash y el Systick[cite: 1].
  2. **`SystemClock_Config()`**: Configura el reloj del sistema usando el oscilador HSI[cite: 1]. Ajusta el PLL dividiendo la fuente HSI por 2 y multiplicándola por 16[cite: 1]. También establece los divisores para los buses AHB y APB[cite: 1].
  3. **Inicialización de hardware y aplicación**: Ejecuta la configuración de GPIO (`MX_GPIO_Init()`), la UART (`MX_USART2_UART_Init()`) y la inicialización propia del usuario (`app_init()`)[cite: 1].
  4. **Bucle principal**: Entra en un `while (1)` donde ejecuta continuamente `app_update()`[cite: 1].

---

# 2. Evolución de las Variables `SystemCoreClock` y `SysTick`

### A. Etapa 1: Desde `Reset_Handler` hasta antes de `main()`
* **`SystemCoreClock`**: Arranca asumiendo la frecuencia inicial dictada por `SystemInit` (típicamente utilizando el HSI de 8 MHz por defecto)[cite: 3].
* **SysTick**: El hardware SysTick se encuentra deshabilitado y las variables de tiempo (como `uwTick` en `.bss`) son puestas a 0[cite: 3].

### B. Etapa 2: Ejecución de `HAL_Init()` dentro de `main()`
* **`SystemCoreClock`**: Permanece en su valor inicial base (8 MHz).
* **SysTick**: La función `HAL_Init()` inicializa y arranca el temporizador SysTick[cite: 1]. A partir de este momento, genera una interrupción (típicamente cada 1 ms) que incrementa la variable global mediante `SysTick_Handler` y `HAL_IncTick()`[cite: 1, 2].

### C. Etapa 3: Ejecución de `SystemClock_Config()`
* **`SystemCoreClock`**: Tras la configuración del PLL (HSI / 2 * 16), la frecuencia principal del sistema sube a 64 MHz[cite: 1]. La variable global de la HAL encargada de rastrear esta frecuencia se actualiza a este nuevo valor.
* **SysTick**: Las funciones internas de la HAL reajustan el valor de recarga (LOAD) del hardware SysTick basándose en la nueva frecuencia de 64 MHz para mantener un intervalo constante de 1 ms. La variable contadora de tiempo no se reinicia; conserva los milisegundos acumulados desde que inició.

### D. Etapa 4: Entrada al `while (1)`
* **`SystemCoreClock`**: Se mantiene constante en 64 MHz[cite: 1].
* **SysTick**: La interrupción de hardware sigue activa y ejecutándose de forma transparente en segundo plano[cite: 1, 2]. La variable de tiempo continuará incrementándose infinitamente cada milisegundo mientras la aplicación repite `app_update()`[cite: 1, 2].
