# Capitulo 7 - Scheduling

El SO emplea unas politicas de alto nivel, llamadas **politicas de planificacion**. El origen de la planificacion procede de los sistemas computacionales

## Metricas de la planificacion

Una metrica se utiliza para medir los tiempos del proceso, hay diferentes para cada planificacion

t.turnaround = t.completioon - t.arrival

# First in, First out (FIFO)

El algoritmo mas basico que se puede implementar es **FIFO.**

Tiene muchas propiedades positivas: es simple y facil de implemetar

Un ejemplo rapido, llegan 3 procesos A, B y C al sistema, aprox. al mismo tiempo (t.arrival = 0). Dado que FIFO ubica a algun proceso primero, asumimos que A lleg antes que B, y que B apenas antes que C. Suponiendo que A termina en 10, B en 20 y C en 30. Cual sera el turnaroun time promedio?

a.turnaround = a.completion - a.arrival —> a.turnaround = 10

b.turnaround = b.completion - b.arrival —> b.turnaround = 20

c.turnaround = c.completion - c.arrival —> c.turnaround = 30

(10 + 20 + 30) / 3 = 20

Ahora suponiendo que tienen tiempos de ejecucion distintos, y que A tarda 100, B y C 10 cada uno

Por lo tanto el tiempo de turnaround promedio sera:

a.turnaround = a.completion - a.arrival —> a.turnaround = 100

b.turnaround = b.completion - b.arrival —> b.turnaround = 110

c.turnaround = c.completion - c.arrival —> c.turnaround = 120

(100 + 110 + 120) / 3 = 110

# Shortest job first (SBJ)

**Shortest job first** es como dice el nombre, ejecuta el proceso mas corto primero, luego el siguiente mas corto y asi sucesivamente

Ejemplo anterior, A dura 100, B y C duran 10. Simplemente ejecutando B y C antes que A, se reduce el tiempo promedio de 110 a 50, menos de la mitad del tiempo

Ahora suponiendo que a.arrival = 0 y b.arrival = 10 = c.arrival, B y C estan forzados a que A termine para ejecutarse, por lo que en este ejemplo seria lo mismo o muy paraecido a FIFO

# Shortest time-to-completion First (STCF)

Cuando entra un nuevo proceso al sistema, este planificador determina a cual de los procesos que quedan le queda menor tiempo, y planifica ese. Por lo tanto, en el ejemplo anterior, STCF reemplaza A, y ejecuta B y C hasta completarse; solo entonces podra planificarse el tiempo que le queda a A.

Basicamente:

a.completion = 100, a.arrival = 0

b.completion = 10, b.arrival = 10

c.completion = 10, c.arrival = 10

A se ejecuta durante el tiempo que sea el unico, luego llegan B y C y toman prioridad porque tardan menos. Se frena A, se ejecutan B y C respectivamente hasta completarse y luego sigue A

((120 - 0) + (20 - 10) + (30 - 10)) / 3 = 50 ← turnaround time

## Una nueva metrica: Tiempo de respuesta

Si conocemos la longitud de los procesos, y que procesos usan la CPU, y la unica metrica fuera el turnaround time, STCF seria una gran politica. Sin embargo, la introduccion de maquinas de timepo compartido cambio esto. Ahora los usuarios pueden sentarse en la terminal y demandar un desesmpeño interactivo del sistema,  es por esp que se define el **response time**

T.respone = T.firstrun - T.turnaround

# Round Robin

La idea de **RR** es simple: en vez de ejecurtar procesos hasta que finalicen, RR ejecuta un proceso por porcion de tiempo (**quantum shceduling**) y entonces cambia al siguiente proceso en la cola. Hace esto repetidamente hasta que todos los procesos se completen. 

Podemos tener un RR con time slice de lo que queramos, 1ms, 10m, etc

Si tenemos A, B y C con un time slice de 1s, nuestro tiempo de respuesta promedio con **RR** sera:

(0 + 1 + 2) / 3 = 1

Y con **SJF** sera:

(0 + 5 + 10) / 3 = 5

La eleccion del time slice en RR es critica, cuanto mas corta, mejor sera el desempeño de RR bajo la metrica de tiempo de respuesta. Sin embargo hacerla muy corta puede traer un problema: el costo de cambio de contexto dominaria todo el desempeño. Por lo tanto, decidir el time slice tiene que tener en cuenta hacerlo lo suficientemente largo para **amortizar** el costo del cambio de contexto sin hacerlo tan largo que el ssitema no sea sensible al tiempo de respuesta

## Incorporando I/O

Imaginemos que un programa no puede tomar una entrada, esto producira la misma salida todo el tiempo. Imaginemos uno sin salida: es el proverbio del arbol que se cae en el bosque (si un arbol en el bosque se cae, pero nadie lo escucha, hace ruido?), si nadie puede ver lo que hace el programa; es como si no se hubiera ejecutado.

Un planificador, claramente, tiene una decision que tomar cuando un proceso inicia una peticion de I/O; porque el proceso que se esta ejecutando actualmente no puede usar la CPU dirante la I/O; se **bloquea** esperando que se complete la I/O. Si la I/O en enviada al disco duro, el proceso sera bloqueado por algunos milisengundos o mas, dependiendo la carga real I/O del disco. Por lo tanto, el planificador deberia planificar otro proceso en la CPU en ese tiempo.

El planificador tambien debera tomar un descision cuando una I/O se completa. Cuando esto ocurre, se genera una interrupcion, y el OS cambio es el estado *blocked*del proceso que hizo la peticion a *ready*. Por supuesto , todavia debera decidir ejecutar el proceso o no. Como deberia tratar el OS a cada proceso?

Para entender mejor este problema, vamos a asumir que tenenos dos procesos, A y B, los cuales necesitan 50ms de tiempo de CPU. Sin embargo, hay una diferencia obvia: A se ejecuta por 10ms y emite una peticion de I/O (aqui asumimos que cada I/O tarda 10ms), cuando B se ejecuta por 50ms y no hace ninguna I/O. El planificador ejecuta primero A y despues B.

Ahora asumamos que estamos tratando de construir un planificador STCF. Como explicaria un planificador de este tipo el hecho de que A se divida en 5 subtrabajos de 10ms, cuando B solo demanda 50ms de CPU? Claramente, ejecutar un proceso y recien despues otro sin tener en cuenta las I/O no tiene sentido

Un enfoque comun es trata cada subproceso de 10ms de A como un proceso independiente. Por lo tanto, cuando el sistema comienza, su opcion seria, ya sea ejecutar 10ms A o ejecutar 50ms B. Con **STCF**, la opcion es clara: elejir la mas corta, en este caso A. Entonces, cuando la primer 
subproceso de A se completa, solo queda B y empieza a ejecutarse. 
Entonces un nuevo subproceso de A en mandado y reemplaza a B y se ejecuta por 10ms. Hacer esto permite la **superposicion (overlap)**, con la CPU siendo usada por otro proceso mientras espera que el I/O de otro proceso se complete; por lo tanto el sistema es mejor utilizado (de forma mas eficiente).


### En resumen

FIFO --> El primero que llega se ejecuta
SBJ --> Se ejecuta primero el mas corto
STCF --> Ejecuto primero el que tarde menos tiempo en completarse
RR --> Ejecuto varios procesos por un cierto periodo de tiempo (quantum)

Si buscamos menos tiempo de entrega -> Ejecutamos los mas cortos primero
Si buscamos menos tiempo de respuesta -> El sistema se siente interactivo, pero se aumenta el tiempo de entrega
