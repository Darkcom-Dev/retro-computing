## EL PROCESADOR DE VISUALIZACIÓN DE VIDEO (VDP)

La Mk3 utiliza un chip controlador de video avanzado llamado VDP (Procesador de Visualización de Video). El VDP es un chip personalizado diseñado por Sega cuya arquitectura se asemeja a un TMS9918A mejorado.

La resolución de pantalla del VDP es de 256 píxeles horizontales y 192 píxeles verticales. Un píxel puede mostrarse como uno de dieciséis colores, seleccionados de una paleta de 64 colores diferentes.

Se dedican 16 Kilobytes de RAM al sistema de video. Esta RAM (llamada RAM de video o VRAM) está conectada directamente al chip VDP y no aparece en el espacio de memoria del Z80. El Z80 lee y escribe en la RAM de video a través de los registros del VDP.

El VDP implementa dos sistemas gráficos independientes: un sistema de fondo (background) y un sistema de sprites.

### ORGANIZACIÓN DE LA RAM DE VIDEO

Los 16 Kilobytes de RAM de video se dividen en tres secciones:

1.  **Un mapa de pantalla de 1792 bytes.** Este mapa determina la ubicación de las celdas de caracteres (a veces conocidas como "tiles" o mosaicos) en la cuadrícula de 32 por 24 de la pantalla de fondo.
2.  **Una tabla de atributos de sprites de 256 bytes.** Esta tabla establece las coordenadas X-Y y el número de carácter de hasta 64 objetos móviles, o sprites.
3.  **Un generador de caracteres de 14,336 bytes.** Estos son patrones de caracteres de 8 por 8 píxeles para el fondo, y/o patrones de caracteres de 8(h) por 8(v) o 8(h) por 16(v) píxeles para los sprites.

32 bytes definen un único carácter de 8 por 8 píxeles. Por lo tanto, la parte del generador de caracteres de la RAM de video es capaz de definir hasta 448 caracteres.

La ubicación de las tres secciones de la RAM de video se controla mediante registros en el VDP. Como veremos, usted tiene cierta capacidad de elección sobre dónde aparecen en el mapa de memoria de los 16K de la RAM de video.

### COLOR

El atributo de color se transporta en los patrones de caracteres en la sección del generador de caracteres de la RAM de video. Cada píxel en el patrón de caracteres contiene 4 bits de información. Por lo tanto, hay una opción de dieciséis colores disponible para cada píxel.

Los dieciséis colores disponibles no son fijos. Se almacenan en una RAM de Color dentro del chip VDP. La RAM de Color está organizada como 32 valores de 6 bits. Los seis bits proporcionan cuatro niveles (2 bits) de rojo, verde y azul.

Tenga en cuenta que 32 valores de color representan el doble de la capacidad de direccionamiento de los píxeles de 4 bits. Otros bits de control del VDP permiten la selección de la tabla de 16 valores de color ya sea desde la primera mitad o desde la segunda mitad de la RAM de Color.

---

## SISTEMA DE FONDO (BACKGROUND)

El sistema de fondo se compone de "caracteres" que tienen ocho píxeles de alto y ocho píxeles de ancho. La pantalla está organizada en 768 caracteres visibles: 32 horizontales por 24 verticales.

Existen 4 filas de caracteres adicionales debajo de las 24 filas verticales visibles. Estas filas adicionales son útiles para desplazar (scrolling) nueva información de fondo hacia arriba en la pantalla.

2048 bytes de la RAM de video funcionan como el mapa de pantalla. Este mapa define las posiciones en pantalla de 896 caracteres (768 visibles).

En cada posición de carácter, hay una palabra de 16 bits (2 bytes) que describe lo siguiente:

1.  Cuál de los 512 caracteres mostrar en la posición del carácter (9 bits).
2.  Si se debe invertir o no el carácter horizontal o verticalmente (2 bits).
3.  Cuál de los dos conjuntos de 16 colores usar para el carácter (1 bit).
4.  Si los sprites ocultan el fondo, o viceversa (1 bit).
5.  Tres bits no asignados, útiles para indicadores (flags) de software (3 bits).

El mapa de pantalla visible ocupa 1536 bytes de la RAM de video (768 caracteres, dos bytes por carácter).

Las cuatro filas no visibles de caracteres debajo de la pantalla visible ocupan 256 bytes (128 caracteres, dos bytes por carácter).

Esto deja 256 bytes de memoria sin usar en la dirección más alta de los 2048 bytes de la RAM del mapa de pantalla. Estos 256 bytes se utilizan normalmente para almacenar la Tabla de Atributos de Sprites.

Esta memoria también podría usarse para almacenar ocho patrones de caracteres, si la Tabla de Atributos de Sprites se encuentra en otro lugar. Si el mapa de pantalla se coloca en $3800 (el caso habitual), los números de carácter para los 256 bytes no utilizados de la RAM del mapa de pantalla son desde $1F0 hasta $1F7.

### BITS DE INHIBICIÓN DE DESPLAZAMIENTO (SCROLL INHIBIT)

La pantalla de fondo se puede desplazar en incrementos de un píxel tanto horizontal como verticalmente. Dos bits de "inhibición de desplazamiento" le permiten inhibir el desplazamiento en dos áreas de la pantalla:

*   Si **HSI** (Inhibición de Desplazamiento Horizontal) se establece en 1, la franja horizontal de 2 caracteres de alto en la parte superior de la pantalla no se desplaza.
*   Si **VSI** (Inhibición de Desplazamiento Vertical) se establece en 1, la banda vertical de 8 caracteres en el borde derecho de la pantalla no se desplaza.

Esta característica simplifica la colocación de información de puntuación en la parte superior o en el lado derecho de la pantalla. Cuando el campo de juego de la pantalla de fondo se desplaza, las puntuaciones permanecen en su lugar si se han activado los bits de inhibición de desplazamiento.

---

## SPRITES

Un sprite es un objeto fácilmente movible. El VDP proporciona 64 sprites independientes. Un bit de control regula el tamaño de todos los sprites: 0 para un sprite de 8H por 8V, 1 para un sprite de 8H por 16V.

Cada sprite utiliza una entrada de tres bytes en una Tabla de Atributos de Sprites (en la RAM de video) para establecer su posición vertical y horizontal, y para seleccionar uno de los 256 caracteres que se mostrarán para el sprite.

Un bit de control del VDP (Sprite Shift, bit 3 de R0) desplaza todos los sprites 8 píxeles a la izquierda. Esto permite que los sprites se desplacen suavemente fuera del borde izquierdo de la pantalla.

Los caracteres de sprite y de fondo se extraen del mismo "pool" de patrones de caracteres en la RAM de video. Por lo tanto, los sprites tienen las mismas capacidades de color que los caracteres de fondo: 16 colores a la vez de una elección de 64 colores.

Los colores de los sprites siempre se toman del segundo grupo de 16 colores en la RAM de color.

Los sprites no se desplazan cuando se desplaza la escena de fondo.

Los sprites pueden colocarse encima o debajo de otros sprites. Los sprites también pueden aparecer encima o debajo de los caracteres en la escena de fondo.

Hasta ocho sprites pueden ocupar una sola línea de barrido (raster line) horizontal. Se proporciona un bit en el registro de estado del VDP para alertarle cuando nueve o más sprites están posicionados en la misma línea. Una línea de barrido horizontal que contiene nueve o más sprites no se muestra correctamente.

Otro bit del registro de estado del VDP indica que dos patrones de sprites se tocaron (colisionaron).

### TABLA DE ATRIBUTOS DE SPRITES (SAT)

Una sección de 256 bytes de la RAM de video funciona como la Tabla de Atributos de Sprites. El registro R5 del VDP se establece generalmente en $FF para posicionar la Tabla de Atributos de Sprites en $3F00, los últimos 256 bytes de la RAM de video.

La Tabla de Atributos de Sprites se organiza como se muestra en la tabla de la página siguiente.
