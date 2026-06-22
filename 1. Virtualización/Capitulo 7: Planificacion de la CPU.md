# Planificación de la CPU

Las políticas de planificación son algoritmos utilizados por el **scheduler** del SO para decidir en que orden y que procesos ejecutar

### Métricas de planificación
El **turnaround time** es la metrica que mide un proceso desde que llega al sistema hasta que finaliza
			**Tturnaround = Tcompletion - Tarrival**
		Tiempo de entrega = Tiempo de finalizacion - Tiempo de llegada

Tturnaround es una metrica de **performance**

Las computadoras actuales deben tener una performance interactiva con el usuario, y esto se mide en Response Time, el cual es el tiempo desde que el proceso llega hasta que es ejecutado por primera vez:
			**Tresponse = Tfirstrun - Tarrival**
		Tiempo de respuesta = Tiempo de primera ejecucion - Tiempo de llegada

# FIFO (First in, First out)
Fácil de implementar pero mal Tturnaround. Puede sufrir **convoy effect**, un proceso mas largo que los demas llega primero y relentiza el resto, ya que cada procveso corre hasta finalizar

# SJF (Shortest Job First)
Siempre corre el proceso **mas corto** primero, minimizando el Tturnaround promedio
Si llega un proceso lago será ejecutado, y si luego llega un corto tendra que esperar, lo mismo pasa si siguen llegando procesos cortos y uno largo puede llegar a nunca ejecutarse

# RR (Round Robin)
Corre los procesos durante un periodo de tiempo fijo llamado **Quantum**. Al ser cortado por el Q, el proceso va al final de una FIFO, cambia de proceso y se eejcuta durante Q tiempo, cada vez que el Quantum se cumpla, se cambia entre los procesos que siguen

### Incorporando I/O
Todos los programas que usan I/O no usan la CPU mientras realizan ese trabajo, ya que esperan a que este se complete (quedando el proceso en estado Blocked). Durante este tiempo, el scheduler puede correr otro proceso. 

Cuando el I/O termina, el Scheduler también decide qué hacer; si lo deja en espera (Ready) o si lo ejecuta nuevamente (Running, con un context switch).

De esta forma, se utiliza con más eficiencia el CPU, superponiendo cortos periodos de uso del CPU y el I/O a la vez entre programas, tratando cada pequeño tiempo de uso del CPU como un proceso y maximizar el uso de los recursos.
