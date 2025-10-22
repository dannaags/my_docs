# Tarea 8 UART
---

## 1) Botón con instrucción "LED ON - LED OFF"

Este código permite encender y apagar un LED mediante un botón físico.
Cada vez que se presiona el botón, el LED cambia de estado: si está apagado se enciende (“LED ON”) y si está encendido se apaga (“LED OFF”).


**Código Recepción**

```C++

#include "pico/stdlib.h"
#include "hardware/uart.h"
#include <stdio.h>
#include <string.h>
 
#define UART_ID uart0
#define BAUD_RATE 115200
#define TX_PIN 0
#define RX_PIN 1
#define LED_PIN 2
 
int main() {
    stdio_init_all();
 
    gpio_set_function(TX_PIN, GPIO_FUNC_UART);
    gpio_set_function(RX_PIN, GPIO_FUNC_UART);
    uart_init(UART_ID, BAUD_RATE);
    uart_set_format(UART_ID, 8, 1, UART_PARITY_NONE);
 
    gpio_init(LED_PIN);
    gpio_set_dir(LED_PIN, GPIO_OUT);
 
    char buffer[10];
    int i = 0;
 
    while (true) {
        if (uart_is_readable(UART_ID)) {
            char c = uart_getc(UART_ID);
 
            if (c == '\n') {
                buffer[i] = '\0';  
 
                if (strcmp(buffer, "LED ON") == 0) {
                    gpio_put(LED_PIN, 1);
                    printf(" LED ON\n");
                } else if (strcmp(buffer, "LED OFF") == 0) {
                    gpio_put(LED_PIN, 0);
                    printf(" LED OFF\n");
                }
 
                i = 0;
            } else if (i < sizeof(buffer) - 1) {
                buffer[i++] = c;
            }
        }
    }
}

```


**Código Envío**

```C++
#include "pico/stdlib.h"
#include "hardware/uart.h"
#include <stdio.h>
#include <string.h>

#define LED_PIN 14        
#define BUTTON_PIN 3      
#define UART_ID uart0
#define BAUD_RATE 115200
#define TX_PIN 0
#define RX_PIN 1

int main() {
    stdio_init_all();

    uart_init(UART_ID, BAUD_RATE);
    gpio_set_function(TX_PIN, GPIO_FUNC_UART);
    gpio_set_function(RX_PIN, GPIO_FUNC_UART);

    gpio_init(BUTTON_PIN);
    gpio_set_dir(BUTTON_PIN, GPIO_IN);
    gpio_pull_up(BUTTON_PIN);

    gpio_init(LED_PIN);
    gpio_set_dir(LED_PIN, GPIO_OUT);

    bool estado = false;
    bool presionado_anterior = false;

    while (true) {
        bool presionado = !gpio_get(BUTTON_PIN);  

        if (presionado && !presionado_anterior) { 
            estado = !estado;  

            if (estado) {
                gpio_put(LED_PIN, 1);
                uart_puts(UART_ID, "LED ON\n");
                printf("LED ON\n");
            } else {
                gpio_put(LED_PIN, 0);
                uart_puts(UART_ID, "LED OFF\n");
                printf("LED OFF\n");
            }

            sleep_ms(200); 
        }

        presionado_anterior = presionado;
    }
}

 
```

**Video**

<iframe width="560" height="315" src="https://www.youtube.com/embed/hItRuD8XJHQ?si=8YOV_0s4Jcjccc1J" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


---

## 2) Terminal con instrucción "LED ON - LED OFF"

Este código permite controlar el encendido y apagado de un LED desde el monitor serial.
El usuario escribe los comandos “LED ON” o “LED OFF” en la terminal, y el microcontrolador ejecuta la acción correspondiente.

**Código Recepción**


```C++

#include "pico/stdlib.h"
#include "hardware/uart.h"
#include <stdio.h>
#include <string>

#define UART_ID uart0
#define BAUD_RATE 115200
#define TX_PIN 0
#define RX_PIN 1
#define LED_PIN 2

using namespace std;

string mensaje = "";

int main() {
    stdio_init_all();

    gpio_init(LED_PIN);
    gpio_set_dir(LED_PIN, GPIO_OUT);
    gpio_put(LED_PIN, 0);

    uart_init(UART_ID, BAUD_RATE);
    gpio_set_function(TX_PIN, GPIO_FUNC_UART);
    gpio_set_function(RX_PIN, GPIO_FUNC_UART);
    uart_set_format(UART_ID, 8, 1, UART_PARITY_NONE);

    printf("Esperando mensajes...\n");

    while (true) {
        if (uart_is_readable(UART_ID)) {
            char ch = uart_getc(UART_ID);
            printf("%c", ch);
            mensaje += ch;

            if (ch == ';') {
                string comando = mensaje.substr(0, mensaje.length() - 1);

                if (comando == "on") {
                    gpio_put(LED_PIN, 1);
                    printf("\nLED ENCENDIDO\n");
                } else if (comando == "off") {
                    gpio_put(LED_PIN, 0);
                    printf("\nLED APAGADO\n");
                } else {
                    printf("\nMensaje no reconocido: '%s'\n", comando.c_str());
                }

                mensaje = "";
            }
        }
    }
    return 0;
}


```


**Código Envío**


```C++

#include "pico/stdlib.h"
#include "hardware/uart.h"
#include <stdio.h>
#include <string>

#define UART_ID uart0
#define BAUD_RATE 115200
#define TX_PIN 0
#define RX_PIN 1

using namespace std;

string mensaje = "";

int main() {
    stdio_init_all();

    uart_init(UART_ID, BAUD_RATE);
    gpio_set_function(TX_PIN, GPIO_FUNC_UART);
    gpio_set_function(RX_PIN, GPIO_FUNC_UART);
    uart_set_format(UART_ID, 8, 1, UART_PARITY_NONE);

    sleep_ms(2000);
    printf("\nConexión lista. Escribe 'on;' o 'off;'.\n");

    while (true) {
        int ch_int = getchar_timeout_us(10000);

        if (ch_int == PICO_ERROR_TIMEOUT) {
            continue;
        }

        char ch = (char)ch_int;

        if (ch == '\r' || ch == '\n') {
            continue;
        }

        mensaje += ch;

        if (ch == ';') {
            string comando = mensaje.substr(0, mensaje.length() - 1);

            if (comando == "on" || comando == "off") {
                printf("Instrucción: %s\n", mensaje.c_str());
                uart_puts(UART_ID, mensaje.c_str());
            } else {
                printf("Instrucción inválida: '%s'\n", comando.c_str());
            }

            mensaje = "";
        }
    }
    return 0;
}




```

**Video**

<iframe width="560" height="315" src="https://www.youtube.com/embed/mgG0l3hQ_20?si=8YOV_0s4Jcjccc1J" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---