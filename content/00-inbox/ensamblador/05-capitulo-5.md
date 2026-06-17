# Capítulo 5. Trabajando con Archivos

Gran parte de la programación de computadoras trata con archivos. Después de todo, cuando reiniciamos nuestras computadoras, lo único que queda de las sesiones anteriores son las cosas que se han guardado en el disco. Los datos que se almacenan en archivos se llaman **datos persistentes**, porque persisten en archivos que permanecen en el disco incluso cuando el programa no se está ejecutando.

## El Concepto de Archivo en UNIX

Cada sistema operativo tiene su propia forma de tratar con los archivos. Sin embargo, el método UNIX, que se utiliza en Linux, es el más simple y universal. Todos los archivos UNIX, sin importar qué programa los haya creado, pueden ser accedidos como un flujo secuencial de bytes. Cuando accedes a un archivo, comienzas abriéndolo por su nombre. El sistema operativo te da entonces un número, llamado **descriptor de archivo** (file descriptor), que utilizas para referirte al archivo hasta que hayas terminado con él. Luego puedes leer y escribir en el archivo utilizando su descriptor de archivo. Cuando hayas terminado de leer y escribir, cierras el archivo, lo que hace que el descriptor de archivo quede inutilizado.

En nuestros programas trataremos con archivos de las siguientes maneras:

1. Decirle a Linux el nombre del archivo a abrir y en qué modo quieres que se abra (lectura, escritura, tanto lectura como escritura, crearlo si no existe, etc.). Esto se maneja con la llamada al sistema `open`, que toma como parámetros un nombre de archivo, un número que representa el modo y un conjunto de permisos. `%eax` contendrá el número de la llamada al sistema, que es 5. La dirección del primer carácter del nombre del archivo debe almacenarse en `%ebx`. Las intenciones de lectura/escritura, representadas como un número, deben almacenarse en `%ecx`. Por ahora, usa 0 para los archivos de los que quieras leer y 03101 para los archivos en los que quieras escribir (debes incluir el cero inicial). [^1] Finalmente, el conjunto de permisos debe almacenarse como un número en `%edx`. Si no estás familiarizado con los permisos de UNIX, simplemente usa 0666 para los permisos (de nuevo, debes incluir el cero inicial).
2. Linux te devolverá entonces un descriptor de archivo en `%eax`. Recuerda, este es un número que usas para referirte a este archivo a lo largo de tu programa.
3. A continuación, operarás sobre el archivo realizando lecturas y/o escrituras, dando cada vez a Linux el descriptor de archivo que quieres usar. `read` es la llamada al sistema 3, y para llamarla necesitas tener el descriptor de archivo en `%ebx`, la dirección de un búfer para almacenar los datos leídos en `%ecx` y el tamaño del búfer en `%edx`. Los búferes se explicarán en la sección llamada *Búferes y .bss*. `read` devolverá el número de caracteres leídos del archivo o un código de error. Los códigos de error se pueden distinguir porque siempre son números negativos (puedes encontrar más información sobre números negativos en el Capítulo 10). `write` es la llamada al sistema 4, y requiere los mismos parámetros que la llamada al sistema `read`, excepto que el búfer ya debe estar lleno con los datos a escribir. La llamada al sistema `write` devolverá el número de bytes escritos en `%eax` o un código de error.
4. Cuando hayas terminado con tus archivos, puedes decirle a Linux que los cierre. Después, tu descriptor de archivo ya no será válido. Esto se hace usando `close`, llamada al sistema 6. El único parámetro para `close` es el descriptor de archivo, que se coloca en `%ebx`.

## Búferes y .bss

En la sección anterior mencionamos los búferes sin explicar qué eran. Un **búfer** (buffer) es un bloque continuo de bytes utilizado para la transferencia masiva de datos. Cuando solicitas leer un archivo, el sistema operativo necesita tener un lugar para almacenar los datos que lee. Ese lugar se llama búfer. Normalmente, los búferes solo se usan para almacenar datos temporalmente, y luego se leen de los búferes y se convierten a una forma que sea más fácil de manejar para los programas. Nuestros programas no serán lo suficientemente complicados como para necesitar eso. Por ejemplo, supongamos que quieres leer una sola línea de texto de un archivo pero no sabes qué tan larga es esa línea. Entonces simplemente leerías un gran número de bytes/caracteres del archivo en un búfer, buscarías el carácter de fin de línea y copiarías todos los caracteres hasta ese carácter de fin de línea a otra ubicación. Si no encontraras un carácter de fin de línea, asignarías otro búfer y continuarías leyendo. Probablemente terminarías con algunos caracteres sobrantes en tu búfer en este caso, que usarías como punto de partida la próxima vez que necesites datos del archivo. [^2]

Otra cosa a tener en cuenta es que los búferes son de un tamaño fijo, establecido por el programador. Por lo tanto, si quieres leer datos de 500 bytes en 500 bytes, envías a la llamada al sistema de lectura la dirección de una ubicación no utilizada de 500 bytes y le envías el número 500 para que sepa qué tan grande es. Puedes hacerlo más pequeño o más grande, según las necesidades de tu aplicación.

Para crear un búfer, necesitas reservar almacenamiento estático o dinámico. El almacenamiento estático es de lo que hemos hablado hasta ahora, ubicaciones de almacenamiento declaradas mediante las directivas `.long` o `.byte`. El almacenamiento dinámico se discutirá en la sección llamada *Obteniendo Más Memoria* en el Capítulo 9. Sin embargo, hay problemas al declarar búferes usando `.byte`. Primero, es tedioso de escribir. Tendrías que escribir 500 números después de la declaración `.byte`, y no se usarían para nada más que para ocupar espacio. Segundo, ocupa espacio en el ejecutable. En los ejemplos que hemos usado hasta ahora, no ocupa demasiado, pero eso puede cambiar en programas más grandes. Si quieres 500 bytes tienes que escribir 500 números y eso desperdicia 500 bytes en el ejecutable. Hay una solución para ambos problemas. Hasta ahora, hemos discutido dos secciones de programa, las secciones `.text` y `.data`. Hay otra sección llamada **`.bss`**. Esta sección es como la sección de datos, excepto que no ocupa espacio en el ejecutable. Esta sección puede reservar almacenamiento, pero no puede inicializarlo. En la sección `.data`, podrías reservar almacenamiento y establecerlo en un valor inicial. En la sección `.bss`, no puedes establecer un valor inicial. Esto es útil para los búferes porque de todos modos no necesitamos inicializarlos, solo necesitamos reservar almacenamiento. Para hacer esto, ejecutamos los siguientes comandos:

```assembly
.section .bss
.lcomm my_buffer, 500
```

Esta directiva, `.lcomm`, creará un símbolo, `my_buffer`, que se refiere a una ubicación de almacenamiento de 500 bytes que podemos usar como búfer. Luego podemos hacer lo siguiente, asumiendo que hemos abierto un archivo para lectura y hemos colocado el descriptor de archivo en `%ebx`:

```assembly
movl $my_buffer, %ecx
movl 500, %edx
movl 3, %eax
int $0x80
```

Esto leerá hasta 500 bytes en nuestro búfer. En este ejemplo, coloqué un signo de dólar delante de `my_buffer`. Recuerda que la razón de esto es que, sin el signo de dólar, `my_buffer` se trata como una ubicación de memoria y se accede en modo de direccionamiento directo. El signo de dólar lo cambia al direccionamiento en modo inmediato, lo que realmente carga el número representado por `my_buffer` (es decir, la dirección del inicio de nuestro búfer) en `%ecx`.

## Archivos Estándar y Especiales

Podrías pensar que los programas comienzan sin ningún archivo abierto por defecto. Esto no es cierto. Los programas de Linux suelen tener al menos tres descriptores de archivo abiertos cuando comienzan. Son:

- **STDIN**: Esta es la entrada estándar (standard input). Es un archivo de solo lectura y generalmente representa tu teclado. [^3] Este es siempre el descriptor de archivo 0.
- **STDOUT**: Esta es la salida estándar (standard output). Es un archivo de solo escritura y generalmente representa la pantalla. Este es siempre el descriptor de archivo 1.
- **STDERR**: Este es tu error estándar (standard error). Es un archivo de solo escritura y generalmente representa la pantalla. La mayor parte de la salida del procesamiento regular va a STDOUT, pero cualquier mensaje de error que surja en el proceso va a STDERR. De esta manera, si quieres, puedes dividirlos en lugares separados. Este es siempre el descriptor de archivo 2.

Cualquiera de estos "archivos" puede ser redireccionado desde o hacia un archivo real, en lugar de una pantalla o un teclado. Esto está fuera del alcance de este libro, pero cualquier buen libro sobre la línea de comandos de UNIX lo describirá en detalle. El programa en sí ni siquiera necesita ser consciente de esta indirección; simplemente puede usar los descriptores de archivo estándar como de costumbre.

Observa que muchos de los archivos en los que escribes no son archivos en absoluto. Los sistemas operativos basados en UNIX tratan todos los sistemas de entrada/salida como archivos. Las conexiones de red se tratan como archivos, tu puerto serie se trata como un archivo, incluso tus dispositivos de audio se tratan como archivos. La comunicación entre procesos se realiza generalmente a través de archivos especiales llamados **tuberías** (pipes). Algunos de estos archivos tienen métodos diferentes para abrirlos y crearlos que los archivos normales (es decir, no usan la llamada al sistema `open`), pero todos pueden ser leídos y escritos usando las llamadas al sistema estándar `read` and `write`.

## Usando Archivos en un Programa

Vamos a escribir un programa simple para ilustrar estos conceptos. El programa tomará dos archivos y leerá de uno, convertirá todas sus letras minúsculas a mayúsculas y escribirá en el otro archivo. Antes de hacerlo, pensemos en lo que necesitamos hacer para realizar el trabajo:

- Tener una función que tome un bloque de memoria y lo convierta a mayúsculas. Esta función necesitaría la dirección de un bloque de memoria y su tamaño como parámetros.
- Tener una sección de código que lea repetidamente en un búfer, llame a nuestra función de conversión sobre el búfer y luego vuelva a escribir el búfer en el otro archivo.
- Comenzar el programa abriendo los archivos necesarios.

Observa que he especificado las cosas en el orden inverso al que se harán. Ese es un truco útil al escribir programas complejos: primero decide el núcleo de lo que se está haciendo. En este caso, es convertir bloques de caracteres a mayúsculas. Luego, piensa en todo lo que necesita ser configurado y procesado para que eso suceda. En este caso, tienes que abrir archivos y leer y escribir bloques en el disco continuamente. Una de las claves de la programación es descomponer continuamente los problemas en trozos cada vez más pequeños hasta que sean lo suficientemente pequeños como para que puedas resolver el problema fácilmente. Luego puedes volver a armar estos trozos hasta que tengas un programa funcional. [^4]

Puede que hayas estado pensando que nunca recordarás todos estos números que te lanzan: los números de las llamadas al sistema, el número de interrupción, etc. En este programa también introduciremos una nueva directiva, `.equ`, que debería ayudar. `.equ` permite asignar nombres a los números. Por ejemplo, si hicieras `.equ LINUX_SYSCALL, 0x80`, en cualquier momento posterior que escribieras `LINUX_SYSCALL`, el ensamblador sustituiría eso por `0x80`. Así que ahora puedes escribir:

```assembly
int $LINUX_SYSCALL
```

que es mucho más fácil de leer y mucho más fácil de recordar. La codificación es compleja, pero hay muchas cosas que podemos hacer como esta para facilitarla.

Aquí está el programa. Ten en cuenta que tenemos más etiquetas de las que realmente usamos para saltos, porque algunas de ellas están ahí solo por claridad. Intenta seguir el programa y ver qué sucede en varios casos. A continuación seguirá una explicación detallada del programa.

```assembly
#PROPÓSITO:    Este programa convierte un archivo de entrada
#              en un archivo de salida con todas las letras
#              convertidas a mayúsculas.
#
#PROCESAMIENTO: 1) Abrir el archivo de entrada
#               2) Abrir el archivo de salida
#               4) Mientras no estemos al final del archivo de entrada
#                  a) leer parte del archivo en nuestro búfer de memoria
#                  b) recorrer cada byte de memoria
#                     si el byte es una letra minúscula,
#                     convertirla a mayúscula
#                  c) escribir el búfer de memoria en el archivo de salida

.section .data

#######CONSTANTES########

#números de llamadas al sistema
.equ SYS_OPEN, 5
.equ SYS_WRITE, 4
.equ SYS_READ, 3
.equ SYS_CLOSE, 6
.equ SYS_EXIT, 1

#opciones para open (mira
#/usr/include/asm/fcntl.h para
#varios valores. Puedes combinarlos
#sumándolos o aplicando la operación OR)
#Esto se discute con más detalle
#en "Contando como una Computadora"
.equ O_RDONLY, 0
.equ O_CREAT_WRONLY_TRUNC, 03101

#descriptores de archivo estándar
.equ STDIN, 0
.equ STDOUT, 1
.equ STDERR, 2

#interrupción de llamada al sistema
.equ LINUX_SYSCALL, 0x80

.equ END_OF_FILE, 0  #Este es el valor de retorno
                     #de read que significa que hemos
                     #llegado al final del archivo

.equ NUMBER_ARGUMENTS, 2

.section .bss
#Búfer: aquí es donde se cargan los datos desde el
#       archivo de datos y desde donde se escriben
#       en el archivo de salida. Esto nunca debería
#       exceder 16,000 por varias razones.
.equ BUFFER_SIZE, 500
.lcomm BUFFER_DATA, BUFFER_SIZE

.section .text

#POSICIONES EN LA PILA
.equ ST_SIZE_RESERVE, 8
.equ ST_FD_IN, -4
.equ ST_FD_OUT, -8
.equ ST_ARGC, 0      #Número de argumentos
.equ ST_ARGV_0, 4    #Nombre del programa
.equ ST_ARGV_1, 8    #Nombre del archivo de entrada
.equ ST_ARGV_2, 12   #Nombre del archivo de salida

.globl _start
_start:
 ###INICIALIZAR PROGRAMA###
 #guardar el puntero de pila
 movl %esp, %ebp

 #Asignar espacio para nuestros descriptores de archivo
 #en la pila
 subl $ST_SIZE_RESERVE, %esp

open_files:
open_fd_in:
 ###ABRIR ARCHIVO DE ENTRADA###
 #syscall open
 movl $SYS_OPEN, %eax
 #nombre del archivo de entrada en %ebx
 movl ST_ARGV_1(%ebp), %ebx
 #indicador de solo lectura
 movl $O_RDONLY, %ecx
 #esto realmente no importa para leer
 movl $0666, %edx
 #llamar a Linux
 int  $LINUX_SYSCALL

store_fd_in:
 #guardar el descriptor de archivo dado
 movl %eax, ST_FD_IN(%ebp)

open_fd_out:
 ###ABRIR ARCHIVO DE SALIDA###
 #abrir el archivo
 movl $SYS_OPEN, %eax
 #nombre del archivo de salida en %ebx
 movl ST_ARGV_2(%ebp), %ebx
 #indicadores para escribir en el archivo
 movl $O_CREAT_WRONLY_TRUNC, %ecx
 #modo para el nuevo archivo (si es creado)
 movl $0666, %edx
 #llamar a Linux
 int  $LINUX_SYSCALL

store_fd_out:
 #almacenar el descriptor de archivo aquí
 movl %eax, ST_FD_OUT(%ebp)

 ###INICIAR BUCLE PRINCIPAL###
read_loop_begin:

 ###LEER UN BLOQUE DEL ARCHIVO DE ENTRADA###
 movl $SYS_READ, %eax
 #obtener el descriptor del archivo de entrada
 movl ST_FD_IN(%ebp), %ebx
 #la ubicación donde leer
 movl $BUFFER_DATA, %ecx
 #el tamaño del búfer
 movl $BUFFER_SIZE, %edx
 #El tamaño del búfer leído se devuelve en %eax
 int  $LINUX_SYSCALL

 ###SALIR SI HEMOS LLEGADO AL FINAL###
 #comprobar si hay marcador de fin de archivo
 cmpl $END_OF_FILE, %eax
 #si se encuentra o en caso de error, ir al final
 jle  end_loop

continue_read_loop:
 ###CONVERTIR EL BLOQUE A MAYÚSCULAS###
 pushl $BUFFER_DATA     #ubicación del búfer
 pushl %eax             #tamaño del búfer
 call  convert_to_upper
 popl  %eax             #recuperar el tamaño
 addl  $4, %esp         #restaurar %esp

 ###ESCRIBIR EL BLOQUE EN EL ARCHIVO DE SALIDA###
 #tamaño del búfer
 movl %eax, %edx
 movl $SYS_WRITE, %eax
 #archivo a usar
 movl ST_FD_OUT(%ebp), %ebx
 #ubicación del búfer
 movl $BUFFER_DATA, %ecx
 int  $LINUX_SYSCALL

 ###CONTINUAR EL BUCLE###
 jmp  read_loop_begin

end_loop:
 ###CERRAR LOS ARCHIVOS###
 #NOTA: no necesitamos realizar comprobación de errores
 #      en estos, porque las condiciones de error
 #      no significan nada especial aquí
 movl $SYS_CLOSE, %eax
 movl ST_FD_OUT(%ebp), %ebx
 int  $LINUX_SYSCALL

 movl $SYS_CLOSE, %eax
 movl ST_FD_IN(%ebp), %ebx
 int  $LINUX_SYSCALL

 ###SALIR###
 movl $SYS_EXIT, %eax
 movl $0, %ebx
 int  $LINUX_SYSCALL


#PROPÓSITO:    Esta función realiza realmente la
#              conversión a mayúsculas para un bloque
#
#ENTRADA:      El primer parámetro es la ubicación
#              del bloque de memoria a convertir
#              El segundo parámetro es la longitud de
#              ese búfer
#
#SALIDA:       Esta función sobrescribe el búfer actual
#              con la versión convertida a mayúsculas.
#
#VARIABLES:
#              %eax - inicio del búfer
#              %ebx - longitud del búfer
#              %edi - desplazamiento actual del búfer
#              %cl - byte actual que se está examinando
#                    (primera parte de %ecx)
#

###CONSTANTES##
#El límite inferior de nuestra búsqueda
.equ LOWERCASE_A, 'a'
#El límite superior de nuestra búsqueda
.equ LOWERCASE_Z, 'z'
#Conversión entre mayúsculas y minúsculas
.equ UPPER_CONVERSION, 'A' - 'a'

###COSAS DE LA PILA###
.equ ST_BUFFER_LEN, 8 #Longitud del búfer
.equ ST_BUFFER, 12    #búfer real

convert_to_upper:
 pushl %ebp
 movl  %esp, %ebp

 ###CONFIGURAR VARIABLES###
 movl  ST_BUFFER(%ebp), %eax
 movl  ST_BUFFER_LEN(%ebp), %ebx
 movl  $0, %edi

 #si se nos dio un búfer con longitud cero,
 #simplemente salir
 cmpl  $0, %ebx
 je    end_convert_loop

convert_loop:
 #obtener el byte actual
 movb  (%eax,%edi,1), %cl

 #ir al siguiente byte a menos que esté entre
 #'a' y 'z'
 cmpb  $LOWERCASE_A, %cl
 jl    next_byte
 cmpb  $LOWERCASE_Z, %cl
 jg    next_byte

 #de lo contrario, convertir el byte a mayúscula
 addb  $UPPER_CONVERSION, %cl
 #y volver a almacenarlo
 movb  %cl, (%eax,%edi,1)

next_byte:
 incl  %edi              #siguiente byte
 cmpl  %edi, %ebx         #continuar a menos que
                         #hayamos llegado al
                         #final
 jne   convert_loop

end_convert_loop:
 #sin valor de retorno, simplemente salir
 movl  %ebp, %esp
 popl  %ebp
 ret
```

Escribe este programa como `toupper.s` y luego introduce los siguientes comandos:

```bash
as toupper.s -o toupper.o
ld toupper.o -o toupper
```

Esto construye un programa llamado `toupper`, que convierte todos los caracteres en minúscula de un archivo a mayúsculas. Por ejemplo, para convertir el archivo `toupper.s` a mayúsculas, escribe el siguiente comando:

```bash
./toupper toupper.s toupper.uppercase
```

Ahora encontrarás en el archivo `toupper.uppercase` una versión en mayúsculas de tu archivo original.

Examinemos cómo funciona el programa.

La primera sección del programa está marcada como **CONSTANTES** (CONSTANTS). En programación, una constante es un valor que se asigna cuando un programa se ensambla o compila, y nunca se cambia. Tengo la costumbre de colocar todas mis constantes juntas al principio del programa. Solo es necesario declararlas antes de usarlas, pero ponerlas todas al principio hace que sean fáciles de encontrar. Hacerlas todas en mayúsculas hace obvio en tu programa qué valores son constantes y dónde encontrarlos. [^5] En lenguaje ensamblador, declaramos constantes con la directiva `.equ` como se mencionó antes. Aquí, simplemente damos nombres a todos los números estándar que hemos usado hasta ahora, como los números de las llamadas al sistema, el número de interrupción de syscall y las opciones de apertura de archivos.

La siguiente sección está marcada como **BÚFERES** (BUFFERS). Solo usamos un búfer en este programa, al que llamamos `BUFFER_DATA`. También definimos una constante, `BUFFER_SIZE`, que contiene el tamaño del búfer. Si siempre nos referimos a esta constante en lugar de escribir el número 500 cada vez que necesitemos usar el tamaño del búfer, si luego cambia, solo necesitamos modificar este valor, en lugar de tener que recorrer todo el programa y cambiar todos los valores individualmente.

En lugar de pasar a la sección `_start` del programa, ve al final donde definimos la función `convert_to_upper`. Esta es la parte que realmente realiza la conversión. Esta sección comienza con una lista de constantes que usaremos. La razón por la que se ponen aquí en lugar de en la parte superior es que solo tratan con esta única función. Tenemos estas definiciones:

```assembly
.equ LOWERCASE_A, 'a'
.equ LOWERCASE_Z, 'z'
.equ UPPER_CONVERSION, 'A' - 'a'
```

Las dos primeras simplemente definen las letras que son los límites de lo que estamos buscando. Recuerda que en la computadora, las letras se representan como números. Por lo tanto, podemos usar `LOWERCASE_A` en comparaciones, sumas, restas o cualquier otra cosa en la que podamos usar números. Además, observa que definimos la constante `UPPER_CONVERSION`. Como las letras se representan como números, podemos restarlas. Restar una letra mayúscula de la misma letra minúscula nos indica cuánto necesitamos sumar a una letra minúscula para convertirla en mayúscula. Si eso no tiene sentido, mira las tablas de códigos ASCII (ver el Apéndice D). Notarás que el número para el carácter `A` es 65 y el carácter `a` es 97. El factor de conversión es entonces -32. Para cualquier letra minúscula, si le sumas -32, obtendrás su equivalente en mayúscula.

Después de esto, tenemos algunas constantes etiquetadas como **POSICIONES EN LA PILA** (STACK POSITIONS). Recuerda que los parámetros de la función se meten en la pila antes de las llamadas a funciones. Estas constantes (con el prefijo `ST` para mayor claridad) definen en qué parte de la pila debemos esperar encontrar cada dato. La dirección de retorno está en la posición `4 + %esp`, la longitud del búfer está en la posición `8 + %esp` y la dirección del búfer está en la posición `12 + %esp`. Usar símbolos para estos números en lugar de los números mismos hace que sea más fácil ver qué datos se están usando y moviendo.

A continuación viene la etiqueta `convert_to_upper`. Este es el punto de entrada de la función. Las dos primeras líneas son nuestras líneas estándar de función para guardar el puntero de pila. Las siguientes dos líneas:

```assembly
movl ST_BUFFER(%ebp), %eax
movl ST_BUFFER_LEN(%ebp), %ebx
```

mueven los parámetros de la función a los registros apropiados para su uso. Luego, cargamos cero en `%edi`. Lo que vamos a hacer es iterar a través de cada byte del búfer cargando desde la ubicación `%eax + %edi`, incrementando `%edi` y repitiendo hasta que `%edi` sea igual a la longitud del búfer almacenada en `%ebx`. Las líneas:

```assembly
cmpl $0, %ebx
je end_convert_loop
```

son solo una comprobación de cordura para asegurarnos de que nadie nos dio un búfer de tamaño cero. Si lo hicieron, simplemente limpiamos y salimos. Protegerse contra posibles errores del usuario y de programación es una tarea importante de un programador. Siempre puedes especificar que tu función no debe aceptar un búfer de tamaño cero, pero es aún mejor que la función lo compruebe y tenga un plan de salida fiable si sucede.

Ahora comenzamos nuestro bucle. Primero, mueve un byte a `%cl`. El código para esto es:

```assembly
movb (%eax,%edi,1), %cl
```

Está utilizando un **modo de direccionamiento indirecto indexado**. Dice que comience en `%eax` y avance `%edi` ubicaciones, siendo cada ubicación de 1 byte de tamaño. Toma el valor encontrado allí y lo pone en `%cl`. Después de esto, comprueba si ese valor está en el rango de la `a` minúscula a la `z` minúscula. Para comprobar el rango, simplemente comprueba si la letra es más pequeña que la `a`. Si lo es, no puede ser una letra minúscula. Del mismo modo, si es más grande que la `z`, no puede ser una letra minúscula. Así que, en cada uno de estos casos, simplemente continúa. Si está en el rango adecuado, entonces suma la conversión a mayúsculas y la almacena de nuevo en el búfer.

De cualquier manera, luego pasa al siguiente valor incrementando `%edi`. A continuación, comprueba si estamos al final del búfer. Si no estamos al final, saltamos de vuelta al principio del bucle (la etiqueta `convert_loop`). Si estamos al final, simplemente continúa hasta el final de la función. Debido a que estamos modificando el búfer directamente, no necesitamos devolver nada al programa que realizó la llamada; los cambios ya están en el búfer. La etiqueta `end_convert_loop` no es necesaria, pero está ahí para que sea fácil ver dónde están las partes del programa.

Ahora sabemos cómo funciona el proceso de conversión. Ahora necesitamos averiguar cómo meter y sacar los datos de los archivos. Antes de leer y escribir los archivos debemos abrirlos. La llamada al sistema UNIX `open` es la que maneja esto. Toma los siguientes parámetros:

- `%eax` contiene el número de la llamada al sistema como de costumbre: 5 en este caso.
- `%ebx` contiene un puntero a una cadena que es el nombre del archivo a abrir. La cadena debe estar terminada con el carácter nulo.
- `%ecx` contiene las opciones utilizadas para abrir el archivo. Estas le indican a Linux cómo abrir el archivo. Pueden indicar cosas como abrir para lectura, abrir para escritura, abrir para lectura y escritura, crear si no existe, eliminar el archivo si ya existe, etc. No entraremos en cómo crear los números para las opciones hasta la sección llamada *Verdad, Falsedad y Números Binarios* en el Capítulo 10. Por ahora, simplemente confía en los números que proponemos.
- `%edx` contiene los permisos que se utilizan para abrir el archivo. Esto se usa en caso de que el archivo tenga que ser creado primero, para que Linux sepa con qué permisos crear el archivo. Estos se expresan en octal, al igual que los permisos regulares de UNIX. [^6]

Después de realizar la llamada al sistema, el descriptor de archivo del archivo recién abierto se almacena en `%eax`.

Entonces, ¿qué archivos estamos abriendo? En este ejemplo, abriremos los archivos especificados en la línea de comandos. Afortunadamente, los parámetros de la línea de comandos ya son almacenados por Linux en una ubicación de fácil acceso y ya están terminados en nulo. Cuando comienza un programa Linux, todos los punteros a los argumentos de la línea de comandos se almacenan en la pila. El número de argumentos se almacena en `8(%esp)`, el nombre del programa se almacena en `12(%esp)` y los argumentos se almacenan a partir de `16(%esp)`. En el lenguaje de programación C, esto se conoce como el array `argv`, por lo que nos referiremos a él de esa manera en nuestro programa.

Lo primero que hace nuestro programa es guardar la posición actual de la pila en `%ebp` y luego reservar algo de espacio en la pila para almacenar los descriptores de archivo. Después de esto, comienza a abrir archivos. El primer archivo que abre el programa es el archivo de entrada, que es el primer argumento de la línea de comandos. Hacemos esto configurando la llamada al sistema. Ponemos el nombre del archivo en `%ebx`, el número del modo de solo lectura en `%ecx`, el modo por defecto de `$0666` en `%edx` y el número de la llamada al sistema en `%eax`. Después de la llamada al sistema, el archivo está abierto y el descriptor de archivo se almacena en `%eax`. [^7] El descriptor de archivo se transfiere entonces a su lugar correspondiente en la pila.

Luego se hace lo mismo para el archivo de salida, excepto que se crea con un modo de solo escritura, crear-si-no-existe, truncar-si-existe. Su descriptor de archivo también se almacena.

Ahora llegamos a la parte principal: el bucle de lectura/escritura. Básicamente, leeremos trozos de datos de tamaño fijo del archivo de entrada, llamaremos a nuestra función de conversión sobre ellos y los escribiremos de nuevo en el archivo de salida. Aunque estamos leyendo trozos de tamaño fijo, el tamaño de los trozos no importa para este programa; simplemente estamos operando sobre secuencias directas de caracteres. Podríamos leerlo en trozos tan pequeños o tan grandes como quisiéramos, y seguiría funcionando correctamente.

La primera parte del bucle es leer los datos. Esto utiliza la llamada al sistema `read`. Esta llamada solo toma un descriptor de archivo del que leer, un búfer en el que escribir y el tamaño del búfer (es decir, el número máximo de bytes que podrían escribirse). La llamada al sistema devuelve el número de bytes realmente leídos o fin de archivo (el número 0). Después de leer un bloque, comprobamos `%eax` buscando un marcador de fin de archivo. Si se encuentra, sale del bucle. De lo contrario, seguimos adelante.

Después de leer los datos, se llama a la función `convert_to_upper` con el búfer que acabamos de leer y el número de caracteres leídos en la llamada al sistema anterior. Después de que esta función se ejecuta, el búfer debería estar en mayúsculas y listo para ser escrito. Los registros se restauran entonces con lo que tenían antes.

Finalmente, emitimos una llamada al sistema `write`, que es exactamente igual que la llamada al sistema `read`, excepto que mueve los datos del búfer al archivo. Ahora simplemente volvemos al principio del bucle. Después de que el bucle sale (recuerda que sale si, tras una lectura, detecta el fin del archivo), simplemente cierra sus descriptores de archivo y sale. La llamada al sistema `close` solo toma el descriptor de archivo a cerrar en `%ebx`. ¡El programa ha terminado entonces!

## Revisión

### Conoce los Conceptos

- Describe el ciclo de vida de un descriptor de archivo.
- ¿Cuáles son los descriptores de archivo estándar y para qué se utilizan?
- ¿Qué es un búfer (buffer)?
- ¿Cuál es la diferencia entre la sección `.data` y la sección `.bss`?
- ¿Cuáles son las llamadas al sistema relacionadas con la lectura y escritura de archivos?

### Usa los Conceptos

- Modifica el programa `toupper` para que lea de STDIN y escriba en STDOUT en lugar de usar los archivos de la línea de comandos.
- Cambia el tamaño del búfer.
- Reescribe el programa para que use almacenamiento en la sección `.bss` en lugar de la pila para almacenar los descriptores de archivo.
- Escribe un programa que cree un archivo llamado `heynow.txt` y escriba las palabras "¡Hey diddle diddle!" en él.

### Yendo más allá

- ¿Qué diferencia supone el tamaño del búfer?
- ¿Qué resultados de error puede devolver cada una de estas llamadas al sistema?
- Haz que el programa sea capaz de operar sobre los argumentos de la línea de comandos o usar STDIN o STDOUT basándose en el número de argumentos de la línea de comandos especificados por ARGC.
- Modifica el programa para que compruebe los resultados de cada llamada al sistema e imprima un mensaje de error en STDOUT cuando ocurra uno.

---

[^1]: Esto se explicará con más detalle en la sección llamada *Verdad, Falsedad y Números Binarios* en el Capítulo 10.
[^2]: Aunque esto suena complicado, la mayoría de las veces en programación no necesitarás tratar directamente con búferes y descriptores de archivo. En el Capítulo 8 aprenderás cómo usar el código existente presente en Linux para manejar la mayoría de las complicaciones de la entrada/salida de archivos por ti.
[^3]: Como mencionamos anteriormente, en Linux, casi todo es un "archivo". Tu entrada de teclado se considera un archivo, y también lo es tu pantalla.
[^4]: *Problem Solving and Programming Concepts* de Maureen Sprankle es un excelente libro sobre el proceso de resolución de problemas aplicado a la programación de computadoras.
[^5]: Esta es una práctica bastante estándar entre los programadores de todos los lenguajes.
[^6]: Si no estás familiarizado con los permisos de UNIX, simplemente pon `$0666` aquí. No olvides el cero inicial, ya que significa que el número es un número octal.
[^7]: Observa que no realizamos ninguna comprobación de errores en esto. Se hace solo para mantener el programa simple. En los programas normales, cada llamada al sistema debería ser normalmente comprobada para ver si tuvo éxito o falló. En caso de fallo, `%eax` contendrá un código de error en lugar de un valor de retorno. Los códigos de error son negativos, por lo que pueden detectarse comparando `%eax` con cero y saltando si es menor que cero.
