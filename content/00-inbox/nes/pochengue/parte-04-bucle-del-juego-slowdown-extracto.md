# La NES a Fondo — Parte 4: El Bucle de Juego y el Slowdown

> Basado en la transcripción del video homónimo.  
> Cómo la CPU, la PPU y la tele de tubo bailaban al mismo ritmo… y qué pasaba cuando perdían el paso.

---

## Índice

1. [Los tres protagonistas](#1-los-tres-protagonistas)
2. [La tele de tubo y el barrido](#2-la-tele-de-tubo-y-el-barrido)
   - [2.1 Scanlines](#21-scanlines)
   - [2.2 NTSC vs. PAL](#22-ntsc-vs-pal)
   - [2.3 Blanking](#23-blanking)
3. [La CPU](#3-la-cpu)
   - [3.1 Program Counter y ciclos](#31-program-counter-y-ciclos)
4. [La PPU](#4-la-ppu)
5. [Coordinación: el reloj maestro](#5-coordinación-el-reloj-maestro)
6. [Comunicación CPU ↔ PPU](#6-comunicación-cpu--ppu)
   - [6.1 Registros de PPU](#61-registros-de-ppu)
   - [6.2 Shadow OAM y OAM DMA](#62-shadow-oam-y-oam-dma)
   - [6.3 La regla de oro: actualizar en VBlank](#63-la-regla-de-oro-actualizar-en-vblank)
7. [La NMI — Non-Maskable Interrupt](#7-la-nmi--non-maskable-interrupt)
8. [El bucle de juego ideal](#8-el-bucle-de-juego-ideal)
9. [El slowdown: cuando el bucle falla](#9-el-slowdown-cuando-el-bucle-falla)
   - [9.1 La pérdida de la NMI](#91-la-pérdida-de-la-nmi)
   - [9.2 Frame rate atado a la lógica](#92-frame-rate-atado-a-la-lógica)
   - [9.3 Delta Time](#93-delta-time)
10. [Estrategias de los desarrolladores](#10-estrategias-de-los-desarrolladores)
    - [10.1 Ghosts 'n Goblins: actualización parcial](#101-ghosts-n-goblins-actualización-parcial)
    - [10.2 Teenage Mutant Ninja Turtles: 30 fps fijos](#102-teenage-mutant-ninja-turtles-30-fps-fijos)
11. [El problema de PAL](#11-el-problema-de-pal)
12. [Conclusión](#12-conclusión)

---

## 1. Los tres protagonistas

```mermaid
flowchart LR
    CPU["CPU<br/>1.79 MHz<br/>Ejecuta el código<br/>del juego"]
    PPU["PPU<br/>5.37 MHz<br/>Genera la imagen"]
    CRT["Tele CRT<br/>Barrido imparable<br/>60 o 50 Hz"]
    
    CPU -->|"Registros<br/>(tablero de control)"| PPU
    PPU -->|"Señal de video"| CRT
    CRT -->|"NTSC/PAL"| PPU
```

| Componente | Rol | Frecuencia |
|---|---|---|
| **CPU** | Ejecuta el código del juego (lógica, físicas, colisiones, decisiones gráficas) | **1.79 MHz** |
| **PPU** | Genera la señal de video usando las memorias gráficas (tile sets, OAM, name tables) | **5.37 MHz** |
| **CRT** | Muestra la imagen mediante barrido de electrones; impone el ritmo | **60 Hz (NTSC)** / 50 Hz (PAL) |

---

## 2. La tele de tubo y el barrido

### 2.1 Scanlines

La tele de tubo no muestra una imagen completa de golpe. Un **rayo de electrones** barre la pantalla de izquierda a derecha, línea por línea:

```mermaid
flowchart LR
    A["Inicio ← rayo encendido<br/>Scanline 1"] --> B["..." ]
    B --> C["Scanline 240"]
    C --> D["Rayo apagado<br/>→ vuelve arriba<br/>(VBlank)"]
    D --> A
```

Cada línea horizontal se llama **scanline** (o línea de dibujado). Al terminar un scanline, el rayo se apaga brevemente y se reposiciona al inicio de la siguiente (**HBlank**). Al terminar el último scanline, se apaga por un periodo más largo y vuelve arriba (**VBlank**).

### 2.2 NTSC vs. PAL

| Estándar | Frecuencia | Regiones |
|---|---|---|
| **NTSC** | ~**60 Hz** (≈60 fps) | Japón, Estados Unidos |
| **PAL** | **50 Hz** (50 fps) | Europa, Argentina |

La NES original (Japón/USA) estaba diseñada para NTSC.

### 2.3 Blanking

```mermaid
flowchart TD
    subgraph "Un frame (≈16.67 ms a 60 Hz)"
        DRAW["DIBUJO ACTIVO<br/>~240 scanlines<br/>El rayo pinta la imagen"]
        HBLANK["HBlank<br/>entre scanlines<br/>~corto"]
        VBLANK["VBLANK<br/>entre frames<br/>~largo"]
    end
    
    DRAW --> HBLANK
    HBLANK --> DRAW
    DRAW --> VBLANK
    VBLANK --> DRAW
```

| Periodo | Ocurre | Duración (CPU) | Utilidad |
|---|---|---|---|
| **HBlank** | Entre scanlines | Corto | Reposicionar rayo |
| **VBlank** | Entre frames | **2,273 ciclos de CPU** | **Actualizar memoria de PPU** |

El VBlank es la **ventana de tiempo** donde la CPU puede actualizar las memorias de la PPU sin romper la imagen.

---

## 3. La CPU

La CPU ejecuta el código del juego almacenado en la **PRG-ROM** del cartucho. Son instrucciones simples (sumas, escrituras en memoria) ejecutadas muy rápido.

```mermaid
flowchart LR
    subgraph "CPU ejecuta"
        PC[Program Counter<br/>→ instrucción actual]
        CLK[Señal de reloj<br/>→ avanza un ciclo]
    end
    
    PRG[PRG-ROM<br/>Código del juego] --> PC
    CLK --> PC
    PC --> R1[Ejecutar<br/>instrucción]
    R1 --> R2[Incrementar<br/>Program Counter]
    R2 --> R1
```

### 3.1 Program Counter y ciclos

- Cada ciclo de CPU ejecuta **parte de una instrucción** (las instrucciones llevan varios ciclos)
- La CPU avanza cuando recibe la **señal de reloj**
- Frecuencia de CPU: **1.79 MHz** (1,790,000 ciclos por segundo)

---

## 4. La PPU

La PPU genera la señal de video. En **cada ciclo de PPU** (llamado **dot**) pinta **un píxel** en pantalla.

> Frecuencia PPU: 5.37 MHz → 5,370,000 dots por segundo

---

## 5. Coordinación: el reloj maestro

La NES tiene un **reloj maestro** que corre a **21.47727 MHz**. De ahí se derivan todas las frecuencias:

```mermaid
flowchart TD
    MASTER["Reloj Maestro<br/>21.47727 MHz"] 
    
    MASTER -->|"÷ 4"| PPUCLK["PPU: 5.37 MHz<br/>1 ciclo = 1 dot = 1 píxel"]
    MASTER -->|"÷ 12"| CPUCLK["CPU: 1.79 MHz<br/>1 ciclo = 3 dots PPU"]
    
    PPUCLK --> RELACION["Relación fija:<br/>3 ciclos PPU<br/>por cada ciclo CPU"]
```

| Chip | División | Frecuencia |
|---|---|---|
| Reloj maestro | — | **21.47727 MHz** |
| PPU | ÷ 4 | **5.37 MHz** (1 dot = 1 píxel) |
| CPU | ÷ 12 | **1.79 MHz** |

**Cada ciclo de CPU ocurre exactamente cada 3 dots de PPU.** Esta relación constante permite una coreografía a nivel de píxel entre ambos chips. En el emulador Mesen se puede ver cómo los dots se dibujan en grupos de 3.

---

## 6. Comunicación CPU ↔ PPU

CPU y PPU tienen **mundos de memoria separados**. La CPU no accede directamente a las memorias de la PPU. En su lugar, usa **registros de PPU** expuestos en su propio mapa de direcciones — como un tablero de control.

```mermaid
flowchart LR
    subgraph "Mapa de CPU"
        WRAM[W-RAM<br/>2 KB]
        REGS["Registros de PPU<br/>(tablero de control)"]
    end
    
    subgraph "Mapa de PPU"
        VRAM["V-RAM + Name Tables"]
        OAM[OAM interna]
        PALS[Paletas]
    end
    
    REGS -->|"Lee/escribe<br/>a través de registros"| VRAM
    REGS --> OAM
    REGS --> PALS
```

### 6.1 Registros de PPU

| Registro | Función |
|---|---|
| **PPU Address** + **PPU Data** | Leer/escribir en el mapa de direcciones de la PPU (VRAM, name tables, paletas) |
| **OAM Address** + **OAM Data** | Leer/escribir en la OAM (sprite por sprite) |
| **OAM DMA** | Copiar la **OAM entera** desde la W-RAM de la CPU |
| **PPU Scroll** | Actualizar el offset de scrolling |
| **PPU Mask** | Activar/desactivar dibujado de sprites y fondo |
| **PPU Control** | Configurar NMI, modo de sprites, etc. |

### 6.2 Shadow OAM y OAM DMA

Como los sprites se mueven constantemente, la CPU mantiene una **copia exacta de la OAM en la W-RAM** (llamada **Shadow OAM**):

```mermaid
flowchart LR
    subgraph "CPU (W-RAM)"
        SHADOW["Shadow OAM<br/>(copia de 256 bytes)"]
        LOGICA[Lógica del juego<br/>actualiza posiciones<br/>en Shadow OAM]
    end
    
    subgraph "PPU"
        OAM_REAL["OAM real<br/>256 bytes"]
    end
    
    LOGICA --> SHADOW
    SHADOW -->|"OAM DMA<br/>(513 ciclos CPU)"| OAM_REAL
```

El registro **OAM DMA** permite copiar los 256 bytes de la Shadow OAM a la OAM real en **513 ciclos de CPU** (~15 scanlines). Esta copia debe hacerse durante el VBlank.

### 6.3 La regla de oro: actualizar en VBlank

```mermaid
flowchart LR
    subgraph "DURANTE el dibujo"
        MAL["Actualizar registro PPU<br/>en medio del frame"]
        MAL --> ROTO["Imagen partida:<br/>arriba en una posición,<br/>abajo en otra"]
    end
    
    subgraph "DURANTE VBlank"
        BIEN["Actualizar registros<br/>en VBlank"]
        BIEN --> LIMPIO["Frame completo y<br/>coherente"]
    end
```

Si la CPU actualiza el scroll o la OAM en medio de un scanline, la imagen se **parte** — la mitad superior se dibuja con un estado y la inferior con otro.

---

## 7. La NMI — Non-Maskable Interrupt

La PPU necesita avisarle a la CPU que el VBlank está por empezar. Usa una **interrupción**: la **NMI** (Non-Maskable Interrupt).

```mermaid
flowchart TD
    PPU_DRAW["PPU termina de dibujar<br/>el último scanline"] --> PPU_NMI["PPU activa<br/>la señal NMI"]
    PPU_NMI --> CPU_SAVE["CPU guarda estado<br/>y salta a la<br/>rutina NMI"]
    CPU_SAVE --> CPU_NMI["CPU ejecuta<br/>rutina NMI:<br/>actualizar scroll,<br/>OAM, tiles, paletas"]
    CPU_NMI --> CPU_RTI["Instrucción RTI<br/>(Return from Interrupt)"]
    CPU_RTI --> CPU_BACK["CPU vuelve a<br/>donde estaba antes"]
```

```
Sin NMI:
  PPU dibuja → PPU dibuja → PPU dibuja → ... (la CPU nunca se entera del VBlank)

Con NMI:
  PPU dibuja → ¡VBlank! → PPU: "NMI!" → CPU: "Actualizo gráficos" → PPU dibuja siguiente frame
```

La NMI se puede **desactivar** mediante el registro **PPU Control**. ¿Para qué querría un desarrollador desactivarla? Lo veremos en el bucle de juego.

---

## 8. El bucle de juego ideal

```mermaid
sequenceDiagram
    participant CPU as CPU
    participant PPU as PPU
    
    Note over CPU,PPU: Frame N (dibujándose)
    PPU->>PPU: Dibuja scanlines...
    CPU->>CPU: Prepara Frame N+1<br/>(lógica, físicas,<br/>escribe Shadow OAM)
    
    Note over PPU: Fin de Frame N
    PPU->>CPU: NMI (VBlank!)
    CPU->>CPU: Desactiva NMI<br/>(para que no interrumpa<br/>la preparación del frame)
    CPU->>CPU: Actualiza PPU:<br/>OAM DMA, scroll, tiles
    CPU->>CPU: Reactiva NMI
    CPU->>CPU: Espera próxima NMI
    
    Note over CPU,PPU: Frame N+1 (dibujándose)
    PPU->>PPU: Dibuja con datos actualizados
    CPU->>CPU: Prepara Frame N+2...
```

**Flujo ideal (60 fps):**

1. La PPU dibuja el Frame N (toma ~16.67 ms)
2. **Mientras tanto**, la CPU prepara el Frame N+1: ejecuta lógica de juego, escribe posiciones en la Shadow OAM, prepara actualizaciones de tiles
3. Al terminar el dibujo, la PPU entra en VBlank y **dispara la NMI**
4. La CPU **desactiva la NMI** (para que no haya interrupciones durante la preparación del frame)
5. La CPU copia los datos preparados a la PPU (OAM DMA, tiles, scroll, paletas)
6. La CPU **reactiva la NMI** y se pone en espera hasta la próxima
7. La PPU dibuja el Frame N+1 con los datos actualizados

La CPU apaga la NMI mientras prepara el frame para que el VBlank del frame anterior no interrumpa la preparación. La vuelve a activar **solo cuando ya tiene todo listo**.

---

## 9. El slowdown: cuando el bucle falla

### 9.1 La pérdida de la NMI

```mermaid
sequenceDiagram
    participant CPU as CPU
    participant PPU as PPU
    
    Note over CPU,PPU: Frame N
    PPU->>PPU: Dibujando...
    CPU->>CPU: Preparando Frame N+1<br/>Lógica de enemigos<br/>Colisiones...
    
    Note right of CPU: ¡Hay demasiados enemigos!<br/>La lógica se alarga.
    
    PPU->>CPU: NMI (VBlank!)
    Note over CPU: CPU NO reacciona:<br/>todavía preparando,<br/>NMI desactivada
    
    PPU->>PPU: VBlank termina<br/>Sin actualizaciones
    PPU->>PPU: Redibuja Frame N<br/>(mismos datos)
    
    CPU->>CPU: Termina preparación
    CPU->>CPU: Reactiva NMI
    CPU->>CPU: Espera...
    
    PPU->>PPU: Dibuja Frame N (repetido)
    
    PPU->>CPU: NMI del Frame N+1
    CPU->>CPU: Ahora sí actualiza PPU
    
    Note over CPU,PPU: Se perdió un frame entero<br/>60 fps → 30 fps
```

**Cuando la CPU no termina de preparar el frame antes del VBlank:**

1. La CPU todavía tiene la NMI desactivada (sigue preparando)
2. La NMI se **pierde** — la PPU no recibe actualizaciones
3. La PPU redibuja el **mismo frame de antes** (los datos no cambiaron)
4. La CPU termina su preparación, reactiva la NMI y **espera un frame entero**
5. **El frame rate se reduce a la mitad: 60 → 30 fps**

### 9.2 Frame rate atado a la lógica

**Los juegos de la NES ataban la lógica al frame rate.** La posición de los personajes, el temporizador, todo se calculaba como "por frame", no como "por unidad de tiempo real":

```mermaid
flowchart LR
    subgraph "Lógica atada al frame (NES)"
        F1["Frame 1"] --> M1["Mario: posición X<br/>+1 pixel"]
        F2["Frame 2"] --> M2["Mario: posición X<br/>+1 pixel"]
        F3["Se pierde frame"] --> M3["Mario: NO avanza<br/>(frame perdido)"]
        F4["Frame 3"] --> M4["Mario: posición X<br/>+1 pixel"]
    end
    
    subgraph "En tiempo real"
        T["60 fps: 60 posiciones/segundo<br/>30 fps: 30 posiciones/segundo"]
        T --> SLOW["¡El juego se ralentiza!"]
    end
```

```mermaid
flowchart LR
    subgraph "60 fps (ideal)"
        POS60["Posición avanza 60 unidades/segundo<br/>Mario llega en 667 ms"]
    end
    
    subgraph "30 fps (slowdown)"
        POS30["Posición avanza 30 unidades/segundo<br/>Mario llega en 1,334 ms<br/>→ EL DOBLE DE TIEMPO"]
    end
```

**Consecuencia:** el juego no solo se ve entrecortado, sino que **todo se mueve más lento** — personajes, enemigos, proyectiles, temporizadores.

### 9.3 Delta Time

Los juegos modernos usan un enfoque diferente: **Delta Time** — el tiempo real transcurrido entre frames.

```mermaid
flowchart LR
    subgraph "Sin Delta Time (NES)"
        S1["Mario.velocidad = 1 px/frame<br/>A 60 fps: 60 px/s<br/>A 30 fps: 30 px/s"]
        S1 --> SLOW2["Ralentización"]
    end
    
    subgraph "Con Delta Time (moderno)"
        D1["Mario.velocidad = 60 px/s<br/>Δt = tiempo entre frames<br/>A 60 fps: 60 × 0.0167 = 1 px/frame<br/>A 30 fps: 60 × 0.0333 = 2 px/frame"]
        D1 --> CONST["Velocidad constante<br/>→ sin ralentización"]
    end
```

| Método | 60 fps | 30 fps | Resultado |
|---|---|---|---|
| **Frame-based (NES)** | 60 px/s | 30 px/s | Ralentización |
| **Delta Time (moderno)** | 60 px/s | 60 px/s | **Velocidad constante** |

Con Delta Time, cuando el frame rate cae, el personaje avanza **el doble por frame** para compensar, manteniendo la velocidad real constante.

---

## 10. Estrategias de los desarrolladores

### 10.1 Ghosts 'n Goblins: actualización parcial

Ghosts 'n Goblins no actualizaba todo en VBlank. Actualizaba el scroll en un punto del frame y los sprites en otro, generando una imagen **parcialmente rota** pero funcional. El juego se ve "trabado" pero no es un slowdown clásico — son **actualizaciones escalonadas** por diseño.

```mermaid
flowchart LR
    A["Frame 1:<br/>Se actualiza scroll"] --> B["Frame 2:<br/>Se actualizan sprites"]
    B --> C["Frame 3:<br/>Se actualiza scroll"]
    C --> D["..." ]
```

### 10.2 Teenage Mutant Ninja Turtles: 30 fps fijos

TMNT sabía que la pantalla se llenaría de enemigos y el bucle colapsaría. En lugar de lidiar con slowdowns impredecibles, **apuntaba directamente a 30 fps**:

```mermaid
flowchart LR
    A[Frame 1<br/>procesado] --> B[Frame 2<br/>REPETIDO<br/>sin cambios]
    B --> C[Frame 3<br/>procesado]
    C --> D[Frame 4<br/>REPETIDO]
```

Al repetir siempre un frame intermedio, la carga se distribuía en **dos ciclos de CPU** y se garantizaba que nunca se cayeran más frames de la cuenta. La experiencia era consistente, aunque a mitad de velocidad.

---

## 11. El problema de PAL

Las regiones PAL corren a **50 Hz** (50 fps). Si un juego desarrollado para NTSC (60 fps) se ejecuta en una consola PAL sin adaptar:

```mermaid
flowchart LR
    NTSC["Juego NTSC<br/>60 fps<br/>Mario: 60 px/s"] --> PAL_RAW["Ejecutado en PAL<br/>50 fps<br/>Mario: 50 px/s"]
    PAL_RAW --> LENTO["¡16% más lento!"]
```

| Región | Frecuencia | Velocidad del juego |
|---|---|---|
| NTSC (Japón/USA) | 60 Hz | 100% (original) |
| PAL (Europa) sin adaptar | 50 Hz | **~83%** (16% más lento) |
| PAL con adaptación de código | 50 Hz | 100% (movimiento ajustado por frame) |

La corrección requería **modificar el código del juego** para que moviera más distancia por frame. También se cambiaba el reloj maestro y se usaban variantes de PPU y CPU para PAL.

El hardware se adaptaba con un cristal de reloj diferente y cambios en la división de frecuencia:

```
NTSC: Reloj maestro 21.47727 MHz → CPU ÷ 12 = 1.79 MHz
PAL:  Reloj maestro 21.28137 MHz → CPU ÷ 12 = 1.77 MHz (ligeramente distinto)
```

---

## 12. Conclusión

```mermaid
flowchart TD
    MASTER[Reloj Maestro 21.47 MHz] --> CPU_F["CPU: 1.79 MHz<br/>Ejecuta lógica"]
    MASTER --> PPU_F["PPU: 5.37 MHz<br/>Dibuja píxeles"]
    
    CPU_F --> PREP[Prepara frame N+1<br/>durante dibujo de frame N]
    PPU_F --> DRAW[Dibuja frame N]
    
    DRAW --> VBLANK[VBlank]
    VBLANK --> NMI[NMI → CPU actualiza PPU]
    
    NMI --> OK[60 fps<br/>Bucle perfecto]
    
    PREP --> SLOW_CAUSE["Lógica se alarga<br/>(muchos enemigos,<br/>cálculos complejos)"]
    SLOW_CAUSE --> NMI_MISS["NMI perdida<br/>Frame repetido"]
    NMI_MISS --> SLOW["30 fps<br/>Juego ralentizado"]
    
    SLOW --> FRAME_BASED["Lógica atada al frame<br/>(no a tiempo real)"]
    FRAME_BASED --> RALENTI["Todo se mueve<br/>más lento"]
```

### Resumen

| Concepto | Clave |
|---|---|
| **Coordinación** | Reloj maestro único → CPU y PPU sincronizadas a nivel de píxel (3:1) |
| **VBlank** | Única ventana segura para actualizar memoria de PPU (2,273 ciclos CPU) |
| **NMI** | Interrupción que avisa a la CPU que empezó el VBlank |
| **Slowdown** | CPU no termina preparación → se pierde NMI → frame repetido → 30 fps |
| **Frame-based** | Lógica atada a frames, no a tiempo real → velocidad depende del frame rate |
| **Delta Time** | Solución moderna que desacopla velocidad del frame rate |
| **PAL** | 50 Hz vs 60 Hz → juegos NTSC corren 16% más lento sin adaptación |
| **Estrategias** | Ghosts 'n Goblins (actualización parcial), TMNT (30 fps fijos) |

---

> **Fuente:** Transcripción del video *"La NES a FONDO - Parte 4: El bucle de Juego y el SLOWDOWN"* (YouTube)
