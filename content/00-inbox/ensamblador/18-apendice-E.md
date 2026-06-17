# Apéndice E. Modismos de C en Lenguaje Ensamblador

Este apéndice es para programadores de C que aprenden [[02-capitulo-2|lenguaje ensamblador]]. Está destinado a dar una idea general sobre cómo las construcciones de C pueden implementarse en lenguaje ensamblador.

## Sentencia If

En C, una sentencia `if` consta de tres partes - la condición, la rama verdadera y la rama falsa. Sin embargo, como el lenguaje ensamblador no es un lenguaje estructurado por bloques, tienes que trabajar un poco para implementar la naturaleza de bloques de C. Por ejemplo, mira el siguiente código en C:

```c
if(a == b)
{
    /* Código de la Rama Verdadera Aquí */
}
else
{
    /* Código de la Rama Falsa Aquí */
}

/* En Este Punto, Re convergencia */
```

En lenguaje ensamblador, esto puede representarse como:

```assembly
#Mover a y b a registros para comparar
movl a, %eax
movl b, %ebx

#Comparar
cmpl %eax, %ebx

#Si es Verdadero, ir a la rama verdadera
je true_branch

false_branch: #Esta etiqueta es innecesaria,
              #solo está aquí para documentación
#Código de la Rama Falsa Aquí

#Saltar al punto de reconvergencia
jmp reconverge

true_branch:
#Código de la Rama Verdadera Aquí

reconverge:
#Ambas ramas reconvergen a este punto
```

Como puedes ver, como el lenguaje ensamblador es lineal, los bloques tienen que saltar unos sobre otros. La reconvergencia es manejada por el programador, no por el sistema.

Una sentencia `case` se escribe como una secuencia de sentencias `if`.

## Llamada a Función

Una llamada a función en lenguaje ensamblador simplemente requiere empujar los argumentos a la función en la pila en orden inverso, y emitir una instrucción `call`. Después de llamar, los argumentos se sacan de la pila. Por ejemplo, considera el código en C:

```c
printf("The number is %d", 88);
```

En lenguaje ensamblador, esto se representaría como:

```assembly
.section .data
text_string:
.ascii "The number is %d\0"

.section .text
pushl $88
pushl $text_string
call printf
popl %eax
popl %eax #%eax es solo una variable dummy,
          #no se está haciendo nada realmente
          #con el valor. También puedes
          #reajustar %esp directamente a la
          #ubicación adecuada.
```

## Variables y Asignación

Las variables globales y estáticas se declaran usando entradas `.data` o `.bss`. Las variables locales se declaran reservando espacio en la pila al principio de la función. Este espacio se devuelve al final de la función.

Curiosamente, las variables globales se acceden de manera diferente a las variables locales en lenguaje ensamblador. Las variables globales se acceden usando direccionamiento directo, mientras que las variables locales se acceden usando direccionamiento de puntero base. Por ejemplo, considera el siguiente código en C:

```c
int my_global_var;

int foo()
{
    int my_local_var;
    my_local_var = 1;
    my_global_var = 2;

    return 0;
}
```

Esto se representaría en lenguaje ensamblador como:

```assembly
.section .data
.lcomm my_global_var, 4

.type foo, @function
foo:
pushl %ebp             #Guardar el puntero base anterior
movl %esp, %ebp        #hacer que el puntero de pila sea el puntero base
subl $4, %esp          #Hacer espacio para my_local_var
.equ my_local_var, -4  #Ahora podemos usar my_local_var para
                       #encontrar la variable local

movl $1, my_local_var(%ebp)
movl $2, my_global_var

movl %ebp, %esp        #Limpiar la función y regresar
popl %ebp
ret
```

Lo que puede no ser obvio es que acceder a la variable global toma menos ciclos de máquina que acceder a la variable local. Sin embargo, eso puede no importar porque la pila tiene más probabilidades de estar en memoria física (en lugar de swap) que la variable global.

También nota que en el lenguaje de programación C, después de que el compilador carga un valor en un registro, ese valor probablemente permanecerá en ese registro hasta que ese registro sea necesario para otra cosa. También puede mover registros. Por ejemplo, si tienes una variable `foo`, puede comenzar en la pila, pero el compilador eventualmente la moverá a registros para procesamiento. Si no hay muchas variables en uso, el valor puede simplemente permanecer en el registro hasta que se necesite de nuevo. De lo contrario, cuando ese registro se necesita para otra cosa, el valor, si ha cambiado, se copia de vuelta a su ubicación de memoria correspondiente. En C, puedes usar la palabra clave `volatile` para asegurarte de que todas las modificaciones y referencias a la variable se hagan a la ubicación de memoria misma, en lugar de a una copia en registro, en caso de que otros procesos, hilos o hardware puedan estar modificando el valor mientras tu función se está ejecutando.

## Bucles

Los bucles funcionan muy parecido a las sentencias `if` en lenguaje ensamblador - los bloques se forman saltando. En C, un bucle `while` consiste en un cuerpo de bucle, y una prueba para determinar si es hora de salir del bucle. Un bucle `for` es exactamente lo mismo, con secciones opcionales de inicialización e incremento de contador. Estos pueden simplemente moverse para hacer un bucle `while`.

En C, un bucle `while` se ve así:

```c
while(a < b)
{
    /* Hacer cosas aquí */
}

/* Terminado de recorrer el bucle */
```

Esto puede representarse en lenguaje ensamblador así:

```assembly
loop_begin:
movl a, %eax
movl b, %ebx
cmpl %eax, %ebx
jge loop_end

loop_body:
#Hacer cosas aquí

jmp loop_begin

loop_end:
#Terminado de recorrer el bucle
```

El lenguaje ensamblador x86 tiene algún soporte directo para bucles también. El registro `%ecx` puede usarse como un contador que termina en cero. La instrucción `loop` decrementará `%ecx` y saltará a una dirección especificada a menos que `%ecx` sea cero. Por ejemplo, si quisieras ejecutar una sentencia 100 veces, harías esto en C:

```c
for(i=0; i < 100; i++)
{
    /* Hacer proceso aquí */
}
```

En lenguaje ensamblador se escribiría así:

```assembly
loop_initialize:
movl $100, %ecx

loop_begin:
#
#Hacer Proceso Aquí
#

#Decrementar %ecx y repetir si no es cero
loop loop_begin

rest_of_program:
#Continúa hasta aquí
```

Una cosa a notar es que la instrucción `loop` requiere que estés contando hacia atrás hasta cero. Si necesitas contar hacia adelante o usar otro número final, deberías usar la forma de bucle que no incluya la instrucción `loop`.

Para bucles realmente ajustados de operaciones con cadenas de caracteres, también está la instrucción `rep`, pero dejaremos aprender sobre eso como un ejercicio para el lector.

## Structs

Los structs son simplemente descripciones de bloques de memoria. Por ejemplo, en C puedes decir:

```c
struct person {
    char firstname[40];
    char lastname[40];
    int age;
};
```

Esto no hace nada por sí mismo, excepto darte formas de usar inteligentemente 84 bytes de datos. Puedes hacer básicamente lo mismo usando directivas `.equ` en lenguaje ensamblador. Así:

```assembly
.equ PERSON_SIZE, 84
.equ PERSON_FIRSTNAME_OFFSET, 0
.equ PERSON_LASTNAME_OFFSET, 40
.equ PERSON_AGE_OFFSET, 80
```

Cuando declaras una variable de este tipo, todo lo que estás haciendo es reservar 84 bytes de espacio. Por lo tanto, si tienes esto en C:

```c
void foo()
{
    struct person p;

    /* Hacer cosas aquí */
}
```

En lenguaje ensamblador tendrías:

```assembly
foo:
#Cabecera estándar de inicio
pushl %ebp
movl %esp, %ebp

#Reservar nuestra variable local
subl $PERSON_SIZE, %esp
#Este es el desplazamiento de la variable desde %ebp
.equ P_VAR, 0 - PERSON_SIZE

#Hacer Cosas Aquí

#Final estándar de función
movl %ebp, %esp
popl %ebp
ret
```

Para acceder a los miembros de la estructura, solo tienes que usar direccionamiento de puntero base con los desplazamientos definidos anteriormente. Por ejemplo, en C podrías establecer la edad de la persona así:

```c
p.age = 30;
```

En lenguaje ensamblador se vería así:

```assembly
movl $30, P_VAR + PERSON_AGE_OFFSET(%ebp)
```

## Punteros

Los punteros son muy fáciles. Recuerda, los punteros son simplemente la dirección en la que reside un valor. Comencemos viendo las variables globales. Por ejemplo:

```c
int global_data = 30;
```

En lenguaje ensamblador, esto sería:

```assembly
.section .data
global_data:
.long 30
```

Tomar la dirección de estos datos en C:

```c
a = &global_data;
```

Tomar la dirección de estos datos en lenguaje ensamblador:

```assembly
movl $global_data, %eax
```

Verás, con el lenguaje ensamblador, casi siempre estás accediendo a la memoria a través de punteros. De eso se trata el direccionamiento directo. Para obtener el puntero mismo, solo tienes que usar el direccionamiento en modo inmediato.

Las variables locales son un poco más difíciles, pero no mucho. Aquí está cómo tomar la dirección de una variable local en C:

```c
void foo()
{
    int a;
    int *b;

    a = 30;
    b = &a;

    *b = 44;
}
```

El mismo código en lenguaje ensamblador:

```assembly
foo:
#Apertura estándar
pushl %ebp
movl %esp, %ebp

#Reservar dos palabras de memoria
subl $8, %esp
.equ A_VAR, -4
.equ B_VAR, -8

#a = 30
movl $30, A_VAR(%ebp)

#b = &a
movl $A_VAR, B_VAR(%ebp)
addl %ebp, B_VAR(%ebp)

#*b = 30
movl B_VAR(%ebp), %eax
movl $30, (%eax)

#Cierre estándar
movl %ebp, %esp
popl %ebp
ret
```

Como puedes ver, para tomar la dirección de una variable local, la dirección tiene que calcularse de la misma manera que la computadora calcula las direcciones en el direccionamiento de puntero base. Hay una forma más fácil - el procesador proporciona la instrucción `leal`, que significa "load effective address" (cargar dirección efectiva). Esto permite que la computadora calcule la dirección, y luego la cargue donde quieras. Por lo tanto, podríamos decir simplemente:

```assembly
#b = &a
leal A_VAR(%ebp), %eax
movl %eax, B_VAR(%ebp)
```

Es la misma cantidad de líneas, pero un poco más limpio. Luego, para usar este valor, simplemente tienes que moverlo a un registro de propósito general y usar direccionamiento indirecto, como se muestra en el ejemplo anterior.

## Obteniendo Ayuda de GCC

Una de las cosas buenas de GCC es su capacidad de generar código en lenguaje ensamblador. Para convertir un archivo de lenguaje C a ensamblador, puedes simplemente hacer:

```bash
gcc -S file.c
```

La salida estará en `file.s`. No es la salida más legible - la mayoría de los nombres de las variables han sido eliminados y reemplazados ya sea con ubicaciones numéricas de la pila o referencias a etiquetas generadas automáticamente. Para empezar, probablemente quieras desactivar las optimizaciones con `-O0` para que la salida del lenguaje ensamblador siga mejor tu código fuente.

Otra cosa que podrías notar es que GCC reserva más espacio en la pila para variables locales del que nosotros hacemos, y luego hace AND a `%esp`.<sup>1</sup> Esto es para aumentar la eficiencia de la memoria y el caché al alinear variables con palabras dobles.

Finalmente, al final de las funciones, usualmente hacemos las siguientes instrucciones para limpiar la pila antes de emitir una instrucción `ret`:

```assembly
movl %ebp, %esp
popl %ebp
```

Sin embargo, la salida de GCC generalmente incluirá solo la instrucción `leave`. Esta instrucción es simplemente la combinación de las dos instrucciones anteriores. No usamos `leave` en este texto porque queremos ser claros sobre exactamente lo que está sucediendo a nivel del procesador.

Te animo a que tomes un programa en C que hayas escrito y lo compiles a lenguaje ensamblador y traces la lógica. Luego, añade optimizaciones y prueba de nuevo. Observa cómo el compilador eligió reorganizar tu programa para que esté más optimizado, e intenta descubrir por qué eligió la disposición y las instrucciones que eligió.

---
<sup>1</sup> Nota que diferentes versiones de GCC hacen esto de manera diferente.
