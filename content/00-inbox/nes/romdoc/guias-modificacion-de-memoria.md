# Modificación de memoria

En NES si deseas buscar trucos, hay que toquetear la memoria, para hacerlo

0000-07FF = donde estan los trucos
0800-1FFF = de aqui para arriba no hay nada interesante - solo una copia espejo de la pagina anterior

Hay cuatro replicas de los mismos datos: 
```hex
	0000-07FF, 
	0800-0FFF, 
	1000-17FF,
	1800-1FFF. 
```
> Solo se utiliza realmente la memoria encontrada en las direcciones $0000-07FF.

### PPU (graficos)
$2000-$3FFF: puertos de E / S para la comunicacion con la unidad de procesamiento de imagenes (PPU)

### APU (audio)
$4000-$4017: puertos de E / S para los circuitos de audio, entrada y DMA (APU)

Para modificar el fondo directamente desde la memoria hay que abrir el NameTable Viewer, te saca toda una copia de la pantalla y te da las posciciones en x y Y, ademas de la direccion en memoria de la PPU

### Pulsacion de botones
$0000F0: 8v0 y 6to byte = detecta pulsacion de botones

### Donde estan ubicadas las cosas en la memoria
```
$0000-$0FFF: BG tiles
$1000-$17FF: Sprite tiles
$1800-$1BFF: Nametable
$1F00-$1F1F: Palette
```