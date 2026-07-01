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
**(a)** ¿El multiprograma termina?
No, no termina ya que while(1) es siemore true y siempre se ejecuta

**(b)** ¿Que valores puede tomar x?
La variable `x` puede tomar **cualquier valor entero** (es decir: ..., -3, -2, -1, 0, 1, 2, 3, ... hacia el infinito o menos infinito).

**Justificación:**
1. **Falta de atomicidad:** Las sentencias `x++` y `x--` no son atómicas. A nivel de lenguaje ensamblador, cada una requiere tres pasos independientes: leer el valor de la memoria a un registro de la CPU, modificar el registro (sumar o restar 1), y escribir el nuevo valor en la memoria.
2. **Condiciones de carrera (Race conditions):** El planificador (scheduler) del Sistema Operativo puede interrumpir (hacer un cambio de contexto) a cualquiera de los procesos en medio de estos tres pasos. Esto provoca que un proceso pueda sobrescribir la memoria con un cálculo desactualizado, haciendo que se "pierda" el efecto del incremento o del decremento del otro proceso (problema conocido como _Lost Update_).
3. **Deriva no acotada (Unbounded Drift):** Como estas operaciones están dentro de un bucle infinito `while(1)`, los procesos ejecutan iteraciones constantemente. El estado corrupto de `x` al finalizar una iteración se convierte en el estado inicial de la siguiente. De esta forma, las pérdidas de actualizaciones (incrementos o decrementos perdidos) se **acumulan** en el tiempo sin ningún límite. Si sistemáticamente se pierden decrementos, `x` tomará valores positivos cada vez más altos; si se pierden incrementos, tomará valores negativos cada vez más bajos. Por lo tanto, el rango de valores posibles no está acotado.

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
- Si cont = false se ejecuta primero entonces y = 4 y x = 1
- Si cont = false se eejcuta despues, x puede tomar valores x={1,2,4,8,16}, si y = 4, solo toma x={1,4,16}. y siempre valdra 4 al final

**(b)** Si en P1 se cambia la instrucción y = y + 2; por y = y + 1; y = y + 1; en dos líneas distintas.
¿Cambia esto los posibles valores finales? Justifique.

Si cambia, ya que puede hacer que x tome valores potencias de 2, 3 y 4 


**Ejercicio 4.** Considere los procesos 
```c
Pre: n = 0 && m = 0

P0: while (n<100){
	n = n*2;
	m = n;
}

P1: while (n<100){
	n++;
	m = n;
}
```
**(a)** ¿A cuanto pueden diferir como máximo m y n durante la ejecución?
Pueden diferir máximo en 100, ya que si n++; se ejecuta 100 veces y m=n nunca entonces m=0 y n=100

**(b)** ¿En cuantas iteraciones termina? Indicar mínimo y máximo.
Máximo 99 iteraciones
Mínimo 7 iteraciones

**(c)** ¿Que valores pueden tomar n y m en la Post? Justifique de manera rigurosa.
- Si el ultimo en escribir fue P1, entonces el valor de n es **100**  
- Si el ultimo en escribir fue P0, entonces n puede ser cualquier valor de la multiplicación, como máximo sera **198**
- m tomara el ultimo valor de n al finalizar la ultima iteración ejecutada
**Justificación Rigurosa:** Debido a la **no atomicidad** del bloque `while`, no hay garantía de que `m` siempre sea igual a `n` al finalizar el programa.
- Si el último proceso en terminar fue P1, es muy probable que **`m = 100`** y **`n = 100`**.
- Si el último proceso en terminar fue P0, puede haber un escenario donde `n` saltó a 198, pero `m` todavía no fue actualizado, o fue actualizado a 198.
- **Conclusión:** `n` es siempre ≥100. `m` será igual al valor de `n` al final de la última iteración completada por el proceso que "ganó" la última escritura antes de que la condición del `while` se volviera falsa para ambos.
# Locks
un **Lock** es un mecanismo que garantiza que **solo un proceso a la vez** pueda ejecutar una porción de código específica. A esa porción de código protegida la llamamos **Sección Crítica**.
Un Lock tiene dos operaciones fundamentales:

1. `acquire(L)` (o `lock`): El proceso pide pide usar la variable. Si ya hay otro proceso **se queda esperando (dormido)** hasta el que esta bloqueado se libere.
2. `release(L)` (o `unlock`): El proceso termina de usar la variable, devuelve el control


**Ejercicio 5.** La modificación en el punto (b) del Ejercicio 3 introduce cambios en los posibles valores finales, utilice locks para que vuelvan a devolver los mismos valores del punto (a).
```c
// Pre: cont && x = 1 && y = 2

lock L = LIBRE; // Creo candado

P0:
	while(cont && x < 20){
		acquire(L); // P0 pide el lock antes de leer y
		x = x * y;
		release(L); // P0 termina de usar el lock y lo suelta
	}

P1:
	acquire(L);
	y++;
	y++;
	release(L);
	cont = false;
```
Esto soluciona el problema del estado intermedio y=3 
Ambos deben usar el candado. Si se lo ponemos a uno de los dos, como los dos usan la variable, puede ser que P0 o P1 usen la variable aunque uno de los dos tenga el lock

**Ejercicio 6.** Dar una secuencia de ejecución de las sentencias de dos procesos P0 y P1 que corren el código de simple flag donde ambos entran en la región crítica
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
El problema es la falta de atomicidad en los chequeos del mutex, por lo que es posible que ambos entren en la CZ
1. P0 llama a lock(mutex)
2. P0 evalua while(mutex->flag= =1). Lee la flag=0, sale del bucle y se prepara para poner la flag en 1
3. El SO interrumpe a P0 justo antes de que se cambia el valor de la la flag, por lo que sigue valiendo 0
4. P1 llama a lock(mutex), lee flag=0 porque p0 nunca cambio, sale del bucle y ahora si flag=1.
5. P1 dentro de la region critica, pero el SO pausa a P1 y vuelve con P0
6. ejecuta la instruccion pendiente que era la de poner la flag en 1
7. P0 entra en la CZ
En el ultimo paso, ambos procesos se encuentran ejecutando codigo dentro de la CZ simultaneamente. 
Para que el lock funcione de verdad, el cambio de flag debe ser atomico. Las acciones de testear valor y setear valor deben ocurrir en un unico paso ininterrumpible

**Ejercicio 8.** El siguiente programa asegura exclusión mutua en las regiones críticas:
```c
Pre: t = 0 ∧ ¬c0 ∧ ¬c1
P0:
while (1) {
	//Region no crítica
	(c0,t) = (true,1)
	while (t!=0 && c1);
	// Region crítica
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
¡Llegamos a un punto histórico de la materia! El código que estás viendo es la base del famosísimo **Algoritmo de Peterson** para exclusión mutua por software.

Cuando rompemos la asignación múltiple `(c0,t) = (true,1)` en dos instrucciones separadas, nos quedan **4 combinaciones posibles** (2 opciones para P0 multiplicadas por 2 opciones para P1).

Vamos a ver por qué **solo una de ellas es correcta** y cómo el Sistema Operativo destruye a las otras 3 usando lo que ya aprendiste sobre cambios de contexto.

**Las 4 posibles realizaciones:**
1. **Caso 1 (El Correcto):** Ambos primero avisan su intención (`c`) y luego ceden el turno (`t`).
    - **P0:** `c0 = true; t = 1;
    - **P1:** `c1 = true; t = 0;
2. **Caso 2:** Ambos primero ceden el turno (`t`) y luego avisan su intención (`c`).
    - **P0:** `t = 1; c0 = true;`
    - **P1:** `t = 0; c1 = true;`
3. **Caso 3 (Mixto A):** P0 hace `c` luego `t`. P1 hace `t` luego `c`.
4. **Caso 4 (Mixto B):** P0 hace `t` luego `c`. P1 hace `c` luego `t`
**¿Por qué la única correcta es el Caso 1?**
**¿Por qué fallan estrepitosamente los otros 3 casos?**

Vamos a hacer la prueba de fuego con el **Caso 2** (hacer `t` y luego `c`).

Si ceden el turno _antes_ de levantar la mano, el planificador (scheduler) puede hacer que **ambos procesos entren a la Región Crítica al mismo tiempo**.

Mirá esta traza de ejecución donde el SO hace los cortes en el peor momento:

|**Paso**|**Proceso**|**Instrucción**|**Estado Global de Variables**|
|---|---|---|---|
|1|**P0**|`t = 1;` (Cede el turno a P1)|`t=1, c0=F, c1=F`|
|2|---|**¡CAMBIO DE CONTEXTO!**||
|3|**P1**|`t = 0;` (Cede el turno a P0)|`t=0, c0=F, c1=F`|
|4|**P1**|`c1 = true;` (Levanta la mano)|`t=0, c0=F, c1=T`|
|5|**P1**|Evalúa `while (t!=1 && c0)`|`c0` es Falso. La condición da Falso. **P1 ENTRA a la Región Crítica.**|
|6|---|**¡CAMBIO DE CONTEXTO!**|P1 está adentro de la Región Crítica.|
|7|**P0**|`c0 = true;` (Levanta la mano)|`t=0, c0=T, c1=T`|
|8|**P0**|Evalúa `while (t!=0 && c1)`|Como `t` es 0 (lo piso P1 en el paso 3), la expresión `t!=0` da Falso. Toda la condición da Falso.|
|9|**P0**|**P0 ENTRA a la Región Crítica.**|**¡DESASTRE! Ambos están en la RC.**|

Como podés ver, P0 leyó que era su turno (`t=0`), ¡pero ese turno se lo había dado P1 un rato antes! Al no haber levantado sus "banderas" (`c0` y `c1`) desde el principio, se engañaron mutuamente.

_(Los casos 3 y 4 fallan exactamente por la misma lógica: el proceso que hace `t` primero puede ser interrumpido, permitiendo que el otro lea variables inconsistentes)._

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
- `mutex_t mutex`: Asegura la exclusión mutua
- `cond_t empty, fill`: Variables de condición, `empty` es para los hilos que necesitan buffer vacío y `fill` para los que necesitan el buffer lleno
- `pthread_cond_wait(cond, mutex)`: Suelta el cnadado, manda a dormir al hilo a la espera de cond. Cuando alguien lo despierta, vuelve a adquirir el candado automaticamente antes de continuar a la siguiente linea
- `pthread_cond_signal(cond)`: despierta a uno de los hilos que este durmiendo 

**Anatomía del productor:**
- `pthread_mutex_lock(&mutex)`: Adquiere el candado
- `if count == 1` Pregunta si el buffer esta lleno
- `pthread_con_wait(&empty, &mutex)`:  Si estaba lleno, el productor suelta el candado para que el consumidor pueda entrar y vaciarlo
- `put(i)` Si el buffer estaba no estaba lleno o si acaba de despertar, mete el dato
- `pthread_cond_signal(&fill)`: Avisa que llena el buffer para que despertar al consumidor
- `pthread_mutex_signal(&mutex)`: Suelta el candado

**Anatomía del consumidor:**
- `pthread_mutex_lock(&mutex)`: Adquiere el candado
- `if count == 1` Pregunta si el buffer esta vacío
- `pthread_con_wait(&fill, &mutex)`:  Si estaba lleno, el consumidor suelta el candado para que el productor pueda llenarlo
- `int tmp = get()`Si habia un dato o se desperto porque acaban de poner uno, lo guarda en tmp
- `pthread_cond_signal(&empty)`: Avisa que ya vació elbuffer y despierta al productor que este durmiendo
-  `pthread_mutex_signal(&mutex)`: Suelta el candado

¿Cual es el problema? La **semánica de mesa** 
El error esta en usar un simple if en lugar de un while al evaluar la condición antes del wait

# Semáforos
Un semáforo es un numero entero y una cola de hilos dormidos. A diferencia de un Lock común, un semáforo puede tomar distintos valores iniciales dependiendo para que se quiera usar

Se interactua con ese numero mediante dos funciones atómicas e ininterrumpibles:
1. `wait()` intenta **restarle 1** al semáforo
	- Si el valor es > 0, decrementa en 1 y el hilo sigue su ejecución normal
	- Si el valor es = 0, se va a dormir y espera en la cola del semáforo
2. `post()` **suma 1** al semáforo
	- Al sumarle 1, si había algún hilo durmiendo en la cola esperando para pasar, **despierta a uno** de ellos para que retome su ejecución

**Ejercicio 11.** Agregar semáforos para sincronizar multiprogramas anteriores.
(a) Modifique el programa del Ejercicio 1 agregando semáforos para que el resultado del multiprograma sea determinista (es decir, que no dependa del planificador) y devuelva el mínimo valor posible.
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

// Debemos retornar el menor valor posible (x = 1)

pre: x = 0
sem_t s1 = 0;
sem_t s2 = 0;
sem_t s3 = 0;

// P0
a0 = x;   // P0 es el unico que no esta bloqueado. Lee x = 0
post(s1); // La da permiso a P1 para arrancar
wait(s3); // Se duerme P0
a0++;
x = a0;

// P1
wait(s1); // Esperando a que P0 lo despierte
x++;
x++;
post(s2); // Despierta a P2

// P2
wait(s2); // Esperando a que P1 lo despierte
a2 = x;
a2++;
x = a2;
post(s3); // Despierta a P0 para la ultima ejecucion
``` 

(b) Sincronice los procesos del Ejercicio 4 con semáforos de manera que se alternen entre P0 y P1 en cada iteración hasta el final de sus ejecuciones. ¿Que valores toman n y m al finalizar?
```c
Pre: n = 0 && m = 0

sem_t s1 = 1;
sem_t s2 = 0;

// P0
while (1) {
    wait(s1);
    if (n >= 100) { 
        post(s2); // Despierto a P1 por si se quedó trabado antes de irme
        break; 
    }
    n = n * 2;
    m = n;
    post(s2);
}

// P1
while (1) {
    wait(s2);
    if (n >= 100) {
        post(s1); // Despierto a P0 por si se quedó trabado antes de irme
        break;
    }
    n++;
    m = n;
    post(s1);
}

// n y m terminan valiendo 127

```

**Ejercicio 12.** Explicar que hace este programa para cada una de las siguientes combinaciones de
valores iniciales de los semáforos: (E, F ) = {(0, 0), (0, 1), (1, 0), (1, 1)}.
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
// Caso (0,0)
// arranca ping, hace wait y se bloquea
// arranca pong, hace wait y se bloquea
// Resultado: deadlock

// Caso (0,1)
// arranca ping, hace wait y se bloquea
// arranca pong, hace wait, ejecuta pong y despierta a ping
// resultado: pong, ping, pong, ping

// Caso (1,0)
// Igual que el caso 0,1, pero al reves
// Resultado: ping, pong, ping, pong

// Caso (1,1)
// Es no deterministico, se entrelazan los procesos
// Resultado: mezcla impredecible
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
