# Trabajo Práctico: Codificación de Diagramas de Estado en C

## 1. Concepto Básico
La forma más estructurada, legible y común de traducir un diagrama de estados (también conocido como Máquina de Estados Finitos o FSM) a lenguaje C se basa en dos pilares fundamentales:
*   **Enumeraciones (`enum`)**: Se utilizan para definir todos los estados posibles (los círculos del diagrama) y los eventos (las flechas o condiciones de transición).
*   **Estructura condicional (`switch-case`)**: Se emplea para evaluar en qué estado nos encontramos actualmente y qué transición debemos ejecutar en función del evento recibido.

## 2. Plantilla de Código C
A continuación, se presenta un código base funcional. En este ejemplo, el sistema transita entre tres estados: Inicial, Activo y Error.

```c
#include <stdio.h>

// 1. Definir los estados posibles (Nodos del diagrama)
typedef enum {
    ESTADO_INICIAL,
    ESTADO_ACTIVO,
    ESTADO_ERROR
} Estado_t;

// 2. Definir los eventos (Flechas o disparadores)
typedef enum {
    EVENTO_INICIAR,
    EVENTO_FALLO,
    EVENTO_RESET
} Evento_t;

// Variable global o local que guarda la memoria del sistema
Estado_t estado_actual = ESTADO_INICIAL;

// 3. Función principal de la Máquina de Estados
void actualizar_estado(Evento_t evento) {
    switch (estado_actual) {
        
        case ESTADO_INICIAL:
            if (evento == EVENTO_INICIAR) {
                estado_actual = ESTADO_ACTIVO;
                printf("Transicion: INICIAL -> ACTIVO\n");
            }
            break;
            
        case ESTADO_ACTIVO:
            if (evento == EVENTO_FALLO) {
                estado_actual = ESTADO_ERROR;
                printf("Transicion: ACTIVO -> ERROR\n");
            }
            break;
            
        case ESTADO_ERROR:
            if (evento == EVENTO_RESET) {
                estado_actual = ESTADO_INICIAL;
                printf("Transicion: ERROR -> INICIAL\n");
            }
            break;
            
        default:
            // Buenas prácticas: Manejo de seguridad
            estado_actual = ESTADO_INICIAL;
            break;
    }
}

int main() {
    printf("Sistema iniciado.\n");
    
    // Simulacion de eventos que llegan al sistema
    actualizar_estado(EVENTO_INICIAR); // Transita a ACTIVO
    actualizar_estado(EVENTO_FALLO);   // Transita a ERROR
    actualizar_estado(EVENTO_RESET);   // Transita a INICIAL
    
    return 0;
}
