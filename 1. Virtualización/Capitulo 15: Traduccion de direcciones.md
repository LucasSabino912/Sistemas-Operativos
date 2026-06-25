# Traducción de direcciones

En la **virtualización de la memoria** se obtener **control** y **eficiencia** mientras se provee la virtualización. La eficiencia es lo que dicta que se use el apoyo del hardware. Controlar
implica que el SO asegure que ninguna aplicación tenga permitido acceder a otra memoria salvo la suya, y así proteger aplicaciones unas de otras y al SO de las aplicaciones (lo que también requiere ayuda del hardware).

Algo más que necesitaremos del sistema de memoria virtual es **flexibilidad**; que los
procesos puedan usar su address space como quieran, haciendo así el sistema más sencillo.

## Virtualizar memoria eficiente y flexiblemente
La idea es realizar una **hardware-based address translation**; en cada referencia a memoria se hace una traducción de dirección por el hardware para redireccionar la referencia a memoria de la aplicación (dirección virtual) a su localización real en memoria.
El hardware no virtualiza la memoria, solo provee un mecanismo de bajo nivel para lograrlo
eficientemente. Es el SO quien se involucra y maneja la memoria, sabiendo qué localizaciones están libres y cuales en uso, y manteniendo así control sobre cómo la memoria es usada.

### Reubicación dinámica de memoria (apoyada en el hardware)
Para virtualizar la memoria, el SO debe poner a cada programa en un lugar diferente a la dirección 0.
Para hacerlo de forma transparente (sin que el proceso se de cuenta), se usan las ideas de **base and bounds** y **dynamic relocation** (reubicación dinámica):
Se utilizan 2 registros de hardware; uno llamado **base register** y el otro **bounds** (límite). La base es el desplazamiento (offset) desde la posición 0. El límite marca hasta donde se puede acceder a memoria, de manera continua desde la base.
Este par de registros nos permiten direccionar espacio en cualquier lugar de la memoria física y al
mismo tiempo asegurar que el programa solo accede a su propio address space (protección).

Como cada programa cree estar en la dirección 0 y es el SO quien al cargarlos decide dónde ponerlos en memoria física, y establece los registros base y bounds con ese valor. Cualquier referencia a memoria generada por el programa será traducida por el procesador usando:
		**physical address = virtual address + base**

Cada referencia a memoria creada por el proceso es una dirección virtual, el hardware almacena los contenidos de la base a la dirección y el resultado es la **dirección física**.

El transformar una dirección virtual en una física es a lo que nos referimos con **address translation**. El **hardware** toma la dirección virtual que el proceso cree referenciar y la transforma en la memoria física donde está realmente la información. Debido a que esto ocurre durante la ejecución, y porque podemos mover el address space incluso después de que el programa
comenzó a ejecutarse, la técnica se llama dynamic relocation.

En todo este proceso el SO **verifica** que la dirección a la cual quiere acceder el proceso esté dentro de los límites de su address space con el registro bounds. En caso de que el proceso acceda algo fuera de su address space (o una dirección negativa) el CPU levanta una excepción.
### Apoyo del Hardware
El hardware debe proveer al SO distintas mecanismos para lograr la virtualización de la memoria:
- La posibilidad de soportar dos modos de ejecución (usuario y kernel)
- Proporcionar los registros Base y Bound.
- La capacidad de chequear que la memoria a la que se intenta acceder se encuentre dentro de los límites de base/bound y, en ese caso,traducir la memoria virtual/física.
- Otorgar las instrucciones privilegiadas para que el SO pueda modificar los registros Base y Bounds (igual que poder modificar los exception handlers), mientras un proceso está en ejecución.
- Generar excepciones cuando un programa trata de acceder a memoria ilegal o fuera de su address space, parando el proceso y retornando el control al SO corriendo el exception handler; lo mismo que ocurre si un proceso trata de ejecutar una instrucción privilegiada.

### Problemas del SO
Usando las herramientas proporcionadas por el hardware, el SO logra la virtualización de la memoria. Para ello, cuenta con tres responsabilidades:
1) **Administración de la memoria (heap)**: Encontrar espacio para el address space de un proceso cuando este es creado, para lo cual el SO busca el espacio en la estructura de datos (free list) y la asigna. Luego, cuando un proceso termina (por sí mismo o la fuerza por el SO) debe quitarlo del scheduler, reclamar la memoria y agregar dicho espacio a la free list.
2) **Manejo de los registros base/bounds:** debe guardar y restaurar los registros Base y Bounds en cada context switch, guardar sus valores en memoria, y al restaurarlos debe pasarle dichos valores al CPU.
3) Definición de los **exception handlers** (manejo de excepciones) en el momento de booteo, para luego ser ejecutados en caso de accesos a memoria ilegal (errores out of bounds; fuera de rango) o intentos de uso de instrucciones privilegiadas.
