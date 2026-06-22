# La abstracción de los procesos

Un **proceso** es la abstracción del SO de un programa en ejecución, el programa es un objeto estático (código), mientras que el proceso es un objeto dinámico (corriendo en memoria).

La técnica de ir intercalando varios procesos concurrentes, ejecutando uno a la vez, es conocida
como **Time Sharing**. 

Su contrapartida, Space Sharing, consiste en dividir un recurso (en el espacio) entre procesos que deseen utilizarlo.

## Virtualización

Para lograr la ilusión de que hay muchos CPU (y que los procesos no tengan el control directo de lo que se ejecuta en la CPU física), el SO usa mecanismos de bajo nivel: protocolos para
implementar distintas funcionalidades. Algunas de ellas son el **context switch**, el cual permite cambiar un proceso en ejecución por otro, estos se rigen mediante diferentes políticas de los planificadores.
Los **planificadores** son algoritmos que toman decisiones sobre la distribución de los recursos limitados en base a distintos factores y prioridades.

## Proceso
Un proceso puede ser descrito por su estado:
1) **Memoria:** Las instrucciones y la información que el proceso lee están en memoria. La memoria a la que un proceso puede acceder (address space) es parte del proceso mismo.
2) **Registros de la CPU:** El programa puede leer o actualizar registros program counter (indica qué instrucción se está ejecutando), el stack pointer, o el frame pointer.
3) **Almacenamiento**: Un proceso puede acceder al almacenamiento (dispositivos I/O) para asegurar la persistencia.


## API de los procesos

Cualquier interfaz de procesos de un SO debe poder:
1) **Crear** nuevos procesos.
2) **Destruir** procesos en caso de que no terminen por sí mismos.
3) Esperar la **finalización** de un proceso.
4) **Tener control**: otros tipos de control, como la **suspensión** de un proceso.
5) **Conocer su estado**: poder mostrar información de un proceso.

Los programas utilizan estas funciones mediante system calls proporcionadas por la API.


###  Creación de procesos

Para correr un programa el SO debe cargar su código y su información estática (con un formato ejecutable) del disco a la memoria, en el address space del proceso.
Se deben proporcionar **memoria Stack** (variables locales, parámetros de llamada, dirección de
retorno) y **Heap** (información dinámica y variable en tamaño, estructuras de datos como listas; todo lo relacionado con malloc y free) para el programa. En el stack, además, el SO establece los parámetros ‘argv’ y ‘argc count’.

Luego, se deben iniciar los file descriptors (std in / std out/ std error) (0, 1, 2). Por último, el SO setea todos los registros a 0 (menos el PC) y pasa el control al proceso creado, dejándolo ejecutarse.


#### Estados de los procesos
Cada proceso está en alguno (solo uno) de los siguientes estados:
- **Running**: en ejecución
- **Ready**: en espera, listo para ejecutarse
- **Blocked**: no puede correr hasta que otro evento ocurra (por ej. un dispositivo I/O)
- **Zombie**: el proceso hijo finaliza, pero el padre aún no ha llamado a wait() para leerlo

El paso entre los distintos estados está dado por eventos del software o hardware, llamadas
interrupciones (por ej., de reloj, del disco duro, etc.)
A lo sumo, puede haber n procesos **running**, siendo n = cantidad de cores (núcleos).

### Estructuras de datos del SO
- **Lista de procesos (process list):** contiene el listado de los procesos listos para correr o corriendo.
- **PCB (process control block):** estructura de datos que, en cada entrada de sus tuplas, almacena el contexto de los registros, el mapa de la memoria, el estado del proceso, su process id (pid), los archivos abiertos, el directorio actual (pwd), y el puntero al proceso padre. Además, almacena los datos de bajo nivel manejados por el procesador: el trap frame y el kernel stack.

Esta información sobre los procesos del sistema nos permite realizar un context switch:
pausarlos, guardar o colocar la información almacenada en los registros, y continuar la ejecución
de un proceso. El SO almacena un array de PCBs.
- **UserTime:** tiempo de cómputo del proceso en modo usuario.
- **CPU time:** tiempo total que un proceso ha usado la CPU. Si ejecuta más de un hilo, es la suma del tiempo de todos ellos.
- **WallTime:** tiempo total real transcurrido, desde que comienza a ejecutarse un proceso hasta que termina.
- **S:** modo del proceso: sleeping, active, etc

WallTime ≥ CPU Time: cuando el proceso se ejecuta en un solo núcleo o en múltiples núcleos pero sin paralelismo (un núcleo alternando entre varios hilos).

WallTime < CPU Time: puede llegar a suceder si el proceso se ejecuta en múltiples núcleos
simultáneamente (paralelismo), ya que el CPU Time se suma en cada núcleo que está en uso.

UserTime < WallTime: el proceso pasa parte del tiempo esperando: ya sea I/O, bloqueado, o al return del trap cuando hace una syscall y pasa a modo kernel.

UserTime = WallTime: un proceso sin hilos se ejecuta en modo usuario, sin interrupciones ni
operaciones de I/O, sin llamadas al sistema, y no se bloquea ni o sufre context switches.

WallTime < UserTime: proceso multihilo, en el que cada hilo ejecuta en un core distinto y luego se
suman todos sus tiempos de usuario.
