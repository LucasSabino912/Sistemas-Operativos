# Gestión de espacio libre

El manejo del espacio libre resulta sencillo cuando el espacio está dividido en unidades de espacio constante; si se solicita espacio solo entregar la primera entrada libre (paginación). Pero se vuelve difícil al tratar con bloques de espacio libre de diferentes tamaños.

## Suposiciones

Contamos con una interfaz para administrar la memoria que provee las funciones `malloc(size_of size)` y `free()`, tal y como las describimos en capítulos anteriores. Esta librería maneja la memoria _heap_, y la estructura genérica que maneja el _free space_ es la **free list**, la cual contiene una referencia a todos los bloques de espacio libre en la región que maneja la memoria. Esta abstracción se usa tanto en espacio de usuario como en espacio de kernel.

Una vez se llama a `malloc()` con memoria dentro del heap, esta no puede ser tocada por la librería hasta que se use `free()`. Esto implica que hasta ese momento no es posible compactar el espacio libre (porque no podemos mover el bloque ya asignado). La compactación puede ser usada por el SO para combatir la fragmentación de usar segmentación.

También asumimos que el _allocator_ maneja una región de bytes continua (no va a crecer).

## Mecanismos de bajo nivel: División y fusión (Splitting and Coalescing)

Una _free list_ contiene un conjunto de elementos que describen los espacios libres que quedan en la memoria heap. Consiste en una serie de listas ligadas con la dirección de los bloques, su longitud, y la dirección del siguiente espacio libre. Si se solicita un lugar mayor al disponible (puede que el espacio esté desocupado pero no en un bloque continuo) el pedido falla.

Los _allocators_ cuentan con dos mecanismos usados para administrar el espacio libre:

- **Splitting (División):** Cuando se solicita memoria, el _allocator_ busca un bloque de memoria que satisfaga el pedido. Si el mismo tiene un tamaño mayor al requerido, lo partirá en dos y devolverá un puntero al primero (del tamaño justo necesario), manteniendo al segundo en la _free list_.
    
- **Coalescing (Fusión):** Consiste en combinar los bloques de espacio libre continuos en uno solo. Este procedimiento se realiza cuando se libera espacio.
    

## Seguimiento del tamaño de las regiones asignadas

Al llamar a `free()` no se especifica el tamaño de la región a liberar; la librería es capaz de darse cuenta sola y recuperar la memoria. Para lograr esto, la mayoría de _allocators_ guardan información en un **header block** al inicio de cada bloque de memoria asignado.

El header contiene el tamaño de memoria asignada a ese bloque. Además, puede contener punteros para acelerar la recuperación de memoria, un “número mágico” para verificar la integridad, y otra información extra.

Notar entonces que el tamaño del bloque de memoria, es decir el espacio a liberar, ahora puede saberse rápidamente y consiste del tamaño del espacio solicitado más el tamaño de la estructura de su header (`size` y `magic`, mismo al hacer alloc). Esto es: `tamaño solicitado + 8 bytes`.

## Encastración en una lista de espacios libres

La _free list_ se encuentra en un bloque dentro del espacio libre mismo.

Cuando se solicita espacio, se asigna al comienzo de la _free list_ (header + bloque). Cada vez que se recibe un nuevo pedido, es asignado al final del bloque anterior (siempre y cuando el espacio disponible sea el suficiente).

Al liberarse un bloque, la celda “magic” pasa a ser un puntero “next” al siguiente _chunk_. De haber dos bloques contiguos que cumplan esa condición, se los une y se modifica el puntero next.

## Aumentando el tamaño del heap

Los _allocators_ suelen comenzar con una memoria heap pequeña e ir pidiendo más espacio al SO a medida que lo necesitan. Este, si tiene éxito, les devuelve la dirección del nuevo final del heap.

## Estrategias básicas

El _allocator_ ideal es rápido, eficiente en el uso del espacio (minimizando fragmentaciones) y escalable. No hay un único enfoque mejor que el resto, pero existen varias políticas de manejo de espacio libre que persiguen ese objetivo:

- **Best fit (Mejor ajuste):** Se busca a través de la _free list_ al bloque libre más pequeño de los espacios iguales o superiores al solicitado. Al hacer una búsqueda exhaustiva en la _free list_, genera una penalización de performance, y una fragmentación en espacios libres pequeños.
    
- **Worst fit (Peor ajuste):** Se busca al bloque más grande disponible, se usa el espacio necesario, y se devuelve lo restante a la _free list_. Genera los mismos _overheads_ al realizar también una búsqueda exhaustiva (fragmentando esta vez en bloques libres grandes).
    
- **First fit (Primer ajuste):** Usa el primer bloque de la lista lo suficientemente grande para cumplir con lo solicitado. Su ventaja es la velocidad ya que evita realizar una búsqueda exhaustiva, pero “contamina” el comienzo de la _free list_ al concentrar allí la fragmentación en bloques pequeños.
    
- **Next fit (Siguiente ajuste):** Igual a _First Fit_ pero utiliza un puntero que le permite comenzar la búsqueda desde la última posición revisada la vez anterior. Desparrama la fragmentación a lo largo de la _free list_ y mantiene la velocidad del enfoque anterior, pero requiere un puntero extra en la implementación.
    
- **Listas segregadas (Segregated lists):** Ante aplicaciones que tengan peticiones recurrentes de tamaño similar, se crea una nueva lista para el manejo de objetos de ese tipo, y se envían las demás peticiones al _allocator_ general. La fragmentación es menor y los pedidos de dicho tamaño se satisfacen más rápido.
    
    - _Ejemplo:_ El _slab allocator_ asigna un número de _object caches_ para objetos del kernel que se solicitan seguido. Si le falta espacio pide más _slabs_ (bloques pequeños) de memoria. Este allocator también mantiene los _free objects_ de las listas en un estado pre-inicializado.
        
- **Buddy Allocation:** Tanto la memoria libre como la memoria que se asigna son espacios de tamaño $2^N$. Cuando se solicita un bloque, se divide el espacio libre por 2 hasta encontrar uno que satisfaga el pedido. Cuando un bloque se libera se chequea que su “buddy” (compañero) del mismo tamaño esté libre, y si lo está los combina, y así recursivamente hasta hacer _coalescing_ de todo o encontrar un “buddy” en uso; simplifica el _coalescing_ pero genera fragmentación interna.
