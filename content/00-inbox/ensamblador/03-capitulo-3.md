# Capítulo 3. Tus Primeros Programas

En este capítulo aprenderás el proceso para escribir y construir programas en lenguaje ensamblador (assembly-language) para Linux. Además, aprenderás la estructura de los programas en lenguaje ensamblador y algunos comandos de dicho lenguaje. Mientras avanzas en este capítulo, es posible que también quieras consultar el Apéndice B y el Apéndice F.

Estos programas pueden abrumarte al principio. Sin embargo, revísalos con diligencia, léelos y lee sus explicaciones tantas veces como sea necesario, y tendrás una base sólida de conocimiento sobre la cual construir. Por favor, experimenta con los programas tanto como puedas. Incluso si tus experimentos no funcionan, cada fallo te ayudará a aprender.

## Introducción del Programa

Bien, este primer programa es simple. De hecho, ¡no va a hacer nada más que salir! Es corto, pero muestra algunos conceptos básicos sobre el lenguaje ensamblador y la programación en Linux. Debes introducir el programa en un editor exactamente como está escrito, con el nombre de archivo `exit.s`. El programa es el siguiente. No te preocupes si no lo entiendes. Esta sección solo trata de escribirlo y ejecutarlo. En la sección llamada *Esquema de un Programa en Lenguaje Ensamblador* describiremos cómo funciona.

```assembly
#PROPÓSITO: Programa simple que sale y devuelve un
#           código de estado al kernel de Linux
#
#ENTRADA:   ninguna
#
#SALIDA:    devuelve un código de estado. Esto se puede ver
#           escribiendo
#
#           echo $?
#
#           después de ejecutar el programa
#
#VARIABLES:
#           %eax contiene el número de la llamada al sistema
#           %ebx contiene el estado de retorno
#
.section .data

.section .text
.globl _start
_start:
 movl $1, %eax      # este es el número del comando del kernel
                    # de Linux (llamada al sistema) para salir
                    # de un programa

 movl $0, %ebx      # este es el número de estado que
                    # devolveremos al sistema operativo.
                    # Cambia esto y devolverá
                    # cosas diferentes a
                    # echo $?

 int $0x80          # esto despierta al kernel para ejecutar
                    # el comando de salida
```

Lo que has escrito se llama **código fuente** (source code). El código fuente es la forma de un programa legible por humanos. Para transformarlo en un programa que una computadora pueda ejecutar, necesitamos **ensamblarlo** y **enlazarlo**.

El primer paso es ensamblarlo. **Ensamblar** (assembling) es el proceso que transforma lo que escribiste en instrucciones para la máquina. La máquina misma solo lee conjuntos de números, pero los humanos prefieren palabras. Un lenguaje ensamblador es una forma más legible para los humanos de las instrucciones que entiende una computadora. El ensamblaje transforma el archivo legible por humanos en uno legible por la máquina. Para ensamblar el programa, escribe el comando:

```bash
as exit.s -o exit.o
```

`as` es el comando que ejecuta el ensamblador, `exit.s` es el archivo fuente y `-o exit.o` le dice al ensamblador que ponga su salida en el archivo `exit.o`. `exit.o` es un **archivo objeto** (object file). Un archivo objeto es código que está en el lenguaje de la máquina, pero que no ha sido completamente armado. En la mayoría de los programas grandes, tendrás varios archivos fuente y convertirás cada uno en un archivo objeto. El **enlazador** (linker) es el programa responsable de juntar los archivos objeto y añadirles información para que el kernel sepa cómo cargarlo y ejecutarlo. En nuestro caso, solo tenemos un archivo objeto, por lo que el enlazador solo está añadiendo la información para permitir que se ejecute. Para enlazar el archivo, introduce el comando:

```bash
ld exit.o -o exit
```

`ld` es el comando para ejecutar el enlazador, `exit.o` es el archivo objeto que queremos enlazar y `-o exit` instruye al enlazador para que genere el nuevo programa en un archivo llamado `exit`. [^1] Si alguno de estos comandos reportó errores, es porque has escrito mal tu programa o el comando. Después de corregir el programa, tienes que volver a ejecutar todos los comandos. *Siempre debes volver a ensamblar y enlazar los programas después de modificar el archivo fuente para que los cambios ocurran en el programa.* Puedes ejecutar `exit` escribiendo el comando:

```bash
./exit
```

El `./` se usa para decirle a la computadora que el programa no está en uno de los directorios de programas normales, sino en el directorio actual [^2]. Notarás que cuando escribes este comando, lo único que sucede es que pasarás a la siguiente línea. Eso es porque este programa no hace nada más que salir. Sin embargo, inmediatamente después de ejecutar el programa, si escribes:

```bash
echo $?
```

Dirá `0`. Lo que está sucediendo es que cada programa, cuando sale, le da a Linux un **código de estado de salida** (exit status code), que le indica si todo salió bien. Si todo estuvo bien, devuelve 0. Los programas de UNIX devuelven números distintos de cero para indicar fallos u otros errores, advertencias o estados. El programador determina qué significa cada número. Puedes ver este código escribiendo `echo $?`. En la siguiente sección veremos qué hace cada parte del código.

## Esquema de un Programa en Lenguaje Ensamblador

Echa un vistazo al programa que acabamos de introducir. Al principio hay muchas líneas que comienzan con almohadillas (`#`). Estos son **comentarios**. Los comentarios no son traducidos por el ensamblador. Se utilizan solo para que el programador hable con cualquiera que mire el código en el futuro. La mayoría de los programas que escribas serán modificados por otros. Acostúmbrate a escribir comentarios en tu código que les ayuden a entender tanto por qué existe el programa como cómo funciona. Incluye siempre lo siguiente en tus comentarios:

- El propósito del código
- Una visión general del procesamiento involucrado
- Cualquier cosa extraña que haga tu programa y por qué la hace [^3]

Después de los comentarios, la siguiente línea dice:

```assembly
.section .data
```

Cualquier cosa que comience con un punto no se traduce directamente en una instrucción de máquina. En cambio, es una instrucción para el ensamblador mismo. Estas se llaman **directivas del ensamblador** o **pseudo-operaciones** porque son manejadas por el ensamblador y no son ejecutadas realmente por la computadora. El comando `.section` divide tu programa en secciones. Este comando inicia la **sección de datos** (data section), donde enumeras cualquier almacenamiento de memoria que necesites para los datos. Nuestro programa no usa ninguno, así que no necesitamos la sección. Está aquí solo por completitud. Casi todos los programas que escribas en el futuro tendrán datos.

Justo después de esto tienes:

```assembly
.section .text
```

que inicia la **sección de texto** (text section). La sección de texto de un programa es donde viven las instrucciones del programa.

La siguiente instrucción es:

```assembly
.globl _start
```

Esto instruye al ensamblador que `_start` es importante de recordar. `_start` es un **símbolo**, lo que significa que va a ser reemplazado por algo más ya sea durante el ensamblaje o el enlace. Los símbolos se utilizan generalmente para marcar ubicaciones de programas o datos, de modo que puedas referirte a ellos por su nombre en lugar de por su número de ubicación. Imagina si tuvieras que referirte a cada ubicación de memoria por su dirección. En primer lugar, sería muy confuso porque tendrías que memorizar o buscar la dirección de memoria numérica de cada pieza de código o datos. Además, ¡cada vez que tuvieras que insertar una pieza de datos o código tendrías que cambiar todas las direcciones en tu programa! Los símbolos se utilizan para que el ensamblador y el enlazador puedan encargarse de llevar la cuenta de las direcciones, y tú puedas concentrarte en escribir tu programa.

`.globl` significa que el ensamblador no debe descartar este símbolo después del ensamblaje, porque el enlazador lo necesitará. `_start` es un símbolo especial que siempre necesita ser marcado con `.globl` porque marca la ubicación del inicio del programa. Sin marcar esta ubicación de esta manera, cuando la computadora cargue tu programa no sabrá dónde comenzar a ejecutarlo.

La siguiente línea:

```assembly
_start:
```

define el valor de la **etiqueta** (label) `_start`. Una etiqueta es un símbolo seguido de dos puntos. Las etiquetas definen el valor de un símbolo. Cuando el ensamblador está ensamblando el programa, tiene que asignar a cada valor de datos e instrucción una dirección. Las etiquetas le dicen al ensamblador que el valor del símbolo sea dondequiera que esté la próxima instrucción o elemento de datos. De esta manera, si la ubicación física real de los datos o la instrucción cambia, no tienes que reescribir ninguna referencia a ella; el símbolo obtiene automáticamente el nuevo valor.

Ahora entramos en las instrucciones reales de la computadora. La primera de estas instrucciones es esta:

```assembly
movl $1, %eax
```

Cuando el programa se ejecuta, esta instrucción transfiere el número 1 al registro `%eax`. En el lenguaje ensamblador, muchas instrucciones tienen **operandos**. `movl` tiene dos operandos: el *origen* (source) y el *destino* (destination). En este caso, el origen es el número literal 1 y el destino es el registro `%eax`. Los operandos pueden ser números, referencias a ubicaciones de memoria o registros. Diferentes instrucciones permiten diferentes tipos de operandos. Consulta el Apéndice B para obtener más información sobre qué instrucciones aceptan qué tipos de operandos.

En la mayoría de las instrucciones que tienen dos operandos, el primero es el operando de origen y el segundo es el de destino. Ten en cuenta que en estos casos, el operando de origen no se modifica en absoluto. Otras instrucciones de este tipo son, por ejemplo, `addl`, `subl` e `imull`. Estas suman/restan/multiplican el operando de origen al/del/por el operando de destino y guardan el resultado en el operando de destino. Otras instrucciones pueden tener un operando prefijado. `idivl`, por ejemplo, requiere que el dividendo esté en `%eax` y que `%edx` sea cero, y el cociente se transfiere luego a `%eax` y el resto a `%edx`. Sin embargo, el divisor puede ser cualquier registro o ubicación de memoria.

En los procesadores x86, hay varios **registros de propósito general** [^4] (todos los cuales pueden usarse con `movl`):

- `%eax`
- `%ebx`
- `%ecx`
- `%edx`
- `%edi`
- `%esi`

Además de estos registros de propósito general, también hay varios **registros de propósito especial**, que incluyen:

- `%ebp`
- `%esp`
- `%eip`
- `%eflags`

Discutiremos estos más adelante, solo ten en cuenta que existen. [^5] Algunos de estos registros, como `%eip` y `%eflags`, solo pueden accederse a través de instrucciones especiales. A los otros se puede acceder utilizando las mismas instrucciones que los registros de propósito general, pero tienen significados especiales, usos especiales o simplemente son más rápidos cuando se usan de una manera específica.

Entonces, la instrucción `movl` mueve el número 1 a `%eax`. El signo de dólar delante del uno indica que queremos usar **direccionamiento en modo inmediato** (vuelve a consultar la sección llamada *Métodos de Acceso a Datos* en el Capítulo 2). Sin el signo de dólar, realizaría direccionamiento directo, cargando cualquier número que esté en la dirección 1. Queremos que se cargue el número 1 real, por lo que tenemos que usar el modo inmediato.

La razón por la que movemos el número 1 a `%eax` es porque nos estamos preparando para llamar al Kernel de Linux. El número 1 es el número de la **llamada al sistema exit**. Discutiremos las llamadas al sistema (system calls) con más profundidad pronto, pero básicamente son solicitudes de ayuda al sistema operativo. Los programas normales no pueden hacerlo todo. Muchas operaciones, como llamar a otros programas, tratar con archivos y salir, tienen que ser manejadas por el sistema operativo a través de **llamadas al sistema**. Cuando realizas una llamada al sistema, lo cual haremos en breve, el número de la llamada al sistema tiene que cargarse en `%eax` (para una lista completa de las llamadas al sistema y sus números, consulta el Apéndice C). Dependiendo de la llamada al sistema, otros registros pueden tener que contener valores también. Ten en cuenta que las llamadas al sistema no son el único uso, ni siquiera el uso principal de los registros. Es solo el que estamos tratando en este primer programa. Programas posteriores usarán registros para computación regular.

El sistema operativo, sin embargo, suele necesitar más información que solo qué llamada realizar. Por ejemplo, al tratar con archivos, el sistema operativo necesita saber con qué archivo estás tratando, qué datos quieres escribir y otros detalles. Los detalles adicionales, llamados **parámetros**, se almacenan en otros registros. En el caso de la llamada al sistema de salida (exit), el sistema operativo requiere que se cargue un código de estado en `%ebx`. Este valor se devuelve luego al sistema. Este es el valor que recuperaste cuando escribiste `echo $?`. Por lo tanto, cargamos `%ebx` con 0 escribiendo lo siguiente:

```assembly
movl $0, %ebx
```

Ahora bien, cargar los registros con estos números no hace nada por sí mismo. Los registros se usan para todo tipo de cosas además de las llamadas al sistema. Es donde ocurre toda la lógica del programa, como sumas, restas y comparaciones. Linux simplemente requiere que ciertos registros se carguen con ciertos valores de parámetros antes de realizar una llamada al sistema. Siempre se requiere que `%eax` se cargue con el número de la llamada al sistema. Para los otros registros, sin embargo, cada llamada al sistema tiene requisitos diferentes. En la llamada al sistema de salida, se requiere que `%ebx` se cargue con el estado de salida. Discutiremos diferentes llamadas al sistema a medida que sean necesarias. Para una lista de llamadas al sistema comunes y lo que se requiere en cada registro, consulta el Apéndice C.

La siguiente instrucción es la "mágica". Se ve así:

```assembly
int $0x80
```

`int` significa **interrupción** (interrupt). El `0x80` es el número de interrupción a usar. [^6] Una interrupción interrumpe el flujo normal del programa y transfiere el control de nuestro programa a Linux para que realice una llamada al sistema. [^7] Puedes pensarlo como llamar a Batman (o Larry-Boy [^8], si lo prefieres). Necesitas que se haga algo, envías la señal y luego él viene al rescate. No te importa cómo hace su trabajo, es más o menos magia, y cuando termina vuelves a tener el control. En este caso, lo único que estamos haciendo es pedirle a Linux que termine el programa, en cuyo caso no volveremos a tener el control. Si no señaláramos la interrupción, no se habría realizado ninguna llamada al sistema.

> **Revisión Rápida de Llamadas al Sistema:** Para recapitular: las funciones del Sistema Operativo se acceden a través de llamadas al sistema. Estas se invocan configurando los registros de una manera especial y emitiendo la instrucción `int $0x80`. Linux sabe a qué llamada al sistema queremos acceder por lo que almacenamos en el registro `%eax`. Cada llamada al sistema tiene otros requisitos sobre lo que debe almacenarse en los otros registros. El número de llamada al sistema 1 es la llamada al sistema *exit*, que requiere que el código de estado se coloque en `%ebx`.

Ahora que has ensamblado, enlazado, ejecutado y examinado el programa, deberías realizar algunas ediciones básicas. Haz cosas como cambiar el número que se carga en `%ebx` y observa cómo sale al final con `echo $?`. No olvides ensamblarlo y enlazarlo de nuevo antes de ejecutarlo. Añade algunos comentarios. No te preocupes, lo peor que podría pasar es que el programa no se ensamble o no se enlace, o que congele tu pantalla. ¡Eso es solo parte del aprendizaje!

## Planificación del Programa

En nuestro próximo programa intentaremos encontrar el máximo de una lista de números. Las computadoras están muy orientadas a los detalles, por lo que para escribir el programa tendremos que haber planificado una serie de detalles. Estos detalles incluyen:

- ¿Dónde se almacenará la lista original de números?
- ¿Qué procedimiento necesitaremos seguir para encontrar el número máximo?
- ¿Cuánto almacenamiento necesitamos para llevar a cabo ese procedimiento?
- ¿Caberá todo el almacenamiento en registros o necesitamos usar también algo de memoria?

Podrías pensar que algo tan simple como encontrar el número máximo de una lista no requeriría mucha planificación. Normalmente puedes decirle a la gente que encuentre el número máximo y pueden hacerlo sin muchos problemas. Sin embargo, nuestras mentes están acostumbradas a armar tareas complejas automáticamente. Las computadoras necesitan ser instruidas a través del proceso. Además, generalmente podemos mantener cualquier número de cosas en nuestra mente sin muchos problemas. Normalmente ni siquiera nos damos cuenta de que lo estamos haciendo. Por ejemplo, si escaneas una lista de números buscando el máximo, probablemente mantendrás en mente tanto el número más alto que has visto hasta ahora como en qué parte de la lista te encuentras. Mientras que tu mente hace esto automáticamente, con las computadoras tienes que configurar explícitamente el almacenamiento para mantener la posición actual en la lista y el número máximo actual. También tienes otros problemas, como saber cuándo parar. Al leer un papel, puedes parar cuando se te acaban los números. Sin embargo, la computadora solo contiene números, por lo que no tiene idea de cuándo ha llegado al último de *tus* números.

En las computadoras, tienes que planificar cada paso del camino. Así que, hagamos un poco de planificación. En primer lugar, solo como referencia, nombremos la dirección donde comienza la lista de números como `data_items`. Digamos que el último número de la lista será un cero, así sabemos dónde parar. También necesitamos un valor para mantener la posición actual en la lista, un valor para mantener el elemento de la lista actual que se está examinando y el valor más alto actual de la lista. Asignemos a cada uno de estos un registro:

- `%edi` mantendrá la posición actual en la lista.
- `%ebx` mantendrá el valor más alto actual en la lista.
- `%eax` mantendrá el elemento actual que se está examinando.

Cuando comencemos el programa y miremos el primer elemento de la lista, dado que no hemos visto ningún otro elemento, ese elemento será automáticamente el elemento más grande actual en la lista. Además, estableceremos la posición actual en la lista en cero: el primer elemento. A partir de ahí, seguiremos los siguientes pasos:

1. Verificar el elemento actual de la lista (`%eax`) para ver si es cero (el elemento de terminación).
2. Si es cero, salir.
3. Incrementar la posición actual (`%edi`).
4. Cargar el siguiente valor de la lista en el registro de valor actual (`%eax`). ¿Qué modo de direccionamiento podríamos usar aquí? ¿Por qué?
5. Comparar el valor actual (`%eax`) con el valor más alto actual (`%ebx`).
6. Si el valor actual es mayor que el valor más alto actual, reemplazar el valor más alto actual con el valor actual.
7. Repetir.

Ese es el procedimiento. Muchas veces en ese procedimiento utilicé la palabra "si". Estos lugares son donde se deben tomar decisiones. Como ves, la computadora no sigue exactamente la misma secuencia de instrucciones cada vez. Dependiendo de qué "si" sean correctos, la computadora puede seguir un conjunto diferente de instrucciones. La segunda vez, puede que no tenga el valor más alto. En ese caso, omitirá el paso 6, pero volverá al paso 7. En todos los casos excepto en el último, omitirá el paso 2. En programas más complicados, los saltos aumentan drásticamente.

Estos "si" son una clase de instrucciones llamadas **instrucciones de control de flujo** (flow control instructions), porque le dicen a la computadora qué pasos seguir y qué caminos tomar. En el programa anterior, no teníamos ninguna instrucción de control de flujo, ya que solo había un camino posible a tomar: salir. Este programa es mucho más dinámico ya que está dirigido por datos. Dependiendo de qué datos reciba, seguirá diferentes rutas de instrucciones.

En este programa, esto se logrará mediante dos instrucciones diferentes, el **salto condicional** (conditional jump) y el **salto incondicional** (unconditional jump). El salto condicional cambia de ruta basándose en los resultados de una comparación o cálculo anterior. El salto incondicional simplemente va directamente a una ruta diferente sin importar qué. El salto incondicional puede parecer inútil, pero es muy necesario ya que todas las instrucciones se dispondrán en línea. Si una ruta necesita converger de nuevo a la ruta principal, tendrá que hacerlo mediante un salto incondicional. Veremos más de ambos tipos de saltos en la siguiente sección.

Otro uso del control de flujo es en la implementación de **bucles** (loops). Un bucle es una parte del código del programa que está destinada a ser repetida. En nuestro ejemplo, la primera parte del programa (establecer la posición actual en 0 y cargar el valor más alto actual con el valor actual) solo se hizo una vez, por lo que no fue un bucle. Sin embargo, la siguiente parte se repite una y otra vez para cada número de la lista. Solo se abandona cuando llegamos al último elemento, indicado por un cero. Esto se llama **bucle** porque ocurre una y otra vez. Se implementa haciendo saltos incondicionales al principio del bucle al final del mismo, lo que hace que comience de nuevo. Sin embargo, ¡siempre debes recordar tener un salto condicional para salir del bucle en algún lugar, o el bucle continuará para siempre! Esta condición se llama **bucle infinito** (infinite loop). Si accidentalmente omitiéramos el paso 1, 2 o 3, el bucle (y nuestro programa) nunca terminaría.

En la siguiente sección, implementaremos este programa que hemos planeado. La planificación de programas suena complicada, y lo es, hasta cierto punto. Cuando empiezas a programar, a menudo es difícil convertir nuestro proceso de pensamiento normal en un procedimiento que la computadora pueda entender. A menudo olvidamos la cantidad de "ubicaciones de almacenamiento temporal" que nuestras mentes están usando para procesar problemas. A medida que leas y escribas programas, sin embargo, esto eventualmente se volverá muy natural para ti. Solo ten paciencia.

## Encontrar un Valor Máximo

Introduce el siguiente programa como `maximum.s`:

```assembly
#PROPÓSITO: Este programa encuentra el número máximo de un
#           conjunto de elementos de datos.
#
#VARIABLES: Los registros tienen los siguientes usos:
#
# %edi - Mantiene el índice del elemento de datos que se está examinando
# %ebx - El elemento de datos más grande encontrado
# %eax - Elemento de datos actual
#
# Se utilizan las siguientes ubicaciones de memoria:
#
# data_items - contiene los datos de los elementos. Se usa un 0
#              para terminar los datos
#
.section .data
data_items:         #Estos son los elementos de datos
 .long 3,67,34,222,45,75,54,34,44,33,22,11,66,0

.section .text
.globl _start
_start:
 movl $0, %edi                   # mueve 0 al registro de índice
 movl data_items(,%edi,4), %eax  # carga el primer byte de datos
 movl %eax, %ebx                 # como este es el primer elemento, %eax es
                                 # el más grande

start_loop:                      # inicio del bucle
 cmpl $0, %eax                   # comprueba si hemos llegado al final
 je loop_exit
 incl %edi                       # carga el siguiente valor
 movl data_items(,%edi,4), %eax
 cmpl %ebx, %eax                 # compara los valores
 jle start_loop                  # salta al principio del bucle si el nuevo
                                 # no es más grande
 movl %eax, %ebx                 # mueve el valor como el más grande
 jmp start_loop                  # salta al principio del bucle

loop_exit:
 # %ebx es el código de estado para la llamada al sistema exit
 # y ya contiene el número máximo
 movl $1, %eax                   # 1 es la llamada al sistema exit()
 int $0x80
```

Ahora, ensámblalo y enlázalo con estos comandos:

```bash
as maximum.s -o maximum.o
ld maximum.o -o maximum
```

Ahora ejecútalo y comprueba su estado:

```bash
./maximum
echo $?
```

Notarás que devuelve el valor 222. Echemos un vistazo al programa y lo que hace. Si miras en los comentarios, verás que el programa encuentra el máximo de un conjunto de números (¡los comentarios son maravillosos!). También puedes notar que en este programa realmente tenemos algo en la sección de datos. Estas líneas son la sección de datos:

```assembly
data_items: #Estos son los elementos de datos
 .long 3,67,34,222,45,75,54,34,44,33,22,11,66,0
```

Veamos esto. `data_items` es una etiqueta que se refiere a la ubicación que la sigue. Luego, hay una directiva que comienza con `.long`. Eso hace que el ensamblador reserve memoria para la lista de números que la siguen. `data_items` se refiere a la ubicación del primero. Debido a que `data_items` es una etiqueta, en cualquier momento de nuestro programa en el que necesitemos referirnos a esta dirección podemos usar el símbolo `data_items`, y el ensamblador lo sustituirá por la dirección donde comienzan los números durante el ensamblaje. Por ejemplo, la instrucción `movl data_items, %eax` movería el valor 3 a `%eax`. Hay varios tipos diferentes de ubicaciones de memoria además de `.long` que pueden reservarse. Los principales son los siguientes:

- **`.byte`**: Los bytes ocupan una ubicación de almacenamiento para cada número. Están limitados a números entre 0 y 255.
- **`.int`**: Los ints (que difieren de la instrucción `int`) ocupan dos ubicaciones de almacenamiento para cada número. Estos están limitados a números entre 0 y 65535. [^9]
- **`.long`**: Los longs ocupan cuatro ubicaciones de almacenamiento. Esta es la misma cantidad de espacio que usan los registros, razón por la cual se usan en este programa. Pueden contener números entre 0 y 4294967295.
- **`.ascii`**: La directiva `.ascii` es para introducir caracteres en la memoria. Cada carácter ocupa una ubicación de almacenamiento (se convierten internamente en bytes). Por lo tanto, si dieras la directiva `.ascii "Hello there\0"`, el ensamblador reservaría 12 ubicaciones de almacenamiento (bytes). El primer byte contiene el código numérico para `H`, el segundo byte contiene el código numérico para `e`, y así sucesivamente. El último carácter está representado por `\0`, y es el carácter de terminación (nunca se mostrará, simplemente le dice a otras partes del programa que ese es el final de los caracteres). Las letras y números que comienzan con una barra invertida representan caracteres que no se pueden escribir en el teclado o que no se ven fácilmente en la pantalla. Por ejemplo, `\n` se refiere al carácter de "nueva línea" (newline) que hace que la computadora comience la salida en la siguiente línea y `\t` se refiere al carácter de "tabulación" (tab). Todas las letras en una directiva `.ascii` deben estar entre comillas.

En nuestro ejemplo, el ensamblador reserva 14 `.long`s, uno justo después del otro. Como cada long ocupa 4 bytes, eso significa que toda la lista ocupa 56 bytes. Estos son los números que buscaremos para encontrar el máximo. `data_items` es utilizado por el ensamblador para referirse a la dirección del primero de estos valores.

Ten en cuenta que el último elemento de datos de la lista es un cero. Decidí usar un cero para decirle a mi programa que ha llegado al final de la lista. Podría haberlo hecho de otras maneras. Podría haber tenido el tamaño de la lista codificado en el programa. Además, podría haber puesto la longitud de la lista como el primer elemento, o en una ubicación separada. También podría haber creado un símbolo que marcara la última ubicación de los elementos de la lista. No importa cómo lo haga, debo tener algún método para determinar el final de la lista. La computadora no sabe nada, solo puede hacer lo que se le dice. No va a dejar de procesar a menos que le dé algún tipo de señal. De lo contrario, continuaría procesando más allá del final de la lista en los datos que la siguen, e incluso en ubicaciones donde no hemos puesto ningún dato.

Observa que no tenemos una declaración `.globl` para `data_items`. Esto se debe a que solo nos referimos a estas ubicaciones dentro del programa. Ningún otro archivo o programa necesita saber dónde se encuentran. Esto contrasta con el símbolo `_start`, que Linux necesita saber dónde está para saber por dónde comenzar la ejecución del programa. No es un error escribir `.globl data_items`, simplemente no es necesario. De todos modos, juega con esta línea y añade tus propios números. Aunque sean `.long`, el programa producirá resultados extraños si algún número es mayor que 255, porque ese es el estado de salida máximo permitido. También nota que si mueves el 0 a una posición anterior de la lista, el resto se ignora. Recuerda que cada vez que cambies el archivo fuente, tienes que volver a ensamblar y enlazar tu programa. Hazlo ahora y mira los resultados.

Muy bien, hemos jugado un poco con los datos. Ahora miremos el código. En los comentarios notarás que hemos marcado algunas variables que planeamos usar. Una **variable** es una ubicación de almacenamiento dedicada utilizada para un propósito específico, generalmente con un nombre distintivo dado por el programador. Hablamos de estas en la sección anterior, pero no les dimos un nombre. En este programa, tenemos varias variables:

- una variable para el número máximo actual encontrado
- una variable para saber qué número de la lista estamos examinando actualmente, llamada **índice** (index)
- una variable que contiene el número actual que se está examinando

En este caso, tenemos tan pocas variables que podemos mantenerlas todas en registros. En programas más grandes, tienes que ponerlas en la memoria y luego moverlas a los registros cuando estés listo para usarlas. Discutiremos cómo hacerlo más adelante. Cuando la gente empieza a programar, suele subestimar el número de variables que necesitará. Las personas no están acostumbradas a tener que pensar en cada detalle de un proceso y, por lo tanto, omiten variables necesarias en sus primeros intentos de programación.

En este programa, estamos usando `%ebx` como la ubicación del elemento más grande que hemos encontrado. `%edi` se usa como el **índice** para el elemento de datos actual que estamos mirando. Ahora, hablemos de lo que es un índice. Cuando leemos la información de `data_items`, comenzaremos con el primero (elemento de datos número 0), luego iremos al segundo (elemento de datos número 1), luego al tercero (elemento de datos número 2), y así sucesivamente. El número de elemento de datos es el **índice** de `data_items`. Notarás que la primera instrucción que le damos a la computadora es:

```assembly
movl $0, %edi
```

Dado que estamos usando `%edi` como nuestro índice, y queremos empezar a mirar el primer elemento, cargamos `%edi` con 0. Ahora, la siguiente instrucción es truculenta, pero crucial para lo que estamos haciendo. Dice:

```assembly
movl data_items(,%edi,4), %eax
```

Ahora, para entender esta línea, debes tener en cuenta varias cosas:

- `data_items` es el número de ubicación del inicio de nuestra lista de números.
- Cada número se almacena a lo largo de 4 ubicaciones de almacenamiento (porque lo declaramos usando `.long`)
- `%edi` contiene 0 en este punto

Básicamente, lo que hace esta línea es decir: "comienza al principio de `data_items`, toma el primer número de elemento (porque `%edi` es 0) y recuerda que cada número ocupa cuatro ubicaciones de almacenamiento". Luego almacena ese número en `%eax`. Así es como se escriben las instrucciones en modo de direccionamiento indexado en lenguaje ensamblador. La instrucción en forma general es esta:

`movl DIRECCIÓN_INICIAL(,%REGISTRO_ÍNDICE,TAMAÑO_PALABRA)`

En nuestro caso, `data_items` era nuestra dirección inicial, `%edi` era nuestro registro de índice y 4 era nuestro tamaño de palabra. Este tema se discute más a fondo en la sección llamada *Modos de Direccionamiento*.

Si miras los números en `data_items`, verás que el número 3 está ahora en `%eax`. Si `%edi` se estableciera en 1, el número 67 estaría en `%eax`, y si se estableciera en 2, el número 34 estaría en `%eax`, y así sucesivamente. Pasarían cosas muy extrañas si usáramos un número distinto de 4 como el tamaño de nuestras ubicaciones de almacenamiento. [^10] La forma en que se escribe esto es muy peculiar, pero si sabes qué hace cada pieza, no es demasiado difícil. Para más información sobre esto, consulta la sección llamada *Modos de Direccionamiento*.

Miremos la siguiente línea:

```assembly
movl %eax, %ebx
```

Tenemos el primer elemento a mirar almacenado en `%eax`. Como es el primer elemento, sabemos que es el más grande que hemos mirado. Lo almacenamos en `%ebx`, ya que es allí donde guardamos el número más grande encontrado. Además, aunque `movl` significa mover (move), en realidad copia el valor, por lo que tanto `%eax` como `%ebx` contienen el valor inicial. [^11]

Ahora entramos en un **bucle** (loop). Un bucle es un segmento de tu programa que podría ejecutarse más de una vez. Hemos marcado la ubicación inicial del bucle en el símbolo `start_loop`. La razón por la que estamos haciendo un bucle es porque no sabemos cuántos elementos de datos tenemos que procesar, pero el procedimiento será el mismo sin importar cuántos haya. No queremos tener que reescribir nuestro programa para cada longitud de lista posible. De hecho, ni siquiera queremos tener que escribir código para una comparación por cada elemento de la lista. Por lo tanto, tenemos una sola sección de código (un bucle) que ejecutamos una y otra vez para cada elemento en `data_items`.

En la sección anterior, esbozamos lo que este bucle necesitaba hacer. Repasemos:

- Comprobar si el valor actual que se está mirando es cero. Si es así, significa que estamos al final de nuestros datos y debemos salir del bucle.
- Tenemos que cargar el siguiente valor de nuestra lista.
- Tenemos que ver si el siguiente valor es más grande que nuestro valor más grande actual.
- Si lo es, tenemos que copiarlo a la ubicación en la que guardamos el valor más grande.
- Ahora tenemos que volver al principio del bucle.

Bien, ahora vayamos al código. Tenemos el inicio del bucle marcado con `start_loop`. Eso es para que sepamos a dónde volver al final de nuestro bucle. Luego tenemos estas instrucciones:

```assembly
cmpl $0, %eax
je loop_exit
```

La instrucción `cmpl` compara los dos valores. Aquí, estamos comparando el número 0 con el número almacenado en `%eax`. Esta instrucción de comparación también afecta a un registro no mencionado aquí, el registro `%eflags`. Este también se conoce como el **registro de estado** (status register) y tiene muchos usos que discutiremos más adelante. Solo ten en cuenta que el resultado de la comparación se almacena en el registro de estado. La siguiente línea es una instrucción de control de flujo que dice **saltar** (jump) a la ubicación `loop_exit` si los valores que se acaban de comparar son iguales (eso es lo que significa la *e* de `je`, del inglés *equal*). Utiliza el registro de estado para mantener el valor de la última comparación. Usamos `je`, pero hay muchas sentencias de salto que puedes usar:

- **`je`**: Salta si los valores eran iguales
- **`jg`**: Salta si el segundo valor era mayor que el primer valor [^12]
- **`jge`**: Salta si el segundo valor era mayor o igual que el primer valor
- **`jl`**: Salta si el segundo valor era menor que el primer valor
- **`jle`**: Salta si el segundo valor era menor o igual que el primer valor
- **`jmp`**: Salta sin importar qué. Esto no necesita ir precedido de una comparación.

La lista completa está documentada en el Apéndice B. En este caso, saltamos si `%eax` contiene el valor cero. Si es así, hemos terminado y vamos a `loop_exit`. [^13]

Si el último elemento cargado no fue cero, pasamos a las siguientes instrucciones:

```assembly
incl %edi
movl data_items(,%edi,4), %eax
```

Si recuerdas de nuestra discusión anterior, `%edi` contiene el índice de nuestra lista de valores en `data_items`. `incl` incrementa el valor de `%edi` en uno. Luego el `movl` es igual al que hicimos anteriormente. Sin embargo, como ya incrementamos `%edi`, `%eax` está obteniendo el siguiente valor de la lista. Ahora `%eax` tiene el siguiente valor a ser probado. ¡Así que probémoslo!

```assembly
cmpl %ebx, %eax
jle start_loop
```

Aquí comparamos nuestro valor actual, almacenado en `%eax`, con nuestro valor más grande hasta ahora, almacenado en `%ebx`. Si el valor actual es menor o igual a nuestro valor más grande hasta ahora, no nos interesa, así que simplemente saltamos de vuelta al principio del bucle. De lo contrario, necesitamos registrar ese valor como el más grande:

```assembly
movl %eax, %ebx
jmp start_loop
```

lo que mueve el valor actual a `%ebx`, que estamos usando para almacenar el valor más grande actual, y comienza el bucle de nuevo.

Bien, el bucle se ejecuta hasta que llega a un 0, momento en el que salta a `loop_exit`. Esta parte del programa llama al kernel de Linux para salir. Si recuerdas del último programa, cuando llamas al sistema operativo (recuerda que es como llamar a Batman), almacenas el número de la llamada al sistema en `%eax` (1 para la llamada exit) y almacenas los otros valores en los otros registros. La llamada exit requiere que pongamos nuestro estado de salida en `%ebx`. Ya tenemos el estado de salida allí puesto que estamos usando `%ebx` como nuestro número más grande, así que todo lo que tenemos que hacer es cargar `%eax` con el número uno y llamar al kernel para salir. Así:

```assembly
movl $1, %eax
int $0x80
```

Bien, eso fue mucho trabajo y explicación, especialmente para un programa tan pequeño. Pero oye, ¡estás aprendiendo mucho! Ahora, lee todo el programa de nuevo, prestando especial atención a los comentarios. Asegúrate de entender lo que sucede en cada línea. Si no entiendes una línea, vuelve a leer esta sección y averigua qué significa.

También podrías tomar un trozo de papel e ir recorriendo el programa paso a paso, anotando cada cambio en cada registro, para que puedas ver más claramente lo que está pasando.

## Modos de Direccionamiento

En la sección llamada *Métodos de Acceso a Datos* en el Capítulo 2 aprendimos los diferentes tipos de modos de direccionamiento disponibles para su uso en lenguaje ensamblador. Esta sección tratará sobre cómo se representan esos modos de direccionamiento en las instrucciones del lenguaje ensamblador.

La forma general de las referencias a direcciones de memoria es esta:

`DIRECCIÓN_O_DESPLAZAMIENTO(%BASE_O_DESPLAZAMIENTO,%ÍNDICE,MULTIPLICADOR)`

Todos los campos son opcionales. Para calcular la dirección, simplemente realiza el siguiente cálculo:

`DIRECCIÓN FINAL = DIRECCIÓN_O_DESPLAZAMIENTO + %BASE_O_DESPLAZAMIENTO + MULTIPLICADOR * %ÍNDICE`

`DIRECCIÓN_O_DESPLAZAMIENTO` y `MULTIPLICADOR` deben ser constantes, mientras que los otros dos deben ser registros. Si alguna de las piezas se omite, simplemente se sustituye por cero en la ecuación.

Todos los modos de direccionamiento mencionados en la sección llamada *Métodos de Acceso a Datos* en el Capítulo 2, excepto el modo inmediato, pueden representarse de esta manera.

**modo de direccionamiento directo**

Esto se hace usando solo la parte `DIRECCIÓN_O_DESPLAZAMIENTO`. Ejemplo:

`movl DIRECCIÓN, %eax`

Esto carga `%eax` con el valor en la dirección de memoria `DIRECCIÓN`.

**modo de direccionamiento indexado**

Esto se hace usando la parte `DIRECCIÓN_O_DESPLAZAMIENTO` y la parte `%ÍNDICE`. Puedes usar cualquier registro de propósito general como registro de índice. También puedes tener un multiplicador constante de 1, 2 o 4 para el registro de índice, para facilitar la indexación por bytes, bytes dobles y palabras. Por ejemplo, supongamos que tenemos una cadena de bytes llamada `string_start` y queremos acceder al tercero (un índice de 2 ya que empezamos a contar el índice en cero), y `%ecx` contiene el valor 2. Si quisieras cargarlo en `%eax`, podrías hacer lo siguiente:

`movl string_start(,%ecx,1), %eax`

Esto comienza en `string_start`, suma `1 * %ecx` a esa dirección y carga el valor en `%eax`.

**modo de direccionamiento indirecto**

El modo de direccionamiento indirecto carga un valor desde la dirección indicada por un registro. Por ejemplo, si `%eax` contuviera una dirección, podríamos mover el valor en esa dirección a `%ebx` haciendo lo siguiente:

`movl (%eax), %ebx`

**modo de direccionamiento por puntero base**

El direccionamiento por puntero base es similar al direccionamiento indirecto, excepto que suma un valor constante a la dirección del registro. Por ejemplo, si tienes un registro donde el valor de la edad está a 4 bytes del inicio del registro, y tienes la dirección del registro en `%eax`, puedes recuperar la edad en `%ebx` emitiendo la siguiente instrucción:

`movl 4(%eax), %ebx`

**modo inmediato**

El modo inmediato es muy simple. No sigue la forma general que hemos estado usando. El modo inmediato se usa para cargar valores directos en registros o ubicaciones de memoria. Por ejemplo, si quisieras cargar el número 12 en `%eax`, simplemente harías lo siguiente:

`movl $12, %eax`

Nota que para indicar el modo inmediato, usamos un signo de dólar delante del número. Si no lo hiciéramos, sería el modo de direccionamiento directo, en cuyo caso se cargaría en `%eax` el valor ubicado en la ubicación de memoria 12 en lugar del número 12 en sí.

**modo de direccionamiento por registro**

El modo de registro simplemente mueve datos dentro o fuera de un registro. En todos nuestros ejemplos, el modo de direccionamiento por registro se utilizó para el otro operando.

Estos modos de direccionamiento son muy importantes, ya que cada acceso a la memoria utilizará uno de ellos. Todos los modos, excepto el modo inmediato, pueden usarse como operando de origen o de destino. El modo inmediato solo puede ser un operando de origen.

Además de estos modos, también existen diferentes instrucciones para diferentes tamaños de valores a mover. Por ejemplo, hemos estado usando `movl` para mover datos una palabra a la vez. En muchos casos, solo querrás mover datos un byte a la vez. Esto se logra mediante la instrucción `movb`. Sin embargo, dado que los registros que hemos discutido son del tamaño de una palabra y no de un byte, no puedes usar el registro completo. En su lugar, tienes que usar una parte del registro.

Toma por ejemplo `%eax`. Si solo quisieras trabajar con dos bytes a la vez, podrías usar `%ax`. `%ax` es la mitad menos significativa (es decir, la última parte del número) del registro `%eax`, y es útil cuando se trata de cantidades de dos bytes. `%ax` se divide además en `%al` y `%ah`. `%al` es el byte menos significativo de `%ax` y `%ah` es el byte más significativo. [^14] Cargar un valor en `%eax` borrará lo que hubiera en `%al` y `%ah` (y también en `%ax`, ya que `%ax` está compuesto por ellos). De manera similar, cargar un valor en `%al` o `%ah` corromperá cualquier valor que estuviera anteriormente en `%eax`. Básicamente, es prudente usar un registro solo para un byte o para una palabra, pero nunca para ambos al mismo tiempo.

### Disposición del registro %eax

| 31 ... 16 | 15 ... 8 (%ah) | 7 ... 0 (%al) |
|:---:|:---:|:---:|
| | \<\-\-\-\-\-\-\- | %ax | \-\-\-\-\-\-\-\> |
| \<\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\- | %eax | \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\> |

Para una lista más completa de instrucciones, consulta el Apéndice B.

## Revisión

### Conoce los Conceptos

- ¿Qué significa que una línea en el programa comience con el carácter '#'?
- ¿Cuál es la diferencia entre un archivo de lenguaje ensamblador y un archivo de código objeto?
- ¿Qué hace el enlazador (linker)?
- ¿Cómo compruebas el código de estado del resultado del último programa que ejecutaste?
- ¿Cuál es la diferencia entre `movl $1, %eax` y `movl 1, %eax`?
- ¿Qué registro contiene el número de la llamada al sistema?
- ¿Para qué se utilizan los índices?
- ¿Por qué los índices suelen empezar en 0?
- Si emitiera el comando `movl data_items(,%edi,4), %eax` y `data_items` fuera la dirección 3634 y `%edi` contuviera el valor 13, ¿qué dirección estarías usando para mover a `%eax`?
- Enumera los registros de propósito general.
- ¿Cuál es la diferencia entre `movl` y `movb`?
- ¿Qué es el control de flujo?
- ¿Qué hace un salto condicional?
- ¿Qué cosas tienes que planificar al escribir un programa?
- Repasa cada instrucción y enumera qué modo de direccionamiento se está utilizando para cada operando.

### Usa los Conceptos

- Modifica el primer programa para que devuelva el valor 3.
- Modifica el programa del máximo para que encuentre el mínimo en su lugar.
- Modifica el programa del máximo para que use el número 255 para finalizar la lista en lugar del número 0.
- Modifica el programa del máximo para que use una dirección final en lugar del número 0 para saber cuándo parar.
- Modifica el programa del máximo para que use un conteo de longitud en lugar del número 0 para saber cuándo parar.
- ¿Qué haría la instrucción `movl _start, %eax`? Sé específico, basándote en tu conocimiento tanto de los modos de direccionamiento como del significado de `_start`. ¿En qué se diferenciaría esto de la instrucción `movl $_start, %eax`?

### Yendo más allá

- Modifica el primer programa para omitir la línea de la instrucción `int`. Ensambla, enlaza y ejecuta el nuevo programa. ¿Qué mensaje de error obtienes? ¿Por qué crees que puede ser esto?
- Hasta ahora, hemos discutido tres enfoques para encontrar el final de la lista: usar un número especial, usar la dirección final y usar el conteo de longitud. ¿Qué enfoque crees que es mejor? ¿Por qué? ¿Qué enfoque usarías si supieras que la lista está ordenada? ¿Por qué?

---

[^1]: Si eres nuevo en Linux y UNIX®, puede que no sepas que los archivos no tienen por qué tener extensiones. De hecho, mientras que Windows® usa la extensión .exe para significar un programa ejecutable, los ejecutables de UNIX normalmente no tienen extensión.
[^2]: `.` se refiere al directorio actual en los sistemas Linux y UNIX.
[^3]: Descubrirás que muchos programas terminan haciendo las cosas de formas extrañas. Normalmente hay una razón para ello, pero, desafortunadamente, los programadores nunca documentan tales cosas en sus comentarios. Por lo tanto, los futuros programadores tienen que aprender la razón de la manera difícil modificando el código y viendo cómo se rompe, o simplemente dejándolo en paz esté o no todavía en uso. Siempre debes documentar cualquier comportamiento extraño que realice tu programa. Desafortunadamente, descubrir qué es extraño y qué es directo viene principalmente con la experiencia.
[^4]: Ten en cuenta que en los procesadores x86, incluso los registros de propósito general tienen algunos propósitos especiales, o los tenían antes de que pasaran a ser de 32 bits. Sin embargo, estos son registros de propósito general para la mayoría de las instrucciones. Cada uno de ellos tiene al menos una instrucción donde se usa de manera especial. Sin embargo, para la mayoría de ellos, esas instrucciones no se cubren en este libro.
[^5]: Quizás te preguntes, ¿por qué todos estos registros comienzan con la letra *e*? La razón es que las primeras generaciones de procesadores x86 eran de 16 bits en lugar de 32 bits. Por lo tanto, los registros tenían solo la mitad de la longitud que tienen ahora. En generaciones posteriores de procesadores x86, el tamaño de los registros se duplicó. Mantuvieron los nombres antiguos para referirse a la primera mitad del registro y añadieron una *e* (de *extended*) para referirse a las versiones extendidas del registro. Normalmente solo usarás las versiones extendidas. Los modelos más nuevos también ofrecen un modo de 64 bits, que duplica el tamaño de estos registros una vez más y utiliza un prefijo *r* para indicar los registros más grandes (es decir, `%rax` es la versión de 64 bits de `%eax`). Sin embargo, estos procesadores no se usan tan ampliamente y no se cubren en este libro.
[^6]: Quizás te preguntes por qué es `0x80` en lugar de simplemente 80. La razón es que el número está escrito en hexadecimal. En hexadecimal, un solo dígito puede contener 16 valores en lugar de los 10 normales. Esto se hace utilizando las letras de la 'a' a la 'f' además de los dígitos normales. 'a' representa 10, 'b' representa 11, y así sucesivamente. `0x10` representa el número 16, y así sucesivamente. Esto se discutirá más a fondo más adelante, pero solo ten en cuenta que los números que comienzan con `0x` están en hexadecimal. A veces también se usa añadir una `H` al final, pero no lo haremos en este libro. Para más información sobre esto, consulta el Capítulo 10.
[^7]: En realidad, la interrupción transfiere el control a quienquiera que haya configurado un manejador de interrupciones para el número de interrupción. En el caso de Linux, todas ellas están configuradas para ser manejadas por el kernel de Linux.
[^8]: Si no ves Veggie Tales, deberías hacerlo. Empieza con *Dave and the Giant Pickle*.
[^9]: Ten en cuenta que ningún número en lenguaje ensamblador (o en cualquier otro lenguaje de computadora que haya visto) tiene comas incrustadas. Por lo tanto, escribe siempre los números como 65535, y nunca como 65,535.
[^10]: La instrucción en realidad no usa 4 para el tamaño de las ubicaciones de almacenamiento, aunque verlo de esa manera funciona para nuestros propósitos ahora. Es en realidad lo que se llama un multiplicador. Básicamente, la forma en que funciona es que empiezas en la ubicación especificada por `data_items`, luego añades `%edi * 4` ubicaciones de almacenamiento y recuperas el número allí. Normalmente, usas el tamaño de los números como multiplicador, pero en algunas circunstancias querrás hacer otras cosas.
[^11]: Además, la *l* en `movl` significa *move long*, ya que estamos moviendo un valor que ocupa cuatro ubicaciones de almacenamiento.
[^12]: Nota que la comparación es para ver si el *segundo* valor es mayor que el primero. Yo habría pensado que era al revés. Encontrarás muchas cosas como esta cuando aprendas programación. Ocurre porque diferentes cosas tienen sentido para diferentes personas. De todos modos, tendrás que memorizar tales cosas y seguir adelante.
[^13]: Los nombres de estos símbolos pueden ser los que quieras, siempre que solo contengan letras y el carácter de subrayado (`_`). El único que es forzado es `_start`, y posiblemente otros que declares con `.globl`. Sin embargo, si es un símbolo que tú defines y solo tú usas, siéntete libre de llamarlo como quieras siempre que sea adecuadamente descriptivo (recuerda que otros tendrán que modificar tu código más tarde y tendrán que averiguar qué significan tus símbolos).
[^14]: Cuando hablamos del byte más o menos significativo, puede ser un poco confuso. Tomemos el número 5432. En ese número, 54 es la mitad más significativa de ese número y 32 es la mitad menos significativa. No puedes dividirlo exactamente así para los registros, ya que operan en base 2 en lugar de en base 10, pero esa es la idea básica. Para más información sobre este tema, consulta el Capítulo 10.
