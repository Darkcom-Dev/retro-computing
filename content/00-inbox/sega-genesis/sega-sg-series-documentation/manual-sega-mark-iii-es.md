# Manual de Referencia de Software para la Consola SEGA Mark III

---

## Tabla de Contenidos

**La Consola de Juegos Sega Mark III** 

**[[la-cpu]]**
[[procesador-de-visualizacion-de-video-vdp]]
- organización de la ram de video
- color
- sistema de fondo
- bits de inhibición de desplazamiento
*   Sprites
*   Tabla de Atributos de Sprites 
*   [[organizacion-de-tabla-de-atributos-de-sprites]] 
*   [[prioridad-visualizacion-sprite-fondo]]
*   [[ram-de-color]]
*   [[patrones-de-caracteres]] 
*   [[ejemplo-de-patron-de-caracter]] 
*   [[registros-del-vdp]]
    
*   Actualizaciones de Registros del VDP 
*   [[descripcion-de-los-registros-del-vdp]]
    

**[[sistema-gestion-de-memoria]]**


**[[puertos-de-entrada-salida-es]]** 


**[[generador-de-sonido-programable-psg]]**


**[[apendice-a-registros-del-vdp]]**
**[[apendice-b-puertos-de-entrada-salida]]**
**[[apendice-c-codigo-de-ejemplo-para-leer-pistola]]**
**[[apendice-d-codigo-de-ejemplo-para-leer-trackball]]** 

**[[notas-del-desarrollador]]**


**[[ilustraciones-de-apoyo]]**


---

## La Consola de Juegos Sega Mark III

Este manual describe el hardware de la consola de juegos SEGA Mark III. El manual se divide en cinco secciones:

*   La CPU.
*   El Procesador de Visualización de Video (VDP).
*   El Sistema de Gestión de Memoria.
*   El Sistema de Entrada/Salida.
*   El Generador de Sonido Programable (PSG).

Cada sección comienza con una descripción general, seguida de una descripción detallada de cada bit de control. Varios diagramas pictóricos al final del manual vinculan todos los conceptos descritos. Si desea ver cómo encaja una característica particular en el sistema total, consulte estas páginas finales.

### ENCABEZADOS

Los temas importantes dentro de cada sección están marcados como el anterior con un "encabezado". Esto permite un escaneo rápido de los elementos específicos de interés.

### REFERENCIAS FUTURAS

En algunas secciones es imposible describir el sistema sin hacer referencias a detalles técnicos descritos más adelante en el manual. Por ejemplo, encontrará referencias a "Interrupciones de Sprite" en la sección de la CPU, antes de que se discuta la noción de un sprite en la sección del VDP.

Por esta razón, podría resultarle útil hojear las primeras partes de cada sección para tener una idea del sistema total antes de sumergirse en las descripciones detalladas.

El sistema se denominará en lo sucesivo como la "Mk3".
