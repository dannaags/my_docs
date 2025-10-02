# Tarea 7.2 PWM
---

## 1) Control de Frecuencia — Canción con Buzzer

!!! note "PUNTO EXTRA"
    PUNTO EXTRA


Programar un buzzer piezoeléctrico para reproducir una melodía reconocible.

Variar la frecuencia del PWM para las notas, manteniendo el duty en 50 %.

Cada nota debe incluir su frecuencia y duración en el código.

Documentar:

- Tabla con notas, frecuencias y duración usadas.
- Evidencia en audio o video de la melodía funcionando.




!!! tip "Recomendación"
    La mejor frecuencia de trabajo del buzzer es típicamente entre 532 Hz y 4 kHz y adaptar las notas a una octava que suene clara en ese rango.


**Tabla con notas, frecuencias y duración usadas.**

# Notas Musicales

| NOTA | FRECUENCIA | DURACIÓN |
|------|------------|----------|
| **ESTROFA 1** |            |          |
| DO   | 560        | 125      |
| DO   | 560        | 125      |
| DO   | 560        | 125      |
| FA   | 700        | 200      |
| LA   | 880        | 200      |
| **ESTROFA 2** |            |          |
| DO   | 560        | 125      |
| DO   | 560        | 125      |
| DO   | 560        | 125      |
| FA   | 700        | 200      |
| LA   | 880        | 200      |
| **ESTROFA 3** |            |          |
| FA   | 700        | 100      |
| FA   | 700        | 100      |
| MI   | 720        | 100      |
| MI   | 720        | 100      |
| RE   | 600        | 100      |
| RE   | 600        | 100      |
| DO   | 560        | 200      |
| **ESTROFA 4** |            |          |
| DO   | 560        | 125      |
| DO   | 560        | 125      |
| DO   | 560        | 125      |
| MI   | 720        | 125      |
| SOL  | 800        | 125      |
| **ESTROFA 5** |            |          |
| DO   | 560        | 125      |
| DO   | 560        | 125      |
| DO   | 560        | 125      |
| MI   | 720        | 125      |
| SOL  | 800        | 125      |
| **ESTROFA 6** |            |          |
| SOL  | 800        | 125      |
| LA   | 880        | 125      |
| SOL  | 800        | 125      |
| FA   | 700        | 125      |
| MI   | 720        | 125      |
| RE   | 600        | 125      |
| DO   | 560        | 125      |



**Código**

```C++

#include "pico/stdlib.h"
#include "hardware/pwm.h"

#define LED_PIN 0
#define TOP 4096
#define MI  660.0f
#define SOL 800.0f
#define DO 560.0f
#define SI  1000.0f
#define RE  600.0f
#define LA  880.0f
#define FA  700.0f
#define DO2  525.0f 
#define RE2  1200.0f
#define NADA 1.0f

#define delay_nota0 100
#define delay_nota1 125
#define delay_nota2 200
#define delay_nota3 350
#define silencio 100
#define silencio1 125
#define silencio3 500

int main() {
    stdio_init_all();

    gpio_set_function(LED_PIN, GPIO_FUNC_PWM);
    uint slice = pwm_gpio_to_slice_num(LED_PIN);
    uint chan  = pwm_gpio_to_channel(LED_PIN);

    pwm_set_wrap(slice, TOP);
    pwm_set_chan_level(slice, chan, 2000);
    pwm_set_enabled(slice, true);

    float f_clk = 150000000.0f;
    float div;

    while (true) {
        
        div = f_clk / (DO * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio);  

       
        div = f_clk / (DO * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);
    

        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio); 

        div = f_clk / (DO * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000);
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio); 

        
        div = f_clk / (FA * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota2);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio);  

        
        div = f_clk / (LA * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota2);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio); 
        
        
        div = f_clk / (DO * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio);  

       
        div = f_clk / (DO * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);
    

        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio); 

        div = f_clk / (DO * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000);
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio); 

        
        div = f_clk / (FA * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota2);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio);  

        
        div = f_clk / (LA * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota2);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio);

    //TERCERA ESTROFA

        div = f_clk / (FA * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota0);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio1);  

        div = f_clk / (FA * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota0);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio1);  

        div = f_clk / (MI * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota0);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio1);  

        div = f_clk / (MI * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota0);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio1);  

        div = f_clk / (RE * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota0);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio1);  

        div = f_clk / (RE * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota0);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio1);  

        div = f_clk / (DO2 * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota2);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio3);  


    //CUARTO ESTROFA

        div = f_clk / (DO * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio);  

        div = f_clk / (DO * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio);  

        div = f_clk / (DO * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio);  

        div = f_clk / (MI * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio);  

        div = f_clk / (SOL * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio3);  


        //QUINTO ESTROFA

        div = f_clk / (DO * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio);  

        div = f_clk / (DO * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio);  

        div = f_clk / (DO * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio);  

        div = f_clk / (MI * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio);  

        div = f_clk / (SOL * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio3);  

        //SEXTO ESTROFA

        div = f_clk / (SOL * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio); 

        div = f_clk / (LA * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio); 

        div = f_clk / (SOL * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio); 

        div = f_clk / (FA * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio); 

        div = f_clk / (MI * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio); 

        div = f_clk / (RE * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(silencio); 

        div = f_clk / (DO * (TOP + 1));
        pwm_set_clkdiv(slice, div);
        pwm_set_chan_level(slice, chan, 2000); 
        sleep_ms(delay_nota1);

        
        pwm_set_chan_level(slice, chan, 0); 
        sleep_ms(4000); 

        
    }
}



```


**Video**

<iframe width="560" height="315" src="https://www.youtube.com/embed/lftfwQg7x9w?si=8YOV_0s4Jcjccc1J" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

