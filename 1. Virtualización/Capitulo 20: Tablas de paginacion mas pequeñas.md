# Tablas de paginación más pequeñas
Un gran problema de la paginación lineal es que las **Page Tables** ocupan demasiado espacio en memoria RAM, ya que cada proceso requiere su propia tabla independiente y estas deben mapear todo el espacio de direcciones virtuales, incluyendo las regiones vacías o inválidas.

## Alternativas Ineficientes

- **Aumentar el tamaño de página:** Si se usan páginas más grandes, se reduce el tamaño de la Page Table, pero se genera un enorme desperdicio de memoria dentro de las páginas asignadas a procesos pequeños, provocando **fragmentación interna**.
- **Combinar paginación con segmentación (Tablas por segmento):** Crear una Page Table por cada segmento activo del programa (_code, heap, stack_) en lugar de una sola lineal reduce el espacio muerto. Sin embargo, asume un patrón fijo de uso y genera **fragmentación externa**, ya que las Page Tables adquieren tamaños arbitrarios y es difícil encontrar bloques libres contiguos en la RAM para almacenarlas.

## La Solución: Page Tables Multinivel

Este enfoque convierte la estructura lineal en una jerarquía similar a un árbol ramificado. En lugar de mantener una única tabla gigante, **la Page Table se segmenta en unidades del tamaño de una página física**. Si una región completa de entradas virtuales está vacía o es inválida, esa porción de la Page Table simplemente **no se asigna en la memoria RAM**.

Para coordinar este árbol, se introducen los siguientes componentes:

- **Page Directory (Directorio de Páginas):** Actúa como una "meta-tabla de páginas". Sus entradas (**PDE - Page Directory Entries**) apuntan directamente a la base de las diferentes porciones válidas de la Page Table.
- **PDE (Page Directory Entry):** Contiene un _valid bit_ (vale 1 si al menos una página de la Page Table subordinada contiene traducciones válidas) y un _present bit_ (indica si esa porción está cargada en la RAM).
- **PDBR / CR3:** Registro físico de control del procesador que almacena la dirección base donde arranca el Page Directory del proceso activo actual. Debe estar alineado a saltos de 4KB (por lo que sus últimos 12 bits siempre se leen como 0 en hexadecimal).

### El Gran Trade-off: Memoria vs. Tiempo

- **Ventaja:** Excelente economía de memoria física; solo se consume RAM para almacenar mapeos de las regiones que el proceso realmente está utilizando. Además, las porciones de las tablas caben exactamente en una página, facilitando la asignación al SO.
- **Desventaja:** Añade niveles de indirección. Ante un **TLB miss**, el hardware debe realizar múltiples accesos consecutivos a la memoria RAM (primero lee el Page Directory, luego la Page Table, y finalmente el dato) duplicando o triplicando la penalización de tiempo de espera del CPU.

## Jerarquías de Más Niveles y Variaciones de Arquitectura

Cuando el espacio de direcciones crece, un solo nivel de directorio se vuelve demasiado grande para entrar en una sola página. La solución es añadir más niveles jerárquicos (Page Directories superiores que apuntan a Page Directories inferiores).

### Esquemas Clásicos de División de Bits (Dirección Virtual de 32/64 bits)

#### 1. x86 Estándar de 32 bits (Esquema `10 / 10 / 12`)

Diseñado para páginas de 4 KiB con entradas (PTE) de 4 bytes (32 bits):

- **Bits 31-22 (10 bits):** Índice del Page Directory (mapea $2^{10} = 1024$ entradas).
- **Bits 21-12 (10 bits):** Índice de la Page Table (mapea 1024 entradas).
- **Bits 11-0 (12 bits):** Offset dentro de la página física ($2^{12} = 4\text{ KB}$).

#### 2. x86 con PAE (Physical Address Extension) (Esquema `2 / 9 / 9 / 12`)

Mantiene el espacio virtual en 32 bits pero expande las entradas de tabla a 64 bits (8 bytes) para poder direccionar más de 4 GB de RAM física. Al agrandar las entradas, caben menos por página, por lo que se introduce un tercer nivel de jerarquía para fragmentar los índices (`9` bits por tabla en lugar de `10`).

#### 3. RISC-V Sv39 (Esquema `9 / 9 / 9 / 12`)

Estructura nativa para punteros de 64 bits (donde se usan efectivamente los 39 bits más bajos). Las entradas de tabla ocupan 8 bytes (64 bits) y el Page Directory maneja una arquitectura de tres niveles de 9 bits cada uno junto al offset clásico de 12 bits.

## Soporte de Almacenamiento: Swapping al Disco

Incluso con el ahorro multinivel, en sistemas con miles de procesos concurrentes las tablas pueden saturar la memoria RAM. Para mitigar esto, los sistemas operativos modernos permiten colocar ciertas porciones de las Page Tables en la memoria virtual del kernel. Si una porción requerida fue trasladada (**swapped**) al disco duro, el acceso fallará disparando un **Page Fault**, obligando al sistema a pausar el proceso mientras recupera la tabla desde el almacenamiento secundario.

Para complementar tu estudio y permitirte experimentar cómo se descomponen exactamente las direcciones de memoria en estas estructuras arbóreas según la arquitectura, puedes utilizar el siguiente simulador interactivo.

### 1. Arquitectura x86 Estándar (32 bits)

**Esquema de Paginación: 10 / 10 / 12** (Páginas de 4 KB)

|**Rango de Bits**|**Tamaño**|**Componente**|**Descripción**|
|---|---|---|---|
|**Bits 31 - 22**|10 bits|**Page Directory Index** (Directorio)|Índice que apunta a una de las $2^{10}$ (1024) entradas en el Page Directory.|
|**Bits 21 - 12**|10 bits|**Page Table Index** (Tabla)|Índice que apunta a una de las $2^{10}$ (1024) entradas en la Page Table seleccionada.|
|**Bits 11 - 0**|12 bits|**Page Offset** (Desplazamiento)|Indica el byte exacto dentro de la página física de 4 KB ($2^{12}$ = 4096 bytes).|

### 2. Arquitectura RISC-V Sv39 (64 bits)

**Esquema de Paginación: 9 / 9 / 9 / 12** (Páginas de 4 KB, jerarquía de 3 niveles)

|**Rango de Bits**|**Tamaño**|**Componente**|**Descripción**|
|---|---|---|---|
|**Bits 63 - 39**|25 bits|**EXT** (Extensión / Reservado)|Bits no utilizados para la traducción en Sv39. El hardware exige que sean iguales al bit 38 (extensión de signo).|
|**Bits 38 - 30**|9 bits|**VPN[2]** (Directorio Nivel 2)|Índice del primer nivel del árbol. Apunta a una de las $2^9$ (512) entradas.|
|**Bits 29 - 21**|9 bits|**VPN[1]** (Directorio Nivel 1)|Índice del nivel intermedio. Apunta a 512 entradas posibles.|
|**Bits 20 - 12**|9 bits|**VPN[0]** (Page Table)|Índice del nivel más bajo (Tabla de Páginas). Apunta a las últimas 512 entradas que dan el PFN final.|
|**Bits 11 - 0**|12 bits|**Page Offset** (Desplazamiento)|Indica el byte exacto dentro de la página física de 4 KB ($2^{12}$ = 4096 bytes).|
