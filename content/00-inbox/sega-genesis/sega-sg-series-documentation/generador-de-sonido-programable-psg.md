
## GENERADOR DE SONIDO PROGRAMABLE (PSG)

El PSG contiene cuatro canales de sonido, que consisten en tres generadores de tonos y un generador de ruido. Cada uno de los cuatro canales tiene un control de volumen independiente (atenuador). El PSG se controla a través del puerto de salida **$7F**.

### MOC
*   Frecuencia del Generador de Tonos
*   Control del Generador de Ruido 
*   Atenuadores

### FRECUENCIA DEL GENERADOR DE TONOS

La frecuencia (tono) de un generador de tonos se establece mediante un valor de 10 bits. Este valor se descuenta hasta que llega a cero, momento en el cual la salida del tono conmuta (toggles) y el valor de 10 bits se recarga en el contador. Por lo tanto, los números de 10 bits más altos producen frecuencias más bajas.

Para cargar un nuevo valor de frecuencia en uno de los generadores de tonos, se escribe un par de bytes en la ubicación de E/S **$7F** de acuerdo con el siguiente formato:

| Byte | Formato de Bits |
| :--- | :--- |
| Primer Byte: | `1 | R2 | R1 | R0 | d3 | d2 | d1 | d0` |
| Segundo Byte: | `0 | 0 | d9 | d8 | d7 | d6 | d5 | d4` |

El campo **R2:R1:R0** selecciona el canal de tono de la siguiente manera:

| R2 | R1 | R0 | Canal de Tono |
| :---: | :---: | :---: | :--- |
| 0 | 0 | 0 | #1 |
| 0 | 1 | 0 | #2 |
| 1 | 0 | 0 | #3 |

Los datos de 10 bits son: (msb) `d9 d8 d7 d6 d5 d4 d3 d2 d1 d0` (lsb)

---

## CONTROL DEL GENERADOR DE RUIDO

El generador de ruido utiliza tres bits de control para seleccionar el "carácter" del sonido de ruido. Un bit llamado "**FB**" (Retroalimentación/Feedback) produce ruido periódico o ruido "blanco" (hiss):

| FB | Tipo de Ruido |
| :---: | :--- |
| 0 | Periódico (similar a un tono de baja frecuencia) |
| 1 | Blanco (siseo) |

La frecuencia del ruido se selecciona mediante dos bits **NF1:NF0** de acuerdo con la siguiente tabla:

| NF1 | NF0 | Fuente de Reloj del Generador de Ruido |
| :---: | :---: | :--- |
| 0 | 0 | Reloj/2 [Tono más alto, "menos grueso"] |
| 0 | 1 | Reloj/4 |
| 1 | 0 | Reloj/8 [Tono más bajo, "más grueso"] |
| 1 | 1 | Generador de Tonos #3 |

**Nota:** El "Reloj" tiene una frecuencia fija. Es una señal de oscilador controlada por cristal conectada al PSG.

Cuando **NF1:NF0** es 11, el Generador de Tonos #3 suministra la fuente de reloj de ruido. Esto permite que el ruido sea "barrido" en frecuencia. Este efecto podría usarse para el arranque de un motor a reacción, por ejemplo.

Para cargar estos bits de control del generador de ruido, escriba el siguiente byte en el puerto de E/S **$7F**:

**Salida ($7F):** `1 | 1 | 1 | 0 | 0 | FB | NF1 | NF0`

---

## ATENUADORES

Cuatro atenuadores ajustan el volumen de los tres generadores de tonos y el canal de ruido. Cuatro bits **A3:A2:A1:A0** mantienen el control de la atenuación de la siguiente manera:

| A3 | A2 | A1 | A0 | Atenuación |
| :---: | :---: | :---: | :---: | :--- |
| 0 | 0 | 0 | 0 | 0 db (volumen máximo) |
| 0 | 0 | 0 | 1 | 2 db **NOTA:** una atenuación mayor |
| 0 | 0 | 1 | 0 | 4 db da como resultado un |
| 0 | 0 | 1 | 1 | 6 db sonido más silencioso. |
| 0 | 1 | 0 | 0 | 8 db |
| 0 | 1 | 0 | 1 | 10 db |
| 0 | 1 | 1 | 0 | 12 db |
| 0 | 1 | 1 | 1 | 14 db |
| 1 | 0 | 0 | 0 | 16 db |
| 1 | 0 | 0 | 1 | 18 db |
| 1 | 0 | 1 | 0 | 20 db |
| 1 | 0 | 1 | 1 | 22 db |
| 1 | 1 | 0 | 0 | 24 db |
| 1 | 1 | 0 | 1 | 26 db |
| 1 | 1 | 1 | 0 | 28 db |
| 1 | 1 | 1 | 1 | -Off- (Apagado) |

Los atenuadores se configuran para los cuatro canales escribiendo los siguientes bytes en la ubicación de E/S **$7F**:

| Canal | Formato de Bits |
| :--- | :--- |
| Generador de Tonos #1: | `1 | 0 | 0 | 1 | A3 | A2 | A1 | A0` |
| Generador de Tonos #2: | `1 | 0 | 1 | 1 | A3 | A2 | A1 | A0` |
| Generador de Tonos #3: | `1 | 1 | 0 | 1 | A3 | A2 | A1 | A0` |
| Generador de Ruido: | `1 | 1 | 1 | 1 | A3 | A2 | A1 | A0` |

### EJEMPLO

Cuando se enciende la Mk3, se ejecuta el siguiente código:

```assembly
        LD HL,CLRTB   ; limpiar tabla
        LD C,PSG_PRT  ; puerto psg es $7F
        LD B,4        ; cargar cuatro bytes
        OTIR          ; escribirlos
        (etc.)

CLRTB   defb $9F,$BF,$DF,$FF
```

Este código apaga los cuatro canales de sonido. Es una buena idea ejecutar también este código cuando se presiona el botón **PAUSE**, para que el sonido no permanezca continuamente durante el intervalo de pausa.
