# Hackeo de Sonido - Bionic Commando
```
+-------------------+----------------------------------------------------------+
| Hackeo de Sonido  | A diferencia de la música de Bionic Commando, el sonido  |
+-------------------+ no está configurado de forma tan sencilla. Mientras que |
|   Dean Tersigni   | la música está simplemente en una línea de datos, el     |
+-------------------+ código de sonido está por todas partes. Aún no he       |
| Creado: 19/09/03  | encontrado todos los efectos de sonido, pero sigo        |
| Act.:   00/00/00  | buscando. Esta guía solo muestra cómo alterar el efecto  |
+-------------------+ de sonido reproducido, no los datos de sonido en sí.     |
+-------------------+----------------------------------------------------------+
```
Donde se reproduce el efecto de sonido en la ROM se indica a continuación. Todo lo que tienes que hacer es cambiar el valor existente por el sonido que quieras, el cual se encuentra en la tabla de sonidos al final de este documento.

Por ejemplo, si quieres cambiar el sonido que hace la pistola normal, simplemente abre la ROM de Bionic Commando en un editor hexadecimal, ve al offset $37D84 y cambia el valor existente que es $10 (Disparo de pistola normal) por $1B (Explosión mediana). A partir de ahora, cuando dispares tu arma, hará un sonido de explosión.

Notarás que todavía hay algunos datos desconocidos en las tablas. Si alguna vez te aburres, por favor tómate el tiempo para encontrar los datos que faltan y envíame un mensaje. Gracias.
```
-------------------------------------------------------------------------------
                              Efectos de Sonido
-------------------------------------------------------------------------------
Offset ROM
       - Valor Predeterminado
             - Descripción del Sonido
--------------------------------------------------------------------------------
$35B1D - $1F - Sonido Lanzacohetes Enemigo
$362D6 - $07 - Sonido Música de Jefe
$36B0C - $3C - Sonido de Pausa
$362CB - $40 - Sonido de Diálogo al Hablar con el Jefe
$36467 - $0B - Sonido Mapa de Selección de Área
$36688 - $32 - Sonido de Encuentro con Enemigo
$3754C - $15 - Sonido de Jugador Herido
$376A5 - $16 - Sonido de Jugador Muerto
$37AC7 - $37 - Sonido de Medicina
$37D84 - $10 - Sonido de Disparo de Pistola Normal
$37DAB - $18 - Sonido de Bala Golpea Pared
$37DD2 - $11 - Sonido de Cohete/Ancho/3-Vías
$38209 - $1A - Sonido de Muerte de Soldado Enemigo
$39138 - $13 - Sonido de Lanzamiento de Brazo
$391B9 - $12 - Sonido de Agarre de Brazo
$391D9 - $19 - Sonido de Fallo de Brazo
$398C6 - $26 - Sonido de Inicio de Planta
$398DC - $27 - Sonido de Ataque de Planta
$39ABE - $24 - Sonido de Barrera Eléctrica
$39B6F - $39 - Sonido de Ascensor Rompiéndose
$39C86 - $3A - Sonido de Grúa Moviéndose
$39CB5 - $1B - Sonido de Explosión Mediana (Grúas, etc.)
$39E87 - $25 - Sonido de Limo
$3A83F - $17 - Sonido de Puerta
$3B9A6 - $34 - Sonido de Adquirir Balas
$3BE74 - $35 - Sonido de Destrucción del Sistema Principal
$3C412 - $2B - Sonido de Inicio de Transmisión
$3C541 - $2F - Sonido de Cambio de Selección

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
