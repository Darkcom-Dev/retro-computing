# Capítulo 8. Compartir Funciones con Bibliotecas de Código

A estas alturas ya deberías darte cuenta de que la computadora tiene que trabajar mucho incluso para tareas simples. Debido a eso, tú tienes que trabajar mucho para escribir el código para que una computadora realice incluso tareas sencillas. Además, las tareas de programación no suelen ser muy simples. Por lo tanto, necesitamos una forma de facilitarnos este proceso. Hay varias formas de hacerlo, entre ellas:

- Escribir código en un lenguaje de alto nivel en lugar de en lenguaje ensamblador.
- Tener gran cantidad de código ya escrito que puedas copiar y pegar en tus propios programas.
- Tener un conjunto de funciones en el sistema que sean compartidas entre cualquier programa que desee utilizarlas.

Normalmente, las tres se utilizan en cierta medida en cualquier proyecto dado. La primera opción se explorará más a fondo en el Capítulo 11. La segunda opción es útil pero tiene algunos inconvenientes, entre ellos:

- El código que se copia a menudo tiene que ser modificado sustancialmente para que encaje con el código circundante.
- Cada programa que contiene el código copiado tiene el mismo código en su interior, desperdiciando así mucho espacio.
- Si se encuentra un bug en cualquier parte del código copiado, tiene que ser corregido en cada programa de aplicación.

Por lo tanto, la segunda opción suele utilizarse con moderación. Normalmente solo se usa en casos en los que se copia y pega código de esqueleto para un tipo específico de tarea y se añaden los detalles específicos del programa. La tercera opción es la que se utiliza más a menudo. La tercera opción incluye tener un repositorio central de código compartido. Entonces, en lugar de que cada programa desperdicie espacio almacenando las mismas copias de las funciones, simplemente pueden apuntar a las bibliotecas compartidas que contienen las funciones que necesitan. Si se encuentra un bug en una de estas funciones, solo tiene que corregirse dentro del archivo de la biblioteca de funciones individual, y todas las aplicaciones que la usan se actualizan automáticamente. El principal inconveniente de este enfoque es que crea algunos problemas de dependencia, entre ellos:

- Si varias aplicaciones están usando el archivo compartido, ¿cómo sabemos cuándo es seguro eliminar el archivo? Por ejemplo, si tres aplicaciones comparten un archivo de funciones y se eliminan 2 de los programas, ¿cómo sabe el sistema que todavía existe una aplicación que utiliza ese código y que, por lo tanto, no debe eliminarse?
- Algunos programas dependen inadvertidamente de bugs dentro de las funciones compartidas. Por lo tanto, si la actualización del programa compartido corrige un bug del que dependía un programa, podría causar que esa aplicación deje de funcionar.

Estos problemas son los que conducen a lo que se conoce como el "infierno de las DLL" (DLL hell). Sin embargo, generalmente se asume que las ventajas superan a las desventajas.

En programación, estos archivos de código compartido se denominan bibliotecas compartidas (shared libraries), objetos compartidos (shared objects), bibliotecas de enlace dinámico (dynamic-link libraries), DLL o archivos `.so`. Nos referiremos a ellos como **bibliotecas compartidas**.

## Uso de una Biblioteca Compartida

El programa que examinaremos aquí es simple: escribe los caracteres "hello world" en la pantalla y sale. El programa normal, `helloworld-nolib.s`, se ve así:

```assembly
#PROPÓSITO:    Este programa escribe el mensaje "hello world" y
#              sale
#

.include "linux.s"

.section .data
helloworld:
 .ascii "hello world\n"
helloworld_end:
 .equ helloworld_len, helloworld_end - helloworld

.section .text
.globl _start
_start:
 movl $STDOUT, %ebx
 movl $helloworld, %ecx
 movl $helloworld_len, %edx
 movl $SYS_WRITE, %eax
 int  $LINUX_SYSCALL

 movl $0, %ebx
 movl $SYS_EXIT, %eax
 int  $LINUX_SYSCALL
```

No es demasiado largo. Sin embargo, mira qué corto es `helloworld-lib.s` utilizando una biblioteca:

```assembly
#PROPÓSITO:    Este programa escribe el mensaje "hello world" y
#              sale
#

.section .data
helloworld:
 .ascii "hello world\n\0"

.section .text
.globl _start
_start:
 pushl $helloworld
 call  printf

 pushl $0
 call  exit
```

¡Es incluso más corto!

Ahora bien, construir programas que utilizan bibliotecas compartidas es un poco diferente de lo normal. Puedes construir el primer programa normalmente haciendo esto:

```bash
as helloworld-nolib.s -o helloworld-nolib.o
ld helloworld-nolib.o -o helloworld-nolib
```

Sin embargo, para construir el segundo programa, tienes que hacer esto:

```bash
as helloworld-lib.s -o helloworld-lib.o
ld -dynamic-linker /lib/ld-linux.so.2 \
   -o helloworld-lib helloworld-lib.o -lc
```

Recuerda que la barra invertida en la primera línea simplemente significa que el comando continúa en la línea siguiente. La opción `-dynamic-linker /lib/ld-linux.so.2` permite que nuestro programa se enlace a bibliotecas. Esto construye el ejecutable de modo que, antes de ejecutarse, el sistema operativo cargará el programa `/lib/ld-linux.so.2` para cargar las bibliotecas externas y enlazarlas con el programa. Este programa se conoce como **enlazador dinámico** (dynamic linker).

La opción `-lc` indica que se enlace a la *biblioteca c*, llamada `libc.so` en sistemas GNU/Linux. Dado el nombre de una biblioteca, `c` en este caso (normalmente los nombres de las bibliotecas son más largos que una sola letra), el enlazador de GNU/Linux antepone la cadena `lib` al principio del nombre de la biblioteca y le añade `.so` al final para formar el nombre de archivo de la biblioteca. Esta biblioteca contiene muchas funciones para automatizar todo tipo de tareas. Las dos que estamos usando son `printf`, que imprime cadenas, y `exit`, que sale del programa.

Observa que los símbolos `printf` y `exit` simplemente se mencionan por su nombre dentro del programa. En capítulos anteriores, el enlazador resolvía todos los nombres a direcciones de memoria física y los nombres se descartaban. Cuando se utiliza el enlace dinámico, el nombre mismo reside dentro del ejecutable y es resuelto por el enlazador dinámico cuando se ejecuta. Cuando el usuario ejecuta el programa, el enlazador dinámico carga las bibliotecas compartidas enumeradas en nuestra sentencia de enlace y, a continuación, busca todos los nombres de funciones y variables que fueron nombrados por nuestro programa pero que no se encontraron en el momento del enlace, y los empareja con las entradas correspondientes en las bibliotecas compartidas que carga. Luego sustituye todos los nombres por las direcciones en las que se cargan. Esto suena a que consume mucho tiempo. Lo hace en una pequeña medida, pero solo ocurre una vez: en el momento del inicio del programa.

## Cómo Funcionan las Bibliotecas Compartidas

En nuestros primeros programas, todo el código estaba contenido en el archivo fuente. Tales programas se denominan **ejecutables enlazados estáticamente** (statically-linked executables), porque contenían toda la funcionalidad necesaria para el programa que no era manejada por el kernel. En los programas que escribimos en el Capítulo 6, utilizamos tanto nuestro archivo de programa principal como archivos que contenían rutinas utilizadas por múltiples programas. En estos casos, combinamos todo el código utilizando el enlazador en el momento del enlace, por lo que seguía estando enlazado estáticamente.

Sin embargo, en el programa `helloworld-lib`, empezamos a utilizar bibliotecas compartidas. Cuando utilizas bibliotecas compartidas, tu programa está entonces **enlazado dinámicamente** (dynamically-linked), lo que significa que no todo el código necesario para ejecutar el programa está realmente contenido en el archivo del programa en sí, sino en bibliotecas externas.

Cuando pusimos el `-lc` en el comando para enlazar el programa `helloworld`, le indicamos al enlazador que utilizara la biblioteca c (`libc.so`) para buscar cualquier símbolo que no estuviera ya definido en `helloworld.o`. Sin embargo, en realidad no añade ningún código a nuestro programa, solo anota en el programa dónde buscar. Cuando comienza el programa `helloworld`, el archivo `/lib/ld-linux.so.2` se carga primero. Este es el enlazador dinámico. Este mira nuestro programa `helloworld` y ve que necesita la biblioteca c para ejecutarse. Por lo tanto, busca un archivo llamado `libc.so` en los lugares estándar (enumerados en `/etc/ld.so.conf` y en el contenido de la variable de entorno `LD_LIBRARY_PATH`), luego busca en él todos los símbolos necesarios (`printf` y `exit` en este caso) y, finalmente, carga la biblioteca en la memoria virtual del programa. Por último, sustituye todas las instancias de `printf` en el programa por la ubicación real de `printf` en la biblioteca.

Ejecuta el siguiente comando:

```bash
ldd ./helloworld-nolib
```

Debería informar `not a dynamic executable` (no es un ejecutable dinámico). Esto es justo lo que dijimos: `helloworld-nolib` es un ejecutable enlazado estáticamente. Sin embargo, prueba esto:

```bash
ldd ./helloworld-lib
```

Informará algo como:

```text
libc.so.6 => /lib/libc.so.6 (0x4001d000)
/lib/ld-linux.so.2 => /lib/ld-linux.so.2 (0x400000000)
```

Los números entre paréntesis pueden ser diferentes en tu sistema. Esto significa que el programa `helloworld` está enlazado con `libc.so.6` (el `.6` es el número de versión), que se encuentra en `/lib/libc.so.6`, y `/lib/ld-linux.so.2` se encuentra en `/lib/ld-linux.so.2`. Estas bibliotecas tienen que cargarse antes de que se pueda ejecutar el programa. Si tienes interés, ejecuta el programa `ldd` en varios programas que estén en tu distribución de Linux y observa de qué bibliotecas dependen.

## Búsqueda de Información sobre las Bibliotecas

Bien, ahora que ya conoces las bibliotecas, la pregunta es: ¿cómo averiguas qué bibliotecas tienes en tu sistema y qué hacen? Bueno, vamos a saltarnos esa pregunta por un momento y hagamos otra: ¿Cómo se describen los programadores las funciones entre sí en su documentación? Echemos un vistazo a la función `printf`. Su interfaz de llamada (normalmente denominada **prototipo** o prototype) se ve así:

```c
int printf(char *string, ...);
```

En Linux, las funciones se describen en el lenguaje de programación C. De hecho, la mayoría de los programas de Linux están escritos en C. Por eso, la mayor parte de la documentación y la compatibilidad binaria se definen utilizando el lenguaje C. La interfaz de la función `printf` anterior se describe utilizando el lenguaje de programación C.

Esta definición significa que existe una función `printf`. Las cosas dentro de los paréntesis son los parámetros o argumentos de la función. El primer parámetro aquí es `char *string`. Esto significa que hay un parámetro llamado `string` (el nombre no es importante, excepto para hablar de él), que tiene un tipo `char *`. `char` significa que quiere un carácter de un solo byte. El `*` después de él significa que en realidad no quiere un carácter como argumento, sino que quiere la dirección de un carácter o secuencia de caracteres. Si vuelves a mirar nuestro programa `helloworld`, notarás que la llamada a la función se veía así:

```assembly
pushl $helloworld
call  printf
```

Así que introdujimos en la pila la dirección de la cadena `helloworld`, en lugar de los caracteres reales. Te habrás dado cuenta de que no introdujimos la longitud de la cadena. La forma en que `printf` encontró el final de la cadena fue porque la terminamos con un carácter nulo (`\0`). Muchas funciones funcionan así, especialmente las funciones del lenguaje C. El `int` antes de la definición de la función indica qué tipo de valor devolverá la función en `%eax` cuando retorne. `printf` devolverá un `int` cuando termine. Ahora, después del `char *string`, tenemos una serie de puntos, `...`. Esto significa que puede aceptar un número indefinido de argumentos adicionales después de la cadena. La mayoría de las funciones solo pueden aceptar un número específico de argumentos. `printf`, sin embargo, puede aceptar muchos. Mirará dentro del parámetro string y, en cada lugar donde vea los caracteres `%s`, buscará otra cadena de la pila para insertar, y en cada lugar donde vea `%d` buscará un número de la pila para insertar. Esto se describe mejor con un ejemplo:

```assembly
#PROPÓSITO:    Este programa es para demostrar cómo llamar a printf
#

.section .data

#Esta cadena se llama cadena de formato (format string). Es el primer
#parámetro, y printf la utiliza para averiguar cuántos parámetros
#se le pasaron y de qué tipo son.
firststring:
 .ascii "Hello! %s is a %s who loves the number %d\n\0"

name:
 .ascii "Jonathan\0"

personstring:
 .ascii "person\0"

#Esto también podría haber sido un .equ, pero decidimos darle una
#ubicación de memoria real solo por diversión
numberloved:
 .long 3

.section .text
.globl _start
_start:
 #ten en cuenta que los parámetros se pasan en el
 #orden inverso al que aparecen en el
 #prototipo de la función.
 pushl numberloved      #Este es el %d
 pushl $personstring    #Este es el segundo %s
 pushl $name            #Este es el primero %s
 pushl $firststring     #Esta es la cadena de formato
                        #en el prototipo
 call  printf

 pushl $0
 call  exit
```

Escríbelo con el nombre de archivo `printf-example.s` y luego ejecuta los siguientes comandos:

```bash
as printf-example.s -o printf-example.o
ld printf-example.o -o printf-example -lc \
   -dynamic-linker /lib/ld-linux.so.2
```

Luego ejecuta el programa con `./printf-example`, y debería decir esto:

`Hello! Jonathan is a person who loves the number 3`

Ahora, si miras el código, verás que en realidad metemos la cadena de formato al final, a pesar de que es el primer parámetro listado. Siempre metes los parámetros de una función en orden inverso. [^1] Te estarás preguntando cómo sabe la función `printf` cuántos parámetros hay. Pues bien, busca en tu cadena y cuenta cuántos `%d` y `%s` encuentra, y luego toma ese número de parámetros de la pila. Si el parámetro coincide con un `%d`, lo trata como un número, y si coincide con un `%s`, lo trata como un puntero a una cadena terminada en nulo. `printf` tiene muchas más funciones que estas, pero estas son las más utilizadas. Así que, como puedes ver, `printf` puede facilitar mucho la salida, pero también tiene mucha sobrecarga, porque tiene que contar el número de caracteres de la cadena, buscar en ella todos los caracteres de control que necesita sustituir, sacarlos de la pila, convertirlos a una representación adecuada (los números tienen que convertirse en cadenas, etc.) y unirlos todos apropiadamente.

Hemos visto cómo utilizar los prototipos del lenguaje de programación C para llamar a funciones de biblioteca. Sin embargo, para utilizarlos eficazmente, necesitas conocer varios tipos de datos más de los posibles para leer las funciones. Aquí están los principales:

- **`int`**: Un int es un número entero (4 bytes en el procesador x86).
- **`long`**: Un long es también un número entero (4 bytes en un procesador x86).
- **`long long`**: Un long long es un número entero más grande que un long (8 bytes en un procesador x86).
- **`short`**: Un short es un número entero más corto que un int (2 bytes en un procesador x86).
- **`char`**: Un char es un número entero de un solo byte. Se utiliza principalmente para almacenar datos de caracteres, ya que las cadenas ASCII suelen representarse con un byte por carácter.
- **`float`**: Un float es un número de punto flotante (4 bytes en un procesador x86). Los números de punto flotante se explicarán con más profundidad en la sección llamada *Números de Punto Flotante* en el Capítulo 10.
- **`double`**: Un double es un número de punto flotante que es más grande que un float (8 bytes en un procesador x86).
- **`unsigned`**: unsigned es un modificador utilizado para cualquiera de los tipos anteriores que evita que se utilicen como cantidades con signo. La diferencia entre números con signo y sin signo se discutirá en el Capítulo 10.
- **`*`**: Un asterisco (\*) se utiliza para denotar que el dato no es un valor real, sino que es un puntero a una ubicación que contiene el valor dado (4 bytes en un procesador x86). Por lo tanto, supongamos que en la ubicación de memoria `my_location` tienes almacenado el número 20. Si el prototipo dice que se pase un `int`, utilizarías el modo de direccionamiento directo y harías `pushl my_location`. Sin embargo, si el prototipo dijera que se pase un `int *`, harías `pushl $my_location`, un push en modo inmediato de la dirección en la que reside el valor. Además de indicar la dirección de un único valor, los punteros también pueden utilizarse para pasar una secuencia de ubicaciones consecutivas, empezando por la apuntada por el valor dado. Esto se denomina **array** o arreglo.
- **`struct`**: Un struct es un conjunto de elementos de datos que se han agrupado bajo un nombre. Por ejemplo, podrías declarar:
  ```c
  struct teststruct {
    int a;
    char *b;
  };
  ```
  y cada vez que te encontraras con `struct teststruct` sabrías que en realidad son dos palabras una al lado de la otra, siendo la primera un entero y la segunda un puntero a un carácter o grupo de caracteres. Nunca verás structs pasados como argumentos a funciones. En su lugar, normalmente verás punteros a structs pasados como argumentos. Esto se debe a que pasar structs a funciones es bastante complicado, ya que pueden ocupar muchas ubicaciones de almacenamiento.
- **`typedef`**: Un typedef permite básicamente renombrar un tipo. Por ejemplo, puedo hacer `typedef int myowntype;` en un programa C, y cada vez que escriba `myowntype`, sería exactamente como si escribiera `int`. Esto puede llegar a ser un poco molesto, porque tienes que buscar qué significan realmente todos los typedefs y structs en un prototipo de función. Sin embargo, los typedefs son útiles para dar a los tipos nombres más significativos y descriptivos.

> **Nota de Compatibilidad:** Los tamaños indicados son para máquinas compatibles con Intel (x86). Otras máquinas tendrán tamaños diferentes. Además, incluso cuando se pasan a las funciones parámetros más cortos que una palabra, se pasan como longs en la pila.

Así es como se lee la documentación de las funciones. Ahora, volvamos a la pregunta de cómo averiguar sobre las bibliotecas. La mayoría de las bibliotecas del sistema están en `/usr/lib` o `/lib`. Si quieres ver simplemente qué símbolos definen, ejecuta `objdump -R NOMBRE_ARCHIVO` donde `NOMBRE_ARCHIVO` es la ruta completa a la biblioteca. La salida de eso no es muy útil, sin embargo, para encontrar una interfaz que puedas necesitar. Normalmente, tienes que saber qué biblioteca quieres al principio y luego simplemente leer la documentación. La mayoría de las bibliotecas tienen manuales o páginas de manual (man pages) para sus funciones. La web es la mejor fuente de documentación para las bibliotecas. La mayoría de las bibliotecas del proyecto GNU también tienen páginas info sobre ellas, que son un poco más exhaustivas que las páginas man.

## Funciones Útiles

Varias funciones útiles de la biblioteca c que querrás conocer son:

- `size_t strlen (const char *s)` calcula el tamaño de las cadenas terminadas en nulo.
- `int strcmp (const char *s1, const char *s2)` compara dos cadenas alfabéticamente.
- `char * strdup (const char *s)` toma el puntero a una cadena y crea una nueva copia en una nueva ubicación, devolviendo la nueva ubicación.
- `FILE * fopen (const char *filename, const char *opentype)` abre un archivo gestionado y con búfer (permite leer y escribir más fácilmente que usando directamente descriptores de archivo). [^2] [^3]
- `int fclose (FILE *stream)` cierra un archivo abierto con `fopen`.
- `char * fgets (char *s, int count, FILE *stream)` extrae una línea de caracteres en la cadena `s`.
- `int fputs (const char *s, FILE *stream)` escribe una cadena en el archivo abierto dado.
- `int fprintf (FILE *stream, const char *template, ...)` es igual que `printf`, pero utiliza un archivo abierto en lugar de usar por defecto la salida estándar.

Puedes encontrar el manual completo de esta biblioteca en http://www.gnu.org/software/libc/manual/

## Construcción de una Biblioteca Compartida

Supongamos que queremos tomar todo nuestro código compartido del Capítulo 6 y convertirlo en una biblioteca compartida para usarla en nuestros programas. Lo primero que haríamos sería ensamblarlos como siempre:

```bash
as write-record.s -o write-record.o
as read-record.s -o read-record.o
```

Ahora, en lugar de enlazarlos en un programa, queremos enlazarlos en una biblioteca compartida. Esto cambia nuestro comando del enlazador a este:

```bash
ld -shared write-record.o read-record.o -o librecord.so
```

Esto enlaza estos dos archivos en una biblioteca compartida llamada `librecord.so`. Este archivo puede utilizarse ahora para varios programas. Si necesitamos actualizar las funciones contenidas en él, solo tenemos que actualizar este único archivo y no tenemos que preocuparnos de qué programas lo utilizan.

Veamos cómo enlazaríamos contra esta biblioteca. Para enlazar el programa `write-records`, haríamos lo siguiente:

```bash
as write-records.s -o write-records.o
ld -L . -dynamic-linker /lib/ld-linux.so.2 \
   -o write-records -lrecord write-records.o
```

En este comando, `-L .` le indica al enlazador que busque bibliotecas en el directorio actual (normalmente solo busca en el directorio `/lib`, el directorio `/usr/lib` y algunos otros). Como hemos visto, la opción `-dynamic-linker /lib/ld-linux.so.2` especificó el enlazador dinámico. La opción `-lrecord` indica al enlazador que busque funciones en el archivo llamado `librecord.so`.

Ahora el programa `write-records` está construido, pero no se ejecutará. Si lo intentamos, obtendremos un error como el siguiente:

`./write-records: error while loading shared libraries: librecord.so: cannot open shared object file: No such file or directory`

Esto se debe a que, por defecto, el enlazador dinámico solo busca bibliotecas en `/lib`, `/usr/lib` y en los directorios enumerados en `/etc/ld.so.conf`. Para ejecutar el programa, tienes que mover la biblioteca a uno de estos directorios o ejecutar el siguiente comando:

```bash
LD_LIBRARY_PATH=.
export LD_LIBRARY_PATH
```

Alternativamente, si eso te da un error, haz esto en su lugar:

```bash
setenv LD_LIBRARY_PATH .
```

Ahora puedes ejecutar `write-records` normalmente escribiendo `./write-records`. Establecer `LD_LIBRARY_PATH` le indica al enlazador que añada cualquier ruta que le des a la ruta de búsqueda de bibliotecas para bibliotecas dinámicas.

Para más información sobre el enlace dinámico, consulta las siguientes fuentes en Internet:

- La página man de `ld.so` contiene mucha información sobre cómo funciona el enlazador dinámico de Linux.
- http://www.benyossef.com/presentations/dlink/ es una gran presentación sobre el enlace dinámico en Linux.
- http://www.linuxjournal.com/article.php?sid=1059 y http://www.linuxjournal.com/article.php?sid=1060 proporcionan una buena introducción al formato de archivo ELF, con más detalles disponibles en http://www.cs.ucdavis.edu/~haungs/paper/node10.html
- http://www.iecc.com/linker/linker10.html contiene una gran descripción de cómo funciona el enlace dinámico con archivos ELF.

## Revisión

### Conoce los Conceptos

- ¿Cuáles son las ventajas y desventajas de las bibliotecas compartidas?
- Dada una biblioteca llamada 'foo', ¿cuál sería el nombre de archivo de la biblioteca?
- ¿Qué hace el comando `ldd`?
- Supongamos que tenemos los archivos `foo.o` y `bar.o`, y queremos enlazarlos y enlazarlos dinámicamente a la biblioteca 'kramer'. ¿Cuál sería el comando de enlace para generar el ejecutable final?
- ¿Para qué sirve `typedef`?
- ¿Para qué sirven los `structs`?
- ¿Cuál es la diferencia entre un elemento de datos de tipo `int` e `int *`? ¿Cómo los accederías de forma diferente en tu programa?
- Si tuvieras un archivo objeto llamado `foo.o`, ¿cuál sería el comando para crear una biblioteca compartida llamada 'bar'?
- ¿Cuál es el propósito de `LD_LIBRARY_PATH`?

### Usa los Conceptos

- Reescribe uno o más de los programas de los capítulos anteriores para que impriman sus resultados en la pantalla utilizando `printf` en lugar de devolver el resultado como el código de estado de salida. Además, haz que el código de estado de salida sea 0.
- Utiliza la función `factorial` que desarrollaste en la sección llamada *Funciones Recursivas* en el Capítulo 4 para crear una biblioteca compartida. Luego vuelve a escribir el programa principal para que se enlace con la biblioteca dinámicamente.
- Reescribe el programa anterior para que también se enlace con la biblioteca 'c'. Utiliza la función `printf` de la biblioteca 'c' para mostrar el resultado de la llamada a factorial.
- Reescribe el programa `toupper` para que utilice las funciones de la biblioteca c para archivos en lugar de las llamadas al sistema.

### Yendo más allá

- Haz una lista de todas las variables de entorno utilizadas por el enlazador dinámico de GNU/Linux.
- Investiga los diferentes tipos de formatos de archivos ejecutables que se utilizan hoy en día y en la historia de la informática. Explica los puntos fuertes y débiles de cada uno.
- ¿Qué tipos de programación te interesan (gráficos, bases de datos, ciencia, etc.)? Busca una biblioteca para trabajar en esa área y escribe un programa que haga un uso básico de esa biblioteca.
- Investiga el uso de `LD_PRELOAD`. ¿Para qué se utiliza? Intenta construir una biblioteca compartida que contenga la función `exit` y haz que escriba un mensaje en `STDERR` antes de salir. Utiliza `LD_PRELOAD` y ejecuta varios programas con ella. ¿Cuáles son los resultados?

---

[^1]: La razón por la que los parámetros se meten en la pila en orden inverso es debido a las funciones que aceptan un número variable de parámetros como `printf`. Los parámetros metidos al final estarán en una posición conocida con respecto a la parte superior de la pila. El programa puede entonces utilizar estos parámetros para determinar en qué parte de la pila se encuentran los argumentos adicionales y de qué tipo son. Por ejemplo, `printf` utiliza la cadena de formato para determinar cuántos otros parámetros se están enviando. Si metiéramos primero los argumentos conocidos, no podrías saber dónde estaban en la pila.
[^2]: `stdin`, `stdout` y `stderr` (todo en minúsculas) pueden utilizarse en estos programas para referirse a los archivos de sus correspondientes descriptores de archivo.
[^3]: `FILE` es un struct. No necesitas conocer su contenido para usarlo. Solo tienes que almacenar el puntero y pasarlo a las otras funciones pertinentes.
