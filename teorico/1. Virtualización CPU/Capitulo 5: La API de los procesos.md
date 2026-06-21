# La API de los procesos

La creación de procesos en Unix se da con dos system calls (llamadas al sistema): **fork()** y **exec()**.
Puede usarse wait() para esperar que un proceso creado termine.
Cada proceso tiene un identificador único, un número llamado **PID**.

- **argc**: entero que representa el numero de argumentos pasados a la linea de comandos. Incluye nombre de programa como primer argumento
- **argv[ ]**: array de punteros a cadenas de caracteres que contiene el nombre del programa en la posicion 0, y los argumentos del pasados al programa en las demas posiciones (argv[ argc ] = NULL)

## fork()
Crea un nuevo proceso. El proceso creado es una copia exacta del proceso padre. 
Tiene una copia del **address space** del padre pero usa su priopia memoria privada, sus propios **registros** pero con el mismo contenido, el mismo **PC**, etc. 
No toma argumentos y retorna 0 para el preoceso child y el pid del child para el proceso padre. 
Son dos ejecuciones diferentes y no deterministas, el scheduler va a decidir que se ejecuta en cada momento

## exec()
Es una familia de syscalls que se utilizan para **correr un programa diferente** al programa desde el **cual se la llama**. Toma como argumento el nombre del programa, carga el **ejecutable** y sobreescribe el segmento de código actual. Luego el SO corre ese programa

- **execv():** el primer argumento es el path al programa, el segundo argumento es un array de punteros a los argumentos, terminado en NULL.
- **execvp():** el primer argumento es el nombre file del programa (se busca en el path actual), el segundo argumento es un array de punteros a los argumentos.
- **execl():** se pasan los argumentos como una lista explícita en la llamada, uno por uno, terminando con NULL.

## wait() 
Es usada por un proceso parent para esperar a que el proceso child **termine de ejecutarse**. Recién en ese momento el parent continúa su ejecución.
Si un parent tiene múltiples childs, puede usar la versión waitpid() para especificar el pid de un child específico al cual esperar (si no, el retorno del wait se vuelve no determinista)

