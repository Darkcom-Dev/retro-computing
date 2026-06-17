# Los Gráficos de la NES — Parte 1: Tiles, Paletas, Sprites y Fondos

> Basado en la transcripción del video homónimo.  
> Cómo se construían los gráficos en la era de los 8 bits, cuando 8 KB eran la panacea.

---

## Índice

1. [Introducción](#1-introducción)
2. [El Píxel: el átomo gráfico](#2-el-píxel-el-átomo-gráfico)
3. [Tiles: los ladrillos del mundo 2D](#3-tiles-los-ladrillos-del-mundo-2d)
4. [Tile Set y Tile Map](#4-tile-set-y-tile-map)
5. [Ventajas del sistema de tiles](#5-ventajas-del-sistema-de-tiles)
6. [El Color en la NES](#6-el-color-en-la-nes)
   - [6.1 True Color vs. Paletas](#61-true-color-vs-paletas)
   - [6.2 Paletas indexadas en la NES](#62-paletas-indexadas-en-la-nes)
   - [6.3 La paleta universal de 64 colores](#63-la-paleta-universal-de-64-colores)
   - [6.4 Paletas activas: 4 + 4](#64-paletas-activas-4--4)
7. [El Fondo (Background)](#7-el-fondo-background)
   - [7.1 Tile map gigante](#71-tile-map-gigante)
   - [7.2 Scrolling](#72-scrolling)
8. [Los Sprites](#8-los-sprites)
   - [8.1 Características](#81-características)
   - [8.2 Límite de 8 por línea](#82-límite-de-8-por-línea)
   - [8.3 El parpadeo como solución](#83-el-parpadeo-como-solución)
9. [Conclusión](#9-conclusión)

---

## 1. Introducción

Los gráficos modernos se construyen con **motores 3D basados en polígonos**. En las generaciones de 8 y 16 bits el paradigma era completamente distinto: **tiles, sprites y fondos**.

```mermaid
flowchart LR
    subgraph "Gráficos modernos"
        A1[Escena 3D] --> A2[Polígonos] --> A3[Texturas] --> A4[Pixel Shaders]
    end
    
    subgraph "Gráficos 8/16 bits (NES)"
        B1[Tiles 8x8] --> B2[Tile Map] --> B3[Fondo + Sprites]
        B4[Paletas] --> B3
    end
```

La NES (Nintendo Entertainment System), conocida como **Family Game** en Argentina y **Famicom** en Japón, corría a una resolución de **256 × 240 píxeles** (área visible típica: 256 × 224 por overscan de las teles de tubo).

---

## 2. El Píxel: el átomo gráfico

**Píxel** = abreviatura de *Picture Element*. Es el elemento mínimo para representar una imagen y tiene una sola cualidad: **un color**.

```mermaid
flowchart LR
    A[Píxel] --> B[1 sola cualidad: color]
    B --> C[Guardado como N bits en memoria]
    C --> D[2 bits, 8 bits, 16 bits, 32 bits...]
```

Juntando píxeles se construye todo lo demás.

---

## 3. Tiles: los ladrillos del mundo 2D

Un **tile** es una imagen pequeña de **tamaño fijo**. En la NES: **8 × 8 píxeles**.

```mermaid
block-beta
    columns 8
    P1[" "] P2[" "] P3[" "] P4[" "] P5[" "] P6[" "] P7[" "] P8[" "]
    P9[" "] P10[" "] P11[" "] P12[" "] P13[" "] P14[" "] P15[" "] P16[" "]
    P17[" "] P18[" "] P19[" "] P20[" "] P21[" "] P22[" "] P23[" "] P24[" "]
    P25[" "] P26[" "] P27[" "] P28[" "] P29[" "] P30[" "] P31[" "] P32[" "]
    P33[" "] P34[" "] P35[" "] P36[" "] P37[" "] P38[" "] P39[" "] P40[" "]
    P41[" "] P42[" "] P43[" "] P44[" "] P45[" "] P46[" "] P47[" "] P48[" "]
    P49[" "] P50[" "] P51[" "] P52[" "] P53[" "] P54[" "] P55[" "] P56[" "]
    P57[" "] P58[" "] P59[" "] P60[" "] P61[" "] P62[" "] P63[" "] P64[" "]
```

> Tile de 8×8 = 64 píxeles.  
> Cada pixel ocupa 2 bits → 64 × 2 = 128 bits = **16 bytes por tile**.

Los tiles son la base con la que se construyen todos los gráficos. Por ejemplo, Mario se forma juntando **4 tiles** (2×2).

---

## 4. Tile Set y Tile Map

### Tile Set

Conjunto de tiles **únicos** con un **identificador numérico** cada uno.

```
Tile Set: [Tile #0] [Tile #1] [Tile #2] ... [Tile #N]
```

### Tile Map

Grilla de números que indica **qué tile va en cada posición** para reconstruir la imagen.

```mermaid
flowchart LR
    subgraph "Tile Set (datos gráficos)"
        T0[Tile #0: ☁️] 
        T1[Tile #1: 🌳]
        T2[Tile #2: 🟫]
    end
    
    subgraph "Tile Map (diseño de nivel)"
        M0[0, 1, 0, 2<br/>0, 1, 0, 2<br/>2, 2, 2, 2]
    end
    
    subgraph "Resultado en pantalla"
        R0[☁️🌳☁️🟫<br/>☁️🌳☁️🟫<br/>🟫🟫🟫🟫]
    end
    
    T0 --> M0
    T1 --> M0
    T2 --> M0
    M0 --> R0
```

Cada referencia en el tile map ocupa **1 byte** (permite direccionar hasta 256 tiles distintos).

---

## 5. Ventajas del sistema de tiles

```mermaid
flowchart TD
    A[Sistema de Tiles] --> B[Desacoplamiento<br/>gráfico vs. diseño]
    A --> C[Manejo de imágenes<br/>gigantes en memorias<br/>chicas]
    A --> D[Ahorro de<br/>almacenamiento]
    
    B --> B1[Cambiar tile set<br/>sin tocar niveles]
    C --> C1[Cargar tiles<br/>bajo demanda]
    D --> D1["Tile repetido =<br/>1 byte en tile map<br/>(vs. 16 bytes si<br/>fuera tile nuevo)"]
```

### Desacoplamiento

Se separa:
- **Tile set** → la información gráfica (los píxeles)
- **Tile map** → el diseño del nivel / construcción de objetos

Si Miyamoto se despertaba y quería cambiar el piso de nieve a pasto, solo tocaba el **tile set**. Los niveles (tile maps) no había que modificarlos.

### Ahorro de almacenamiento

| Elemento | Tamaño |
|---|---|
| 1 tile (8×8, 2 bits/pixel) | **16 bytes** |
| 1 referencia en tile map | **1 byte** |
| Por cada tile repetido | Se ahorran **15 bytes** |

Los 16 bytes por tile se explican porque cada píxel usa 2 bits (64 píxeles × 2 bits = 128 bits = 16 bytes). Y cada píxel usa 2 bits por la forma en que la NES manejaba el color, que vemos a continuación.

---

## 6. El Color en la NES

### 6.1 True Color vs. Paletas

Hoy en día lo común es **True Color**: 4 valores de 8 bits (R, G, B, A) → más de 16 millones de colores.

```
En True Color:
  - 1 tile (8×8) = 64 × 4 bytes = 256 bytes
  - Mario completo (4 tiles) = 1 KB
  - Memoria de video total de la NES = 8 KB
  → IMPOSIBLE
```

Solución: **paletas indexadas**.

### 6.2 Paletas indexadas en la NES

```mermaid
flowchart LR
    subgraph "Tile en memoria"
        P[Pixel 1: 00<br/>Pixel 2: 01<br/>Pixel 3: 10<br/>Pixel 4: 11<br/>...]
    end
    
    subgraph "Paleta"
        C0["#0 → Color concreto<br/>(rojo, verde, azul...)"]
        C1["#1 → ..."]
        C2["#2 → ..."]
        C3["#3 → ..."]
    end
    
    P --> C0
    P --> C1
    P --> C2
    P --> C3
    
    R["Resultado: cada pixel<br/>usa SOLO 2 bits<br/>(valores 0-3)"] -.-> P
```

- Cada píxel del tile ocupa **2 bits** → puede referenciar 4 colores (0, 1, 2, 3)
- Esos 4 colores se definen en una **paleta**
- La paleta guarda **referencias** a otra paleta (no colores concretos)

### 6.3 La paleta universal de 64 colores

La NES tenía una **paleta universal** de **64 colores fijos**. Todos los juegos debían elegir sus colores de aquí.

```mermaid
flowchart LR
    subgraph "Paleta Universal (64 colores)"
        U0["#00"] 
        U1["#01"] 
        U2["#02"] 
        U3["#3F"]
    end
    
    subgraph "Paleta activa de tile"
        A0["Índice 0 → #12"]
        A1["Índice 1 → #2A"]
        A2["Índice 2 → #35"]
        A3["Índice 3 → #0F"]
    end
    
    subgraph "Tile en pantalla"
        B0["Pixel = 00 → color #12"]
        B1["Pixel = 01 → color #2A"]
        B2["Pixel = 10 → color #35"]
        B3["Pixel = 11 → color #0F"]
    end
    
    U0 --> A0
    U1 --> A0
    U0 --> A1
    U0 --> A2
    U0 --> A3
    
    A0 --> B0
    A1 --> B1
    A2 --> B2
    A3 --> B3
```

Cada tile tiene **solamente 3 colores visibles** + 1 transparente. Si ves un tile con más colores, es porque se superponían objetos (ej: el ojito de un personaje se lograba con un tile extra encima).

### 6.4 Paletas activas: 4 + 4

La NES tenía **8 paletas activas** en total:

| Grupo | Cantidad | Destino |
|---|---|---|
| **Fondo** | 4 paletas | Para tiles del escenario |
| **Sprites** | 4 paletas | Para sprites |

En teoría: 8 paletas × 3 colores cada una = **32 colores** en pantalla.  
En la práctica: **25 colores** (el color #0 de cada paleta de fondo se comparte, y hay limitaciones adicionales que se explican en la Parte 2).

Ventaja clave de las paletas: **reutilización de tiles**.

```mermaid
flowchart LR
    subgraph "Mismo tile set"
        T[Tile de nube] --> P1[Paleta: azul/blanco]
        T --> P2[Paleta: verde/marrón]
    end
    
    subgraph "Resultado"
        P1 --> Nube[Nube blanca ☁️]
        P2 --> Arbusto[Arbusto verde 🌳]
    end
```

La nube y el arbusto en Super Mario Bros. usan **los mismos tiles**, solo cambia la paleta. Esto también permitía efectos como Mario con estrella (rotación de colores) o convertir a Mario en Luigi sin cambiar ni un píxel del sprite.

---

## 7. El Fondo (Background)

### 7.1 Tile map gigante

El fondo es un **tile map de gran tamaño** que se mueve en conjunto.

### 7.2 Scrolling

```mermaid
flowchart LR
    subgraph "Fondo completo (tile map)"
        FM["[0,1,2,3,4,5,6,7,...]<br/>[8,9,10,11,12,13,...]<br/>[...]"]
    end
    
    subgraph "Viewport (256×240)"
        VP["Porción visible<br/>del fondo"]
    end
    
    subgraph "En pantalla"
        SCR[Imagen visible]
    end
    
    FM --> VP --> SCR
```

La interpretación correcta es que el **viewport se mueve sobre el fondo**, no el fondo sobre la pantalla.

Tipos de scrolling en la NES:

```mermaid
flowchart TD
    A[Scrolling en NES] --> B[Horizontal<br/>Super Mario Bros.]
    A --> C[Vertical<br/>Ice Climber]
    A --> D[Horizontal + Vertical<br/>Metroid]
    A --> E[Mixto avanzado<br/>Super Mario 3, Kirby]
    
    E --> F["Logrado mediante<br/>técnicas avanzadas<br/>(casi hackeo de la máquina)"]
```

---

## 8. Los Sprites

### 8.1 Características

Los sprites son entidades dinámicas que se mueven **individualmente** con una **posición relativa al viewport** (como si flotaran sobre la pantalla).

| Propiedad | Valor |
|---|---|
| Tamaño (modo más común) | **8 × 8** píxeles |
| Tamaño alternativo | 8 × 16 (usado en juegos avanzados como SMB3) |
| Máximo en pantalla | **64** sprites |
| Máximo por línea horizontal | **8** sprites |

Un personaje como Mario en realidad son **4 sprites de 8×8 moviéndose en conjunto**.

### 8.2 Límite de 8 por línea

La NES podía tener hasta **64 sprites** en pantalla, pero el límite más crítico era de **8 sprites por línea de dibujado** (línea horizontal de la imagen).

```mermaid
flowchart LR
    subgraph "Línea de dibujado individual"
        SP1[Sprite 1] SP2[Sprite 2] SP3[...] SP4[...] SP5[...] SP6[...] SP7[...] SP8[Sprite 8] SP9["Sprite 9<br/>✗ INVISIBLE"]
    end
```

Si en una misma línea horizontal hay 9 o más sprites, el noveno (y siguientes) **no se dibuja** — queda invisible.

### 8.3 El parpadeo como solución

Para evitar que un sprite desaparezca por completo (como un enemigo que te hace daño), los desarrolladores rotaban el orden de los sprites en memoria **cada frame (60 fps)**:

```mermaid
flowchart TD
    subgraph "Frame 1"
        F1A["Sprite A: dibujado"]
        F1B["Sprite B: dibujado"]
        F1C["Sprite C: INVISIBLE"]
    end
    
    subgraph "Frame 2"
        F2A["Sprite A: INVISIBLE"]
        F2B["Sprite B: dibujado"]
        F2C["Sprite C: dibujado"]
    end
    
    subgraph "Frame 3"
        F3A["Sprite A: dibujado"]
        F3B["Sprite B: INVISIBLE"]
        F3C["Sprite C: dibujado"]
    end
    
    F1A --> F2A --> F3A
    F1B --> F2B --> F3B
    F1C --> F2C --> F3C
```

Rotando posiciones a 60 fps, los sprites que desaparecen van cambiando constantemente. El ojo humano percibe esto como **parpadeo**.

> **El parpadeo no es un defecto de la consola, sino un efecto colateral de la estrategia de los desarrolladores** para evitar que un sprite quede permanentemente invisible.

---

## 9. Conclusión

El sistema de tiles de la NES era una solución de ingeniería brillante para las limitaciones de la época:

```mermaid
flowchart TD
    Memoria["8 KB de VRAM"] --> Tiles["Tiles 8×8 (16 bytes)<br/>en lugar de imágenes<br/>enteras"]
    Tiles --> TileMap["Tile Map: referencias<br/>de 1 byte en lugar de<br/>píxeles repetidos"]
    
    Bits["2 bits por píxel"] --> Paletas["Paletas: 4 colores<br/>por tile"]
    Paletas --> Reuso["Reuso de tiles<br/>cambiando paletas"]
    Paletas --> Limite64["64 colores totales<br/>en la paleta universal"]
    
    Sprites["64 sprites máx."] --> Linea8["8 por línea"]
    Linea8 --> Parpadeo["Parpadeo como<br/>workaround"]
    
    Fondo["Fondo: tile map<br/>gigante"] --> Scroll["Scrolling horizontal<br/>/ vertical"]
```

Todo estaba gobernado por una **economía de recursos furiosa**: cada byte contaba, cada límite tenía una razón de hardware.

---

> **Próximo video (Parte 2):** Especificaciones técnicas a nivel hardware, la señal de TV, y cómo la consola construía estos gráficos desde el punto de vista electrónico.
>
> **Fuente:** Transcripción del video *"Los GRÁFICOS de la NES - PARTE 1 - Tiles, Paletas, Sprites y Fondos"* (YouTube)
