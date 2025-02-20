# Capitulo 15 - Traduccion de direcciones

En el desarrollo de la **virtualizacion de la CPU**, nos centramos en un mecanismo general conocido como **ejecucion directa limitada (LDE)**. 

La idea detras de **LDE** es simple: para la mayor parte, dejar al programa ejecutarse directamente en el hardware; sin embargo, en ciertos puntos clave de tiempo (como cuando un procesos hace una system call, o cuando ocurre una interrupcion), organizar que el OS se involucre y se asegure 
de que pasen cosas "correctas".

En la **virtualizacion de la memoria**, propusimos una estrategia similar, logrando eficiencia y control mientras proveiamos la virtualizacion deseada. 

Finalmente, necesitaremos un poco mas que solo VM, en terminos de *flexibilidad*; especificamente, nos gustaria que los programas sean capaces de usar su espacio de direcciones en la forma que ellos quieran, haciendo al sistema mas facil de programar. 

Y llegamos al punto crucial: Como virtualizar memoria de forma eficiente y flexible? Como podemos mantener control sobre a que direcciones de memoria puede acceder una aplicacion, y asegurar que el acceso a memoria de la aplicacion esta restringido apropiadamente? Como hacemos todo esto eficientemente? la tecnica general que vamos a usar, la cual puede considerar una adicion a la ejecucion derecta limitada, es algo a lo cual nos referimos como **traduccion de direccion.** 

Con la traduccion de direcciones, el hardware tranforma cada acceso a memoria (fetch, load, store), cambiando la direccion **virtual** provista por la instruccion a una direccion **fisica** donde esta guardada la informacion deseada. Por lo tanto, en cada referencia a memoria, un traductor de direcciones es ejecutado por el hardware para redirigir referencia de memoria de la aplicacion a la 
ubicacion real en memoria.

la **traducción de direcciones** en la virtualización de la memoria, un proceso en el que el hardware transforma direcciones virtuales generadas por un programa en direcciones físicas reales en la memoria. Esto permite que múltiples programas se ejecuten sin interferir entre sí, proporcionando la ilusión de que cada uno tiene su propio espacio de memoria privado.

Para lograr esta virtualización, se usa la **reubicación dinámica** basada en hardware, implementada con dos registros clave en la CPU: **base** y **límite**.

- El registro **base** almacena la dirección física donde comienza el espacio de direcciones del proceso.
- El registro **límite** define el tamaño máximo del espacio de direcciones permitido para evitar accesos indebidos.

Cuando un proceso genera una dirección virtual, el hardware la traduce sumándole el valor del registro base, obteniendo así la dirección física correspondiente. Si la dirección está fuera del límite permitido, se lanza una excepción.

El **sistema operativo (OS)** juega un papel crucial en este mecanismo:

1. **Asigna memoria** al iniciar un proceso.
2. **Libera memoria** cuando un proceso finaliza.
3. **Gestiona cambios de contexto**, actualizando los registros base y límite al cambiar entre procesos.
4. **Maneja excepciones** cuando un proceso intenta acceder a memoria no permitida.

En resumen, la traducción de direcciones permite una virtualización eficiente y controlada de la memoria, asegurando que los procesos sean independientes y que el OS mantenga el control del sistema.

### Un ejemplo

Para entender mejor que necesitamos para implementar la traduccion de direcciones, y porque necesitamos tal mecanismo, vemos un simple ejemplo. Imagina un procesos cuyo espacio de direcciones es el siguiente:

![https://i.postimg.cc/pXNXK5WZ/address-space2.png](https://i.postimg.cc/pXNXK5WZ/address-space2.png)

```c
void func(){
	int x = 3000;
	x = x + 3;
}
```

El compilador traduce esto a codigo assembly:

```nasm
movl 0x0 (%ebx), %eax ; load 0+eax
addl $0x03, %eax      ; add 3 to eax register
movl %eax, 0x0 (%ebx) ; store eax back to mem
```

Cuando se ejecutan las instrucciones, desde la perspectiva del proceso, se ejecutan los siguientes accesos a memoria:

- **Fetch**: se busca al instruccion en la direccion 128
- **Execute**: se ejecuta la instruccion (cargar de la direccion 15KB)
- **Fetch**: se busca la instuccion en la direccion 132
- **Execute**: se ejecuta la instruccion (sin referencia a memoria)
- **Fetch**: se busca la instruccion en la direccion 135
- **Execute**: se ejecuta la instruccion (guardar en la direccion 15KB)

Desde la perspectiva del programa, su **espacio de direcciones** comienza en la direccion 0 y crece hasta un maximo de 16KB; todas las referencias de memoria que genera deben estar en ese rango. Sin embargo, al virtualizar memoria, el OS quiere ubicar el proceso en cualquier parte de la memoria fisica, no necesariamente en la direccion 0. Por lo tanto, tenemos un problema: como podemos **reubicar** ese proceso en la memoria de forma que sea **transparente** para el proceso? Como podemos proprosionar la ilusion de un espacio de direcciones virtual que empiece en 0, cuando en realidad el espacio de direcciones esta ubicado en algun otro lugar de la memoria fisica?

## **Reubicacion Dinamica (Hardware-based)**

Para entender mas sobre la traduccion de direcciones basada en el hardware, 
primero discutiremos su primera encarnacion. Introducida en las primeras maquinas de tiempo compartida a finales de los 50's es una idea simple a la cual nos referimos como **base y limites**; aunque tambien se la conoce como **reubicacion dinamica**; usaremos ambos terminos de forma indiscriminada.

Especificamente, necesitaremos dos registros de hardware en cada CPU: uno es llamada resgistro **base**, y el otro **limite**. Este par base-limite nos permitira ubicar el espacio de direcciones en cualquier lugar que querramos de la memoria fisica, y asi asegurarnos que el proceso solo puede accedera su espacio de direcciones.

En esta configuracion, cada programa es escrito y compilado como si hubiera sido cargado en la direccion creo. Sin embargo, cuando un programa comienza a ejecutarse, el OS decide donde ubicarlo en la memoria fisica y configura el registro base con ese valor. En el ejemplo de arriba el OS decidio cargar el proceso en la direccion fisica 32KB y por lo tanto configura el el registro base con ese valor.

Cosas interesantes comienzan a suceder cuando el proceso se ejecuta. 
Ahora, cuando cualquier referencia a memoria es generada por el proceso, es **traducida** por el procesador de la siguiente manera:

**physicaladdress = virtualaddress + base**

## Ejemplo de traduccion

Para entender la traduccion de direcciones via base-y-limite con mas detalle veamos un ejemplo. Imagina un proceso con un espacio de direcciones de tamaño de 4KB (si, irrealisticamente chica) ha sido cargada en la direccion fisica 16KB. Aqui estan los resultados de algunas traducciones de direcciones.

| Vitual Address |  | Physical Address |
| --- | --- | --- |
| 0 | → | 16KB |
| 1KB | → | 17KB |
| 3000 | → | 19384 |
| 4400 | → | *Fault (out of bounds)* |

Como puedes ver en el ejemplo, es facil simplemente agregar la direccion base a la direccion virtual (la cual puede ser vista correctamente como un *offset* dentro del espacio de direcciones) para obtener la direccion fisica resultante. Solo si la direccion virtual es demasiado grande o negativa resultara en una falla de resultado, causando que se lance una excepcion.

Los siguientes cuadros ilustran mucho de la interaccion hardware/OS en una linea de tiempo.

| **OS @ boot (kernel mode)** | **Hardware** |
| --- | --- |
| **initialize trap table** |  |
|  | remember addresses of... |
|  | system call handler |
|  | timer handler |
|  | illegal mem-access handler |
|  | illegal instruccion handler |
| **start interrupt timer** |  |
|  | start timer; interrupt after X ms |
| **initilize process table** |  |
| **initialize free list** |  |

| **OS @ run (kernel mode)** | **Hardware** | **Program (user mode)** |
| --- | --- | --- |
| **To start process A:** |  |  |
| allcoate entry in process table |  |  |
| alloc memory for process |  |  |
| set base/bound registers |  |  |
| **return-from-trap** (into A) |  |  |
|  | restore registers of A |  |
|  | move to **user mode** |  |
|  | jump to A's (intial) PC |  |
|  |  | **Proces A runs** |
|  |  | Fecth instruccions |
|  | translate virtual address |  |
|  | perform fecth |  |
|  |  | execute instruccion |
|  | if explicit load/store: |  |
|  | ensure address is legal |  |
|  | trnslate virtual address |  |
|  | perform loas/store |  |
|  |  | (A runs..) |
|  | **Timer interrupt** |  |
|  | move to **kernel mode** |  |
|  | jump to handler |  |
| **Handler timer** |  |  |
| decide: stop A, run B |  |  |
| call *switch()* routine |  |  |
| save regs(A) |  |  |
| to proc-struct(A) |  |  |
| (including base/bounds) |  |  |
| restore regs(B) |  |  |
| (including base/bounds) |  |  |
| **return-from-trap** (into B) |  |  |
|  | restore registers of B |  |
|  | move to **user mode** |  |
|  | jump to B's PC |  |
|  |  | **Process B runs** |
|  |  | execute bad load |
|  | load is oput-of-bounds; |  |
|  | move to **kernel mode** |  |
|  | jump to trap handler |  |
| **Handle the trap** |  |  |
| decide to kill proces B |  |  |
| deallocate B's memory |  |  |
| free B's entry |  |  |
| in process table |  |  |

La primera tabla muestra que hace al OS al iniciarse para preparas la maquina para su uso, y la segunda muestra que sucede cuando un proceso (A) empieza a ejecutarse; notar como su traduccion de memoria es manejada por el hardware sin intervencion del OS. En algun punto, una interrucion ocurre, y el OS cambia al proceso B, el cual ejecuta una mala carga; en este punto, el OS debe intervenir, terminando el proceso y limpiandolo liberando la memoria de B y remiviendo su entrada de la tabla de procesos. Como puedes ver en los cuadros, todavia estamos siguiendo el enfoque de ejecucion directa limitada. En muchos casos, el OS solo configura el hardware apropiadamente y deja a los procesos ejecutarse directamente en la CPU; solo cuando el proceso se comporta mal hace que el OS se involucre