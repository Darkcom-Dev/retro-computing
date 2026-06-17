# Apéndice A. Programación de GUI

## Introducción a la Programación de GUI

El propósito de este apéndice no es enseñarte cómo hacer Interfaces Gráficas de Usuario (GUI). Simplemente está destinado a mostrar cómo escribir aplicaciones gráficas es lo mismo que escribir otras aplicaciones, solo usando una biblioteca adicional para manejar las partes gráficas. Como programador, necesitas acostumbrarte a aprender nuevas bibliotecas. La mayor parte de tu tiempo se gastará pasando datos de una biblioteca a otra.

## Las Bibliotecas GNOME

El proyecto GNOME es uno de varios proyectos para proporcionar un escritorio completo a los usuarios de Linux. El proyecto GNOME incluye un panel para contener lanzadores de aplicaciones y mini-aplicaciones llamadas applets, varias aplicaciones estándar para hacer cosas como gestión de archivos, gestión de sesiones y configuración, y una API para crear aplicaciones que encajen con la forma en que funciona el resto del sistema.

Una cosa a notar sobre las bibliotecas GNOME es que constantemente crean y te dan punteros a grandes estructuras de datos, pero nunca necesitas saber cómo están organizadas en la memoria. Toda la manipulación de las estructuras de datos de la GUI se hace enteramente a través de llamadas a funciones. Esta es una característica del buen diseño de bibliotecas. Las bibliotecas cambian de versión a versión, y también lo hacen los datos que contiene cada estructura de datos. Si tuvieras que acceder y manipular esos datos tú mismo, entonces cuando la biblioteca se actualice tendrías que modificar tus programas para trabajar con la nueva biblioteca, o al menos recompilarlos. Cuando accedes a los datos a través de funciones, las funciones se encargan de saber dónde está cada pieza de datos dentro de la estructura. Los punteros que recibes de la biblioteca son **opacos** - no necesitas saber específicamente cómo es la estructura a la que apuntan, solo necesitas conocer las funciones que la manipularán adecuadamente. Al diseñar bibliotecas, incluso para uso dentro de un solo programa, esta es una buena práctica a tener en cuenta.

Este capítulo no entrará en detalles sobre cómo funciona GNOME. Si te gustaría saber más, visita el sitio web de desarrollo de GNOME en http://developer.gnome.org/. Este sitio contiene tutoriales, listas de correo, documentación de API, y todo lo demás que necesitas para comenzar a programar en el entorno GNOME.

## Un Programa Simple de GNOME en Varios Lenguajes

Este programa simplemente mostrará una Ventana que tiene un botón para salir de la aplicación. Cuando se hace clic en ese botón, te preguntará si estás seguro, y si haces clic en sí, cerrará la aplicación. Para ejecutar este programa, escribe lo siguiente como `gnome-example.s`:

```assembly
#PROPÓSITO: Este programa está destinado a ser un ejemplo
# de cómo se ven los programas de GUI escritos
# con las bibliotecas GNOME
#
#ENTRADA: El usuario solo puede hacer clic en el botón "Salir"
# o cerrar la ventana
#
#SALIDA: La aplicación se cerrará
#
#PROCESO: Si el usuario hace clic en el botón "Salir",
# el programa mostrará un diálogo preguntando
# si está seguro. Si hace clic en Sí,
# cerrará la aplicación. De lo contrario
# continuará ejecutándose
#

.section .data

###Definiciones de GNOME - Estas se encontraron en los
# archivos de cabecera de GNOME para el lenguaje C
# y se convirtieron a sus equivalentes
# en ensamblador

#Nombres de Botones GNOME
GNOME_STOCK_BUTTON_YES:
.ascii "Button_Yes\0"
GNOME_STOCK_BUTTON_NO:
.ascii "Button_No\0"

#Tipos de MessageBox de Gnome
GNOME_MESSAGE_BOX_QUESTION:
.ascii "question\0"

#Definición estándar de NULL
.equ NULL, 0

#Definiciones de señales GNOME
signal_destroy:
.ascii "destroy\0"
signal_delete_event:
.ascii "delete_event\0"
signal_clicked:
.ascii "clicked\0"

###Definiciones específicas de la aplicación

#Información de la aplicación
app_id:
.ascii "gnome-example\0"
app_version:
.ascii "1.000\0"
app_title:
.ascii "Gnome Example Program\0"

#Texto para Botones y ventanas
button_quit_text:
.ascii "Quiero Salir del Programa Ejemplo GNOME\0"
quit_question:
.ascii "¿Estás seguro de que quieres salir?\0"

.section .bss

#Variables para guardar los widgets creados
.equ WORD_SIZE, 4
.lcomm appPtr, WORD_SIZE
.lcomm btnQuit, WORD_SIZE

.section .text

.globl main
.type main,@function
main:
pushl %ebp
movl %esp, %ebp

#Inicializar bibliotecas GNOME
pushl 12(%ebp) #argv
pushl 8(%ebp)  #argc
pushl $app_version
pushl $app_id
call gnome_init
addl $16, %esp #recuperar la pila

#Crear nueva ventana de aplicación
pushl $app_title #Título de la ventana
pushl $app_id    #ID de la aplicación
call gnome_app_new
addl $8, %esp #recuperar la pila
movl %eax, appPtr #guardar el puntero de la ventana

#Crear nuevo botón
pushl $button_quit_text #texto del botón
call gtk_button_new_with_label
addl $4, %esp #recuperar la pila
movl %eax, btnQuit #guardar el puntero del botón

#Hacer que el botón se muestre dentro de la ventana de la aplicación
pushl btnQuit
pushl appPtr
call gnome_app_set_contents
addl $8, %esp

#Hace que el botón se muestre (solo después de que su ventana
#se muestre, sin embargo)
pushl btnQuit
call gtk_widget_show
addl $4, %esp

#Hace que la ventana de la aplicación se muestre
pushl appPtr
call gtk_widget_show
addl $4, %esp

#Hacer que GNOME llame a nuestra función delete_handler
#cuando ocurra un evento "delete"
pushl $NULL #datos extra para pasar a nuestra
            #función (no usamos ninguno)
pushl $delete_handler #dirección de la función a llamar
pushl $signal_delete_event #nombre de la señal
pushl appPtr #widget para escuchar eventos
call gtk_signal_connect
addl $16, %esp #recuperar pila

#Hacer que GNOME llame a nuestra función destroy_handler
#cuando ocurra un evento "destroy"
pushl $NULL #datos extra para pasar a nuestra
            #función (no usamos ninguno)
pushl $destroy_handler #dirección de la función a llamar
pushl $signal_destroy #nombre de la señal
pushl appPtr #widget para escuchar eventos
call gtk_signal_connect
addl $16, %esp #recuperar pila

#Hacer que GNOME llame a nuestra función click_handler
#cuando ocurra un evento "click". Observa que
#las señales anteriores estaban escuchando en la
#ventana de la aplicación, mientras que esta solo
#escucha en el botón
pushl $NULL
pushl $click_handler
pushl $signal_clicked
pushl btnQuit
call gtk_signal_connect
addl $16, %esp

#Transferir el control a GNOME. Todo lo que
#sucede de aquí en adelante es en reacción a eventos
#del usuario, que llaman a los manejadores de señales. Esta función
#main solo configura la ventana principal y conecta
#los manejadores de señales, y los manejadores de señales se
#encargan del resto
call gtk_main

#Después de que el programa termine, salir
movl $0, %eax
leave
ret

#Un evento "destroy" ocurre cuando el widget está siendo
#eliminado. En este caso, cuando la ventana de la aplicación
#está siendo eliminada, simplemente queremos que el bucle de eventos
#termine
destroy_handler:
pushl %ebp
movl %esp, %ebp

#Esto hace que gtk salga de su bucle de eventos
#tan pronto como pueda.
call gtk_main_quit

movl $0, %eax
leave
ret

#Un evento "delete" ocurre cuando se hace clic en la ventana de la
#aplicación en la "x" que normalmente usas para
#cerrar una ventana
delete_handler:
movl $1, %eax
ret

#Un evento "click" ocurre cuando se hace clic en el widget
click_handler:
pushl %ebp
movl %esp, %ebp

#Crear el diálogo "¿Estás seguro?"
pushl $NULL #Fin de botones
pushl $GNOME_STOCK_BUTTON_NO #Botón 1
pushl $GNOME_STOCK_BUTTON_YES #Botón 0
pushl $GNOME_MESSAGE_BOX_QUESTION #Tipo de diálogo
pushl $quit_question #Mensaje del diálogo
call gnome_message_box_new
addl $16, %esp #recuperar pila

#%eax ahora tiene el puntero a la ventana de diálogo

#Establecer Modal a 1 evita cualquier otra interacción
#del usuario mientras se muestra el diálogo
pushl $1
pushl %eax
call gtk_window_set_modal
popl %eax
addl $4, %esp

#Ahora mostramos el diálogo
pushl %eax
call gtk_widget_show
popl %eax

#Esto configura todos los manejadores de señales necesarios
#para simplemente mostrar el diálogo, cerrarlo cuando
#se hace clic en uno de los botones, y devolver el
#número del botón en el que el usuario hizo clic.
#El número del botón se basa en el orden en que los botones
#se empujaron en la función gnome_message_box_new
pushl %eax
call gnome_dialog_run_and_close
addl $4, %esp

#El botón 0 es el botón Sí. Si este es el
#botón en el que hicieron clic, decirle a GNOME que termine
#su bucle de eventos. De lo contrario, no hacer nada
cmpl $0, %eax
jne click_handler_end

call gtk_main_quit

click_handler_end:
leave
ret
```

Para construir esta aplicación, ejecuta los siguientes comandos:

```bash
as gnome-example.s -o gnome-example.o
gcc gnome-example.o `gnome-config --libs gnomeui` \
-o gnome-example
```

Luego escribe `./gnome-example` para ejecutarlo.

Este programa, como la mayoría de los programas de GUI, hace un uso intensivo de pasar punteros a funciones como parámetros. En este programa creas widgets con las funciones de GNOME y luego configuras funciones para que sean llamadas cuando ocurren ciertos eventos. Estas funciones se llaman **funciones de callback (callback functions)**. Todo el procesamiento de eventos es manejado por la función `gtk_main`, por lo que no tienes que preocuparte por cómo se procesan los eventos. Todo lo que tienes que hacer es tener callbacks configurados para esperarlos.

Aquí hay una breve descripción de todas las funciones de GNOME que se usaron en este programa:

### `gnome_init`
Toma los argumentos de línea de comandos, el conteo de argumentos, el id de la aplicación y la versión de la aplicación e inicializa las bibliotecas GNOME.

### `gnome_app_new`
Crea una nueva ventana de aplicación, y devuelve un puntero a ella. Toma el id de la aplicación y el título de la ventana como argumentos.

### `gtk_button_new_with_label`
Crea un nuevo botón y devuelve un puntero a él. Toma un argumento - el texto que está en el botón.

### `gnome_app_set_contents`
Esto toma un puntero a la ventana de la aplicación gnome y cualquier widget que quieras (un botón en este caso) y hace que el widget sea el contenido de la ventana de la aplicación.

### `gtk_widget_show`
Esto debe llamarse en cada widget creado (ventana de aplicación, botones, cuadros de entrada de texto, etc.) para que sean visibles. Sin embargo, para que un widget dado sea visible, todos sus padres deben ser visibles también.

### `gtk_signal_connect`
Esta es la función que conecta los widgets y sus funciones de callback de manejo de señales. Esta función toma el puntero del widget, el nombre de la señal, la función de callback y un puntero de datos extra. Después de que se llama a esta función, cada vez que se activa el evento dado, se llamará al callback con el widget que produjo la señal y el puntero de datos extra. En esta aplicación, no usamos el puntero de datos extra, así que simplemente lo establecemos a `NULL`, que es 0.

### `gtk_main`
Esta función hace que GNOME entre en su bucle principal. Para facilitar la programación de aplicaciones, GNOME maneja el bucle principal del programa por nosotros. GNOME verificará los eventos y llamará a las funciones de callback apropiadas cuando ocurran. Esta función continuará procesando eventos hasta que se llame a `gtk_main_quit` por un manejador de señales.

### `gtk_main_quit`
Esta función hace que GNOME salga de su bucle principal en la primera oportunidad.

### `gnome_message_box_new`
Esta función crea una ventana de diálogo que contiene una pregunta y botones de respuesta. Toma como parámetros el mensaje a mostrar, el tipo de mensaje (advertencia, pregunta, etc.), y una lista de botones a mostrar. El parámetro final debe ser `NULL` para indicar que no hay más botones que mostrar.

### `gtk_window_set_modal`
Esta función hace que la ventana dada sea una ventana modal. En programación de GUI, una ventana modal es aquella que evita el procesamiento de eventos en otras ventanas hasta que esa ventana se cierra. Esto se usa a menudo con ventanas de Diálogo.

### `gnome_dialog_run_and_close`
Esta función toma un puntero de diálogo (el puntero devuelto por `gnome_message_box_new` puede usarse aquí) y configurará todos los manejadores de señales apropiados para que se ejecute hasta que se presione un botón. En ese momento cerrará el diálogo y te devolverá qué botón se presionó. El número del botón se refiere al orden en que los botones se configuraron en `gnome_message_box_new`.

El siguiente es el mismo programa escrito en el lenguaje C. Escríbelo como `gnome-example-c.c`:

```c
/* PROPÓSITO: Este programa está destinado a ser un ejemplo
   de cómo se ven los programas de GUI escritos
   con las bibliotecas GNOME
*/
#include <gnome.h>

/* Definiciones del programa */
#define MY_APP_TITLE "Gnome Example Program"
#define MY_APP_ID "gnome-example"
#define MY_APP_VERSION "1.000"
#define MY_BUTTON_TEXT "Quiero Salir del Programa Ejemplo"
#define MY_QUIT_QUESTION "¿Estás seguro de que quieres salir?"

/* Debe declarar las funciones antes de usarlas */
int destroy_handler(gpointer window, GdkEventAny *e, gpointer data);
int delete_handler(gpointer window, GdkEventAny *e, gpointer data);
int click_handler(gpointer window, GdkEventAny *e, gpointer data);

int main(int argc, char **argv)
{
    gpointer appPtr; /* ventana de la aplicación */
    gpointer btnQuit; /* botón de salir */

    /* Inicializar bibliotecas GNOME */
    gnome_init(MY_APP_ID, MY_APP_VERSION, argc, argv);

    /* Crear nueva ventana de aplicación */
    appPtr = gnome_app_new(MY_APP_ID, MY_APP_TITLE);

    /* Crear nuevo botón */
    btnQuit = gtk_button_new_with_label(MY_BUTTON_TEXT);

    /* Hacer que el botón se muestre dentro de la ventana de la aplicación */
    gnome_app_set_contents(appPtr, btnQuit);

    /* Hace que el botón se muestre */
    gtk_widget_show(btnQuit);

    /* Hace que la ventana de la aplicación se muestre */
    gtk_widget_show(appPtr);

    /* Conectar los manejadores de señales */
    gtk_signal_connect(appPtr, "delete_event", GTK_SIGNAL_FUNC(delete_handler), NULL);
    gtk_signal_connect(appPtr, "destroy", GTK_SIGNAL_FUNC(destroy_handler), NULL);
    gtk_signal_connect(btnQuit, "clicked", GTK_SIGNAL_FUNC(click_handler), NULL);

    /* Transferir el control a GNOME */
    gtk_main();

    return 0;
}

/* Función para recibir la señal "destroy" */
int destroy_handler(gpointer window, GdkEventAny *e, gpointer data)
{
    /* Salir del bucle de eventos de GNOME */
    gtk_main_quit();
    return 0;
}

/* Función para recibir la señal "delete_event" */
int delete_handler(gpointer window, GdkEventAny *e, gpointer data)
{
    return 0;
}

/* Función para recibir la señal "clicked" */
int click_handler(gpointer window, GdkEventAny *e, gpointer data)
{
    gpointer msgbox;
    int buttonClicked;

    /* Crear el diálogo "¿Estás seguro?" */
    msgbox = gnome_message_box_new(
        MY_QUIT_QUESTION,
        GNOME_MESSAGE_BOX_QUESTION,
        GNOME_STOCK_BUTTON_YES,
        GNOME_STOCK_BUTTON_NO,
        NULL);
    gtk_window_set_modal(msgbox, 1);
    gtk_widget_show(msgbox);

    /* Ejecutar cuadro de diálogo */
    buttonClicked = gnome_dialog_run_and_close(msgbox);

    /* El botón 0 es el botón Sí. Si este es el
       botón en el que hicieron clic, decirle a GNOME que termine
       su bucle de eventos. De lo contrario, no hacer nada */
    if(buttonClicked == 0)
    {
        gtk_main_quit();
    }
    return 0;
}
```

Para compilarlo, escribe:

```bash
gcc gnome-example-c.c `gnome-config --cflags --libs gnomeui` -o gnome-example-c
```

Ejecútalo escribiendo `./gnome-example-c`.

Finalmente, tenemos una versión en Python. Escríbelo como `gnome-example.py`:

```python
#PROPÓSITO: Este programa está destinado a ser un ejemplo
# de cómo se ven los programas de GUI escritos
# con las bibliotecas GNOME
#

#Importar bibliotecas GNOME
import gtk
import gnome.ui

####DEFINIR FUNCIONES DE CALLBACK PRIMERO####

#En Python, las funciones tienen que definirse antes de
#que se usen, por lo que tenemos que definir nuestras funciones de
#callback primero.

def destroy_handler(event):
    gtk.mainquit()
    return 0

def delete_handler(window, event):
    return 0

def click_handler(event):
    #Crear el diálogo "¿Estás seguro?"
    msgbox = gnome.ui.GnomeMessageBox(
        "¿Estás seguro de que quieres salir?",
        gnome.ui.MESSAGE_BOX_QUESTION,
        gnome.ui.STOCK_BUTTON_YES,
        gnome.ui.STOCK_BUTTON_NO)
    msgbox.set_modal(1)
    msgbox.show()
    result = msgbox.run_and_close()

    #El botón 0 es el botón Sí. Si este es el
    #botón en el que hicieron clic, decirle a GNOME que termine
    #su bucle de eventos. De lo contrario, no hacer nada
    if (result == 0):
        gtk.mainquit()
    return 0

####PROGRAMA PRINCIPAL####

#Crear nueva ventana de aplicación
myapp = gnome.ui.GnomeApp("gnome-example", "Gnome Example Program")

#Crear nuevo botón
mybutton = gtk.GtkButton("Quiero Salir del Programa Ejemplo GNOME")
myapp.set_contents(mybutton)

#Hace que el botón se muestre
mybutton.show()

#Hace que la ventana de la aplicación se muestre
myapp.show()

#Conectar manejadores de señales
myapp.connect("delete_event", delete_handler)
myapp.connect("destroy", destroy_handler)
mybutton.connect("clicked", click_handler)

#Transferir control a GNOME
gtk.mainloop()
```

Para ejecutarlo, escribe `python gnome-example.py`.

## Constructores de GUI

En el ejemplo anterior, has creado la interfaz de usuario para la aplicación llamando a las funciones de creación para cada widget y colocándolo donde querías. Sin embargo, esto puede ser bastante pesado para aplicaciones más complejas. Muchos entornos de programación, incluyendo GNOME, tienen programas llamados **constructores de GUI (GUI builders)** que pueden usarse para crear automáticamente tu GUI por ti. Solo tienes que escribir el código para los manejadores de señales y para inicializar tu programa. El principal constructor de GUI para aplicaciones GNOME se llama **GLADE**. GLADE se distribuye con la mayoría de las distribuciones de Linux.

Hay constructores de GUI para la mayoría de los entornos de programación. Borland tiene un rango de herramientas que construirán GUIs rápida y fácilmente en sistemas Linux y Win32. El entorno KDE tiene una herramienta llamada **QT Designer** que te ayuda a desarrollar automáticamente la GUI para su sistema.

Hay una amplia gama de opciones para desarrollar aplicaciones gráficas, pero espero que este apéndice te haya dado una muestra de cómo es la programación de GUI.
