# Concurrencia

**Ejercicio 1.** Dados estos 3 procesos en paralelo
```c
Pre: x = 0

P0: a0 = x;
	a0 = a0+1;
	x = a0;

P1: x = x+1;
	x = x+1;

P2: a2 = x;
	a2 = a2+1;
	x = a2;
```

**(a)** ¿Que valores finales puede tomar x?
x puede valer 1, 2, 3, 4

**(b)** Muestre para cada uno de los valores un escenario de ejecución que los produzca
Para x = 1:
- Se ejecuta primero a2 = x, y los dos pasos finales a2++ y x = a2
Para x = 2:
- Se ejecuta se puede ejecutar primero a2 = x, y los dos pasos finales sean x = a2 y x++
Para x = 3:
- Se ejecuta se puede ejecutar primero a2 = x, y los tres pasos finales sean x = a2, x++, x++
Para x = 4:
- p0, p1 y p2 en orden

(c) ¿Cuantos escenarios de ejecución hay? ¿Cuántos para cada valor final de x?
La fórmula general para procesos con $N$ instrucciones es: 
	**Total = $(N_0 + N_1 + N_2)! / (N_0! \times N_1! \times N_2!)$**

**Ejercicio 2.** Dados estos dos procesos en paralelo
```c
Pre: x = 0
P0: while(1) {
	x++;
	x--;
}


P1: while(1) {
	x++;
	x--;
}
```
**(a)** ¿El mulitprograma termina?


**(b)** ¿Que valores puede tomar x?


**Ejercicio 3.** Considere los procesos
```c
Pre: cont && x = 1 && y = 2

P0: while (cont && x<20){
	x = x*y;
}

P1: y = y + 2;
	cont = false;

```
**(a)** Calcule los posibles valores finales de x e y.


**(b)** Si en P1 se cambia la instrucción y = y + 2; por y = y + 1; y = y + 1; en dos líneas distintas.
¿Cambia esto los posibles valores finales? Justifique.

**Ejercicio 4.** Considere los procesos 
```c
Pre: n = 0 && m = 0

P0: while (n<100){
	n = n*2;
	m = n;
}

P1: while (n<100){
	n++;
	m=n;
}

```
**(a)** ¿A cuanto pueden diferir como ma# ximo m y n durante la ejecucion?
**(b)** ¿En cuantas iteraciones termina? Indicar mınimo y maximo.
**(c)** ¿Que valores pueden tomar n y m en la Post? Justifique de manera rigurosa.

# Locks

**Ejercicio 5.** La modificación en el punto (b) del Ejercicio 3 introduce cambios en los posibles valores finales, utilice locks para que vuelvan a devolver los mismos valores del punto (a).


**Ejercicio 6.** Dar una secuencia de ejecución de las sentencias de dos procesos P0 y P1 que corren el código de simple flag donde ambos entran en la region crítica
```c
typedef struct __lock_t { int flag; } lock_t;
void init(lock_t *mutex){
	mutex->flag = 0;
}

void lock(lock_t *mutex){
	while(mutex->flag == 1);
	mutex->flag = 1;
}

void unlock(lock_t *mutex){
	mutex->flag = 0;
}

```

**Ejercicio 8.** El siguiente programa asegura exlusi´on mutua en las regiones cr´ıticas:
```c
Pre: t = 0 ∧ ¬c0 ∧ ¬c1
P0:
while (1) {
	//Region no crıtica
	(c0,t) = (true,1)
	while (t!=0 && c1);
	// Region crıtica
	c0 = false
}

P1:
while (1) {
	//Region no crítica}
	(c1,t) = (true,0)
	while (t!=1 && c0);
	// Region critica}
	c1 = false
}
```
Las sentencias 3 y C son asignaciones multiples que se realizan de manera atomica. Por ejemplo,
para el caso de la sentencia 3, las asignaciones c0 = true y t = 1 se realizarıan en un solo paso de
ejecución.

Este protocolo es demasiado exigente en el sentido de que requiere la ejecucion de multiples asignaciones en un solo paso de ejecucion . 
Analice cual de las 4 posibles realizaciones de este protocolo de exclusión mutua —en el cual las asignaciones ya no son atomicas y por lo tanto hay que darle un orden determinado— es correcta.

# Variables de condicion

**Ejercicio 9.** Para el siguiente codigo que intenta implementar productor/consumidor, buscar una
falla instanciando dos consumidores y un productor C1, C2, P1. Dar una secuencia de lıneas que provoca una condicion no-deseada.
```c
int loops;
cond_t empty, fill;
mutex_t mutex;

void *producer(void *arg) {
	int i;
	for (i = 0; i < loops; i++) {
		pthread_mutex_lock(&mutex);
		if (count == 1)
			pthread_cond_wait(&empty, &mutex);
		put(i);
		pthread_cond_signal(&fill);
		pthread_mutex_unlock(&mutex);
	}
}

void *consumer(void *arg) {
	int i;
	for (i = 0; i < loops; i++) {
		pthread_mutex_lock(&mutex);
		if (count == 0)
			pthread_cond_wait(&fill, &mutex);
		int tmp = get();
		pthread_cond_signal(&empty);
		pthread_mutex_unlock(&mutex);
		printf("%d\n", tmp);
	}
}
```

# Semáforos
**Ejercicio 11.** Agregar semáforos para sincronizar multiprogramas anteriores.
(a) Modifique el programa del Ejercicio 1 agregando semáforos para que el resultado del multiprograma sea determinista (es decir, que no dependa del planificador) y devuelva el mínimo valor posible.

(b) Sincronice los procesos del Ejercicio 4 con sem´aforos de manera que se alternen entre P0 y P1 en cada iteración hasta el final de sus ejecuciones. ¿Que valores toman n y m al finalizar?

**Ejercicio 12.** Explicar que hace este programa para cada una de las siguientes combinaciones de
valores iniciales de los semaforos: (E, F ) = {(0, 0), (0, 1), (1, 0), (1, 1)}.
```c
sem_t empty;
sem_t full;

void *ping(void *arg) {
	int i;
	for (i = 0; i < loops; i++) {
	sem_wait(&empty);
		printf("Ping\n");
		sem_post(&full);
	}
}

void *pong(void *arg) {
	int i;
	for (i = 0; i < loops; i++) {
	sem_wait(&full);
		printf("Pong\n");
		sem_post(&empty);
	}
}

int main(int argc, char *argv[]) {
	// ...
	sem_init(&empty, 0, E);
	sem_init(&full, 0, F);
	// ...
}
```

**Ejercicio 13.** ¿Que primitiva de sincronizan implementa el código de abajo con una variable de condición y un mutex?
```c
typedef struct __unknown_t {
	int a;
	pthread_cond_t b;
	pthread_mutex_t c;
} unknown_t;

void unknown_0(unknown_t *u, int a) {
	u->a = a;
	cond_init(&u->b);
	mutex_init(&u->c);
}

void unknown_A(unknown_t *u) {
	mutex_lock(&u->c);
	u->a++;
	cond_signal(&u->b);
	mutex_unlock(&u->c);
}

void unknown_B(unknown_t *u) {
	mutex_lock(&u->c);
	while (u->a <= 0)
		cond_wait(&u->b, &u->c);
	u->a--;
	mutex_unlock(&u->c);
}

```

# Deadlock
**Ejercicio 14.** Considere los siguientes tres procesos que se ejecutan concurrentemente:
```c
P0: lock(printer);
	lock(disk);
	unlock(disk);
	unlock(printer)
P1: lock(printer);
	unlock(printer);
	lock(cd);
	lock(disk);
	unlock(disk);
	unlock(cd)

P2: lock(cd);
unlock(cd);
lock(printer);
lock(disk);
lock(cd);
unlock(cd);
unlock(disk);
unlock(printer)
```
(a) De la planificacion que lleva a un estado de deadlock.
(b) Agregue semáforos de manera de evitar que los procesos entren en deadlock. Trate de maximizar la concurrencia.
(c) Como solucion alternativa, modifique minimamente el orden de los pedidos y liberaciones para
que no haya riesgo de deadlock.

**Ejercicio 15.** Asuma un sistema operativo donde periodicamente se mata alg´un proceso al azar.¿Puede haber deadlock en este contexto?
