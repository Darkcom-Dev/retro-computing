# Capítulo 12. Optimización

La optimización es el proceso de hacer que tu aplicación funcione de manera más efectiva. Puedes optimizar para muchas cosas - velocidad, uso de espacio en memoria, uso de espacio en disco, etc. Sin embargo, este capítulo se centra en la optimización de velocidad.

## Cuándo Optimizar

Es mejor no optimizar en absoluto que optimizar demasiado pronto. Cuando optimizas, tu código generalmente se vuelve menos claro, porque se vuelve más complejo. Los lectores de tu código tendrán más problemas para descubrir por qué hiciste lo que hiciste, lo que aumentará el costo de mantenimiento de tu proyecto. Incluso cuando sabes cómo y por qué tu programa funciona como lo hace, el código optimizado es más difícil de depurar y extender. Ralentiza el proceso de desarrollo considerablemente, tanto por el tiempo que lleva optimizar el código, como por el tiempo que lleva modificar tu código optimizado.

Agravando este problema está que ni siquiera sabes de antemano dónde estarán los problemas de velocidad en tu programa. Incluso los programadores experimentados tienen problemas para predecir qué partes del programa serán los cuellos de botella que necesitan optimización, por lo que probablemente terminarás perdiendo tu tiempo optimizando las partes equivocadas. La sección llamada *Dónde Optimizar* discutirá cómo encontrar las partes de tu programa que necesitan optimización.

Mientras desarrollas tu programa, debes tener las siguientes prioridades:

*   Todo está documentado
*   Todo funciona como está documentado
*   El código está escrito en una forma modular y fácilmente modificable

La documentación es esencial, especialmente cuando se trabaja en grupos. El funcionamiento correcto del programa es esencial. Notarás que la velocidad de la aplicación no estaba en esa lista. La optimización no es necesaria durante el desarrollo temprano por las siguientes razones:

*   Los problemas menores de velocidad generalmente pueden resolverse a través del hardware, que a menudo es mucho más barato que el tiempo de un programador.
*   Tu aplicación cambiará dramáticamente a medida que la revises, por lo tanto desperdiciando la mayoría de tus esfuerzos por optimizarla.<sup>1</sup>
*   Los problemas de velocidad generalmente están localizados en unos pocos lugares de tu código - encontrar estos es difícil antes de tener la mayor parte del programa terminado.

Por lo tanto, el momento de optimizar es hacia el final del desarrollo, cuando has determinado que tu código correcto realmente tiene problemas de rendimiento.

En un proyecto de comercio electrónico basado en web en el que estuve involucrado, me centré enteramente en la corrección. Esto fue muy decepcionante para mis colegas, que estaban preocupados por el hecho de que cada página tardaba doce segundos en procesarse antes de comenzar a cargarse (la mayoría de las páginas web se procesan en menos de un segundo). Sin embargo, estaba decidido a hacerlo bien primero, y poner la optimización como la última prioridad. Cuando el código finalmente fue correcto después de 3 meses de trabajo, solo tomó tres días encontrar y eliminar los cuellos de botella, reduciendo el tiempo promedio de procesamiento a menos de un cuarto de segundo. Al centrarme en el orden correcto, pude terminar un proyecto que era tanto correcto como eficiente.

---
<sup>1</sup> Muchos proyectos nuevos a menudo tienen una primera base de código que es completamente reescrita a medida que los desarrolladores aprenden más sobre el problema que están tratando de resolver. Cualquier optimización realizada en la primera base de código se desperdicia completamente.

## Dónde Optimizar

Una vez que has determinado que tienes un problema de rendimiento, debes determinar dónde en el código ocurren los problemas. Puedes hacer esto ejecutando un **analizador de rendimiento (profiler)**. Un profiler es un programa que te permite ejecutar tu programa, y te dirá cuánto tiempo se gasta en cada función, y cuántas veces se ejecutan. `gprof` es la herramienta de perfilado estándar de GNU/Linux, pero una discusión sobre el uso de profilers está fuera del alcance de este texto. Después de ejecutar un profiler, puedes determinar qué funciones se llaman más o tienen más tiempo invertido en ellas. Estas son en las que debes centrar tus esfuerzos de optimización.

Si un programa solo gasta el 1% de su tiempo en una función dada, entonces no importa cuánto la aceleres, solo lograrás un máximo de un 1% de mejora general de velocidad. Sin embargo, si un programa gasta el 20% de su tiempo en una función dada, entonces incluso mejoras menores en la velocidad de esa función serán notables. Por lo tanto, el perfilado te da la información que necesitas para tomar buenas decisiones sobre dónde gastar tu tiempo de programación.

Para optimizar funciones, necesitas entender de qué maneras están siendo llamadas y usadas. Cuanto más sepas sobre cómo y cuándo se llama a una función, mejor posición tendrás para optimizarla adecuadamente.

Hay dos categorías principales de optimización - optimizaciones locales y optimizaciones globales. Las **[[#optimizaciones-locales|optimizaciones locales]]** consisten en optimizaciones que son específicas del hardware - como la forma más rápida de realizar un cálculo dado - o específicas del programa - como hacer que una pieza específica de código funcione mejor para el caso más frecuente. La **optimización global** consiste en optimizaciones que son estructurales. Por ejemplo, si estuvieras tratando de encontrar la mejor manera para que tres personas en diferentes ciudades se reúnan en St. Louis, una optimización local sería encontrar un mejor camino para llegar allí, mientras que una optimización global sería decidir hacer una teleconferencia en lugar de reunirse en persona. La optimización global a menudo implica reestructurar el código para evitar problemas de rendimiento, en lugar de tratar de encontrar la mejor manera a través de ellos.

## Optimizaciones Locales

Las siguientes son algunos métodos bien conocidos de optimizar piezas de código. Cuando se usan lenguajes de alto nivel, algunos de estos pueden ser hechos automáticamente por el optimizador de tu compilador.

### Precomputación de Cálculos

A veces una función tiene un número limitado de entradas y salidas posibles. De hecho, puede ser tan pocas que puedas precomputar todas las respuestas posibles de antemano, y simplemente buscar la respuesta cuando se llame a la función. Esto ocupa algo de espacio ya que tienes que almacenar todas las respuestas, pero para conjuntos pequeños de datos funciona muy bien, especialmente si el cálculo normalmente toma mucho tiempo.

### Recordar Resultados de Cálculos

Esto es similar al método anterior, pero en lugar de computar los resultados de antemano, se almacena el resultado de cada cálculo solicitado. De esta manera, cuando la función comienza, si el resultado se ha computado antes, simplemente devolverá la respuesta anterior, de lo contrario hará el cálculo completo y almacenará el resultado para una búsqueda posterior. Esto tiene la ventaja de requerir menos espacio de almacenamiento porque no estás precomputando todos los resultados. Esto a veces se denomina **caching** o **memoización**.

### Localidad de Referencia

La localidad de referencia es un término para dónde en la memoria se encuentran los elementos de datos a los que estás accediendo. Con la [[09-capitulo-9|memoria virtual]], puedes acceder a páginas de memoria que están almacenadas en el disco. En tal caso, el [[02-capitulo-2#system-calls|sistema operativo]] tiene que cargar esa página de memoria desde el disco, y descargar otras al disco. Digamos, por ejemplo, que el sistema operativo te permitirá tener 20k de memoria en memoria física y fuerza al resto a estar en el disco, y tu aplicación usa 60k de memoria. Digamos que tu programa tiene que hacer 5 operaciones en cada pieza de datos. Si hace una operación en cada pieza de datos, y luego pasa a hacer la siguiente operación en cada pieza de datos, eventualmente cada página de datos será cargada y descargada del disco 5 veces. En cambio, si hicieras las 5 operaciones en un elemento de datos dado, solo tienes que cargar cada página del disco una vez. Cuando agrupas tantas operaciones en datos que están físicamente cerca entre sí en la memoria, entonces estás aprovechando la localidad de referencia.

Además, los procesadores usualmente almacenan algunos datos en el chip en un [[#recordar-resultados-de-cálculos|caché]]. Si mantienes todas tus operaciones dentro de un área pequeña de memoria física, tu programa puede incluso evitar la memoria principal y solo usar la memoria caché ultrarrápida del chip. Esto se hace por ti - todo lo que tienes que hacer es tratar de operar en pequeñas secciones de memoria a la vez, en lugar de saltar por todas partes.

### Uso de Registros

Los [[15-apendice-B|registros]] son las ubicaciones de memoria más rápidas en la computadora. Cuando accedes a la memoria, el procesador tiene que esperar mientras se carga desde el bus de memoria. Sin embargo, los registros están ubicados en el procesador mismo, por lo que el acceso es extremadamente rápido. Por lo tanto, hacer un uso inteligente de los registros es extremadamente importante. Si tienes pocos elementos de datos con los que estás trabajando, trata de almacenarlos todos en registros. En lenguajes de alto nivel, no siempre tienes esta opción - el compilador decide qué va en los registros y qué no.

### Funciones Inline

Las funciones son excelentes desde el punto de vista de la gestión del programa - facilitan la división de tu programa en partes independientes, comprensibles y reutilizables. Sin embargo, las llamadas a funciones implican la sobrecarga de empujar argumentos a la pila y hacer los saltos (recuerda la localidad de referencia - tu código puede estar intercambiado en el disco en lugar de en la memoria). Para lenguajes de alto nivel, a menudo es imposible para los compiladores hacer optimizaciones a través de los límites de las llamadas a funciones. Sin embargo, algunos lenguajes soportan funciones inline o macros de funciones. Estas funciones se ven, huelen, saben y actúan como funciones reales, excepto que el compilador tiene la opción de simplemente insertar el código exactamente donde fue llamado. Esto hace que el programa sea más rápido, pero también aumenta el tamaño del código. También hay muchas funciones, como las funciones recursivas, que no pueden ser inlineadas porque se llaman a sí mismas directa o indirectamente.

### Instrucciones Optimizadas

A menudo hay múltiples instrucciones en lenguaje ensamblador que logran el mismo propósito. Un programador experto en ensamblador sabe qué instrucciones son las más rápidas. Sin embargo, esto puede cambiar de un procesador a otro. Para más información sobre este tema, necesitas ver el manual del usuario que se proporciona para el chip específico que estás usando. Como ejemplo, veamos el proceso de cargar el número 0 en un registro. En la mayoría de los procesadores, hacer `movl $0, %eax` no es la forma más rápida. La forma más rápida es hacer un exclusive-or del registro consigo mismo, `xorl %eax, %eax`. Esto es porque solo tiene que acceder al registro, y no tiene que transferir ningún dato. Para los usuarios de lenguajes de alto nivel, el compilador maneja este tipo de optimizaciones por ti. Para los programadores de lenguaje ensamblador, necesitas conocer bien tu procesador.

### Modos de Direccionamiento

Diferentes [[03-capitulo-3|modos de direccionamiento]] funcionan a diferentes velocidades. Los más rápidos son los modos de direccionamiento inmediato y de registro. El directo es el siguiente más rápido, el indirecto es el siguiente, y el puntero base y el indirecto indexado son los más lentos. Trata de usar los modos de direccionamiento más rápidos, cuando sea posible. Una consecuencia interesante de esto es que cuando tienes una pieza de memoria estructurada a la que estás accediendo usando direccionamiento de puntero base, el primer elemento puede ser accedido más rápidamente. Ya que su desplazamiento es 0, puedes acceder a él usando direccionamiento indirecto en lugar de direccionamiento de puntero base, lo que lo hace más rápido.

### Alineación de Datos

Algunos procesadores pueden acceder a datos en límites de memoria alineados con palabras (es decir, direcciones divisibles por el tamaño de palabra) más rápido que datos no alineados. Por lo tanto, al configurar estructuras en la memoria, es mejor mantenerlas alineadas con palabras. Algunos procesadores que no son x86, de hecho, no pueden acceder a datos no alineados en algunos modos.

Estos son solo una muestra de ejemplos de los tipos de optimizaciones locales posibles. Sin embargo, recuerda que la mantenibilidad y legibilidad del código es mucho más importante excepto en circunstancias extremas.

## Optimización Global

La optimización global tiene dos objetivos. El primero es poner tu código en una forma donde sea fácil hacer optimizaciones locales. Por ejemplo, si tienes un procedimiento grande que realiza varios cálculos lentos y complejos, podrías ver si puedes dividir partes de ese procedimiento en sus propias funciones donde los valores puedan ser precomputados o memoizados.

Las funciones sin estado (funciones que solo operan en los parámetros que se les pasaron - es decir, sin globales o llamadas al sistema) son el tipo más fácil de funciones de optimizar en una computadora. Cuantas más partes sin estado de tu programa tengas, más oportunidades tienes de optimizar. En la situación de comercio electrónico que escribí anteriormente, la computadora tenía que encontrar todas las partes asociadas para elementos de inventario específicos. Esto requería alrededor de 12 llamadas a la base de datos, y en el peor de los casos tomaba alrededor de 20 segundos. Sin embargo, el objetivo de este programa era ser interactivo, y una larga espera destruiría ese objetivo. Sin embargo, sabía que estas configuraciones de inventario no cambian. Por lo tanto, convertí las llamadas a la base de datos en sus propias funciones, que eran sin estado. Pude entonces memoizar las funciones. Al comienzo de cada día, los resultados de las funciones se limpiaban en caso de que alguien los hubiera cambiado, y varios elementos de inventario se precargaban automáticamente. A partir de entonces durante el día, la primera vez que alguien accedía a un elemento de inventario, tomaba los 20 segundos que tomaba antes, pero después tomaba menos de un segundo, porque los resultados de la base de datos habían sido memoizados.

La optimización global generalmente implica lograr las siguientes propiedades en tus funciones:

### [[#paralelización|Paralelización]]

La paralelización significa que tu algoritmo puede dividirse efectivamente entre múltiples procesos. Por ejemplo, el embarazo no es muy paralelizable porque no importa cuántas mujeres tengas, todavía toma nueve meses. Sin embargo, construir un auto es paralelizable porque puedes tener un trabajador trabajando en el motor mientras otro trabaja en el interior. Generalmente, las aplicaciones tienen un límite de cuán paralelizables son. Cuanto más paralelizable sea tu aplicación, mejor puede aprovechar las configuraciones de computadoras multiprocesador y en clúster.

### Sin Estado (Statelessness)

Como hemos discutido, las funciones y programas sin estado son aquellos que dependen enteramente de los datos explícitamente pasados para su funcionamiento. La mayoría de los procesos no son completamente sin estado, pero pueden serlo dentro de límites. En mi ejemplo de comercio electrónico, la función no era completamente sin estado, pero lo era dentro de los confines de un solo día. Por lo tanto, la optimicé como si fuera una función sin estado, pero hice concesiones para cambios durante la noche. Dos grandes beneficios resultantes de la falta de estado son que la mayoría de las funciones sin estado son paralelizables y a menudo se benefician de la memoización.

La optimización global requiere bastante práctica para saber qué funciona y qué no. Decidir cómo abordar los problemas de optimización en el código implica mirar todos los problemas, y saber que arreglar algunos problemas puede causar otros.

## Revisión

### Conoce los Conceptos

*   ¿En qué nivel de importancia está la optimización en comparación con las otras prioridades en la programación?
*   ¿Cuál es la diferencia entre optimizaciones locales y globales?
*   Nombra algunos tipos de optimizaciones locales.
*   ¿Cómo determinas qué partes de tu programa necesitan optimización?
*   ¿En qué nivel de importancia está la optimización en comparación con las otras prioridades en la programación? ¿Por qué crees que repetí esa pregunta?

### Usa los Conceptos

*   Vuelve a cada programa en este libro e intenta hacer optimizaciones según los procedimientos descritos en este capítulo.
*   Elige un programa del ejercicio anterior e intenta calcular el impacto en el rendimiento de tu código bajo entradas específicas.<sup>2</sup>

### Yendo Más Allá

*   Encuentra un programa de código abierto que encuentres particularmente rápido. Contacta a uno de los desarrolladores y pregunta sobre qué tipo de optimizaciones realizaron para mejorar la velocidad.
*   Encuentra un programa de código abierto que encuentres particularmente lento, e intenta imaginar las razones de la lentitud. Luego, descarga el código e intenta perfilario usando `gprof` o una herramienta similar. Encuentra dónde el código está pasando la mayoría del tiempo e intenta optimizarlo. ¿Fue la razón de la lentitud diferente de lo que imaginaste?
*   ¿Ha eliminado el compilador la necesidad de optimizaciones locales? ¿Por qué o por qué no?
*   ¿Qué tipo de problemas podría encontrar un compilador si intentara optimizar código a través de los límites de las llamadas a funciones?

---
<sup>2</sup> Dado que estos programas suelen ser lo suficientemente cortos como para no tener problemas de rendimiento notables, recorrer los programas miles de veces exagerará el tiempo que lleva ejecutarlos lo suficiente como para hacer cálculos.
