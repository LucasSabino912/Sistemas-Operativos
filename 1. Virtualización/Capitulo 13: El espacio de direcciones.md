# El espacio de direcciones

EL **address space** es la abstracción de la memoria física que crea el SO, y es lo que ve un programa corriendo, la virtualización de memoria que le propociona a los procesos la ilusión de un espacio de mempria amplio y privado
El **address space** (espacio direccionable) de un proceso contiene todo el estado de la memoria del
programa en ejecución; el código mismo del programa, el stack y el heap:
- El **stack** es usado para guardar la cadena de llamadas a función; dirección de retorno, variables locales y parámetros.
- El **heap** se utiliza para almacenar elementos dinámicamente (dynamically allocated), es decir, es memoria manejada por el usuario (usando funciones como malloc en C).

El código tiene tamaño fijo (pero no es necesariamente estático; puede ser self-modifying code) lo cual lo hace fácil de poner en memoria. 
En cambio, el stack y el heap pueden crecer o decrecer mientras el programa corre. Al poner en forma opuesta el heap y stack (por convención) podemos permitirles cambiar su tamaño siguiendo direcciones opuestas (luego, al trabajar con múltiples hilos esta forma simple de ubicarlos ya no sirve).

La dirección en la que el proceso se ve a sí mismo es su memoria virtual, y en base a ella hace
requisitos al SO, el cual debe traducir estas direcciones virtuales a memoria física real a la hora de
responder los pedidos del programa (como, por ej., guardar un archivo), para lo cual utiliza la ayuda del hardware.

El trabajo del SO es virtualizar la memoria. Para hacerlo bien, debe cumplir 3 objetivos:
1) **Transparencia:** la implementación de la memoria virtual debe “ser invisible” para el programa, el cual debe creer que tiene su propia memoria física. El SO junto al hardware crean esta ilusión.
2) **Eficiencia:** la virtualización debe ser lo más eficiente posible en términos de tiempo y espacio. Para esto el SO utiliza distintas características del hardware, como la TLB.
3) **Protección:** el SO debe proteger los procesos unos de otros, así como proteger al SO de los procesos. Para eso aísla a la memoria de los procesos (solo pueden acceder a su address space) para que no puedan
