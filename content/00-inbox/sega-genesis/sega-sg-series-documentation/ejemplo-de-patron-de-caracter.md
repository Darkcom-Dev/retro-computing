## EJEMPLO DE UN PATRÓN DE CARÁCTER

| Byte | b7 b6 b5 b4 b3 b2 b1 b0 | | Descripción |
| :--- | :--- | :--- | :--- |
| 0 | 1 1 1 1 1 1 1 1 | c0 | PRIMERA FILA DE PÍXELES: |
| 1 | . . . . . . . . | c1 | '0101' = color #5 |
| 2 | 1 1 1 1 1 1 1 1 | c2 | |
| 3 | . . . . . . . . | c3 | |
| 4 | . . . . . . . . | c0 | SEGUNDA FILA DE PÍXELES: |
| 5 | . . . . . . . . | c1 | '1100' = color #12, |
| 6 | 1 1 . . . . . . | c2 | '0000' = color #0. |
| 7 | 1 1 . . . . . . | c3 | |
| 8 | . . . . . . . . | c0 | TERCERA FILA DE PÍXELES |
| 9 | . . . . . . . . | c1 | |
| 10 | 1 1 . . . . . . | c2 | |
| 11 | 1 1 . . . . . . | c3 | |
| 12 | . . . . . . . . | c0 | CUARTA FILA DE PÍXELES |
| 13 | . . . . . . . . | c1 | |
| 14 | 1 1 . . . . . . | c2 | |
| 15 | 1 1 . . . . . . | c3 | |
| 16 | . . . . . . . . | c0 | QUINTA FILA DE PÍXELES |
| 17 | . . . . . . . . | c1 | |
| 18 | 1 1 . . . . . . | c2 | |
| 19 | 1 1 . . . . . . | c3 | |
| 20 | . . . . . . . . | c0 | SEXTA FILA DE PÍXELES |
| 21 | . . . . . . . . | c1 | |
| 22 | 1 1 . . . . . . | c2 | |
| 23 | 1 1 . . . . . . | c3 | |
| 24 | . . . . . . . . | c0 | SÉPTIMA FILA DE PÍXELES |
| 25 | . . . . . . . . | c1 | |
| 26 | 1 1 . . . . . . | c2 | |
| 27 | 1 1 . . . . . . | c3 | |
| 28 | . . . . . . . . | c0 | OCTAVA FILA DE PÍXELES |
| 29 | 1 1 1 1 1 1 1 1 | c1 | '0010' = color #2 |
| 30 | . . . . . . . . | c2 | |
| 31 | . . . . . . . . | c3 | |

Este patrón representa una letra "C" multicolor.
La sección superior es el color #5 (leyendo los bits correspondientes de MSB a LSB, 0101).
La sección vertical es el color #12 (1100).
La sección inferior es el color #2 (0010).

Qué colores representan estos depende de los valores almacenados en las ubicaciones 2, 5 y 12 de la RAM de color.

**NOTA:** el color de fondo del carácter de 8 por 8 es el #0 (0000).

---

