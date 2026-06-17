# Capítulo 2. Arquitectura de Computadoras

Antes de aprender a programar, primero debes entender cómo interpreta una computadora los programas. No necesitas un título en ingeniería eléctrica, pero sí comprender algunos conceptos básicos.

La arquitectura de las computadoras modernas se basa en una arquitectura llamada arquitectura de Von Neumann, nombrada así por su creador. La arquitectura de Von Neumann divide la computadora en dos partes principales: la CPU (del inglés Central Processing Unit, Unidad Central de Procesamiento) y la memoria. Esta arquitectura se utiliza en todas las computadoras modernas, incluyendo computadoras personales, supercomputadoras, mainframes e incluso teléfonos celulares.

## Estructura de la Memoria de la Computadora

Para entender cómo ve la computadora la memoria, imagina tu oficina de correos local. Normalmente tienen una habitación llena de apartados de correos. Estos casilleros son similares a la memoria de la computadora en el sentido de que cada uno es una secuencia numerada de ubicaciones de almacenamiento de tamaño fijo. Por ejemplo, si tienes 256 megabytes de memoria, eso significa que tu computadora contiene aproximadamente 256 millones de ubicaciones de almacenamiento de tamaño fijo. O, para usar nuestra analogía, 256 millones de apartados de correos. Cada ubicación tiene un número y cada ubicación tiene el mismo tamaño de longitud fija. La diferencia entre un apartado de correos y la memoria de la computadora es que puedes guardar todo tipo de cosas diferentes en un apartado de correos, pero solo puedes guardar un único número en una ubicación de almacenamiento de la memoria de la computadora.

### Las ubicaciones de memoria son como apartados de correos

| | | | |
|---|---|---|---|
| 2204 | 2205 | 2206 | 2207 |
| 2208 | 2209 | 2210 | 2211 |
| 2212 | 2213 | 2214 | 2215 |
| 2216 | 2217 | 2218 | 2219 |

Te preguntarás por qué una computadora está organizada de esta manera. Es porque es sencillo de implementar. Si la computadora estuviera compuesta por muchas ubicaciones de diferentes tamaños, o si pudieras almacenar diferentes tipos de datos en ellas, sería difícil y costoso de implementar.

La memoria de la computadora se utiliza para una serie de cosas diferentes. Todos los resultados de cualquier cálculo se almacenan en la memoria. De hecho, todo lo que se "almacena" se guarda en la memoria. Piensa en tu computadora en casa e imagina todo lo que está almacenado en su memoria:

- La ubicación de tu cursor en la pantalla
- El tamaño de cada ventana en la pantalla
- La forma de cada letra de cada fuente utilizada
- El diseño de todos los controles en cada ventana
- Los gráficos de todos los iconos de la barra de herramientas
- El texto de cada mensaje de error y cuadro de diálogo
- La lista sigue y sigue...

Además de todo esto, la arquitectura de Von Neumann especifica que no solo los datos de la computadora deben vivir en la memoria, sino que los programas que controlan el funcionamiento de la computadora también deben vivir allí. De hecho, en una computadora, no hay diferencia entre un programa y los datos de un programa, excepto en cómo los usa la computadora. Ambos se almacenan y se accede a ellos de la misma manera.

## La CPU

Entonces, ¿cómo funciona la computadora? Obviamente, el simple hecho de almacenar datos no ayuda mucho: es necesario poder acceder a ellos, manipularlos y moverlos. Ahí es donde entra la CPU.

La CPU lee las instrucciones de la memoria una por una y las ejecuta. Esto se conoce como el *ciclo de búsqueda-ejecución* (fetch-execute cycle). La CPU contiene los siguientes elementos para lograr esto:

- Contador de programa (Program Counter)
- Decodificador de instrucciones (Instruction Decoder)
- Bus de datos (Data bus)
- Registros de propósito general
- Unidad aritmética lógica (Arithmetic and logic unit)

El **contador de programa** se utiliza para decirle a la computadora de dónde debe buscar la siguiente instrucción. Mencionamos anteriormente que no hay diferencia entre la forma en que se almacenan los datos y los programas, simplemente la CPU los interpreta de manera diferente. El contador de programa contiene la dirección de memoria de la próxima instrucción que se ejecutará. La CPU comienza mirando el contador de programa y buscando cualquier número que esté almacenado en la memoria en la ubicación especificada. Luego se pasa al **decodificador de instrucciones**, que determina qué significa la instrucción. Esto incluye qué proceso debe llevarse a cabo (suma, resta, multiplicación, movimiento de datos, etc.) y qué ubicaciones de memoria estarán involucradas en este proceso. Las instrucciones de la computadora generalmente consisten tanto en la instrucción real como en la lista de ubicaciones de memoria que se utilizan para llevarla a cabo.

Ahora la computadora utiliza el **bus de datos** para buscar las ubicaciones de memoria que se utilizarán en el cálculo. El bus de datos es la conexión entre la CPU y la memoria. Es el cable real que los conecta. Si miras la placa base de la computadora, los cables que salen de la memoria son tu bus de datos.

Además de la memoria en el exterior del procesador, el propio procesador tiene algunas ubicaciones de memoria especiales de alta velocidad llamadas **registros**. Hay dos tipos de registros: **registros generales** y **registros de propósito especial**. Los registros de propósito general es donde ocurre la acción principal. La suma, resta, multiplicación, comparaciones y otras operaciones generalmente usan registros de propósito general para el procesamiento. Sin embargo, las computadoras tienen muy pocos registros de propósito general. La mayor parte de la información se almacena en la memoria principal, se lleva a los registros para su procesamiento y luego se devuelve a la memoria cuando se completa el procesamiento. Los registros de propósito especial son registros que tienen propósitos muy específicos. Discutiremos estos a medida que los encontremos.

Ahora que la CPU ha recuperado todos los datos que necesita, pasa los datos y la instrucción decodificada a la **unidad aritmética lógica** para su posterior procesamiento. Aquí la instrucción se ejecuta realmente. Después de que se han calculado los resultados de la computación, los resultados se colocan en el bus de datos y se envían a la ubicación adecuada en la memoria o en un registro, según lo especificado por la instrucción.

Esta es una explicación muy simplificada. Los procesadores han avanzado bastante en los últimos años y ahora son mucho más complejos. Aunque el funcionamiento básico sigue siendo el mismo, se complica por el uso de jerarquías de caché, procesadores superescalares, segmentación (pipelining), predicción de saltos, ejecución fuera de orden, traducción de microcódigo, coprocesadores y otras optimizaciones. No te preocupes si no sabes qué significan esas palabras, puedes usarlas como términos de búsqueda en Internet si quieres aprender más sobre la CPU.

## Algunos Términos

La memoria de la computadora es una secuencia numerada de ubicaciones de almacenamiento de tamaño fijo. El número asignado a cada ubicación de almacenamiento se llama su **dirección** (address). El tamaño de una única ubicación de almacenamiento se llama **byte**. En los procesadores x86, un byte es un número entre 0 y 255.

Te preguntarás cómo las computadoras pueden mostrar y usar texto, gráficos e incluso números grandes cuando todo lo que pueden hacer es almacenar números entre 0 y 255. En primer lugar, el hardware especializado, como las tarjetas gráficas, tiene interpretaciones especiales de cada número. Al mostrar en la pantalla, la computadora usa tablas de códigos ASCII para traducir los números que le envías en letras para mostrar en la pantalla, traduciendo cada número a exactamente una letra o cifra. [^1] Por ejemplo, la letra A mayúscula está representada por el número 65. El número 1 está representado por el número 49. Entonces, para imprimir "HELLO", en realidad le darías a la computadora la secuencia de números 72, 69, 76, 76, 79. Para imprimir el número 100, le darías a la computadora la secuencia de números 49, 48, 48. Una lista de caracteres ASCII y sus códigos numéricos se encuentra en el Apéndice D.

Además de usar números para representar caracteres ASCII, tú como programador también puedes hacer que los números signifiquen lo que quieras. Por ejemplo, si estoy administrando una tienda, usaría un número para representar cada artículo que estoy vendiendo. Cada número estaría vinculado a una serie de otros números que serían los códigos ASCII de lo que quería mostrar cuando se escanearan los artículos. Tendría más números para el precio, cuántos tengo en inventario, y así sucesivamente.

Entonces, ¿qué pasa si necesitamos números mayores que 255? Simplemente podemos usar una combinación de bytes para representar números más grandes. Se pueden usar dos bytes para representar cualquier número entre 0 y 65536. Se pueden usar cuatro bytes para representar cualquier número entre 0 y 4294967295. Ahora bien, es bastante difícil escribir programas para unir bytes con el fin de aumentar el tamaño de tus números, y requiere un poco de matemáticas. Afortunadamente, la computadora lo hará por nosotros para números de hasta 4 bytes de largo. De hecho, los números de cuatro bytes son con los que trabajaremos por defecto.

Mencionamos anteriormente que, además de la memoria normal que tiene la computadora, también tiene ubicaciones de almacenamiento de propósito especial llamadas **registros**. Los registros son lo que la computadora usa para los cálculos. Piensa en un registro como un lugar en tu escritorio: guarda las cosas en las que estás trabajando actualmente. Puede que tengas mucha información guardada en carpetas y cajones, pero lo que estás trabajando ahora mismo está en el escritorio. Los registros mantienen los contenidos de los números que estás manipulando actualmente.

En las computadoras que estamos usando, los registros miden cuatro bytes cada uno. El tamaño de un registro típico se llama **tamaño de palabra** (word size) de una computadora. Los procesadores x86 tienen palabras de cuatro bytes. Esto significa que lo más natural en estas computadoras es realizar cálculos de cuatro bytes a la vez. Esto nos da aproximadamente 4 mil millones de valores.

Las **direcciones** también tienen cuatro bytes (1 palabra) de largo y, por lo tanto, también caben en un registro. Los procesadores x86 pueden acceder a hasta 4,294,967,296 bytes si hay suficiente memoria instalada. Nota que esto significa que podemos almacenar direcciones de la misma manera que almacenamos cualquier otro número. De hecho, la computadora no puede distinguir entre un valor que es una dirección, un valor que es un número, un valor que es un código ASCII o un valor que has decidido usar para otro propósito. Un número se convierte en un código ASCII cuando intentas mostrarlo. Un número se convierte en una dirección cuando intentas buscar el byte al que apunta. Tómate un momento para pensar en esto, porque es crucial para entender cómo funcionan los programas de computadora.

Las direcciones que se almacenan en la memoria también se llaman **punteros** (pointers), porque en lugar de tener un valor regular en ellas, te apuntan a una ubicación diferente en la memoria.

Como hemos mencionado, las instrucciones de la computadora también se almacenan en la memoria. De hecho, se almacenan exactamente de la misma manera que se almacenan otros datos. La única forma en que la computadora sabe que una ubicación de memoria es una instrucción es que un registro de propósito especial llamado **puntero de instrucción** (instruction pointer) apunta a ellas en un momento u otro. Si el puntero de instrucción apunta a una palabra de memoria, se carga como una instrucción. Aparte de eso, la computadora no tiene forma de saber la diferencia entre los programas y otros tipos de datos. [^2]

## Interpretando la Memoria

Las computadoras son muy exactas. Debido a que son exactas, los programadores tienen que ser igualmente exactos. Una computadora no tiene idea de lo que se supone que debe hacer tu programa. Por lo tanto, solo hará exactamente lo que tú le digas que haga. Si accidentalmente imprimes un número normal en lugar de los códigos ASCII que forman los dígitos del número, la computadora te dejará hacerlo, y terminarás con galimatías en tu pantalla (intentará buscar qué representa tu número en ASCII e imprimirá eso). Si le dices a la computadora que comience a ejecutar instrucciones en una ubicación que contiene datos en lugar de instrucciones de programa, quién sabe cómo lo interpretará la computadora, pero ciertamente lo intentará. La computadora ejecutará tus instrucciones en el orden exacto que especifiques, incluso si no tiene sentido.

El punto es que la computadora hará exactamente lo que le digas, sin importar el poco sentido que tenga. Por lo tanto, como programador, debes saber exactamente cómo tienes organizados tus datos en la memoria. Recuerda, las computadoras solo pueden almacenar números, por lo que las letras, imágenes, música, páginas web, documentos y cualquier otra cosa son solo largas secuencias de números en la computadora, que ciertos programas saben cómo interpretar.

Por ejemplo, supongamos que deseas almacenar información de clientes en la memoria. Una forma de hacerlo sería establecer un tamaño máximo para el nombre y la dirección del cliente, por ejemplo, 50 caracteres ASCII para cada uno, lo que representaría 50 bytes para cada uno. Luego, después de eso, tendrías un número para la edad del cliente y su ID de cliente. En este caso, tendrías un bloque de memoria que se vería así:

**Inicio del Registro:**
- Nombre del cliente (50 bytes) - inicio del registro
- Dirección del cliente (50 bytes) - inicio del registro + 50 bytes
- Edad del cliente (1 palabra - 4 bytes) - inicio del registro + 100 bytes
- Número de ID del cliente (1 palabra - 4 bytes) - inicio del registro + 104 bytes

De esta manera, dada la dirección de un registro de cliente, sabes dónde se encuentran el resto de los datos. Sin embargo, limita el nombre y la dirección del cliente a solo 50 caracteres ASCII cada uno.

¿Qué pasaría si no quisiéramos especificar un límite? Otra forma de hacer esto sería tener en nuestro registro punteros a esta información. Por ejemplo, en lugar del nombre del cliente, tendríamos un puntero a su nombre. En este caso, la memoria se vería así:

**Inicio del Registro:**
- Puntero al nombre del cliente (1 palabra) - inicio del registro
- Puntero a la dirección del cliente (1 palabra) - inicio del registro + 4
- Edad del cliente (1 palabra) - inicio del registro + 8
- Número de ID del cliente (1 palabra) - inicio del registro + 12

El nombre y la dirección reales se almacenarían en otro lugar de la memoria. De esta manera, es fácil saber dónde está cada parte de los datos desde el inicio del registro, sin limitar explícitamente el tamaño del nombre y la dirección. Si la longitud de los campos dentro de nuestros registros pudiera cambiar, no tendríamos idea de dónde comienza el siguiente campo. Debido a que los registros tendrían diferentes tamaños, también sería difícil encontrar dónde comienza el siguiente registro. Por lo tanto, casi todos los registros tienen longitudes fijas. Los datos de longitud variable suelen almacenarse por separado del resto del registro.

## Métodos de Acceso a Datos

Los procesadores tienen varias formas diferentes de acceder a los datos, conocidas como **modos de direccionamiento** (addressing modes).

El modo más simple es el **modo inmediato** (immediate mode), en el cual los datos a los que se va a acceder están incrustados en la propia instrucción. Por ejemplo, si queremos inicializar un registro a 0, en lugar de darle a la computadora una dirección para leer el 0, especificaríamos el modo inmediato y le daríamos el número 0.

En el **modo de direccionamiento por registro** (register addressing mode), la instrucción contiene un registro al que acceder, en lugar de una ubicación de memoria. El resto de los modos tratarán con direcciones.

En el **modo de direccionamiento directo** (direct addressing mode), la instrucción contiene la dirección de memoria a la que acceder. Por ejemplo, podría decir: por favor, carga este registro con los datos en la dirección 2002. La computadora iría directamente al byte número 2002 y copiaría el contenido en nuestro registro.

En el **modo de direccionamiento indexado** (indexed addressing mode), la instrucción contiene una dirección de memoria a la que acceder y también especifica un *registro de índice* para desplazar esa dirección. Por ejemplo, podríamos especificar la dirección 2002 y un registro de índice. Si el registro de índice contiene el número 4, la dirección real de la que se cargan los datos sería 2006. De esta manera, si tienes un conjunto de números que comienzan en la ubicación 2002, puedes alternar entre cada uno de ellos usando un registro de índice. En los procesadores x86, también puedes especificar un *multiplicador* para el índice. Esto te permite acceder a la memoria byte a byte o palabra a palabra (4 bytes). Si estás accediendo a una palabra completa, tu índice deberá multiplicarse por 4 para obtener la ubicación exacta del cuarto elemento desde tu dirección. Por ejemplo, si quisieras acceder al cuarto byte desde la ubicación 2002, cargarías tu registro de índice con 3 (recuerda, empezamos a contar en 0) y establecerías el multiplicador en 1, ya que vas byte a byte. Esto te daría la ubicación 2005. Sin embargo, si quisieras acceder a la cuarta palabra desde la ubicación 2002, cargarías tu registro de índice con 3 y establecerías el multiplicador en 4. Esto cargaría desde la ubicación 2014, la cuarta palabra. Tómate el tiempo para calcular estos tú mismo para asegurarte de que entiendes cómo funciona.

En el **modo de direccionamiento indirecto** (indirect addressing mode), la instrucción contiene un registro que contiene un puntero a donde se debe acceder a los datos. Por ejemplo, si usáramos el modo de direccionamiento indirecto y especificáramos el registro `%eax`, y el registro `%eax` contuviera el valor 4, se usaría cualquier valor que estuviera en la ubicación de memoria 4. En el direccionamiento directo, simplemente cargaríamos el valor 4, pero en el direccionamiento indirecto, usamos 4 como la dirección para encontrar los datos que queremos.

Finalmente, está el **modo de direccionamiento por puntero base** (base pointer addressing mode). Este es similar al direccionamiento indirecto, pero también incluyes un número llamado *desplazamiento* (offset) para sumar al valor del registro antes de usarlo para la búsqueda. Usaremos este modo bastante en este libro. En la sección llamada Interpretando la Memoria discutimos tener una estructura en la memoria que contiene información del cliente. Digamos que quisiéramos acceder a la edad del cliente, que era el octavo byte de los datos, y tuviéramos la dirección del inicio de la estructura en un registro. Podríamos usar el direccionamiento por puntero base y especificar el registro como el puntero base y 8 como nuestro desplazamiento. Esto se parece mucho al direccionamiento indexado, con la diferencia de que el desplazamiento es constante y el puntero se mantiene en un registro, mientras que en el direccionamiento indexado el desplazamiento está en un registro y el puntero es constante.

Existen otras formas de direccionamiento, pero estas son las más importantes.

## Revisión

### Conoce los Conceptos

- Describe el ciclo de búsqueda-ejecución (fetch-execute cycle).
- ¿Qué es un registro? ¿Cómo sería la computación más difícil sin registros?
- ¿Cómo representas números mayores a 255?
- ¿Qué tan grandes son los registros en las máquinas que utilizaremos?
- ¿Cómo sabe una computadora cómo interpretar un byte o conjunto de bytes de memoria determinados?
- ¿Cuáles son los modos de direccionamiento y para qué se utilizan?
- ¿Qué hace el puntero de instrucción?

### Usa los Conceptos

- ¿Qué datos usarías en un registro de empleado? ¿Cómo lo organizarías en la memoria?
- Si tuviera el puntero al comienzo del registro de empleado anterior y quisiera acceder a una pieza de datos en particular dentro de él, ¿qué modo de direccionamiento usaría?
- En el modo de direccionamiento por puntero base, si tienes un registro con el valor 3122 y un desplazamiento de 20, ¿a qué dirección intentarías acceder?
- En el modo de direccionamiento indexado, si la dirección base es 6512, el registro de índice tiene un 5 y el multiplicador es 4, ¿a qué dirección intentarías acceder?
- En el modo de direccionamiento indexado, si la dirección base es 123472, el registro de índice tiene un 0 y el multiplicador es 4, ¿a qué dirección intentarías acceder?
- En el modo de direccionamiento indexado, si la dirección base es 9123478, el registro de índice tiene un 20 y el multiplicador es 1, ¿a qué dirección intentarías acceder?

### Yendo más allá

- ¿Cuál es el número mínimo de modos de direccionamiento necesarios para la computación?
- ¿Por qué incluir modos de direccionamiento que no son estrictamente necesarios?
- Investiga y luego describe cómo la segmentación (pipelining, o uno de los otros factores complicados) afecta el ciclo de búsqueda-ejecución.
- Investiga y luego describe las compensaciones entre instrucciones de longitud fija e instrucciones de longitud variable.

---

[^1]: Con la llegada de los juegos de caracteres internacionales y Unicode, esto ya no es del todo cierto. Sin embargo, para mantener esto simple para los principiantes, asumiremos que un número se traduce directamente a un carácter. Para más información, consulta el Apéndice D.
[^2]: Ten en cuenta que aquí estamos hablando de teoría general de computadoras. Algunos procesadores y sistemas operativos marcan las regiones de la memoria que se pueden ejecutar con una marca especial que indica esto.
