# Ejecución directa limitada

Para virtualizar el CPU el SO necesita compartir el CPU físico entre varios trabajos simultáneos. La idea principal es ejecutar uno un poco y cambiar a otro rápidamente generando la ilusión de procesamiento simultáneo. 

Con **Time Sharing se alcanza** la virtualización.

Además, debe lograrse asegurando la **performance** (virtualizar sin sobrecargar el sistema) y el
control (como correr cada proceso mientras mantenemos control del CPU).

La **ejecución directa limitada** consiste en correr directamente el programa en la CPU; el SO hace los preparativos:
- Crea un nuevo PCB
- Reserva memoria para el programa 
- Carga desde el sistema de archivos el código ejecutable a la memoria 
- Pone el argc y argv en la pila (stack), limpia los registros
- Llama al main y lo ejecuta.

### Operaciones restringidas
En **user mode** (restringido) el código corre con restricciones (como accesos a I/O o a memoria no
permitida; hacerlo llevaría a una excepción, lo que haría que el SO termine el proceso).

En **kernel mode** (privilegiado), modo en el que funciona el SO, el proceso no tiene restricciones y
puede hacer operaciones privilegiadas.

El modo de usuario cuenta con System calls para solicitar operaciones que tiene restringidas. Estas instrucciones ejecutan una **trap** que salta a **kernel mode** elevando el privilegio y realizando la instrucción (si el SO la permite). Al terminar, se realiza un **return from trap** y baja el nivel de privilegio volviendo a user mode.

Antes de ejecutar una trap se guardan los registros (contexto) del proceso que llamó a la trap, en un kernel stack (uno por proceso), para su posterior restablecimiento al volver a user mode. 
El kernel debe verificar que código ejecutar cuando ocurran determinadas excepciones, para lo cual el SO setea una trap table al momento del booteo que establece eso y además indica la localización de los trap handlers. Estos últimos serán ejecutados por el SO en modo kernel.

Esta indirección garantiza seguridad y genera una abstracción que permite cambiar de kernel
mientras se mantenga el número id de las system calls.
Una trap puede ser ocasionada por:
- Un timer interrupt (evento asíncrono)
- Un hardware device interrupt (evento asíncrono)
- Una syscall (evento síncrono)
- Una exception (errores, acceso indebido a memoria; evento síncrono)
El SO reacciona a cualquiera de ellas de la misma forma (trap handler).

Para especificar la system call, se usa una **system-call number**; el código que el usuario señala que
quiere ejecutar y que el kernel en el trap handler verifica y, si es válida, ejecuta. Esto ofrece
protección para no darle control total al usuario (para que no salte a una dirección de memoria no
autorizada), pero falla al no controlar el input que el usuario usa como argumento de las syscalls.

En **LDE** en el booteo inicia la trap table y el CPU recuerda su localización (operación privilegiada).
Luego, cuando corre un proceso, el kernel configura algunas cosas (nodo en el process list, allocating memory) antes de hacer el return from trap, para ejecutar el proceso cambiando el CPU a user mode.
## Intercambio entre procesos
Si un proceso corre en el CPU, el SO no está corriendo. Sin embargo, el mismo debe asegurarse poder recuperar el control para cambiar de proceso y lograr time sharing.

### Enfoque colaborativo: esperar a nuevas system calls
El SO confía en el proceso en ejecución y da por sentado que cada cierto periodo liberaría el CPU
para que el SO decida que correr; al ejecutarse una trap o al llamar a una System call (incluso de
manera explícita, con la llamada yield) se cedería y transferiría el control al SO.

### Enfoque no colaborativo: el SO toma el control
Utilizando mecanismos del hardware, el SO puede retomar el control del CPU mediante el Timer
interrupt; un dispositivo que cada varios milisegundos realiza una interrupción. Esta cuando ocurre se ejecuta un interrupt handler en el SO, que le permite retomar el control.
En el booteo el SO deja explicitado que código correr en una interrupción, momento en el que
además comienza el timer.
El hardware se encarga de guardar el estado del proceso actual al momento de la interrupción para su posterior return-from-trap.


### Guardar y restaurar el contexto
Si cuando el SO toma control y decide cambiar a otro proceso (usando la rutina switch), ejecuta un
código de bajo nivel, el context switch, que le permite guardar los valores de los registros del
programa en ejecución (en el kernel stack del proceso) y restaurar otros para el proceso que pasará a ejecutarse (desde su kernel stack).
Si hay un timer interrupt(trap), es el hardware quien guarda los registros en el stack de kernel.
Si hay un switch por parte del SO, es el software quien guarda/restaura los registros del kernel en la estructura del proceso.
Cada vez que se da un context switch, por ej. ante una syscall exception, el proceso pierde
performance, y tarda determinados ciclos en retomar la velocidad que tenía antes de la misma.

### Concurrencia
Para que no ocurran interrupts simultáneas, se suelen deshabilitar las interrupciones mientras se
está lidiando con interrupciones. También se usan locks para proteger las estructuras internas.

En resumen, en LDE se ejecuta directamente al programa en el CPU, habiendo configurado antes al hardware para poder limitar lo que cada proceso puede ejecutar, y habiendo el SO preparado a la CPU configurando al momento del boot al controlador de traps y al timer de interrupciones, y luego ejecutando procesos solo en modo restringido.
