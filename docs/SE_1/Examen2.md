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

#include "pico/stdlib.h"    // Funciones básicas del RP2040
#include "hardware/pwm.h"   // Control PWM para el servo
#include <stdio.h>         // Entrada/salida estándar
#include <string>          // Manejo de strings para comandos
using namespace std;
 
// Definición de pines GPIO
#define BUTTON_MODE 14     // Pin para botón de cambio de modo
#define BOTON_NEXT 13     // Pin para botón de avanzar posición
#define BOTON_PREV 11     // Pin para botón de retroceder posición
#define SERVO_PIN 1      // Pin para la señal PWM del servo
 
// Variables para control de modo y estado de botones
// volatile: pueden cambiar en ISR
volatile int modo = 1;                         // Modo actual (1-3)
volatile bool boton_modo_presionado = false;   // Estado del botón modo
volatile bool boton_next_presionado = false;   // Estado del botón next
volatile bool boton_prev_presionado = false;   // Estado del botón prev
 
// Variables para el control de la lista de posiciones
string comando = "";       // Almacena el comando actual siendo ingresado
int lista[10];            // Arreglo de posiciones del servo (0-180 grados)
int cantidad = 0;         // Cantidad actual de posiciones guardadas
uint slice_num;           // Número de slice PWM asignado al servo
 
void cambio_modo(uint gpio, uint32_t events) {
    // Variables de tiempo para el debouncing
    static uint32_t last_time = 0;     // Último tiempo de activación (persiste entre llamadas)
    uint32_t now = to_ms_since_boot(get_absolute_time());  // Tiempo actual
   
    // Implementación de debounce por software
    if (now - last_time < 200)         // Si han pasado menos de 200ms
        return;                        // Ignora la interrupción (probablemente rebote)
    last_time = now;                  // Actualiza el tiempo de última activación válida
 
    // Identifica qué botón causó la interrupción y activa su bandera
    if (gpio == BUTTON_MODE)          // ¿Fue el botón de modo?
        boton_modo_presionado = true;
    else if (gpio == BOTON_NEXT)      // ¿Fue el botón siguiente?
        boton_next_presionado = true;
    else if (gpio == BOTON_PREV)      // ¿Fue el botón anterior?
        boton_prev_presionado = true;
}
 
 
void configurar_servo() {
    // Configura el pin como salida PWM
    gpio_set_function(SERVO_PIN, GPIO_FUNC_PWM);
 
    // Obtiene el número de slice PWM asignado al pin
    slice_num = pwm_gpio_to_slice_num(SERVO_PIN);
 
    // Divisor de clock para ticks de 1µs (125MHz/125 = 1MHz)
    pwm_set_clkdiv(slice_num, 125.0f);
 
    // Configura periodo de 20ms (20000µs) para frecuencia de 50Hz
    pwm_set_wrap(slice_num, 20000);
 
    // Habilita la salida PWM
    pwm_set_enabled(slice_num, true);
}
 
// Función para mover el servo a un ángulo específico
void mover_servo(int angulo) {
    // Limita el ángulo al rango válido
    if (angulo < 0) angulo = 0;        // No permitir ángulos negativos
    if (angulo > 180) angulo = 180;    // No permitir ángulos mayores a 180
 
    // Convierte ángulo a ancho de pulso:
    // - Mínimo: 500µs (0 grados)
    // - Máximo: 2500µs (180 grados)
    int pulso_us = 500 + (angulo * 2000) / 180;
 
    // Aplica el ancho de pulso al PWM
    pwm_set_gpio_level(SERVO_PIN, pulso_us);
 
    // Muestra el ángulo actual por consola
    printf("Servo en %d°\n", angulo);
}
 /* La fórmula mapea linealmente el ángulo al ancho de pulso:
 * pulso = 500 + (angulo * 2000)/180
 *
 * La función limita el ángulo al rango válido y muestra feedback por serial
 */
 
void modo1() {
    // Muestra el menú de comandos disponibles
    printf("\nModo 1: Control de lista\n");
    printf("Comandos: escribir, reemplazar, borrar (terminar con ';')\n> ");
 
    // Inicializa variables para lectura de comando
    comando = "";             // Buffer para almacenar el comando
    char c;                  // Variable para cada carácter leído
   
    // Loop de lectura de caracteres
    while (true) {
        // Verifica si se debe salir del modo
        if (boton_modo_presionado) return;
       
        // Intenta leer un carácter (timeout = 0 para no bloquear)
        int ch = getchar_timeout_us(0);
       
        // Si no hay caracteres disponibles, continúa el loop
        if (ch == PICO_ERROR_TIMEOUT) continue;
       
        // Convierte el código ASCII a carácter
        c = (char)ch;
       
        // Si es punto y coma, termina la lectura del comando
        if (c == ';') break;
       
        // Agrega el carácter al comando en construcción
        comando += c;
    }
 
    // === Comando BORRAR: Elimina todas las posiciones guardadas ===
    if (comando == "borrar") {
        cantidad = 0;                    // Reinicia el contador de posiciones a cero
        printf("OK. Lista borrada.\n");  // Informa al usuario que la operación fue exitosa
        return;                         // Sale de la función modo1
    }
 
    // === Comando ESCRIBIR: Ingresa una nueva secuencia de posiciones ===
    if (comando == "escribir") {
        // Solicita al usuario ingresar los valores con el formato correcto
        printf("Ingrese valores (0–180) separados por comas, termine con ';'\n> ");
        cantidad = 0;                    // Reinicia el contador para la nueva lista
        string valor = "";              // Buffer temporal para construir cada número
        while (true) {
            // Verifica si se debe salir del modo
            if (boton_modo_presionado) return;
            // Intenta leer un carácter de forma no bloqueante
            int ch = getchar_timeout_us(0);
            // Si no hay datos disponibles, continúa el loop
            if (ch == PICO_ERROR_TIMEOUT) continue;
            // Convierte el código ASCII a carácter
            char c = (char)ch;
 
            // Si encuentra separador y hay un número acumulado
            if ((c == ',' || c == ';') && valor != "") {
                // Convierte la cadena a número
                int num = atoi(valor.c_str());
                // Valida que el ángulo esté en el rango permitido
                if (num < 0 || num > 180) {
                    printf("Error argumento invalido\n");
                    return;
                }
                // Almacena el número y aumenta el contador
                lista[cantidad++] = num;
                valor = "";              // Limpia el buffer para el siguiente número
                if (c == ';') break;    // Si es punto y coma, termina la entrada
            } else if (c != ',' && c != ' ') {
                // Si no es separador ni espacio, acumula el dígito
                valor += c;
            }
 
            // Termina si encuentra punto y coma (redundante pero seguro)
            if (c == ';') break;
        }
 
        // Verifica que se haya ingresado al menos un valor
        if (cantidad == 0) {
            printf("Error argumento invalido\n");
        } else {
            // Muestra la lista completa ingresada
            printf("OK. Lista: ");
            for (int i = 0; i < cantidad; i++) {
                printf("%d", lista[i]);
                // Agrega coma excepto después del último número
                if (i < cantidad - 1) printf(", ");
            }
            printf("\n");
        }
        return;
    }
 
    // === Comando REEMPLAZAR: Modifica una posición existente ===
    if (comando == "reemplazar") {
        // Verifica que haya posiciones para modificar
        if (cantidad == 0) {
            printf("No hay valores en la lista.\n");
            return;
        }
 
        // Solicita el índice y nuevo valor con formato específico
        printf("Ingrese índice y nuevo valor separados por coma, termine con ';'\n> ");
        string valor = "";              // Buffer para construir los números
        int datos[2];                  // Almacena [índice, nuevo_valor]
        int n = 0;                     // Contador de números leídos
 
        // Loop de lectura de caracteres
        while (true) {
            // Permite salir con el botón de modo
            if (boton_modo_presionado) return;
            // Lectura no bloqueante de caracteres
            int ch = getchar_timeout_us(0);
            if (ch == PICO_ERROR_TIMEOUT) continue;
            char c = (char)ch;
 
            // Procesa el número cuando encuentra un separador
            if ((c == ',' || c == ';') && valor != "") {
                datos[n++] = atoi(valor.c_str());    // Convierte y guarda el número
                valor = "";                          // Limpia buffer para siguiente número
                if (c == ';' || n == 2) break;      // Sale si tiene los dos valores
            } else if (c != ',' && c != ' ') {
                // Acumula dígitos si no es separador ni espacio
                valor += c;
            }
 
            // Sale si encuentra punto y coma (redundante pero seguro)
            if (c == ';') break;
        }
 
        // Verifica que se ingresaron dos números
        if (n < 2) {
            printf("Error argumento invalido\n");
            return;
        }
 
        // Procesa los valores ingresados
        int i = datos[0] - 1;          // Convierte índice a base 0
        int v = datos[1];              // Nuevo valor a guardar
       
        // Valida que el índice esté en rango
        if (i < 0 || i >= cantidad) {
            printf("Error indice invalido\n");
            return;
        }
        // Valida que el ángulo esté en rango
        if (v < 0 || v > 180) {
            printf("Error argumento invalido\n");
            return;
        }
 
        // Actualiza la posición y muestra la lista completa
        lista[i] = v;
        printf("OK. Lista actualizada: ");
        for (int j = 0; j < cantidad; j++) {
            printf("%d", lista[j]);
            // Agrega coma excepto después del último número
            if (j < cantidad - 1) printf(", ");
        }
        printf("\n");
        return;
    }
 
    // Si llegamos aquí, el comando no coincide con ninguno de los válidos
    printf("Comando no reconocido.\n");
}
 
 
void modo2() {
    printf("\nModo 2: Movimiento continuo\n");
    // Verificar que haya posiciones para reproducir
    if (cantidad == 0) {
        printf("No hay valores en la lista.\n");
        sleep_ms(500);
        return;
    }
 
    printf("Moviendo continuamente según lista...\n");
    while (!boton_modo_presionado) {           // Loop principal
        for (int i = 0; i < cantidad; i++) {   // Recorrer lista
            mover_servo(lista[i]);             // Mover a posición actual
            sleep_ms(1500);                    // Esperar entre movimientos
            if (boton_modo_presionado) return; // Permitir salir en cualquier momento
        }
    }
}
 
 
void modo3() {
    // Verificar que haya posiciones para mostrar
    if (cantidad == 0) {
        printf("No hay valores en la lista.\n");
        sleep_ms(500);
        return;
    }
 
    static int indice = 0;                    // Mantener posición entre llamadas
    mover_servo(lista[indice]);               // Mostrar posición inicial
 
    while (!boton_modo_presionado) {
        // Procesar botón NEXT
        if (boton_next_presionado) {
            boton_next_presionado = false;    // Limpiar bandera
            indice++;                         // Avanzar
            if (indice >= cantidad) indice = 0; // Volver al inicio si necesario
            mover_servo(lista[indice]);       // Actualizar posición
        }
 
        // Procesar botón PREV
        if (boton_prev_presionado) {
            boton_prev_presionado = false;    // Limpiar bandera
            indice--;                         // Retroceder
            if (indice < 0) indice = cantidad - 1; // Ir al final si necesario
            mover_servo(lista[indice]);       // Actualizar posición
        }
 
        sleep_ms(20);                        // Breve delay para estabilidad
    }
}
 
 
int main() {
    // Inicialización de comunicación serial
    stdio_init_all();                    // Habilita UART para comunicación
    sleep_ms(500);                       // Espera para estabilización del sistema
 
    // === Configuración de GPIOs para botones ===
    // Botón de cambio de modo
    gpio_init(BUTTON_MODE);              // Inicializa el pin
    gpio_set_dir(BUTTON_MODE, GPIO_IN);  // Configura como entrada
 
    // Botón para avanzar en la secuencia
    gpio_init(BOTON_NEXT);               // Inicializa el pin
    gpio_set_dir(BOTON_NEXT, GPIO_IN);   // Configura como entrada
 
    // Botón para retroceder en la secuencia
    gpio_init(BOTON_PREV);               // Inicializa el pin
    gpio_set_dir(BOTON_PREV, GPIO_IN);   // Configura como entrada
 
    // === Configuración de interrupciones ===
    // Todos los botones:
    // - Activan en flanco de subida (presionar)
    // - Usan la misma función de callback
    // - Tienen debouncing por software
    gpio_set_irq_enabled_with_callback(BUTTON_MODE, GPIO_IRQ_EDGE_RISE, true, &cambio_modo);
    gpio_set_irq_enabled_with_callback(BOTON_NEXT, GPIO_IRQ_EDGE_RISE, true, &cambio_modo);
    gpio_set_irq_enabled_with_callback(BOTON_PREV, GPIO_IRQ_EDGE_RISE, true, &cambio_modo);
 
    // === Inicialización del sistema ===
    configurar_servo();                  // Configura PWM para el servo
    printf("Programa iniciado. Modo 1.\n");  // Mensaje inicial
 
    // === Loop principal del programa ===
    while (true) {
        // Manejo del cambio de modo
        if (boton_modo_presionado) {
            boton_modo_presionado = false;    // Limpia la bandera
            modo++;                           // Avanza al siguiente modo
            if (modo > 3) modo = 1;           // Vuelve a modo 1 si excede
            printf("\n--- Modo cambiado a: %d ---\n", modo);  // Notifica cambio
        }
 
        // Ejecución del modo actual
        if (modo == 1) modo1();              // Modo Entrenamiento
        else if (modo == 2) modo2();         // Modo Continuo
        else if (modo == 3) modo3();         // Modo Paso a Paso
 
        sleep_ms(50);
    }
}

```


**Video**

[Ver video en YouTube](https://youtube.com/shorts/V5-u_odLcfA)

<iframe width="560" height="315" src="https://www.youtube.com/embed/9hpX5afNj2s?si=bhrV-okcdAV0-Pva" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---
