## Paso 07

Pasar un diagrama de estados (también conocido como Máquina de Estados Finitos) a código C es una de las habilidades más útiles en programación, especialmente para sistemas embebidos, automatización y desarrollo de videojuegos.El patrón "estándar" y más fácil de implementar en C se basa en dos pilares:Un enum para definir todos los estados posibles.Un bloque switch-case para evaluar el estado actual y decidir si ocurre una transición basada en los eventos (entradas).Para que lo veas con claridad, vamos a traducir el siguiente diagrama de ejemplo directamente a código:Traducción de Diagrama a Código CFíjate cómo cada círculo del diagrama se convierte en un case dentro del switch, y cada flecha de transición se convierte en una condición if que evalúa un evento.C#include <stdio.h>

// 1. Definimos los estados (los círculos del diagrama)
typedef enum {
    ESTADO_A,
    ESTADO_B,
    ESTADO_C,
    ESTADO_D
} Estado;

// 2. Definimos los eventos o disparadores (las flechas)
typedef enum {
    EVENTO_FORWARD,
    EVENTO_BACK,
    EVENTO_RESET,
    EVENTO_NINGUNO // Útil si en un ciclo no pasó nada
} Evento;

// Variable global (o local dentro de tu bucle principal) para el estado
Estado estadoActual = ESTADO_A; 

// 3. La lógica de la máquina de estados
void procesarEstado(Evento evento) {
    switch (estadoActual) {
        
        case ESTADO_A:
            // Desde A, solo podemos ir hacia adelante (Forward) a B
            if (evento == EVENTO_FORWARD) {
                estadoActual = ESTADO_B;
                printf("Transicion: A -> B\n");
            }
            break;

        case ESTADO_B:
            if (evento == EVENTO_FORWARD) {
                estadoActual = ESTADO_C;
                printf("Transicion: B -> C\n");
            } else if (evento == EVENTO_BACK) {
                estadoActual = ESTADO_A;
                printf("Transicion: B -> A\n");
            } else if (evento == EVENTO_RESET) {
                estadoActual = ESTADO_A;
                printf("Reset: B -> A\n");
            }
            break;

        case ESTADO_C:
            if (evento == EVENTO_FORWARD) {
                estadoActual = ESTADO_D;
                printf("Transicion: C -> D\n");
            } else if (evento == EVENTO_BACK) {
                estadoActual = ESTADO_B;
                printf("Transicion: C -> B\n");
            } else if (evento == EVENTO_RESET) {
                estadoActual = ESTADO_A;
                printf("Reset: C -> A\n");
            }
            break;

        case ESTADO_D:
            // Desde D, no hay Forward, solo podemos retroceder o resetear
            if (evento == EVENTO_BACK) {
                estadoActual = ESTADO_C;
                printf("Transicion: D -> C\n");
            } else if (evento == EVENTO_RESET) {
                estadoActual = ESTADO_A;
                printf("Reset: D -> A\n");
            }
            break;
    }
}

int main() {
    // Ejemplo de uso: simulamos una secuencia de eventos
    printf("Iniciando en Estado A...\n");
    procesarEstado(EVENTO_FORWARD); // Debería ir a B
    procesarEstado(EVENTO_FORWARD); // Debería ir a C
    procesarEstado(EVENTO_BACK);    // Debería volver a B
    procesarEstado(EVENTO_RESET);   // Debería volver a A
    
    return 0;
}
3 Consejos de Oro para tu Trabajo PrácticoSepara las acciones de las transiciones: En el ejemplo de arriba solo cambiamos de estado. En un caso real, puede que necesites encender un motor o imprimir un ticket. Es mejor que el código que "hace cosas" se ejecute después de evaluar las transiciones, o en funciones separadas, para que tu switch no se convierta en un espagueti de miles de líneas.Valida el estado inicial: Siempre asegúrate de que tu variable estadoActual esté inicializada en el estado de reposo correcto antes de que arranque el bucle de tu programa.Cuidado con los bloqueos: Asegúrate de que todos los estados tengan al menos una vía de escape (una transición a otro estado). Si un estado no tiene transiciones de salida, tu programa se quedará atascado ahí para siempre (a esto se le llama un estado "sumidero").
