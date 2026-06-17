# Apéndice B. Instrucciones Comunes de x86

## Leyendo las Tablas

Las tablas de instrucciones presentadas en este apéndice incluyen:

*   El código de instrucción
*   Los operandos utilizados
*   Las banderas utilizadas
*   Una breve descripción de lo que hace la instrucción

En la sección de operandos, se listará el tipo de operandos que toma. Si toma más de un operando, cada operando estará separado por una coma. Cada operando tendrá una lista de códigos que indican si el operando puede ser un valor en modo inmediato (I), un [[15-apendice-B#data-transfer-instructions|registro]] (R), o una dirección de memoria (M). Por ejemplo, la instrucción `movl` se lista como `I/R/M, R/M`. Esto significa que el primer operando puede ser cualquier tipo de valor, mientras que el segundo operando debe ser un registro o ubicación de memoria. Observa, sin embargo, que en lenguaje ensamblador x86 no puedes tener más de un operando que sea una ubicación de memoria.

En la sección de banderas, se listan las banderas en el registro `%eflags` afectadas por la instrucción. Las siguientes banderas se mencionan:

**O**
Bandera de desbordamiento (Overflow). Se establece a verdadero si el operando destino no era lo suficientemente grande para contener el resultado de la instrucción.

**S**
Bandera de signo (Sign). Se establece al signo del último resultado.

**Z**
Bandera de cero (Zero). Esta bandera se establece a verdadero si el resultado de la instrucción es cero.

**A**
Bandera de acarreo auxiliar (Auxiliary carry). Esta bandera se establece para acarreos y préstamos entre el tercer y cuarto bit. No se usa a menudo.

**P**
Bandera de paridad (Parity). Esta bandera se establece a verdadero si el byte bajo del último resultado tenía un número par de bits 1.

**C**
Bandera de acarreo (Carry). Se usa en aritmética para decir si el resultado debería llevarse a un byte adicional. Si la bandera de acarreo está establecida, eso generalmente significa que el registro destino no pudo contener el resultado completo. Depende del programador decidir qué acción tomar (es decir, propagar el resultado a otro byte, señalar un error, o ignorarlo completamente).

Existen otras banderas, pero son mucho menos importantes.

## Instrucciones de Transferencia de Datos

Estas instrucciones realizan poco, si es que realizan algún cálculo. En cambio, se usan principalmente para mover datos de un lugar a otro.

### Tabla B-1. Instrucciones de Transferencia de Datos

| Instrucción | Operandos | Banderas Afectadas | Descripción |
| :--- | :--- | :--- | :--- |
| `movl` | I/R/M, I/R/M | O/S/Z/A/C | Esto copia una palabra de datos de una ubicación a otra. `movl %eax, %ebx` copia el contenido de `%eax` a `%ebx` |
| `movb` | I/R/M, I/R/M | O/S/Z/A/C | Igual que `movl`, pero opera en bytes individuales. |
| `leal` | M, I/R/M | O/S/Z/A/C | Esto toma una ubicación de memoria dada en el formato estándar, y, en lugar de cargar el contenido de la ubicación de memoria, carga la dirección calculada. Por ejemplo, `leal 5(%ebp,%ecx,1), %eax` carga la dirección calculada por `5 + %ebp + 1*%ecx` y la almacena en `%eax` |
| `popl` | R/M | O/S/Z/A/C | Saca el tope de la pila en la ubicación dada. Esto es equivalente a realizar `movl (%esp), R/M` seguido de `addl $4, %esp`. `popfl` es una variante que saca el tope de la pila en el registro `%eflags`. |
| `pushl` | I/R/M | O/S/Z/A/C | Empuja el valor dado a la pila. Esto es equivalente a realizar `subl $4, %esp` seguido de `movl I/R/M, (%esp)`. `pushfl` es una variante que empuja el contenido actual del registro `%eflags` al tope de la pila. |
| `xchgl` | R/M, R/M | O/S/Z/A/C | Intercambia los valores de los operandos dados. |

## Instrucciones de Enteros

Estas son instrucciones básicas de cálculo que operan en enteros con o sin signo.

### Tabla B-2. Instrucciones de Enteros

| Instrucción | Operandos | Banderas Afectadas | Descripción |
| :--- | :--- | :--- | :--- |
| `adcl` | I/R/M, R/M | O/S/Z/A/P/C | Suma con acarreo. Suma el bit de acarreo y el primer operando al segundo, y, si hay desbordamiento, establece desbordamiento y acarreo a verdadero. Esto se usa generalmente para operaciones más grandes que una palabra de máquina. La suma en la palabra menos significativa se realizaría usando `addl`, mientras que las sumas a las otras palabras usarían la instrucción `adcl` para tener en cuenta el acarreo de la suma anterior. Para el caso habitual, esto no se usa, y se usa `addl` en su lugar. |
| `addl` | I/R/M, R/M | O/S/Z/A/P/C | Suma. Suma el primer operando al segundo, almacenando el resultado en el segundo. Si el resultado es más grande que el registro destino, los bits de desbordamiento y acarreo se establecen a verdadero. Esta instrucción opera tanto en enteros con signo como sin signo. |
| `cdq` | | O/S/Z/A/P/C | Convierte la palabra `%eax` en la doble palabra que consiste en `%edx:%eax` con extensión de signo. La `q` significa quad-word. En realidad es una doble palabra, pero se llama quad-word debido a la terminología usada en los días de 16 bits. Esto se usa generalmente antes de emitir una instrucción `idivl`. |
| `cmpl` | I/R/M, R/M | O/S/Z/A/P/C | Compara dos enteros. Lo hace restando el primer operando del segundo. Descarta los resultados, pero establece las banderas en consecuencia. Generalmente se usa antes de un salto condicional. |
| `decl` | R/M | O/S/Z/A/P | Decrementa el registro o ubicación de memoria. Usa `decb` para decrementar un byte en lugar de una palabra. |
| `divl` | R/M | O/S/Z/A/P | Realiza división sin signo. Divide el contenido de la doble palabra contenida en los registros combinados `%edx:%eax` por el valor en el registro o ubicación de memoria especificado. El registro `%eax` contiene el cociente resultante, y el registro `%edx` contiene el resto resultante. Si el cociente es demasiado grande para caber en `%eax`, desencadena una interrupción de tipo 0. |
| `idivl` | R/M | O/S/Z/A/P | Realiza división con signo. Opera igual que `divl` arriba. |
| `imull` | R/M/I, R | O/S/Z/A/P/C | Realiza multiplicación con signo y almacena el resultado en el segundo operando. Si el segundo operando se omite, se asume que es `%eax`, y el resultado completo se almacena en la doble palabra `%edx:%eax`. |
| `incl` | R/M | O/S/Z/A/P | Incrementa el registro o ubicación de memoria dado. Usa `incb` para incrementar un byte en lugar de una palabra. |
| `mull` | R/M/I, R | O/S/Z/A/P/C | Realiza multiplicación sin signo. Las mismas reglas que aplican a `imull`. |
| `negl` | R/M | O/S/Z/A/P/C | Niega (da la inversión de complemento a dos de) el registro o ubicación de memoria dado. |
| `sbbl` | I/R/M, R/M | O/S/Z/A/P/C | Resta con préstamo. Se usa de la misma manera que `adc`, excepto para resta. Normalmente solo se usa `subl`. |
| `subl` | I/R/M, R/M | O/S/Z/A/P/C | Resta los dos operandos. Esto resta el primer operando del segundo, y almacena el resultado en el segundo operando. Esta instrucción puede usarse tanto en números con signo como sin signo. |

## Instrucciones Lógicas

Estas instrucciones operan en la memoria como bits en lugar de palabras.

### Tabla B-3. Instrucciones Lógicas

| Instrucción | Operandos | Banderas Afectadas | Descripción |
| :--- | :--- | :--- | :--- |
| `andl` | I/R/M, R/M | O/S/Z/P/C | Realiza un and lógico de los contenidos de los dos operandos, y almacena el resultado en el segundo operando. Establece las banderas de desbordamiento y acarreo a falso. |
| `notl` | R/M | | Realiza un not lógico en cada bit del operando. También conocido como complemento a uno. |
| `orl` | I/R/M, R/M | O/S/Z/A/P/C | Realiza un or lógico entre los dos operandos, y almacena el resultado en el segundo operando. Establece las banderas de desbordamiento y acarreo a falso. |
| `rcll` | I/%cl, R/M | O/C | Rota los bits de la ubicación dada a la izquierda el número de veces en el primer operando, que es un valor en modo inmediato o el registro `%cl`. La bandera de acarreo se incluye en la rotación, haciendo que use 33 bits en lugar de 32. También establece la bandera de desbordamiento. |
| `rcrl` | I/%cl, R/M | O/C | Igual que arriba, pero rota a la derecha. |
| `roll` | I/%cl, R/M | O/C | Rota bits a la izquierda. Establece las banderas de desbordamiento y acarreo, pero no cuenta la bandera de acarreo como parte de la rotación. El número de bits a rotar se especifica en modo inmediato o está contenido en el registro `%cl`. |
| `rorl` | I/%cl, R/M | O/C | Igual que arriba, pero rota a la derecha. |
| `sall` | I/%cl, R/M | C | Desplazamiento aritmético a la izquierda. El bit de signo se desplaza hacia la bandera de acarreo, y un bit cero se coloca en el bit menos significativo. Otros bits simplemente se desplazan a la izquierda. Esto es lo mismo que el desplazamiento regular a la izquierda. El número de bits a desplazar se especifica en modo inmediato o está contenido en el registro `%cl`. |
| `sarl` | I/%cl, R/M | C | Desplazamiento aritmético a la derecha. El bit menos significativo se desplaza hacia la bandera de acarreo. El bit de signo se desplaza hacia adentro, y se mantiene como el bit de signo. Otros bits simplemente se desplazan a la derecha. El número de bits a desplazar se especifica en modo inmediato o está contenido en el registro `%cl`. |
| `shll` | I/%cl, R/M | C | Desplazamiento lógico a la izquierda. Esto desplaza todos los bits a la izquierda (el bit de signo no se trata especialmente). El bit más a la izquierda se empuja a la bandera de acarreo. El número de bits a desplazar se especifica en modo inmediato o está contenido en el registro `%cl`. |
| `shrl` | I/%cl, R/M | C | Desplazamiento lógico a la derecha. Esto desplaza todos los bits en el registro a la derecha (el bit de signo no se trata especialmente). El bit más a la derecha se empuja a la bandera de acarreo. El número de bits a desplazar se especifica en modo inmediato o está contenido en el registro `%cl`. |
| `testl` | I/R/M, R/M | O/S/Z/A/P/C | Hace un and lógico de ambos operandos y descarta los resultados, pero establece las banderas en consecuencia. |
| `xorl` | I/R/M, R/M | O/S/Z/A/P/C | Hace un or exclusivo en los dos operandos, y almacena el resultado en el segundo operando. Establece las banderas de desbordamiento y acarreo a falso. |

## Instrucciones de Control de Flujo

Estas instrucciones pueden alterar el flujo del programa.

### Tabla B-4. Instrucciones de Control de Flujo

| Instrucción | Operandos | Banderas Afectadas | Descripción |
| :--- | :--- | :--- | :--- |
| `call` | dirección destino | O/S/Z/A/C | Esto empuja lo que sería el siguiente valor para `%eip` a la pila, y salta a la dirección destino. Se usa para llamadas a funciones. Alternativamente, la dirección destino puede ser un asterisco seguido de un registro para una llamada indirecta a función. Por ejemplo, `call *%eax` llamará a la función en la dirección en `%eax`. |
| `int` | I | O/S/Z/A/C | Causa una interrupción del número dado. Esto se usa generalmente para llamadas al sistema y otras interfaces del kernel. |
| `Jcc` | dirección destino | O/S/Z/A/C | Rama condicional. `cc` es el código de condición. Salta a la dirección dada si el código de condición es verdadero (establecido desde la instrucción anterior, probablemente una comparación). De lo contrario, va a la siguiente instrucción. Los códigos de condición son: <ul><li>`[n]a[e]` - above (mayor que sin signo). Se puede añadir una `n` para "not" y una `e` para "or equal to"</li><li>`[n]b[e]` - below (menor que sin signo)</li><li>`[n]e` - equal (igual a)</li><li>`[n]z` - zero (cero)</li><li>`[n]g[e]` - greater than (mayor que con signo)</li><li>`[n]l[e]` - less than (menor que con signo)</li><li>`[n]c` - carry flag set (bandera de acarreo establecida)</li><li>`[n]o` - overflow flag set (bandera de desbordamiento establecida)</li><li>`[p]p` - parity flag set (bandera de paridad establecida)</li><li>`[n]s` - sign flag set (bandera de signo establecida)</li><li>`ecxz` - `%ecx` es cero</li></ul> |
| `jmp` | dirección destino | O/S/Z/A/C | Un salto incondicional. Esto simplemente establece `%eip` a la dirección destino. Alternativamente, la dirección destino puede ser un asterisco seguido de un registro para un salto indirecto. Por ejemplo, `jmp *%eax` saltará a la dirección en `%eax`. |
| `ret` | | O/S/Z/A/C | Saca un valor de la pila y luego establece `%eip` a ese valor. Se usa para regresar de llamadas a funciones. |

## Directivas del Ensamblador

Estas son instrucciones para el ensamblador y el enlazador, en lugar de instrucciones para el procesador. Se usan para ayudar al ensamblador a armar tu código correctamente, y hacerlo más fácil de usar.

### Tabla B-5. Directivas del Ensamblador

| Directiva | Operandos | Descripción |
| :--- | :--- | :--- |
| `.ascii` | CADENA ENTRE COMILLAS | Toma la cadena entre comillas dada y la convierte en datos de byte. |
| `.byte` | VALORES | Toma una lista de valores separados por comas y los inserta justo allí en el programa como datos. |
| `.endr` | | Termina una sección repetitiva definida con `.rept`. |
| `.equ` | ETIQUETA, VALOR | Establece la etiqueta dada como equivalente al valor dado. El valor puede ser un número, un carácter, o una expresión constante que se evalúa a un número o carácter. Desde ese punto, el uso de la etiqueta será sustituido por el valor dado. |
| `.globl` | ETIQUETA | Establece la etiqueta dada como global, lo que significa que puede ser usada desde archivos objeto compilados por separado. |
| `.include` | ARCHIVO | Incluye el archivo dado como si se hubiera escrito justo allí. |
| `.lcomm` | SÍMBOLO, TAMAÑO | Esto se usa en la sección `.bss` para especificar almacenamiento que debe asignarse cuando se ejecuta el programa. Define el símbolo con la dirección donde se ubicará el almacenamiento, y se asegura de que tenga el número dado de bytes de largo. |
| `.long` | VALORES | Toma una secuencia de números separados por comas, e inserta esos números como palabras de 4 bytes justo donde están en el programa. |
| `.rept` | CONTEO | Repite todo entre esta directiva y la directiva `.endr` el número de veces especificado. |
| `.section` | NOMBRE DE SECCIÓN | Cambia la sección en la que se está trabajando. Las secciones comunes incluyen `.text` (para código), `.data` (para datos incrustados en el programa mismo), y `.bss` (para datos globales no inicializados). |
| `.type` | SÍMBOLO, @function | Le dice al enlazador que el símbolo dado es una función. |

## Diferencias en Otras Sintaxis y Terminología

La sintaxis para el lenguaje ensamblador utilizado en este libro se conoce como **sintaxis AT&T**. Es la soportada por el conjunto de herramientas GNU que viene estándar con cada distribución de Linux. Sin embargo, la sintaxis oficial para el lenguaje ensamblador x86 (conocida como **sintaxis Intel®**) es diferente. Es el mismo lenguaje ensamblador para la misma plataforma, pero se ve diferente. Algunas de las diferencias incluyen:

*   En la sintaxis Intel, los operandos de las instrucciones a menudo están invertidos. El operando destino se lista antes del operando origen.
*   En la sintaxis Intel, los registros no tienen el prefijo del signo de porcentaje (`%`).
*   En la sintaxis Intel, no se requiere un signo de dólar (`$`) para hacer direccionamiento en modo inmediato. En cambio, el direccionamiento no inmediato se logra rodeando la dirección con corchetes (`[]`).
*   En la sintaxis Intel, el nombre de la instrucción no incluye el tamaño de los datos que se están moviendo. Si eso es ambiguo, se establece explícitamente como `BYTE`, `WORD`, o `DWORD` inmediatamente después del nombre de la instrucción.
*   La forma en que se representan las direcciones de memoria en el lenguaje ensamblador Intel es muy diferente (se muestra a continuación).
*   Debido a que la línea de procesadores x86 originalmente comenzó como un procesador de 16 bits, la mayoría de la literatura sobre procesadores x86 se refiere a las palabras como valores de 16 bits, y llama a los valores de 32 bits palabras dobles. Sin embargo, nosotros usamos el término "palabra" para referirnos al tamaño de registro estándar en un procesador, que es de 32 bits en un procesador x86. La sintaxis también mantiene esta convención de nomenclatura - `DWORD` significa "double word" en la sintaxis Intel y se usa para registros de tamaño estándar, que nosotros llamaríamos simplemente una "palabra".
*   El lenguaje ensamblador Intel tiene la capacidad de direccionar memoria como un par segmento/desplazamiento. No mencionamos esto porque Linux no soporta memoria segmentada, y por lo tanto es irrelevante para la programación normal de Linux.

Existen otras diferencias, pero son pequeñas en comparación. Para mostrar algunas de las diferencias, considera la siguiente instrucción:

**Sintaxis AT&T:**
```assembly
movl %eax, 8(%ebx,%edi,4)
```

**Sintaxis Intel:**
```assembly
mov [8 + ebx + 4 * edi], eax
```

La referencia a memoria es un poco más fácil de leer que su contraparte AT&T porque explica exactamente cómo se calculará la dirección. Sin embargo, el orden de los operandos en la sintaxis Intel puede ser confuso.

## Dónde Ir para Más Información

Intel tiene un conjunto de guías completas para sus procesadores. Estas están disponibles en http://www.intel.com/design/pentium/manuals/ Nota que todas usan la sintaxis Intel, no la sintaxis AT&T. Las más importantes son su IA-32 Intel Architecture Software Developer's Manual en sus tres volúmenes:

*   **Volumen 1: System Programming Guide** (http://developer.intel.com/design/pentium4/manuals/245470.htm)
*   **Volumen 2: Instruction Set Reference** (http://developer.intel.com/design/pentium4/manuals/245471.htm)
*   **Volumen 3: System Programming Guide** (http://developer.intel.com/design/pentium4/manuals/245472.htm)

Además, puedes encontrar mucha información en el manual para el ensamblador GNU, disponible en línea en http://www.gnu.org/software/binutils/manual/gas-2.9.1/as.html. Del mismo modo, el manual para el enlazador GNU está disponible en línea en http://www.gnu.org/software/binutils/manual/ld-2.9.1/ld.html.
