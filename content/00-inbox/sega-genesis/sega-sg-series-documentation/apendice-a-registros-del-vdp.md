## Apéndice A - Registros del VDP

```
     +-------+-------+-------+-------+-------+-------+-------+-------+
     |Inhib. |Inhib. | COL.  |       |Despl. |       |       |       |
 R0  |Despl. |Despl. | IZQ.  |  IE1  |Sprite |   1   |   1   |   0   |
     | Vert. | Horiz.| Blanco|       | (SS)  |       | (M3)  |       |
     +-------+-------+-------+-------+-------+-------+-------+-------+
     +-------+-------+-------+-------+-------+-------+-------+-------+
     |       | Habil.|       |   0   |   0   |       | Tamaño| Bit   |
 R1  |   1   | Pant. |  IE   | (M1)  | (M2)  |   0   | Sprite| Mag.  |
     |       | (Bit) |       |       |       |       |  8/16 | Sprite|
     +-------+-------+-------+-------+-------+-------+-------+-------+
     +-------+-------+-------+-------+-------+-------+-------+-------+
     |       |       |       |       | BASE  | BASE  | BASE  |       | $FF
 R2  |   1   |   1   |   1   |   1   | PANT. | PANT. | PANT. |   1   | (Base Pantalla
     |       |       |       |       |   2   |   1   |   0   |       | en $3800)
     +-------+-------+-------+-------+-------+-------+-------+-------+
     +-------+-------+-------+-------+-------+-------+-------+-------+
     |       |       |       |       |       |       |       |       |
 R3  |   1   |   1   |   1   |   1   |   1   |   1   |   1   |   1   | $FF
     |       |       |       |       |       |       |       |       |
     +-------+-------+-------+-------+-------+-------+-------+-------+
     +-------+-------+-------+-------+-------+-------+-------+-------+
     |       |       |       |       |       |       |       |       |
 R4  |   1   |   1   |   1   |   1   |   1   |   1   |   1   |   1   | $FF
     |       |       |       |       |       |       |       |       |
     +-------+-------+-------+-------+-------+-------+-------+-------+
     +-------+-------+-------+-------+-------+-------+-------+-------+
     |       | BASE  | BASE  | BASE  | BASE  | BASE  | BASE  |       | $FF (SAT en $3F00)
 R5  |   1   | SAT   | SAT   | SAT   | SAT   | SAT   | SAT   |   1   | SAT=Tabla de
     |       |   5   |   4   |   3   |   2   |   1   |   0   |       | Atrib. Sprites
     +-------+-------+-------+-------+-------+-------+-------+-------+
     +-------+-------+-------+-------+-------+-------+-------+-------+
     |       |       |       |       |       | BANCO |       |       | $FB (CG en $0000)
 R6  |   1   |   1   |   1   |   1   |   1   | CGEN  |   1   |   1   | CGEN=Generador
     |       |       |       |       |       |   0   |       |       | de Caracteres
     +-------+-------+-------+-------+-------+-------+-------+-------+
     +-------+-------+-------+-------+-------+-------+-------+-------+
     |       |       |       |       | COLOR | COLOR | COLOR | COLOR |
 R7  |   1   |   1   |   1   |   1   | BORDE | BORDE | BORDE | BORDE | Color del Borde
     |       |       |       |       |   3   |   2   |   1   |   0   |
     +-------+-------+-------+-------+-------+-------+-------+-------+
     +-------+-------+-------+-------+-------+-------+-------+-------+
     |       |       |       |       |       |       |       |       | $00 Desplaz.
 R8  | SCROL | SCROL | SCROL | SCROL | SCROL | SCROL | SCROL | SCROL | Horizontal
     |  H7   |  H6   |  H5   |  H4   |  H3   |  H2   |  H1   |  H0   | <-----
     +-------+-------+-------+-------+-------+-------+-------+-------+
     +-------+-------+-------+-------+-------+-------+-------+-------+
     |       |       |       |       |       |       |       |       | $00 Desplaz.
 R9  | SCROL | SCROL | SCROL | SCROL | SCROL | SCROL | SCROL | SCROL | Vertical
     |  V7   |  V6   |  V5   |  V4   |  V3   |  V2   |  V1   |  V0   |
     +-------+-------+-------+-------+-------+-------+-------+-------+
     +-------+-------+-------+-------+-------+-------+-------+-------+
     |       |       |       |       |       |       |       |       | $FF Interrup.
 R10 |  HLI  |  HLI  |  HLI  |  HLI  |  HLI  |  HLI  |  HLI  |  HLI  | Línea Horiz.
     |   7   |   6   |   5   |   4   |   3   |   2   |   1   |   0   | ($C1-$FF=Deshab)
     +-------+-------+-------+-------+-------+-------+-------+-------+
```

**Lectura Puerto $BF (ESTADO VDP)**
```
 +-------+-------+-------+-------+-------+-------+-------+-------+
 | FUENTE| NUEVE | COLIS.|       |       |       |       |       |
 | INTR. |SPRITES| SPRIT.|   X   |   X   |   X   |   X   |   X   |
 | (VBL) | FLAG  | FLAG  |       |       |       |       |       |
 +-------+-------+-------+-------+-------+-------+-------+-------+
```

**Escritura Puerto $BF (CONTROL VDP)**
```
 Para escribir en Registro VDP  Primer Byte:  |  Datos del Registro   |
 r3:r2:r1:r0 = No. de Reg.      Segundo Byte: |1 |0 |X |X |r3|r2|r1|r0|
```

**NOTA:** Primero deshabilite las interrupciones, ya que el registro de comando del VDP siempre debe escribirse dos veces. (Las interrupciones leen el registro de estado y los accesos a los registros del VDP son sensibles a la secuencia).

---

## REGISTROS DEL VDP

El VDP está controlado por once registros internos de 8 bits. Esta sección describe el método mediante el cual el Z80A accede a estos registros y luego analiza la función de los registros individuales.

El Z80 "ve" el chip VDP a través de dos ubicaciones de puertos de E/S, **$BE** y **$BF**. La ubicación de E/S **$BF** es el registro de **COMANDO** (COMMAND) para una escritura, y el registro de **ESTADO** (STATUS) para una lectura.

El puerto de E/S **$BE** es el registro de **DATOS** (DATA) de lectura/escritura.

El registro de Comando se escribe dos veces seguidas para todas las operaciones de comando; el registro de datos puede leerse o escribirse cualquier número de veces seguidas, dependiendo de la operación.

Debido a que el registro de comando es sensible a la secuencia, requiriendo un par de bytes de salida por operación, una instrucción **DI** debe preceder a las actualizaciones del registro de comando. Esto evita, por ejemplo, que una rutina de servicio de interrupción que lee el registro de estado del VDP interrumpa la sincronización de dos bytes.

**Existe una restricción de tiempo para acceder al chip VDP.**

El chip VDP no puede procesar datos más rápido que las siguientes tasas:
*   16 Estados-T del Z80A durante VBLANK.
*   29 Estados-T del Z80A durante el video activo.

Esto significa que nunca debe emitir dos instrucciones OUT o IN consecutivas al VDP; deben estar separadas por al menos una instrucción NOP.

Esta restricción se aplica tanto a la RAM de video y la RAM de color como a los registros internos.

Un campo de modo de dos bits en los bits 7 y 6 del segundo byte de COMANDO determina una de cuatro operaciones:

| b7 | b6 | Operación |
| :--- | :--- | :--- |
| 0 | 0 | Leer 16 Kbytes de RAM del VDP |
| 0 | 1 | Escribir 16 Kbytes de RAM del VDP |
| 1 | 0 | Escribir Registro del VDP |
| 1 | 1 | Escribir RAM de Color |

Para configurar las dos primeras operaciones, lectura/escritura de la RAM de video, los dos bytes de COMANDO tienen el siguiente formato:

### Leer RAM del VDP

**Primer byte** escrito en $BF: `A7 | A6 | A5 | A4 | A3 | A2 | A1 | A0`
**Segundo byte** escrito en $BF: ` 0 | 0 | A13| A12| A11| A10| A9 | A8`

### Escribir RAM del VDP

**Primer byte** escrito en $BF: `A7 | A6 | A5 | A4 | A3 | A2 | A1 | A0`
**Segundo byte** escrito en $BF: ` 0 | 1 | A13| A12| A11| A10| A9 | A8`

---

Después de que se hayan emitido los bytes de COMANDO, los datos se pueden leer en el puerto de entrada $BE o escribir en el puerto de salida $BE.

El VDP tiene una función de **auto-incremento de dirección**. Una vez que se ha cargado la dirección inicial de la RAM de video, se puede acceder repetidamente a los datos (puerto de E/S $BE) y la dirección aumentará automáticamente en un byte por cada acceso.

**NOTA:** La función de auto-incremento funciona solo con lecturas continuas o escrituras continuas. Cambiar de lectura a escritura o viceversa requiere dos escrituras en el registro de comando para volver a seleccionar el modo.

El formato del registro de COMANDO para escribir en un registro del VDP (son de solo escritura) se muestra a continuación:

### Escribir Registro del VDP

**Primer byte** escrito en $BF: `d7 | d6 | d5 | d4 | d3 | d2 | d1 | d0`
**Segundo byte** escrito en $BF: ` 1 | 0 | 0 | 0 | r3 | r2 | r1 | r0`

El número de registro es r3-r0, y va del 0 al 10 ($00 al $0A). Los datos que se escribirán en el registro son d7-d0.

Para la operación de "escribir registro" únicamente, no hay escritura en el registro de DATOS en $BE, ya que los datos (d7-d0) están contenidos en el primer byte del par de bytes de COMANDO.

El modo de comando final permite escribir valores de color en la RAM de color del VDP, que es de solo escritura.

### Escribir RAM de Color del VDP

**Primer byte** escrito en $BF: ` 0 | 0 | 0 | A4 | A3 | A2 | A1 | A0`
**Segundo byte** escrito en $BF: ` 1 | 1 | 0 | 0 | 0 | 0 | 0 | 0`

Después de cargar este par de bytes en el registro de COMANDO, los datos de la RAM de color se escriben en el puerto de salida $BE en el siguiente formato:

**Datos RAM Color** escritos en $BE: ` 0 | 0 | b1 | b0 | g1 | g0 | r1 | r0`

Donde b1-b0 establecen cuatro intensidades para azul, g1-g0 para verde y r1-r0 para rojo.

Al igual que con la RAM de video, las escrituras repetidas en el puerto de salida $BE incrementan automáticamente la dirección de la RAM de color. Las direcciones de la RAM de color se mapean como se muestra en la página siguiente.

## REGISTROS DEL VDP

El VDP está controlado por once registros internos de 8 bits. Esta sección describe el método mediante el cual el Z80A accede a estos registros y luego analiza la función de los registros individuales.

El Z80 "ve" el chip VDP a través de dos ubicaciones de puertos de E/S, **$BE** y **$BF**. La ubicación de E/S **$BF** es el registro de **COMANDO** (COMMAND) para una escritura, y el registro de **ESTADO** (STATUS) para una lectura.

El puerto de E/S **$BE** es el registro de **DATOS** (DATA) de lectura/escritura.

El registro de Comando se escribe dos veces seguidas para todas las operaciones de comando; el registro de datos puede leerse o escribirse cualquier número de veces seguidas, dependiendo de la operación.

Debido a que el registro de comando es sensible a la secuencia, requiriendo un par de bytes de salida por operación, una instrucción **DI** debe preceder a las actualizaciones del registro de comando. Esto evita, por ejemplo, que una rutina de servicio de interrupción que lee el registro de estado del VDP interrumpa la sincronización de dos bytes.

**Existe una restricción de tiempo para acceder al chip VDP.**

El chip VDP no puede procesar datos más rápido que las siguientes tasas:
*   16 Estados-T del Z80A durante VBLANK.
*   29 Estados-T del Z80A durante el video activo.

Esto significa que nunca debe emitir dos instrucciones OUT o IN consecutivas al VDP; deben estar separadas por al menos una instrucción NOP.

Esta restricción se aplica tanto a la RAM de video y la RAM de color como a los registros internos.

Un campo de modo de dos bits en los bits 7 y 6 del segundo byte de COMANDO determina una de cuatro operaciones:

| b7 | b6 | Operación |
| :--- | :--- | :--- |
| 0 | 0 | Leer 16 Kbytes de RAM del VDP |
| 0 | 1 | Escribir 16 Kbytes de RAM del VDP |
| 1 | 0 | Escribir Registro del VDP |
| 1 | 1 | Escribir RAM de Color |

Para configurar las dos primeras operaciones, lectura/escritura de la RAM de video, los dos bytes de COMANDO tienen el siguiente formato:

### Leer RAM del VDP

**Primer byte** escrito en $BF: `A7 | A6 | A5 | A4 | A3 | A2 | A1 | A0`
**Segundo byte** escrito en $BF: ` 0 | 0 | A13| A12| A11| A10| A9 | A8`

### Escribir RAM del VDP

**Primer byte** escrito en $BF: `A7 | A6 | A5 | A4 | A3 | A2 | A1 | A0`
**Segundo byte** escrito en $BF: ` 0 | 1 | A13| A12| A11| A10| A9 | A8`

---

Después de que se hayan emitido los bytes de COMANDO, los datos se pueden leer en el puerto de entrada $BE o escribir en el puerto de salida $BE.

El VDP tiene una función de **auto-incremento de dirección**. Una vez que se ha cargado la dirección inicial de la RAM de video, se puede acceder repetidamente a los datos (puerto de E/S $BE) y la dirección aumentará automáticamente en un byte por cada acceso.

**NOTA:** La función de auto-incremento funciona solo con lecturas continuas o escrituras continuas. Cambiar de lectura a escritura o viceversa requiere dos escrituras en el registro de comando para volver a seleccionar el modo.

El formato del registro de COMANDO para escribir en un registro del VDP (son de solo escritura) se muestra a continuación:

### Escribir Registro del VDP

**Primer byte** escrito en $BF: `d7 | d6 | d5 | d4 | d3 | d2 | d1 | d0`
**Segundo byte** escrito en $BF: ` 1 | 0 | 0 | 0 | r3 | r2 | r1 | r0`

El número de registro es r3-r0, y va del 0 al 10 ($00 al $0A). Los datos que se escribirán en el registro son d7-d0.

Para la operación de "escribir registro" únicamente, no hay escritura en el registro de DATOS en $BE, ya que los datos (d7-d0) están contenidos en el primer byte del par de bytes de COMANDO.

El modo de comando final permite escribir valores de color en la RAM de color del VDP, que es de solo escritura.

### Escribir RAM de Color del VDP

**Primer byte** escrito en $BF: ` 0 | 0 | 0 | A4 | A3 | A2 | A1 | A0`
**Segundo byte** escrito en $BF: ` 1 | 1 | 0 | 0 | 0 | 0 | 0 | 0`

Después de cargar este par de bytes en el registro de COMANDO, los datos de la RAM de color se escriben en el puerto de salida $BE en el siguiente formato:

**Datos RAM Color** escritos en $BE: ` 0 | 0 | b1 | b0 | g1 | g0 | r1 | r0`

Donde b1-b0 establecen cuatro intensidades para azul, g1-g0 para verde y r1-r0 para rojo.

Al igual que con la RAM de video, las escrituras repetidas en el puerto de salida $BE incrementan automáticamente la dirección de la RAM de color. Las direcciones de la RAM de color se mapean como se muestra en la página siguiente.
