## ORGANIZACIÓN DE LA TABLA DE ATRIBUTOS DE SPRITES

| Dirección | Atributo |
| :--- | :--- |
| 3F00 | vpos #0 |
| 3F01 | vpos #1 |
| 3F02 | vpos #2 |
| 3F03 | vpos #3 |
| 3F04 | vpos #4 |
| 3F05 | vpos #5 |
| 3F06 | vpos #6 |
| 3F07 | $D0 (terminador) |
| . | . |
| 3F3E | |
| 3F3F | vpos #63 |
| 3F40 | 64 bytes sin usar. [Se pueden poner dos |
| . | patrones de caracteres de 32 bytes aquí |
| . | y direccionarlos como carac. $1FA y $1FB]. |
| . | |
| 3F7F | |
| 3F80 | hpos #0 |
| 3F81 | código de carac. #0 |
| 3F82 | hpos #1 |
| 3F83 | código de carac. #1 |
| 3F84 | hpos #2 |
| 3F85 | código de carac. #2 |
| 3F86 | |
| 3F87 | |
| . | . |
| 3FFE | hpos #63 |
| 3FFF | código de carac. #63 |

El código especial **$D0** se coloca como un código de posición vertical para indicar al hardware de sprites que deje de buscar en la lista. Todos los sprites que aparezcan después de la entrada $D0 quedan deshabilitados.

Cuando dos sprites se superponen, el sprite con el número más alto se mostrará encima. Por lo tanto, la prioridad de superposición se establece por las posiciones de los sprites en la tabla.

Los bytes de posición horizontal ubican la esquina superior izquierda del sprite en una de las 255 coordenadas horizontales de la pantalla.

Un **hpos=0** coloca el sprite en la columna izquierda de la pantalla (los 8 píxeles de la izquierda). Un **pos=255** coloca el sprite en la última columna de píxeles; solo se ven los píxeles de la columna izquierda del sprite y las otras 7 columnas en el patrón del sprite se ocultan.

Esto facilita el desplazamiento suave de un sprite fuera del borde derecho de la pantalla.

Configurar un bit de **Sprite Shift del VDP** (R0, bit 3) desplaza todos los patrones de sprites ocho píxeles hacia la izquierda. Esto permite que los sprites se desplacen suavemente fuera del borde izquierdo de la pantalla.

Si se desea que la aparición y desaparición de los sprites sea suave en ambos bordes de la pantalla, se puede establecer en 1 otro bit de control del VDP (**R0, bit 5**) para dejar en blanco la columna izquierda de caracteres.

De este modo, si el bit 3 de R0 se establece en 0 (sin desplazamiento a la izquierda) y el bit 5 de R0 se establece en 1 (columna izquierda en blanco), la pantalla se reduce a 31 columnas de caracteres, y los sprites entran y salen de la pantalla suavemente. El borde izquierdo se maneja por el hecho de que un sprite con hpos=0 se coloca en la columna izquierda, la cual está en blanco en este modo.

---

