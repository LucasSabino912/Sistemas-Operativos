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

### 1. Las Direcciones (El origen y el destino)

- **DV (Dirección Virtual):** La dirección "de mentira" que genera tu programa. El programa cree que es el dueño de toda la memoria.
- **DF (Dirección Física):** La dirección "real" en la plaqueta de memoria RAM de tu computadora.

### 2. Los Pedacitos de la Dirección (Para ubicarnos)

- **VPN (Virtual Page Number - Número de Página Virtual):** Es la primera mitad de la DV. Te dice en qué "página" o bloque de su mundo imaginario está trabajando el programa. _(Ejemplo: "Estoy en el barrio 5")._
- **PFN (Physical Frame Number - Número de Marco Físico):** Es la primera mitad de la DF. Te dice en qué "caja" real de la RAM se guardó esa página. _(Ejemplo: "Te guardé en la caja fuerte 8")._
- **Offset (Desplazamiento):** Es la segunda mitad de la dirección (la parte que sobra). Te indica el byte exacto dentro de esa página/caja. **El offset es sagrado: nunca cambia** al pasar de virtual a físico. _(Ejemplo: "Puerta 128")._

### 3. Las Estructuras de Nivel 1 (El diccionario simple)

- **PT (Page Table - Tabla de Páginas):** Es el diccionario que usa el Sistema Operativo para traducir. Entras con un VPN y te devuelve un PFN.
- **PTE (Page Table Entry - Entrada de la Tabla):** Es **un solo renglón** dentro de esa tabla. Contiene la traducción exacta de una sola página.

### 4. Las Estructuras de Nivel 2 (Para que no explote la memoria)

Como una sola **PT** gigante ocuparía muchísima memoria RAM, la cortamos en pedacitos más chicos y creamos un "índice de índices":

- **PD (Page Directory - Directorio de Páginas):** Es la tabla maestra o índice principal. Te dice dónde están guardados los pedacitos de la Tabla de Páginas.
- **PDE (Page Directory Entry - Entrada del Directorio):** Es **un renglón** del Directorio. Te dice: _"La Tabla de Páginas (PT) que estás buscando está guardada en este Marco Físico"_.

**El viaje de la CPU en una frase:**

La CPU agarra el **VPN** -> va al Directorio (**PD**) y lee un renglón (**PDE**) para encontrar la Tabla -> va a la Tabla (**PT**) y lee un renglón (**PTE**) para encontrar el **PFN** -> le pega el **Offset** y ¡listo, llegó al dato en la memoria física!

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

Tiempo Promedio = (Probabilidad de acierto x tiempo de acierto) + (Probabilidad de fallo + tiempo de fallo)

**Tiempo promedio** = 136ns

**Ejercicio 11.** Considere el siguiente programa que ejecuta en un microprocesador con soporte de paginación, páginas de 4KiB y una TLB de 64 entradas
```c
int x[N];
int step = M;
for (int i = 0; i < N; i+=step)
	x[i] = x[i]+1;
```
Datos del problmea
- **Página = 4 KiB:** Son 4096 bytes. La memoria se divide en estos bloques
- **Tamaño de un** `int`: Un entero ocupa 4 bytes
- **El arreglo** `x[N]`: Es un bloque de gigate de memoria contigua
- **TLB = 64 entradas:** Solo puede recordar las traducciones de 64 paginas distintas al tiempo, si entra una pagina 65, tiene que borrar una de las viejas para hacerle espacio
- La TLB aprende de una sola pagina por viaje, y puede aprender hasta 64 paginas

¿Que valores de N, M hacen que la TLB falle en cada iteración del ciclo?
El ciclo avanza de a saltos de tamaño M (i+=step)
Para que la TLB falle en cada iteracion, necesitamos que cada vez que el programa intente leer x[i], ese dato este en una pagina de memoria nueva, una que la TLB no haya visto antes
	Cantidad de int x pagina = 4096 bytes / 4bytes int = 1024 enteros

Si M = 1, el codigo lee `x[0]`, `x[1]`, `x[2]`,... El primer acceso `x[0]` genera un TLB miss porque es una pagina nueva. La TLB la carga y ya los siguientes 1023 accesos van en la misma página
Para forzar un fallo siempre, M debe ser al menos 1024, asi se lee una pagina nueva por cada iteracion del for

¿Cambia en algo si el ciclo se repite muchas veces?
Como la TLB tiene una capacidad de 64 páginas, si el ciclo se repite 65 o mas veces se desborda la TLB y hace que se olvide de las primeras paginas

**Ejercicio 12.** Dado un tamaño de página de $4 \text{ KiB} = 2^{12} \text{ bytes}$ y la siguiente tabla de paginado

| **VPN** | **PFN** | **¿Válida?** |
| ------- | ------- | ------------ |
| 0       | 000     | 1            |
| 1       | 111     | 1            |
| 2       | 000     | 0            |
| 3       | 101     | 1            |
| 4       | 100     | 1            |
| 5       | 001     | 1            |
| 6       | 000     | 0            |
| 7       | 000     | 0            |
| 8       | 011     | 1            |
| 9       | 110     | 1            |
| 10      | 100     | 1            |
| 11      | 000     | 0            |
| 12      | 000     | 0            |
| 13      | 000     | 0            |
| 14      | 000     | 0            |
| 15      | 010     | 1            |

**Datos iniciales:**
- **Tamaño de página:** 4Kib = 4096 bytes = 2¹² bytes
- Esto implica que el **offset (desplazamiento)** utiliza **12 bits**
- **Page table:** tiene 16 entradas (VPN del 0 al 15) VPN => numero de pagina virtual
- **PFN máximo visible:** 111 (7 en decimal) PFN => Numero de marco fisico 

**(a)** ¿Cuántos bits de direccionamiento hay para cada espacio?
1. **Espacio de direcciones virtuales:**
	- Como la tabla tiene 16 entradas, el numero de paginas virtuales es 16 (2⁴). Por lo tanto se necesitan **4 bits para el vpn** 
2. **Espacio de direcciones físicas:**
	- El PFN mas grande es 111, lo que requiere 3 bits para el PFN

**(b)** Determine las direcciones físicas a partir de las virtuales: 39424, 12416, 26112, 63008, 21760, 32512, 43008, 36096, 7424, 4032.
**Formulas utilizadas:**
- VPN = [ DV/4096 ]
- Offset = DV mod 4096
- DF = (PFN x 4096) + Offset (solo si el bit de validez es 1)

1. **DV = 39424:** 
	- VPN = 39424/4096 = 9
	- Offset = 39424 mod 4096 = 2560
	- DF = (6 x 4096) + 2560 = 27136
2. **DV=26112:**
	- VPN = 26112/4096 = 6
	- Tabla VPN 6 -> Valida = 0
	- **Page Fault**

**(c)** Determine el mapeo inverso, o sea las direcciones virtuales a partir de las direcciones físicas: 16385, 4321.
**Formulas utilizadas:**
- PFN = [ DV/4096 ]
- Offset = DF mod 4096
- DV = (VPN x 4096) + Offset (solo si el bit de validez es 1)

1. **DF = 16385:** PFN = 16385 / 4096 = 4
	- En la tabla hay dos VPN validos que apuntan a un PFN 100, por lo tanto hay dos direcciones virtuales posibles
	- Offset = 16385 - (4 x 4096) = 1
	- Opción A: DV = (4 x 4096) + 1 = 16385
	- Opcion B: DV = 5 x 4096 + 1 = 20705

**Ejercicio 14.** Dado el sistema de paginado de dos niveles del i386 (direcciones virtuales de 32 bits, direcciones físicas de 32 bits, 10 bits de índice de _page directory_, 10 bits de índice de _table directory_, y 12 bits de offset dentro de la página, o sea un 10, 10, 12), indicar:

**El viejo esquema (Paginación de 1 nivel)**
En los primeros ejercicios la Dirección Virtual se partía solo en dos: `[ VPN ]` + `[ Offset ]`
El VPN era el único índice. Ibas a la única Tabla de Páginas, buscabas ese VPN y listo. 

**El nuevo esquema i386 (Paginación de 2 niveles)**
Como el diccionario de un solo tomo (Tabla de 1 nivel) ocupaba demasiada memoria de golpe, los ingenieros decidieron partir el diccionario en **varios tomos más pequeños**.

Ahora la Dirección Virtual se parte en **tres pedazos**: 
`[ Índice PD (10 bits) ]` + `[ Índice PT (10 bits) ]` + `[ Offset (12 bits) ]`

**Datos del problema:** ¿Cuanta memoria puede mapear una sola Page Table?
- **Offset de 12 bits:** Significa que el tamaño de la página es 2¹² = 4096 bytes = 4KiB
- **Entradas por directorio (PD) y por tabla (PT):** 2¹⁰ = 1024 entradas
- **Tamaño por cada tabla (PD o PT):** 1 página = 4KiB
- **Cobertura de 1 Page table:** 1024 x 4KiB = 4MiB
**(a)** Tamaño total ocupado por el directorio y las tablas de página para mapear 32 MiB al principio de la memoria virtual.
Para mapear 32 MiB necesitamos saber cuantas Page tables hacen falta
- 32 MiB / 4 MiB (por tabla) = 8 tablas de paginas
- Ademas siempre necesitamos obligatoriamente 1 **directorio de páginas (PD)** 
- **Total de estructuras en memoria:** 1PD + 8PT = 9 páginas
- **Tamaño total:** 9 páginas x 4KiB = 36 KiB

**(b)** Tamaño total del directorio y tablas de páginas si están mapeados los 4 GiB de memoria.
Si queremos mapear absolutamente toda la memoria (4GiB), necesitamos usar todas las entradas del directorio
- 4 GiB = 4096MiB
- 4096 MiB/4MiB por tabla = 1024 tablas de páginas
- Total de paginas: 1024 + 1 = 1025 estructuras
- Tamaño total: 1025 x 4KiB = 4100KiB

**(d)** Mostrar el directorio y las tablas de página para el siguiente mapeo de virtual a física:
**Mapeo 1:** Virtual: `[0 MiB, 4 MiB)` Física: `[0 MiB, 4 MiB)`
Este rango virtual cae exactamente en los primeros  4 MiB. corresponde a la entrada 0 del directorio (PDE 0)
- Necesita mapear 4MiB enteros (1024 páginas)
- PFN inicial físico: 0MiB = PFN 0

**Mapeo 2:**  Virtual: `[8 MiB, 8 MiB + 32 KiB)` Física: `[128 MiB, 128 MiB + 32 KiB)`
¿Que entrada de directorio cubre a partir de 8MiB?
- PDE 0: 0 a 4MiB, PDE 1: 4 a 8MiB, PDE 2: 8 a 12MiB -> Válida
- Tamaño a mapear: 32 KiB => 32KiB/ 4KiB = 8 paginas. Por lo tanto, solo se usan las primeras 8 entradas de esta tabla (PTE 0 a 7)
- PFN Inicial físico: 128 MiB. Para convertirlo a número de marco (PFN): 128MiB / 4MiB = 131072 KiB / 4 KiB = 32768


**Ejercicio 18.** Se define un page directory donde la ´ultima entrada, la 1023, apunta a la base del mismo.
**(a)** ¿A donde apunta la direccion virtual 0xFFC00000?
Pasamos a binario
1111 1111 11 00 0000 0000 0000 0000 0000 0000
- Índice PD (10 bits) = 1111 1111 11 = 1023
- Indice PT (10 bits) = 00 0000 0000 = 0
- Offset (12 bits) = 0000 0000 0000 = 0
 
 **¿Qué hace la CPU?**
1. Mira el Índice PD (1023). Va a la entrada 1023 del Directorio. Esa entrada apunta al Directorio mismo
2. Ahora usa el Índice PT (0). Como está parada en el Directorio, lee la **entrada 0 del Directorio (PDE 0)** creyendo que es una PTE. El PDE 0 apunta a la **Tabla de Páginas 0**. La CPU cree que esa tabla es el marco de datos final.
3. El offset es 0, así que lee el primer byte de ese lugar. **Conclusión:** Apunta exactamente a la base (el primer byte) de la **Tabla de Páginas 0 (PT 0)**

(b) ¿Y la direccion virtual 0xFFFE0000?
Pasamos a binario
1111 1111 11 11 1110 0000 0000 0000 0000 0000
- PD = 1023
- PT = 11 1110 0000 = 992
- Offset = 0000 0000 0000 = 0

1. Índice PD la manda de nuevo al directorio
2. Índice PT hace que lea la entrada 992 del directorio (PDE 992). El PDE 992 apunta a la tabla de paginas 992
3. Offset 0
Apunta a la base de la tabla de paginas 992

(c) Indique a donde apunta la direccion virtual 0xFFFFF000.
Pasamos a binario
1111 1111 11 | 11 1111 1111 | 0000 0000 0000
- PD = 1023
- PT = 1023
- Offset = 0

1. Índice PD (1023) la manda al Directorio.
2. Índice PT (1023) hace que lea la entrada 1023 del Directorio... ¡Y la entrada 1023 apunta de nuevo al Directorio!    
3. Offset 0. Conclusión: Al hacer doble recursión, esta dirección apunta a la base del mismísimo **Page Directory (Directorio de Páginas)**.

(d) Finalmente, describa para que sirve este esquema de memoria virtual.

Este esquema se denomina **Paginación Recursiva (Recursive Paging o Fractal Mapping)**. Sirve para que el Sistema Operativo pueda acceder y modificar dinámicamente sus propias estructuras de paginación (el Directorio y las Tablas de Páginas) **sin tener que crear mapeos adicionales**.

Al sacrificar solo 1 entrada del Directorio (la 1023), el Sistema Operativo reserva los últimos 4 MiB del espacio de direcciones virtuales (`0xFFC00000` a `0xFFFFFFFF`) como una "ventana" directa hacia la memoria física donde residen las tablas. De este modo, si el Kernel necesita alterar los permisos de memoria o mapear nuevas páginas para un proceso, simplemente escribe en esas direcciones virtuales precalculadas.

