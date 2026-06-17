---
version: "1.0"
author: dark_x
---


# Tutorial de hacking de paletas de NES para novatos.
   
   
¡Este es un tutorial sencillo sobre el hacking de paletas  
para roms de NES hecho por MÍ! Dark_X.

![312](file:///tmp/lu5083igv3yu.tmp/lu5083igv3yz_tmp_d586b5b4a21170b3.png)  
   
Si eres perezoso y no quieres leer esto, puedes descargar este  
tutorial aquí -->****[http://www.angelfire.com/retro/dark_x](http://www.angelfire.com/retro/dark_x)** 

## Utilidades necesarias.  
   
- Nesticle (Emulador)  
- un editor hexadecimal (yo prefiero Hexposure)  
- una rom *obvio*  
- un trozo de papel y un lápiz.  
   
## Primeros pasos.
   
En primer lugar, abre nuestra ROM en Nesticle.  
- presiona "ALT + P" cuando llegues al lugar donde quieras cambiar la paleta y luego presiona "F" para ver la paleta.
- Presiona en uno de los cuadros grandes con colores y mira el número HEX, como `0F`. Ese es el color negro en hexadecimal. Anota el número hexadecimal, comienza en el primer cuadro (arriba a la izquierda) y haz 4 filas de todos los cuadros de la paleta (Ejemplo: `0F 0F 28 12`). (NO CAMBIES NINGUNO DE LOS COLORES)
   
## ¡Ahora al cambio real de paleta!
   
Cierra Nesticle y abre tu editor hexadecimal.  
- Tomaré Hexposure como ejemplo (Y TÚ TAMBIÉN DEBERÍAS).  
- abre tu ROM y presiona "F4", escribe los números hexadecimales que anotaste y presiona "ENTER". Ahora el editor te llevará a los números hexadecimales que escribiste.  
- Ahora, cambia los números por los colores que quieras (en HEX, mejor conoce los números de la paleta HEX antes de hacer esto). Ahora guarda y cierra el editor.  
- Abre tu ROM en tu emulador. Ahora los colores deberían ser diferentes.  
- Si los colores no son diferentes, abre Nesticle y hazlo una vez más.  
- Si todavía no son diferentes, la paleta puede ser ASM y no puede cambiarse de esta manera.  
   
Aquí tienes la paleta de Nes en HEX, es muy útil:**

![](file:///tmp/lu5083igv3yu.tmp/lu5083igv3yz_tmp_67e4ff8e2323a690.png)

**Gracias a Kain_Xiorcal por informar de estos errores tipográficos ![](file:///tmp/lu5083igv3yu.tmp/lu5083igv3yz_tmp_d586b5b4a21170b3.png)** **Fin**
