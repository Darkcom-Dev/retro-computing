## RAM DE COLOR

El chip VDP contiene una RAM de color de 32 valores de 6 bits. Estos valores tienen el formato de bytes de 8 bits de "solo escritura":

```
+---+---+----+----+----+----+----+----+
| X | X | B1 | B0 | G1 | G0 | R1 | R0 |
+---+---+----+----+----+----+----+----+
```

donde R1:R0, G1:G0 y B1:B0 definen cuatro intensidades para rojo, verde y azul de la siguiente manera:

| R,G,B-1 | R,G,B-0 | Intensidad |
| :---: | :---: | :--- |
| 0 | 0 | Apagado (Off) |
| 0 | 1 | 1/3 |
| 1 | 0 | 2/3 |
| 1 | 1 | 3/3 (más brillante) |

La RAM de color está organizada como dos bancos de 16 colores cada uno. Los colores se seleccionan de cualquiera de los bancos con un código de color de cuatro bits.

