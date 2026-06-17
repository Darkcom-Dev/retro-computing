# Capítulo 6. Lectura y Escritura de Registros Simples

Como se mencionó en el Capítulo 5, muchas aplicaciones tratan con datos que son persistentes, lo que significa que los datos viven más tiempo que el programa al ser almacenados en el disco en archivos. Puedes cerrar el programa y volver a abrirlo, y estarás de vuelta donde empezaste. Ahora bien, existen dos tipos básicos de datos persistentes: estructurados y no estructurados. Los datos no estructurados son como los que tratamos en el programa `toupper`. Simplemente trataba con archivos de texto que fueron introducidos por una persona. El contenido de los archivos no era utilizable por un programa porque un programa no puede interpretar lo que el usuario intenta decir en un texto aleatorio.

Los datos estructurados, por otro lado, son en lo que las computadoras sobresalen. Los **datos estructurados** (structured data) son datos que se dividen en campos y registros. En su mayor parte, los campos y registros son de longitud fija. Debido a que los datos se dividen en registros de longitud fija y campos de formato fijo, la computadora puede interpretar los datos. Los datos estructurados pueden contener campos de longitud variable, pero en ese punto suele ser mejor utilizar una base de datos. [^1]

Este capítulo trata sobre la lectura y escritura de registros simples de longitud fija. Digamos que queremos almacenar información básica sobre personas que conocemos. Podríamos imaginar el siguiente ejemplo de registro de longitud fija sobre personas:

- Nombre (Firstname) - 40 bytes
- Apellido (Lastname) - 40 bytes
- Dirección (Address) - 240 bytes
- Edad (Age) - 4 bytes

En esto, todo son datos de caracteres excepto la edad, que es simplemente un campo numérico que utiliza una palabra estándar de 4 bytes (podríamos usar un solo byte para esto, pero mantenerlo como una palabra facilita el procesamiento).

En programación, a menudo tienes ciertas definiciones que usarás una y otra vez dentro del programa, o quizás dentro de varios programas. Es bueno separar estas definiciones en archivos que simplemente se incluyan en los archivos de lenguaje ensamblador según sea necesario. Por ejemplo, en nuestros próximos programas necesitaremos acceder a las diferentes partes del registro anterior. Esto significa que necesitamos conocer los desplazamientos (offsets) de cada campo desde el principio del registro para acceder a ellos utilizando el direccionamiento por puntero base. Las siguientes constantes describen los desplazamientos de la estructura anterior. Ponlas en un archivo llamado `record-def.s`:

```assembly
.equ RECORD_FIRSTNAME, 0
.equ RECORD_LASTNAME, 40
.equ RECORD_ADDRESS, 80
.equ RECORD_AGE, 320
.equ RECORD_SIZE, 324
```

Además, hay varias constantes que hemos estado definiendo una y otra vez en nuestros programas, y es útil ponerlas en un archivo para no tener que estar introduciéndolas constantemente. Pon las siguientes constantes en un archivo llamado `linux.s`:

```assembly
#Definiciones comunes de Linux
#Números de llamadas al sistema
.equ SYS_EXIT, 1
.equ SYS_READ, 3
.equ SYS_WRITE, 4
.equ SYS_OPEN, 5
.equ SYS_CLOSE, 6
.equ SYS_BRK, 45

#Número de interrupción de llamada al sistema
.equ LINUX_SYSCALL, 0x80

#Descriptores de archivo estándar
.equ STDIN, 0
.equ STDOUT, 1
.equ STDERR, 2

#Códigos de estado comunes
.equ END_OF_FILE, 0
```

Escribiremos tres programas en este capítulo utilizando la estructura definida en `record-def.s`. El primer programa construirá un archivo que contenga varios registros como los definidos anteriormente. El segundo programa mostrará los registros del archivo. El tercer programa sumará 1 año a la edad de cada registro.

Además de las constantes estándar que utilizaremos en todos los programas, también hay dos funciones que usaremos en varios de los programas: una que lee un registro y otra que escribe un registro. ¿Qué parámetros necesitan estas funciones para operar? Básicamente necesitamos:

- La ubicación de un búfer en el que podamos leer un registro
- El descriptor de archivo del que queremos leer o en el que queremos escribir

Veamos primero nuestra función de lectura:

```assembly
.include "record-def.s"
.include "linux.s"

#PROPÓSITO:    Esta función lee un registro desde el
#              descriptor de archivo
#
#ENTRADA:      El descriptor de archivo y un búfer
#
#SALIDA:       Esta función escribe los datos en el búfer
#              y devuelve un código de estado.
#
#VARIABLES LOCALES DE LA PILA
.equ ST_READ_BUFFER, 8
.equ ST_FILEDES, 12

.section .text
.globl read_record
.type read_record, @function
read_record:
 pushl %ebp
 movl  %esp, %ebp

 pushl %ebx
 movl  ST_FILEDES(%ebp), %ebx
 movl  ST_READ_BUFFER(%ebp), %ecx
 movl  $RECORD_SIZE, %edx
 movl  $SYS_READ, %eax
 int   $LINUX_SYSCALL

 #NOTA: %eax tiene el valor de retorno, que daremos
 #      de vuelta a nuestro programa llamador
 popl  %ebx

 movl  %ebp, %esp
 popl  %ebp
 ret
```

Es una función bastante simple. Simplemente lee datos del tamaño de nuestra estructura en un búfer del tamaño adecuado desde el descriptor de archivo dado. La de escritura es similar:

```assembly
.include "linux.s"
.include "record-def.s"

#PROPÓSITO:    Esta función escribe un registro en
#              el descriptor de archivo dado
#
#ENTRADA:      El descriptor de archivo y un búfer
#
#SALIDA:       Esta función produce un código de estado
#
#VARIABLES LOCALES DE LA PILA
.equ ST_WRITE_BUFFER, 8
.equ ST_FILEDES, 12

.section .text
.globl write_record
.type write_record, @function
write_record:
 pushl %ebp
 movl  %esp, %ebp

 pushl %ebx
 movl  $SYS_WRITE, %eax
 movl  ST_FILEDES(%ebp), %ebx
 movl  ST_WRITE_BUFFER(%ebp), %ecx
 movl  $RECORD_SIZE, %edx
 int   $LINUX_SYSCALL

 #NOTA: %eax tiene el valor de retorno, que daremos
 #      de vuelta a nuestro programa llamador
 popl  %ebx

 movl  %ebp, %esp
 popl  %ebp
 ret
```

Ahora que tenemos nuestras definiciones básicas, estamos listos para escribir nuestros programas.

## Escribiendo Registros

Este programa simplemente escribirá algunos registros codificados directamente al disco. Hará lo siguiente:

- Abrir el archivo
- Escribir tres registros
- Cerrar el archivo

Escribe el siguiente código en un archivo llamado `write-records.s`:

```assembly
.include "linux.s"
.include "record-def.s"

.section .data

#Datos constantes de los registros que queremos escribir
#Cada elemento de datos de texto se rellena a la longitud
#adecuada con bytes nulos (es decir, 0).

#.rept se utiliza para rellenar cada elemento. .rept le dice
#al ensamblador que repita la sección entre
#.rept y .endr el número de veces especificado.
#Esto se usa en este programa para añadir caracteres nulos
#adicionales al final de cada campo para rellenarlo
record1:
 .ascii "Fredrick\0"
 .rept 31 #Relleno hasta 40 bytes
 .byte 0
 .endr

 .ascii "Bartlett\0"
 .rept 31 #Relleno hasta 40 bytes
 .byte 0
 .endr

 .ascii "4242 S Prairie\nTulsa, OK 55555\0"
 .rept 209 #Relleno hasta 240 bytes
 .byte 0
 .endr

 .long 45

record2:
 .ascii "Marilyn\0"
 .rept 32 #Relleno hasta 40 bytes
 .byte 0
 .endr

 .ascii "Taylor\0"
 .rept 33 #Relleno hasta 40 bytes
 .byte 0
 .endr

 .ascii "2224 S Johannan St\nChicago, IL 12345\0"
 .rept 203 #Relleno hasta 240 bytes
 .byte 0
 .endr

 .long 29

record3:
 .ascii "Derrick\0"
 .rept 32 #Relleno hasta 40 bytes
 .byte 0
 .endr

 .ascii "McIntire\0"
 .rept 31 #Relleno hasta 40 bytes
 .byte 0
 .endr

 .ascii "500 W Oakland\nSan Diego, CA 54321\0"
 .rept 206 #Relleno hasta 240 bytes
 .byte 0
 .endr

 .long 36

#Este es el nombre del archivo en el que escribiremos
file_name:
 .ascii "test.dat\0"

.equ ST_FILE_DESCRIPTOR, -4

.globl _start
_start:
 #Copiar el puntero de pila a %ebp
 movl %esp, %ebp
 #Asignar espacio para mantener el descriptor de archivo
 subl $4, %esp

 #Abrir el archivo
 movl $SYS_OPEN, %eax
 movl $file_name, %ebx
 movl $0101, %ecx #Esto indica crear si no
                  #existe, y abrir para
                  #escritura
 movl $0666, %edx
 int  $LINUX_SYSCALL

 #Guardar el descriptor de archivo
 movl %eax, ST_FILE_DESCRIPTOR(%ebp)

 #Escribir el primer registro
 pushl ST_FILE_DESCRIPTOR(%ebp)
 pushl $record1
 call  write_record
 addl  $8, %esp

 #Escribir el segundo registro
 pushl ST_FILE_DESCRIPTOR(%ebp)
 pushl $record2
 call  write_record
 addl  $8, %esp

 #Escribir el tercer registro
 pushl ST_FILE_DESCRIPTOR(%ebp)
 pushl $record3
 call  write_record
 addl  $8, %esp

 #Cerrar el descriptor de archivo
 movl $SYS_CLOSE, %eax
 movl ST_FILE_DESCRIPTOR(%ebp), %ebx
 int  $LINUX_SYSCALL

 #Salir del programa
 movl $SYS_EXIT, %eax
 movl $0, %ebx
 int  $LINUX_SYSCALL
```

Este es un programa bastante sencillo. Consiste meramente en definir los datos que queremos escribir en la sección `.data` y luego llamar a las llamadas al sistema y llamadas a funciones adecuadas para lograrlo. Para un repaso de todas las llamadas al sistema utilizadas, consulta el Apéndice C.

Habrás notado las líneas:

```assembly
.include "linux.s"
.include "record-def.s"
```

Estas sentencias hacen que los archivos dados se peguen básicamente allí mismo en el código. No necesitas hacer esto con las funciones, porque el enlazador puede encargarse de combinar las funciones exportadas con `.globl`. Sin embargo, las constantes definidas en otro archivo sí deben ser importadas de esta manera.

Además, habrás notado el uso de una nueva directiva del ensamblador, `.rept`. Esta directiva repite el contenido del archivo entre las directivas `.rept` y `.endr` el número de veces especificado después de `.rept`. Normalmente se usa de la forma en que la usamos nosotros: para rellenar valores en la sección `.data`. En nuestro caso, estamos añadiendo caracteres nulos al final de cada campo hasta que alcancen sus longitudes definidas.

Para construir la aplicación, ejecuta los comandos:

```bash
as write-records.s -o write-records.o
as write-record.s -o write-record.o
ld write-record.o write-records.o -o write-records
```

Aquí estamos ensamblando dos archivos por separado y luego combinándolos mediante el enlazador. Para ejecutar el programa, simplemente escribe lo siguiente:

```bash
./write-records
```

Esto hará que se cree un archivo llamado `test.dat` que contiene los registros. Sin embargo, como contienen caracteres no imprimibles (específicamente el carácter nulo), puede que no sean visibles con un editor de texto. Por lo tanto, necesitamos el siguiente programa para que los lea por nosotros.

## Lectura de Registros

Ahora consideraremos el proceso de lectura de registros. En este programa, leeremos cada registro y mostraremos el primer nombre listado en cada uno. Como el nombre de cada persona tiene una longitud diferente, necesitaremos una función para contar el número de caracteres que queremos escribir. Dado que rellenamos cada campo con caracteres nulos, simplemente podemos contar caracteres hasta llegar a un carácter nulo. [^2] Ten en cuenta que esto significa que nuestros registros deben contener al menos un carácter nulo cada uno.

Aquí está el código. Ponlo en un archivo llamado `count-chars.s`:

```assembly
#PROPÓSITO:  Contar los caracteres hasta que se alcance un byte nulo.
#
#ENTRADA:    La dirección de la cadena de caracteres
#
#SALIDA:     Devuelve el conteo en %eax
#
#PROCESO:
#  Registros utilizados:
#    %ecx - conteo de caracteres
#    %al - carácter actual
#    %edx - dirección del carácter actual

.type count_chars, @function
.globl count_chars

#Aquí es donde está nuestro único parámetro en la pila
.equ ST_STRING_START_ADDRESS, 8

count_chars:
 pushl %ebp
 movl  %esp, %ebp

 #El contador empieza en cero
 movl  $0, %ecx
 #Dirección inicial de los datos
 movl  ST_STRING_START_ADDRESS(%ebp), %edx

count_loop_begin:
 #Obtener el carácter actual
 movb  (%edx), %al
 #¿Es nulo?
 cmpb  $0, %al
 #Si es sí, hemos terminado
 je    count_loop_end
 #De lo contrario, incrementar el contador y el puntero
 incl  %ecx
 incl  %edx
 #Volver al principio del bucle
 jmp   count_loop_begin

count_loop_end:
 #Hemos terminado. Mover el conteo a %eax
 #y retornar.
 movl  %ecx, %eax
 popl  %ebp
 ret
```

Como puedes ver, es una función bastante directa. Simplemente recorre los bytes en un bucle, contando a medida que avanza, hasta que encuentra un carácter nulo. Luego devuelve el conteo.

Nuestro programa de lectura de registros también será bastante directo. Hará lo siguiente:

- Abrir el archivo
- Intentar leer un registro
- Si estamos al final del archivo, salir
- De lo contrario, contar los caracteres del primer nombre
- Escribir el primer nombre en STDOUT
- Escribir una nueva línea en STDOUT
- Volver a leer otro registro

Para escribir esto, necesitamos una función simple más: una función para escribir una nueva línea en STDOUT. Pon el siguiente código en `write-newline.s`:

```assembly
.include "linux.s"

.globl write_newline
.type  write_newline, @function

.section .data
newline:
 .ascii "\n"

.section .text
.equ ST_FILEDES, 8
write_newline:
 pushl %ebp
 movl  %esp, %ebp

 movl  $SYS_WRITE, %eax
 movl  ST_FILEDES(%ebp), %ebx
 movl  $newline, %ecx
 movl  $1, %edx
 int   $LINUX_SYSCALL

 movl  %ebp, %esp
 popl  %ebp
 ret
```

Ahora estamos listos para escribir el programa principal. Aquí está el código de `read-records.s`:

```assembly
.include "linux.s"
.include "record-def.s"

.section .data
file_name:
 .ascii "test.dat\0"

.section .bss
.lcomm record_buffer, RECORD_SIZE

.section .text
#Programa principal
.globl _start
_start:
 #Estas son las ubicaciones en la pila donde
 #almacenaremos los descriptores de entrada y salida
 #(Para tu información, podríamos haber usado direcciones de memoria en
 #una sección .data en su lugar)
 .equ ST_INPUT_DESCRIPTOR, -4
 .equ ST_OUTPUT_DESCRIPTOR, -8

 #Copiar el puntero de pila a %ebp
 movl %esp, %ebp
 #Asignar espacio para mantener los descriptores de archivo
 subl $8, %esp

 #Abrir el archivo
 movl $SYS_OPEN, %eax
 movl $file_name, %ebx
 movl $0, %ecx    #Esto indica abrir en modo solo lectura
 movl $0666, %edx
 int  $LINUX_SYSCALL

 #Guardar el descriptor de archivo
 movl %eax, ST_INPUT_DESCRIPTOR(%ebp)

 #Aunque es una constante, estamos
 #guardando el descriptor del archivo de salida en
 #una variable local para que, si más adelante
 #decidimos que no siempre va a
 #ser STDOUT, podamos cambiarlo fácilmente.
 movl $STDOUT, ST_OUTPUT_DESCRIPTOR(%ebp)

record_read_loop:
 pushl ST_INPUT_DESCRIPTOR(%ebp)
 pushl $record_buffer
 call  read_record
 addl  $8, %esp

 #Devuelve el número de bytes leídos.
 #Si no es el mismo número que
 #solicitamos, entonces es o bien un
 #fin de archivo, o un error, así que
 #terminamos
 cmpl  $RECORD_SIZE, %eax
 jne   finished_reading

 #De lo contrario, imprimir el primer nombre
 #pero primero, debemos conocer su tamaño
 pushl $RECORD_FIRSTNAME + record_buffer
 call  count_chars
 addl  $4, %esp

 movl  %eax, %edx
 movl  ST_OUTPUT_DESCRIPTOR(%ebp), %ebx
 movl  $SYS_WRITE, %eax
 movl  $RECORD_FIRSTNAME + record_buffer, %ecx
 int   $LINUX_SYSCALL

 pushl ST_OUTPUT_DESCRIPTOR(%ebp)
 call  write_newline
 addl  $4, %esp

 jmp   record_read_loop

finished_reading:
 movl  $SYS_EXIT, %eax
 movl  $0, %ebx
 int   $LINUX_SYSCALL
```

Para construir este programa, necesitamos ensamblar todas las partes y enlazarlas juntas:

```bash
as read-record.s -o read-record.o
as count-chars.s -o count-chars.o
as write-newline.s -o write-newline.o
as read-records.s -o read-records.o
ld read-record.o count-chars.o write-newline.o \
   read-records.o -o read-records
```

La barra invertida en la penúltima línea simplemente significa que el comando continúa en la siguiente línea. Puedes ejecutar tu programa haciendo `./read-records`.

Como puedes ver, este programa abre el archivo y luego ejecuta un bucle de lectura, comprobación de fin de archivo y escritura del nombre. El único constructor que podría ser nuevo es la línea que dice:

```assembly
pushl $RECORD_FIRSTNAME + record_buffer
```

Parece que estamos combinando una instrucción de suma con una de apilamiento (push), pero no es así. Verás, tanto `RECORD_FIRSTNAME` como `record_buffer` son constantes. La primera es una constante directa, creada mediante el uso de una directiva `.equ`, mientras que la segunda es definida automáticamente por el ensamblador mediante su uso como etiqueta (siendo su valor la dirección donde comenzarán los datos que la siguen). Como ambas son constantes que el ensamblador conoce, es capaz de sumarlas mientras está ensamblando tu programa, por lo que toda la instrucción es un único push en modo inmediato de una única constante. La constante `RECORD_FIRSTNAME` es el número de bytes tras el comienzo de un registro antes de llegar al primer nombre. `record_buffer` es el nombre de nuestro búfer para guardar registros. Sumarlos nos da la dirección del miembro primer nombre del registro almacenado en `record_buffer`.

## Modificando los Registros

En esta sección, escribiremos un programa que:

- Abre un archivo de entrada y otro de salida
- Lee registros de la entrada
- Incrementa la edad
- Escribe el nuevo registro en el archivo de salida

Como la mayoría de los programas que hemos encontrado recientemente, este programa es bastante directo. [^3]

```assembly
.include "linux.s"
.include "record-def.s"

.section .data
input_file_name:
 .ascii "test.dat\0"
output_file_name:
 .ascii "testout.dat\0"

.section .bss
.lcomm record_buffer, RECORD_SIZE

#Desplazamientos en la pila de las variables locales
.equ ST_INPUT_DESCRIPTOR, -4
.equ ST_OUTPUT_DESCRIPTOR, -8

.section .text
.globl _start
_start:
 #Copiar puntero de pila y hacer espacio para variables locales
 movl %esp, %ebp
 subl $8, %esp

 #Abrir archivo para lectura
 movl $SYS_OPEN, %eax
 movl $input_file_name, %ebx
 movl $0, %ecx
 movl $0666, %edx
 int  $LINUX_SYSCALL

 movl %eax, ST_INPUT_DESCRIPTOR(%ebp)

 #Abrir archivo para escritura
 movl $SYS_OPEN, %eax
 movl $output_file_name, %ebx
 movl $0101, %ecx
 movl $0666, %edx
 int  $LINUX_SYSCALL

 movl %eax, ST_OUTPUT_DESCRIPTOR(%ebp)

loop_begin:
 pushl ST_INPUT_DESCRIPTOR(%ebp)
 pushl $record_buffer
 call  read_record
 addl  $8, %esp

 #Devuelve el número de bytes leídos.
 #Si no es el mismo número que
 #solicitamos, entonces es o bien un
 #fin de archivo, o un error, así que
 #terminamos
 cmpl  $RECORD_SIZE, %eax
 jne   loop_end

 #Incrementar la edad
 incl  record_buffer + RECORD_AGE

 #Escribir el registro
 pushl ST_OUTPUT_DESCRIPTOR(%ebp)
 pushl $record_buffer
 call  write_record
 addl  $8, %esp

 jmp   loop_begin

loop_end:
 movl  $SYS_EXIT, %eax
 movl  $0, %ebx
 int   $LINUX_SYSCALL
```

Puedes escribirlo como `add-year.s`. Para construirlo, escribe lo siguiente [^4]:

```bash
as add-year.s -o add-year.o
ld add-year.o read-record.o write-record.o -o add-year
```

Para ejecutar el programa, simplemente escribe lo siguiente [^5]:

```bash
./add-year
```

Esto sumará un año a cada registro listado en `test.dat` y escribirá los nuevos registros en el archivo `testout.dat`.

Como puedes ver, escribir registros de longitud fija es bastante simple. Solo tienes que leer bloques de datos en un búfer, procesarlos y volver a escribirlos. Desafortunadamente, este programa no escribe las nuevas edades en la pantalla para que puedas verificar la efectividad de tu programa. Esto se debe a que no llegaremos a mostrar números hasta el Capítulo 8 y el Capítulo 10. Después de leer esos, es posible que quieras volver y reescribir este programa para mostrar los datos numéricos que estamos modificando.

## Revisión

### Conoce los Conceptos

- ¿Qué es un registro (record)?
- ¿Cuál es la ventaja de los registros de longitud fija sobre los registros de longitud variable?
- ¿Cómo incluyes constantes en múltiples archivos fuente de ensamblador?
- ¿Por qué querrías dividir un proyecto en múltiples archivos fuente?
- ¿Qué hace la instrucción `incl record_buffer + RECORD_AGE`? ¿Qué modo de direccionamiento está utilizando? ¿Cuántos operandos tiene la instrucción `incl` en este caso? ¿Qué partes están siendo manejadas por el ensamblador y qué partes se manejan cuando se ejecuta el programa?

### Usa los Conceptos

- Añade otro miembro de datos a la estructura de persona definida en este capítulo y reescribe las funciones y programas de lectura y escritura para tenerlos en cuenta. Recuerda volver a ensamblar y enlazar tus archivos antes de ejecutar tus programas.
- Crea un programa que use un bucle para escribir 30 registros idénticos en un archivo.
- Crea un programa para encontrar la edad más grande en el archivo y devolver esa edad como el código de estado del programa.
- Crea un programa para encontrar la edad más pequeña en el archivo y devolver esa edad como el código de estado del programa.

### Yendo más allá

- Reescribe los programas de este capítulo para usar argumentos de la línea de comandos para especificar los nombres de archivo.
- Investiga la llamada al sistema `lseek`. Reescribe el programa `add-year` para abrir el archivo de origen tanto para lectura como para escritura (usa `$2` para el modo lectura/escritura) y escribe los registros modificados de nuevo en el mismo archivo del que fueron leídos.
- Investiga los diversos códigos de error que pueden devolver las llamadas al sistema realizadas en estos programas. Elige uno para reescribir y añade código que compruebe `%eax` en busca de condiciones de error y, si se encuentra una, escriba un mensaje al respecto en STDERR y salga.
- Escribe un programa que añada un único registro al archivo leyendo los datos desde el teclado. Recuerda, tendrás que asegurarte de que los datos tengan al menos un carácter nulo al final, y necesitas tener una forma de que el usuario indique que ha terminado de escribir. Debido a que no hemos entrado en la conversión de caracteres a números, no podrás leer la edad desde el teclado, así que tendrás que tener una edad por defecto.
- Escribe una función llamada `compare-strings` que compare dos cadenas de hasta 5 caracteres. Luego escribe un programa que permita al usuario introducir 5 caracteres y haga que el programa devuelva todos los registros cuyo primer nombre comience con esos 5 caracteres.

---

[^1]: Una base de datos es un programa que maneja datos estructurados persistentes por ti. No tienes que escribir los programas para leer y escribir los datos en el disco, para realizar búsquedas o incluso para realizar un procesamiento básico. Es una interfaz de muy alto nivel para datos estructurados que, aunque añade algo de sobrecarga y complejidad adicional, es muy útil para tareas complejas de procesamiento de datos. En el Capítulo 13 se enumeran referencias para aprender cómo funcionan las bases de datos.
[^2]: Si has usado C, esto es lo que hace la función `strlen`.
[^3]: Descubrirás que después de aprender la mecánica de la programación, la mayoría de los programas son bastante directos una vez que sabes exactamente qué es lo que quieres hacer. La mayoría de ellos inicializan datos, realizan algún procesamiento en un bucle y luego limpian todo.
[^4]: Esto asume que ya has construido los archivos objeto `read-record.o` y `write-record.o` en los ejemplos anteriores. Si no es así, tendrás que hacerlo.
[^5]: Esto asume que creaste el archivo en una ejecución previa de `write-records`. Si no es así, necesitas ejecutar `write-records` primero antes de ejecutar este programa.
