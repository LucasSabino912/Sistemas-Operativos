# Capitulo 8 - MLFQ

El problema que intenta abordar MLFQ es doble. Primero, trata de **optimizar el tiempo de entrega,** que como sabemos se logra ejecutando los procesos mas cortos primero; desafortunadamente el OS, muchas veces, no sabe que tan largo es un proceso, la informacion que algoritmo como SJF o STCF necesitan. Segundo, MLFQ intenta hacer que el sistema se sienta interectivo, y por lo tanto minimizar el tiempo de respuesta; desafutunadamente algoritmos como RR reducen el tiempo de respuesta pero aumnetando muchisimo el tiempo de entrega. Por lo tanto, nuestro problema es: dado que, en general, no sabemos nada sobre los procesos, como podemos contruir un plinificador que logra todos nuestro objetivos? 

## MLFQ: Reglas basicas

MLFQ tiene un numero de diferentes **colas (queues)**, cada una asignada a un **nivel de prioridad (priority level)** diferente. En un momento dado, un procesos que esta listo para ejecutarse esta en una sola cola. MLFQ usa prioridades para decidir que proceso debe ejecutarse en cierto momento: un proceso con una mayor prioridad es elegido para ejecutarse.

Por supuesto, muchos procesos estaran en la misma cola, y por lo tanto, tendran la misma prioridad. En ese caso, usameros RR para planificar entre ellos.

- **Regla 1**: If Priority(A) > Priority(B) → A runs (B doesn't).
- **Regla 2**: If Priority(A) = Priority(B) → A & B runs in RR.

Por lo tanto, la clave de MLFQ radica en como el **planificador establece las prioridades.** En vez de dar una prioridad fija a cada proceso, MLFQ *varia* la prioridad de cada proceso basado en *comportamiento observado*. Si, por ejemplo, un procesos cede el control de la CPU repetidamente 
mientras espera por una accion del teclado, MLFQ mantendra su prioridad alta, porque asi es como debe comportarse un proceso interactivo. Si, en cambio, un proceso usa la CPU intensivamente por largos periodos de tiempo, MLFQ reducira su prioridad. De estra forma, MLFQ tratara de *aprender* sobre los procesos a medida que se ejecutan, y por lo tanto, usar la *historia* de esos procesos para predecir su *futuro* comportamiento

## Intento #1: Como cambiar prioridades

Ahora debemos decirdir como MLFQ cambiara los niveles de prioridades de un proceso a traves 
del tiempo de vida del trabajo. Para hacer esto, debemos mantener una mezcla de procesos de corta ejecucion (y pueden ceder la CPU frecuentemente), y algunos procesos de larga duracion que necesitan mucho tiempo de CPU pero que el tiempo de respuesta no es importante. Aqui esta algoritmo de ajuste de prioridades:

- **Regla 3:** Cuando un trabajo entra en el sistema, es ubicado en la cola de mas alto nivel de prioridad.
- **Regla 4a:** Si un proceso usa una porcion de tiempo completa durante su ejecucion, re reduce su prioridad.
- **Regla 4b:** Si un procesos entrega la CPU antes de que la porcion de tiempo se acabe, se mantiene en el mismo nivel.

### Ejemplo 1: Un solo proceso de larga ejecucion

Supongamos que tenemos un MLFQ de 3 colas y un quantum de 10 ms. Un proceso largo ingresa en la Q2 (mas alta priority), se ejecuta hasta acabar el quantum y baja a la Q1 (que tiene menos priority que la Q2), como es el unico, se ejecuta hasta acabar el quantum y baja a la Q0, la cola de mas baja prioridad, donde se queda ahi a terminar de ejecutarse

![https://i.postimg.cc/02h3z9ZL/proceso-largo.png](https://i.postimg.cc/02h3z9ZL/proceso-largo.png)

### Ejemplo 2: En medio de un proceso laego llega uno corto

En este ejemplo tenemos dos procesos: A, el cual es un proceso largo y de uso intensivo de la CPU, y el B, es un proceso corto e interactivo. A se estuvo ejecutando un tiempo, esta en Q0, y luego llega B, el cual ingresa en Q2. Por lo tanto B se eejecuta por el quantum, pasa a Q1 y sigue siendo el de mayor prioridad, y ahi es donde se termina de ejecutar, A reanuda su ejecucion.

Dado que no se sabe si un proceso es largo o corto, se asume que es corto por eso siempre se pone en la cola de prioridad mas alta, asi si realmente es corto, se ejecuta hasta terminar; y si es largo lentamente ira bajando la prioridad. De esta manera MLFQ se parece a SJF

![https://i.postimg.cc/9XRsC4YG/along-came-1.png](https://i.postimg.cc/9XRsC4YG/along-came-1.png)

### Ejemplo 3: Que hay sobre I/O

La regla 4b declara que si un proceso entrega la CPU antes de consumir el quantum, mantiene su nivel de prioridad. La intencion de esta regla es que si un proceso interactvo esta haciendo, por ejemplo, muchas I/O (esperando mouse o teclado), cedera la CPU antes del quantum, en tal caso, MLFQ no quier penalizar el proceso y lo mantendra siempre que haga una peticion en ejecucion

![https://i.postimg.cc/CLwCJX5z/mixed-process.png](https://i.postimg.cc/CLwCJX5z/mixed-process.png)

El proceso interactivo B (gris) que necesita la CPU solo 1ms antes de hacer un I/O compitiendo con A que es de larga duracion (negro). MLFQ mantiene a B en la maxima prioridad dado que B mantiene la CPU liberada

### Problemas con este MLFQ

Hasta ahora tenemos un MLFQ basico. Parece que hace un buen y equitativo trabajo, compartiendo la CPU entre los procesos largos y dejando que los procesos cortos o interactivos se ejecuten rapidamente. 
Desafortunadamente, el enfoque que hemos desarrollado tiene un par de defectos.

Primero, hay un problema de **inanicion (starvation)**: si hay demasiados procesos interactivos en el sistema, combinados consumiran todo el tiempo de CPU, y por lo tanto los procesos de larga 
duracion nunca recibiran nada del tiempo de la CPU y estos **pasan hambre (*starve*)**. Nos gustaria hacer un progreso con estos procesos incluse en este escenario.

Un usuario inteligente podria reescribir su programa para jugar con nuestro planificador. Con "jugar con el planificador" nos referimos a la idea de hacer algo sigiloso (sneaky) para engañar al planificador y que le de mas de los recursos de los que tendria equitativamente. El algoritmo que describimos es suceptible al siguiente ataque: antes de que finaliceel quantum, lanza una operacion de I/O (para algun archivo que no nos interesa) y por lo tanto cede la CPU; hacer esto nos permite mantenernos en la misma cola de prioridad, y por lo tanto ganar el maximo porcentaje de CPU. Cuando se hace correctamente (es decir, usando el 99% del quantum antes de ceder la CPU), un proceso puede monopolizar la CPU.

Finalmente, un proceso puede cambiar su comportamiento a lo largo del tiempo, el que estaba vinculado al uso intensivo de la CPU puede cambiar a una fase interactiva. Con nuestro enfoque actual, tal proceso no tendria suerte y no seria tratado como los demas procesos interactivos 
en el sistema.

## Intento #2: Impulso de prioridad

Que se puede hacer para que los procesos que consuman mucha CPU hagan alagun progreso, por mas minimo que sea. La idea es ir elevando la prioridad de los procesos. La regla es simple:

- **Regla 5:** Despues de un tiempo S, mover todos los procesos del sistema a la cola de maxima prioridad

Esta regla soluciona dos problemas a la vez. Primero se garantiza que los procesos tengan algo de tiempo de CPU ubicandolos en la cola mas alta, haciendo un proceso compartira la CPU con otro proceso de maxima prioridad en una forma RR. Y segundo, si un proceso largo se vuelve interactivo se mantendra arriba gracias al **priority boost**

![https://i.postimg.cc/yx1PJbSg/priority-boost.png](https://i.postimg.cc/yx1PJbSg/priority-boost.png)

A la izquierda, no hay impulso de prioridad, por lo tanto el proceso largo no se alimenta (no consume CPU) una vez que llegan los dos procesos cortos; a la derecha, hay impulso de prioridad cada 50ms (el cual es un valor muy bajo, pero es usado a modo de ejemplo), por lo tanto al menos nos aseguramos que el proceso largo hara algun avance, siendo impulsado a la cola mas alto de prioridad cada 50ms y por lo tanto siendo ejecutado periodicamente.

Por supuesto, agregar el periodo de tiempo S no guia a la pregunta obvia: cuanto tiempo deber ser S?, estos tiempos son llamados **voo-doo constants**, dado que parecia que para darles un valor correcto era necesaria alguna forma de magia negra. Desafortunadamente, S es asi. Si es muy alto, los procesos largos no comeran; y si es muy chico, a los procesos interactivos no se les compartira CPU apropiadamente.

## Intento #3: un mejor conteo

Ahora tenemos un problema mas para resolver: como prevenir que los procesos jueguen con el planificador? Los verdaderos culpables son las reglas 4a y 4b, la cual le permite a un proceso 
mantener su prioridad cediendo la CPU antes de que se agote su porcion de tiempo. 

La solucion es hacer un mejor conteo del tiempo de CPU en cada nivel del MLFQ. En vez de olvidar cuanto de una porcion de tiempo uso un proceso en un nivel dado, el planificador le hara un seguimiento, una vez que un proceso haya hecho su parte, sera degradado de cola. Entonces si usa su porcion de tiempo de una sola vez o en varias partes no importara. Por lo tanto reescribiremos las regla 4a y 4b en una sola regla:

- **Regla 4:** Una vez el proceso haya usado su quantum, su prioridad es reducida, es decir, se mueve una cola abajo

![https://i.postimg.cc/BQ18TCcX/gaming-tolerance.png](https://i.postimg.cc/BQ18TCcX/gaming-tolerance.png)

La imagen (a la izquierda) muestra que sucede cuando una carga de trabajo intenta jugar con el planificador con las reglas 4a y 4b, y que sucede con la nueva regla 4 (a la derecha). Sin ninguna proteccion, un proceso puede lanzar una I/O justo antes de que finalice su porcion de tiempo y por lo tanto dominara la CPU. En cambio, con proteccion, sin importar el comportamiento del proceso sobre las I/O, lentamente se movera hacia abajo en las colas, y no puede ganar injustamente tiempo de CPU.

## **Modificando MLFQ y otros invonvenientes**

Algunos otros inconvenientes surgen con la planificacion MLFQ. Una de las grandes preguntas es como **parametrizar** tal planificador. Por ejemplo, cuantas colas deberia tener? Que tan grande deberia ser la porcion de tiempo para cada cola? Cada cuanto tiempo deberia hacer el impulso de prioridad para evitar que procesos no se alimenten y tomar los chambios en el comportamiento? Estas preguntas no son faciles de responder, por lo que solo la expericiencia con la carga de trabajo y la subsecuente modificacion del planificador guiaran a un balance sasisfactorio.

Por ejemplo, muchas variantes de MLFQ permiten modificar la porcion de tiempo en diferentes colas. La cola con mas prioridad por lo general tiene una menor porcion de tiempo; ya que estan compremetidas para procesos interactivos, despues de todo, alternar rapidamente entre ellos tiene sentido (10ms o menos). Las colas de baja prioridad, en contraste, contienen procesos de larga durancion que usan mucha CPU; por lo tanto una porcion de tiempo mas larga funciona bien (100ms o mas)

![https://i.postimg.cc/tJ6BGxPD/different-time.png](https://i.postimg.cc/tJ6BGxPD/different-time.png)

En la imagen se ve un ejemplo en el cual dos procesos se ejecutan por 20ms en la cola de prioridad mas alta (con una porcion de tiempo de 10ms), 40ms en la cola del medio (20s de porcion de tiempo), y con una porcion de tiempo de 40ms en la cola mas baja.

# MLFQ Final

- **Regla 1:** if Priority(A) > Priority(B) → ejecuta A (B no)
- **Regla 2:** if Priority(A) = Priority(B) → ejecuta A y B con RR
- **Regla 3:** Cuando un proceso entra al sistema es ubicado en la cola de prioridad mas alta
- **Regla 4:** Una vez que un proceso usa su quantum, baja su prioridad
- **Regla 5:** Despues de un periodo de tiempo S, todos los procesos pasan a la prioridad mas alta