## GESTIÓN DE MEMORIA

La unidad base de la Mk3 contiene 8 kilobytes de ROM y 8 kilobytes de RAM. La ROM del programa del juego se conecta a la unidad mediante el uso de una de las dos ranuras (slots).

La ranura frontal acepta un cartucho delgado que se asemeja a una tarjeta de crédito. La capacidad actual de esta tarjeta es de 32 Kilobytes. Se está trabajando en una tarjeta de 128 Kilobytes.

La ranura superior acepta un cartucho de juego con carcasa de plástico más convencional. Los cartuchos de 128 Kilobytes ya están en producción. Cartuchos de 256 Kilobytes y mayores están en proceso de desarrollo.

El procesador Z80A es capaz de direccionar 64 kilobytes de memoria. Debido a que la memoria de los cartuchos de juego añadidos puede superar los 64 Kilobytes, se ha implementado un sistema sencillo de gestión de memoria en la Mk3.

Cuando se enciende la Mk3, el mapa de memoria contiene una ROM de 8 kilobytes en $0000 y una RAM de 8 Kilobytes en $C000.

### MOC
*   RAM del Sistema 
*   Habilitaciones de Memoria
*   Selección de Memoria al Encender
*   Mapas de Memoria
    *   32 Kilobytes
    *   128 Kilobytes
*   La Cabecera del Cartucho 


### RAM DEL SISTEMA

La RAM está mapeada en dos lugares, $C000-$DFFF y $E000-$FFFF. Esta RAM se utiliza para la pila (stack) del Z80A y para uso de bloc de notas (scratchpad).

Las ocho ubicaciones superiores de la RAM están reservadas para los registros de control de gestión de memoria y no deben ser utilizadas por un programa de juego.

Para compatibilidad futura, la RAM debe direccionarse en $C000-$DFFF. La pila debe colocarse en $DFF8.

### HABILITACIONES DE MEMORIA (MEMORY ENABLES)

El puerto de salida **$3E** controla las habilitaciones de memoria. Bits individuales en este puerto habilitan la ROM del sistema, la RAM del sistema, la ranura de tarjeta (Card Slot), la ranura de cartucho (Cartridge Slot) y la ranura externa. (La ranura externa no se utiliza actualmente).

El puerto de salida $3E tiene el siguiente formato:

**OUT ($3E):**
*   **bit 0:** no importa (don't care)
*   **bit 1:** no importa (don't care)
*   **bit 2:** 0: habilitar joysticks, 1: deshabilitar joysticks
*   **bit 3:** 0: habilitar ROM de 8K de la unidad base, 1: deshabilitar ROM de 8K de la unidad base
*   **bit 4:** 0: habilitar RAM interna de 8K, 1: deshabilitar RAM interna de 8K
*   **bit 5:** 0: habilitar ROM de tarjeta, 1: deshabilitar ROM de tarjeta
*   **bit 6:** 0: habilitar ROM de cartucho, 1: deshabilitar ROM de cartucho
*   **bit 7:** 0: habilitar puerto externo, 1: deshabilitar puerto externo

---

## SELECCIÓN DE MEMORIA AL ENCENDER (POWER-ON)

Cuando se enciende el sistema, el Z80A comienza a ejecutar código en la ubicación $0000 en la ROM residente de 8 Kilobytes. Este código de la ROM realiza los siguientes pasos:

1.  Se inicializa el sistema.
2.  Un pequeño programa se copia de la ROM a la RAM (en $C700).
3.  El programa salta a $C700 para ejecutar este programa.
4.  Se deshabilitan todos los espacios de la ROM del sistema. Es por esto que el programa se copia a la RAM y se ejecuta allí: los espacios de la ROM pueden entonces ser verificados uno por uno.
5.  Los espacios de la ROM se habilitan individualmente y se verifica la presencia de memoria. Se verifican en el siguiente orden:
    A. Ranura de tarjeta frontal.
    B. Ranura de cartucho superior.
    C. Ranura externa (trasera).
6.  Si se detecta memoria en cualquiera de estas tres ranuras, la ranura se habilita y el Z80A ejecuta un salto a la ubicación $0000.
7.  Si no se detecta memoria, la ROM de la consola se habilita y se pone un mensaje de "Insert a cartridge" (Inserte un cartucho) en la pantalla.

Tenga en cuenta que si tanto la tarjeta como el cartucho están conectados, la tarjeta se habilitará porque se verifica primero.

Un programa de juego se escribe como un programa de aplicación Z80A completamente autónomo con origen en $0000. Un programa de juego debe encargarse de todo el "mantenimiento" (housekeeping) del sistema, por ejemplo, configurar los vectores de interrupción.

**Una vez que un juego se está ejecutando, no hay acceso a la ROM de arranque.**

---

## MAPAS DE MEMORIA

Las figuras de la página siguiente muestran los mapas de memoria para dos tamaños de memoria definidos actualmente: 32 Kilobytes y 128 Kilobytes.

La Figura M-1 muestra el mapa de memoria del Z80A al encenderse.

### 32 KILOBYTES

La Figura M-2 muestra el mapa de memoria del Z80A para la tarjeta de 32 Kilobytes. Este es el caso más simple: 32 Kilobytes comenzando en la ubicación $0000.

### 128 KILOBYTES

La Figura M-3 muestra el mapa de memoria para un cartucho de 128 Kilobytes. (Esto se denomina en la literatura de Sega como un cartucho "Mega"(bit)).

Al igual que con la memoria de 32 KByte, hay una sección contigua de ROM en la mitad inferior de la memoria del Z80A. Esta sección siempre está disponible para el Z80A.

Adicionalmente, hay seis bancos de 16 Kilobytes de ROM en las direcciones de memoria $8000-$BFFF. Solo uno de estos bancos está disponible para el Z80A a la vez. Cada vez que se selecciona uno, los otros cinco están inactivos.

Cuál de los seis bancos se cambia al espacio de memoria del Z80A se controla mediante un registro en la ubicación de memoria **$FFFF**.

El Registro de Control de Banco tiene el siguiente formato:

`$FFFF: 0 0 0 0 0 b2 b1 b0`

Los bancos están numerados del 2 al 7 (b2:b1:b0).

Aunque este registro de solo escritura existe en el cartucho de memoria, las escrituras en el registro se duplican en la RAM en $FFFF. Esto significa que el registro de selección de banco parece ser de lectura/escritura, aunque lo que realmente se lee es la imagen en RAM del registro, en lugar del contenido del registro en sí.

---

## LA CABECERA DEL CARTUCHO (CARTRIDGE HEADER)

La presencia de un cartucho se comprueba buscando información específica en la ROM. Esta información debe ser correcta para que el cartucho sea reconocido.

Normalmente, Sega añadirá la información correcta al cartucho antes de la producción. La información a continuación se da solo como referencia.

El formato de la cabecera es el siguiente (las direcciones se muestran para cartuchos de 32 KByte y mayores):

### En $7FF0:
La cadena, "TMR SEGA " `[54,4D,52,20,53,45,47,41,20,20]`

### En $7FFA-$7FFB:
Un checksum de 16 bits para la ROM (orden bajo-alto). $7FFA y $7FFB no se incluyen en este checksum.

### En $7FFC-$7FFD:
Un número de serie de 16 bits, asignado por Sega.

### En $7FFE:
Un byte de número de revisión de software.

### En $7FFF:
Código de tamaño de la ROM:

*   $4A: 8K (cabecera en $1FF0)
*   $4B: 16K (cabecera en $3FF0)
*   $4C: 32K
*   $4D: 48K
*   $4E: 64K
*   $4F: 128K (1M)
*   $40: 256K (2M)
