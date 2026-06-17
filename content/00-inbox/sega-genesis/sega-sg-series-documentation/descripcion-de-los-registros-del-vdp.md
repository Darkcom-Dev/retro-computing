## DESCRIPCIÓN DE LOS REGISTROS DEL VDP

### MOC
*   Registro 0 - Control del VDP
    *   Registro 1 - Control del VDP
    *   Registro 2 - Dirección Base del Mapa de Pantalla
    *   Registro 3
    *   Registro 4
    *   Registro 5 - Dirección Base de la Tabla de Atributos de Sprites (SAT)
    *   Registro 6 - Dirección Base de Patrones de Sprites 
    *   Registro 7 - Color del Borde 
    *   Registro 8 - Desplazamiento Horizontal
    *   Registro 9 - Desplazamiento Vertical
    *   Registro 10 - Interrupción de Línea de Barrido

### Registro 0
### Registro 1

R0 y R1 configuran el modo de visualización del VDP y el modo de interrupción.

**R0: b7 VSI** - Inhibición de Desplazamiento Vertical (8 columnas de carac. de la derecha).
*   0: Desplazamiento junto con el fondo.
*   1: Fijo (sin desplazamiento).

**b6 HSI** - Inhibición de Desplazamiento Horizontal (2 filas de carac. superiores).
*   0: Desplazamiento junto con el fondo.
*   1: Fijo (sin desplazamiento).

**b5 LCB** - Columna Izquierda en Blanco (Left Column Blank).
*   0: Visualización normal.
*   1: Columna izquierda de caracteres en blanco (color del borde).

**b4 IE1** - Habilitación de Interrupción de Línea (H-Line).

**b3 SS** - Desplazamiento de Sprites (Sprite Shift).
*   0: Posición normal de los sprites.
*   1: Desplaza todos los sprites 8 píxeles a la izquierda.

**b2 1** - [Establezca siempre esto en 1].
**b1 1** - [Establezca siempre esto en 1].
**b0 ES** - Sincronización Externa. [Establezca siempre esto en 0].

**R1: b7 1** - [Establezca siempre esto en 1].

**b6 DIS** - Pantalla Encendida (Display ON).
*   0: Pantalla completa en blanco.
*   1: Video normal.

**b5 IE** - Habilitación de Interrupción (V-Line/VBLANK).

**b4 0** - [Establezca siempre esto en 0].
**b3 0** - [Establezca siempre esto en 0].
**b2 0** - [Establezca siempre esto en 0].

**b1 SZ** - Tamaño de los Sprites (Sprite Size).
*   0: Todos los sprites son de 8 por 8.
*   1: Todos los sprites son de 8 por 16.

**b0 0** - [Establezca siempre esto en 0].

El campo de control de interrupciones de dos bits se define de la siguiente manera:

| IE | IE1 | Interrupción Habilitada |
| :--- | :--- | :--- |
| 0 | 1 | Solo Línea de Barrido (Raster Line). |
| 1 | 1 | VBLANK y Línea de Barrido. |

---

### Registro 2

R2 establece la dirección base para el mapa de pantalla. Ubica el mapa de pantalla de 2048 bytes en una de las ocho direcciones iniciales, en límites de 2048 bytes, según la siguiente tabla:

| Valor R2 | Dirección Base del Mapa de Pantalla |
| :--- | :--- |
| $FF | $3800 <--- Configuración Normal |
| $FD | $3000 |
| $FB | $2800 |
| $F9 | $2000 |
| $F7 | $1800 |
| $F5 | $1000 |
| $F3 | $0800 |
| $F1 | $0000 |

La configuración normal para R2 es $FF, que posiciona la dirección base en $3800, la sección de 2 KBytes con las direcciones más altas de los 16 KBytes de la RAM de Video.

### Registro 3
R3 siempre debe establecerse en $FF.

### Registro 4
R4 siempre debe establecerse en $FF.

### Registro 5

R5 controla la dirección base para la Tabla de Atributos de Sprites. Esta tabla de 256 bytes se puede posicionar en una de las 64 direcciones iniciales en la RAM de Video.

La dirección base se formatea en R5 de la siguiente manera:
`R5: | 1 | A13 | A12 | A11 | A10 | A9 | A8 | 1 |`

La configuración normal para R5 es $FF, que posiciona la Tabla de Atributos de Sprites en $3F00. Cuando R2 es $FF, colocando el mapa de pantalla en $3800, y R5 es $FF, los 256 bytes superiores no utilizados de la memoria del mapa de pantalla son ocupados por la Tabla de Atributos de Sprites de 256 bytes.

---

### Registro 6

R6 establece la dirección base para los patrones de sprites. Solo dos valores son válidos para R6:

| Valor R6 | Patrones de Sprites |
| :--- | :--- |
| $FB | Primeros 8K de la RAM de Video <--- Valor Normal |
| $FF | Segundos 8K de la RAM de Video |

La ubicación preferida para los patrones de sprites es en los primeros 8K de la RAM de video, ya que los 8192 bytes completos están disponibles para patrones. En los segundos 8K, se pierden 2048 bytes por el mapa de pantalla y la Tabla de Atributos de Sprites. Por lo tanto, se pueden almacenar 256 patrones de sprites en los primeros 8K, pero solo 192 patrones de sprites en los segundos 8K.

### Registro 7

R7 establece el color del borde. El color del borde se toma del segundo banco de colores en la RAM de Color del VDP.

`R7: | 1 | 1 | 1 | 1 | C3 | C2 | C1 | C0 |`

### Registro 8

R8 establece el valor de desplazamiento (scroll) horizontal para la escena de fondo. Un valor de $00 no produce desplazamiento. Un valor de $01 desplaza la escena de fondo un píxel a la izquierda y mueve la columna de píxeles situada más a la izquierda a la posición de la columna situada más a la derecha (el desplazamiento es "cíclico" o "wrap around").

Valores más altos de R8 "rotan" la pantalla horizontalmente hasta un máximo de 255 píxeles para un valor de $FF. Tenga en cuenta que el desplazamiento horizontal se asemeja a un cilindro vertical en rotación, con la información gráfica desapareciendo por el lado izquierdo de la pantalla y reapareciendo por el lado derecho.

### Registro 9

R9 establece el valor de desplazamiento (scroll) vertical para la escena de fondo. Un valor de $00 no produce desplazamiento. Un valor de $01 desplaza la escena de fondo un píxel hacia arriba. El desplazamiento vertical se asemeja a un cilindro horizontal en rotación, pero el punto de "retorno" (wrap around) no está en el borde de la pantalla, como ocurre con el desplazamiento horizontal. Más bien, está en la fila de caracteres número 28. (Recuerde que la pantalla está organizada en 28 filas de caracteres, con las 24 superiores mostradas en pantalla).

El Registro 9 se bloquea (latched) durante el borrado vertical.

---

### Registro 10

R10 controla la Interrupción de Línea de Barrido (Raster Line Interrupt).

El haz de la TV barre líneas horizontales (llamadas líneas de barrido o raster lines) de izquierda a derecha, moviéndose de la parte superior a la inferior de la pantalla. La imagen visible contiene 192 de estas líneas de barrido.

Al final de cada línea de barrido hay un breve intervalo de borrado (HBLANK) en el que el haz de la TV se apaga mientras el haz retrocede del lado derecho al lado izquierdo de la pantalla. Este intervalo HBLANK dura aproximadamente 10 microsegundos.

Es deseable recibir una interrupción justo después de que se haya mostrado una línea de barrido seleccionada. Por ejemplo, es posible que desee cambiar un valor de color después de que el haz haya barrido la mitad superior de la pantalla. En este caso, querría saber cuándo ha finalizado la línea de barrido número 95.

R10 le permite hacer esto. Si carga R10 con 94, se generará una Solicitud de Interrupción de Línea de Barrido cada vez que la línea horizontal número 95 finalice su barrido.

El valor cargado en R10 debe ser uno menos que la línea de barrido que desea que "dispare" la interrupción.

La Interrupción de Línea de Barrido continúa generando solicitudes de interrupción a medida que el haz barre la pantalla. La siguiente tabla muestra las configuraciones de R10 y las líneas de barrido que generan solicitudes de interrupción en el momento del HBLANK:

| Valor R10 | Solicitudes de Interrupción en estos tiempos HBLANK |
| :--- | :--- |
| $C0-$FF | Ninguna |
| $00 | 1, 2, 3, 4, 5, ... 191 |
| $01 | 2, 4, 6, 8, 10, ... 190 |
| $02 | 3, 6, 9, 12, ... 189 |
| $03 | 4, 8, 12, 16, ... 188 |
| (etc) | (etc) |

Hay dos casos especiales: $FF desactiva las solicitudes de interrupción, y $00 genera una solicitud de interrupción de Línea de Barrido para cada línea horizontal.

Hay un retraso de una interrupción entre la carga de R10 y el momento en que el valor surte efecto. Por ejemplo, si configura R10 para la línea 20 y responde a la interrupción restableciendo R10 a 150, la siguiente interrupción de Línea de Barrido ocurrirá en la 20, y todas las subsiguientes ocurrirán en la 150.

Los registros del VDP se inicializan al encenderse con los siguientes valores:

*   **R0** - $36 ; modo
*   **R1** - $A0 ; modo
*   **R2** - $FF ; Base de la Tabla del Mapa de Pantalla
*   **R3** - $FF ; (Siempre $FF)
*   **R4** - $FF ; (Siempre $FF)
*   **R5** - $FF ; Base de la Tabla de Sprites
*   **R6** - $FB ; Base de la Tabla de Patrones de Sprites
*   **R7** - $00 ; Color de borde #0
*   **R8** - $00 ; Desplazamiento-H
*   **R9** - $00 ; Desplazamiento-V
*   **R10** - $FF ; Interrupción de línea-H ($FF=APAGADO)
