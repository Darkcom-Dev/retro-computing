# La NES a Fondo — Parte 6: Mappers

> Basado en la transcripción del video *"El CHIP que REVOLUCIONÓ los GRÁFICOS de la NES"*.  
> Cómo unos chips en los cartuchos transformaron los gráficos de la NES de básicos a extraordinarios.

---

## Índice

1. [Introducción](#1-introducción)
2. [El cartucho básico](#2-el-cartucho-básico)
   - [2.1 Limitaciones](#21-limitaciones)
3. [Bank Switching — El origen de los mappers](#3-bank-switching--el-origen-de-los-mappers)
   - [3.1 City Connection y Jaleco](#31-city-connection-y-jaleco)
   - [3.2 Ventanas: bancos más chicos, más control](#32-ventanas-bancos-más-chicos-más-control)
4. [Animación de tiles con mappers](#4-animación-de-tiles-con-mappers)
   - [4.1 La forma tradicional: cambiar el tile map](#41-la-forma-tradicional-cambiar-el-tile-map)
   - [4.2 Con mappers: cambiar el tile set](#42-con-mappers-cambiar-el-tile-set)
   - [4.3 Comparativa: SMB1 vs SMB3](#43-comparativa-smb1-vs-smb3)
5. [Comunicación CPU ↔ Mapper](#5-comunicación-cpu--mapper)
6. [Bank Switching en PRG ROM](#6-bank-switching-en-prg-rom)
7. [Mirroring Dinámico](#7-mirroring-dinámico)
   - [7.1 Metroid: scroll que cambia sobre la marcha](#71-metroid-scroll-que-cambia-sobre-la-marcha)
8. [La IRQ del Mapper — Scanline Counting](#8-la-irq-del-mapper--scanline-counting)
   - [8.1 Cómo cuenta scanlines](#81-cómo-cuenta-scanlines)
   - [8.2 Split screen sin que la CPU se involucre](#82-split-screen-sin-que-la-cpu-se-involucre)
   - [8.3 Efectos posibles](#83-efectos-posibles)
9. [CHR RAM — Escribir tiles en tiempo real](#9-chr-ram--escribir-tiles-en-tiempo-real)
   - [9.1 Battletoads: parallax por sprite](#91-battletoads-parallax-por-sprite)
   - [9.2 Elite: 3D en la NES](#92-elite-3d-en-la-nes)
10. [Famicom Disk System](#10-famicom-disk-system)
11. [Filosofía de diseño](#11-filosofía-de-diseño)
12. [Conclusión](#12-conclusión)

---

## 1. Introducción

En 1983 se lanzó la Famicom en Japón (1985 como NES en EE.UU.). En sus 10 años de vida, los juegos pasaron de **esto** a **esto**:

```
1983: Circus Charlie, Popeye
    → Fondos estáticos, pocas tiles, cambios de paleta
    
1993: Kirby's Adventure, Battletoads, Ninja Gaiden 3
    → Scroll variable, fondos animados, parallax, ¡y hasta 3D!
```

El causante: el **Mapper** — un chip auxiliar dentro del cartucho que expandía las capacidades de la consola.

```mermaid
flowchart LR
    subgraph "Cartucho básico (1983)"
        CHR_BAS["CHR-ROM: 8 KB<br/>256 tiles × 2"]
        PRG_BAS["PRG-ROM: 32 KB"]
    end
    
    subgraph "Cartucho con Mapper (1990+)"
        CHR_EX["CHR-ROM: hasta 256 KB<br/>con bank switching"]
        PRG_EX["PRG-ROM: hasta 512 KB"]
        MAPPER[Mapper<br/>Chip auxiliar]
    end
    
    CHR_BAS -->|"Sin mappers<br/>no hay evolución"| CHR_EX
    PRG_BAS --> PRG_EX
```

---

## 2. El cartucho básico

```mermaid
flowchart LR
    subgraph "Cartucho básico NES"
        CHR[CHR-ROM<br/>8 KB<br/>Gráficos]
        PRG[PRG-ROM<br/>32 KB<br/>Código]
    end
    
    subgraph "Consola"
        PPU[PPU] --> CHR
        CPU[CPU] --> PRG
    end
    
    CHR -->|"2 pattern tables<br/>256 tiles c/u"| PPU
    PRG -->|"32 KB de código"| CPU
```

| Memoria | Tamaño máx. | Conectada a | Contenido |
|---|---|---|---|
| **CHR-ROM** | 8 KB | PPU | 2 pattern tables × 256 tiles (8×8 px, 4 colores) |
| **PRG-ROM** | 32 KB | CPU | Código del juego |

### 2.1 Limitaciones

La CHR-ROM está **rígidamente estructurada**:

```
CHR-ROM (8 KB):
  ├─ Pattern Table 0 (4 KB): 256 tiles → sprites o fondo
  └─ Pattern Table 1 (4 KB): 256 tiles → fondo o sprites
  Total: 512 tiles fijas. Sin importar cuánta memoria metas.
```

La PPU asigna **8 KB de direcciones** a la CHR-ROM (2 rangos de 4 KB). La CPU asigna **32 KB** a la PRG-ROM. No se puede direccionar más sin ayuda externa.

---

## 3. Bank Switching — El origen de los mappers

### 3.1 City Connection y Jaleco

Jaleco decidió meter una **CHR-ROM de 16 KB** en City Connection, el doble de lo que la PPU podía direccionar. Para ello agregó **chips lógicos (chips TTL)** que permitían seleccionar qué parte de la memoria se le mostraba a la PPU:

```mermaid
flowchart LR
    subgraph "Lo que VE la PPU (8 KB)"
        VISTO["2 pattern tables<br/>256 tiles c/u<br/>8 KB visibles"]
    end
    
    subgraph "Lo que HAY en el cartucho (16 KB)"
        BANCO0["Banco 0<br/>(primer nivel)"]
        BANCO1["Banco 1<br/>(segundo nivel)"]
        BANCO2["Banco 2<br/>(pantallas extra)"]
        BANCO3["Banco 3<br/>(pantallas extra)"]
    end
    
    MAP["Mapper<br/>(chips TTL)"] -->|"Elige qué banco<br/>mostrar en cada momento"| VISTO
    BANCO0 --> MAP
    BANCO1 --> MAP
    BANCO2 --> MAP
    BANCO3 --> MAP
```

**Bank switching**: tomar una memoria más grande y mostrar solo una parte (un "banco") en el rango de direcciones fijo de la PPU/CPU.

**Impacto inmediato:**

| Sin mapper | Con mapper |
|---|---|
| 512 tiles para TODO el juego | 512 tiles POR NIVEL |
| Variedad solo por cambio de paleta | Cada nivel tiene su propio tile set |
| Circus Charlie, Popeye | City Connection, Contra |

### 3.2 Ventanas: bancos más chicos, más control

Pronto aparecieron **ASICs** (Application-Specific ICs) como el **VRC de Konami**, que permitían dividir las pattern tables en **ventanas** más pequeñas:

```mermaid
flowchart LR
    subgraph "Pattern Table 0 (4 KB)"
        V0["Ventana 0<br/>1 KB<br/>(64 tiles)"]
        V1["Ventana 1<br/>1 KB<br/>(64 tiles)"]
    end
    
    subgraph "Pattern Table 1 (4 KB)"
        V2["Ventana 2<br/>1 KB<br/>(64 tiles)"]
        V3["Ventana 3<br/>1 KB<br/>(64 tiles)"]
    end
    
    BANK_POOL["Banco de memoria<br/>128 KB de CHR-ROM"] -->|"Cada ventana<br/>puede cargar<br/>un banco distinto"| V0
    BANK_POOL --> V1
    BANK_POOL --> V2
    BANK_POOL --> V3
```

Cada ventana se podía cargar con un banco de memoria diferente **de forma independiente**. De este modo se podía mantener parte de la pattern table estática mientras se cambiaba solo una ventana (por ejemplo, para animación).

---

## 4. Animación de tiles con mappers

### 4.1 La forma tradicional: cambiar el tile map

Para animar el fondo sin mapper, había que **reescribir el tile map** (name table):

```
Frame 1: tile #10 en posición (5,5) → hoja verde
Frame 2: tile #11 en posición (5,5) → hoja amarilla
```

Cada cambio requería una escritura en los registros de PPU **por cada tile**. Y solo cambiaba **esa instancia** de la tile.

### 4.2 Con mappers: cambiar el tile set

Con bank switching, se cambia **la tile en la pattern table**:

```
Frame 1: banco A → tile #10 = hoja verde
Frame 2: banco B → tile #10 = hoja amarilla
```

**Una sola escritura al mapper** cambia **todas las hojas del nivel a la vez**.

```mermaid
flowchart LR
    subgraph "Sin mapper"
        TM["Tile map: tile #10<br/>Tile #10<br/>Tile #10"] --> ESC["Escribir 3 veces<br/>en registros PPU"]
        ESC --> A["3 escrituras<br/>3 tiles cambiadas"]
    end
    
    subgraph "Con mapper"
        BANK["Banco de tiles"] --> SW["1 escritura<br/>al registro del mapper"]
        SW --> B["TODAS las tiles #10<br/>del nivel cambian<br/>a la vez"]
    end
```

### 4.3 Comparativa: SMB1 vs SMB3

| Juego | Técnica | Cómo se ve |
|---|---|---|
| **Super Mario Bros. (1985)** — Sin mapper | Cambio de **paleta** en tiles específicas | Bloque de interrogación parpadea cambiando colores |
| **Super Mario Bros. 3 (1988)** — Con mapper | **Bank switching** por frame | Animaciones complejas de bloques, tuberías, decorados |

Cada frame se cambiaban los bancos de CHR-ROM, generando ciclos de animación fluidos en los decorados del fondo.

---

## 5. Comunicación CPU ↔ Mapper

El mapper se asigna **las primeras direcciones del rango de la PRG-ROM** (originalmente para el código del juego). Escribir en esas direcciones **no escribe en memoria, sino en los registros internos del mapper**:

```mermaid
flowchart LR
    subgraph "Rango de direcciones de PRG-ROM (CPU)"
        DIRS["$8000 - $FFFF<br/>(32 KB)"]
    end
    
    DIRS -->|"$8000-$9FFF<br/>Registros del MAPPER"| MAP[Mapper]
    DIRS -->|"$A000-$FFFF<br/>Código del juego"| PRG[PRG-ROM]
    
    CPU[CPU] -->|"Escribe en $8000"| MAP
    CPU -->|"Escribe en $A000"| PRG
```

Si ves en un depurador que un juego escribe en direcciones como `$8000`, `$8001`, está **interactuando con el mapper**.

**Ejemplo:** escribir en `$8000` para cambiar el banco de la ventana 0.

---

## 6. Bank Switching en PRG-ROM

El bank switching también se aplicó a la **PRG-ROM** para meter más código. Los 32 KB originales se quedaron cortos:

| Juego | Tamaño de PRG-ROM |
|---|---|
| Super Mario Bros. | 32 KB |
| Super Mario Bros. 3 | 256 KB |
| **Kirby's Adventure** | **512 KB de código** + 256 KB de tiles |

Kirby's Adventure: **17 veces más código** que SMB1.

---

## 7. Mirroring Dinámico

En los cartuchos básicos, el **mirroring** (horizontal o vertical) se definía soldando un pin:

```mermaid
flowchart LR
    subgraph "Cartucho básico"
        PIN[Pin soldado<br/>a H o V] --> FIJ[Mirroring fijo<br/>para todo el juego]
    end
    
    subgraph "Cartucho con mapper"
        PIN2[Línea de mirroring<br/>→ va al MAPPER] --> DYN[Mirroring cambiable<br/>en tiempo de ejecución]
        CPU2[CPU escribe<br/>en registro] --> DYN
    end
```

### 7.1 Metroid: scroll que cambia sobre la marcha

Metroid cambia constantemente entre scroll horizontal (pasillos anchos) y vertical (habitaciones altas). Con el mirroring dinámico, el mapper cambia el tipo de mirroring **en el medio del juego** cuando el jugador atraviesa una puerta.

```mermaid
flowchart LR
    A["Habitación horizontal<br/>Mirroring: vertical"] -->|"Puerta"| MAPPER
    MAPPER -->|"Cambia registro<br/>de mirroring"| B["Habitación vertical<br/>Mirroring: horizontal"]
```

---

## 8. La IRQ del Mapper — Scanline Counting

### 8.1 Cómo cuenta scanlines

El mapper monitoriza dos señales de la consola:

```mermaid
flowchart LR
    subgraph "Señales que espía el mapper"
        PPU_A12["PPU A12<br/>Oscila cada vez que la PPU<br/>accede a CHR-ROM<br/>= INICIO de scanline"]
        CPU_M2["CPU M2<br/>Oscila cada ciclo de CPU<br/>= CONTADOR de ciclos"]
    end
    
    PPU_A12 --> MAP["Mapper<br/>sabe cuándo empieza<br/>cada scanline"]
    CPU_M2 --> MAP
    MAP --> IRQ["IRQ a la CPU<br/>en la scanline N<br/>exacta"]
```

El mapper puede **contar scanlines** sin que la CPU tenga que hacerlo. Cuando llega a la línea configurada, dispara una **IRQ** a la CPU.

### 8.2 Split screen sin que la CPU se involucre

Antes de los mappers, partir la pantalla requería que la CPU **contara ciclos manualmente** para saber en qué scanline estaba. Esto:
- Consumía tiempo de CPU
- Solo permitía partir la pantalla **una vez** (para el HUD)
- Era propenso a errores

Con la IRQ del mapper:

```
CPU: "Mapper, avísame en la scanline #80."
Mapper: (cuenta 80 scanlines) → ¡IRQ!
CPU: (en la IRQ) → cambio scroll / paleta / lo que sea
```

### 8.3 Efectos posibles

```mermaid
flowchart TD
    IRQ_F["Mapper IRQ"] --> PARALLAX["Parallax<br/>Cada sección de pantalla<br/>scrollea a velocidad distinta"]
    IRQ_F --> MULTI_SCROLL["Multi-scroll<br/>Cada sección scrollea<br/>en dirección distinta"]
    IRQ_F --> FILTROS["Filtros dinámicos<br/>Ej: escala de grises<br/>que sube y baja"]
    
    PARALLAX --> NG["Ninja Gaiden 2,<br/>Super Mario 3<br/>(minijuego 3 pantallas)"]
    MULTI_SCROLL --> SMB3_MIN["Super Mario 3<br/>minijuego de cartas"]
    FILTROS --> NOAH["Noah's Ark<br/>(efecto de agua)"]
    FILTROS --> SHARKY["Sharky's Shell<br/>(lava subiendo)"]
```

**Ninja Gaiden 2** y su parallax: en cada scanline específico, el mapper dispara una IRQ, la CPU cambia el scroll, y se generan **3 o más capas de parallax** a distintas velocidades, todo con un solo fondo.

**Super Mario 3 — minijuego:** la pantalla se parte en 3 secciones, cada una scrolleando horizontal y verticalmente de forma independiente.

---

## 9. CHR RAM — Escribir tiles en tiempo real

Hasta ahora hablamos de **CHR-ROM** (Read Only Memory). Pero algunos cartuchos usaban **CHR-RAM** — memoria RAM en lugar de ROM en el espacio de direcciones de la PPU.

```mermaid
flowchart LR
    subgraph "CHR-ROM (solo lectura)"
        ROM["Tiles fijas<br/>en el cartucho"]
        PPU["PPU lee"] --> ROM
    end
    
    subgraph "CHR-RAM (lectura/escritura)"
        RAM["Memoria RAM<br/>en el cartucho"]
        PPU2["PPU lee"] --> RAM
        CPU2["CPU escribe<br/>tiles en tiempo real"] --> RAM
    end
```

La CPU escribe en la CHR-RAM usando los registros **PPU Address** + **PPU Data** (los mismos que para name tables y paletas). Los datos pueden venir de la PRG-ROM o de la W-RAM.

### 9.1 Battletoads: parallax por sprite

Battletoads usa CHR-RAM para generar dos efectos de parallax **imposibles** con el hardware estándar:

```mermaid
flowchart LR
    subgraph "Parallax vertical"
        CPU_SCR["CPU scrollea tiles<br/>individuales píxel a píxel"]
        SCR_EFFECT["Los bordes se mueven<br/>a velocidad diferente<br/>que el fondo"]
    end
    
    subgraph "Parallax dual"
        CPU_DUAL["CPU mueve tiles<br/>en horizontal Y vertical"]
        DUAL_EFFECT["Parallax que responde<br/>al movimiento del viewport<br/>en ambas direcciones"]
    end
```

La CPU **no usa el scroll de la PPU**. La CPU misma mueve píxel a píxel las tiles escritas en CHR-RAM, creando un scroll paralelo independiente.

### 9.2 Elite: 3D en la NES

Elite es un juego de PC con gráficos 3D de alambre portado a la NES. El truco:

```mermaid
flowchart TD
    CPU_ELITE["CPU (1.79 MHz)<br/>Calcula vértices 3D,<br/>rotaciones, proyección"]
    CPU_ELITE --> W_RAM["W-RAM<br/>Cálculos vectoriales<br/>y de matrices"]
    W_RAM --> CHR_RAM["CHR-RAM<br/>CPU rasteriza<br/>píxeles aquí"]
    CHR_RAM --> NAME_TABLES["Name Tables<br/>(fondo del juego)"]
    NAME_TABLES --> PPU_DRAW["PPU dibuja<br/>sin saber que<br/>es 3D"]
    
    SPRITES["Sprites: accesorios"] --> PPU_DRAW
```

**Pipeline del 3D en NES:**

1. CPU calcula vértices 3D, rotaciones, transformaciones → en W-RAM
2. CPU aplica clipping y proyección (matemática de punto fijo)
3. CPU rasteriza: escribe los píxeles de los triángulos proyectados en la **CHR-RAM** (convertidos a tiles)
4. El fondo del juego (name tables) referencia esas tiles → la PPU dibuja el "3D" sin enterarse

> **[ ]** La PPU no sabe nada de 3D. Solo dibuja tiles. La CPU hace todo el trabajo de rasterización.

Todo esto corriendo en una CPU de 1.79 MHz. Los desarrolladores dejaron documentadas todas las optimizaciones en un sitio web.

---

## 10. Famicom Disk System

Antes de que los mappers explotaran, Nintendo intentó otra vía: el **Famicom Disk System** (1986, solo Japón).

```mermaid
flowchart LR
    subgraph "Famicom Disk System"
        DISK[Disquete<br/>Mayor capacidad<br/>Regrabable<br/>Podía guardar partida]
        ADAPTER[Adaptador<br/>se enchufa en<br/>la ranura de cartuchos]
    end
    
    DISK --> ADAPTER --> CONSOLE[Famicom]
```

Ventajas del Disk System:
- Disquetes más baratos de producir que cartuchos
- Mayor capacidad de almacenamiento
- Permitía guardar la partida en el propio disquete
- El **mirroring dinámico** se originó aquí (el tipo de mirroring se almacenaba en el disquete)

Pero fracasó:
- Nintendo quería controlar la producción y cobrar regalías
- Los desarrolladores preferían mantener el control con cartuchos
- La tecnología de chips avanzó rápido → los cartuchos con mappers superaron al Disk System
- **Nunca salió de Japón**

Para lanzar Zelda, Metroid y SMB2 en EE.UU., Nintendo desarrolló su propio mapper: el **MMC** (Memory Management Controller). Y como Nintendo controlaba la producción de cartuchos en EE.UU. (por el sello de calidad post-crisis del 83), las third parties como Konami tenían que adaptarse al MMC si querían lanzar juegos con mapper en ese mercado.

---

## 11. Filosofía de diseño

La NES fue diseñada para ser **barata**, con el sobrecoste de juegos más complejos recayendo en los **cartuchos**:

```mermaid
flowchart LR
    subgraph "Diseño de la NES"
        CHEAP["Consola barata<br/>Hardware mínimo"]
        EXPAND["Capacidades de expansión<br/>en el conector del cartucho"]
    end
    
    EXPAND --> MEM_EXTRA["Memoria extra en direcciones<br/>de PPU y CPU<br/>(no usada inicialmente)"]
    EXPAND --> IRQ_LINE["Línea IRQ desde el cartucho<br/>(no usada inicialmente)"]
    EXPAND --> CUSTOM["Chips custom en el cartucho<br/>(mappers)"]
    
    MEM_EXTRA -->|"Jaleco, Konami"| BANK_SW["Bank switching"]
    IRQ_LINE --> SCANLINE["Scanline IRQ<br/>→ split screen"]
    CUSTOM --> CHR_RAM_CUSTOM["CHR-RAM<br/>→ tiles dinámicas, 3D"]
```

> No es que la NES "sin los chips extra no funcionaba". La consola **estaba diseñada** para que los cartuchos pudieran expandir sus capacidades.
>
> *"Tampoco es que metían un Ryzen en el cartucho y salían dando el Call of Duty."*

---

## 12. Conclusión

```mermaid
flowchart TD
    BASICO["Cartucho básico<br/>CHR: 8 KB<br/>PRG: 32 KB<br/>Mirroring fijo"] --> BANK_SW["Bank Switching<br/>CHR: hasta 256 KB<br/>PRG: hasta 512 KB"]
    BASICO --> VENTANAS["Ventanas<br/>Animación de tiles<br/>por frame"]
    BASICO --> DYN_MIR["Mirroring Dinámico<br/>Metroid"]
    BASICO --> IRQ_SCAN["IRQ por scanline<br/>Parallax, split screen,<br/>filtros dinámicos"]
    BASICO --> CHR_RAM_END["CHR-RAM<br/>Tiles dinámicas<br/>Battletoads parallax<br/>Elite 3D"]
    
    BANK_SW --> KIRBY["Kirby's Adventure:<br/>512 KB código + 256 KB tiles"]
    VENTANAS --> SMB3["SMB3: animaciones<br/>de fondo complejas"]
    IRQ_SCAN --> NG2["Ninja Gaiden 2:<br/>parallax de 3 capas"]
    CHR_RAM_END --> ELITE["Elite: 3D wireframe<br/>en NES"]
```

### Evolución de los mappers

| Generación | Característica | Juego representativo |
|---|---|---|
| **TTL discreto** | Bank switching básico (CHR-ROM mayor) | City Connection (Jaleco) |
| **ASIC (VRC, MMC)** | Ventanas, mirroring dinámico, IRQ | Contra (Konami), SMB3 (Nintendo) |
| **Avanzado (MMC3, VRC6)** | Scanline IRQ precisa, CHR-RAM | Ninja Gaiden 2, Battletoads |
| **Último grito** | CHR-RAM + CPU haciendo 3D | Elite |

Los mappers fueron la conjunción de:
1. **Ingenio descomunal** de los desarrolladores
2. **Arquitectura expandible** de la NES
3. Una época donde "se le sacaba jugo a las piedras"

---

> **Fin de la saga NES.**  
> Próximos temas posibles: Super Nintendo, Game Boy (arquitectura similar a NES), comparativas con Sega.
>
> **Fuente:** Transcripción del video *"El CHIP que REVOLUCIONÓ los GRÁFICOS de la NES"* (YouTube)
