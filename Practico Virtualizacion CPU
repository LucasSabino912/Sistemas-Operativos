**Ejercicio 1.** En un sistema operativo que implementa procesos se ejecutan instancias del proceso pi que computa los dígitos de pi con precisión arbitraria.
```bash
$ time pi 1000000 > /dev/null & ... & time pi 1000000 > /dev/null &
```
Y se registran los siguientes resultados, donde en las mediciones se muestra (real, user), es decir el tiempo del reloj de la pared (walltime) y el tiempo que insumió de CPU (cputime).
**Instancias**                         **Medición**
1                                          (2.56,2.44)
2                                          (2.53,2.42), (2.58,2.40)
1                                          (3.44,2.41)
4                                          (5.12,2.44), (5.13,2.44), (5.17,2.46), (5.18,2.46)
3                                          (3.71,2.42), (3.85,2.42), (3.86,2.44)
2                                          (5.04,2.36), (5.09,2.43)
4                                          (7.67,2.41), (7.67,2.44), (7.73,2.44), (7.75,2.46)
(a) ¿Cuantos núcleos tiene el sistema?
(b) ¿Porque a veces el cputime es menor que el walltime?
(c) Indique en la Descripción que estaba pasando en cada medicion.

a) Vemos que en la primera corrida una instancia tiene 2.56 de real time, en la segunda vemos que 2 instancias del mismo proceso tienen un real time casi igual, lo que nos da la pauta de que tiene mínimo 2 núcleos. Si vemos la corrida que tiene 3 instancias vemos que el real time sube un poco y en la de 4 instancias se duplica en relación a la de 2 instancias. Por lo tanto el sistema tiene **2 núcleos**

b) El walltime es mayor porque el cputime solo toma el tiempo en el que el proceso es ejecutado, y el walltime toma desde que estaba en la cola de ejecución esperando a pasar de **ready** a **running**. cputime **solo** toma cuando el proceso esta en **running** 

**Ejercicio 2.** En un sistema operativo que implementa procesos e hilos se ejecutan el siguiente proceso. Explique porque ahora walltime < cputime.
```bash

	$ time ./dgemm 2000 2000 2000
	test!
	m=2000,n=2000,k=2000,alpha=1.200000,beta=0.001000,sizeofc=4000000
	real 0m1.027s
	user 0m1.752s
```
Al implementar hilos (multithreading), el programa `dgemm` dividió su trabajo en múltiples tareas simultáneas.

Si el planificador del sistema operativo agarró 2 hilos de este programa y puso a correr uno en el Núcleo 1 y el otro en el Núcleo 2 **al mismo tiempo**, ambos núcleos empiezan a sumar tiempo de CPU en paralelo.

Imaginemos un escenario ideal:
1. El programa arranca y lanza dos hilos.
2. Vos mirás tu reloj de pared durante **~0.876 segundos** (tiempo `real`).
3. Durante esos 0.876 segundos, el Núcleo 1 trabajó al 100%. Suma **0.876s** de `cputime`.
4. Durante esos mismos 0.876 segundos, el Núcleo 2 trabajó al 100%. Suma otros **0.876s** de `cputime`
5. El sistema operativo suma el esfuerzo total de la CPU: `0.876 + 0.876 =` **1.752 segundos de `user time`**.
Siempre que el cputime sea mayor al realtime significa que el programa esta utilizando paralelizacion y multihilos

**Ejercicio 3.** Describir donde se cumplen las condiciones user<real, user=real, real<user.

**Caso `user < real`**
Cuando el proceso **tiene que esperar** y no puede usar la CPU durante todo el tiempo que dura su ejecución.

**Condiciones que lo provocan:**
- **Carga alta en el sistema (Multiprogramación):** Hay muchos otros procesos compitiendo. Tu proceso pasa tiempo en estado **READY** (listo) esperando que el planificador (scheduler) le asigne un quantum de CPU.
- **Operaciones de Entrada/Salida (I/O):** El proceso necesita leer un archivo del disco, esperar un paquete de red o esperar que el usuario teclee algo. Pasa a estado **BLOCKED** (bloqueado) y la CPU se le asigna a otro programa.
- **Llamadas a dormir (`sleep`):** El proceso suspende explícitamente su ejecución por un tiempo determinado.

**Caso `user ≈ real`** 
**¿Cuándo ocurre?** Cuando el proceso tiene el monopolio de un núcleo del procesador desde que arranca hasta que termina, sin interrupciones.

**Condiciones que lo provocan:**
- Es un programa **monohilo** (single-threaded).
- Es un programa **CPU-bound** (limitado por CPU), es decir, solo hace cálculos matemáticos pesados en memoria (como calcular Pi) y **no** hace operaciones lentas de Entrada/Salida (no lee disco, no usa red).
- El sistema operativo está "ocioso" (libre de carga externa), por lo que el proceso casi nunca es expulsado del estado **RUNNING** para darle lugar a otro.

**Caso `real < user`**
**¿Cuándo ocurre?** Cuando el programa ejecuta múltiples tareas simultáneamente en distintos núcleos físicos.

**Condiciones que lo provocan:**
- El programa utiliza **paralelismo puro** (mediante múltiples hilos / _multithreading_ o procesos hijos).
- El hardware es **Multicore** (multinúcleo).
- Al ejecutarse en paralelo, el tiempo `user` suma el tiempo de procesamiento de _todos_ los núcleos al mismo tiempo. Por ejemplo, si 4 hilos corren a tope durante 1 segundo real en 4 núcleos distintos, el tiempo `real` será 1 segundo, pero el tiempo `user` será 4 segundos.

**Ejercicio 4**. Un programa define la variable int x=100 dentro de main() y hace fork().
(a) ¿Cuanto vale x en el proceso hijo?
(b) ¿Que le pasa a la variable cuando el proceso padre y el proceso hijo le cambian de valor?
(c) Contestar nuevamente las preguntas si el compilador genera código de maquina colocando esta variable en un registro del microprocesador.

El proceso padre copia todas sus variables con sus valores a sus hijos, pero cada hijo tiene su propia copia, por lo que si el valor cambia en el hijo o en el padre se mantiene como estaba en los demás procesos
Aunque la variable esté en un registro físico del microprocesador, gracias a la virtualización de la CPU y al guardado/restaurado de registros durante el cambio de contexto, **el padre y el hijo siguen operando en universos totalmente aislados** y los cambios de uno no afectan jamás al otro.


**Ejercicio 5.** Indique cuantas letras “a” imprime este programa, describiendo su funcionamiento.
```c
printf("a\n"); // Imprime 1
fork();
printf("a\n"); // Imprime 2 + 1 = 3
fork();
printf("a\n"); // Imprime 4 + 3 = 7
fork();
printf("a\n"); // Imprime 8 + 7 = 15
```
Generalice a n forks. Analice para n=1, luego para n=2, etc., busque la serie y deduzca la expresión
general en función del n.

Imprime 15 'a', y la formula general es 2^n−1

**Ejercicio 6.** Indique cuantas letras “a” imprime este programa
```c
char * const args[] = {"/bin/date", "-R", NULL};
execv(args[0], args);
printf("a\n");
```
Imprime 1 'a' solamente si `execv()` falla

**Ejercicio 7.** Indique que hacen estos programas
```c
int main(int argc, char ** argv) {
	if (0 < --argc) {
		argv[argc] = NULL;
		execvp(argv[0], argv);
	}
	return 0;
}

/* Si la linea de comandos se ingresan mas de un argumento (nombre de programa,
   y algun otro argumento), entonces se ejecuta el programa. Anteriormente, 
   hace NULL la ultima posicion de argv para que */
```

```c
int main(int argc, char ** argv) {
	if (argc<=1)
		return 0;
	
	int rc = fork();
	if (rc<0) // Fallo el fork
		return -1;
	else if (0==rc) // Proceso hijo
		return 0;
	else {
		argv[argc-1] = NULL;
		execvp(argv[0], argv);
	}
}

/* Recorta el arreglo y ejecuta el archivo pero solamente en el proceso padre.
   El peoblema es que el padre no hace wait entonces se reescribe su cdoigo y 
   se olvida del hijo, lo que puede provocar que el hijo quede como proceso zombi*/
```

Ejercicio 8. Si estos programas hacen lo mismo. ¿Para que esta la syscall dup()? ¿UNIX tiene un
mal diseño de su API?
```c
close(STDOUT_FILENO);
open("salida.txt", O_CREAT | O_WRONLY | O_TRUNC, S_IRWXU);
printf("¡Mira mama salgo por un archivo!");
```
```c
fd = open("salida.txt", O_CREAT | O_WRONLY | O_TRUNC, S_IRWXU);
close(STDOUT_FILENO);
dup(fd);
printf("¡Mira mama salgo por un archivo!");
```
Suponiendo que los fd 0, 1 y 2 están usándose
El primer código cierra el fd 1, entonces open toma el mas bajo abierto el cual es 1
printf siempre escribe a ciegas en el 1 asi que funciona

El segundo código hace lo mismo pero no depende del fd mas bajo
lo que hace dup es una copia del fd en el fd disponible mas bajo
Entonces al abrir un fd nuevo y hacer dup, se crea en el cerrado e cual es el 1, lo que evita condiciones de carrera seria usar dup2, que duplica y redirecciona en la misma accion

**Ejercicio 11.** Dentro de xv6 el archivo x86.h contiene struct trapframe donde se guarda toda la información cuando se produce un trap. Indicar que parte es la que apila el hardware cuando se
produce un trap y que parte apila el software.

El **hardware** guarda lo **minimo e indispensable** para garantizar que se pueda retornar al programa original mas tarde. Especificamente el hardware apila:
- **Program counter**: Para recordar en que linea de codigo se quedo el programa
- **Registro de estado (flags):** Para no perder el resultado de operaciones logicas recientes
- **Stack pointer:** El puntero a la pila de usuario para saber donde estaba la memoria del programa

El **software** del kernel **apila todo el resto del estado de la CPU**. Específicamente:
- **Todos los registros de uso general:** Registros como `eax`, `ebx`, `ecx`, `edx`, etc.
- A esta estructura completa que queda armada en la Pila del Kernel (los registros generales del software + los registros críticos del hardware) se la conoce como **Trap Frame** (Marco de Trampa). En procesadores x86, el SO suele hacer esto ejecutando una sola instrucción de assembly llamada `pusha` o `pushal` (_Push All_).

**Ejercicio 12.** Verdadero o falso. Explique.
	(a) Es posible que user+sys < real. **VERDADERO.** Si el proceso pasa muchi tiempo bloqueado o esperando esto puede cumplirse
	(b) Dos procesos no pueden usar la misma dirección de memoria virtual. **FALSO** Pueden tener la misma direccion virtual, el hw y so se encargan de mapearla a direcciones de memoria en la ram completamente distintas
	(c) Para guardar el estado del proceso es necesario salvar el valor de todos los registros del microprocesador. **FALSO** Solo se gurda el del proceso
	(d) Un proceso puede ejecutar cualquier instrucción de la ISA. **FALSO** Solamente los que tiene permiso
	(e) Puede haber traps por timer sin que esto implique cambiar de contexto. **VERDADERO** Se puede disparar una trap y esto no implica que se realice un cambio de contexto
	(f) fork() devuelve 0 para el hijo, porque ningun proceso tiene PID 0. **FALSO** El fork devuelve 0 como simple convencion de diseño de la API
	(g) Las syscall fork() y execv() estan separadas para poder redireccionar los descriptores de archivo.
	(h) Si un proceso padre llama a exit() el proceso hijo termina su ejecución de manera inmediata. **FALSO** El hijo no muere, pasa a convertirse en huérfano
	(i) Es posible pasar información de padre a hijo a través de argv, pero el hijo no puede comunicar información al padre ya que son espacios de memoria independientes. **FALSO** Son espacios de memoria totalmente distnitos
	(j) Nunca se ejecuta el codigo que esta despues de execv(). **FALSO** Si execv falla entonces el codigo despues se ejecuta
	(k) Un proceso hijo que termina, no se puede liberar de la Tabla de Procesos hasta que el padre no haya leıdo el exit status via wait(). **VERDADERO**

### Políticas

**Fórmulas:** * $T_{turnaround} = T_{completion} - T_{arrival}$

**Ejercicio 13.** Dados tres procesos CPU-bound puros A, B, C con Tarrival en 0 para todos y Tcpu de
30, 20 y 10 respectivamente. Dibujar la linea de tiempo para las políticas de planificación FIFO y
SJF. Calcular el promedio de Tturnaround y Tresponse para cada política.

**FIFO**: Asumo que entraron A -> B -> C
Línea de tiempo
0 ---- 30 ---- 50 ---- 60
A      A/B      B/C      C

**SCF:** C -> B -> A
0 ---- 30 ---- 50 ---- 60
C      C/B      B/A      A

**Ejercicio 14.** Para esto procesos CPU-bound puros dibujar la l´ınea de tiempo y completar la tabla
para las políticas apropiativas (con flecha de running a ready): STCF, RR(Q=2). Calcular el promedio
de Tturnaround y Tresponse en cada caso.

**1. STCF (Shortest Time-to-Completion First / Apropiativo)**

El planificador siempre elige el proceso al que le falte menos tiempo. Si llega uno más corto, expulsa al actual.

- **t=0:** Llega B. Se ejecuta.
    
- **t=2:** Llega A (le faltan 4). A B le falta 1. Sigue B.
    
- **t=3:** Termina B. Entra A.
    
- **t=4:** Llega C (le falta 1). A A le faltan 3. ¡C expulsa a A! Entra C.
    
- **t=5:** Termina C. Vuelve A y completa los 3 restantes.
    
- **t=8:** Termina A.
    

**Línea de tiempo:** `[0]- B -[3]- A -[4]- C -[5]--- A ---[8]`

**2. RR (Round Robin con Quantum = 2)**

Alterna procesos cada 2 unidades de tiempo.

- **t=0:** Cola [B]. B ejecuta 2. Le falta 1.
    
- **t=2:** Llega A. Cola [B, A] (B pasa al fondo por fin de quantum). Ejecuta A por 2.
    
- **t=4:** Llega C. Cola [A, C, B]. Ejecuta B por 1 (termina en 5).
    
- **t=5:** Cola [A, C]. Ejecuta C por 1 (termina en 6).
    
- **t=6:** Cola [A]. Ejecuta A sus 2 restantes (termina en 8).
    

**Línea de tiempo:** `[0]- B -[2]- A -[4]- B -[5]- C -[6]- A -[8]`

**Ejercicio 15.** Las políticas de planificación se pueden clasificar en dos grades grupos: por lotes
(batch) e interactivas. Otra criterio posible es si la planificación necesita el TCP U o no. Clasificar
FCFS, SJF, STCF, RR, MLFQ segun estos dos criterios.

Clasificación de las políticas:

1. **FCFS:** Por Lotes (Batch) | **NO** necesita saber el Tcpu a priori.
    
2. **SJF:** Por Lotes (Batch) | **SÍ** necesita saber el Tcpu a priori.
    
3. **STCF:** Por Lotes (Apropiativa) | **SÍ** necesita saber el Tcpu a priori.
    
4. **RR:** Interactiva (Time-sharing) | **NO** necesita saber el Tcpu a priori.
    
5. **MLFQ:** Interactiva | **NO** necesita saber el Tcpu a priori (lo deduce sobre la marcha).

**Ejercicio 17.** Realice el diagrama de planificaci´on para un planificador MLFQ con cuatro colas
(Q=1, 2, 4 y 8) para los siguientes procesos CPU-bound:
Proceso Tarrival Tcpu
   A                0          7
   B                1          3
   C                2          4
   D                4         3
   E                 7         4


**Traza de MLFQ:** Colas Q0(Q=1), Q1(Q=2), Q2(Q=4), Q3(Q=8). Todo proceso nuevo entra en Q0.

- **t=0:** A entra en Q0. Usa su Q=1 y baja a Q1 (le faltan 6).
    
- **t=1:** B entra en Q0. Usa su Q=1 y baja a Q1 (le faltan 2).
    
- **t=2:** C entra en Q0. Usa su Q=1 y baja a Q1 (le faltan 3).
    
- **t=3:** Q0 está vacía. Ejecuta A en Q1.
    
- **t=4:** A usó 1 tick de su quantum en Q1. **¡Llega D a Q0!** D interrumpe a A (A se queda en Q1). D usa su Q=1 y baja a Q1 (le faltan 2).
    
- **t=5:** Q0 vacía. A termina su tick restante en Q1 (t=5 a 6) y baja a Q2 (le faltan 4).
    
- **t=6:** B usa su quantum en Q1. (t=6 a 7).
    
- **t=7:** **¡Llega E a Q0!** E interrumpe a B. E usa su Q=1 y baja a Q1 (le faltan 3).
    
- **t=8:** Q0 vacía. B completa su 2do tick en Q1. ¡B termina! (T=9).
    
- **t=9:** C usa sus 2 ticks en Q1 (t=9 a 11) y baja a Q2 (le falta 1).
    
- **t=11:** D usa sus 2 ticks en Q1. ¡D termina! (T=13).
    
- **t=13:** E usa sus 2 ticks en Q1 (t=13 a 15) y baja a Q2 (le falta 1).
    
- **t=15:** Q0 y Q1 vacías. Ejecuta A en Q2. Usa sus 4 ticks (t=15 a 19). ¡A termina!
    
- **t=19:** Ejecuta C en Q2. Usa el tick que le falta y termina (T=20).
    
- **t=20:** Ejecuta E en Q2. Usa el tick que le falta y termina (T=21).

**Ejercicio 18.** Verdadero o falso.

**(a) FALSO.** Un proceso puede devolver el control _voluntariamente_ antes de que se le acabe el quantum si hace una llamada al sistema para Entrada/Salida (I/O) o si ejecuta explícitamente la instrucción `yield()`. Además, puede perder el control si llega un proceso de mayor prioridad a otra cola.

**(b) VERDADERO.** SJF (Shortest Job First) está demostrado matemáticamente como la política óptima para minimizar el promedio de Tturnaround en sistemas por lotes sin interrupciones. Por lo tanto, SJF siempre será igual o superior a FCFS en esta métrica.

**(c) VERDADERO.** Si el quantum de un Round Robin es infinito, el temporizador (timer interrupt) jamás expulsará al proceso de la CPU. El proceso correrá de principio a fin, y luego seguirá el próximo de la cola, exactamente igual que FCFS.

**(d) VERDADERO.** Si hay un flujo constante de procesos interactivos cortos entrando a las colas altas, los procesos pesados (CPU-bound) que cayeron a las colas inferiores nunca tendrán turno para usar el procesador, provocando inanición (starvation). El _priority boost_ los sube a todos periódicamente para evitar esto.

**(e) VERDADERO.** Antes, los procesos maliciosos podían soltar la CPU llamando a `yield()` justo al 99% de su quantum; el planificador viejo les renovaba el quantum entero al volver y nunca bajaban de prioridad. OSTEP detalla que la regla moderna de MLFQ lleva la cuenta del "tiempo total consumido" (Allotment) en ese nivel para castigar al proceso y bajarlo de cola cuando suma el 100%, sin importar si lo hizo de una vez o en muchos pedacitos.
