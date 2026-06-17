---
version: "1.2"
author: InVerse
fecha: 1999-04-13T00:00:00
---


---
# Construcción de Tablas para Hacking de ROMs de NES

## Introducción

El propósito de este documento es sencillo: enseñar a las personas nuevas en el hacking/traducción de ROMs cómo crear tablas para usarlas con ROMs de NES.

Si ya tienes experiencia en esto, no hay razón para que sigas leyendo, pero últimamente varias personas han estado entrando en canales de IRC pidiendo ayuda para crear tablas, así que decidí escribir mi propio archivo sobre el tema.

No reclamo haber creado ninguna de las técnicas aquí descritas, simplemente transmito la información a quien la necesite.

## Descargo de Responsabilidad

Si las técnicas descritas aquí fallan por cualquier razón, te devolveré todo tu dinero.
Como este archivo es gratuito, supongo que no recuperarás mucho. ;)
Si las técnicas funcionan, hazle un favor a todos y crea un buen hack o traduce un buen juego. ;)

## Herramientas
- Nesticle.
- ROM de NES.
- Bloc de notas.
(Y sí, una computadora también ayuda.)

## Comenzando
Primero debes abrir la ROM para la que quieres crear una tabla en Nesticle.
Para esta lección usaré *The Legend of Zelda*, así que si quieres seguir el ejemplo, te sugiero usar esa ROM.

Abre la ROM y ponla en pausa presionando **ALT-P**. Presiona **F2** para abrir el *Pattern Editor* de Nesticle.
Ahora debes localizar la tabla de fuentes. ¿Ves el alfabeto y los números en la esquina superior izquierda del *Pattern Editor*? Esa es la tabla de fuentes.

En algunas ROMs no será tan fácil encontrarla. A veces hay que dejar correr el juego un poco para que aparezca; a veces (especialmente en juegos más viejos) puede que no haya una tabla completa; otras veces puede haber más de una tabla.
Por lo general, tendrás suerte (como con Zelda) y estará completa desde el inicio.

Una vez localizada la tabla, haz clic en el primer carácter que quieras incluir.
Se abrirá una ventana pequeña con el carácter. Arriba de esa ventana verás la dirección hexadecimal del carácter en la ROM.
Ahora escribe el carácter y su dirección en el Bloc de notas (o en papel si no quieres estar cambiando de ventana).
  
Cuando tengas la dirección de todos los caracteres (no olvides los signos de puntuación), es hora de crear la tabla.

## Creando una Tabla

Cuando este documento fue escrito, había dos formatos distintos de tabla, pero con la versión 0.40 de Hexposure, SnowBro adaptó Hexposure para usar el mismo formato que Thingy.

Para hacer una tabla, simplemente escribe el valor hexadecimal seguido de un signo igual (`=`) y el carácter correspondiente.

Usando el ejemplo de *The Legend of Zelda* donde el valor hexadecimal de `A` es `0A`, escribirías:

```
0A=A
```

Haz esto para todos los caracteres que quieras incluir en tu tabla, y guarda el archivo como `<juego>.tbl`, donde `<juego>` es idéntico al nombre de tu ROM (por ejemplo, `zelda.tbl` para `zelda.nes`).

Si no los nombras igual, seguirá funcionando, pero Hexposure abrirá automáticamente la tabla si encuentra una que coincida en nombre con la ROM, lo cual ahorra unos segundos.

También puedes tener más de un valor por línea para representar palabras completas o usar DTE (*Dual Tile Encoding*).

Esto es más avanzado y no será cubierto aquí.

Un ejemplo simple usando Zelda:

```
230E150D0A=ZELDA
```

## Tabla de Ejemplo
Aquí tienes una tabla de ejemplo de *The Legend of Zelda*.

Puedes copiarla y pegarla si quieres, pero te recomiendo intentar hacerla tú mismo primero.

```
00=0
01=1
02=2
03=3
04=4
05=5
06=6
07=7
08=8
09=9
0A=A
0B=B
0C=C
0D=D
0E=E
0F=F
10=G
11=H
12=I
13=J
14=K
15=L
16=M
17=N
18=O
19=P
1A=Q
1B=R
1C=S
1D=T
1E=U
1F=V
20=W
21=X
22=Y
23=Z
28=,
29=!
2A='
2B=&
2C=.
2D="
2E=?
2F=-
```

*(Recuerda: las líneas de guiones son divisores, ¡no las incluyas en tu tabla!)*

## Conclusión
Este fue nuestro pequeño tutorial para crear archivos de tabla.

Si no entendiste el concepto, probablemente no avances mucho en la escena del hacking/traducción de ROMs.

Este documento no enseña a hackear/traducir ROMs; solo cubre una técnica básica.

Lee otros documentos (especialmente los README de las herramientas) y pregunta en IRC o foros.

Zophar's Domain ([www.zophar.net](http://www.zophar.net)) tiene un foro de ROM hacking, y el canal #romhack en EFnet también es útil (aunque espera que te flameen: la proporción de idiotas a gente inteligente es de 5:1).

Eso es todo. Espero que haya sido de ayuda.

Si encuentras errores, avísame.

No me escribas preguntando temas avanzados: hay muchos documentos que cubren casi todo.

## Agradecimientos Especiales

- **SnowBro** – Autor de Hexposure y otros grandes programas.
  [http://home.sol.no/~kenhanse/nes/index.htm](http://home.sol.no/~kenhanse/nes/index.htm)
- **Necrosaro** – Autor de Thingy.
  [http://members.aol.com/sabindude/](http://members.aol.com/sabindude/)
- **Patrikus** – Por enseñarme originalmente a hacer archivos .tbl.
- **Chojin** – Uno de los pocos en la escena de emulación a quien no quiero sodomizar con un taco de billar.
- **_Bnu** – El único otro en la escena de emulación que tampoco quiero sodomizar con un taco de billar.
- **Taskforce** – Por sacarme de un humor homicida enviándome el ROM de *Sanrio Time Net Future*.
- **Lina`chan** – Porque sé que se quejaría si no la menciono. ;) (es broma)

## Historial
- **11/01/99** – v1.0 – Lanzamiento inicial
- **13/04/99** – v1.5 – Actualizado para Hexposure 0.40, info de contacto

## Información de Contacto
No soy un gran hacker de ROMs.

Escribí este documento porque estoy harto de ver a gente preguntar cómo hacer archivos de tabla. Ahora simplemente les enviaré esto.

Si tienes preguntas sobre hacking de ROMs, **no me escribas**.
Si ves algún error en el documento, mándame un correo a **inverse@pigtails.net** y lo corregiré.

La versión más reciente de este archivo siempre estará en mi sitio:
**Suicidal Translations**: [http://www.pigtails.net/ST](http://www.pigtails.net/ST)

*-InVerseü*
