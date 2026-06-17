# Capítulo 4. Todo Sobre las Funciones

## Lidiando con la Complejidad

En el Capítulo 3, los programas que escribimos solo consistían en una sección de código. Sin embargo, si escribiéramos programas reales de esa manera, sería imposible mantenerlos. Sería muy difícil lograr que varias personas trabajaran en el proyecto, ya que cualquier cambio en una parte podría afectar negativamente a otra parte en la que otro desarrollador esté trabajando.

Para ayudar a los programadores a trabajar juntos en grupos, es necesario dividir los programas en piezas separadas, que se comuniquen entre sí a través de interfaces bien definidas. De esta manera, cada pieza puede ser desarrollada y probada independientemente de las demás, facilitando que múltiples programadores trabajen en el proyecto.

Los programadores utilizan **funciones** para dividir sus programas en piezas que pueden ser desarrolladas y probadas de forma independiente. Las funciones son unidades de código que realizan una tarea definida sobre tipos de datos especificados. Por ejemplo, en un programa procesador de textos, puedo tener una función llamada `handle_typed_character` que se activa cada vez que un usuario presiona una tecla. Los datos que utiliza la función probablemente serían la pulsación de tecla misma y el documento que el usuario tiene abierto actualmente. La función modificaría entonces el documento de acuerdo con la pulsación de tecla que se le informó.

Los elementos de datos que se le dan a una función para procesar se llaman sus **parámetros**. En el ejemplo del procesamiento de textos, la tecla que se presionó y el documento se considerarían parámetros de la función `handle_typed_characters`. La lista de parámetros y las expectativas de procesamiento de una función (lo que se espera que haga con los parámetros) se denominan la **interfaz** (interface) de la función. Se pone mucho cuidado en el diseño de las interfaces de las funciones, porque si se llaman desde muchos lugares dentro de un proyecto, es difícil cambiarlas si es necesario.

Un programa típico se compone de cientos o miles de funciones, cada una con una tarea pequeña y bien definida que realizar. Sin embargo, en última instancia, hay cosas para las que no puedes escribir funciones y que deben ser proporcionadas por el sistema. Estas se llaman **funciones primitivas** (o simplemente primitivas): son los elementos básicos sobre los cuales se construye todo lo demás. Por ejemplo, imagina un programa que dibuja una interfaz gráfica de usuario. Tiene que haber una función para crear los menús. Esa función probablemente llama a otras funciones para escribir texto, escribir iconos, pintar el fondo, calcular dónde está el puntero del ratón, etc. Sin embargo, finalmente, llegarán a un conjunto de primitivas proporcionadas por el sistema operativo para realizar dibujos básicos de líneas o puntos. La programación puede verse como la descomposición de un programa grande en piezas más pequeñas hasta llegar a las funciones primitivas, o como la construcción incremental de funciones sobre primitivas hasta enfocar la imagen general. En lenguaje ensamblador, las primitivas suelen ser lo mismo que las llamadas al sistema (system calls), aunque las llamadas al sistema no sean funciones verdaderas como las que trataremos en este capítulo.

## Cómo Funcionan las Funciones

Las funciones se componen de varias piezas diferentes:

- **nombre de la función**: El nombre de una función es un símbolo que representa la dirección donde comienza el código de la función. En lenguaje ensamblador, el símbolo se define escribiendo el nombre de la función como una etiqueta antes del código de la función. Esto es igual a las etiquetas que has usado para saltar.
- **parámetros de la función**: Los parámetros de una función son los elementos de datos que se entregan explícitamente a la función para su procesamiento. Por ejemplo, en matemáticas, existe la función seno. Si le pides a una computadora que encuentre el seno de 2, seno sería el nombre de la función y 2 sería el parámetro. Algunas funciones tienen muchos parámetros, otras no tienen ninguno. [^1]
- **variables locales**: Las variables locales son almacenamiento de datos que una función utiliza mientras procesa y que se descartan cuando la función retorna. Es como un bloc de notas de papel. Las funciones obtienen una nueva hoja de papel cada vez que se activan y tienen que tirarla cuando terminan el procesamiento. Las variables locales de una función no son accesibles para ninguna otra función dentro de un programa.
- **variables estáticas**: Las variables estáticas son almacenamiento de datos que una función utiliza mientras procesa y que no se descartan después, sino que se reutilizan cada vez que se activa el código de la función. Estos datos no son accesibles para ninguna otra parte del programa. Generalmente no se usan variables estáticas a menos que sea absolutamente necesario, ya que pueden causar problemas más adelante.
- **variables globales**: Las variables globales son almacenamiento de datos que una función utiliza para el procesamiento y que se gestionan fuera de la función. Por ejemplo, un editor de texto simple puede poner todo el contenido del archivo en el que está trabajando en una variable global para no tener que pasarlo a cada función que opere sobre él. [^2] Los valores de configuración también se almacenan a menudo en variables globales.
- **dirección de retorno**: La dirección de retorno (return address) es un parámetro "invisible" en el sentido de que no se usa directamente durante la función. La dirección de retorno es un parámetro que le dice a la función dónde reanudar la ejecución después de que la función se complete. Esto es necesario porque las funciones pueden ser llamadas para procesar desde muchas partes diferentes de tu programa, y la función necesita poder volver a donde fue llamada. En la mayoría de los lenguajes de programación, este parámetro se pasa automáticamente cuando se llama a la función. En lenguaje ensamblador, la instrucción `call` se encarga de pasar la dirección de retorno por ti, y `ret` se encarga de usar esa dirección para volver a donde llamaste a la función.
- **valor de retorno**: El valor de retorno (return value) es el método principal para transferir datos de vuelta al programa principal. La mayoría de los lenguajes de programación solo permiten un único valor de retorno para una función.

Estas piezas están presentes en la mayoría de los lenguajes de programación. Sin embargo, la forma en que especificas cada pieza es diferente en cada uno. La manera en que se almacenan las variables y cómo la computadora transfiere los parámetros y los valores de retorno también varía de un lenguaje a otro. Esta varianza se conoce como la **convención de llamada** (calling convention) de un lenguaje, porque describe cómo las funciones esperan obtener y recibir datos cuando son llamadas. [^3]

El lenguaje ensamblador puede usar cualquier convención de llamada que desee. Incluso puedes inventar una tú mismo. Sin embargo, si quieres interoperar con funciones escritas en otros lenguajes, debes obedecer sus convenciones de llamada. Usaremos la convención de llamada del lenguaje de programación C para nuestros ejemplos porque es la más utilizada y porque es el estándar para las plataformas Linux.

## Funciones en Lenguaje Ensamblador usando la Convención de Llamada de C

No puedes escribir funciones en lenguaje ensamblador sin entender cómo funciona la **pila** (stack) de la computadora. Cada programa de computadora que se ejecuta utiliza una región de memoria llamada pila para permitir que las funciones funcionen correctamente. Piensa en una pila como un montón de papeles en tu escritorio a los que se pueden añadir más indefinidamente. Generalmente mantienes las cosas en las que estás trabajando hacia la parte superior y quitas cosas a medida que terminas de trabajar con ellas.

Tu computadora también tiene una pila. La pila de la computadora reside en las direcciones de memoria más altas. Puedes meter valores en la parte superior de la pila mediante una instrucción llamada `pushl`, que empuja un registro o un valor de memoria a la parte superior de la pila. Bueno, decimos que es la parte superior, pero el "top" de la pila es en realidad la parte inferior de la memoria de la pila. Aunque esto es confuso, la razón es que cuando pensamos en una pila de cualquier cosa (platos, papeles, etc.), pensamos en añadir y quitar de la parte superior. Sin embargo, en la memoria, la pila comienza en la parte superior de la memoria y crece hacia abajo debido a consideraciones arquitectónicas. Por lo tanto, cuando nos refiramos a la "parte superior de la pila", recuerda que está en la parte inferior de la memoria de la pila. También puedes sacar valores de la parte superior usando una instrucción llamada `popl`. Esto elimina el valor superior de la pila y lo coloca en un registro o ubicación de memoria de tu elección.

Cuando metemos un valor en la pila, la parte superior de la pila se mueve para acomodar el valor adicional. De hecho, podemos meter valores continuamente en la pila y esta seguirá creciendo cada vez más hacia abajo en la memoria hasta que lleguemos a nuestro código o datos. Entonces, ¿cómo sabemos dónde está la "parte superior" actual de la pila? El registro de pila, `%esp`, siempre contiene un puntero a la parte superior actual de la pila, dondequiera que esté. Cada vez que metemos algo en la pila con `pushl`, a `%esp` se le resta 4 para que apunte a la nueva parte superior de la pila (recuerda que cada palabra tiene cuatro bytes de largo y la pila crece hacia abajo). Si queremos quitar algo de la pila, simplemente usamos la instrucción `popl`, que suma 4 a `%esp` y coloca el valor superior anterior en el registro que hayas especificado. `pushl` y `popl` toman cada uno un operando: el registro que se va a meter en la pila para `pushl`, o el que recibirá los datos que se sacan de la pila para `popl`.

Si simplemente queremos acceder al valor en la parte superior de la pila sin quitarlo, podemos usar el registro `%esp` en modo de direccionamiento indirecto. Por ejemplo, el siguiente código mueve lo que esté en la parte superior de la pila a `%eax`:

```assembly
movl (%esp), %eax
```

Si simplemente hiciéramos esto:

```assembly
movl %esp, %eax
```

entonces `%eax` solo contendría el puntero a la parte superior de la pila en lugar del valor en la parte superior. Poner `%esp` entre paréntesis hace que la computadora pase al modo de direccionamiento indirecto y, por lo tanto, obtenemos el valor apuntado por `%esp`. Si queremos acceder al valor justo debajo de la parte superior de la pila, podemos simplemente emitir esta instrucción:

```assembly
movl 4(%esp), %eax
```

Esta instrucción utiliza el modo de direccionamiento por puntero base (ver la sección llamada *Métodos de Acceso a Datos* en el Capítulo 2), que simplemente suma 4 a `%esp` antes de buscar el valor al que se apunta.

En la convención de llamada del lenguaje C, la pila es el elemento clave para implementar las variables locales, los parámetros y la dirección de retorno de una función.

Antes de ejecutar una función, un programa mete todos los parámetros de la función en la pila en el orden inverso al que están documentados. Luego, el programa emite una instrucción `call` indicando qué función desea iniciar. La instrucción `call` hace dos cosas. Primero, mete en la pila la dirección de la siguiente instrucción, que es la **dirección de retorno**. Luego, modifica el puntero de instrucción (`%eip`) para que apunte al inicio de la función. Así, en el momento en que comienza la función, la pila se ve así (la "parte superior" de la pila está en la parte inferior en este ejemplo):

| Contenido de la Pila | Desplazamiento (Offset) |
|:---:|:---:|
| Parámetro #N | |
| ... | |
| Parámetro 2 | |
| Parámetro 1 | |
| Dirección de Retorno | <--- (%esp) |

Cada uno de los parámetros de la función ha sido metido en la pila, y finalmente la dirección de retorno está allí. Ahora la función misma tiene algo de trabajo que hacer. Lo primero que hace es guardar el registro del puntero base actual, `%ebp`, haciendo `pushl %ebp`. El **puntero base** (base pointer) es un registro especial utilizado para acceder a los parámetros de la función y a las variables locales. A continuación, copia el puntero de pila a `%ebp` haciendo `movl %esp, %ebp`. Esto te permite acceder a los parámetros de la función como índices fijos desde el puntero base. Podrías pensar que puedes usar el puntero de pila para esto. Sin embargo, durante tu programa puedes hacer otras cosas con la pila, como meter argumentos para otras funciones.

Copiar el puntero de pila en el puntero base al comienzo de una función te permite saber siempre dónde están tus parámetros (y, como veremos, también las variables locales), incluso cuando estés metiendo y sacando cosas de la pila. `%ebp` siempre estará donde estaba el puntero de pila al comienzo de la función, por lo que es más o menos una referencia constante al **marco de pila** (stack frame). El marco de pila consta de todas las variables de la pila utilizadas dentro de una función, incluidos los parámetros, las variables locales y la dirección de retorno.

En este punto, la pila se ve así:

| Contenido de la Pila | Desplazamiento (Offset) |
|:---:|:---:|
| Parámetro #N | <--- N*4+4(%ebp) |
| ... | |
| Parámetro 2 | <--- 12(%ebp) |
| Parámetro 1 | <--- 8(%ebp) |
| Dirección de Retorno | <--- 4(%ebp) |
| Viejo %ebp | <--- (%esp) y (%ebp) |

Como puedes ver, se puede acceder a cada parámetro utilizando el modo de direccionamiento por puntero base usando el registro `%ebp`.

A continuación, la función reserva espacio en la pila para cualquier variable local que necesite. Esto se hace simplemente apartando el puntero de pila. Supongamos que vamos a necesitar dos palabras de memoria para ejecutar una función. Podemos simplemente mover el puntero de pila hacia abajo dos palabras para reservar el espacio. Esto se hace así:

```assembly
subl $8, %esp
```

Esto resta 8 a `%esp` (recuerda que una palabra tiene cuatro bytes de largo). [^4] De esta manera, podemos usar la pila para el almacenamiento de variables sin preocuparnos por sobrescribirlas con los `push` que podamos hacer para las llamadas a funciones. Además, dado que se asigna en el marco de pila para esta llamada a función, la variable solo estará viva durante esta función. Cuando retornemos, el marco de pila desaparecerá, y también estas variables. Por eso se llaman **locales**: solo existen mientras se llama a esta función.

Ahora tenemos dos palabras para almacenamiento local. Nuestra pila se ve ahora así:

| Contenido de la Pila | Desplazamiento (Offset) |
|:---:|:---:|
| Parámetro #N | <--- N*4+4(%ebp) |
| ... | |
| Parámetro 2 | <--- 12(%ebp) |
| Parámetro 1 | <--- 8(%ebp) |
| Dirección de Retorno | <--- 4(%ebp) |
| Viejo %ebp | <--- (%ebp) |
| Variable Local 1 | <--- -4(%ebp) |
| Variable Local 2 | <--- -8(%ebp) y (%esp) |

Así que ahora podemos acceder a todos los datos que necesitamos para esta función utilizando el direccionamiento por puntero base usando diferentes desplazamientos desde `%ebp`. `%ebp` fue creado específicamente para este propósito, por lo que se llama puntero base. Puedes usar otros registros en el modo de direccionamiento por puntero base, pero la arquitectura x86 hace que el uso del registro `%ebp` sea mucho más rápido.

A las variables globales y estáticas se accede igual que hemos estado accediendo a la memoria en capítulos anteriores. La única diferencia entre las variables globales y las estáticas es que las variables estáticas solo son usadas por una función, mientras que las globales son usadas por muchas funciones. El lenguaje ensamblador las trata exactamente igual, aunque la mayoría de los otros lenguajes las distinguen.

Cuando una función termina de ejecutarse, hace tres cosas:

1. Almacena su valor de retorno en `%eax`.
2. Restablece la pila a como estaba cuando fue llamada (se deshace del marco de pila actual y vuelve a poner en vigor el marco de pila del código que la llamó).
3. Devuelve el control a donde fue llamada. Esto se hace usando la instrucción `ret`, que saca cualquier valor que esté en la parte superior de la pila y establece el puntero de instrucción, `%eip`, a ese valor.

Por lo tanto, antes de que una función devuelva el control al código que la llamó, debe restaurar el marco de pila anterior. Ten en cuenta también que sin hacer esto, `ret` no funcionaría, porque en nuestro marco de pila actual, la dirección de retorno no está en la parte superior de la pila. Por lo tanto, antes de retornar, tenemos que restablecer el puntero de pila `%esp` y el puntero base `%ebp` a lo que eran cuando comenzó la función.

Por lo tanto, para retornar de la función tienes que hacer lo siguiente:

```assembly
movl %ebp, %esp
popl %ebp
ret
```

En este punto, debes considerar que todas las variables locales han sido eliminadas. La razón es que después de mover el puntero de pila hacia atrás, los futuros `push` en la pila probablemente sobrescribirán todo lo que pusiste allí. Por lo tanto, nunca debes guardar la dirección de una variable local más allá de la vida de la función en la que fue creada, o de lo contrario será sobrescrita después de que termine la vida de su marco de pila.

El control ha sido devuelto al código que realizó la llamada, que ahora puede examinar `%eax` para obtener el valor de retorno. El código que realizó la llamada también necesita sacar todos los parámetros que metió en la pila para que el puntero de pila vuelva a estar donde estaba (también puedes simplemente sumar `4 * número de parámetros` a `%esp` usando la instrucción `addl`, si ya no necesitas los valores de los parámetros). [^5]

### Destrucción de Registros

Cuando llamas a una función, debes asumir que todo lo que hay actualmente en tus registros será borrado. El único registro que se garantiza que quedará con el valor con el que comenzó es `%ebp`. Se garantiza que `%eax` será sobrescrito, y los demás probablemente también. Si hay registros que deseas guardar antes de llamar a una función, debes guardarlos metiéndolos en la pila antes de meter los parámetros de la función. Luego puedes volver a sacarlos en orden inverso después de sacar los parámetros. Incluso si sabes que una función no sobrescribe un registro, deberías guardarlo, porque las futuras versiones de esa función podrían hacerlo.

Las convenciones de llamada de otros lenguajes pueden ser diferentes. Por ejemplo, otras convenciones de llamada pueden imponer la carga a la función de guardar cualquier registro que utilice. Asegúrate de comprobar que las convenciones de llamada de tus lenguajes son compatibles antes de intentar mezclar lenguajes. O en el caso del lenguaje ensamblador, asegúrate de saber cómo llamar a las funciones del otro lenguaje.

**Especificación Extendida:** Los detalles de la convención de llamada del lenguaje C (también conocida como ABI, o Application Binary Interface) están disponibles en línea. Hemos simplificado demasiado y omitido varias piezas importantes para hacer esto más sencillo para los nuevos programadores. Para obtener todos los detalles, debes consultar los documentos disponibles en http://www.linuxbase.org/spec/refspecs/. Específicamente, debes buscar el *System V Application Binary Interface - Intel386 Architecture Processor Supplement*.

## Un Ejemplo de Función

Veamos cómo funciona una llamada a función en un programa real. La función que vamos a escribir es la función potencia (power). Le daremos a la función potencia dos parámetros: el número y la potencia a la que queremos elevarlo. Por ejemplo, si le diéramos los parámetros 2 y 3, elevaría 2 a la potencia de 3, o 2\*2*2, dando 8. Para simplificar este programa, solo permitiremos números 1 y mayores.

A continuación se muestra el código del programa completo. Como de costumbre, sigue una explicación. Nombra el archivo `power.s`.

```assembly
#PROPÓSITO: Programa para ilustrar cómo funcionan las funciones
#           Este programa calculará el valor de
#           2^3 + 5^2
#

#Todo en el programa principal se almacena en registros,
#así que la sección de datos no tiene nada.
.section .data

.section .text

.globl _start
_start:
 pushl $3                  #meter el segundo argumento
 pushl $2                  #meter el primer argumento
 call  power               #llamar a la función
 addl  $8, %esp            #mover el puntero de pila de vuelta
 pushl %eax                #guardar la primera respuesta antes
                           #de llamar a la siguiente función

 pushl $2                  #meter el segundo argumento
 pushl $5                  #meter el primer argumento
 call  power               #llamar a la función
 addl  $8, %esp            #mover el puntero de pila de vuelta

 popl  %ebx                #La segunda respuesta ya está
                           #en %eax. Guardamos la
                           #primera respuesta en la pila,
                           #así que ahora simplemente podemos sacarla
                           #a %ebx

 addl  %eax, %ebx          #sumarlos
                           #el resultado está en %ebx

 movl  $1, %eax            #salir (%ebx es retornado)
 int   $0x80

#PROPÓSITO: Esta función se utiliza para calcular
#           el valor de un número elevado a
#           una potencia.
#
#ENTRADA:   Primer argumento - el número base
#           Segundo argumento - la potencia a la que
#           elevarlo
#
#SALIDA:    Dará el resultado como un valor de retorno
#
#NOTAS:     La potencia debe ser 1 o mayor
#
#VARIABLES:
#           %ebx - contiene el número base
#           %ecx - contiene la potencia
#
#           -4(%ebp) - contiene el resultado actual
#
#           %eax se utiliza para almacenamiento temporal
#
.type power, @function
power:
 pushl %ebp                #guardar el viejo puntero base
 movl  %esp, %ebp          #hacer que el puntero de pila sea el puntero base
 subl  $4, %esp            #obtener espacio para nuestro almacenamiento local

 movl  8(%ebp), %ebx       #poner el primer argumento en %ebx
 movl  12(%ebp), %ecx      #poner el segundo argumento en %ecx

 movl  %ebx, -4(%ebp)      #almacenar el resultado actual

power_loop_start:
 cmpl  $1, %ecx            #si la potencia es 1, hemos terminado
 je    end_power
 movl  -4(%ebp), %eax      #mover el resultado actual a %eax
 imull %ebx, %eax          #multiplicar el resultado actual por
                           #el número base
 movl  %eax, -4(%ebp)      #almacenar el resultado actual

 decl  %ecx                #disminuir la potencia
 jmp   power_loop_start    #ejecutar para la siguiente potencia

end_power:
 movl  -4(%ebp), %eax      #el valor de retorno va en %eax
 movl  %ebp, %esp          #restaurar el puntero de pila
 popl  %ebp                #restaurar el puntero base
 ret
```

Escribe el programa, ensámblalo y ejecútalo. Prueba a llamar a `power` con diferentes valores, pero recuerda que el resultado tiene que ser menor de 256 cuando se devuelve al sistema operativo. También intenta restar los resultados de los dos cálculos. Intenta añadir una tercera llamada a la función `power` y suma su resultado de nuevo.

El código del programa principal es bastante sencillo. Metes los argumentos en la pila, llamas a la función y luego mueves el puntero de pila hacia atrás. El resultado se almacena en `%eax`. Ten en cuenta que entre las dos llamadas a `power`, guardamos el primer valor en la pila. Esto se debe a que el único registro que se garantiza guardar es `%ebp`. Por lo tanto, metemos el valor en la pila y lo volvemos a sacar una vez completada la segunda llamada a la función.

Veamos cómo se escribe la función en sí. Observa que antes de la función hay documentación sobre lo que hace la función, cuáles son sus argumentos y qué da como valor de retorno. Esto es útil para los programadores que usan esta función. Esta es la interfaz de la función. Esto permite al programador saber qué valores se necesitan en la pila y qué habrá en `%eax` al final.

Luego tenemos la siguiente línea:

```assembly
.type power, @function
```

Esto le dice al enlazador que el símbolo `power` debe ser tratado como una función. Como este programa está solo en un archivo, funcionaría igual si se omitiera. Sin embargo, es una buena práctica.

Después de eso, definimos el valor de la etiqueta `power`:

```assembly
power:
```

Como se mencionó anteriormente, esto define el símbolo `power` como la dirección donde comienzan las instrucciones que siguen a la etiqueta. Así es como funciona `call power`. Transfiere el control a este punto del programa. La diferencia entre `call` y `jmp` es que `call` también mete en la pila la dirección de retorno para que la función pueda retornar, mientras que `jmp` no lo hace.

A continuación, tenemos nuestras instrucciones para configurar nuestra función:

```assembly
pushl %ebp
movl %esp, %ebp
subl $4, %esp
```

En este punto, nuestra pila se ve así:

| Contenido de la Pila | Desplazamiento (Offset) |
|:---:|:---:|
| Número Base | <--- 12(%ebp) |
| Potencia | <--- 8(%ebp) |
| Dirección de Retorno | <--- 4(%ebp) |
| Viejo %ebp | <--- (%ebp) |
| Resultado Actual | <--- -4(%ebp) y (%esp) |

Aunque podríamos usar un registro para almacenamiento temporal, este programa usa una variable local para mostrar cómo configurarla. A menudo no hay suficientes registros para almacenarlo todo, así que tienes que descargarlos en variables locales. Otras veces, tu función necesitará llamar a otra función y enviarle un puntero a algunos de tus datos. No puedes tener un puntero a un registro, por lo que tienes que almacenarlo en una variable local para poder enviar un puntero a ella.

Básicamente, lo que hace el programa es empezar con el número base y almacenarlo tanto como el multiplicador (almacenado en `%ebx`) como el valor actual (almacenado en `-4(%ebp)`). También tiene la potencia almacenada en `%ecx`. Luego multiplica continuamente el valor actual por el multiplicador, disminuye la potencia y sale del bucle si la potencia (en `%ecx`) llega a 1.

Por ahora, deberías ser capaz de repasar el programa sin ayuda. Lo único que deberías saber es que `imull` realiza la multiplicación de enteros y almacena el resultado en el segundo operando, y `decl` disminuye el registro dado en 1. Para más información sobre estas y otras instrucciones, consulta el Apéndice B.

Un buen proyecto para intentar ahora es ampliar el programa para que devuelva el valor de un número si la potencia es 0 (pista: cualquier número elevado a la potencia cero es 1). Sigue intentándolo. Si no funciona al principio, intenta repasar tu programa a mano con un papel sucio, haciendo un seguimiento de a dónde apuntan `%ebp` y `%esp`, qué hay en la pila y cuáles son los valores en cada registro.

## Funciones Recursivas

El siguiente programa pondrá a prueba vuestros cerebros aún más. El programa calculará el factorial de un número. Un **factorial** es el producto de un número y todos los números entre él y el uno. Por ejemplo, el factorial de 7 es 7\*6\*5\*4\*3\*2\*1, y el factorial de 4 es 4\*3\*2\*1. Ahora bien, una cosa que puedes notar es que el factorial de un número es lo mismo que el producto de un número y el factorial inmediatamente inferior. Por ejemplo, el factorial de 4 es 4 veces el factorial de 3. El factorial de 3 es 3 veces el factorial de 2. 2 es 2 veces el factorial de 1. El factorial de 1 es 1. Este tipo de definición se llama **definición recursiva**. Eso significa que la definición de la función factorial se incluye a sí misma. Sin embargo, dado que todas las funciones deben terminar, una definición recursiva debe incluir un **caso base**. El caso base es el punto donde la recursión se detendrá. Sin un caso base, la función seguiría llamándose a sí misma eternamente hasta que finalmente se agotara el espacio de la pila. En el caso del factorial, el caso base es el número 1. Cuando llegamos al número 1, no ejecutamos el factorial de nuevo, simplemente decimos que el factorial de 1 es 1. Así que, veamos cómo queremos que sea el código de nuestra función factorial:

1. Examinar el número
2. ¿Es el número 1?
3. Si es así, la respuesta es uno
4. De lo contrario, la respuesta es el número multiplicado por el factorial del número menos uno

Esto sería problemático si no tuviéramos variables locales. En otros programas, el almacenamiento de valores en variables globales funcionaba bien. Sin embargo, las variables globales solo proporcionan una copia de cada variable. ¡En este programa, tendremos múltiples copias de la función ejecutándose al mismo tiempo, y todas ellas necesitarán sus propias copias de los datos! [^6] Como las variables locales existen en el marco de pila, y cada llamada a la función obtiene su propio marco de pila, no tenemos problemas.

Veamos el código para ver cómo funciona esto:

```assembly
#PROPÓSITO - Dado un número, este programa calcula el
#           factorial. Por ejemplo, el factorial de
#           3 es 3 * 2 * 1, o 6. El factorial de
#           4 es 4 * 3 * 2 * 1, o 24, y así sucesivamente.
#

#Este programa muestra cómo llamar a una función recursivamente.

.section .data
#Este programa no tiene datos globales

.section .text
.globl _start
.globl factorial           #esto no es necesario a menos que queramos compartir
                           #esta función con otros programas
_start:
 pushl $4                  #El factorial toma un argumento: el
                           #número del que queremos el factorial. Así que,
                           #se mete en la pila
 call  factorial           #ejecutar la función factorial
 addl  $4, %esp            #Limpia el parámetro que se metió en
                           #la pila
 movl  %eax, %ebx          #factorial devuelve la respuesta en %eax, pero
                           #queremos que esté en %ebx para enviarla como nuestro estado
                           #de salida
 movl  $1, %eax            #llamar a la función de salida del kernel
 int   $0x80

#Esta es la definición real de la función
.type factorial,@function
factorial:
 pushl %ebp                #cosas estándar de funciones: tenemos que
                           #restaurar %ebp a su estado anterior antes
                           #de retornar, así que tenemos que meterlo en la pila
 movl  %esp, %ebp          #Esto es porque no queremos modificar
                           #el puntero de pila, así que usamos %ebp.
 movl  8(%ebp), %eax       #Esto mueve el primer argumento a %eax
                           #4(%ebp) contiene la dirección de retorno, y
                           #8(%ebp) contiene el primer parámetro
 cmpl  $1, %eax            #Si el número es 1, ese es nuestro caso
                           #base, y simplemente retornamos (el 1 ya está
                           #en %eax como valor de retorno)
 je    end_factorial
 decl  %eax                #de lo contrario, disminuir el valor
 pushl %eax                #meterlo en la pila para nuestra llamada a factorial
 call  factorial           #llamar a factorial
 movl  8(%ebp), %ebx       #%eax tiene el valor de retorno, así que
                           #recargamos nuestro parámetro en %ebx
 imull %ebx, %eax          #multiplicar eso por el resultado de la
                           #última llamada a factorial (en %eax)
                           #la respuesta se almacena en %eax, lo cual
                           #es bueno ya que ahí es donde van los valores
                           #de retorno.
end_factorial:
 movl  %ebp, %esp          #cosas estándar de retorno de funciones - tenemos
 popl  %ebp                #que restaurar %ebp y %esp a donde
                           #estaban antes de que comenzara la función
 ret                       #retornar a la función (esto saca el
                           #valor de retorno de la pila también)
```

Ensámblalo, enlázalo y ejecútalo con estos comandos:

```bash
as factorial.s -o factorial.o
ld factorial.o -o factorial
./factorial
echo $?
```

Esto debería darte el valor 24. 24 es el factorial de 4, puedes comprobarlo tú mismo con una calculadora: 4 \* 3 \* 2 \* 1 = 24.

Supongo que no habrás entendido todo el código. Vamos a recorrerlo línea por línea para ver qué está pasando.

```assembly
_start:
 pushl $4
 call factorial
```

Bien, este programa tiene la intención de calcular el factorial del número 4. Al programar funciones, se supone que debes poner los parámetros de la función en la parte superior de la pila justo antes de llamarla. Recuerda, los parámetros de una función son los datos con los que quieres que trabaje la función. En este caso, la función factorial toma 1 parámetro: el número del que quieres el factorial.

La instrucción `pushl` coloca el valor dado en la parte superior de la pila. La instrucción `call` realiza entonces la llamada a la función. A continuación tenemos estas líneas:

```assembly
addl $4, %esp
movl %eax, %ebx
movl $1, %eax
int $0x80
```

Esto ocurre después de que `factorial` haya terminado y calculado el factorial de 4 para nosotros. Ahora tenemos que limpiar la pila. La instrucción `addl` mueve el puntero de la pila de vuelta a donde estaba antes de meter el `$4` en la pila. Siempre debes limpiar los parámetros de tu pila después de que retorne una llamada a función.

La siguiente instrucción mueve `%eax` a `%ebx`. ¿Qué hay en `%eax`? Es el valor de retorno de `factorial`. En nuestro caso, es el valor de la función factorial. Con 4 como nuestro parámetro, 24 debería ser nuestro valor de retorno. Recuerda, los valores de retorno siempre se almacenan en `%eax`. Queremos devolver este valor como el código de estado al sistema operativo. Sin embargo, Linux requiere que el estado de salida del programa se almacene en `%ebx`, no en `%eax`, así que tenemos que moverlo. Luego hacemos la llamada al sistema estándar de salida (exit).

Lo bueno de las llamadas a funciones es que:

- Otros programadores no tienen que saber nada sobre ellas, excepto sus argumentos para usarlas.
- Proporcionan bloques de construcción estandarizados a partir de los cuales se puede formar un programa.
- Pueden ser llamadas múltiples veces y desde múltiples ubicaciones y siempre saben cómo volver a donde estaban ya que `call` mete la dirección de retorno en la pila.

Estas son las principales ventajas de las funciones. Los programas más grandes también utilizan funciones para descomponer piezas complejas de código en otras más pequeñas y sencillas. De hecho, casi toda la programación consiste en escribir y llamar a funciones.

Veamos ahora cómo se implementa la función factorial en sí. Antes de que comience la función, tenemos esta directiva:

```assembly
.type factorial,@function
factorial:
```

La directiva `.type` le dice al enlazador que `factorial` es una función. Esto no es realmente necesario a menos que estuviéramos usando `factorial` en otros programas. Lo hemos incluido por completitud. La línea que dice `factorial:` le da al símbolo `factorial` la ubicación de almacenamiento de la siguiente instrucción. Así es como `call` sabía a dónde ir cuando dijimos `call factorial`.

Las primeras instrucciones reales de la función son:

```assembly
pushl %ebp
movl %esp, %ebp
```

Como se mostró en el programa anterior, esto crea el marco de pila para esta función. Estas dos líneas deberían ser la forma en que comiences cada función. La siguiente instrucción es esta:

```assembly
movl 8(%ebp), %eax
```

Esto usa el direccionamiento por puntero base para mover el primer parámetro de la función a `%eax`. Recuerda que `(%ebp)` tiene el antiguo `%ebp`, `4(%ebp)` tiene la dirección de retorno y `8(%ebp)` es la ubicación del primer parámetro de la función. Si haces memoria, este será el valor 4 en la primera llamada, ya que eso fue lo que metimos en la pila antes de llamar a la función (con `pushl $4`).

A continuación, comprobamos si hemos llegado a nuestro caso base (un parámetro de 1). Si es así, saltamos a la instrucción en la etiqueta `end_factorial`, donde será retornado. Ya está en `%eax`, que mencionamos antes que es donde se ponen los valores de retorno. Eso se logra con estas líneas:

```assembly
cmpl $1, %eax
je end_factorial
```

Si no es nuestro caso base, ¿qué dijimos que haríamos? Llamaríamos a la función factorial de nuevo con nuestro parámetro menos uno. Así que, primero disminuimos `%eax` en uno:

```assembly
decl %eax
```

`decl` significa decremento (decrement). Resta 1 del registro o ubicación de memoria dado (`%eax` en nuestro caso). `incl` es el inverso: suma 1. Después de decrementar `%eax` lo metemos en la pila ya que va a ser el parámetro de la siguiente llamada a la función. ¡Y luego llamamos a `factorial` de nuevo!

```assembly
pushl %eax
call factorial
```

Bien, ahora hemos llamado a `factorial`. Una cosa que hay que recordar es que después de una llamada a una función, nunca podemos saber qué hay en los registros (excepto en `%esp` y `%ebp`). Así que aunque tuviéramos el valor con el que fuimos llamados en `%eax`, ya no está allí. Por lo tanto, necesitamos extraerlo de la pila del mismo lugar donde lo obtuvimos la primera vez (en `8(%ebp)`). Así que hacemos esto:

```assembly
movl 8(%ebp), %ebx
```

Ahora queremos multiplicar ese número con el resultado de la función factorial. Si recuerdas nuestra discusión anterior, el resultado de las funciones se deja en `%eax`. Por lo tanto, necesitamos multiplicar `%ebx` con `%eax`. Esto se hace con esta instrucción:

```assembly
imull %ebx, %eax
```

Esto también almacena el resultado en `%eax`, ¡que es exactamente donde queremos que esté el valor de retorno de la función! Como el valor de retorno está en su sitio, solo tenemos que salir de la función. Si recuerdas, al inicio de la función metimos `%ebp` en la pila y movimos `%esp` a `%ebp` para crear el marco de pila actual. Ahora invertimos la operación para destruir el marco de pila actual y reactivar el anterior:

```assembly
end_factorial:
 movl %ebp, %esp
 popl %ebp
```

Ahora estamos listos para retornar, así que emitimos el siguiente comando:

```assembly
ret
```

Esto saca el valor superior de la pila y luego salta a él. Si recuerdas nuestra discusión sobre `call`, dijimos que `call` primero metía la dirección de la siguiente instrucción en la pila antes de saltar al inicio de la función. Así que aquí la sacamos de nuevo para poder volver allí. ¡La función ha terminado y tenemos nuestra respuesta!

Al igual que en nuestro programa anterior, deberías volver a mirar el programa y asegurarte de saber qué hace cada cosa. Vuelve a leer esta sección y las secciones anteriores para obtener la explicación de cualquier cosa que no entiendas. Luego, toma un trozo de papel y recorre el programa paso a paso, haciendo un seguimiento de cuáles son los valores de los registros en cada paso y qué valores hay en la pila. Hacer esto debería profundizar tu comprensión de lo que está sucediendo.

## Revisión

### Conoce los Conceptos

- ¿Qué son las primitivas?
- ¿Qué son las convenciones de llamada?
- ¿Qué es la pila (stack)?
- ¿Cómo afectan `pushl` y `popl` a la pila? ¿A qué registro de propósito especial afectan?
- ¿Qué son las variables locales y para qué se utilizan?
- ¿Por qué son tan necesarias las variables locales en las funciones recursivas?
- ¿Para qué se utilizan `%ebp` y `%esp`?
- ¿Qué es un marco de pila (stack frame)?

### Usa los Conceptos

- Escribe una función llamada `square` que reciba un argumento y devuelva el cuadrado de ese argumento.
- Escribe un programa para probar tu función `square`.
- Convierte el programa `maximum` dado en la sección llamada *Encontrar un Valor Máximo* en el Capítulo 3 para que sea una función que tome un puntero a varios valores y devuelva su máximo. Escribe un programa que llame a `maximum` con 3 listas diferentes y devuelva el resultado de la última como código de estado de salida del programa.
- Explica los problemas que surgirían sin una convención de llamada estándar.

### Yendo más allá

- ¿Crees que es mejor para un sistema tener un conjunto grande de primitivas o uno pequeño, asumiendo que el conjunto más grande puede escribirse en términos del más pequeño?
- La función factorial se puede escribir de forma no recursiva. Hazlo.
- Encuentra una aplicación en la computadora que uses regularmente. Intenta localizar una característica específica y practica la división de esa característica en funciones. Define las interfaces de las funciones entre esa característica y el resto del programa.
- Inventa tu propia convención de llamada. Reescribe los programas de este capítulo usándola. Un ejemplo de una convención de llamada diferente sería pasar parámetros en registros en lugar de en la pila, pasarlos en un orden diferente, devolver valores en otros registros o ubicaciones de memoria. Elijas lo que elijas, sé coherente y aplícalo a todo el programa.
- ¿Puedes construir una convención de llamada sin usar la pila? ¿Qué limitaciones podría tener?
- ¿Qué casos de prueba deberíamos usar en nuestro programa de ejemplo para comprobar si funciona correctamente?

---

[^1]: Los parámetros de las funciones también pueden utilizarse para contener punteros a datos que la función desea devolver al programa.
[^2]: Esto se considera generalmente una mala práctica. Imagina que un programa se escribe de esta manera, y en la siguiente versión deciden permitir que una única instancia del programa edite múltiples archivos. Cada función tendría entonces que ser modificada para que el archivo que se está manipulando se pasara como un parámetro. Si simplemente lo hubieras pasado como un parámetro desde el principio, la mayoría de tus funciones habrían sobrevivido a la actualización sin cambios.
[^3]: Una convención es una forma de hacer las cosas que está estandarizada, pero no obligatoriamente. Por ejemplo, es una convención que la gente se dé la mano cuando se conoce. Si me niego a darte la mano, puedes pensar que no me agradas. Seguir las convenciones es importante porque facilita que los demás entiendan lo que estás haciendo, y facilita que los programas escritos por múltiples autores independientes trabajen juntos.
[^4]: Solo un recordatorio: el signo de dólar delante del ocho indica el modo de direccionamiento inmediato, lo que significa que cargamos el número 8 en `%esp` en lugar del valor en la dirección 8.
[^5]: Esto no siempre es estrictamente necesario a menos que estés guardando registros en la pila antes de una llamada a una función. El puntero base mantiene el marco de la pila en un estado razonablemente coherente. Sin embargo, sigue siendo una buena idea, y es absolutamente necesario si estás guardando temporalmente registros en la pila.
[^6]: Al decir "ejecutándose al mismo tiempo" me refiero al hecho de que una no habrá terminado antes de que se active una nueva. No estoy dando a entender que sus instrucciones se estén ejecutando al mismo tiempo.
