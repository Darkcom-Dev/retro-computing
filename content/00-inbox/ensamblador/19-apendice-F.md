# Apéndice F. Usando el Depurador GDB

Cuando leas este apéndice, probablemente ya habrás escrito al menos un programa con un error. En [[02-capitulo-2|lenguaje ensamblador]], incluso los errores menores suelen tener resultados como que todo el programa se bloquee con un error de segmentación. En la mayoría de los lenguajes de programación, puedes simplemente imprimir los valores de tus variables a medida que avanzas, y usar esa salida para descubrir dónde te equivocaste. En lenguaje ensamblador, llamar a funciones de salida no es tan fácil. Por lo tanto, para ayudar a determinar la fuente de los errores, debes usar un depurador de código fuente.

Un depurador es un programa que te ayuda a encontrar errores recorriendo el programa paso a paso, permitiéndote examinar el contenido de la memoria y los registros en el camino. Un depurador de código fuente es un depurador que te permite vincular la operación de depuración directamente al código fuente de un programa. Esto significa que el depurador te permite mirar el código fuente tal como lo escribiste - completo con símbolos, etiquetas y comentarios.

El depurador que veremos es GDB - el Depurador GNU. Esta aplicación está presente en casi todas las distribuciones de GNU/Linux. Puede depurar programas en múltiples lenguajes de programación, incluyendo el lenguaje ensamblador.

## Una Sesión de Depuración de Ejemplo

La mejor manera de explicar cómo funciona un depurador es usándolo. El programa que usaremos con el depurador es el programa máximo usado en el [[03-capitulo-3|Capítulo 3]]. Digamos que ingresaste el programa perfectamente, excepto que omitiste la línea:

```assembly
incl %edi
```

Cuando ejecutas el programa, entra en un bucle infinito - nunca sale. Para determinar la causa, necesitas ejecutar el programa bajo GDB. Sin embargo, para hacer esto, necesitas que el ensamblador incluya información de depuración en el ejecutable. Todo lo que necesitas hacer para habilitar esto es añadir la opción `--gstabs` al comando `as`. Por lo tanto, lo ensamblarías así:

```bash
as --gstabs maximum.s -o maximum.o
```

El enlace sería igual que lo normal. "stabs" es el formato de depuración usado por GDB. Ahora, para ejecutar el programa bajo el depurador, escribirías `gdb ./maximum`. Asegúrate de que los archivos fuente estén en el directorio actual. La salida debería verse similar a esto:

```text
GNU gdb Red Hat Linux (5.2.1-4)
Copyright 2002 Free Software Foundation, Inc.
GDB es software libre, cubierto por la GNU General Public
License, y eres bienvenido a cambiarlo y/o
distribuir copias del mismo bajo ciertas condiciones. Escribe
"show copying" para ver las condiciones. No hay
absolutamente ninguna garantía para GDB. Escribe "show warranty"
para más detalles.
Este GDB fue configurado como "i386-redhat-linux"...
(gdb)
```

Dependiendo de la versión de GDB que estés ejecutando, esta salida puede variar ligeramente. En este punto, el programa está cargado, pero aún no se está ejecutando. El depurador está esperando tu comando. Para ejecutar tu programa, simplemente escribe `run`. Esto no regresará, porque el programa se está ejecutando en un bucle infinito. Para detener el programa, presiona control-c. La pantalla dirá entonces esto:

```text
Iniciando programa: /home/johnnyb/maximum

Programa recibió la señal SIGINT, Interrupción.
start_loop () en maximum.s:34
34              movl data_items(,%edi,4), %eax
Idioma actual: auto; actualmente asm
(gdb)
```

Esto te dice que el programa fue interrumpido por la señal SIGINT (de tu control-c), y estaba dentro de la sección etiquetada `start_loop`, y estaba ejecutando en la línea 34 cuando se detuvo. Te da el código que está a punto de ejecutar.

Dependiendo de exactamente cuándo presionaste control-c, puede haberse detenido en una línea diferente o una instrucción diferente al ejemplo.

Una de las mejores maneras de encontrar errores en un programa es seguir el flujo del programa para ver dónde se está ramificando incorrectamente. Para seguir el flujo de este programa, sigue ingresando `stepi` (de "step instruction" - paso de instrucción), que hará que la computadora ejecute una instrucción a la vez. Si haces esto varias veces, tu salida se verá algo así:

```text
(gdb) stepi
35              cmpl %ebx, %eax
(gdb) stepi
36              jle start_loop
(gdb) stepi
32              cmpl $0, %eax
(gdb) stepi
33              je loop_exit
(gdb) stepi
34              movl data_items(,%edi,4), %eax
(gdb) stepi
35              cmpl %ebx, %eax
(gdb) stepi
36              jle start_loop
(gdb) step
32              cmpl $0, %eax
```

Como puedes ver, ha repetido el bucle. En general, esto es bueno, ya que lo escribimos para que hiciera un bucle. Sin embargo, el problema es que nunca se detiene. Por lo tanto, para descubrir cuál es el problema, veamos el punto en nuestro código donde deberíamos salir del bucle:

```assembly
cmpl $0, %eax
je loop_exit
```

Básicamente, está verificando si `%eax` llega a cero. Si es así, debería salir del bucle. Hay varias cosas que verificar aquí. En primer lugar, puede que hayas omitido esta pieza por completo. No es raro que un programador olvide incluir una manera de salir de un bucle. Sin embargo, este no es el caso aquí. En segundo lugar, deberías asegurarte de que `loop_exit` esté realmente fuera del bucle. Si colocamos la etiqueta en el lugar equivocado, sucederían cosas extrañas. Sin embargo, de nuevo, este no es el caso.

Ninguno de esos problemas potenciales es el culpable. Entonces, la siguiente opción es que quizás `%eax` tiene el valor incorrecto. Hay dos maneras de verificar el contenido de los registros en GDB. La primera es el comando `info register`. Esto mostrará el contenido de todos los registros en hexadecimal. Sin embargo, solo estamos interesados en `%eax` en este punto. Para mostrar solo `%eax` podemos hacer `print/$eax` para imprimirlo en hexadecimal, o hacer `print/d $eax` para imprimirlo en decimal. Observa que en GDB, los registros tienen el prefijo de signos de dólar en lugar de signos de porcentaje. Tu pantalla debería tener esto:

```text
(gdb) print/d $eax
$1 = 3
(gdb)
```

Esto significa que el resultado de tu primera consulta es 3. Cada consulta que hagas recibirá un número prefijado con un signo de dólar. Ahora, si miras de vuelta al código, encontrarás que 3 es el primer número en la lista de números a buscar. Si avanzas por el bucle unas cuantas veces más, encontrarás que en cada iteración del bucle `%eax` tiene el número 3. Esto no es lo que debería estar sucediendo. `%eax` debería ir al siguiente valor en la lista en cada iteración.

Bien, ahora sabemos que `%eax` se está cargando con el mismo valor una y otra vez. Busquemos para ver de dónde se está cargando `%eax`. La línea de código es esta:

```assembly
movl data_items(,%edi,4), %eax
```

Entonces, avanza hasta que esta línea de código esté lista para ejecutarse. Ahora, este código depende de dos valores - `data_items` y `%edi`. `data_items` es un símbolo, y por lo tanto constante. Es una buena idea verificar tu código fuente para asegurarte de que la etiqueta está delante de los datos correctos, pero en nuestro caso lo está. Por lo tanto, necesitamos mirar `%edi`. Entonces, necesitamos imprimirlo. Se verá así:

```text
(gdb) print/d $edi
$2 = 0
(gdb)
```

Esto indica que `%edi` está establecido a cero, que es la razón por la que sigue cargando el primer elemento del array. Esto debería hacer que te hagas dos preguntas - ¿cuál es el propósito de `%edi`, y cómo debería cambiarse su valor? Para responder a la primera pregunta, solo necesitamos mirar los comentarios. `%edi` está conteniendo el índice actual de `data_items`. Como nuestra búsqueda es una búsqueda secuencial a través de la lista de números en `data_items`, tendría sentido que `%edi` debería incrementarse con cada iteración del bucle.

Escaneando el código, no hay ningún código que altere `%edi` en absoluto. Por lo tanto, deberíamos añadir una línea para incrementar `%edi` al principio de cada iteración del bucle. Esto resulta ser exactamente la línea que descartamos al principio. Ensamblar, enlazar y ejecutar el programa de nuevo mostrará que ahora funciona correctamente.

Espero que este ejercicio haya proporcionado una visión sobre el uso de GDB para ayudarte a encontrar errores en tus programas.

## Puntos de Interrupción y Otras Características de GDB

El programa que ingresamos en la última sección tenía un bucle infinito, y podía detenerse fácilmente usando control-c. Otros programas pueden simplemente abortar o terminar con errores. En estos casos, control-c no ayuda, porque para cuando presionas control-c, el programa ya ha terminado. Para solucionar esto, necesitas establecer puntos de interrupción (breakpoints). Un punto de interrupción es un lugar en el código fuente que has marcado para indicar al depurador que debe detener el programa cuando llegue a ese punto.

Para establecer puntos de interrupción, tienes que configurarlos antes de ejecutar el programa. Antes de emitir el comando run, puedes configurar puntos de interrupción usando el comando `break`. Por ejemplo, para interrumpir en la línea 27, emite el comando `break 27`. Entonces, cuando el programa cruce la línea 27, se detendrá y mostrará la línea e instrucción actuales. Puedes entonces avanzar por el programa desde ese punto y examinar registros y memoria. Para ver las líneas y números de línea de tu programa, puedes simplemente usar el comando `l`. Esto imprimirá tu programa con números de línea una pantalla a la vez.

Cuando trabajas con funciones, también puedes interrumpir en los nombres de las funciones. Por ejemplo, en el programa factorial del [[04-capitulo-4|Capítulo 4]], podríamos establecer un punto de interrupción para la función factorial escribiendo `break factorial`. Esto hará que el depurador se interrumpa inmediatamente después de la llamada a la función y la configuración de la función (salta el push de `%ebp` y la copia de `%esp`).

Cuando avanzas por el código, a menudo no querrás tener que avanzar por cada instrucción de cada función. Las funciones bien probadas suelen ser una pérdida de tiempo para recorrer excepto en raras ocasiones. Por lo tanto, si usas el comando `nexti` en lugar del comando `stepi`, GDB esperará hasta que la función se complete antes de continuar. De lo contrario, con `stepi`, GDB te llevaría a través de cada instrucción dentro de cada función llamada.

> **Advertencia**
> Un problema que GDB tiene es con el manejo de interrupciones. A menudo, GDB omitirá la instrucción que sigue inmediatamente a una interrupción. La instrucción se ejecuta realmente, pero GDB no la recorre. Esto no debería ser un problema - solo sé consciente de que puede suceder.

## Referencia Rápida de GDB

Esta tabla de referencia rápida tiene derechos de autor 2002 de Robert M. Dondero, Jr., y se usa con permiso en este libro. Los parámetros listados entre corchetes son opcionales.

### Tabla F-1. Comandos Comunes de Depuración de GDB

| Comando | Descripción |
| :--- | :--- |
| **Varios** | |
| `quit` | Salir de GDB |
| `help [cmd]` | Imprime la descripción del comando de depurador `cmd`. Sin `cmd`, imprime una lista de temas. |
| `directory [dir1] [dir2] ...` | Añade los directorios `dir1`, `dir2`, etc. a la lista de directorios buscados para archivos fuente. |
| **Ejecutando el Programa** | |
| `run [arg1] [arg2] ...` | Ejecuta el programa con los argumentos de línea de comandos `arg1`, `arg2`, etc. |
| `set args arg1 [arg2] ...` | Establece los argumentos de línea de comandos del programa a `arg1`, `arg2`, etc. |
| `show args` | Imprime los argumentos de línea de comandos del programa. |
| **Usando Puntos de Interrupción** | |
| `info breakpoints` | Imprime una lista de todos los puntos de interrupción y sus números (los números de puntos de interrupción se usan para otros comandos de puntos de interrupción). |
| `break linenum` | Establece un punto de interrupción en el número de línea `linenum`. |
| `break *addr` | Establece un punto de interrupción en la dirección de memoria `addr`. |
| `break fn` | Establece un punto de interrupción al principio de la función `fn`. |
| `condition bpnum expr` | Interrumpe en el punto de interrupción `bpnum` solo si la expresión `expr` es distinta de cero. |
| `command [bpnum] cmd1 [cmd2] ...` | Ejecuta los comandos `cmd1`, `cmd2`, etc. cada vez que se alcanza el punto de interrupción `bpnum` (o el punto de interrupción actual). |
| `continue` | Continúa ejecutando el programa. |
| `kill` | Detiene la ejecución del programa. |
| `delete [bpnum1] [bpnum2] ...` | Elimina los puntos de interrupción `bpnum1`, `bpnum2`, etc., o todos los puntos de interrupción si no se especifica ninguno. |
| `clear *addr` | Limpia el punto de interrupción en la dirección de memoria `addr`. |
| `clear [fn]` | Limpia el punto de interrupción en la función `fn`, o el punto de interrupción actual. |
| `clear linenum` | Limpia el punto de interrupción en el número de línea `linenum`. |
| `disable [bpnum1] [bpnum2] ...` | Deshabilita los puntos de interrupción `bpnum1`, `bpnum2`, etc., o todos los puntos de interrupción si no se especifica ninguno. |
| `enable [bpnum1] [bpnum2] ...` | Habilita los puntos de interrupción `bpnum1`, `bpnum2`, etc., o todos los puntos de interrupción si no se especifica ninguno. |
| **Avanzando por el Programa** | |
| `nexti` | "Saltar sobre" la siguiente instrucción (no sigue las llamadas a funciones). |
| `stepi` | "Entrar en" la siguiente instrucción (sigue las llamadas a funciones). |
| `finish` | "Salir de" la función actual. |
| **Examinando Registros y Memoria** | |
| `info registers` | Imprime el contenido de todos los registros. |
| `print/f $reg` | Imprime el contenido del registro `reg` usando el formato `f`. El formato puede ser `x` (hexadecimal), `u` (decimal sin signo), `o` (octal), `a` (dirección), `c` (carácter), o `f` (punto flotante). |
| `x/rsf addr` | Imprime el contenido de la dirección de memoria `addr` usando el conteo de repetición `r`, tamaño `s`, y formato `f`. El conteo de repetición por defecto es 1 si no se especifica. El tamaño puede ser `b` (byte), `h` (media palabra), `w` (palabra), o `g` (palabra doble). El tamaño por defecto es palabra si no se especifica. El formato es el mismo que para `print`, con las adiciones de `s` (cadena) e `i` (instrucción). |
| `info display` | Muestra una lista numerada de expresiones configuradas para mostrarse automáticamente en cada interrupción. |
| `display/f $reg` | En cada interrupción, imprime el contenido del registro `reg` usando el formato `f`. |
| `display/si addr` | En cada interrupción, imprime el contenido de la dirección de memoria `addr` usando el tamaño `s` (las mismas opciones que para el comando `x`). |
| `display/ss addr` | En cada interrupción, imprime la cadena de tamaño `s` que comienza en la dirección de memoria `addr`. |
| `undisplay displaynum` | Elimina `displaynum` de la lista de visualización. |
| **Examinando la Pila de Llamadas** | |
| `where` | Imprime la pila de llamadas. |
| `backtrace` | Imprime la pila de llamadas. |
| `frame` | Imprime la parte superior de la pila de llamadas. |
| `up` | Mueve el contexto hacia la parte inferior de la pila de llamadas. |
| `down` | Mueve el contexto hacia la parte superior de la pila de llamadas. |
