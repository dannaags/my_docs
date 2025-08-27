# Tarea 2

> Documenta en tu pagina los siguientes codigos usando logica y mascaras no se pueden poner todas las combinaciones:

---

## 1) Contador binario 4 bits

En cuatro leds debe mostrarse cad segundo la representacion binaria del 0 al 15.

**Código**

```C++

#include "pico/stdlib.h"
#include "hardware/structs/sio.h"
#define PIN_A 2
#define PIN_B 4
#define PIN_C 5
#define PIN_D 6

int main() {
    // 1) Máscara con varios pines
    const uint32_t MASK = (1u<<0) | (1u<<1) | (1u<<2)| (1u<<3);

    // 2) Asegura función SIO en cada pin (necesario una sola vez)
    gpio_init(0);
    gpio_init(1);
    gpio_init(2);
    gpio_init(3);
    // 3) Dirección: salida (OE=1) para TODOS los pines con UNA sola instrucción
    gpio_set_dir_out_masked(MASK);
    uint8_t contador=0;

    while (true) {

        gpio_put_masked(MASK, contador);
        sleep_ms(500);
        
        contador++;

        if (contador>=16){
            contador=0;
        }
}
}



```
**Esquematico de conexión**

![Diagrama del sistema](../recursos/imgs/esquematico_tarea2.jpg)


**Video**


---

## 2) Barrido de leds

Correr un “1” por cinco LEDs P0..P3 y regresar (0→1→2→3→2→1…)

**Código**

```C++

#include "pico/stdlib.h"
#include "hardware/structs/sio.h"

#define PIN_A 2
#define PIN_B 4
#define PIN_C 5
#define PIN_D 6

int main() {
    // 1) Máscara con varios pines
    const uint32_t MASK = (1u<<0) | (1u<<1) | (1u<<2)| (1u<<3);

    // 2) Asegura función SIO en cada pin (necesario una sola vez)
    gpio_init(0);
    gpio_init(1);
    gpio_init(2);
    gpio_init(3);
    // 3) Dirección: salida (OE=1) para TODOS los pines con UNA sola instrucción
    gpio_set_dir_out_masked(MASK);

    int pos=0;
    int mov=1;

    while (true) {

        gpio_put_masked(MASK, 1<<pos);
            // alto en 2,4,6
        sleep_ms(500);
        
        pos=pos+mov;

        if (pos==3){
            mov=-1;
        }

        else if (pos==0){
            mov=1;
        }
}
}


```


**Esquematico de conexión**

![Diagrama del sistema](../recursos/imgs/esquematico_tarea2.jpg)


**Video**

---
## 3) Secuencia en codigo Gray

**Código**


**Esquematico de conexión**
![Diagrama del sistema](../recursos/imgs/esquematico_tarea2.jpg)


**Video**