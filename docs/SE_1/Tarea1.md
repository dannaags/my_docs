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


- **General:** _Qué se pretende lograr en términos amplios._
- **Específicos:**
  - _OE1…_
  - _OE2…_
  - _OE3…_

## 3) Alcance y Exclusiones

- **Incluye:** _Qué funcionalidades/entregables sí están en el proyecto._
- **No incluye:** _Qué queda fuera para evitar malentendidos._

---

## 4) Requisitos

**Software**
- _SO compatible (Windows/Linux/macOS)_
- _Python 3.x / Node 18+ / Arduino IDE / etc._
- _Dependencias (p. ej., pip/requirements, npm packages)_

**Hardware (si aplica)**
- _MCU / Sensores / Actuadores / Fuente de poder_
- _Herramientas (multímetro, cautín, etc.)_

**Conocimientos previos**
- _Programación básica en X_
- _Electrónica básica_
- _Git/GitHub_

---

## 5) Instalación

```bash
# 1) Clonar
git clone https://github.com/<usuario>/<repo>.git
cd <repo>

# 2) (Opcional) Crear entorno virtual
python -m venv .venv
# macOS/Linux
source .venv/bin/activate
# Windows (PowerShell)
.venv\Scripts\Activate.ps1

# 3) Instalar dependencias (ejemplos)
pip install -r requirements.txt
# o, si es Node:
npm install


```