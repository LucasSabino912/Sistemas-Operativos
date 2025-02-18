# Capitulo 4 - La abstraccion de los procesos

# Procesos

Los procesos son la abstraccion mas fundamental que los SO proporcionan al usuario. La definicion de un proceso es simple: **es un programa en ejecucion**. El programa es solo un puñado de instrucciones esperando a ser ejecutados, el So toma esos bytes y los pone en ejecucion, transformando el programa en algo util.

Resulta que a menudo queremos ejecutar varios programas a la vez. 

Para dar la ilusion de que se ejecutan al mismo tiempo, el OS **virtualiza** la CPU. Ejecutando un proceso, frenandolo, ejecutando otro y parandolo, asi sucesivamente. El SO puede fomanentar la idea de que hay varias CPUs virtuales corriendo procesos cuando en realidad hay una sola CPU fisica, o unas pocas. Esta tecnica es conocida como **time sharing of CPU**, y permite a los usuarios correr tantos procesos de forma concurrente como ellos quieran, el unico costo sera el desempeño, cada proceso se ejecutara mas lento si la CPU es compartida.

# La abstraccion: un proceso

La abstraccion que el SO proporciona de un programa en ejecucion es llamada proceso. Para entender que constituye un proceso, tenemos que entender maquina de estado **(machine state)**: lo que un programa puede leer o actualizar mientras se ejecuta.

Un componente obvio de la machine state que comprende un proceso es la memoria. Las instrucciones viven en la memoria; los datos que el programa lee y escribe tambien estan en la memoria. Por lo tanto, la memorua que el proceso puede direccionar es parte del proceso

# API de procesos

Lo que cualquier interface de un SO deberia incluir. 

- **Crear (create):** metodo para crear un proceso nuevo
- **Destruir (destroy):** metodo para destruir un proceso; forzar su detencion
- **Esperar (wait):** metodo para esperar un proceso, por ejemplo antes de terminar la ejecucion
- **Control miscelaneo (Miscellaneous control):** metodo para suspender un proceso por un tiempo, detener su ejecucion y despues continuar ejecutandolo
- **Estado (status):** metodo para obtener informacion de estado de un proceso, como por cuanto tiempo se estuvo ejecutando o su estado interno

![https://i.postimg.cc/fTBYWhrf/1000001107.png](https://i.postimg.cc/fTBYWhrf/1000001107.png)

# Creacion de un proceso

Lo **primero** que hace el SO para ejecutar un programa es cargar su codigo y cualquier data estatico (ej variables inicializadas) dentro de la memoria, en el espacio de mempria del proceso. Los programas residen en el disco en formato ejecutable. Una vez el codifo y los datos estaticos  estan cargados en la memoria, hay un par de cosas mas que el SO debe hacer, como asignar **(allocate)** algo de memorua para el **run-time** o solo **stack**. Los programas escirtos en C usan el stack para variables locales, parametros de funciones y return de direcciones. El SO tambien debe inicializar el stack con argumentos (los que van en main).

El SO tambien puede asignar memoria para el **heap** del programa. En C, el heap es usado cuando se solicita memoria dinamicamente `malloc()` o `alloc()` y liberandolo llamando a `free()`. El heap es necesitado por las estructuras de datos (linkedi lists, hash tables, trees, etc)

El SO tambien hara tareas de inicializacion relacionadas con I/O, Por ej, en los sistemas UNIX, cada proceso por defecto, tiene abierto tres **file descriptors:** para **standard input, standard output** y para **errores**,  estos fd le permiten a los programas leer entradas de la terminar e imprimir por pantalla

# Estado de los procesos

Un proceso puede estar en uno de estos estados:

- **En ejecucion (running):** El proceso esta ejecutandose en un procesador
- **Listo (ready):** El proceso esta listo para ejecutarse y por alguna razon el SO decidio todavia no ejecutarlo
- **Bloqueado (blocked):** El proceso ha realizado algun tipo de operacion que lo hace no etsar listo para ejecutarse hasta que algun otrio evento tome lugar

![https://i.postimg.cc/6pF2T6Ln/Estados-Procesos.png](https://i.postimg.cc/6pF2T6Ln/Estados-Procesos.png)

# **Estructuras de Datos**

El OS es un programa, y como cualquier programa, tiene algunas estructuras de datos clave que rastrean varias partes relevantes de informacion. Para rastrear el estado de cada proceso, por ejemplo, el OS mantiene algun tipo de **lista de procesos** para todos los procesos que estan listos, y alguna informacion adicional para saber que procesos estan actualmente en ejecucion. Tambien el OS debera rastrear de alguna manera los procesos bloqueados; y cuando un evento de I/O se complete debera asegurarse de levantar (**wake**) el proceso correcto y cambiar su estado a *listo* para ejecutarlo nuevamente.

Hay algunos otros estados en los que un proceso puede estar:

- Inicial (**Initial**): un proceso esta en estado *inicial* cuando es recien creado
- Final (**Final**): un proceso puede ser puesto en estado *final* donde a salido, pero todavia no ha sido limpiado (en sistemas basados en UNIX, este estado es llamado **zombie**).
