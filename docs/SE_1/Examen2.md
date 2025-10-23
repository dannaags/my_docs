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
#include <stdio.h>
#include "hardware/pwm.h"
#include "hardware/gpio.h"
#include <string.h> 

#define BOTON_MODO 11
#define BOTON_NEXT 13
#define BOTON_PREV 14
#define PWM 10
#define SERVO_MIN_US 500  //Valores que se pusieron despues de mapear nuestro servo
#define SERVO_MAX_US 2500

static int modo = 0; // 0=Entrenamiento, 1=Continuo, 2=Step
static int inicio_step = -1;  // índice actual para modo Step y es -1 porque no se ha seleccionado ninguna posición
static int inicio_cont = -1; // índice actual para modo Continuo es -1 porque no ha comenzado la secuencia
static uint32_t last_ms = 0; // tiempo del último movimiento en modo continuo
static const uint32_t PERIODO_MS = 1500; // tiempo entre movimientos en modo continuo

#define REBOTE 200 //Tiempo de rebote para nuestros botones
static volatile bool g_btn_mode = false;
static volatile bool g_btn_next = false;
static volatile bool g_btn_prev = false;
static volatile uint32_t t_last_mode = 0;
static volatile uint32_t t_last_next = 0;
static volatile uint32_t t_last_prev = 0;

#define CMD_BUFFER_SIZE 32
static char cmd_buffer[CMD_BUFFER_SIZE];
static int cmd_len = 0;

// arreglo para guardar las posiciones
int Posiciones[10];

// PARA EL SERVO
static uint servo_slice = 0;
static uint servo_chan  = 0;
static const uint16_t SERVO_TOP = 20000;
static const float    SERVO_DIV = 150.0f;


void servo_pwm_init() {
    gpio_set_function(PWM, GPIO_FUNC_PWM);
    servo_slice = pwm_gpio_to_slice_num(PWM);
    servo_chan  = pwm_gpio_to_channel(PWM);
    pwm_set_clkdiv(servo_slice, SERVO_DIV);
    pwm_set_wrap(servo_slice, SERVO_TOP);
    uint16_t mid = (SERVO_MIN_US + SERVO_MAX_US) / 2; //Para el valor medio del servo
    pwm_set_chan_level(servo_slice, servo_chan, mid);
    pwm_set_enabled(servo_slice, true);
}

void servo_mover_grados(int deg) {
    if (deg < 0)  deg = 0;
    if (deg > 180) deg = 180;
    int us = SERVO_MIN_US + (deg * (SERVO_MAX_US - SERVO_MIN_US)) / 180;
    pwm_set_chan_level(servo_slice, servo_chan, (uint16_t)us);
    printf("Servo -> %d° \n", deg);
}

// MODOS


static bool hay_lista() {
    for (int i=0;i<10;i++) if (Posiciones[i] >= 0 && Posiciones[i] <= 180) return true;
    return false;
}
static int obtener_primero() {
    for (int i=0;i<10;i++) if (Posiciones[i] >= 0 && Posiciones[i] <= 180) return i;
    return -1;
}
static int obtener_siguiente(int i) {
    if (i < 0) return obtener_primero();
    for (int k=i+1;k<10;k++) if (Posiciones[k] >= 0 && Posiciones[k] <= 180) return k;
    return -1; // -1 significa que es el último válido
}
static int obtener_anterior(int i) {
    if (i < 0) return obtener_primero();
    for (int k=i-1;k>=0;k--) if (Posiciones[k] >= 0 && Posiciones[k] <= 180) return k;
    return i;
}
static void imprimir_pos(int idx) {
    printf("pos%d: %d\n", idx+1, Posiciones[idx]);
}

// INTERRUPCIÓN GPIO


static void boton_isr(uint gpio, uint32_t events) {
    if (events & GPIO_IRQ_EDGE_RISE) {
        uint32_t now = to_ms_since_boot(get_absolute_time());
        if (gpio == BOTON_MODO) {
            if (now - t_last_mode > REBOTE) {
                g_btn_mode = true;
                t_last_mode = now;
            }
        } else if (gpio == BOTON_NEXT) {
            if (now - t_last_next > REBOTE) {
                g_btn_next = true;
                t_last_next = now;
            }
        } else if (gpio == BOTON_PREV) {
            if (now - t_last_prev > REBOTE) {
                g_btn_prev = true;
                t_last_prev = now;
            }
        }
    }
}

void borrarPosiciones() {
    for(int i = 0; i < 10; i++) Posiciones[i] = -1;
    printf("Todas las posiciones han sido borradas.\n");
}

void escribirPosiciones() {
    printf("\nIngrese posiciones (0-180) separadas por comas y termine con ';'\n");
    printf("Ejemplo: 10,45,90,180;\n");
    for(int i=0;i<10;i++) Posiciones[i] = -1;
    int numPos = 0;
    int val = 0;
    bool leyendoNumero = false;
    bool terminado = false;
    while (!terminado) {
        int ch = getchar_timeout_us(10000);
        if (ch == PICO_ERROR_TIMEOUT) continue;
        if (ch == '\r' || ch == '\n') continue;
        char c = (char)ch;
        printf("%c", c);
        if (c >= '0' && c <= '9') {
            if (!leyendoNumero) { val = 0; leyendoNumero = true; }
            val = val * 10 + (c - '0');
            if (val > 1000000) { leyendoNumero = false; val = 0; }
        } else if (c == ',' || c == ';') {
            if (leyendoNumero) {
                if (numPos >= 10) {
                    printf("\nSe excedió el límite de 10 posiciones. Se usan solo las primeras 10.\n");
                } else {
                    if (val >= 0 && val <= 180) {
                        Posiciones[numPos] = val;
                        printf("\nPosición %d guardada: %d°\n", numPos, val);
                        numPos++;
                    } else {
                        printf("\nValor inválido '%d'. Debe estar entre 0 y 180. Ignorado.\n", val);
                    }
                }
            }
            val = 0;
            leyendoNumero = false;
            if (c == ';') terminado = true;
        }
    }
    printf("\nSe guardaron %d posiciones.\n", numPos);
}

void reemplazarPosicion() {
    printf("\nIgrese la posicion que desea reemplazar y el valor que quiere seguido de una ,\n");
    int pos = -1, val = -1;
    int current_num = 0;
    bool reading_index = true;
    bool reading_value = false;
    bool terminado = false;

    while (!terminado) {
        int ch = getchar_timeout_us(10000);
        if (ch == PICO_ERROR_TIMEOUT || ch == '\r' || ch == '\n' || ch == ' ') continue;

        char c = (char)ch;
        printf("%c", c);

        if (c >= '0' && c <= '9') {
            current_num = current_num * 10 + (c - '0');
        } else if (c == ',') {
            if (reading_index) {
                pos = current_num;
                current_num = 0;
                reading_index = false;
                reading_value = true;
            } else {
                printf("\nError de formato: coma inesperada.\n"); terminado = true;
            }
        } else if (c == ';') {
            if (reading_value) {
                val = current_num;
                terminado = true;
            } else {
                printf("\nError de formato: punto y coma inesperado.\n"); terminado = true;
            }
        } else {
            printf("\nError de formato: carácter '%c' inesperado.\n", c); terminado = true;
        }
    }

    if (pos >= 0 && pos < 10 && val >= 0 && val <= 180) {
        if (Posiciones[pos] != -1) {
            printf("\nReemplazando posición %d: %d° → %d°\n", pos, Posiciones[pos], val);
            Posiciones[pos] = val;
        } else {
            printf("\nError: La posición %d está vacía.\n", pos);
        }
    } else if (pos != -1 || val != -1) {
         printf("\nError: índice (%d) o valor (%d) fuera de rango, o formato incorrecto.\n", pos, val);
    }
}

void mostrarPosiciones() {
    printf("\n=== Posiciones Actuales ===\n");
    bool hay = false;
    for(int i = 0; i < 10; i++) {
        if(Posiciones[i] != -1) { printf("Posición %d: %d°\n", i, Posiciones[i]); hay = true; }
    }
    if(!hay) printf("No hay posiciones guardadas.\n");
    printf("========================\n");
}

static void continuo_tick() {
    uint32_t ms = to_ms_since_boot(get_absolute_time());
    if (ms - last_ms < PERIODO_MS) return;
    last_ms = ms;
    if (!hay_lista()) { printf("Error: no hay posiciones guardadas.\n"); return; }
    if (inicio_cont < 0) {
        inicio_cont = obtener_primero();
    } else {
        int siguiente = obtener_siguiente(inicio_cont);
        if (siguiente == -1) {
            inicio_cont = obtener_primero();
            printf("Reiniciando lista...\n");
        } else {
            inicio_cont = siguiente;
        }
    }
    if (inicio_cont >= 0) {
        servo_mover_grados(Posiciones[inicio_cont]);
        imprimir_pos(inicio_cont);
    }
}

static void step_handle_buttons() {
    if (!hay_lista()) {
        if (g_btn_next || g_btn_prev) printf("Error no hay pos\n");
        g_btn_next = false; g_btn_prev = false;
        return;
    }
    if (inicio_step < 0) inicio_step = obtener_primero();
    if (g_btn_next) {
        g_btn_next = false;
        int nuevo = obtener_siguiente(inicio_step);
        if (nuevo >= 0) inicio_step = nuevo;
        servo_mover_grados(Posiciones[inicio_step]);
        imprimir_pos(inicio_step);
    }
    if (g_btn_prev) {
        g_btn_prev = false;
        int nuevo = obtener_anterior(inicio_step);
        inicio_step = nuevo;
        servo_mover_grados(Posiciones[inicio_step]);
        imprimir_pos(inicio_step);
    }
}



void manejar_comando(const char* comando) {
    if (strcmp(comando, "Borrar") == 0){
        borrarPosiciones();
    } else if (strcmp(comando, "Escribir") == 0){
        escribirPosiciones();
    } else if (strcmp(comando, "Mostrar") == 0) {
        mostrarPosiciones();
    } else if (strcmp(comando, "Reemplazar") == 0){
        reemplazarPosicion();
    } else if (comando[0] != '\0') {
        printf("Instrucción inválida: '%s'\n", comando);
    }
}

int main() {
    stdio_init_all();
    servo_pwm_init();
    servo_mover_grados(90);
    borrarPosiciones();

    gpio_init(BOTON_MODO); gpio_set_dir(BOTON_MODO, GPIO_IN); gpio_disable_pulls(BOTON_MODO);
    gpio_init(BOTON_NEXT); gpio_set_dir(BOTON_NEXT, GPIO_IN); gpio_disable_pulls(BOTON_NEXT);
    gpio_init(BOTON_PREV); gpio_set_dir(BOTON_PREV, GPIO_IN); gpio_disable_pulls(BOTON_PREV);

    gpio_set_irq_enabled_with_callback(BOTON_MODO, GPIO_IRQ_EDGE_RISE, true, &boton_isr);
    gpio_set_irq_enabled(BOTON_NEXT, GPIO_IRQ_EDGE_RISE, true);
    gpio_set_irq_enabled(BOTON_PREV, GPIO_IRQ_EDGE_RISE, true);

    sleep_ms(1500);
    printf("\nMODOS: [0] Entrenamiento  [1] Continuo  [2] Step\n");
    printf("Inicio en MODO 0 (Entrenamiento)\n");

    last_ms = to_ms_since_boot(get_absolute_time());

    while (true) {
        if (g_btn_mode) {
            g_btn_mode = false;
            modo++; if (modo > 2) modo = 0;
            g_btn_next = false; g_btn_prev = false;
            if (modo == 0) {
                printf("\n>> MODO 0: Entrenamiento\n");
                cmd_len = 0;
            } else if (modo == 1) {
                printf("\n>> MODO 1: Continuo\n");
                inicio_cont = -1;
                last_ms = to_ms_since_boot(get_absolute_time());
            } else {
                printf("\n>> MODO 2: Step\n");
                inicio_step = obtener_primero();
                if (inicio_step >= 0) {
                    servo_mover_grados(Posiciones[inicio_step]);
                    imprimir_pos(inicio_step);
                } else {
                    printf("Error no hay pos\n");
                }
            }
        }

        if (modo == 0) {
            int ch_int = getchar_timeout_us(10000);
            if (ch_int != PICO_ERROR_TIMEOUT) {
                char ch = (char)ch_int;
                if (ch != '\r' && ch != '\n' && ch != ';') {
                    if (cmd_len < (CMD_BUFFER_SIZE - 2)) {
                        cmd_buffer[cmd_len++] = ch;
                    }
                } else if (ch == ';') {
                    printf("%c", ch);
                    if (cmd_len > 0) {
                        cmd_buffer[cmd_len] = '\0';
                        manejar_comando(cmd_buffer);
                    }
                    cmd_len = 0;
                } else {
                    printf("%c", ch);
                }
            }
        } else if (modo == 1) {
            continuo_tick();
        } else {
            step_handle_buttons();
        }

        tight_loop_contents();
    }
    return 0;
}

```


**Video**

[Ver video en YouTube](https://youtube.com/shorts/V5-u_odLcfA)

<iframe width="560" height="315" src="https://www.youtube.com/embed/9hpX5afNj2s?si=bhrV-okcdAV0-Pva" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---
