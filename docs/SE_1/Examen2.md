# Examen 2

---

## 1) Control de Servomotores con comandos

# Hardware mínimo

1 × servomotor en un pin PWM (50 Hz).

3 × botones:

- **BTN_MODE:** cambia el modo activo (cíclico: Entrenamiento → Continuo → Step → …).
- **BTN_NEXT:** avanza a la siguiente posición (sólo en Step).
- **BTN_PREV:** retrocede a la posición anterior (sólo en Step).

**Pi Pico 2**

---

# Modos de operación

## 1) Modo Entrenamiento

Se recibe texto por USB-serial con los comandos siguientes (se aceptan minúsculas/mayúsculas indistintamente y también sus alias en inglés):

### Borrar (alias: clear, borrar)

**Sintaxis:**  
`Borrar`

**Efecto:** elimina la lista completa de posiciones.

**Respuesta:**  
`OK`

---

### Escribir (alias: write, escribir)

**Sintaxis:**  
`Escribir, v1, v2, ..., vn`

`vi` son enteros en 0–180.

**Efecto:** sobrescribe la lista con los valores dados en ese orden.

**Respuesta:**  
OK si todos son válidos y la lista de posiciones; si alguno está fuera de rango o la lista queda vacía →  
`Error argumento invalido`

---

### Reemplazar (alias: replace, reemplazar)

**Sintaxis:**  
`Reemplazar, i, v`

Índice `i` en base 1 (1 = primera posición).

`v` en 0–180.

**Efecto:** reemplaza el elemento `i` por `v`.

**Respuesta:**  
OK.  
Si `i` no existe → `Error indice invalido`.  
Si `v` fuera de rango → `Error argumento invalido`.

---

## 2) Modo Continuo

Recorre todas las posiciones de la lista en orden, moviendo el servo e imprimiendo cada 1.5 s:

**Formato:**  
`posX: V` (por ejemplo, `pos1: 90`), donde X es base 1.

Si la lista está vacía: imprimir cada 1.5 s  
`Error no hay pos`  
y no mover el servo.

Al cambiar a otro modo, el ciclo se detiene inmediatamente.

---

## 3) Modo Step

**BTN_NEXT:** avanza una posición (si ya está en la última, se mantiene en esa última).

**BTN_PREV:** retrocede una posición (si ya está en la primera, se mantiene en la primera).

En cada cambio de posición:

- mover el servo a la posición seleccionada;  
- imprimir `posX: V`.

Si la lista está vacía:  
al presionar **BTN_NEXT** o **BTN_PREV**, imprimir  
`Error no hay pos`  
y no mover el servo.

---

# INFO IMPORTANTE

El movimiento de un servo requiere:

- **Alimentación:** 5–6 V  
- **Señal de control:** PWM a 50 Hz  
- **Pulso:** 1–2 ms (representa 0–180 grados)


**Código**

```C++
#include "pico/stdlib.h"
#include "hardware/pwm.h"
#include <stdio.h>
#include <string>
using namespace std;


#define BUTTON_MODE 14
#define BOTON_NEXT 13
#define BOTON_PREV 11
#define SERVO_PIN 15


volatile int modo = 1;
volatile bool boton_modo_presionado = false;
volatile bool boton_next_presionado = false;
volatile bool boton_prev_presionado = false;

string comando = "";
int lista[10];
int cantidad = 0;
uint slice_num;

void cambio_modo(uint gpio, uint32_t events) {
    static uint32_t last_time = 0;
    uint32_t now = to_ms_since_boot(get_absolute_time());
    if (now - last_time < 200) return; // debounce 200 ms
    last_time = now;

    if (gpio == BUTTON_MODE)
        boton_modo_presionado = true;
    else if (gpio == BOTON_NEXT)
        boton_next_presionado = true;
    else if (gpio == BOTON_PREV)
        boton_prev_presionado = true;
}

// PARA EL SERVO
void configurar_servo() {
    gpio_set_function(SERVO_PIN, GPIO_FUNC_PWM);
    slice_num = pwm_gpio_to_slice_num(SERVO_PIN);

    pwm_set_clkdiv(slice_num, 125.0f);
    pwm_set_wrap(slice_num, 20000);
    pwm_set_enabled(slice_num, true);
}

void mover_servo(int angulo) {
    if (angulo < 0) angulo = 0;
    if (angulo > 180) angulo = 180;

    int pulso_us = 500 + (angulo * 2000) / 180;
    pwm_set_gpio_level(SERVO_PIN, pulso_us);
    printf("Servo en %d°\n", angulo);
}

// MODO 1 : ENTRENAMIENTO
void modo1() {
    printf("\nModo 1: Control de lista\n");
    printf("Comandos: escribir, reemplazar, borrar (terminar con ';')\n> ");

    comando = "";
    char c;
    while (true) {
        if (boton_modo_presionado) return;
        int ch = getchar_timeout_us(0);
        if (ch == PICO_ERROR_TIMEOUT) continue;
        c = (char)ch;
        if (c == ';') break;
        comando += c;
    }

    // BORRAR
    if (comando == "borrar") {
        cantidad = 0;
        printf("OK. Lista borrada.\n");
        return;
    }

    //ESCRIBIR
    if (comando == "escribir") {
        printf("Ingrese valores (0–180) separados por comas, termine con ';'\n> ");
        cantidad = 0;
        string valor = "";
        while (true) {
            if (boton_modo_presionado) return;
            int ch = getchar_timeout_us(0);
            if (ch == PICO_ERROR_TIMEOUT) continue;
            char c = (char)ch;

            if ((c == ',' || c == ';') && valor != "") {
                int num = atoi(valor.c_str());
                if (num < 0 || num > 180) {
                    printf("Error argumento invalido\n");
                    return;
                }
                lista[cantidad++] = num;
                valor = "";
                if (c == ';') break;
            } else if (c != ',' && c != ' ') {
                valor += c;
            }

            if (c == ';') break;
        }

        if (cantidad == 0) {
            printf("Error argumento invalido\n");
        } else {
            printf("OK. Lista: ");
            for (int i = 0; i < cantidad; i++) {
                printf("%d", lista[i]);
                if (i < cantidad - 1) printf(", ");
            }
            printf("\n");
        }
        return;
    }

    // REEMPLAZAR
    if (comando == "reemplazar") {
        if (cantidad == 0) {
            printf("No hay valores en la lista.\n");
            return;
        }

        printf("Ingrese índice y nuevo valor separados por coma, termine con ';'\n> ");
        string valor = "";
        int datos[2];
        int n = 0;

        while (true) {
            if (boton_modo_presionado) return;
            int ch = getchar_timeout_us(0);
            if (ch == PICO_ERROR_TIMEOUT) continue;
            char c = (char)ch;

            if ((c == ',' || c == ';') && valor != "") {
                datos[n++] = atoi(valor.c_str());
                valor = "";
                if (c == ';' || n == 2) break;
            } else if (c != ',' && c != ' ') {
                valor += c;
            }

            if (c == ';') break;
        }

        if (n < 2) {
            printf("Error argumento invalido\n");
            return;
        }

        int i = datos[0] - 1;
        int v = datos[1];
        if (i < 0 || i >= cantidad) {
            printf("Error indice invalido\n");
            return;
        }
        if (v < 0 || v > 180) {
            printf("Error argumento invalido\n");
            return;
        }

        lista[i] = v;
        printf("OK. Lista actualizada: ");
        for (int j = 0; j < cantidad; j++) {
            printf("%d", lista[j]);
            if (j < cantidad - 1) printf(", ");
        }
        printf("\n");
        return;
    }

    printf("Comando no reconocido.\n");
}

//MODO 2 CONTINUO
void modo2() {
    printf("\nModo 2: Movimiento continuo\n");
    if (cantidad == 0) {
        printf("No hay valores en la lista.\n");
        sleep_ms(500);
        return;
    }

    printf("Moviendo continuamente según lista...\n");
    while (!boton_modo_presionado) {
        for (int i = 0; i < cantidad; i++) {
            mover_servo(lista[i]);
            sleep_ms(1500);
            if (boton_modo_presionado) return;
        }
    }
}

// MODO 3 PASO A PASO
void modo3() {
    if (cantidad == 0) {
        printf("No hay valores en la lista.\n");
        sleep_ms(500);
        return;
    }

    static int indice = 0;
    mover_servo(lista[indice]);

    while (!boton_modo_presionado) {
        if (boton_next_presionado) {
            boton_next_presionado = false;
            indice++;
            if (indice >= cantidad) indice = 0;
            mover_servo(lista[indice]);
        }

        if (boton_prev_presionado) {
            boton_prev_presionado = false;
            indice--;
            if (indice < 0) indice = cantidad - 1;
            mover_servo(lista[indice]);
        }

        sleep_ms(20);
    }
}

//CODIGO PRINCIPAL
int main() {
    stdio_init_all();
    sleep_ms(500);

    gpio_init(BUTTON_MODE);
    gpio_set_dir(BUTTON_MODE, GPIO_IN);

    gpio_init(BOTON_NEXT);
    gpio_set_dir(BOTON_NEXT, GPIO_IN);

    gpio_init(BOTON_PREV);
    gpio_set_dir(BOTON_PREV, GPIO_IN);

    gpio_set_irq_enabled_with_callback(BUTTON_MODE, GPIO_IRQ_EDGE_RISE, true, &cambio_modo);
    gpio_set_irq_enabled_with_callback(BOTON_NEXT, GPIO_IRQ_EDGE_RISE, true, &cambio_modo);
    gpio_set_irq_enabled_with_callback(BOTON_PREV, GPIO_IRQ_EDGE_RISE, true, &cambio_modo);

    configurar_servo();
    printf("Programa iniciado. Modo 1.\n");

    while (true) {
        if (boton_modo_presionado) {
            boton_modo_presionado = false;
            modo++;
            if (modo > 3) modo = 1;
            printf("\n--- Modo cambiado a: %d ---\n", modo);
        }

        if (modo == 1) modo1();
        else if (modo == 2) modo2();
        else if (modo == 3) modo3();

        sleep_ms(50);
    }
}


```


**Video**

[Ver video en YouTube](https://youtube.com/shorts/V5-u_odLcfA)

<iframe width="560" height="315" src="https://www.youtube.com/embed/9hpX5afNj2s?si=bhrV-okcdAV0-Pva" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---
