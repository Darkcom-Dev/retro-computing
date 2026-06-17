# Apéndice D. Tabla de Códigos ASCII

Para usar esta tabla, simplemente encuentra el carácter o escape para el que quieres el código, y suma el número de la izquierda y el de arriba.

### Tabla D-1. Tabla de códigos ASCII en decimal

| | +0 | +1 | +2 | +3 | +4 | +5 | +6 | +7 |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **0** | NUL | SOH | STX | ETX | EOT | ENQ | ACK | BEL |
| **8** | BS | HT | LF | VT | FF | CR | SO | SI |
| **16** | DLE | DC1 | DC2 | DC3 | DC4 | NAK | SYN | ETB |
| **24** | CAN | EM | SUB | ESC | FS | GS | RS | US |
| **32** | (sp) | ! | " | # | $ | % | & | ' |
| **40** | ( | ) | * | + | , | - | . | / |
| **48** | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| **56** | 8 | 9 | : | ; | < | = | > | ? |
| **64** | @ | A | B | C | D | E | F | G |
| **72** | H | I | J | K | L | M | N | O |
| **80** | P | Q | R | S | T | U | V | W |
| **88** | X | Y | Z | [ | \ | ] | ^ | _ |
| **96** | ` | a | b | c | d | e | f | g |
| **104** | h | i | j | k | l | m | n | o |
| **112** | p | q | r | s | t | u | v | w |
| **120** | x | y | z | { | \| | } | ~ | DEL |

ASCII está siendo reemplazado gradualmente en favor de un estándar internacional conocido como **Unicode**, que te permite mostrar cualquier carácter de cualquier sistema de escritura conocido en el mundo. Como habrás notado, ASCII solo tiene soporte para caracteres en inglés. Unicode es mucho más complicado, sin embargo, porque requiere más de un byte para codificar un solo carácter. Hay varios métodos diferentes para codificar caracteres Unicode. Los más comunes son UTF-8 y UTF-32. UTF-8 es algo compatible hacia atrás con ASCII (se almacena igual para caracteres en inglés, pero se expande a múltiples bytes para caracteres internacionales). UTF-32 simplemente requiere cuatro bytes por cada carácter en lugar de uno. Windows® usa UTF-16, que es una codificación de longitud variable que requiere al menos 2 bytes por carácter, por lo que no es compatible hacia atrás con ASCII.

Un buen tutorial sobre temas de internacionalización, fuentes y Unicode está disponible en un gran artículo de Joe Spolsky, llamado "The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets (No Excuses!)", disponible en línea en http://www.joelonsoftware.com/articles/Unicode.html
