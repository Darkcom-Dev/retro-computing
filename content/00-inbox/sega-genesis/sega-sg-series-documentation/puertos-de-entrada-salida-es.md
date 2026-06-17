## PUERTOS DE ENTRADA/SALIDA (E/S)

La Mk3 utiliza ocho puertos de E/S:

| Puerto | Direc. | Función |
| :--- | :--- | :--- |
| $3E | salida | Habilitaciones de Memoria |
| $3F | salida | Control de puertos de Joystick |
| $7E | entrada | Punto de la Pistola (vertical) |
| $7F | entrada | Punto de la pistola (horizontal) |
| | salida | Generador de Sonido Programable (PSG) |
| $BE | e/s | Registro de Datos del VDP |
| $BF | e/s | Registro de Comando/Estado del VDP |
| $DC | entrada | Puerto de Joystick |
| $DD | entrada | Puerto de Joystick |

El puerto **$3E** se describe en la sección de GESTIÓN DE MEMORIA.
El puerto **$7F** se describe en la sección del PSG.
Los puertos **$BE** y **$BF** se describen en la sección del VDP.

Los puertos restantes se encargan de los controles conectados y se describen en esta sección.

Los puertos **$DC-$DD** están conectados a los dos conectores de 9 pines en la parte frontal de la consola de juegos. Aunque estos puertos se denominan "puertos de joystick", en realidad hay tres dispositivos que se pueden conectar a los conectores frontales:

1.  Los controles de joystick, que incluyen una palanca de mando y dos botones pulsadores.
2.  La pistola (gun).
3.  Un "Sports Pad" que consiste en un trackball y dos botones pulsadores.

### MOC
*   Asignaciones de Bits de los Puertos de Entrada 
*   Cómo Funciona la Pistola 
*   Cómo Funciona el Trackball

---

## ASIGNACIONES DE BITS DE LOS PUERTOS DE ENTRADA

Los bits de los puertos de entrada **$DC** y **$DD** se utilizan para el conjunto del joystick, la pistola y el Sports Pad (Trackball). Estos dos puertos funcionan en conjunto con el puerto **$3F**, que establece la dirección de cuatro de los bits y proporciona dos bits de estrobo (strobe) para el Sports Pad. La siguiente tabla muestra cómo se interpretan los tres puertos para las opciones de joystick y pistola. Todos los bits mostrados son entradas.

| Puerto | Bit | Joystick | Pistola |
| :--- | :--- | :--- | :--- |
| $DC | 0 | P1-arriba | --- |
| $DC | 1 | P1-abajo | --- |
| $DC | 2 | P1-izquierda | --- |
| $DC | 3 | P1-derecha | --- |
| $DC | 4 | P1-BOTÓN1 | Gatillo P1 |
| $DC | 5 | P1-BOTÓN2 | --- |
| $DC | 6 | P2-arriba | --- |
| $DC | 7 | P2-abajo | --- |
| | | | |
| $DD | 0 | P2-izquierda | --- |
| $DD | 1 | P2-derecha | --- |
| $DD | 2 | P2-BOTÓN1 | Gatillo P2 |
| $DD | 3 | P1-BOTÓN2 | --- |
| $DD | 4 | RESET* | RESET* |
| $DD | 5 | --- | --- |
| $DD | 6 | --- | Pulso de Luz P1 |
| $DD | 7 | --- | Pulso de Luz P2 |
| | | | |
| $3F | 0 | 1 | 1 |
| $3F | 1 | 1 | Habilitar Captura P1 |
| $3F | 2 | 1 | 1 |
| $3F | 3 | 1 | Habilitar Captura P2 |
| $3F | 4 | 1 | 1 |
| $3F | 5 | 1 | 1 |
| $3F | 6 | 1 | 1 |
| $3F | 7 | 1 | 1 |

---

## CÓMO FUNCIONA LA PISTOLA (GUN)

La pistola contiene una lente y una fotocélula que "ve" un punto en la pantalla de la TV. Este punto es circular y aumenta de diámetro a medida que aumenta la distancia de la pistola a la pantalla.

Cuando el haz de la TV llega al área a la que apunta la pistola, ocurren dos cosas:

1.  Se genera un pulso negativo de aproximadamente 20-30 microsegundos en el puerto **$DD bit 6** (Jugador 1) o puerto **$DD bit 7** (Jugador 2). Este pulso es lo suficientemente ancho como para permitir que un bucle de software lo reconozca. El ancho de este pulso variará con la configuración del juego y el tipo de TV.
2.  La posición del haz se captura (latched) en los registros horizontal y vertical, que se leen en los puertos de entrada **$7E** (vertical) y **$7F** (horizontal).

La posición del haz puede capturarse ya sea desde una pistola conectada al conector del jugador 1, una pistola conectada al conector del jugador 2, o ambas. [NOTA: El conector del jugador 1 es el conector de la izquierda cuando se mira la consola desde el frente].

Dos bits en el puerto de salida **$3F** determinan qué pistola captura la posición del haz. Estos se muestran como "Habilitar Captura P1" (P1 Latch Enable) y "Habilitar Captura P2" (P2 Latch Enable) en la tabla anterior.

La rutina de lectura de la pistola suele realizar los siguientes pasos:

1.  Se comprueba el interruptor del gatillo. Normalmente, la pistola no se comprueba hasta que alguien aprieta el gatillo. (Sin embargo, los datos están continuamente disponibles desde la pistola).
2.  Cuando se aprieta el gatillo, se espera al siguiente **VBLANK**.
3.  En el VBLANK, se pone una pantalla totalmente blanca (todos los bytes de color son **$3F**). Esto da suficiente intensidad de luz en la pantalla para que la pistola lea cualquier posición.
4.  Durante el siguiente barrido, se comprueba continuamente el bit de pulso de luz para detectar una transición de alto a bajo. (El Pulso de Luz P1 está en el puerto de entrada $DD bit 6; el Pulso de Luz P2 está en el puerto de entrada $DD bit 7).
5.  Cuando se detecta el pulso, se lee el Registro de Posición Horizontal en **$7F** y el Registro de Posición Vertical en **$7E**.
6.  Se espera a una transición de bajo a alto en el bit de Pulso de Luz. Luego se repite el paso 5 para todas las líneas horizontales activas.
7.  Al terminar con un barrido, se restaura la pantalla a los valores de color normales.

La pistola responde a un punto circular en la TV, no a un solo píxel. A medida que la fotocélula de la pistola "ve" el punto circular, se producirán pulsos negativos repetidos en el bit de Pulso de Luz durante varias líneas de barrido horizontal consecutivas.

Los valores horizontales del círculo detectado, a medida que el registro de posición vertical aumenta, disminuirán, aumentarán y luego se detendrán cuando la lente de la pistola lea el borde izquierdo del círculo escaneado.

Por esta razón, es aconsejable realizar algún tipo de promedio y extrapolación para llegar a una posición central del círculo detectado.

---

## CÓMO FUNCIONA EL TRACKBALL

La interfaz del trackball consta de seis bits de entrada y un bit de salida.

| Trackball | dir | Jugador 1 | Jugador 2 |
| :--- | :--- | :--- | :--- |
| b0 | en | $DC bit 0 | $DC bit 6 |
| b1 | en | $DC bit 1 | $DC bit 7 |
| b2 | en | $DC bit 2 | $DD bit 0 |
| b3 | en | $DC bit 3 | $DD bit 1 |
| S1 | en | $DC bit 4 | $DD bit 2 |
| S2 | en | $DC bit 5 | $DD bit 3 |
| ESTROBO | sal | $3F bit 5 | $3F bit 7 |

Antes de usar el trackball, se deben ejecutar las siguientes dos instrucciones para establecer las direcciones del bit de ESTROBO (STROBE) a salida e inicializar las señales de estrobo a 1.

**NOTA:** estas instrucciones se ejecutan en la inicialización al encender el equipo.

```assembly
LD A,10100101b ; b1,b3 establecen dirección de estrobo a salida
OUT (03FH),A   ; b5,b7 son los valores reales de los bits de estrobo
```

La señal de estrobo se utiliza entonces para introducir mediante reloj cuatro nibbles (semibytes) de 4 bits b3-b0, que ocurren en el siguiente orden:

| Nibble # | Datos | Cuando el ESTROBO está en: |
| :--- | :--- | :--- |
| 1 | X7 X6 X5 X4 | BAJO (LO) |
| 2 | X3 X2 X1 X0 | ALTO (HI) |
| 3 | Y7 Y6 Y5 Y4 | BAJO (LO) |
| 4 | Y3 Y2 Y1 Y0 | ALTO (HI) |

Las posiciones X,Y del trackball se representan como X7-X0 y Y7-Y0. Estos son contadores de 8 bits que se reinician (wrap around) de **$FF** a **$00**.

A medida que la bola gira hacia la derecha, el valor de X aumenta; a medida que gira hacia la izquierda, disminuye. A medida que la bola gira hacia arriba, los valores de Y aumentan; a medida que gira hacia abajo, disminuyen.

La sincronización de las transiciones del ESTROBO es importante. Aquí está la secuencia requerida:

**Secuencia del Programa para Leer el Trackball**
----------------------------------
[Comenzar con ESTROBO en ALTO].
1.  ESTROBO en BAJO.
2.  Esperar 80 microsegundos.
3.  Leer el primer nibble.
4.  ESTROBO en ALTO.
5.  Esperar 40 microsegundos.
6.  Leer el segundo nibble.
7.  ESTROBO en BAJO.
8.  Esperar 40 microsegundos.
9.  Leer el tercer nibble.
10. ESTROBO en ALTO.
11. Esperar 40 microsegundos.
12. Leer el cuarto nibble.
13. (terminado - el ESTROBO queda en ALTO).

El Apéndice D muestra código funcional para leer el trackball en cada interrupción.
