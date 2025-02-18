# Capitulo 6 - Ejecucion directa limitada

Para virtualizar la CPU, el SO necesita de alguna forma compartir la CPU fisica entre los muchos trabahos ejecutandose aparentemente al mismo tiempo. La idea es simple: ejecutar el proceso por un tiempo pequeño, pararlo, volverlo a ejecutar y asi sucesivamente. Asi es como se logra la virtualizacion

# Tecnica basica: Ejecucion directa limitada

La parte de “ejecucion directa” de la idea es simple: solo ejecutar el programa directamente desde la CPU. 

## Protocolo de ejecucion directa (sin limitaciones)

| **OS** | **Program** |
| --- | --- |
| Create entry for process list |  |
| Allocate memory for program |  |
| Load program into memory |  |
| Setup with argc/argv |  |
| Clear registers |  |
| Execute **call** main() |  |
|  | Run main() |
|  | Execute **return** from main |
| Free memory of process |  |
| Remove from process list |  |

## Problema #1: Operaciones restringidas

La ejecucion directa tiene la ventaja obvia de ser rapida; el program se ejecuta nativamente en el hardware del CPU y por lo tanto se ejecutara tan rapido como lo esperamos. Pero ejecutar un programa directo en la CPU introduce un problema: que pasa si el proceso quiere hacer algun tipo de operacion restringida, como emitir una solicitud de I/O a un disco, o ganar acceso a mas recursos del sistema como la CPU o la memoria

Un enfoque podria simplemente ser permitirle a cualquier proceso hacer cualquier cosa que ellos quieran en terminos de I/O y otras operaciones relacionadas. Sin embargo, hacer eso evitaria la construccion de muchos tipos de sistemas que son deseables. Por ejemplo, si queremos construir 
un sistema de archivos que verifique permisos antes de conceder acceso a un archivo, no podemos simplemente dejar a cualquier usuario procesar cualquier solicitud de I/O al disco; si lo hicieramos, un proceso podria simplemente leer o escribir el disco entero y por lo tanto todas las 
protecciones se perderian

Por lo tanto, el enfoque que tomamos es introducir un nuevo modo de procesador, conocido como **modo usuario**. El codigo que se ejecuta en el modo usuario es restringido en que puede hacer. Por ejemplo, cuando corremos en modo usuario, un proceso no puede emitir una peticios de I/O, hacerlo resultara en el surgimiento de una excepcion y el OS probablemente matara el proceso.

En contraste el modo usuario esta el **modo kernel**, en el cual se ejecuta el OS, el codigo que se ejecuta en ese modo puede hacer lo que quiera, inclutendo operaciones privilegiadas como emitir 
peticiones de I/O y ejecutar todos los tipos de instrucciones restringidas.

Sin embargo, todavia tenemos un reto: que deberia hacer un programa de usuario cuando quiere hacer algun tipo de operacion privilegiada, como leer desde el disco? Para poder hacer esto, virtualmente todos los hardwares modernos proporsionan a los programas de usuarios hacer una **system call**. 

Las system calls le permiten al kernel exponer cuidadosamente ciertas partes claves de funcionalidad a los programas de usuario, como acceder al sistema de archivos, crear y destruir procesos, comunicarse con otros procesos, y asignar mas memoria. Muchos OS proporsionan unas cientas system calls; al principio los sistemas UNIX exponian un subconjuto mas conciso de alrededor veinte system calls.

Para ejecutar una system call, un programa debe ejecutar una instruccion **trap** especial. Esta instruccion simultanemente salta al kernel y levanta el nivel de privilegios a modo kernel; una vez en el kernel, el sistema puede ejecutar cualquier operaciion privilegiada como necesite, y por lo tanto hacer el trabajo requerido para el proceso que lo pidio. Cuando termina, el OS llama a una funcion especial de **return-from-trap**, la cual, regresa al programa de usuario mientras simultanemente reduce el nivel de privilegios a modo usuario.

El hardware necesita ser un poco mas cuidadoso cunado ejecuta una instruccion **trap**, en el sentido que debe asegurarse de guardar susficientes registros del proceso que lo llama para poder regresar correctamente cuando el OS emita la instruccion **return-from-trap**. En x86 por ejemplo, el procesador empujara el PC, flags, y algunos otros registros en un **kernel stack** por proceso; el **return-from-trap** soltara esos valores del stack y continuara la ejecucion programa en modo usuario. Otros sistemas de hardware usan convenciones diferentes, pero el concepto basicos son similares en todas las plataformas.

Por lo tanto el kernel debe controlar cuidadosamente que codigo ejecutar.

El kernel lo hace seteando una **trap table** cuando bootea. Cuando la maquina bootea, lo hace en modo kernel, y por lo tanto el libre de configurar el hardware de la maquina como lo necesite. Una 
de las primeras cosas que el OS hace es decirle al hardware que codigo ejecutar cuando ocurre un evento expcional. Por, ejemplo. que codigo ejecutar cuando se produce una interrupcion de disco duro, que codigo ejecutar cuando ocurre una interrupcion de teclado, o cuando un programa hace una system call? El OS le informa al hardware la ubicacion de esos **trap handlers**, usualmente con algun tipo especial de instruccion. Una vez el hardware esta informado, recuerda la informacion de la ubicacion de esos manejadores hasta que la maquina es reiniciada, por lo tanto el hardware sabe que hacer (es decir, a que codigo saltar) cuando una system call y otros eventos toman lugar.

Para especificar la system call exacta, se le asigna a cada system cal un **numero de system call**.

| OS @ boot (kernel mode) | Hardware |
| --- | --- |
| **initialize trap table** |  |
|  | remember address of ... |
|  | syscall handler |

| OS @ run (kernel mode) | Hardware | Program (user mode) |
| --- | --- | --- |
| Create entry for process list |  |  |
| Allocate memory for program |  |  |
| Load program into memory |  |  |
| Set up user stack with argv |  |  |
| Fill kernel stack with reg/PC |  |  |
| **return-from-trap** |  |  |
|  | restore regs (from kernel stack) |  |
|  | move to user mode |  |
|  | jump to main |  |
|  |  | Run main() |
|  |  | ... |
|  |  | Call system call |
|  |  | **trap** into OS |
|  | save regs (to kernel stack) |  |
|  | move to kernel mode |  |
|  | jump to trap handler |  |
| Handle trap |  |  |
| Do work of syscall |  |  |
| **return-from-trap** |  |  |
|  | restore regs (from kernel stack) |  |
|  | move to user mode |  |
|  | jump to PC after trap |  |
|  |  | ... |
|  |  | return from main |
|  |  | **trap** (via exit()) |
| Free memory of process |  |  |
| Remove from process list |  |  |

## **Problema #2: Cambiando entre procesos**

El siguiente problema con la ejecucion directa es lograr cambiar de proceso. Cambiar de proceso deberia ser simple, verdad? El OS deberia simplemente decir frenar un proceso y comenzar otro. Cual es el problema? Realmente es un poco mas complicado: especificamente, si un proceso se esta ejecutando en la CPU, por definicion el OS no se esta ejecutando. Y si no se esta ejecutando como lo haria? (ayuda: no puede).

Claramente no hay forma de que el OS tome una accion si no se esta ejecutando en la CPU.

### **Un enfoque cooperativo: Esperar por las system calls**

Un enfoque que muchos sistemas tomaron en el pasado es conocido como el enfoque cooperativo. En este estilo, el OS *confia* en que los procesos del sistema van a comportarse rasonablemente. Se asume que los procesos que van a ejecutarse por mucho tiempo, peridicamente le vana ceder el control al OS para que el OS pueda decidir si es tiempo de ejecutar alguna otra tarea.

Por lo tanto, debes prguntarte, como un amigable proceso transfiere el control de la CPu en este mundo utopico? Resulta que muchos procesos tranfieren el control de la CPU al OS frecuentemente haciendo llamadas a system calls , por ejemplo, para abrir un archivo y subsecuentemente leerlo, o mander un mensaje a otra maquina, o crea un nuevo proceso. 
Muchos sistemas como este, a menudo incluyen una system call explicita, **yield**, la cual no hace nada excepto tranferir el control al OS para asi poder ejecutar otros procesos.

Las apliciones tambien le ceden el control al OS cuando ahcen algo ilegal. Por ejemplo, si una aplicacion divide por cero, o si trata de acceder a memoria que no es accesible. generara una **trap** al OS. Entonces, el OS tendra el control de la CPU de nuevo y probabolemente matara el proceso.

Por lo tanto, en un sitema de organizacion cooperativo, el OS retoma el control de la CPU esperando una system call o que una operacion ilegal de algun tipo tome lugar. Seguramente pensaras: este enfoque pasivo no es una mala idea? Que sucede si, por ejemplo, un proceso (sin querer o malintencinadamente) termina en un blucle infinito y nunca hace una system call? Que podria hacer entonces el OS?

### **Un enfoque no-cooperativo: El OS toma el control**

Resulta que, sin la ayuda del hardware, el OS no puede hacer mucho cuando un proceso se rehusa a hacer una system call y devolver el control al OS. 
De hecho, en el enfoque cooperativo, el unico recurso cuando un proceso se traba en un bucle infinito es recurrir a la solucion mas antigua de todos los problemas en los sistemas computacionales: ***REINICIAR LA MAQUINA***. Por lo tanto, arrivamos a un nuevo problema de nuestra busqueda incial para tener control de la CPU. Como puede hacer el OS para tener el 
control de la CPU incluso si un proceso no esta siendo cooperativo?

La respuesta resulta ser simple y fue descubierta hace muchos años: un **interruptor de tiempo (time interrupt)**. Un timer puede ser programado para lanzar una interrupcion cada tantos milisegundos; cuando la interrupcion es lanzada, el actual proceso en ejecucion es detenido, y se ejecuta en el OS un manejador de interrupciones preconfigurado. En este punto el OS retoma el control, y por lo tanto puede hacer lo que le plazca: detener el actual proceso y ejecutar uno diferente.

**Guardando y restaurando el contexto**

Ahora que el OS ha retomado el control, ya sea mediante system calls cooperativas, o a traves de interrupciones, tiene que tomar una decision: seguir ejecutando el proceso actual o cambiar a otro diferente. Esta decision es tomada por una parte del OS conocida como **scheduler**.

Si la decision fue cambiar de proceso, entonces el OS ejecuta una parte de codigo de bajo nivel a la que nos referiremos como **cambio de contecto (context switch)**. Un cambio de contexto es conceptualmente simple: todo lo que el OS tiene que hacer es guardar un par de registros del proceso que se esta ejecutando actualmente (en el stack del kernel) y recuperar algunos del 
proximo proceso a ejecutar (tambien el stack del kernel). Haciendo eso el OS se asegura que cuando se ejecute la instruccion **return-from-trap**, en vez de volver al proceso que estaba ejecutando, el systema retomara la ejecucion de otro proceso.

| OS @ boot (kernel mode) | Hardware |
| --- | --- |
| **initialize trap table** |  |
|  | rembember address of... |
|  | syscall handler |
|  | timer handler |
| **start interrup timer** |  |
|  | start timer |
|  | interruptt CPU in X ms |

| OS @ run (kernel mode) | Hardware | Program (user mode) |
| --- | --- | --- |
|  |  | Procees A |
|  |  | ... |
|  | **timer interrupt** |  |
|  | save regs(A) → k-stack(A) |  |
|  | move to kernel mode |  |
|  | jump to trap handler |  |
| Handle trap |  |  |
| Call *switch()* routine |  |  |
| save regs(A) → proc_t(A) |  |  |
| restore regs(B) ← proc_t(B) |  |  |
| switch to k-stack(B) |  |  |
| **return-from-trap into(B)** |  |  |
|  | restore regs(B) ← k-stack(B) |  |
|  | move to user mode |  |
|  | jumps to B's PC |  |
|  |  | Process B |
|  |  | ... |

En estos cuadros se ve la linea de tiempo completa. El proceso A esta ejecutandose y es interrupido por el **timer interrupt**. El hardware guarda sus registros y entra en el kernel. En el manejador de interrupciones, el OS decide cambiar la ejecucion del Proceso A al Proceso B. En este punto, llama a la rutina *switch()*, la cual guarda cuidadosamente los valores de los registros (del proceso A), y restaura los registros del Proceso B, y entonces cambia de contexto, especificamente cambiando el *stack pointer* para usar el stack del kernel del proceso B. Finalmente el OS returns-from-trap, el cual restaura los registros del Proceso B y comienza a ejecutarlo.