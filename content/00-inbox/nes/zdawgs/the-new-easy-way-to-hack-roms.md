## 5. La NUEVA y fácil forma de hackear ROMs  

1. Deberías haber descargado Script Inserter, Script Extractor y TBL Maker del sitio de Jair o de 'Creep. Si ya tienes un archivo TBL hecho, sáltate esta siguiente parte.

---

### a. Uso de TBL Maker

   1. Abre NESticle y busca la tabla de fuentes (el alfabeto). Por lo general, comienza con 0 y llega hasta z. De todos modos, obtén el primer valor hexadecimal.

   2. Abre TBL Maker. Ingresa el tblname que desees y luego presiona "2" para la creación de TBL de texto ASCII simple.

   3. Cuando diga: "Enter value to start with" (Ingresa el valor con el que empezar), ingresa ese primer valor hexadecimal y especifica qué es ese valor hexadecimal. Luego, recorre la lista y, cuando hayas terminado, sal del programa. Boom, el archivo TBL está creado.

---

2. Después de que hayas creado tu archivo TBL, es hora de que extraigas el guion (script).

### b. Uso de Script Extractor

1. Abre Script Extractor.
2. Date una palmadita en la espalda. ¡Acabas de abrir un archivo C++!
3. Debería decir: "Enter name of table file:" - A partir de ahí, ingresa el nombre del archivo TBL. Ingresa el nombre. Luego, debería decir: "File to extract text from:". Ingresa el nombre de la ROM. Después de eso, "File to write to:" -- guárdalo como nombrearchivo.txt, donde nombrearchivo es el nombre que quieras darle. Ahora, dice **"Text block's starting address:"** (Dirección de inicio del bloque de texto): esta es la parte difícil.

4. Usando la multitarea, abre Hexpose. Usa el selector de archivos para seleccionar la ROM y presiona F6 para cargar el archivo TBL. Asegúrate de usar el nombre de archivo completo del TBL.

5. Busca en la ROM para encontrar dónde se encuentra el texto. Busca el texto, esas palabras de la historia, a la derecha. Cuando lo veas, detente y mira directamente a la izquierda de donde comienza el PRIMER CARÁCTER de la PRIMERA LÍNEA del texto.

6. Mira en la esquina inferior izquierda de la pantalla de Hexpose. Debería decir: "Ofs", que es la abreviatura de Offset. Por ejemplo, si dice "0000AAAAA", entonces AAAAA es la dirección del bloque de texto. En Dragon Warrior 3, algo de texto se encuentra en 3940C. Intenta usar el TBL que vino con Hexpose, y la ROM, que deberías tener.

7. Luego debería preguntar cuántos bytes volcar. Yo siempre vuelco 10000, es divertido. Después de que hayas terminado el volcado, puedes editar el archivo.

---

3. Abre el archivo TXT que hiciste del volcado. Debería ser lo suficientemente pequeño como para leerlo con el Bloc de notas (Notepad). De cualquier manera, ábrelo y comienza a editar. Verás el texto, todo lo que tienes que hacer es cambiarlo.

4. Si no me equivoco, ¡PUEDES ESCRIBIR MÁS DE LO QUE MIDE LA ORACIÓN ORIGINAL! ¡WHEE!@#!@# Cuando hayas terminado, guárdalo y luego comenzaremos con Jair's Script Inserter.

### c. Uso de Script Inserter

1. Script Inserter debería comenzar de la misma manera que Extractor; debería preguntar: "Table file:". Pon el nombre de ese archivo TBL.
2. Después de eso, debería decir: "Text file to read from:" allí, pon el nombre del archivo TXT extraído que hiciste con el extractor.
3. "File to insert extracted text to:" lo que debes poner aquí es el ROMName, el nombre de la ROM de la que extrajiste el texto.
4. "Address to start inserting text at:" (Dirección para comenzar a insertar el texto) --- ¿recuerdas esa dirección hexadecimal que obtuvimos? Bueno, insértala allí.
5. "¿Está bien sobrescribir XXXX bytes?" --- ¡CLARO QUE SÍ!@#
6. Después de que eso esté hecho... ¡HAS TERMINADO!@!!#@ ¡¡¡WHOOO!!! ¡VE Y HAZ UN PASTEL! ¡CÓMPRAME UN REFRESCO! ¡VE Y MATA A TU ABUELA!
7. No, no mates realmente a tu abuela. No es algo inteligente de hacer. Simplemente no lo es, podrías lastimarte al ir a la cárcel. Lo sé. Lo hice cuatro veces. No preguntes cómo tengo 4 abuelas.  
 

____________________________________________________________________________  
