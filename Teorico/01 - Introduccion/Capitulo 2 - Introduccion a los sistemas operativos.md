# Capitulo 2 - Introduccion a los sistemas operativos

# Que pasa cuando corremos un programa?

Un programa corriendo hace una simple cosa, ejecuta instrucciones. Millones de instrucciones por segundo, el procesador **obtiene** o **recupera** (fetch) una instruccion desde la memoria, la **decodifica, la** interpreta en formato comprensible (decode), y la **ejecuta**. Una vez que termino con esta instruccion el procesador pasa a la siguiente. Estos tres pasos, describen lo basico del modelo computacional **Von Neumann**. 

```c
while(true){
	fetch();
	decode();
	execute();
}
```

Existe un cuerpo de softwares, que son responsables de hacer que los programas sean faciles de correr (hasta haciendo que corran varios al mismo tiempo), permitiendo que los programas compartan memoria, que interactuen con dispositivos, y demas cosas. Ese cuerpo es llamado **Sistema Operativo (OS)**, y esta a cargo de que el sistema opere de forma eficiente y correcta, siendo facil de usar. 

La forma principal de que el SO haga esto es mediante una tecnica llamada **virtualizacion**. El SO toma un **recurso fisico** (procesador, memoria, disco, etc) y lo transforma en una forma virtual de el mismo, la cual es mas poderosa y facil de usar. Para permitir  que los usuarios le digan al SO que hacer y usar la caracteristicas de la maquina virtual (como correr un programa, alocar memoria o acceder a un archivo), el SO provee interfaces (APIs) a las cuales se pueden llamar. Un SO tiene cientos de **system calls** que estan disponibles para las aplicaciones.

Finalmente, gracias a que la virtualizacion permite que varios programas corran al mismo tiempo, y varios programas concurrentemente ingresen sus instrucciones y datos, y programas accedan a los dispositivos, el sistema operativo tambien es conocido como un **administrador de recursos.**

Cada CPU, memoria y dicos son un **recurso** del sistema, y es el rol del SO **administrar** esos recursos.

# Virtualizacion de la CPU

El programa devuelve lo ingresado en el prompt cada 1 segundo

```c
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <sys/time.h>
#include "common.h"

int main(int argc, char *argv[]){
	
	if (argc != 2){
		fprintf(stderr, "usage: cpu <string>\n");
		exit(1);
	}
	char *str = argv[1];
	while(1){
		spin(1);
		printf("%s\n", str);
	}
	return 0;
}
```

```bash
./cpu.c A & ; ./cpu.c B & ; ./cpu.c C & ; ./cpu.c D & 
A
B
C
D
A
B
C
D
```

Al correr este programa varias veces o en varias instancias en el mismo prompt, se hace una ilusion del que peograma corrio esas veces al mismo tiempo, esto es la **virtualizacion de la CPU**.

Para correr programas y frenarlos y decirle al SO que programas correr, las APIs comunican al usuario directamente con el SO

La habilidad de correr varios programas al mismo tiempo trae varias preguntas. Por ejemplo, si dos programas quieren correr en un tiempo particular, cual deberia correr? Esta pregunta se responde con la **politica** del SO.   

# Virtualizando la memoria

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include "common.h"

int main(int argc, char *argv[]){
	int *p = malloc(sizeof(int));
	assert(p != NULL);
	printf("(%d) adress of p: %08x\n", getpid(), (unsigned) p);
	
	*p = 0;
	while(1){
		spin(1);
		*p = +p + 1
		printf("(%d) p: %d\n", getpid(), *p);
	}
	return 0;
}
/* Programa que accede a la memoria */
```

```bash
./mem
memory adress of p: 00200000
p: 1
p: 2
p: 3
p: 4
p: 5
```

El modelo de **memoria fisica** presentado en las maquinas modernas es muy simple. La memoria es un array de bytes; para **leer** la memorua, se debe especificar la **direccion** para poder acceder a los datos guardados ahi; para **cargar (load)** o **guardar (store)** la memoria, el usuario tambien debe especificar que datos se quieren escribir en la direccion dada.

Cuando un programa esta corriendo accede a la memoria todo el tiempo. El programa deja todos los datos en estructuras de datos en la memoria, y los accede en varias instrucciones

El programa aloca memoria, imprime la direccion de memorua y guarda 0 en la primer ranura de la nuevamemoria alocada, Finalmente loopea con un delay de 1 segundo invrementando el valor guardado en la direccion de memoria de p, con cada print, imprime el process ID (PID) del programa. El PID es unico de cada proceso que esta corriendo

```bash
./mem &; ./mem &
memory adress of p: 00200000
memory adress of p: 00200000
p: 1
p: 1
p: 2
p: 2
p: 3
p: 3
p: 4
p: 4
...
```

Cada programa aloca memoria en la misma posicion y actualiza el valor en la posicion 0020000 independientemente. Es como si cada programa corriera con su propia memoria, en vez de compartir la memoria fisica con otros programas

Esto es exactamente lo que pasa con **Virtualizacion de memoria**