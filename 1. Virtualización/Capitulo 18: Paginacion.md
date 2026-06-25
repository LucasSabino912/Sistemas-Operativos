# Introducción a la paginación
Si se corta el espacio disponible en bloques de tamaños diversos se genera fragmentación, lo que
complica la asignación de espacio mientras más espacio se ocupa.
Un enfoque diferente consiste en cortar el espacio en **partes iguales** de un **tamaño estándar**. Esto en memoria virtual se llama **paginación** y cada unidad es una página. A la memoria física se la ve como un **array de slots** iguales llamados page frame.

La paginación tiene carias ventajas comparada a los enfoques anteriores
**Flexibilidad:** Es capaz de apoyar la abstracción del address space eficientemente, mas alla de como el proceso use el address space
**Simplicidad:** permite manejar la free list tan solo otorgando la cantidad de paginas necesarias

Para mantener un registro de donde cada página virtual del address space está situada en memoria física, el SO guarda información de cada proceso en una estructura conocida como page table, la cual almacena las address translations de cada página virtual.

Para que el hardware y el SO traduzcan una dirección virtual debemos dividir en 2 componentes la
dirección: el **virtual page number** (VPN) y el offset de la página:
Con el número de página virtual se indexa la page table para encontrar el **marco físico** (FPN) en el cual reside la página; luego solo se reemplazar el VPN por el PFN y se mantiene el mismo offset (el cual señala el byte, dentro de la página, que estamos solicitando).

## Page Tables
Son las estructuras de datos usadas para mapear las direcciones virtuales a direcciones físicas.
Al ser largas y muchas (cada una tiene muchas page table entry(ies) (PTE); una VPN de 20 bits, por ej., implica 2^20 traducciones) no se almacenan en la MMU, sino directamente en memoria.

**Organización**
La más simple es linear page table; la page table es vista como un array el cual se indexa con el VPN y se busca el PTE de dicha indexación para encontrar el PFN.

Cada PTE tiene algunos bits importantes:
- **Valid bit**, permite reservar adress space al marcar páginas como inválidas, evitando tener que asignarles memoria
- **Protection bits**, indican si una página puede ser leída, escrita o ejecutada.
- **Present bit**, indica si se encuentra en memoria física o en disco.
- **Dirty bit**, indica si una página ha sido modificada desde que fue traída a memoria.
- **Accessed bit** (“reference”), indica si la página ha sido accedida (útil para determinar páginas populares que deben ser mantenidas en memoria).

**Rapidez y paginación**
La paginación requiere que se realice un referencia a memoria extra para buscar la traducción de la page table (de virtual a física, de la VPN a PTE y luego a PFN), lo cual es costoso, y dado el tamaño de las page tables, se ralentiza demasiado el sistema.

Cuando corre, cada instrucción fetcheada genera dos referencias de memoria; una a la page table
para encontrar el physical frame en la que la instrucción reside, y otra a la instrucción en sí misma
para poder fetchearla hacia la CPU.
