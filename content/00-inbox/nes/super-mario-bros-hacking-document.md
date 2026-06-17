---
version: "1.0"
author: dahrk daiz
year: 2003-01-01
---
# Documento de Hacking de Super Mario Bros. 1.0

Todos los datos encontrados/descifrados por Dahrk Daiz (DahrkDaiz@hotmail.com)
Queda prohibida la reproducción de este documento.
Copyright (C) 2003
## Cabecera de Nivel (Level Header)

Los dos primeros bytes de un nivel son su cabecera de nivel. La cabecera de nivel se desglosa así:

```
(   byte 1    )   (   byte 2    )
t t a p p b b b - c c s s g g g g
=================================|===================|
|_| | |_| |_|_|   |_| |_| |_|_|_|| tipo de suelo/    |
 |  |  |    |      |   |     |___| bloque            |
 |  |  |    |      |   |         |===================|
 |  |  |    |      |   |_________| tipo de           |
 |  |  |    |      |             | paisaje           |
 |  |  |    |      |             |===================|
 |  |  |    |      |_____________| complemento       |
 |  |  |    |                    |                   |
 |  |  |    |                    |===================|
 |  |  |    |____________________| tipo de fondo/    |
 |  |  |                         | estación          |
 |  |  |                         |===================|
 |  |  |_________________________| posición de       |
 |  |                            | inicio            |
 |  |                            |===================|
 |  |____________________________| camino automático |
 |                               | encendido/apagado |
 |                               |===================|
 |_______________________________| tiempo            |
                                 |                   |
=====================================================|
```
tipo de suelo/bloque: define el tipo de estructura del suelo para un nivel, esto puede ser qué tan alto es el suelo y qué tan bajo llega el techo (si hay un techo).

tipo de paisaje: determina qué se pondrá en las nubes de fondo

```
00 = ninguno
01 = nubes
10 = montañas/colinas
11 = valla/árboles
```
complemento: afecta un poco a la paleta, pero principalmente determina cómo se ven/actúan los "árboles"

```
00 = árboles
01 = champiñones
10 = torretas de Bullet Bill (también funcionan)
11 = el mundo de nubes (el suelo es una nube)
```

tipo de fondo/estación: este es principalmente un atributo de paleta, pero sirve para otro propósito.

Cuando este atributo es menor que 100 `(0x04)`, el fondo se ve afectado; cualquier valor superior afecta a la paleta.

```
000 = día (paleta normal)
001 = fondo bajo el agua
010 = fondo de muro de castillo
011 = fondo sobre el agua
100 = noche
101 = día/nieve
110 = noche/nieve
111 = blanco y negro (castillo)
```


posición de inicio: determina dónde comienza Mario un nivel.

```
00 = cayendo del cielo
01 = comenzando en el suelo
10 = cayendo del cielo
11 = mitad de la pantalla
```

camino automático (autowalk): determina si Mario comienza el nivel caminando automáticamente (sucede en 1-2, 2-2, 4-2 y 7-2 durante la secuencia de la tubería)

```
0 = no
1 = sí
```

tiempo: determina el tiempo inicial

```
00 = 0
01 = 400
10 = 300
11 = 200
```

## Formato de Nivel

Los datos del nivel son un poco extraños. Cada objeto ocupa 2 bytes de la siguiente manera:

```
(   byte 1    )   (   byte 2    )
x x x x y y y y - p o o o o o o o
=================================================
|_|_|_| |_|_|_|   | |_|_|_|_|_|_| | objeto      |
   |       |      |       |_______|             |
   |       |      |               |=============|
   |       |      |_______________| flag de     |
   |       |                      | nueva página|
   |       |                      |=============|
   |       |______________________| posición y  |
   |                              |             |
   |                              |=============|
   |______________________________| posición x  |
                                  |             |
================================================|
```

objeto: esto es precisamente eso, el objeto que aparecerá en la pantalla del nivel; algunos valores de objeto ponen varios del mismo objeto en la pantalla.

para las tuberías `(0x70 - 0x7F)`, la primera mitad `(0x70 - 0x77)` es una tubería en la que no puedes entrar, pero la segunda mitad `(0x78 - 0x7F)` son accesibles.

flag de nueva página: determina si el objeto es el primero en aparecer en una nueva página (pantalla).

posición y*: la posición y de donde aparecerá un objeto (por un múltiplo de 16 píxeles)

posición x: la posición x de donde aparecerá un objeto (por un múltiplo de 16 píxeles)

El formato del nivel se divide en "páginas", de modo que solo una pequeña cantidad de datos del nivel está en la RAM en cualquier momento dado. El bit del flag de nueva página le indica al juego que el objeto en el que está establecido es el primer objeto de la página.

Luego, cada objeto posterior aparecerá en la misma página hasta que se encuentre un nuevo bit de página. Sin embargo, hay límites para esto. Si hay 3 objetos en la misma posición x para la misma página, cada objeto que siga no aparecerá.

La posición Y sirve para un segundo propósito: si la posición y es superior a 1011 (0x0B), entonces el conjunto de objetos cambia.

Si la posición y es `0x0C`, se utiliza un nuevo conjunto de objetos (pero la posición Y de estos nuevos conjuntos de objetos no se puede cambiar).

Cada incremento de la posición Y es un nuevo conjunto de objetos.

El final de un nivel se representa por el byte XY siendo `0xFD`


## Formato de Enemigo

Los datos de los enemigos no tienen cabecera y funcionan casi de la misma manera que el formato de los datos de nivel.

```
(   byte 1    )   (   byte 2    )
x x x x y y y y - p e e e e e e e
=================================================
|_|_|_| |_|_|_|   | |_|_|_|_|_|_| | enemigo     |
   |       |      |       |_______|             |
   |       |      |               |=============|
   |       |      |_______________| flag de     |
   |       |                      | nueva página|
   |       |                      |=============|
   |       |______________________| posición y  |
   |                              |             |
   |                              |=============|
   |______________________________| posición x  |
                                  |             |
================================================|
```

enemigo: esto es precisamente eso, el enemigo que aparecerá en la pantalla del nivel; algunos valores de enemigo ponen varios de los mismos enemigos en la pantalla.

flag de nueva página: determina si el enemigo es el primero en aparecer en una nueva página (pantalla).

posición y*: la posición y de donde aparecerá un enemigo (por un múltiplo de 16 píxeles). Esta posición NO PUEDE ser `0x0E`. Eso se explica más adelante.

posición x: la posición x de donde aparecerá un enemigo (por un múltiplo de 16 píxeles)

El formato de enemigo se divide en "páginas", de modo que solo una pequeña cantidad de datos de enemigos está en la RAM en cualquier momento dado.

El bit del flag de nueva página le indica al juego que el enemigo en el que está establecido es el primer enemigo de la página. Luego, cada enemigo posterior aparecerá en la misma página hasta que se encuentre un nuevo bit de página.

*La posición Y sirve para un segundo propósito: si la posición está en su máximo (0x0F), eso le indica al juego que salte un cierto número de páginas antes de leer más datos de enemigos; sin embargo, el número de páginas a saltar aún no está muy claro.*

Los datos de los enemigos para un nivel terminan cuando el byte XY es `FF`.


## Punteros de Tubería (Pipe Pointers)

Los punteros de tubería funcionan de una manera muy extraña. En primer lugar, no son punteros verdaderos; más bien, apuntan a mapas (niveles) y a la página para ese mapa.

Cuando se coloca un puntero de tubería en un nivel, la página en la que está y cada página posterior hace que las tuberías accesibles vayan a esa página/nivel.

Lo único que puede cambiarlo es otro puntero de tubería, que cancela el anterior y se activa para cada página posterior. Los punteros de tubería son diferentes en el sentido de que ocupan 3 bytes, en lugar de 2.

```
x x x x y y y y - n m m m m m m m - w w w p p p p p
====================================================================|
|_|_|_| |_|_|_|   | |_|_|_|_|_|_|   |_|_| |_|_|_|_| | puntero de    |
   |       |      |       |           |       |_____| página        |
   |       |      |       |           |             |===============|
   |       |      |       |           |_____________| mundo         |
   |       |      |       |                         | activo        |
   |       |      |       |                         |===============|
   |       |      |       |_________________________| puntero de    |
   |       |      |                                 | mapa          |
   |       |      |                                 |===============|
   |       |      |_________________________________| flag de       |
   |       |                                        | nueva página  |
   |       |                                        |===============|
   |       |________________________________________| posición y    |
   |                                                | (siempre 0x0E)|
   |                                                |===============|
   |________________________________________________| posición x    |
                                                    |               |
====================================================================|
```

puntero de página: la página en la que aparecerás después de entrar en la tubería. La posición real es aproximadamente 2 bloques por encima del suelo y 2 bloques a la derecha de la pantalla izquierda.

mundo activo: esto determina el mundo en el que funciona el puntero de tubería. En SMB1, el valor `0x00` es el mundo 1, por lo que `0x07` es el mundo 8. Si un puntero de tubería aparece en el mundo 3, pero el valor de mundo activo es para el mundo 6, entonces el puntero de tubería se ignora.

puntero de mapa: el número de mapa del nivel que se cargará; ten en cuenta que los mapas corresponden a tipos de nivel y punteros reales, que se explicarán más adelante.

flag de nueva página: el mismo flag que se usa para objetos y enemigos que dice que es el primer enemigo en la página.

posición y: siempre `0x0E`

posición x: no importa para los punteros de tubería


## Mapas

Los niveles no se cargan directamente desde punteros como en muchos juegos. En su lugar, se cargan desde valores de mapa. Un valor de mapa corresponde a una tabla de punteros de nivel, una tabla de punteros de enemigos y un "tipo" de nivel. Los mapas `0x00 - 0x1F` son tipos de agua, `0x20 - 0x3F` son de pradera, `0x40 - 0x5F` son subterráneos y `0x60 - 0x7F` son tipos de castillo. Después, el patrón se repite (es decir, `0x80 - 0x9F` son de agua, etc.).

La tabla de punteros que utilizan los mapas para cargar un nivel se almacena en dos piezas separadas.

El byte bajo de la tabla de punteros comienza en `0x9D2C` en la ROM, el byte alto comienza en `0x9D4E`.

Esto también funciona con la tabla de punteros de enemigos. El byte bajo comienza en `0x9CE4` y el byte alto comienza en `0x9D06`. Sin embargo, desconozco a qué parte de la tabla apuntan realmente los mapas en este momento. Generalmente uso un depurador para encontrar las direcciones reales de los niveles.

También hay una tabla de mapas que determina el orden en que se cargan los mapas. Esta comienza en `0x1CCC` en la ROM, con `0x25` siendo 1-1, `0x29` siendo 1-2 (intro) y así sucesivamente. En los datos del nivel, hay un objeto de una tubería en forma de L invertida. Esta tubería es de un solo tamaño. Esta tubería es especial en el sentido de que, al entrar, aumenta el puntero de la tabla de mapas en 1. El puntero de mapa comienza así:

`0x1CCC: 25 29 C0`

`29` es la intro del nivel 1-2; en esa intro hay una tubería por la que entra Mario; es esa tubería en L invertida la que hace que el puntero avance 1 byte. Por lo tanto, entrar en la tubería cambia el mapa actual a `C0` (1-2 subterráneo).

	[!Note]
	Solo 1 tubería en L invertida no redimensionable hace esto; la redimensionable actúa como una tubería normal y obedece a los punteros de tubería.

Justo antes de la tabla de mapas hay una tabla de offsets de Mundo (World offsets). Esta tabla determina cuántos bytes de la tabla hay que saltar al cargar un nivel.

Lo que hace el juego es cargar un valor del offset del mapa del mundo y luego sumar la etapa actual a eso (0, 1, 2, etc.). Esto comienza en `0x1CC4` con `00 05 0A`, etc.

Observa que el offset del mundo 2 es `05`. Recuerda que la tubería en L invertida aumentará el número de etapa, por lo que, técnicamente, el Mundo 1 tiene 5 etapas, al igual que el Mundo 2, el Mundo 4 y el Mundo 7.
