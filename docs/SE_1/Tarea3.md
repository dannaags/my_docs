# Tarea 3

> Documenta en tu página los siguientes códigos usando lógica y máscaras

---

## 1) Compuertas básicas AND / OR / XOR con 2 botones

 Con dos botones A y B (pull-up; presionado=0) enciende tres LEDs que muestren en paralelo los resultados de AND, OR y XOR. En el video muestra las 4 combinaciones (00, 01, 10, 11).

**Código**

```C++

#include "pico/stdlib.h"
#include "hardware/structs/sio.h"

#define Boton_1 16
#define Boton_2 17
#define LED_1 0
#define LED_2 1
#define LED_3 2

int main() {

    gpio_init(Boton_1);
    gpio_set_dir(Boton_1, false);

    gpio_init(Boton_2);
    gpio_set_dir(Boton_2, false);

    gpio_init(LED_1);
    gpio_set_dir(LED_1, true);

    gpio_init(LED_2);
    gpio_set_dir(LED_2, true);

    gpio_init(LED_3);
    gpio_set_dir(LED_3, true);

    while (true) {
        bool b1 = gpio_get(Boton_1);
        bool b2 = gpio_get(Boton_2);

        if (!b1 && !b2) {
            gpio_put(LED_1, 1);
        } else {
            gpio_put(LED_1, 0);
        }

        if (!b1 || !b2) {
            gpio_put(LED_2, 1);
        } else {
            gpio_put(LED_2, 0);
        }

        if ((b1 && !b2) || (!b1 && b2)) {
            gpio_put(LED_3, 1);
        } else {
            gpio_put(LED_3, 0);
        }

        sleep_ms(10); 
    }
}


```
**Esquematico de conexión**

![Diagrama del sistema](../recursos/imgs/esquematico1_tarea3.jpg)


**Video**

<iframe width="560" height="315" src="https://www.youtube.com/embed/MD8Lvo2fJZ4?si=GZWm4bQJwXlk-F4J" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## 2) Selector cíclico de 4 LEDs con avance/retroceso

Mantén un único LED encendido entre LED0..LED3. Un botón AVANZA (0→1→2→3→0) y otro RETROCEDE (0→3→2→1→0). Un push = un paso (antirrebote por flanco: si dejas presionado no repite). En el video demuestra en ambos sentidos.

**Código**

```C++

#include "pico/stdlib.h"
#include "hardware/structs/sio.h"

int main(void) {
    const uint LED1 = 0;
    const uint LED2 = 1; 
    const uint LED3 = 2;
    const uint LED4 = 3;       
    const uint BTN1 = 16;
    const uint BTN2 = 17; 

    const uint32_t MASK = (1u<<LED1) | (1u<<LED2) | (1u<<LED3) | (1u<<LED4);

    gpio_init_mask(MASK);
    gpio_set_dir_masked(MASK, MASK);  
    gpio_put_masked(MASK, 1u<<LED1); 

    gpio_init(BTN1);
    gpio_set_dir(BTN1,0);
    gpio_pull_up(BTN1);   
    gpio_init(BTN2);
    gpio_set_dir(BTN2,0);
    gpio_pull_up(BTN2); 

    int pos=LED1;
    int preb1 = 1;
    int preb2 = 1;

    while (true) {

        if (gpio_get(BTN1)==0 && preb1 == 1){
            if(pos==LED4) pos=LED1;
            else 
            pos++;
            gpio_put_masked(MASK, (1u<<pos));
        }

        if (gpio_get(BTN2)==0 && preb2 == 1){

            if (pos==LED1)pos=LED4;
            else 
            pos--;
            gpio_put_masked(MASK, (1u<<pos));
        }

        preb1 = gpio_get(BTN1);
        preb2 = gpio_get(BTN2);

        sleep_ms (200);

    }
}

```

**Esquematico de conexión**

![Diagrama del sistema](../recursos/imgs/esquematico2_tarea3.jpg)


**Video**

<iframe width="560" height="315" src="https://www.youtube.com/embed/O4gaQW-YsDw?si=5BoyQ03gQ8n0L9by" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---
