# Sr. Hexadecimal: Tu nuevo mejor amigo.

El hexadecimal es una forma de representar datos informáticos de una manera distinta al código fuente real o al binario. Es un sistema numérico de Base 16, en lugar de Base 10 como el decimal (el sistema numérico que usamos en la vida cotidiana).

La mejor forma de entender cómo funciona la base de un sistema numérico es verlo de esta manera:

En primer lugar, se puede pensar que todos los números constan de dos "columnas": la columna de la izquierda y la columna de la derecha. La columna de la izquierda puede tener cualquier cantidad de dígitos, pero la columna de la derecha es siempre el dígito situado más a la derecha del número. Suena raro, pero aquí tienes un ejemplo:
 

---

 Número: 7: 0|7     10: 1|0    100: 10|0  3578: 357|8  
 Columnas:     L R         L R         L  R         L   R  

---

Como puedes ver, cuando tratas con números de un solo dígito, tienes que imaginar que hay un cero delante, para tener una columna izquierda.

Bien, ahora explicaremos el concepto de las bases.

El sistema decimal tiene los dígitos del 0 al 9. Diez dígitos. Cuando cuentas en decimal, cuentas hasta el número de un solo dígito más grande disponible, el 9. Para ir más allá, tienes que añadir 1 a la columna de la izquierda y restablecer la columna de la derecha a cero, así:

9-10, 19-20, 199-200, 9999-10000, 999999-1000000  
_(Recuerda imaginar que hay un cero delante del 9)_

¿Ves cómo funciona esto? Dado que este cambio de columna siempre ocurre después de una secuencia de diez números (10 dígitos para contar), se dice que el decimal es un sistema numérico de Base Diez.

El hexadecimal, por otro lado, es un sistema de Base Dieciséis. Contiene del 0 al 9 como el decimal, pero también incluye 6 dígitos más: A, B, C, D, E y F, para un total de dieciséis dígitos. Así que, para llegar a un cambio de columna, tienes que añadir 16 dígitos en la columna de la derecha, así:

0, 1, 2, 3, 4, 5, 6, 7, 8, 9, A, B, C, D, E, F, 10

¿Lo pillas? Sé que esto puede ser un rompecabezas cuando empiezas, pero con la práctica el hexadecimal empezará a tener mucho sentido para ti. Sin embargo, cuando tienes una ROM abierta en un editor hexadecimal, no necesitas imaginar un cero delante de los números de un solo dígito, ya que el editor lo muestra. Para mostrarte un ejemplo más de cómo se compara el conteo en hexadecimal con el decimal, mira este ejemplo:

---

  
**Comparación entre Decimal y Hexadecimal:**

**Rojo**_:_ _Hexadecimal,_ **Amarillo**_:_ _Decimal_  

---

  
**00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F 10 11 12 13 14 15 16 17 18**  
**0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 21 22 23 24**

**19 1A 1B 1C 1D 1E 1F 20 21 22 23 24 25 26 27 28 29 2A 2B 2C 2D 2E 2F 30 31**  
**25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49**  

---

Ahora, como probablemente no querrás contar número por número para averiguar cuál es el equivalente decimal de un número hexadecimal, tienes una gran ayuda disponible si tienes Windows 95 o superior. La calculadora de Windows puede convertir números decimales normales a hexadecimales. Solo tienes que escribir el número y hacer clic en el círculo que dice Hex. Debería mostrar el número en su equivalente hexadecimal. El increíble editor hexadecimal [Hex Workshop](http://www.bpsoft.com) también tiene una utilidad, el Base Converter, que puede hacer esto por ti.

Así que te estarás preguntando: "¿Qué demonios tiene que ver todo esto con el hacking de ROMs?".

Todo.

Toda la información de la ROM puede representarse en hexadecimal cuando la estás editando, desde los gráficos hasta la música, pasando por cómo se comporta Mega Man cuando salta, etc. Familiarizarse con el hexadecimal te ayudará enormemente con el hacking básico y también te abrirá las puertas a niveles más avanzados del mismo.

En fin, espero que esta explicación tan rudimentaria del hexadecimal te sirva de ayuda. Ahora que tienes una vaga idea de lo que es, empezaremos a entrar en el hacking de ROMs propiamente dicho.

[(Volver al Índice de Contenidos)](#MAIN)

--------------
