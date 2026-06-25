# API de la memoria

Corriendo un programa hay dos tipos de memoria:
1) El **Stack**, donde las asignaciones y reasignaciones las manjea el compilador **implicitamente**. El compilador asigna la memoria necesaria y cuando ya no es necesaria la desasigna.
2) El heap. Para cosas que requieren más permanencia se usa el heap, donde las asignaciones y reasignaciones las realiza el usuario **explícitamente**.

## Malloc()
Se le da un tamaño pidiendo dicho espacio en memoria heap. Si lo logra, devuelve un puntero (sin tipo, para castear) a dicha memoria. Si falla, devuelve NULL.
Normalmente se utiliza sizeof() como operador para explicitar el tamaño requerido en memoria.

calloc() además inicializa la memoria asignada en cero.
realloc() copia una región de memoria y le asigna un espacio de tamaño diferente.

## Free()
Para **liberar memoria asignada** en el heap se llama a free() con el puntero devuelto por malloc() como argumento; el tamaño de la memoria a liberar es buscado automáticamente por la librería de asignación de memoria.

Tanto malloc() y free() no son system calls; son parte de la librería de manejo de memoria stdlib.
Las systemcalls son brk() y sbrk(), que mueven el break; la memoria máxima a la que puede acceder el heap, y mmap(), que permite mapear un archivo en memoria. Usualmente no se usan.

### Errores comunes

- Olvidarse de asignar memoria: **Segmentation fault**.  Muchas rutinas esperan ya tener memoria asignada al ser llamadas (por ej. strcpy()).
- No asignar memoria suficiente: **Buffer overflow**. Normalmente se asigna la memoria justa, cando no es suficiente, un overflow de bytes puede ocurrir.
- Olvidarse de inicializar la memoria asignada: **Uninitialized read**. Si se llama a malloc() pero no se le asignan valores a la memoria asignada, el programa eventualmente puede acceder a ella y leer del heap información de valor desconocido
- Olvidarse de liberar memoria: **Memory leak**. Cuando ya no se usa, la memoria debe ser liberada. 
- Liberar memoria que todavía se puede necesitar: **Dangling pointer**. Si un programa libera memoria antes de usarla se queda con un puntero colgante
- Liberar memoria repetidamente: **Double free**. Los programas pueden intentar liberar memoria más de una vez, lo cual genera comportamientos indefinidos
- Llamar a free() incorrectamente: **Free incorrectly**. free() solo espera que se le pase como argumento un puntero devuelto con malloc()
