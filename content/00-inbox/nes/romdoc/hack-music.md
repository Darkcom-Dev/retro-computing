# Hackeo de Música - Bionic Commando
```
+-------------------+----------------------------------------------------------+
| Hackeo de Música  | Esta guía te mostrará cómo hackear la música que se      |
+-------------------+ reproduce en cada habitación de Bionic Commando, así     |
|   Dean Tersigni   | como los efectos de sonido que se reproducen por         |
+-------------------+ diferentes cosas a lo largo del juego.                  |
| Creado: 18/09/03  | Esta guía te mostrará cómo cambiar la música reproducida |
| Act.:   19/09/03  | en cada habitación, no cómo alterar el sonido de la      |
+-------------------+ música.                                                  |
+-------------------+----------------------------------------------------------+
```
Cada habitación está en la lista de abajo. El offset es donde se encuentran los datos en la ROM. Para cambiar la música que se reproduce en la habitación, cambia el valor en la ROM por la música/sonido que quieras de la tabla de sonidos de abajo.

Notarás que todavía hay algunos datos desconocidos en las tablas. Si alguna vez te aburres, por favor tómate el tiempo para encontrar los datos que faltan y envíame un mensaje. Gracias.
```
-------------------------------------------------------------------------------
                                Música de Habitación
-------------------------------------------------------------------------------
Offset ROM
       - Valor Predeterminado
             - Descripción de la Habitación
--------------------------------------------------------------------------------
$2EF33 - $03 - Área 1 (Exterior)
$2EF34 - $05 - Área 2 (Exterior)
$2EF35 - $03 - Área 3 (Exterior)
$2EF36 - $03 - Área 4
$2EF37 - $02 - Área 5 (Sección 1)
$2EF38 - $02 - Área 6 (Sección 2)
$2EF39 - $03 - Área 7
$2EF3A - $06 - Área 8 (Sección 1)
$2EF3B - $05 - Área 9
$2EF3C - $02 - Área 10
$2EF3D - $05 - Área 11
$2EF3E - $04 - Área 12 (Exterior)
$2EF3F - $0A - Área 13
$2EF40 - $0A - Área 14
$2EF41 - $0A - Área 15
$2EF42 - $0A - Área 16
$2EF43 - $0A - Área 17
$2EF44 - $0A - Área 18
$2EF45 - $0A - Área 19
$2EF46 - $09 - Encuentro con enemigo - Acantilado
$2EF47 - $09 - Encuentro con enemigo - Pantano
$2EF48 - $09 - Encuentro con enemigo - Subterráneo
$2EF49 -     - Área 1 (Jefe)
$2EF4A -     - Área 2 (Jefe)
$2EF4B -     - Área 4 (Jefe)
$2EF4C -     - Área 3 (Jefe)
$2EF4D -     - Área 6 (Jefe)
$2EF4E -     - Área 7 (Jefe)
$2EF4F -     - Área 9 (Jefe)
$2EF50 -     - Área 5 (Jefe)
$2EF51 -     - Área 11 (Jefe)
$2EF52 -     - Área 10 (Jefe)
$2EF53 -     - Área 8 (Jefe)
$2EF54 -     - Área 12 (Jefe) ?
$2EF55 -     - ? Jefe no utilizado
$2EF56 -     - ? Jefe no utilizado
$2EF57 -     - ? Jefe no utilizado
$2EF58 -     - ? Jefe no utilizado
$2EF59 -     - Área 8 (Pasaje - Cañón 1)
$2EF5A -     - Área 8 (Pasaje - Soldado)
$2EF5B -     - Sala de Comunicaciones (Área 1 - Exterior)
$2EF5C -     - Sala de Comunicaciones (Área 1 - Interior)
$2EF5D -     - Sala de Comunicaciones (Área ?)
$2EF5E -     - Sala de Comunicaciones (Área ?)
$2EF5F -     - Sala de Comunicaciones (Área ?)
$2EF60 -     - Sala de Comunicaciones (Área ?)
$2EF61 -     - Sala de Comunicaciones (Área ?)
$2EF62 -     - Sala de Comunicaciones (Área ?)
$2EF63 -     - Sala de Comunicaciones (Área ?)
$2EF64 -     - Sala de Comunicaciones (Área ?)
$2EF65 -     - Sala de Comunicaciones (Área ?)
$2EF66 -     - Sala de Comunicaciones (Área ?)
$2EF67 -     - Sala de Comunicaciones (Área ?)
$2EF68 -     - Sala de Comunicaciones (Área ?)
$2EF69 -     - Sala de Comunicaciones (Área ?)
$2EF6A -     - Área Neutral Sala 13-1 (1 Bala)
$2EF6B -     - Área Neutral Sala 13-2 (Bengalas)
$2EF6C -     - Área Neutral Sala 14-1 (Comunicador Gamma)
$2EF6D -     - Área Neutral Sala 14-2 (10 Balas)
$2EF6E -     - Área Neutral Sala 15-1 (Vida Extra)
$2EF6F -     - Área Neutral Sala 15-2 (Comunicador Delta)
$2EF70 -     - Área Neutral Sala 16-1 (Comunicador Beta)
$2EF71 -     - Área Neutral Sala 16-2 ("No seas apresurado")
$2EF72 -     - Área Neutral Sala 17-1 (Interrogatorio)
$2EF73 -     - Área Neutral Sala 17-2 (Ubicación de Joe)
$2EF74 -     - Área Neutral Sala 18-1 (Ametralladora falsa)
$2EF75 -     - Área Neutral Sala 18-2 (Ametralladora de Joe)
$2EF76 -     - Área Neutral Sala 19-1 (4 Balas)
$2EF77 -     - Área Neutral Sala 19-2 (Bombardero suicida)
$2EF78 -     - ? Sala de Comunicaciones (Área ?)
$2EF79 -     - ? Sala de Comunicaciones (Área ?)
$2EF7A -     - Área 1 (Interior)
$2EF7B -     - Área 2 (Interior)
$2EF7C -     - Área 3 (Interior)
$2EF7D -     - Área 5 (Sección 2)
$2EF7E -     - Área 5 (Sección 3)
$2EF7F -     - Área 6 (Sección 2)
$2EF80 -     - Área 8 (Pasaje - Cañón 2)
$2EF81 -     - Área 8 (Pasaje - Cañón 3)
$2EF82 -     - Área 8 (Pasaje - Cañón 4)
$2EF83 -     - Área 8 (Sección 2)
$2EF84 -     - Área 8 (Sala de pinchos)
$2EF85 -     - Área 8 (Sala del ascensor - Primera)
$2EF86 -     - Área 8 (Sala del ascensor - Puerta del Jefe)
$2EF87 -     - ? Sala del Jefe vacía
$2EF88 -     - ? Sala del Jefe vacía
$2EF89 -     - ? Sala de Comunicaciones (Área ?)
$2EF8A -     - ? Sala de Comunicaciones (Área ?)
$2EF8B -     - Área 12 Núcleo de Energía 1
$2EF8C -     - Área 12 Núcleo de Energía 2
$2EF8D -     - ? Sala de la Fuente de Energía
$2EF8E -     - ? Sala de la Fuente de Energía
$2EF8F -     - ? Sala de la Fuente de Energía
$2EF90 -     - ? Sala de la Fuente de Energía
$2EF91 -     - Área 12 (Suelo Eléctrico 1)
$2EF92 -     - Área 12 (Suelo Eléctrico 2)
$2EF93 -     - Área 12 (Sala del Helicóptero)
$2EF94 -     - Área 12 (Pasaje de escape)
$2EF95 -     - Área 12 (Final)
$2EF96 -     - Área 12 (Dispositivo de Reactivación)
$2EF97 -     - Área 12 (?)
$2EF98 -     - Área 12 (Albatros)
$2EF99 -     - Intro Parte 2
$2EF9A -     - Intro Parte 1
$2EF9B -     - Intro Parte 3
$2EF9C -     - Final Parte 1
$2EF9D -     - Final Parte 2
$2EF9E -     - Créditos
$2EF9F -     - Pantalla del Comunicador
$2EFA0 -     - Final Parte 3
$2EFA1 -     - ? Habitación no utilizada
$2EFA2 -     - ? Habitación no utilizada
-------------------------------------------------------------------------------
```
La tabla de sonidos muestra cada sonido del juego.
Notarás que esta lista solo llega hasta $7F, porque en $80 la tabla de sonidos se repite desde el principio. Por lo tanto, $80 es lo mismo que $00, $81 es $01, etc.
Esto significa que solo se utilizan los primeros 7 bits del byte para el sonido. No estoy seguro de para qué sirve el primer bit. El valor es probablemente un signed short.
```
-------------------------------------------------------------------------------
                               Tabla de Sonidos
-------------------------------------------------------------------------------
Valor                          Valor                          Valor
    - Descripción                  - Descripción                  - Descripción
-------------------------------------------------------------------------------
$00 - ? Música de Inicio       $30 - Seleccionar              $60 - Ninguno
$01 - Música de Inicio         $31 - Helicóptero              $61 - Ninguno
$02 - Música Área 5, 6, 10     $32 - Encuentro con enemigo    $62 - Estropeado
$03 - Música Área 1, 3, 4, 7   $33 - Alarma                   $63 - Error
$04 - Música Área 12           $34 - Recoger bote de balas    $64 - Ninguno
$05 - Música Área 2, 9, 11     $35 - Destruir Sistema Princ.  $65 - Ninguno
$06 - Música Área 8            $36 - Vida Extra               $66 - Ninguno
$07 - Música de Jefe           $37 - Medicina                 $67 - Ninguno
$08 - Música de Albatros       $38 - Bala golpea armadura     $68 - Ninguno
$09 - Música Encuentro Enemigo $39 - Ascensor se rompe        $69 - Ninguno
$0A - Música Zona Neutral      $3A - Movimiento de grúa       $6A - Ninguno
$0B - Música Selección Área    $3B - Pausa                    $6B - Ninguno
$0C - Música Área Completada   $3C - Pausa                    $6C - Ninguno
$0D - Música Juego Ganado      $3D - Subir Nivel (Nueva HP)   $6D - Error
$0E - Música Game Over         $3E - Explosión Grande         $6E - Ninguno
$0F - Música de Créditos       $3F - Pausa                    $6F - Detiene Música
$10 - Disparo de Pistola       $40 - Música Diálogo Jefe      $70 - Error
$11 - Disparo Cohete/Ancho     $41 - Música Intro Parte 1     $71 - Estropeado
$12 - Agarre de brazo          $42 - Música Intro Parte 2     $72 - Ninguno
$13 - Lanzamiento de brazo     $43 - Música Intro Parte 3     $73 - Error
$14 - Fallo de brazo           $44 - Música de Final          $74 - Error
$15 - Herido                   $45 - Estropeado               $75 - Ninguno
$16 - Muerto                   $46 - Ninguno                  $76 - Ninguno
$17 - Puerta                   $47 - Estropeado               $77 - Estropeado
$18 - Bala golpea pared        $48 - Ninguno                  $78 - Ninguno
$19 - Fallo de brazo           $49 - Error                    $79 - Ninguno
$1A - Soldado Enemigo muere    $4A - Ninguno                  $7A - Ninguno
$1B - Explosión Mediana        $4B - Ninguno                  $7B - Ninguno
$1C - Muerte de polilla        $4C - Ninguno                  $7C - Ninguno
$1D - ? Desconocido            $4D - Ninguno                  $7D - Ninguno
$1E - ? Muerte remota          $4E - Detiene Música           $7E - Estropeado
$1F - Lanzacohetes enemigo     $4F - Estropeado               $7F - Estropeado
$20 - ? Texto                  $50 - Ninguno
$21 - ? Pitido desconocido     $51 - Ninguno
$22 - Bola de pinchos          $52 - Ninguno
$23 - ? Desconocido            $53 - Ninguno
$24 - Barrera eléctrica        $54 - Ninguno
$25 - Limo                     $55 - Ninguno
$26 - Inicio de planta         $56 - Error
$27 - Ataque de planta         $57 - Error
$28 - Polilla                  $58 - Ninguno
$29 - ? Pitido desconocido     $59 - Ninguno
$2A - Láser de techo           $5A - Ninguno
$2B - Conectar transmisión     $5B - Ninguno
$2C - Conectar transmisión     $5C - Error
$2D - Ninguno                  $5D - Ninguno
$2E - ? Desconexión            $5E - Estropeado
$2F - Cambiar selección        $5F - Ninguno
-------------------------------------------------------------------------------

Algunos de los valores pueden no tener sentido.
Ninguno     - No se reproduce música ni efectos de sonido. (Al menos ninguno que yo pueda oír)
Estropeado  - Reproduce un ruido extraño que no está normalmente en el juego.
Error       - Hace colapsar mi emulador.
Detiene Música - Detiene la música de fondo, para que todo quede en silencio.

-------------------------------------------------------------------------------
```
