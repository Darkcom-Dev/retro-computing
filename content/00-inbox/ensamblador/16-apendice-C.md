# Apéndice C. Llamadas al Sistema Importantes

Estas son algunas de las llamadas al sistema más importantes para usar cuando se trabaja con Linux. Sin embargo, en la mayoría de los casos, es mejor usar funciones de biblioteca en lugar de llamadas al sistema directas, porque las llamadas al sistema fueron diseñadas para ser minimalistas mientras que las funciones de biblioteca fueron diseñadas para ser fáciles de programar. Para información sobre la biblioteca C de Linux, consulta el manual en http://www.gnu.org/software/libc/manual/

Recuerda que `%eax` contiene los números de las llamadas al sistema, y que los valores de retorno y los códigos de error también se almacenan en `%eax`.

### Tabla C-1. Llamadas al Sistema Importantes de Linux

| %eax | Nombre | %ebx | %ecx | %edx | Notas |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | `exit` | valor de retorno (int) | | | Sale del programa |
| 3 | `read` | descriptor de archivo | inicio del búfer | tamaño del búfer (int) | Lee en el búfer dado |
| 4 | `write` | descriptor de archivo | inicio del búfer | tamaño del búfer (int) | Escribe el búfer en el descriptor de archivo |
| 5 | `open` | nombre de archivo terminado en nulo | lista de opciones | modo de permiso | Abre el archivo dado. Devuelve el descriptor de archivo o un número de error. |
| 6 | `close` | descriptor de archivo | | | Cierra el descriptor de archivo dado |
| 12 | `chdir` | nombre de directorio terminado en nulo | | | Cambia el directorio actual de tu programa. |
| 19 | `lseek` | descriptor de archivo | offset | modo | Reposiciona dónde estás en el archivo dado. El modo (llamado "whence") debe ser 0 para posicionamiento absoluto, y 1 para posicionamiento relativo. |
| 20 | `getpid` | | | | Devuelve el ID del proceso del proceso actual. |
| 39 | `mkdir` | nombre de directorio terminado en nulo | modo de permiso | | Crea el directorio dado. Asume que todos los directorios que llevan a él ya existen. |
| 40 | `rmdir` | nombre de directorio terminado en nulo | | | Elimina el directorio dado. |
| 41 | `dup` | descriptor de archivo | | | Devuelve un nuevo descriptor de archivo que funciona igual que el descriptor de archivo existente. |
| 42 | `pipe` | array de pipe | | | Crea dos descriptores de archivo, donde escribir en uno produce datos para leer en el otro y viceversa. `%ebx` es un puntero a dos palabras de almacenamiento para contener los descriptores de archivo. |
| 45 | `brk` | nuevo límite del sistema | | | Establece el límite del sistema (es decir, el final de la sección de datos). Si el límite del sistema es 0, simplemente devuelve el límite del sistema actual. |
| 54 | `ioctl` | descriptor de archivo | solicitud | argumentos | Esto se usa para establecer parámetros en archivos de dispositivo. Su uso real varía según el tipo de archivo o dispositivo al que hace referencia tu descriptor. |

Un listado más completo de las llamadas al sistema, junto con información adicional, está disponible en http://www.lxhp.in-berlin.de/lhpsyscal.html También puedes obtener más información sobre una llamada al sistema escribiendo `man 2 NOMBRE_LLAMADA` que te devolverá la información sobre la llamada al sistema de la sección 2 del manual de UNIX. Sin embargo, esto se refiere al uso de la llamada al sistema desde el lenguaje de programación C, y puede o no ser directamente útil.

Para información sobre cómo se implementan las llamadas al sistema en Linux, consulta la sección de Linux Kernel 2.4 Internals sobre cómo se implementan las llamadas al sistema en http://www.faqs.org/docs/kernel_2_4/lki-2.html#ss2.11
