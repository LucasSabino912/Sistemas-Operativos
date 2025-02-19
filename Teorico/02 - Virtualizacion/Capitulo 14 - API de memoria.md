# Capitulo 14 - API de memoria

# Introducción

Como asignar y manejar memoria? Que interfaces son comunmente usadas? Que errores podriamos evitar?

# Tipos de memoria

## Stack

En un programa en ejecución C, hay dos tipos de memoria asignadas. La primera es llamada **stack**, y las asignaciones y desacignaciones son manejadas por el compilador para el programador, por esta razon el stack es memoria automatica

```c
void func(){
	int x; // Declares an integer on the stack
	...
}
```

El compilador hace el espacio en el stack al llamar a *func().* Cuando termina fnunc(), esa memoria asignada, se desasigna, por lo tanto, si queres quedarte con info. mas alla de la funcion, no conviene guardarla en el stack.

## Heap

La necesidad de la memoria de larga vida, nos lleva al **heap**, donde todas las asignaciones y desasignaciones son manejadas por el usuario. Una gran responsabilidad, sin dudas! Y ciertamente la causa de muchos bugs.

```c
void func(){
	int *x = (int *)malloc(sizeof(int));
}
```

Ambas asignaciones, stack y heap, ocurren en esta linea: primero el compilador sabe hacer espacio para un puntero a un entero cuando ve tu declaracion a dicho puntero *(int *x)*; subsecuentemente, cuando el programa llama a *malloc()*, pide memoria para un entero en el heap; la rutina devuelve la direccion de dicho entero (ya sea exitoso, o NULL en caso de fallar), el cual es 
guardado en el stack para usar por el programador

### La llamada malloc()

La llamada **malloc()** es simple: consultas por espacio en el heap pasando el tamaño, y en cualquier caso te devuelve un puntero, al nuevo espacio asignado, o si falla devuelve *NULL*.

```c
#include <stdlib.h>
...
void *malloc(size_t size);
```

De esta informacion, puedes ver que todo lo que necesitas hacer es incluir el archivo encabezado *stdlib.h* para usar malloc. De hecho, realmente no necesitar hacerlo, como la libreria C, a la cual todo los programas estan enlazados por defecto, tiene adentro el codigo para *malloc()*; agregar el encabezado solo le permite al compilador verificar que estes llamando a *malloc()* correctamente 

El unico parametro que toma *malloc()* es de tipo *size_t* el cual solamente describe cuantos bytes necesitas. Sin embargo, la mayoria de los programados no tipean directamente un numero; ya que es una pobre forma de hacerlo. En cambio, se usan muchas rutinas y macros para hacerlo. Por ejemplo, para asignar espacio para un valor flotante de doble precicion (*double*), siemplemente haz esto:

```c
double *d = (double *) malloc(sizeof(double));
```

Esta invoncacion a *malloc()* usa el operador *sizeof()* para solicitar la cantidad correcta de espacio; en C, esto suele pensarse como un operador *en tiempo de ejecucion*, lo que significa que el tamaño real se conoce *al momento de compilar* y por lo tanto un numero (en este caso 8, tamaño para un double) es sustituido como argumento en *malloc()*. Por eso razon, *sizeof()* es pensado como un operador y no como una llamada a una funcion (una llamada a una funcion toma lugar en tiempo de ejecucion.)

Tambien puedes pasarle el nombre de una variable (y no solo el tipo) a *sizeof()*, pero en algunos casos podrias no obtener los resultados esperados, entonces se cuidadoso. Por ejemplo, veamos el siguiente fragmento de codigo:

```c
int *x = malloc(10 * sizeof(int));
printf("%d\n", sizeof(x));
```

En la primera linea, declaramos espacio para un array de 10 integers, la cual esta elegante y fina. Sin embargo, cuando usamos *sizeof()* en la linea siguiente, devuelve un numero chico, algo como 4 (en maquinas de 32 bits) u 8 (en mauinas de 64 bits). La razon es que, en 
este caso, *sizeof()* cree que le estamos preguntando por el tamaño de un *puntero* a un entero, no sobre cuanta memoria tenemos asisgnada dinamicamente. Sin embargo, a veces, *sizeof()* funciona como se espera:

```c
int x[10];
printf("%d\n", sizeof(x));
```

En este caso, hay suficiente informacion estatica para que el compilador sepa que fueron asignados 40 bytes.

### La llamada free()

Como resultado, asignar memoria es la parte facil de la ecuacion;saber cuando, como, e inclusi si liberar memoria es la parte dificil. Para liberar memoria del heap no es largo de hacer, los programadores simplemente llaman a ***free()***:

```c
int *x = malloc(10 * sizeof(int));
...
free(x);
```

### **Errores comunes**

Hay ciertos errores comunes que surgen con el uso de *malloc()* y *free()*. Los siguientes ejemplos compilaron y el compilador no dijo ni pio.

**Olvidar asignar memoria**

Muchas rutinas esperan memoria para ser asignada antes de que las lammes. Por ejemplo, la rutina *strcpy(dst, src)* copia un string de un puntero fuente a un puntero destino. Sin emabrgo, si no tienes cuidado podrias hacer algo como esto:

```c
char *src = "Hello";
char *dst; //ops! unallocated
strcpy(dst, src); // segmentation fault and die
```

Cuando ejecutas este codigo, probablemente conducira a un **segmentation fault (falla se segmentation)**, el cual es un termino lindo para **HICISTE ALGO MAL CON LA MEMORIA PROGRAMADOR IMPRUDENTE, ESTOY ENOJADO**

En este caso, el codigo apropiado deberia ser algo como:

```c
char *src = "Hello";
char *dst = (char *) malloc(strlen(src) + 1);
strcpy(dst, src); //work properly
```

Como alternativa podrias usar *strdup()* y hacerte la vida incluso mas facil.

**No asignar suficiente memoria**

Un error relacionado es no asignar suficiente memoria, a veces llamado **buffer overflow**. En del ejemplo de arriba, un error comun es asignar *casi* el espacio suficiente para el buffer destino.

```c
char *src = "Hello";
char *dst = (char *) malloc(strlen(src)); //to small!!
strcpy(dst, src); //work properly
```

### **Olvidar inicializar la memoria asignada**

Con este error, puedes llamar a malloc de forma apropiada, pero olvidar de llenar tu tipo de dato recien asignado con un valor. Si te olvidas de hacerlo, eventualmente tu programa encontrara una **lectura sin asignar** cuando lea desde el heap algun dato de valor desconocido.

### **Olvidar liberar memoria**

Otro error comun es conocido como **memory leak**, y esto ocurre cuando olvidas liberar la memoria. En los sistemas o aplicaciones de larga ejecucion es un gran problema, y no liberar 
memoria lentamente llevara a que te queden sin memoria, en el que sera neceseario reiniciar la maquina. Por eso, cuando terminas de usar un bloque de memoria asegurate de liberarlo.

### **Liberar memoria antes de terminar de usarla**

A veces un programa leberara memoria antes de terminar de usarla; tal error se conoce como **puntero colgado (dangling pointer)**, y esto tambien es algo malo. Y su uso puede crashear el programa, o sobreescribir memoria valida (por ejemplo, llamaste a *free()* y despues a *malloc()* que reciclara la memoria recien liberada.

### **Liberar memoria repetidamente**

Algun programa tambien liberan memoria mas de una vez; y es conocido como **double free**.
El resultado de hacer esto esta indefinido. Y como pueden imaginar, la libreria de asignacion de memoria podria confundirse y hacer todo tipo de cosas raras; lo mas comun es crashear.

### **LLamar a free() de forma incorrecta**

El ultimo problema que discutiremos es llamar a *free()* de forma incorrecta. Despues de todo, *free()* espera que le pases uno de los punteros que reciviste de un *malloc()*. Cuando le pasas algun otro valor, cosas malas pueden, y van a, pasar. Por lo tanto, ese **acceso invalido (invalid acces) es peligroso y debe ser evitado**