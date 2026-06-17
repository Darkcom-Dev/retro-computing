# LA CPU

La Mk3 utiliza un microprocesador Z80A, funcionando a una frecuencia de reloj de 3.58 MHz.

En esta sección se describen las implementaciones de las características del Z80 tal como pertenecen a la Mk3.

*   Reinicio (Reset)
*   Sistema de Interrupciones 
*   NMI
*   INT
*   Organización de la RAM de Video
*   Color 
*   Sistema de Fondo (Background) 
*   Bits de Inhibición de Desplazamiento 


### RESET (REINICIO)

El Z80A ejecuta un ciclo de RESET cuando la unidad base se enciende. Este es el único mecanismo de reinicio del Z80A implementado.

El botón momentáneo de RESET en la consola no controla el reinicio del Z80A; está conectado a un puerto de entrada para ser consultado (polling) mediante software.

### SISTEMA DE INTERRUPCIONES

La Mk3 implementa dos de las interrupciones del Z80: la Interrupción No Enmascarable (NMI) y la Interrupción Enmascarable (INT).

### NMI (Interrupción No Enmascarable)

El pin NMI está conectado al botón "PAUSE" (Pausa) de la consola. Al presionar este botón, el Z80 ejecuta una instrucción de Reinicio (Restart) en la ubicación $66. Esto actúa como una llamada simple a una subrutina: el Contador de Programa (PC) se coloca en la pila (stack) y se realiza un salto a $0066.

Esta interrupción es "disparada por flanco" (edge-triggered), lo que significa que si el botón PAUSE se mantiene presionado, solo se recibirá una única interrupción.

**NOTA:** La última instrucción de su rutina NMI debe ser "RETN", Retorno de Interrupción No Enmascarable.

El botón PAUSE es para el uso exclusivo de su programa; usted debe configurar la transferencia de control en $0066 hacia su rutina de PAUSA.

La rutina de PAUSA más común alternará un indicador (flag) de "pausa" por software y apagará los sonidos si este indicador acaba de activarse. La rutina INT, que se activa periódicamente, verifica el estado del indicador de pausa y no realiza ningún procesamiento si está activado (hace un bucle sobre el indicador de pausa).

Este método permite que la función de pausa se implemente sin desactivar y activar las interrupciones (con una instrucción DI y luego una EI). Esta combinación DI-EI puede causar una interrupción espuria en el Z80.

La NMI no puede ser inhibida.

---

## INT (Interrupción Enmascarable)

Esta interrupción se habilita con una instrucción "EI" y se deshabilita con una instrucción "DI".

El hardware de la Mk3 solo soporta la interrupción de "modo 1". La ROM de arranque ejecuta la instrucción "IM 1" al encenderse.

En el modo 1, la activación del pin INT del Z80A hace que se ejecute una instrucción de Reinicio (Restart) en la ubicación $38 si las interrupciones están habilitadas.

Esta interrupción puede ser activada por dos eventos en el chip VDP:

1. Intervalo de Borrado Vertical (Vertical Blanking Interval).
2. Contador de Línea Horizontal (Horizontal Line Counter).

Aquí hay un programa esqueleto para una rutina de respuesta a interrupciones:

```assembly
(en $0038)
    JP INT
    .
    .
INT:
    PUSH AF
    IN A,($BF)
    .
    .
    .
    POP AF
    EI
    RET
```

El primer paso es guardar los registros de trabajo y los indicadores (flags). Se deben añadir otras instrucciones "PUSH" para cualquier registro que utilice su rutina de interrupción.

Leer el puerto de E/S en $BF hace dos cosas. Primero, limpia la línea de solicitud de interrupción del chip VDP. Segundo, proporciona información del VDP de la siguiente manera:

*   **bit 7** --- 1: Interrupción VBLANK, 0: Interrupción H-Line [si está habilitada].
*   **bit 6** --- 1: 9 sprites en una línea de barrido (raster line).
*   **bit 5** --- 1: Colisión de Sprites.
*   **bits 4-0** (sin importancia).

Las interrupciones VBLANK y H-Line del VDP se habilitan con los bits IE e IE1 en los registros 0 y 1 del VDP, como se muestra en la siguiente tabla:

### Bits de Habilitación de Interrupción

| IE (R1 bit 5) | IE1 (R0 bit 4) | Fuente de Interrupción |
| :--- | :--- | :--- |
| 0 | 1 | Solo interrupción H-Line. |
| 1 | 1 | Tanto H-Line como VBLANK. |

Cuando se acepta una INT, el sistema de interrupciones se apaga exactamente como si se hubiera ejecutado una instrucción "DI". Por lo tanto, el código para salir de una rutina de interrupción debe restaurar los registros, ejecutar una instrucción "EI" y retornar.
