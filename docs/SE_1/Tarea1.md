# Tarea 1

> Investiga al menos 4 microcontroladores de distintas marcas y haz una tabla comparativa de:  
> - Periféricos  
> - Memoria  
> - Ecosistema  
> - Costos  
> - Arquitectura  
> - Velocidad de trabajo

---

## 1) Proyecto

Lámpara que se iluminan al ritmo de tu corazón. 



---

## 2) Tabla

| Microcontrolador | Periféricos                                                                 | Memoria                                    | Ecosistema                           | Costos       | Arquitectura                           | Velocidad de trabajo |
|-----------------:|------------------------------------------------------------------------------|--------------------------------------------|---------------------------------------|--------------|-----------------------------------------|----------------------|
| Raspberry Pi Pico | ADC, SPI, I2C, PWM, USB 1.1, 30 GPIO programables                           | 264 KB RAM, hasta 16 MB Flash externa      | MicroPython, SDK en C                 | $90–130 MX   | ARM Cortex-M0+ 32 bits, doble núcleo    | 133 MHz              |
| ESP32            | Táctiles capacitivos, Hall, LNAs, SD, Ethernet, SPI, UART, I2S, I2C          | 520 KB RAM interna                         | Arduino IDE, ESP-IDF, MicroPython, IoT | $80–180 MX   | Harvard, 32 bits                        | 240 MHz              |
| Arduino Nano     | UART, SPI, I²C, ADC (10 bits), PWM                                           | 2 KB SRAM, 32 KB Flash, 1 KB EEPROM        | Arduino IDE, bibliotecas abundantes    | $370–460 MX  | AVR ATmega328P, 8 bits                  | 16 MHz               |
| ATTiny85         | UART, I²C, SPI, ADC, PWM                                                     | 512 B SRAM, 8 KB Flash, 512 B EEPROM       | Arduino IDE                           | $20–50 MX    | AVR 8 bits                              | 8–20 MHz             |

---
## 3) Ranking de elección

1.  ESP32:
Tiene buena memoria y velocidad, ademas de que cuenta con conexión WiFi/Bluetooth lo que a futuro podría facilitar el desarrollo de una aplicación. 

2.  Raspberry Pi Pico:
Cuenta con buena memoria pero por el tema de las futuras mejoras lo coloco en segundo lugar.

3.  Arduino Nano:
Es fácil de programar pero tambien tiene poca memoria y velocidad. 

4.  ATTiny85:
Es el microcontrolador más económico pero tiene poca memoria, no podría controlar animaciones complejas en la lámpara. 