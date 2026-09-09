# Serial Interfaces Lab – SPI

## 📌 Información de la práctica

**Materia:** Serial Interfaces
**Práctica:** Practice 2 – SPI
**Microcontrolador:** FRDM-KL25Z
**Controlador:** MAX7219
**Lenguaje:** C
**Interfaz de comunicación:** SPI

---

## 🎯 Objetivo

El objetivo de esta práctica es implementar comunicación **SPI** utilizando el controlador **MAX7219** para controlar cuatro displays de 7 segmentos mediante una tarjeta **FRDM-KL25Z**.

A partir de la comunicación SPI se desarrollaron tres etapas:

1. Control básico de cuatro displays de 7 segmentos.
2. Implementación de un contador de cuatro dígitos controlado mediante botones.
3. Desarrollo de un **temporizador programable MM:SS**, utilizando un keypad, dos botones y el LED RGB de la FRDM-KL25Z.

La práctica permite integrar SPI con diferentes periféricos del microcontrolador y desarrollar una aplicación embebida completa.

---

# 🧩 Parte 1 – Control de cuatro displays

## Descripción

En la primera parte se tomó como base el ejemplo de comunicación SPI con el MAX7219 y se modificó para trabajar con **cuatro displays de 7 segmentos** en lugar de dos.

El MAX7219 permite controlar los cuatro dígitos utilizando diferentes registros de datos. Para esta implementación se configuró la decodificación Code B para los cuatro dígitos y se estableció el límite de escaneo en el cuarto display.

La práctica solicita además comprobar que cada uno de los cuatro dígitos pueda controlarse de manera independiente.

### Configuración utilizada

| Registro MAX7219 |  Valor | Función                                         |
| ---------------- | -----: | ----------------------------------------------- |
| `DECODE`         | `0x0F` | Habilita decodificación para los cuatro dígitos |
| `SCANLIMIT`      |    `3` | Escanea los dígitos 0–3                         |
| `INTENSITY`      |    `4` | Configura el brillo                             |
| `TEST`           |    `0` | Modo normal                                     |
| `SHUTDOWN`       |    `1` | Habilita el dispositivo                         |

### Conexión SPI

| FRDM-KL25Z | Función            |
| ---------- | ------------------ |
| PTD1       | SPI SCK            |
| PTD2       | SPI MOSI           |
| PTD0       | Chip Select (`CS`) |

### Funcionamiento

La función:

```c
max7219_write(command, data);
```

se utiliza para enviar al MAX7219 el registro que se desea modificar y el dato correspondiente.

En la demostración se envían cuatro valores diferentes:

```c
max7219_write(0x01, 0);
max7219_write(0x02, 0);
max7219_write(0x03, 0);
max7219_write(0x04, 0);
```

De esta manera se verifica el control independiente de los cuatro displays.

También se incluyó el registro `TEST`, que puede utilizarse para activar todos los segmentos durante la depuración del hardware.

---

# 🔢 Parte 2 – Contador de cuatro dígitos

## Descripción

En la segunda parte se utilizó el código de la Parte 1 como base para desarrollar un **contador de cuatro dígitos** controlado mediante cuatro botones.

El contador puede trabajar en modo ascendente o descendente y permite modificar manualmente el valor cuando se encuentra pausado.

Estas funcionalidades corresponden a los requerimientos establecidos para la segunda parte de la práctica.

## Funciones principales

| Botón          | Función                 |
| -------------- | ----------------------- |
| Botón 1 – PTA1 | Play / Pause            |
| Botón 2 – PTA2 | Incremento / Decremento |
| Botón 3 – PTA4 | Ajuste manual           |
| Botón 4 – PTA5 | Reset                   |

### Estados del contador

El programa utiliza las siguientes variables para controlar su comportamiento:

```c
int current_count = 0;
int is_playing = 0;
int is_incrementing = 1;
```

* `current_count`: almacena el valor actual del contador.
* `is_playing`: determina si el contador está funcionando o pausado.
* `is_incrementing`: determina si el contador aumenta o disminuye.

### Rango del contador

El contador trabaja dentro del rango:

```text
0000 – 9999
```

Cuando se supera `9999`, el valor regresa a `0000`.

Cuando se intenta disminuir por debajo de `0000`, el valor pasa a `9999`.

```c
if (current_count > 9999) current_count = 0;
if (current_count < 0) current_count = 9999;
```

## Visualización

La función:

```c
display_number(int num);
```

se encarga de separar el número en:

* Unidades
* Decenas
* Centenas
* Millares

y posteriormente enviar cada dígito al registro correspondiente del MAX7219.

```c
max7219_write(4, unidades);
max7219_write(3, decenas);
max7219_write(2, centenas);
max7219_write(1, millares);
```

## Detección de botones

Para evitar múltiples acciones provocadas por una sola pulsación, se utilizan los valores anteriores y actuales de cada botón.

Por ejemplo:

```c
if (curr_b1 == 0 && prev_b1 == 1) {
    is_playing = !is_playing;
}
```

Esto permite detectar el cambio de estado del botón y ejecutar la acción una sola vez.

---

# ⏱️ Parte 3 – Temporizador programable

## Descripción

Para la tercera parte se seleccionó la:

> **Option B – Programmable Timer**

El objetivo es desarrollar un temporizador programable utilizando los cuatro displays de 7 segmentos, un keypad, dos botones y el LED RGB.

El temporizador utiliza el formato:

```text
MM:SS
```

donde los dos primeros dígitos representan los minutos y los dos últimos los segundos.

Por ejemplo:

```text
00:30
```

representa 30 segundos, mientras que:

```text
05:45
```

representa 5 minutos y 45 segundos.

---

## ⌨️ Configuración mediante keypad

El usuario puede introducir el tiempo utilizando un keypad matricial 4×4.

El formato de entrada es:

```text
MMSS
```

Para confirmar el valor se utiliza:

```text
#
```

Mientras que:

```text
*
```

cancela la entrada actual.

Por ejemplo:

```text
0545#
```

configura el temporizador como:

```text
05:45
```

El programa también verifica que los segundos sean válidos:

```text
00 ≤ SS ≤ 59
```

Por lo tanto:

```text
0545#   → válido
0567#   → inválido
```

Cuando se introduce un valor inválido, el programa conserva el valor previamente configurado, siguiendo el comportamiento solicitado en la práctica.

---

# 🔘 Control mediante botones

La aplicación utiliza dos botones:

| Botón                | Función                |
| -------------------- | ---------------------- |
| Push Button 1 – PTA1 | Start / Pause / Resume |
| Push Button 2 – PTA2 | Reset                  |

El comportamiento corresponde a los requerimientos de la práctica.

### Push Button 1

Permite:

```text
READY → RUNNING
RUNNING → READY
```

En otras palabras:

* Inicia el temporizador.
* Pausa el temporizador.
* Reanuda el conteo.

### Push Button 2

Regresa el temporizador al valor configurado originalmente.

Por ejemplo:

```text
Configurado: 05:00

Después de contar:
02:37

Presionar Button 2:

05:00
```

El temporizador permanece detenido después del reset hasta que se presione nuevamente el botón 1.

---

# 🔄 Máquina de estados

El temporizador se implementó mediante una máquina de estados con tres estados principales:

```c
typedef enum {
    STATE_READY,
    STATE_RUNNING,
    STATE_FINISHED
} TimerState;
```

## 🟦 STATE_READY

En este estado:

* El temporizador está detenido.
* Se muestra el tiempo configurado.
* Se pueden introducir nuevos valores mediante el keypad.
* `#` confirma la configuración.
* `*` cancela la entrada.
* El botón 1 inicia el temporizador.

La práctica define este estado como la condición en la que el temporizador está detenido y preparado para recibir una nueva configuración.

---

## 🟢 STATE_RUNNING

En este estado:

* El temporizador está contando.
* El valor disminuye una vez por segundo.
* El keypad no modifica el valor mientras está funcionando.
* El botón 1 pausa el temporizador.

La transición de minutos y segundos se realiza correctamente.

Por ejemplo:

```text
01:00
00:59
00:58
...
00:01
00:00
```

La práctica especifica que el conteo debe disminuir una vez por segundo y manejar correctamente el cambio entre minutos y segundos.

---

## 🟥 STATE_FINISHED

Cuando el temporizador llega a:

```text
00:00
```

se detiene automáticamente y cambia al estado:

```c
STATE_FINISHED
```

En este estado:

* Se mantiene `00:00` en el display.
* El LED RGB indica que el temporizador terminó.
* El botón 2 permite regresar al tiempo configurado.
* Se puede introducir un nuevo tiempo mediante el keypad.

Esto corresponde al comportamiento solicitado para el estado `FINISHED`.

---

# 💡 LED RGB

El LED RGB se utiliza para identificar visualmente el estado actual del temporizador.

La función:

```c
set_rgb_state(TimerState state);
```

selecciona el color correspondiente.

| Estado           | Color implementado |
| ---------------- | ------------------ |
| `STATE_READY`    | 🔵 Azul            |
| `STATE_RUNNING`  | 🟢 Verde           |
| `STATE_FINISHED` | 🔴 Rojo            |

La práctica requiere que los tres estados puedan distinguirse mediante diferentes colores del LED RGB. Los colores específicos pueden ser seleccionados por el equipo.

---

# ⏲️ Temporización mediante SysTick

Para generar el intervalo de un segundo se utilizó el **SysTick** del microcontrolador.

El sistema genera un tick cada:

```text
1 ms
```

mediante:

```c
void SysTick_Handler(void) {
    ms_ticks++;
}
```

Posteriormente se comprueba la diferencia entre el tiempo actual y el último segundo registrado:

```c
if (ms_ticks - last_second_tick >= 1000)
```

Esto permite realizar el decremento una vez cada 1000 ms.

Este método evita utilizar retardos largos que bloqueen la lectura del keypad o de los botones, cumpliendo con el requisito de utilizar un mecanismo basado en timer/SysTick.

---

# 🔌 Periféricos utilizados

La implementación final integra los siguientes periféricos:

```text
                    ┌─────────────────┐
                    │   FRDM-KL25Z    │
                    │                 │
                    │     SPI0        │
                    └────────┬────────┘
                             │
                             ▼
                       ┌───────────┐
                       │ MAX7219   │
                       └─────┬─────┘
                             │
                             ▼
                     ┌───────────────┐
                     │  7-Segmentos  │
                     │    MM : SS    │
                     └───────────────┘

       ┌────────────┐
       │   Keypad   │
       │    4 × 4   │
       └─────┬──────┘
             │
             ▼
        FRDM-KL25Z

       ┌────────────┐
       │ Push Btn 1 │──── Start/Pause/Resume
       └────────────┘

       ┌────────────┐
       │ Push Btn 2 │──── Reset
       └────────────┘

       ┌────────────┐
       │  RGB LED   │──── Estado del temporizador
       └────────────┘
```

---

# 📁 Estructura del proyecto

Una posible organización del repositorio es:

```text
Serial-Interfaces-Lab-SPI/
│
├── README.md
│
├── Part1/
│   └── main.c
│
├── Part2/
│   └── main.c
│
└── Part3/
    └── main.c
```

### `Part1`

Contiene la implementación inicial del MAX7219 mediante SPI y la demostración de los cuatro displays.

### `Part2`

Contiene el contador de cuatro dígitos con control mediante botones.

### `Part3`

Contiene la implementación final del temporizador programable con:

* SPI
* MAX7219
* 4 displays de 7 segmentos
* Keypad 4×4
* 2 push buttons
* LED RGB
* SysTick

---

# 🛠️ Tecnologías y componentes

### Hardware

* FRDM-KL25Z
* MAX7219
* Display de 7 segmentos de 4 dígitos
* Keypad matricial 4×4
* 2 push buttons
* LED RGB

### Software

* C
* CMSIS / `MKL25Z4.H`
* SPI0
* GPIO
* SysTick

---

# 📚 Conceptos aplicados

Durante la práctica se trabajaron los siguientes conceptos:

* Comunicación serial SPI.
* Configuración del periférico SPI0.
* Comunicación entre microcontrolador y MAX7219.
* Control de displays de 7 segmentos.
* Manejo de registros del MAX7219.
* Configuración de GPIO.
* Lectura de push buttons.
* Detección de flancos para evitar múltiples acciones.
* Escaneo de keypad matricial.
* Máquina de estados.
* Temporización mediante SysTick.
* Manejo de interrupciones.
* Control de LED RGB.
* Conversión y separación de números en dígitos.
* Implementación de un temporizador no bloqueante.

---

# ▶️ Funcionamiento esperado

El flujo principal de la aplicación final es:

```text
                 INICIO
                    │
                    ▼
          Inicialización de SPI,
       MAX7219, SysTick, botones,
          keypad y LED RGB
                    │
                    ▼
              STATE_READY
                    │
          ┌─────────┴─────────┐
          │                   │
       Keypad              Button 1
          │                   │
      MMSS + #               ▼
          │             STATE_RUNNING
          ▼                   │
     Nuevo tiempo             │
                              │
                       Button 1 = pausa
                              │
                              ▼
                         STATE_READY
                              │
                              │
                       Cuenta regresiva
                              │
                              ▼
                         00:00
                              │
                              ▼
                       STATE_FINISHED
                         │           │
                    Button 2       Keypad
                         │           │
                         ▼           ▼
                      Reset      Nuevo tiempo
                         │
                         └──────► READY
```

---

# ✅ Resultados

Con la implementación desarrollada se logra:

* Controlar cuatro displays de 7 segmentos mediante SPI.
* Mostrar diferentes valores en cada display.
* Implementar un contador ascendente y descendente.
* Controlar el contador mediante botones.
* Realizar ajustes manuales mientras el contador está pausado.
* Implementar un temporizador configurable en formato `MM:SS`.
* Introducir tiempos mediante un keypad.
* Validar que los segundos estén dentro del rango `00–59`.
* Iniciar, pausar y reanudar el temporizador.
* Restaurar el tiempo configurado mediante el segundo botón.
* Detener automáticamente el temporizador al llegar a `00:00`.
* Utilizar el LED RGB para representar los estados `READY`, `RUNNING` y `FINISHED`.
* Utilizar SysTick para generar una base de tiempo de 1 segundo sin bloquear la interacción del usuario.

---

# 👩‍💻 Equipo
### 👥 Integrantes

* **Vanessa Sarahí Salazar Ibarra A01646141**
* **Ana Cristina Chavez Acosta A01742237**
* **Angeles Araiza García A00574806**
