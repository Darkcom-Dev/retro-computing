# Capítulo 11. Lenguajes de Alto Nivel

En este capítulo comenzaremos a ver nuestro primer lenguaje de programación del "mundo real". El [[02-capitulo-2|lenguaje ensamblador]] es el lenguaje utilizado a nivel de la máquina, pero la mayoría de las personas encuentran la programación en ensamblador demasiado engorrosa para el uso diario. Se han inventado muchos lenguajes de computadora para facilitar la tarea de programación. Conocer una amplia variedad de lenguajes es útil por muchas razones, incluyendo:

*   Diferentes lenguajes se basan en diferentes conceptos, lo que te ayudará a aprender diferentes y mejores métodos e ideas de programación.
*   Diferentes lenguajes son buenos para diferentes tipos de proyectos.
*   Diferentes empresas tienen diferentes lenguajes estándar, por lo que conocer más lenguajes hace que tus habilidades sean más comercializables.
*   Cuantos más lenguajes conozcas, más fácil es aprender nuevos.

Como programador, a menudo tendrás que aprender nuevos lenguajes. Los programadores profesionales normalmente pueden aprender un nuevo lenguaje con aproximadamente una semana de estudio y práctica. Los lenguajes son simplemente herramientas, y aprender a usar una nueva herramienta no debería ser algo que un programador rechace. De hecho, si haces consultoría informática, a menudo tendrás que aprender nuevos lenguajes sobre la marcha para mantenerte empleado. A menudo será tu cliente, no tú, quien decida qué lenguaje se usa. Este capítulo te introducirá a algunos de los lenguajes disponibles para ti. Te animo a explorar tantos lenguajes como te interesen. Personalmente, trato de aprender un nuevo lenguaje cada pocos meses.

## Lenguajes Compilados e Interpretados

Muchos lenguajes son **[[11-capitulo-11#compiled-and-interpreted-languages|lenguajes compilados]]**. Cuando escribes en lenguaje ensamblador, cada instrucción que escribes se traduce en exactamente una instrucción de máquina para procesar. Con los compiladores, una declaración puede traducirse en una o cientos de instrucciones de máquina. De hecho, dependiendo de lo avanzado que sea tu compilador, incluso podría reestructurar partes de tu código para hacerlo más rápido. En lenguaje ensamblador, lo que escribes es lo que obtienes.

También hay lenguajes que son **lenguajes interpretados**. Estos lenguajes requieren que el usuario ejecute un programa llamado intérprete que a su vez ejecuta el programa dado. Estos suelen ser más lentos que los programas compilados, ya que el intérprete tiene que leer e interpretar el código sobre la marcha. Sin embargo, en intérpretes bien hechos, este tiempo puede ser bastante insignificante. También hay una clase de **lenguajes híbridos** que compilan parcialmente un programa antes de la ejecución en byte-codes. Esto se hace porque el intérprete puede leer los byte-codes mucho más rápido de lo que puede leer el lenguaje regular.

Hay muchas razones para elegir uno u otro. Los programas compilados son buenos, porque no tienes que tener ya instalado un intérprete en la máquina del usuario. Tienes que tener un compilador para el lenguaje, pero los usuarios de tu programa no. En un lenguaje interpretado, tienes que asegurarte de que el usuario tenga un intérprete instalado para tu programa, y que la computadora sepa qué intérprete usar para ejecutar tu programa. Sin embargo, los lenguajes interpretados tienden a ser más flexibles, mientras que los lenguajes compilados son más rígidos.

La elección del lenguaje suele estar impulsada por las herramientas disponibles y el soporte para métodos de programación, más que por si un lenguaje es compilado o interpretado. De hecho, muchos lenguajes tienen opciones para cualquiera de los dos.

Los lenguajes de alto nivel, ya sean compilados o interpretados, están orientados a ti, el programador, en lugar de a la máquina. Esto los abre a una amplia variedad de características, que pueden incluir las siguientes:

*   Poder agrupar múltiples operaciones en una sola expresión
*   Poder usar "valores grandes" - valores que son mucho más conceptuales que las palabras de 4 bytes con las que las computadoras tratan normalmente (por ejemplo, poder ver cadenas de texto como un valor único en lugar de como una secuencia de bytes).
*   Tener acceso a mejores construcciones de control de flujo que solo saltos.
*   Tener un compilador que verifique tipos de asignaciones de valores y otras aserciones.
*   Tener la memoria manejada automáticamente.
*   Poder trabajar en un lenguaje que se asemeje al dominio del problema en lugar del hardware de la computadora.

Entonces, ¿por qué uno elige un lenguaje sobre otro? Por ejemplo, muchos eligen **Perl** porque tiene una vasta biblioteca de funciones para manejar casi todos los protocolos o tipos de datos del planeta. **Python**, sin embargo, tiene una sintaxis más limpia y a menudo se presta a soluciones más directas. Sus herramientas GUI multiplataforma también son excelentes. **PHP** hace que escribir aplicaciones web sea simple. **Common LISP** tiene más potencia y características que cualquier otro entorno para aquellos dispuestos a aprenderlo. **Scheme** es el modelo de simplicidad y potencia combinados. **C** es fácil de interactuar con otros lenguajes.

Cada lenguaje es diferente, y cuantos más lenguajes conozcas, mejor programador serás. Conocer los conceptos de diferentes lenguajes te ayudará en toda la programación, porque puedes emparejar mejor el lenguaje de programación con el problema, y tienes un conjunto más grande de herramientas con las que trabajar. Incluso si ciertas características no son directamente compatibles con el lenguaje que estás usando, a menudo pueden simularse. Sin embargo, si no tienes una amplia experiencia con lenguajes, no conocerás todas las posibilidades que tienes para elegir.

## Tu Primer Programa en C

Aquí está tu primer programa en C, que imprime "Hello world" en la pantalla y sale. Escríbelo y dale el nombre `Hello-World.c`.

```c
#include <stdio.h>

/* PROPÓSITO: Este programa está diseñado para mostrar un programa */
/*          básico en C. Todo lo que hace es imprimir           */
/*          "Hello World!" en la pantalla y                     */
/*          salir.                                              */

/* Programa Principal */
int main(int argc, char **argv)
{
    /* Imprimir nuestra cadena en la salida estándar */
    puts("Hello World!\n");

    /* Salir con estado 0 */
    return 0;
}
```

Como puedes ver, es un programa bastante simple. Para compilarlo, ejecuta el comando:

```bash
gcc -o HelloWorld Hello-World.c
```

Para ejecutar el programa, haz:

```bash
./HelloWorld
```

Veamos cómo se armó este programa.

Los comentarios en C comienzan con `/*` y terminan con `*/`. Los comentarios pueden abarcar múltiples líneas, pero muchas personas prefieren comenzar y terminar los comentarios en la misma línea para no confundirse.

`#include <stdio.h>` es la primera parte del programa. Esto es una **directiva de preprocesador**. La compilación de C se divide en dos etapas - el preprocesador y el compilador principal. Esta directiva le dice al preprocesador que busque el archivo `stdio.h` y lo pegue en tu programa. El preprocesador es responsable de juntar el texto del programa. Esto incluye unir diferentes archivos, ejecutar macros en el texto de tu programa, etc. Después de que el texto se junta, el preprocesador termina y el compilador principal comienza a trabajar.

Ahora, todo en `stdio.h` está ahora en tu programa como si lo hubieras escrito tú mismo allí. Los corchetes angulares alrededor del nombre del archivo le dicen al compilador que busque en sus rutas estándar el archivo (`/usr/include` y `/usr/local/include`, usualmente). Si estuviera entre comillas, como `#include "stdio.h"`, buscaría en el directorio actual el archivo. De todos modos, `stdio.h` contiene las declaraciones para las funciones y variables estándar de entrada y salida. Estas declaraciones le dicen al compilador qué funciones están disponibles para entrada y salida. Las siguientes líneas son simplemente comentarios sobre el programa.

Luego está la línea `int main(int argc, char **argv)`. Este es el inicio de una función. Las funciones en C se declaran con su nombre, argumentos y tipo de retorno. Esta declaración dice que el nombre de la función es `main`, devuelve un `int` (entero - 4 bytes de largo en la plataforma x86), y tiene dos argumentos - un `int` llamado `argc` y un `char **` llamado `argv`. No tienes que preocuparte por dónde están posicionados los argumentos en la pila - el compilador de C se encarga de eso por ti. Tampoco tienes que preocuparte por cargar valores en y fuera de los registros porque el compilador también se encarga de eso.

La función `main` es una función especial en el lenguaje C - es el inicio de todos los programas en C (muy parecido a `_start` en nuestros programas en lenguaje ensamblador). Siempre toma dos parámetros. El primer parámetro es el número de argumentos dados a este comando, y el segundo parámetro es una lista de los argumentos que se dieron.

La siguiente línea es una llamada a función. En lenguaje ensamblador, tenías que empujar los argumentos de una función a la pila, y luego llamar a la función. C se encarga de esta complejidad por ti. Simplemente tienes que llamar a la función con los parámetros entre paréntesis. En este caso, llamamos a la función `puts`, con un solo parámetro. Este parámetro es la cadena de caracteres que queremos imprimir. Solo tenemos que escribir la cadena entre comillas, y el compilador se encarga de definir el almacenamiento y mover los punteros a ese almacenamiento a la pila antes de llamar a la función. Como puedes ver, es mucho menos trabajo.

Finalmente, nuestra función devuelve el número 0. En lenguaje ensamblador, almacenábamos nuestro valor de retorno en `%eax`, pero en C simplemente usamos el comando `return` y se encarga de eso por nosotros. El valor de retorno de la función `main` es lo que se usa como código de salida del programa.

Como puedes ver, usar lenguajes de alto nivel hace la vida mucho más fácil. También permite que nuestros programas se ejecuten en múltiples plataformas más fácilmente. En lenguaje ensamblador, tu programa está ligado tanto al sistema operativo como a la plataforma de hardware, mientras que en lenguajes compilados e interpretados el mismo código normalmente puede ejecutarse en múltiples sistemas operativos y plataformas de hardware. Por ejemplo, este programa puede construirse y ejecutarse en hardware x86 con Linux®, Windows®, UNIX®, o la mayoría de los otros sistemas operativos. Además, también puede ejecutarse en hardware Macintosh con varios sistemas operativos.

Información adicional sobre el lenguaje de programación C se puede encontrar en el [[18-apendice-E|Apéndice E]].

## Perl

Perl es un lenguaje interpretado, que existe principalmente en plataformas Linux y UNIX. En realidad, se ejecuta en casi todas las plataformas, pero se encuentra más a menudo en Linux y las basadas en UNIX. De todos modos, aquí está la versión en Perl del programa, que debe escribirse en un archivo llamado `Hello-World.pl`:

```perl
#!/usr/bin/perl
print("Hello world!\n");
```

Ya que Perl es interpretado, no necesitas compilarlo ni enlazarlo. Simplemente ejecútalo con el siguiente comando:

```bash
perl Hello-World.pl
```

Como puedes ver, la versión en Perl es incluso más corta que la versión en C. Con Perl no tienes que declarar ninguna función o puntos de entrada del programa. Puedes simplemente comenzar a escribir comandos y el intérprete los ejecutará a medida que los encuentre. De hecho, este programa solo tiene dos líneas de código, una de las cuales es opcional.

La primera línea, opcional, se usa en máquinas UNIX para indicar qué intérprete usar para ejecutar el programa. El `#!` le dice a la computadora que este es un programa interpretado, y `/usr/bin/perl` le dice a la computadora que use el programa `/usr/bin/perl` para interpretar el programa. Sin embargo, ya que ejecutamos el programa escribiendo `perl Hello-World.pl`, ya habíamos especificado que estábamos usando el intérprete perl.

La siguiente línea llama a una función incorporada de Perl, `print`. Esta tiene un parámetro, la cadena a imprimir. El programa no tiene una declaración de retorno explícita - sabe que debe retornar simplemente porque llega al final del archivo. También sabe retornar 0 porque no hubo errores mientras se ejecutaba. Puedes ver que los lenguajes interpretados a menudo se centran en permitirte obtener código funcional lo más rápido posible, sin tener que hacer mucho trabajo adicional.

Una cosa sobre Perl que no es tan evidente en este ejemplo es que Perl trata las cadenas como un valor único. En lenguaje ensamblador, teníamos que programar de acuerdo con la arquitectura de memoria de la computadora, lo que significaba que las cadenas tenían que ser tratadas como una secuencia de múltiples valores, con un puntero a la primera letra. Perl finge que las cadenas pueden almacenarse directamente como valores, y así oculta la complejidad de manipularlas por ti. De hecho, una de las principales fortalezas de Perl es su habilidad y velocidad para manipular texto.

## Python

La versión en Python del programa se ve casi exactamente como la de Perl. Sin embargo, Python es realmente un lenguaje muy diferente de Perl, aunque no lo parezca en este ejemplo trivial. Escribe el programa en un archivo llamado `Hello-World.py`. El programa es el siguiente:

```python
#!/usr/bin/python
print "Hello World"
```

Deberías poder decir qué hacen las diferentes líneas del programa.

## Revisión

### Conoce los Conceptos

*   ¿Cuál es la diferencia entre un lenguaje interpretado y un lenguaje compilado?
*   ¿Qué razones podrían hacer que necesites aprender un nuevo lenguaje de programación?

### Usa los Conceptos

*   Aprende la sintaxis básica de un nuevo lenguaje de programación. Vuelve a codificar uno de los programas de este libro en ese lenguaje.
*   En el programa que escribiste en la pregunta anterior, ¿qué cosas específicas se automatizaron en el lenguaje de programación que elegiste?
*   Modifica tu programa para que se ejecute 10,000 veces seguidas, tanto en lenguaje ensamblador como en tu nuevo lenguaje. Luego ejecuta el comando `time` para ver cuál es más rápido. ¿Cuál gana? ¿Por qué crees que es así?
*   ¿En qué se diferencian los métodos de entrada/salida del lenguaje de programación de los de las llamadas al sistema de Linux?

### Yendo Más Allá

*   Habiendo visto lenguajes que tienen tanta brevedad como Perl, ¿por qué crees que este libro te comenzó con un lenguaje tan verboso como el ensamblador?
*   ¿Cómo crees que los lenguajes de alto nivel han afectado el proceso de programación?
*   ¿Por qué crees que existen tantos lenguajes?
*   Aprende dos nuevos lenguajes de alto nivel. ¿En qué se diferencian entre sí? ¿En qué se parecen? ¿Qué enfoque para la resolución de problemas tiene cada uno?
