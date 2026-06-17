# Capítulo 7. Desarrollo de Programas Robustos

Este capítulo trata sobre el desarrollo de programas que sean robustos. Los **programas robustos** son aquellos capaces de manejar condiciones de error con elegancia. Son programas que no se bloquean sin importar lo que haga el usuario. Construir programas robustos es esencial para la práctica de la programación. Escribir programas robustos requiere disciplina y trabajo; normalmente implica encontrar cada posible problema que pueda ocurrir e idear un plan de acción para que su programa lo lleve a cabo.

## ¿A dónde se va el tiempo?

Los programadores planifican mal los tiempos. En casi todos los proyectos de programación, los programadores tardarán dos, cuatro o incluso ocho veces más en desarrollar un programa o función de lo que estimaron originalmente. Hay muchas razones para este problema, entre ellas:

- Los programadores no siempre programan tiempo para reuniones u otras actividades ajenas a la codificación que forman parte de cada día.
- Los programadores a menudo subestiman los tiempos de retroalimentación (el tiempo que se tarda en pasar las solicitudes de cambio y las aprobaciones de un lado a otro) en los proyectos.
- Los programadores no siempre comprenden el alcance total de lo que están produciendo.
- Los programadores a menudo tienen que estimar un cronograma en un tipo de proyecto totalmente diferente al que están acostumbrados y, por lo tanto, son incapaces de planificar con precisión.
- Los programadores a menudo subestiman la cantidad de tiempo que se tarda en lograr que un programa sea totalmente robusto.

El último punto es el que nos interesa aquí. Se necesita mucho tiempo y esfuerzo para desarrollar programas robustos. Mucho más de lo que la gente suele suponer, incluidos los programadores experimentados. Los programadores se concentran tanto en resolver simplemente el problema en cuestión que no logran ver los posibles problemas colaterales.

En el programa `toupper`, no tenemos ningún curso de acción si el archivo que selecciona el usuario no existe. El programa seguirá adelante e intentará trabajar de todos modos. No informa de ningún mensaje de error, por lo que el usuario ni siquiera sabrá que escribió mal el nombre. Digamos que el archivo de destino está en una unidad de red y la red falla temporalmente. El sistema operativo nos devuelve un código de estado en `%eax`, pero no lo estamos comprobando. Por lo tanto, si se produce un fallo, el usuario no se entera en absoluto. Este programa definitivamente no es robusto. Como puede ver, incluso en un programa simple hay muchas cosas que pueden salir mal y a las que un programador debe enfrentarse.

En un programa grande, el problema es mucho mayor. Suele haber muchas más condiciones de error posibles que condiciones de éxito posibles. Por lo tanto, siempre debe esperar pasar la mayor parte de su tiempo comprobando códigos de estado, escribiendo controladores de errores y realizando tareas similares para que su programa sea robusto. Si se tardan dos semanas en desarrollar un programa, probablemente se tardarán al menos dos más en hacerlo robusto. Recuerde que cada mensaje de error que aparece en su pantalla tuvo que ser programado por alguien.

## Algunos consejos para desarrollar programas robustos

### Pruebas de Usuario

Las pruebas son una de las cosas más esenciales que hace un programador. Si no ha probado algo, debe asumir que no funciona. Sin embargo, las pruebas no consisten solo en asegurarse de que su programa funciona, sino en asegurarse de que su programa no se rompa. Por ejemplo, si tengo un programa que solo debe tratar con números positivos, debe probar qué sucede si el usuario introduce un número negativo. O una letra. O el número cero. Debe probar qué sucede si ponen espacios antes de sus números, espacios después de sus números y otras pequeñas posibilidades. Debe asegurarse de manejar los datos del usuario de una manera que tenga sentido para el usuario y de pasar esos datos de una manera que tenga sentido para el resto de su programa. Cuando su programa encuentra una entrada que no tiene sentido, debe realizar las acciones apropiadas. Dependiendo de su programa, esto puede incluir finalizar el programa, pedir al usuario que vuelva a introducir los valores, notificar a un registro de errores central, revertir una operación o ignorarlo y continuar.

No solo debe probar sus programas, sino que también debe hacer que otros los prueben. Debe recurrir a otros programadores y usuarios de su programa para que le ayuden a probarlo. Si algo es un problema para sus usuarios, aunque a usted le parezca bien, debe corregirse. Si el usuario no sabe cómo utilizar su programa correctamente, eso debe tratarse como un bug que debe ser corregido.

Descubrirá que los usuarios encuentran muchos más bugs en su programa de los que usted jamás podría encontrar. La razón es que los usuarios no saben lo que espera la computadora. Usted sabe qué tipos de datos espera la computadora y, por lo tanto, es mucho más probable que introduzca datos que tengan sentido para ella. Los usuarios introducen datos que tienen sentido para ellos. Permitir que personas que no son programadores utilicen su programa para realizar pruebas suele darle resultados mucho más precisos sobre lo robusto que es realmente su programa.

### Pruebas de Datos

Al diseñar programas, cada una de sus funciones debe ser muy específica sobre el tipo y el rango de datos que aceptará o no. Luego debe probar estas funciones para asegurarse de que cumplen con las especificaciones cuando se les entregan los datos adecuados. Lo más importante es probar los **casos esquina** (corner cases) o **casos límite** (edge cases). Los casos esquina son las entradas que tienen más probabilidades de causar problemas o de comportarse de forma inesperada.

Al probar datos numéricos, hay varios casos esquina que siempre debe probar:

- El número 0
- El número 1
- Un número dentro del rango esperado
- Un número fuera del rango esperado
- El primer número del rango esperado
- El último número del rango esperado
- El primer número por debajo del rango esperado
- El primer número por encima del rango esperado

Por ejemplo, si tengo un programa que debe aceptar valores entre 5 y 200, debería probar el 0, 1, 4, 5, 153, 200, 201 y 255 como mínimo (el 153 y el 255 se eligieron al azar dentro y fuera del rango, respectivamente). Lo mismo ocurre con cualquier lista de datos que tenga. Debe probar que su programa se comporte como se espera con listas de 0 elementos, 1 elemento, cantidades masivas de elementos, etc. Además, también debe probar cualquier punto de inflexión que tenga. Por ejemplo, si tiene un código diferente para manejar a personas menores y mayores de 30 años, por ejemplo, tendría que probarlo con personas de 29, 30 y 31 años al menos.

Habrá algunas funciones internas que usted asumirá que reciben datos correctos porque ha comprobado los errores antes de este punto. Sin embargo, durante el desarrollo a menudo es necesario comprobar los errores de todos modos, ya que el resto de su código puede tener errores. Para verificar la consistencia y validez de los datos durante el desarrollo, la mayoría de los lenguajes disponen de una herramienta para comprobar fácilmente las suposiciones sobre la corrección de los datos. En el lenguaje C existe la macro `assert`. Simplemente puede poner en su código `assert(a > b);`, y dará un error si llega a ese código cuando la condición no sea verdadera. Además, dado que dicha comprobación es una pérdida de tiempo una vez que su código es estable, la macro `assert` permite desactivar las aserciones en el momento de la compilación. Esto asegura que sus funciones están recibiendo buenos datos sin causar ralentizaciones innecesarias para el código lanzado al público.

### Pruebas de Módulos

No solo debe probar su programa en su conjunto, sino que debe probar las piezas individuales de su programa. A medida que desarrolla su programa, debe probar las funciones individuales proporcionándoles datos creados por usted para asegurarse de que responden adecuadamente.

Para hacerlo con eficacia, tiene que desarrollar funciones cuyo único propósito sea llamar a funciones para realizar pruebas. Estas se llaman **controladores de prueba** o **drivers** (no confundir con los controladores de hardware). Simplemente cargan su función, le suministran datos y comprueban los resultados. Esto es especialmente útil si está trabajando en piezas de un programa inacabado. Como no puede probar todas las piezas juntas, puede crear un programa controlador que probará cada función individualmente.

Además, el código que está probando puede realizar llamadas a funciones que aún no han sido desarrolladas. Para superar este problema, puede escribir una pequeña función llamada **sustituto** o **stub** que simplemente devuelve los valores que esa función necesita para continuar. Por ejemplo, en una aplicación de comercio electrónico, yo tenía una función llamada `is_ready_to_checkout`. Antes de tener tiempo de escribir realmente la función, simplemente la configuré para que devolviera verdadero (true) en cada llamada para que las funciones que dependían de ella tuvieran una respuesta. Esto me permitió probar las funciones que dependían de `is_ready_to_checkout` sin que la función estuviera totalmente implementada.

## Manejo de errores con eficacia

No solo es importante saber cómo realizar pruebas, sino que también es importante saber qué hacer cuando se detecta un error.

### Tenga un código de error para todo

El software verdaderamente robusto tiene un código de error único para cada contingencia posible. Con solo conocer el código de error, debería ser capaz de encontrar la ubicación en su código donde se señaló ese error. Esto es importante porque el código de error suele ser lo único que tiene el usuario cuando informa de errores. Por lo tanto, debe ser lo más útil posible.

Los códigos de error también deben ir acompañados de mensajes de error descriptivos. Sin embargo, solo en raras circunstancias el mensaje de error debe tratar de predecir *por qué* ocurrió el error. Simplemente debe relatar lo que sucedió. Allá por 1995 trabajé para un proveedor de servicios de Internet. Uno de los navegadores web que soportábamos intentaba adivinar la causa de cada error de red, en lugar de limitarse a informar del error. Si la computadora no estaba conectada a Internet y el usuario intentaba conectarse a un sitio web, decía que había un problema con el proveedor de servicios de Internet, que el servidor no funcionaba y que el usuario debía ponerse en contacto con su proveedor de servicios de Internet para corregir el problema. Casi una cuarta parte de nuestras llamadas procedían de personas que habían recibido este mensaje, pero que simplemente necesitaban conectarse a Internet antes de intentar usar su navegador. Como puede ver, intentar diagnosticar cuál es el problema puede acarrear muchos más problemas de los que soluciona. Es mejor limitarse a informar de los códigos y mensajes de error, y disponer de recursos separados para que el usuario solucione los problemas de la aplicación. Una guía de resolución de problemas, y no el programa en sí, es el lugar adecuado para enumerar las posibles razones y cursos de acción para cada mensaje de error.

### Puntos de recuperación

Para simplificar el manejo de errores, a menudo es útil dividir el programa en unidades distintas, donde cada unidad falla y se recupera como un todo. Por ejemplo, podría dividir su programa de modo que la lectura del archivo de configuración fuera una unidad. Si la lectura del archivo de configuración fallara en cualquier punto (al abrir el archivo, al leerlo, al intentar decodificarlo, etc.), el programa simplemente lo trataría como un problema del archivo de configuración y saltaría al punto de recuperación para ese problema. De esta manera, reduce enormemente el número de mecanismos de manejo de errores que necesita para su programa, porque la recuperación de errores se realiza a un nivel mucho más general.

Tenga en cuenta que, incluso con los puntos de recuperación, sus mensajes de error deben ser específicos en cuanto a cuál fue el problema. Los puntos de recuperación son unidades básicas para la recuperación de errores, no para la detección de errores. La detección de errores todavía tiene que ser extremadamente exacta, y los informes de error necesitan códigos y mensajes de error exactos.

Al utilizar puntos de recuperación, a menudo es necesario incluir código de limpieza para manejar diferentes contingencias. Por ejemplo, en nuestro ejemplo del archivo de configuración, la función de recuperación tendría que incluir código para comprobar si el archivo de configuración seguía abierto. Dependiendo de dónde se produjera el error, es posible que el archivo se hubiera quedado abierto. La función de recuperación debe comprobar esta condición, y cualquier otra que pudiera provocar inestabilidad en el sistema, y devolver el programa a un estado coherente.

La forma más sencilla de manejar los puntos de recuperación es envolver todo el programa en un único punto de recuperación. Solo tendría una función simple de informe de errores a la que puede llamar con un código de error y un mensaje. La función los imprimiría y simplemente saldría del programa. Normalmente no es la mejor solución para situaciones del mundo real, pero es un buen mecanismo de reserva de último recurso.

## Haciendo nuestro programa más robusto

Esta sección tratará sobre cómo hacer que el programa `add-year.s` del Capítulo 6 sea un poco más robusto. Dado que se trata de un programa bastante sencillo, nos limitaremos a un único punto de recuperación que cubra todo el programa. Lo único que haremos para recuperarnos será imprimir el error y salir. El código para hacerlo es bastante sencillo:

```assembly
# error-exit.s
.include "linux.s"
.equ ST_ERROR_CODE, 8
.equ ST_ERROR_MSG, 12

.globl error_exit
.type error_exit, @function
error_exit:
 pushl %ebp
 movl  %esp, %ebp

 #Escribir el código de error
 movl  ST_ERROR_CODE(%ebp), %ecx
 pushl %ecx
 call  count_chars
 popl  %ecx
 movl  %eax, %edx
 movl  $STDERR, %ebx
 movl  $SYS_WRITE, %eax
 int   $LINUX_SYSCALL

 #Escribir el mensaje de error
 movl  ST_ERROR_MSG(%ebp), %ecx
 pushl %ecx
 call  count_chars
 popl  %ecx
 movl  %eax, %edx
 movl  $STDERR, %ebx
 movl  $SYS_WRITE, %eax
 int   $LINUX_SYSCALL

 pushl $STDERR
 call  write_newline

 #Salir con estado 1
 movl  $SYS_EXIT, %eax
 movl  $1, %ebx
 int   $LINUX_SYSCALL
```

Introdúcelo en un archivo llamado `error-exit.s`. Para llamarlo, solo tienes que meter la dirección de un mensaje de error y luego un código de error en la pila, y llamar a la función.

Ahora busquemos posibles puntos de error en nuestro programa `add-year`. En primer lugar, no comprobamos si ninguna de nuestras llamadas al sistema open se completa realmente de forma adecuada. Linux devuelve su código de estado en `%eax`, por lo que tenemos que comprobar si hay un error.

```assembly
#Abrir archivo para lectura
movl $SYS_OPEN, %eax
movl $input_file_name, %ebx
movl $0, %ecx
movl $0666, %edx
int $LINUX_SYSCALL

movl %eax, INPUT_DESCRIPTOR(%ebp)

#Esto probará y verá si %eax es
#negativo. Si no es negativo,
#saltará a continue_processing.
#De lo contrario, manejará la condición de error
#que representa el número negativo.
cmpl $0, %eax
jl continue_processing

#Enviar el error
.section .data
no_open_file_code:
 .ascii "0001: \0"
no_open_file_msg:
 .ascii "No se puede abrir el archivo de entrada\0"

.section .text
pushl $no_open_file_msg
pushl $no_open_file_code
call error_exit

continue_processing:
#Resto del programa
```

Así, después de realizar la llamada al sistema, comprobamos si tenemos un error viendo si el resultado de la llamada al sistema es menor que cero. Si es así, llamamos a nuestra rutina de informe de errores y salida. Después de cada llamada al sistema, llamada a función o instrucción que pueda tener resultados erróneos, debería añadir código de comprobación y manejo de errores.

Para ensamblar y enlazar los archivos, haz lo siguiente:

```bash
as add-year.s -o add-year.o
as error-exit.s -o error-exit.o
ld add-year.o write-newline.o error-exit.o read-record.o \
   write-record.o count-chars.o -o add-year
```

Ahora intenta ejecutarlo sin los archivos necesarios. ¡Ahora sale de forma limpia y controlada!

## Revisión

### Conoce los Conceptos

- ¿Cuáles son las razones por las que los programadores tienen problemas con la planificación de tiempos?
- Busca tu programa favorito e intenta utilizarlo de una forma completamente incorrecta. Abre archivos del tipo equivocado, elige opciones no válidas, cierra ventanas que deberían estar abiertas, etc. Cuenta cuántos escenarios de error diferentes tuvieron que tener en cuenta.
- ¿Qué son los casos esquina (corner cases)? ¿Puedes enumerar ejemplos de casos esquina numéricos?
- ¿Por qué son tan importantes las pruebas de usuario?
- ¿Para qué se utilizan los stubs y los drivers? ¿Cuál es la diferencia entre ambos?
- ¿Para qué se utilizan los puntos de recuperación?
- ¿Cuántos códigos de error diferentes debería tener un programa?

### Usa los Conceptos

- Repasa el programa `add-year.s` y añade código de comprobación de errores después de cada llamada al sistema.
- Busca otro programa que hayamos hecho hasta ahora y añádele comprobación de errores.
- Añade un mecanismo de recuperación para `add-year.s` que le permita leer de STDIN si no puede abrir el archivo estándar.

### Yendo más allá

- ¿Qué debería hacer, si es que debe hacer algo, si su función de informe de errores falla? ¿Por qué?
- Intenta encontrar bugs en al menos un programa de código abierto. Presenta un informe de bug para él.
- Intenta corregir el bug que encontraste en el ejercicio anterior.
