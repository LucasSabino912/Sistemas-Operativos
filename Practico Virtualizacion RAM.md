**Ejercicio 1.** Para cada una de las variables del código indicar si están en el segmento de código, stack o heap. Si hay punteros indicar a que segmento apunta. ¿Donde se ubica el arreglo global si hacemos int a[N] = 0?
```c
int a[N];

int main(int argc, char** argv){
	int i;
	register int s = 0;
	int* b = calloc(N, sizeof(int))
	
	for (i = 0; i < N; i++){
		s += a[i] + b[i]
	}
	
	free(b);
	return s;
}

a[N] -> Segmentos de datos
i    -> Registro de CPU, si falla al Stack
b    -> Stack pero apunta a un segnmento del Heap
s    -> Stack
```

**Ejercicio 2.** Debuggear el mal uso de memoria en los siguientes pedacitos de código.
```c
char *s = malloc(512);
gets(s);
// Realiza buffer overflow si entra algo mas grande que 512 bytes, ya que gets
// no sabe cuantos bytes tiene la variable s

char *s = "Hello Waldo";
char *d = malloc(strlen(s));
strcpy(d,s);
// Deberia ser char* d = malloc(strlen(s) + 1)

char *s = "Hello Waldo";
char *d = malloc(strlen(s));
d = strdup(s);
// strdup() ya llama a malloc internamente asi que esta asignando memoia 2 veces

int * a = malloc(16)
a[15] = 42;
// Un entero ocupa 4 bytes, por lo que el malloc(16) solo aloca 4 enteros,
// y a[15] esta intentando acceder a la posicion 15. Deberia haber hecho
// malloc(sizeof(int) * 16) y ahi se podria acceder

```

## Traducción de direcciones
**Ejercicio 4.** Mostrar la secuencia de accesos a la memoria física que se produce al ejecutar este
programa assembler x86_32, donde el registro base=4096 y bounds=256

```c
0:  movl $128,%ebx
5:  movl (%ebx),%eax
8:  shll $1, %ebx
10: movl (%ebx),%eax
13: retq
```
El programa cree que arranca en la dirección 0 (dirección virtual) pero arranca en la dirección de memoria física 4096

Base 4096, donde realmente arranca el programa
Bounds 256, es el tamaño máximo permitido

**Verificación de límite:**
¿Dirección virtual < 256?
	Si es menor traducimos: `Dirección física = Dirección Virtual + 4096` 
	Si es mayor, el sistema operativo **aborta** el programa

**Instrucción 0:** `movl $128,%ebx` 
Guardar el numero 128 en el registro %ebx
**Acceso 1 (Búsqueda de la instrucción):**
- La cpu necesita leer esta instrucción que esta en la posición virtual 0
- 0 < 256 =>  Dirección física  0 + 4096 = 4096
- Se accede a la Dirección Física 4096 para leer la instrucción
Ejecución: Se guarda 128 adentro del en ebx. No se accede a la memoria para leer o escribir datos

**Instrucción 5:** `movl (%ebx), %eax`
Ir a la dirección de memoria de %ebx, leer lo que hay ahí y guardarlo en %eax
**Acceso 2 (Búsqueda de la instrucción):** 
- La cpu busca la instrucción en la posición virtual 5
- 5 < 256 => Dirección física 5 + 4096 = 4101 
- Se accede a la Dirección Física 4101 para leer la instrucción
**Acceso 3 (Lectura de datos):**
- Ahora la cpu tiene que ejecutarla, busca en memoria la dirección que tiene %ebx
- %ebx vale 128, esa es la dirección de memoria virtual
- 128 < 256 => Dirección Física 128 + 4096 = 4224

**Instrucción 8:** `shll $1, %ebx`
Hace un shift a la izquierda ppor 1 bit en ebx (multiplica por 2)
**Acceso 4 (Búsqueda de la instrucción):**
- La cpu busca la instrucción en la posición virtual 8
- 8 < 256 => Dirección Física 8 + 4096 = 4104
- Se accede a la Dirección Física 4104 para leer la instrucción
- Se multiplica %ebx por 2, ahora ebx pasa a valer 256

**Instrucción 10:** movl (%ebx),%eax
Ir a la dirección de memoria de %ebx, leer lo que hay ahí y guardarlo en %eax
**Acceso 5 (Búsqueda de la instrucción):** 
- La cpu busca la instrucción en la posición virtual 10
- 10 < 256 => Dirección Física 10 + 4096 = 4106
- Se accede a la dirección física 4106
**Acceso 6 (lectura de datos):**
- Ahora la cpu la ejecuta instrucción
- %ebx vale 256, esa es la dirección de memoria virtual
- 256 < 256? NO, el so levanta una excepción **out of bounds**

**Instrucción 13:** `retq`
Como el programa expotó antes esto nunca se ejecuta

## Paginación
**Ejercicio 10.** La TLB de una computadora con una pagetable de un nivel tiene una eficiencia del 95 %. Obtener un valor de la TLB toma 10ns. La memoria principal tarda 120ns. ¿Cual es el tiempo promedio para completar una operación de memoria teniendo en cuenta que se usa tabla de páginas lineal?

**Datos:**
- **Tiempo de acceso a la TLB:** 10 ns (tiempo en buscar la traducción rapida)
- **Tiempo de acceso a la memoria principal:** 120 ns
- **Tasa de acierto:** 95%
- **Tasa de fallos:** 5%

**Dos escenarios posibles:**
- **TLB Hit:**
	- Búsqueda en la tabla 10ns
	- Encontramos la traducción ahora directo a la memoria a buscar el dato real
	- tiempo total si hay acierto: 10 + 120 = 130 ns
- **TLB Miss:** 
	- Búsqueda en la tabla 10ns
	- No esta, entonces vamos a la memoria principal a leer la page table para encontrar la traduccion: Tarda 120ns
	- Ahora que se sabe donde esta, se vuelve a ir a la memoria: 120ns
	- Tiempo total: 10 + 120 + 120 = 250ns

Tiempo Promedio = (Probabilidad de acierto x tiempo de acierto) + (Probailidad de fallo + tiempo de fallo)

Tiempo promedio = 136ns
