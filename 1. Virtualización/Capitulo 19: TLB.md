# TLB

La paginación requiere un gran mapeo de información, el cual normalmente está almacenado en memoria física, y requiere una búsqueda extra para cada dirección virtual generada por los procesos. Ir a memoria por la traducción antes de cada instrucción hace perder al CPU demasiados ciclos de clock en espera.

Para aumentar la velocidad de la traducción de las direcciones, el SO se apoya en el hardware; usa una parte del chip MMU llamada **Translation-Lookaside Buffer (TLB)**, el cual es un caché de traducción de direcciones virtuales a físicas (por esto lo llamamos _address-translation cache_).

Cada vez que hay una referencia virtual a memoria, el hardware primero revisa la TLB para ver si la traducción está ahí y, de estarlo, la traducción es realizada rápidamente sin necesidad de acceder a la _Page Table_ y causar un cuello de botella en el _pipeline_ del CPU.

## Algoritmo básico de la TLB

Supongamos una _page table_ lineal y un TLB manejado por hardware. El algoritmo consiste en:

1. Extraer el **VPN** (_Virtual Page Number_) de la _virtual address_ y revisar si el TLB tiene la traducción para este VPN.
2. Si lo tiene (**TLB hit**), obtiene el **PFN** (_Physical Frame Number_) de la _TLB entry_, lo concatena al _offset_ de la dirección virtual original, y consigue la dirección física deseada para acceder a memoria (mientras que los chequeos de protección no fallen).
3. Si es **TLB miss** (no tiene la traducción), el hardware accede a la _page table_ para encontrar la traducción y, asumiendo que la dirección virtual es válida y accesible, la sube a la TLB. La próxima vez que se busque esa traducción, va a ser _TLB hit_ y se ejecutará rápido.
## Localidad Espacial y Temporal

Supongamos un array y un _loop_ que lo recorre linealmente. Si en cada página de la _page table_ tenemos varios elementos del array, el primero al que queramos acceder de la página va a ser un _miss_ pero luego hará _hit_ con todos los elementos que estén en la misma página, mejorando drásticamente el desempeño en comparación a tener que buscar la traducción de la referencia virtual para cada uno de ellos.

> Ejemplo: `a[0]` sería _miss_ pero `a[1]` y `a[2]` serían _hit_, `a[3]` sería _miss_, etc.

Esto se da porque, ante un _TLB miss_, se guarda toda la página en la TLB, y no solo la dirección cuya traducción fue requerida. Es decir, la TLB se beneficia de la **Spatial Locality** (localidad espacial: los elementos están cerca unos de otros) y, si el programa se vuelve a ejecutar rápidamente (por ej. un _loop_), se beneficia de la **Temporal Locality** (localidad temporal: la rápida re-referenciación a ítems en memoria en el tiempo) ya que todavía formarán parte de la TLB y volverían a ser _hit_. Ante mayor tamaño de las páginas o de la TLB, mayor porcentaje de _TLB hit_ se genera y mejor se aprovechan estas localidades.

## Manejo del TLB miss

En sistemas antiguos CISC, los _TLB miss_ eran manejados por hardware. En sistemas modernos RISC son manejados por el software, usando un **software-managed TLB**:

- En un _miss_, el hardware solo levanta una excepción que pausa el flujo de instrucciones, eleva el privilegio a modo kernel y salta al _trap handler_ (un código dentro del SO).
- El SO busca la traducción en la _page table_, y actualiza la TLB con instrucciones privilegiadas, para luego volver del _trap_ y que el hardware re-ejecute la instrucción.

**Detalle clave:** El _return from trap_ luego de un _TLB miss_ debe **volver a ejecutar la misma instrucción** que causó el _miss_ para esta vez dar _hit_, a diferencia del _return from trap_ normal que ejecuta la _siguiente_ instrucción (el registro PC o _Program Counter_ se maneja diferente en cada caso).

El SO debe tener cuidado de no causar un _loop_ de _TLB misses_ (si, por ej., el _trap handler_ se encuentra en una memoria virtual no cacheada en la TLB), y para esto tiene diversas estrategias. Las principales ventajas de que el _TLB miss_ sea manejado por software son su simplicidad y flexibilidad; se puede implementar cualquier estructura de datos que se necesite sin requerir un cambio en el hardware.

## Contenidos de la TLB

Una TLB típica tiene 32, 64 o 128 entradas y es _fully associative_, es decir, el hardware buscará la traducción en paralelo (rapidez constante) en todas las entradas. La estructura típica de una entrada es:

`VPN | PFN | otros bits`

Entre esos "otros bits" suelen incluirse:

- **Valid bit:** Indica si una entrada tiene una traducción válida para esa dirección de memoria virtual o no. _(Notar que el valid bit de la TLB es diferente al valid bit de la Page Table, el cual marca si una entrada no ha sido asignada por el proceso y no debe ser usada)._
- **Protection bits (R/W/X):** Determinan si se puede acceder a una página para leer (_Read_), escribir (_Write_) o ejecutar (_Execute_).
- **Address Space Identifier (ASID):** Identificador del proceso dueño de la traducción.
- **Dirty bit:** Indica si la página fue modificada.

## Context Switches (Cambios de Contexto)

Las traducciones que hay en la TLB sólo sirven para el proceso en ejecución; al hacer un _context switch_ dichas traducciones no deben ser usadas con el nuevo proceso.

1. **Flush (Limpieza):** Un enfoque es borrar el contenido de la TLB, seteando todos los _valid bits_ en 0. Sin embargo, ante cambios de contexto frecuentes, se genera un alto costo (overhead) al tener que buscar las traducciones desde cero cada vez.
2. **ASID (Address Space Identifier):** Frente a esto, algunos sistemas se apoyan en el hardware al añadir un **ASID**; un campo en la TLB que permite identificar a qué proceso pertenecen las traducciones, pudiendo almacenar a la vez las de diferentes procesos sin que colisionen.

## Política de reemplazos (Cache Replacement)

Las memorias caché son veloces pero pequeñas. Si para insertar una nueva entrada en la TLB es necesario reemplazar una vieja, se debe elegir una política para realizar ese reemplazo buscando siempre bajar el porcentaje de _TLB miss_:

- **LRU (Least Recently Used):** Borrar la entrada que más tiempo lleva sin usarse para tratar de mantener la localidad temporal.
- **Random (Aleatoria):** Borrar una entrada al azar. Es útil porque no presenta "casos borde" desastrosos como LRU (por ejemplo, ante un recorrido en _loop_ de un array que justo excede por 1 el tamaño de la TLB, donde LRU fallaría siempre).
