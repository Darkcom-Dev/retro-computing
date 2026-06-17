## Notas del Desarrollador

### MOC
*   Mapa de Memoria del Z80
*   Registros de Control de Memoria
*   ROM de 4M Bits 
*   Dirección Z80 $0000 - $03FF

### (1) Z80 - Mapa de Memoria

```
 +---------------+ $FFFF
 |               |
 |    Frame 3    |
 |               |
 +---------------+ $C000
 |               |
 |    Frame 2    | ------ $FFFF (Registro de Control)
 |               |
 +---------------+ $8000
 |               |
 |    Frame 1    | ------ $FFFE (Registro de Control)
 |               |
 +---------------+ $4000
 |               |
 |    Frame 0    | ------ $FFFD (Registro de Control)
 |               |
 +---------------+ $0000
```

### (2) Registros de Control de Memoria

**\* $FFFF - Registro de Control del Frame 2**
```
 msb                               lsb
 +---+---+---+---+---+---+---+---+
 | 0 | 0 | 0 |   |   |   |   |   |
 +---+---+---+---+---+---+---+---+
                 +---+---+---+---+---- Número de Página ROM para Frame 2
```

**\* $FFFE - Registro de Control del Frame 1**
```
 msb                               lsb
 +---+---+---+---+---+---+---+---+
 | 0 | 0 | 0 |   |   |   |   |   |
 +---+---+---+---+---+---+---+---+
                 +---+---+---+---+---- Número de Página ROM para Frame 1
```

**\* $FFFD - Registro de Control del Frame 0**
```
 msb                               lsb
 +---+---+---+---+---+---+---+---+
 | 0 | 0 | 0 |   |   |   |   |   |
 +---+---+---+---+---+---+---+---+
                 +---+---+---+---+---- Número de Página ROM para Frame 0
```

**\* $FFFC - Registro de Control de RAM del Frame 2**
```
 msb  7       4   3   2   1   0   lsb
     +---+---+---+---+---+---+---+---+
     |   | 0 | 0 |   |   |   |   |   |
     +---+---+---+---+---+---+---+---+
       |   |   |   \_______ ________/
       |   |   |           v
       |   |   +---------- Desplazamiento de Número de Banco (Bank Number Shift)
       |   +-------------- Número de Banco de RAM Externa
       +------------------ Conmutador ROM/RAM (0=ROM, 1=RAM)
                          Interruptor de Protección de Escritura de RAM?
                          Interruptor de Protección de Escritura de Placa de Desarrollo?
```

**NOTA:** Cuando se aplica energía, el programa de la ROM del sistema establece los siguientes datos:
`$FFFF = 2`, `$FFFE = 1`, `$FFFD = 0`, `$FFFC = 0`

El bit 7 de $FFFC es el bit de protección de escritura para la Placa ROM (Solo para la placa de desarrollo).
`0 = L/E (Lectura/Escritura)`, `1 = Solo Lectura`

---

### (3) La ROM de 4M Bits

```
 $80000 ----+-----+--+-----+--+-----+--+-----+
        | $1F |  | $17 |  | $0F |  | $07 |  <-- Número de Banco
        +-----+  +-----+  +-----+  +-----+
        |  .  |  |  .  |  |  .  |  |  .  |
        |  .  |  |  .  |  |  .  |  |  .  |  La dirección que se
        |  .  |  |  .  |  |  .  |  |  .  |  establece mediante Bank Shift 00
        +-----+  +-----+  +-----+  +-----+  y Bank Number 09 puede
        | $19 |  | $11 |  | $09 |  | $01 |  también ser accedida mediante
        +-----+  +-----+  +-----+  +-----+  Bank Shift 01 y
        | $18 |  | $10 |  | $08 |  | $00 |  Bank Number 1.
 $60000 ----+-----+--+-----+--+-----+--+-----+
        | $17 |  | $0F |  | $07 |  | $1F |
        +-----+  +-----+  +-----+  +-----+
        |  .  |  |  .  |  |  .  |  |  .  |
        |  .  |  |  .  |  |  .  |  |  .  |
        |  .  |  |  .  |  |  .  |  |  .  |
        +-----+  +-----+  +-----+  +-----+
        | $11 |  | $09 |  | $01 |  | $19 |
        +-----+  +-----+  +-----+  +-----+
        | $10 |  | $08 |  | $00 |  | $18 |
 $40000 ----+-----+--+-----+--+-----+--+-----+
        | $0F |  | $07 |  | $1F |  | $17 |
        +-----+  +-----+  +-----+  +-----+
        |  .  |  |  .  |  |  .  |  |  .  |
        |  .  |  |  .  |  |  .  |  |  .  |
        |  .  |  |  .  |  |  .  |  |  .  |
        +-----+  +-----+  +-----+  +-----+
        | $09 |  | $01 |  | $19 |  | $11 |
        +-----+  +-----+  +-----+  +-----+
        | $08 |  | $00 |  | $18 |  | $10 |
 $20000 ----+-----+--+-----+--+-----+--+-----+
        | $07 |  | $1F |  | $17 |  | $0F |
        +-----+  +-----+  +-----+  +-----+
        |  .  |  |  .  |  |  .  |  |  .  |
        |  .  |  |  .  |  |  .  |  |  .  |
        |  .  |  |  .  |  |  .  |  |  .  |
        +-----+  +-----+  +-----+  +-----+
        | $01 |  | $19 |  | $11 |  | $09 |
        +-----+  +-----+  +-----+  +-----+
        | $00 |  | $18 |  | $10 |  | $08 |
 $00000 ----+-----+--+-----+--+-----+--+-----+
          00       01       10       11     <-- Bits 0 y 1 de $FFFC 
                                                (Bank Number Shift)
```

### (4) Dirección Z80 $0000 - $03FF

El 1K inferior (1024 bytes) del rango de direcciones del Z80 siempre está mapeado al 1K inferior (1024 bytes) de la ROM para asegurar que los vectores de reinicio e interrupción estén siempre disponibles.

Por ejemplo, si el Banco #8 (Dirección ROM $20000 - $24000) fuera asignado al Frame 0 (Dirección Z80 $0000 - $4000), entonces la Dirección ROM $20000 - $203FF no podría ser accedida, ya que la Dirección Z80 $0000 - $03FF permanece mapeada a la Dirección ROM $00000 - $003FF.
