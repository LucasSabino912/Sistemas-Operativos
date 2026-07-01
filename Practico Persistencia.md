**Ejercicio 1. El disco Seagate Exos 7E8 de 8 TiB e intefaz SAS tiene una velocidad de rotación de 7200 RPM, 4.16 ms de latencia de búsqueda y 215 MiB/s de tasa de transferencia.**

**Datos:**

Velocidad de rotación (RPM): 7200RPM

Latencia de búsqueda: 4.16ms

Tasa de transferencia: 215MiB/s

Tamaño de bloque: 4096 bytes (4KiB)

**(a) Indicar cuantos ms tarda en dar una vuelta completa.**

**Fórmula:**

$\text{Tiempo por vuelta (ms)} = \frac{60 \times 1000}{\text{RPM}}$

$\text{Tiempo por vuelta} = \frac{60000}{7200} = 8.333 \text{ ms}$

**Explicación:** Se convierten los minutos a milisegundos (60 segundos * 1000 ms) y se divide por la cantidad de revoluciones.

**(b) Indicar la tasa de transferencia de lectura al azar de bloques de 4096 bytes.**

En la lectura al azar, el tiempo total para acceder a un bloque no es solo la transferencia de datos, sino la suma de la latencia de búsqueda (seek), la latencia rotacional y el tiempo de transferencia real de los bytes.

**Fórmula del Tiempo Total de Acceso ($T_{access}$):**

$T_{access} = T_{seek} + T_{rotation\_avg} + T_{transfer}$

**Cálculos:**

- $T_{seek} = 4.16 \text{ ms}$ (Dato del problema).
    
- $T_{rotation\_avg} = \frac{\text{Tiempo por vuelta}}{2} = \frac{8.333}{2} = 4.166 \text{ ms}$.
    
- $T_{transfer} = \frac{\text{Tamaño del bloque}}{\text{Tasa de transferencia}}$. Para esto pasamos 215 MiB/s a bytes/ms: $\frac{215 \times 1024 \times 1024 \text{ bytes}}{1000 \text{ ms}} = 225443.84 \text{ bytes/ms}$.
    
    $T_{transfer} = \frac{4096 \text{ bytes}}{225443.84 \text{ bytes/ms}} \approx 0.018 \text{ ms}$.
    
- $T_{access} = 4.16 + 4.166 + 0.018 = 8.344 \text{ ms}$.
    

**Fórmula de Tasa de Transferencia al Azar ($R_{random}$):**

$R_{random} = \frac{\text{Tamaño de bloque}}{T_{access}}$

$R_{random} = \frac{4096 \text{ bytes}}{0.008344 \text{ segundos}} \approx 490891 \text{ bytes/s} \approx 479 \text{ KiB/s}$

**Explicación:** La tasa efectiva cae drásticamente (de 215 MiB/s a casi 0.5 MiB/s) porque la mayor parte del tiempo el disco se la pasa buscando mecánicamente el bloque en lugar de transferir datos.

**Ejercicio 2. Compute el tamaño de la FAT para:**

**Fórmula general:** $\text{Tamaño FAT} = \text{Cantidad de clusters} \times \text{Tamaño de entrada FAT}$

$\text{Cantidad de clusters} = \frac{\text{Tamaño del disco}}{\text{Tamaño del cluster}}$

**(a) Un diskette de doble cara doble densidad 360 KiB (∼1982): FAT12, cluster de 512 bytes.**

- Cantidad de clusters = $\frac{360 \text{ KiB}}{512 \text{ bytes}} = \frac{360 \times 1024}{512} = 720 \text{ clusters}$.
    
- FAT12 usa 12 bits por entrada, es decir, 1.5 bytes.
    
- **Tamaño FAT** = $720 \times 1.5 \text{ bytes} = 1080 \text{ bytes} \approx 1.05 \text{ KiB}$.
    

**(b) Un disco duro de 4 GiB (∼1998): FAT16, cluster de 4096 bytes.**

- Cantidad de clusters matemáticos = $\frac{4 \text{ GiB}}{4 \text{ KiB}} = \frac{2^{32}}{2^{12}} = 2^{20} = 1048576 \text{ clusters}$.
    
- FAT16 usa 16 bits por entrada, es decir, 2 bytes.
    
- **Tamaño FAT (teórico)** = $2^{20} \times 2 \text{ bytes} = 2^{21} \text{ bytes} = 2 \text{ MiB}$.
    
    _(Nota de concepto: En la realidad, FAT16 no soporta $2^{20}$ clusters, su límite es $2^{16}$. Para un disco de 4 GiB en FAT16 se requerían clusters gigantes de 64 KiB, pero resolviendo con los datos exactos del problema, el cálculo matemático arroja 2 MiB)._
    

**(c) Un pendrive 32 GiB (∼2014): FAT32, cluster de 16384 bytes.**

- Cantidad de clusters = $\frac{32 \text{ GiB}}{16 \text{ KiB}} = \frac{2^{35} \text{ bytes}}{2^{14} \text{ bytes}} = 2^{21} \text{ clusters}$.
    
- FAT32 usa 32 bits por entrada, es decir, 4 bytes.
    
- **Tamaño FAT** = $2^{21} \times 4 \text{ bytes} = 2^{23} \text{ bytes} = 8 \text{ MiB}$.
    

**Ejercicio 3. El sistema de archivos de xv6 es una estructura a la Fast-Filesystem for UNIX (UFS), con parámetros: bloque de 512 bytes, 12 bloques directos, 1 bloque indirecto, índices de bloque de 32 bits.**

**(a) Calcule el tamaño máximo de un archivo.**

**Datos y Fórmula:**

- Índices de bloque de 32 bits = 4 bytes.
    
- Punteros por bloque indirecto = $\frac{512 \text{ bytes}}{4 \text{ bytes}} = 128 \text{ punteros}$.
    
- Total de bloques direccionables por i-nodo = $12 \text{ (directos)} + 128 \text{ (indirectos)} = 140 \text{ bloques}$.
    
- **Tamaño máximo** = $140 \text{ bloques} \times 512 \text{ bytes/bloque} = 71680 \text{ bytes} = 70 \text{ KiB}$.
    

**(b) Calcule el tamaño de la sobrecarga para un archivo de tamaño máximo.**

**Explicación:** La "sobrecarga" (overhead) es el espacio utilizado para guardar metadatos en lugar de los datos reales del usuario (excluyendo el i-nodo en sí, nos enfocamos en bloques extra).

Para acceder a los 128 bloques finales, se debe instanciar 1 bloque indirecto completo.

- **Sobrecarga** = 1 bloque = 512 bytes.
    

**(c) ¿Se podrían codificar los números de bloque con menos bits? ¿Qué otros efectos produciría utilizar la mínima cantidad de bits?**

**Explicación:** Sí, se podrían usar menos bits. Por ejemplo, si todo el disco de xv6 es muy pequeño, quizás alcancen 16 bits (2 bytes) para direccionar todos los bloques.

_Efectos de reducir los bits:_

1. **Aumenta el tamaño máximo del archivo:** Al ocupar menos bytes cada índice, entrarían más punteros en el bloque indirecto (ej. si fuesen 16 bits, entrarían 256 punteros en vez de 128).
    
2. **Disminuye el tamaño máximo del disco soportado:** Si usas 16 bits, solo puedes direccionar un máximo de $2^{16} = 65536$ bloques en todo el disco ($65536 \times 512 = 32 \text{ MiB}$ de partición máxima).
    

**Ejercicio 4. Para un sistema de archivos xv6: bloque de 512 bytes, 12 bloques directos, 1 bloque indirecto, índices de bloque de 32 bits.**

**(a) Indique que número de bloque hay que leer para acceder al byte 451, al byte 6200 y al byte 71000.**

**Fórmula:** $\text{Número de bloque lógico del archivo} = \lfloor \frac{\text{Byte}}{512} \rfloor$.

- **Byte 451:** $\lfloor 451 / 512 \rfloor = 0$. Corresponde al **bloque directo 0**.
    
- **Byte 6200:** $\lfloor 6200 / 512 \rfloor = 12$. Los directos van del 0 al 11. El bloque 12 cae en el nivel indirecto. Hay que acceder al bloque apuntado por el **índice 0 del bloque indirecto** ($12 - 12 = 0$).
    
- **Byte 71000:** $\lfloor 71000 / 512 \rfloor = 138$. Está en el nivel indirecto. Su posición dentro del bloque indirecto es el índice **126** ($138 - 12 = 126$).
    

**(b) Dar la expresión matemática que indica a partir del byte que número de bloque hay que acceder. Se puede usar división por casos, ejemplo positivo(x) = 1 si 0 < x, 0 si x ≤ 0.**

**Expresión:**

Sea $N = \lfloor \frac{\text{Byte}}{512} \rfloor$ el bloque lógico del archivo.

$$\text{Acceso} = \begin{cases} \text{Puntero directo } N & \text{si } 0 \le N \le 11 \\ \text{Entrada } (N - 12) \text{ del bloque indirecto} & \text{si } 12 \le N \le 139 \\ \text{Error (Out of bounds)} & \text{si } N \ge 140 \end{cases}$$

**Ejercicio 5. Considere un disco de 16 TiB ($2^{44}$ bytes) con bloques de 4 KiB ($2^{12}$ bytes) e índices de bloque de 32 bits. Suponga que el sistema de archivos esta organizado con i-nodos de hasta 3 niveles de indirección y hay 8 punteros a bloques directos. ¿Cuál es el máximo tamaño de archivo soportado por este sistema de archivos? Justifique su respuesta.**

**Datos y Cálculos:**

- Tamaño del bloque = 4096 bytes ($2^{12}$).
    
- Tamaño del índice = 4 bytes ($2^2$).
    
- Punteros por bloque = $\frac{2^{12}}{2^2} = 2^{10} = 1024$ punteros.
    
- **Capacidad de punteros por nivel:**
    
    - Directos: 8 bloques.
        
    - Indirecto simple: $1024$ bloques.
        
    - Indirecto doble: $1024 \times 1024 = 1048576$ bloques ($2^{20}$).
        
    - Indirecto triple: $1024 \times 1024 \times 1024 = 1073741824$ bloques ($2^{30}$).
        
- **Bloques totales de un archivo:** $8 + 2^{10} + 2^{20} + 2^{30}$ bloques.
    
- **Tamaño máximo:** $(8 + 2^{10} + 2^{20} + 2^{30}) \times 4096 \text{ bytes}$.
    
    - Directos: 32 KiB
        
    - Indirectos: 4 MiB
        
    - Dobles: 4 GiB
        
    - Triples: 4 TiB
        
        **Respuesta:** El máximo tamaño es de **$\approx 4.004$ TiB** ($32 \text{ KiB} + 4 \text{ MiB} + 4 \text{ GiB} + 4 \text{ TiB}$). Se justifica porque la sumatoria de todas las vías de acceso a bloque por el tamaño del bloque nos da el límite teórico de datos que puede apuntar un solo i-nodo en este esquema.
        

**Ejercicio 6. Considere el sistema de archivos de xv6. Indique que bloques directos o indirectos hay que crear. Suponga que el i-nodo ya está creado en disco. Puede usar esquemas.**

Recordatorio xv6: 512 bytes por bloque, 12 directos, 1 indirecto.

**(a) Se crea un archivo nuevo y se agregan 6000 bytes**

- Cálculo de bloques necesarios: $\lceil \frac{6000}{512} \rceil = 12$ bloques lógicos.
    
- **Bloques a crear:** Se deben crear (asignar) exactamente **12 bloques de datos**. Estos ocuparán todos los punteros directos del i-nodo (posiciones del 0 al 11). No se crea ningún bloque indirecto.
    

**(b) Se agregan al final 1000 bytes más.**

- Tamaño total nuevo: $6000 + 1000 = 7000$ bytes.
    
- Total de bloques lógicos necesarios: $\lceil \frac{7000}{512} \rceil = 14$ bloques lógicos.
    
- Como ya teníamos 12 creados, faltan los bloques lógicos 12 y 13.
    
- **Bloques a crear:** Para los bloques lógicos >11, necesitamos usar el nivel indirecto. Por ende, se deben crear **3 bloques en disco**:
    
    1. **1 bloque indirecto** (metadatos) que guardará los punteros.
        
    2. **2 bloques de datos** (para guardar los 1000 bytes nuevos). Estos dos serán referenciados en los índices 0 y 1 del nuevo bloque indirecto.
        

**Ejercicio 7. Supongamos que se borró todo el mapa de bits con los bloques en uso de un filesystem tipo UNIX. Explique el procedimiento para recuperarlo. 1Cluster es la terminología Microsoft para block o bloque.**

**Explicación (Procedimiento de recuperación tipo fsck):**

1. Inicializar un nuevo bitmap en memoria vacío (todos los bits en 0/Libre).
    
2. Marcar en 1 (Usado) los bloques reservados del sistema: Superbloque, tabla de i-nodos, bootblock, etc.
    
3. Recorrer la Tabla de i-nodos desde el principio hasta el final.
    
4. Por cada i-nodo que esté marcado como "en uso", revisar todos sus punteros a bloques (directos, indirectos, indirectos dobles, etc.).
    
5. Por cada bloque de datos (o bloque de punteros) al que apunte un i-nodo válido, marcar el bit correspondiente en el nuevo bitmap como 1 (Usado).
    
6. Una vez procesados todos los i-nodos, volcar el bitmap reconstruido de memoria al disco.
    

**Ejercicio 8. Un programa que revisa la estructura del filesystem (fsck) con el método del ejercicio anterior construyó la siguiente tabla de bloques que aparecen en uso vs. bloques que se indican como libres en el bitmap del disco. In use: 1 0 1 0 0 1 0 1 1 0 1 0 0 1 0 Free: 0 0 0 1 1 1 0 0 0 1 0 1 1 0 1 ¿Hay errores? De ser así ¿resultan importantes? Ayuda: analice los 4 casos posibles y discuta cada uno.**

**Análisis de Casos:**

1. `(In use: 1, Free: 0)`: **Correcto.** Un archivo lo está usando y el bitmap lo marca como ocupado.
    
2. `(In use: 0, Free: 1)`: **Correcto.** Ningún archivo lo reclama y el bitmap lo marca libre.
    
3. `(In use: 0, Free: 0)`: **Error (Space Leak / Fuga de espacio).** El bloque no pertenece a nadie, pero el bitmap dice que está ocupado. _Importancia:_ Leve. El disco pierde capacidad inútilmente, pero no hay corrupción de datos. Se soluciona marcándolo como 1 en la lista Free. _(Vemos este error en la columna 2 y 7 de la tabla)._
    
4. `(In use: 1, Free: 1)`: **Error (Corrupción Fatal).** Un archivo está utilizando este bloque para sus datos, pero el bitmap dice que está libre. _Importancia:_ Crítica. Si el sistema operativo necesita crear un archivo nuevo, podría asignarle este bloque "libre" y sobrescribir los datos del archivo original, causando pérdida de información. _(Vemos este error en la columna 6)._
    
    **Conclusión:** Sí, hay errores. Hay dos "space leaks" (columna 2 y 7) y un error crítico de doble asignación inminente (columna 6). Los errores son sumamente importantes y el fsck debe corregir el bitmap obligatoriamente antes de montar el disco.
    

**Ejercicio 9. Suponiendo a) no hay nada en la caché de bloques de disco, b) cada consulta a un i-nodo genera exactamente una lectura de bloque, c) cada directorio entra en un bloque, d) cada archivo también entra en un bloque. Describa la secuencia de lecturas de bloque para acceder a los archivos:**

**Explicación:** Todo acceso comienza por el directorio raíz `/` (cuyo i-nodo suele estar en un bloque conocido).

**(a) /initrd.img**

1. Leer el bloque del i-nodo del directorio `/` (raíz).
    
2. Leer el bloque de datos del directorio `/` (para buscar el string "initrd.img" y obtener su nº de i-nodo).
    
3. Leer el bloque del i-nodo del archivo `initrd.img`.
    
4. Leer el bloque de datos de `initrd.img`.
    
    **Total: 4 lecturas de bloque.**
    

**(b) /usr/games/moon-buggy**

1. Leer el bloque del i-nodo del directorio `/`.
    
2. Leer el bloque de datos de `/` (buscar `usr`).
    
3. Leer el bloque del i-nodo del directorio `usr`.
    
4. Leer el bloque de datos de `usr` (buscar `games`).
    
5. Leer el bloque del i-nodo del directorio `games`.
    
6. Leer el bloque de datos de `games` (buscar `moon-buggy`).
    
7. Leer el bloque del i-nodo del archivo `moon-buggy`.
    
8. Leer el bloque de datos de `moon-buggy`.
    
    **Total: 8 lecturas de bloque.**
    

**Ejercicio 10. Un novato en diseño e implementación de sistemas de archivos sugiere que la primera parte del contenido de cada archivo UNIX se almacene en el i-nodo mismo. ¿Es una buena idea? Explique.**

**Explicación:** Sí, es una **excelente idea** y, de hecho, se utiliza en sistemas modernos (se conoce como _inline data_ en ext4 o _fast symlinks_). Un i-nodo tiene un tamaño fijo (ej. 256 bytes) y reserva gran parte de ese espacio para los punteros de bloque de datos. Si el archivo es sumamente pequeño (ej. un enlace simbólico de 40 bytes, o un archivo de configuración trivial), los datos caben directamente en el espacio de los punteros.

- **Ventaja:** Ahorra la asignación de un bloque de datos entero (evita fragmentación interna) y ahorra un _seek_ del disco, leyendo metadato y dato en una sola operación de I/O.
    

**Ejercicio 11. El campo link count dentro de un i-nodo resulta redundante. Todo lo que dice es cuantas entradas de directorios apuntan a ese i-nodo, y esto es algo que se puede calcular recorriendo el grafo de directorios. ¿Por qué se usa este campo?**

**Explicación:** Por **rendimiento (Performance)**. Si no existiera, cada vez que un usuario ejecutara un `rm` (unlink) para borrar un archivo, el Sistema Operativo tendría que recorrer _todo_ el grafo de directorios del disco entero para verificar si alguna otra entrada sigue apuntando a ese i-nodo antes de poder marcar sus bloques de datos como libres. Esto haría que borrar un archivo sea una operación $O(N)$ extremadamente lenta (requiriendo miles de lecturas de disco). Con el `link count`, borrar es $O(1)$ (solo se resta uno; si llega a 0, se libera instantáneamente).

**Ejercicio 12. Ya vimos que las estructuras de datos en disco de los FS mantienen información redundante que debería ser consistente. Piense en más pruebas de consistencia de debería realizar un filesystem checker en un sistema de archivos tipo:**

**(a) FAT.**

1. **Cadenas cruzadas (Cross-linking):** Verificar que dos archivos no apunten al mismo cluster en sus cadenas de la FAT.
    
2. **Ciclos:** Verificar que una cadena en la tabla FAT no termine apuntando a un cluster anterior de la misma cadena creando un bucle infinito.
    
3. **Coherencia de tamaño:** Que el tamaño del archivo reportado en la entrada de directorio coincida con la cantidad de clusters contabilizados en la cadena FAT.
    

**(b) UNIX.**

1. **Conteo de enlaces (Link Count):** Verificar que el `link_count` del i-nodo sea exactamente igual a la cantidad de entradas de directorios reales que lo apuntan en todo el árbol de directorios.
    
2. **Punteros fuera de rango:** Comprobar que los punteros a bloques dentro del i-nodo (directos e indirectos) no referencien a direcciones que están más allá del tamaño físico de la partición (out-of-bounds).
    
3. **Formatos de directorios:** Comprobar que el primer directorio `.` apunte a sí mismo y `..` a su padre correctamente.
    

**Ejercicio 13. Explique porque no se pueden realizar enlaces duros de directorios en un sistema de archivos UNIX. $ mkdir dir1 $ ln dir1 dir2 ln: ‘dir1’: hard link not allowed for directory Ayuda: pensar en el campo link count y en dos directorios que se autoreferencian.**

**Explicación:** Si se permiten enlaces duros a directorios, se pueden generar **ciclos (bucles cerrados)** en el grafo del sistema de archivos (dejando de ser un árbol estricto). Esto es peligroso porque los programas que recorren el disco de forma recursiva (como `find`, un cálculo de espacio con `du`, o el mismo `fsck`) entrarían en un bucle infinito al entrar en el directorio y volver a subir por el hardlink. Además, rompería la consistencia estructural a la hora de resolver rutas relativas como `..` (el sistema no sabría quién es el "padre verdadero").

**Ejercicio 14. Explique la diferencia entre un log-filesystem y un journaling-filesystem.**

- **Journaling-filesystem (Ej. ext3, ext4):** Trabaja bajo un modelo de actualización _in-place_ (sobreescribe los datos en su posición original del disco). La diferencia es que antes de realizar cambios estructurales, anota qué va a hacer en una pequeña libreta secuencial (el "journal" o diario). Una vez escrito el diario, procede a actualizar in-place. Si hay un corte de luz, revisa el diario y re-aplica (o cancela) las operaciones a medio hacer.
    
- **Log-structured filesystem (LFS):** Todo el disco se comporta como un journal gigante secuencial (log append-only). NUNCA sobreescribe un bloque in-place. Cuando modificas un archivo o sus metadatos, escribe la nueva versión completa al final del registro (log). Optimiza las escrituras haciéndolas puramente secuenciales. Requiere de un "Garbage Collector" continuo para ir liberando versiones viejas de bloques.
    

**Ejercicio 15. Enumere y explique brevemente las tres (3) capas en las que se estructura un sistema de archivos. Dar dos (2) ejemplos que muestran la conveniencia de esta división.**

**Explicación de las Capas:**

1. **Capa Lógica (API / VFS):** Provee la interfaz al usuario y a los programas (Syscalls como `open`, `read`, `write`). Maneja la tabla de archivos abiertos, abstracción de nombres y permisos.
    
2. **Capa Estructural / Organización (Gestión):** Es la "inteligencia" del FS particular. Conoce cómo mapear nombres lógicos y offsets de archivos a bloques lógicos utilizando las estructuras internas (como leer la FAT, atravesar i-nodos, punteros indirectos).
    
3. **Capa I/O (Driver de Bloques):** Se encarga de traducir los bloques lógicos de la capa 2 a sectores físicos reales y comunicarse con el hardware de almacenamiento usando comandos específicos (SCSI, SATA, NVMe).
    

**Ejemplos de conveniencia:**

1. **Portabilidad de Hardware:** Gracias a la separación de la capa 3, puedes agarrar un sistema de archivos ext4 (Capa 2) e instalarlo indistintamente en un disco magnético, un SSD o un RAID, sin tocar una sola línea de código estructural; solo cambia el Driver.
    
2. **Transparencia para el usuario (VFS):** Gracias a la Capa 1, un programa de usuario como un reproductor de música puede usar un simple `read()` sobre un MP3 alojado en un pendrive FAT32 o en el disco duro EXT4 sin que el programa tenga que saber cómo están organizados internamente.
    

**Ejercicio 16. ¿Verdadero o Falso? Explique.**

**(a) La entrada/salida programada (por interrupciones) y el DMA liberan a la CPU de hacer pooling a los dispositivos.**

**Verdadero.** El CPU delega la tarea de I/O, el dispositivo lo procesa asíncronamente y notifica mediante una interrupción, evitando que la CPU pierda ciclos "preguntando" si ya terminó (polling).

**(b) Los sistemas de archivos tipo UNIX tienen soporte especial para que cuando un directorio crece en cantidad de archivos y no entra más en un bloque, pida más bloques.**

**Verdadero.** Los directorios en UNIX no son mágicos, internamente son tratados simplemente como un "archivo" más, pero cuyo contenido son duplas [nombre, i-nodo]. Como son archivos, sus i-nodos pueden solicitar y asignar más bloques de datos para seguir guardando nuevas entradas.

**(c) Es posible establecer enlaces duros (hardlinks) entre archivos de diferentes particiones.**

**Falso.** Los enlaces duros apuntan directamente al _número de i-nodo_. Los números de i-nodo solo son únicos dentro de su propio sistema de archivos/partición. Si haces un hardlink entre particiones, apuntarías a un i-nodo equivocado en el otro disco. Para esto se usan _Soft links_ (enlaces simbólicos).

**(d) Los dispositivos de I/O se dividen en I/O mapped y memory mapped.**

**Verdadero.** Esas son las dos grandes arquitecturas para que la CPU acceda a los registros de control de los dispositivos: mediante instrucciones especiales dedicadas (`IN`/`OUT`) o mediante direcciones normales de la memoria RAM (Memory Mapped I/O).

**(e) Un disco duro en formato UFS2 puede no estar lleno y aún asi no poder almacenar más archivos.**

**Verdadero.** Esto ocurre si se agota el "pool" fijo de i-nodos (Inode exhaustion). Si creas millones de archivos de 1 byte, te quedarás sin estructuras de i-nodo disponibles, imposibilitando la creación de nuevos archivos a pesar de que haya gigabytes de bloques de datos libres en el disco.

**(f) El tamaño de un archivo no es un atributo realmente necesario en los i-nodos, se puede calcular a partir de los bloques ocupados (FAT o UFS).**

**Falso.** Los bloques de disco rara vez están 100% llenos (existe fragmentación interna). Si un archivo ocupa un bloque de 4096 bytes pero solo usa 5 bytes reales, sin el atributo de `tamaño`, el SO devolvería los otros 4091 bytes de basura (ceros o datos residuales) al leer el archivo. El tamaño exacto marca el EOF (End of File).

**(g) El uso de espacio en disco es siempre mayor o igual a la longitud del archivo.**

**Verdadero.** Debido a la asignación en bloques discretos (Ej. de 4 KiB), un archivo de 10 bytes obligatoriamente debe apropiarse un bloque entero (4096 bytes). Esto genera _fragmentación interna_.

**(h) Un archivo borrado no se puede recuperar.**

**Falso.** Cuando borras un archivo (unlink), el SO generalmente solo remueve el puntero del directorio, desciende a 0 el link-count y marca los bloques/i-nodo como "libres" en el bitmap. Sin embargo, los _datos físicos_ permanecen intactos en el disco hasta que otro archivo los sobrescriba. Por ende, existen herramientas de forensia informática para recuperarlos.

**(i) Los sistemas de archivos como FAT o UFS no sufren de fragmentación externa.**

**Verdadero.** _(Contextualizando respecto a bloques lógicos)_: Debido a que la asignación de espacio subyacente se realiza siempre en trozos de tamaño idéntico y fijo (bloques o clusters), no ocurre el clásico problema de "huecos de distintos tamaños inservibles" en la memoria que define a la fragmentación externa estricta. Cualquier bloque libre puede ser asignado a cualquier archivo (aunque sufre severa fragmentación _interna_ y pérdida de _contigüidad_).

**(j) Los sistemas de archivos modernos como EXT4 o ZFS tienen problemas de escalabilidad en la cantidad de entradas de un directorio.**

**Falso.** Los FS tradicionales antiguos usaban listas enlazadas simples para recorrer directorios (lento, O(N)). Sistemas modernos como EXT4 o ZFS implementan Árboles-B (B-Trees) o Hash Trees para sus directorios, permitiendo encontrar archivos entre millones de entradas en fracciones de segundo (altamente escalables, $O(\log N)$).

**(k) La syscall truncate sirve para recortar el tamaño del archivo.**

**Verdadero.** Truncate modifica el campo del tamaño en el i-nodo, ya sea cortándolo a 0 o al tamaño especificado, liberando los bloques de disco sobrantes si el archivo se achica.

**Ejercicio 17. ¿Porqué resulta conveniente en una estructura tipo UFS tener un bitmap de i-nodos libres? Exactamente lo mismo se puede conseguir utilizando un bit especial en la tabla de i-nodos que indique si está libre o no. Discuta.**

**Explicación:** Un bitmap es extremadamente **denso y eficiente en caché**. Un bloque estándar de 4096 bytes puede almacenar $32768$ bits (cada uno representando un i-nodo). Con leer un solo bloque al RAM, el SO sabe el estado de más de 32000 i-nodos.

Si pusiéramos esa información como "un campo más" en cada i-nodo (digamos, de 256 bytes cada uno), para consultar el estado libre de 32768 i-nodos el disco tendría que leer y recorrer en RAM la absurda cantidad de 2048 bloques enteros de estructuras de i-nodos. El bitmap ahorra enormemente el consumo de I/O durante la búsqueda de espacios y permite el uso de operaciones binarias rápidas en el procesador.

**Ejercicio 18. Los FS con asignación de espacio no-contiguo (FAT, UFS, etc.) suelen tener un defragmentador para que todos los bloques de un archivo estén contiguos y el seek-time no impacte tanto. ¿Cómo se podría mejorar la estructura de datos que mantiene la disposición o secuencia de bloques de FAT o UFS para aprovechar que siempre se intenta que la secuencia bloques sea una secuencia creciente?**

**Explicación:** Se puede mejorar implementando el concepto de **Extensiones (Extents)**. En lugar de que el i-nodo mantenga un puntero individual para cada bloque uno por uno (lo que requiere de bloques indirectos excesivos para archivos grandes), la estructura de datos mantiene una dupla/tupla: `{Bloque Inicial Físico, Longitud de bloques contiguos}`.

- _Por ejemplo:_ Si un archivo ocupa 1000 bloques consecutivos lógicamente a partir del bloque de disco 50, en UFS tendrías que gastar 1000 punteros. Con las extensiones (usadas en ext4, NTFS, XFS), la metadata es simplemente `[{Start: 50, Length: 1000}]`. Esto reduce bestialmente el tamaño de los metadatos y la sobrecarga, fomentando de forma inherente la escritura contigua.
