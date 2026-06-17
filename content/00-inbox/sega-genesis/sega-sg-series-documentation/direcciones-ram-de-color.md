## DIRECCIONES DE LA RAM DE COLOR

| Dirección RAM Color | Número de Color | Notas |
| :--- | :--- | :--- |
| 00000 ($00) | Banco 1, Color 0 | [Nota: Los colores de fondo |
| 00001 ($01) | Banco 1, Color 1 | pueden tomarse tanto del |
| 00010 ($02) | Banco 1, Color 2 | primer grupo de dieciséis |
| 00011 ($03) | Banco 1, Color 3 | como del segundo grupo de |
| 00100 ($04) | Banco 1, Color 4 | dieciséis colores] |
| 00101 ($05) | Banco 1, Color 5 | |
| 00110 ($06) | Banco 1, Color 6 | |
| 00111 ($07) | Banco 1, Color 7 | |
| 01000 ($08) | Banco 1, Color 8 | |
| 01001 ($09) | Banco 1, Color 9 | |
| 01010 ($0A) | Banco 1, Color 10 | |
| 01011 ($0B) | Banco 1, Color 11 | |
| 01100 ($0C) | Banco 1, Color 12 | |
| 01101 ($0D) | Banco 1, Color 13 | |
| 01110 ($0E) | Banco 1, Color 14 | |
| 01111 ($0F) | Banco 1, Color 15 | |
| | | |
| 10000 ($10) | Banco 2, Color 0 | [Nota: Los colores del borde |
| 10001 ($11) | Banco 2, Color 1 | y de los sprites se toman |
| 10010 ($12) | Banco 2, Color 2 | de este segundo grupo de |
| 10011 ($13) | Banco 2, Color 3 | dieciséis colores] |
| 10100 ($14) | Banco 2, Color 4 | |
| 10101 ($15) | Banco 2, Color 5 | |
| 10110 ($16) | Banco 2, Color 6 | |
| 10111 ($17) | Banco 2, Color 7 | |
| 11000 ($18) | Banco 2, Color 8 | |
| 11001 ($19) | Banco 2, Color 9 | |
| 11010 ($1A) | Banco 2, Color 10 | |
| 11011 ($1B) | Banco 2, Color 11 | |
| 11100 ($1C) | Banco 2, Color 12 | |
| 11101 ($1D) | Banco 2, Color 13 | |
| 11110 ($1E) | Banco 2, Color 14 | |
| 11111 ($1F) | Banco 2, Color 15 | |

El registro de comando tiene una función adicional. Cuando se lee con una instrucción **IN ($BF)**, funciona como un registro de estado de interrupción y también limpia las solicitudes de interrupción del chip VDP.

### ACTUALIZACIONES DE LOS REGISTROS DEL VDP

En general, todas las actualizaciones del VDP deben realizarse mientras la señal de video de la TV está en blanco (blanked). Si las operaciones que afectan la apariencia de la pantalla no se realizan durante el borrado de pantalla, la imagen parecerá tener "ruido".

Su programa puede detectar los intervalos de borrado horizontal y vertical utilizando el sistema de interrupciones y los bits de estado (flags) del VDP. Esta capacidad se analiza en la sección de la CPU.

No es necesario sincronizar la inicialización del VDP con los intervalos de borrado, ya que la pantalla puede ponerse en blanco antes de las operaciones de configuración y luego encenderse cuando la inicialización se haya completado.

---

[[descripcion-de-los-registros-del-vdp]]