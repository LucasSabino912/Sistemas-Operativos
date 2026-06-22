# MLFQ

**MLFQ** (Multi-Level Feedback Queue) busca **optimizar** tanto el **turnaround time** como el
**response time**, es decir, tener a la vez buena performance y ser interactivo, brindando una respuesta adaptativa a la carga.

## Reglas básicas
MLFQ cuenta con diferentes colas con diferentes niveles de prioridad cada una. Los procesos en estado ready quedan posicionados en ellas y el MLFQ decide cual correr en base a la prioridad de cada proceso.
Si el proceso **suelta el CPU** para esperar un I/O, el MLFQ mantiene su prioridad alta viendolo como un proceso interactivo.
En cambio, frente a usos prolongaos del CPU, el MLFQ le baja la prioridad

La MLFQ trata de aprender del comportamiento de los procesos para *predecir el futuro comportamiento*. Es decir, la **prioridad cambia** con el tiempo

Además, cada un tiempo S se **aumenta la prioridad** de todos los procesos poniéndolos en la cola de mayor prioridad. Esto evita starvation si se juntan demasiados procesos interactivos, que si un long-running job se vuelve interactivo el scheduler lo trate como tal (en vez de quedarse en las colas de baja prioridad), y que los procesos no puedan jugar con el scheduler

En resumen, las **reglas** de la MLFQ consisten en:
1) Si Prioridad(A) > Prioridad(B), se ejecuta A (B no).
2) Si Prioridad(A) = Prioridad(B), se ejecutan A y B en RR.
3) Cuando un trabajo ingresa al sistema, se coloca en la cola de prioridad más alta.
4) Una vez que un trabajo utilice su tiempo asignado en un nivel dado, su prioridad se reduce.
5) Después de un periodo de tiempo determinado (S), todos los trabajos del sistema se mueven a la cola de más alta prioridad.


Con esta última incorporación, las políticas de planificación estudiadas son:
- **FIFO**: First In First Out                                           (starvation, trade off turnarund)
- **SJF**: Shortest Job First                                          (starvation, trade off turnarund)
- **STCF**: Shortest Time to Completion First
- **RR**: Round Robin                                                      (quantum, response)
- **MLFQ**: Multi Level Feedback Queue                  (quantum, response)
