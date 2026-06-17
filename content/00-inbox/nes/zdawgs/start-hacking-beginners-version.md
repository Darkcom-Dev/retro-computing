## ¡Comienza a hackear! --- Versión para principiantes  

1.  Abre NESticle.  
2.  Usando la multitarea de Windows, abre mi tabla de fuentes predeterminada de Hexpose.  
3.  Vuelve a NESticle y ve al lugar de la ROM donde haya algo de texto.  
4.  En NESticle, abre las Pattern Tables.  
5.  Busca la tabla de fuentes; la tabla de fuentes son todos los caracteres en inglés.  
6.  Haz clic en el primero de la tabla de fuentes (usualmente 0). Ese es el cero. Los números cuentan, si quieres que lo hagan.
7.  Hay un número y algunas letras en el lado izquierdo de la imagen ampliada. Anota ese número.
8.  Todo lo que sigue debería estar en orden; debería ir: `0 1 2 3 4 5 6 7 8 9 A B C D E F G H I J K L M N O P Q R S T U V W X Y Z`, simplemente escribe el número correspondiente (el número siguiente), y así sucesivamente. Un ejemplo de esto sería si `0=00`, entonces 1 sería 01, 2 sería 02, luego 3 sería 3... después de llegar a 09, cambia a 0A. Luego escribe las letras después de las palabras.

	[!Note]
	siempre termina en 9 o en F. Ya verás a qué me refiero. Aquí hay un ejemplo, si `0=00`, entonces...  
 
```
0=00    6=06     C=OC    I=12   O=18  U=1E   a=24  g=2A  m=30    s=36  
1=01    7=07     D=OD    J=13    P=19  V=1F   b=25  h=2B  n=31    t=37  
2=02    8=08     E=OE    K=14   Q=1A  W=20   c=26  i=2C   o=32    u=38  
3=03    9=09     F=0F     L=15   R=1B  X=21   d=27  j=2D   p=33    v=39  
4=04    A=0A     G=10    M=16   S=1C   Y=22   e=28  k=2E  q=34    w=3A  
5=05    B=0B     H=11    N=17   T=1D   Z=23   f=29  l=2F    r=35    x=3B  
 

y=3C    z=3D  
 
```

Puede haber puntuación extra... con la que simplemente harás lo mismo. No tienes que anotar esto; solo eran ejemplos.

9.  Vuelve al archivo .TBL predeterminado de Astrocreep. *¡ASEGÚRATE DE HACER COPIAS EXTRAS!*
10. Completa la información de la tabla de fuentes.  
11. Luego, cuando termines, guarda el archivo. ¡GUÁRDALO CON EL MISMO NOMBRE QUE LA ROM!  
12. Abre HexPose.  
13. Cuando te pregunte qué cosa quieres cargar, selecciona la ROM de la cual tienes la información.  
14. Luego, presiona F6 y escribe el nombre del archivo con el que se llama la tabla de fuentes.  
15. ¡Voilá! Si te desplazas hacia abajo, debería haber texto real en inglés.  

Hackea... pero recuerda esto: ¡ASEGÚRATE DE QUE LO QUE HACKEES NO SEA MÁS LARGO DE LO QUE HABÍA EN LA ROM! Eso se explicará más adelante, y también cómo evitarlo.  

16. Si quieres intentar que el texto sea más largo de lo que hay, usa esto... no es un hack de ASM y, por lo tanto, no es muy confiable. Yo mismo solo he tenido un 10% de éxito.  
17. En HEXPOSE, ve al final de la ROM.  
18. Presiona la tecla INSERT.  
19. Ingresa 1024x16 bytes. No tengo una calculadora conmigo ahora mismo, lo siento. No olvides que esta es la versión 01 de mi documento.  
20. Vuelve a donde te gustaría añadir más texto. No importa cuántas letras más sean, presiona insertar de nuevo, luego pon el texto; después, vuelve al final de la ROM y borra la cantidad de bytes que hayas añadido.  
21. Guárdalo en la ROM y asegúrate de hacer copias de la ROM; puede fallar. Sal de HexPose y NESticle.  
22. Luego, vuelve a abrir NESticle.  
23. Carga la ROM; asegúrate de arreglar la cabecera (header) entrando en ROM Header y sumando 1 a la rom de 16k.  
24. Puede que funcione o no.  
25. ¡Disfruta!  
 

____________________________________________________________________________  
