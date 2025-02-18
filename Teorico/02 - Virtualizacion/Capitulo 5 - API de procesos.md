# Capitulo 5 - API de procesos

Los sistemas UNIX presentan una de las formas mas intrigantes de crear un nuevo proceso con un par de syscalls: ***fork() y exec().*** Una tercera rutina, ***wait()*** puede ser usada por un proceso para esperar que finalice un proceso que el mismo ha creado

# fork()

La system call `fork()` es usada para crear un nuevo proceso. 

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char *argv[]){
	printf("Hello world (pid:%d)\n", (int) getpid());
	int pid = fork();
	if(pid < 0){
		// fork failed
		fprintf(stderr, "fork failed\n");
		exit(1);
	} else if (pid == 0) {
		// child (new process)
		printf("Hello, I am child (pid:%d)\n", (int) getpid());
	} else {
		// parent
		printf("Hello, I am parent of %d (pid:%d)\n", pid, (int) getpid());
	}
	return 0;
}
```

```bash
prompt> ./p
hello world (pid:29146)
hello, I am parent of 29147 (pid:29146)
hello, I am child  (pid:29147) 
```

El proceso padre crea un proceso hijo mediante `fork()`, y crea una copia casi exacta de si mismo, la diferencia es que el proceso hijo tiene su propia memoria, espacio de direcciones y registros, y distinto PID. Para el SO significa que hay dos copias de p.c ejecutandose. El proceso padre se desprende de main, dejando que se ejecute por separado y ejecutandose a el y a su hijo por separado.

El `scheduler` del cpu sera el que determine que proceso se ejecuta primero y cuanto tiempo de ejecucion tendra.

# wait()

A veces, resulta util que el padre espere a que el proceso hijo finalice lo que estaba haciendo. Esta tarea se logra con la syscall `wait()` (o su hermana mas completa `waitpid()`)

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char *argv[]){
	printf("Hello world (pid:%d)\n", (int) getpid());
	int pid = fork();
	if(pid < 0){
		// fork failed
		fprintf(stderr, "fork failed\n");
		exit(1);
	} else if (pid == 0) {
		// child (new process)
		printf("Hello, I am child (pid:%d)\n", (int) getpid());
	} else {
		// parent
		int pidwait = wait(NULL);
		printf("Hello, I am parent of %d (pid_wait:%d) (pid:%d)\n", pid, pidwait, (int) getpid());
	}
	return 0;
}
```

```bash
prompt> ./p
hello world (pid:29146)
hello, I am parent of 29147 (pid:29146)
hello, I am child (pid_wait:29267) (pid:29147) 
```

El proceso llama a `wait()` para retrasar su ejecucion hasta que el hijo termine de ejecutarse. Cuando el hijo termina, `wait()` regresa al padre

# exec()

La syscall `exec()` en sistemas Unix/Linux se utiliza para reemplazar el proceso actual con un nuevo programa. Es parte de la familia de funciones `exec`, que incluyen `execl()`, `execv()`, `execle()`, `execve()`, `execlp()` y `execvp()`.

Cuando un proceso llama a `exec()`, **sustituye su imagen de proceso actual por la de un nuevo programa**. No crea un nuevo proceso, sino que el proceso actual deja de ejecutar su código y comienza a ejecutar el código del nuevo programa.

Cada versión de `exec()` varía en cómo maneja los argumentos y las variables de entorno:

| Función | Argumentos pasados | Usa `PATH` | Pasa entorno |
| --- | --- | --- | --- |
| `execl()` | Lista de argumentos | ❌ No | ❌ No |
| `execlp()` | Lista de argumentos | ✅ Sí | ❌ No |
| `execle()` | Lista de argumentos | ❌ No | ✅ Sí |
| `execv()` | Array de argumentos | ❌ No | ❌ No |
| `execvp()` | Array de argumentos | ✅ Sí | ❌ No |
| `execve()` | Array de argumentos + entorno | ❌ No | ✅ Sí |

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char *argv[]){
	printf("Hello world (pid:%d)\n", (int) getpid());
	int pid = fork();
	if(pid < 0){
		// fork failed
		fprintf(stderr, "fork failed\n");
		exit(1);
	} else if (pid == 0= { // child (new process) 
		printf("Hello, I am child (pid:%d)\n", (int) getpid());
		char *myargs[3];
		myargs[0] = strdup("wc"); // Program: "wc" (world count)
		myargs[1] = strdup("p.c"); // Argument: file to count
		myargs[2] = NULL; 
		// Marks end of array
		execvp(myargs[0], myargs); // runs word count
		printf("This shouldn't print out");
	} else {
		// parent
		int pidwait = wait(NULL);
		printf("Hello, I am parent of %d (pid_wait:%d) (pid:%d)\n", pid, pidwait, (int) getpid());
	}
	return 0;
}
```

En el ejemplo el proceso hijo llama a `execvp()` paraejecutar el programa `wc`, el cualñes un programa contador de palabras

**Porque? Motivando la API**

Porque creariamos una interfaz tan rara para lo que deberia ser un simple acto de creacion de un nuevo proceso? Bueno, resulta que, la separacion de `fork()` y `exec()` es esencial en la construccion de una shell UNIX, porque esto le permite al shell ejecutar codigo **despues** de la llamada `fork()` pero ***antes*** de la llamada `exec(` este codigo puede alterar el ambiente del programa que esta a punto de ejecutarse, y por lo tanto habilitar una variedad de caracteristicas 
interensantes para ser construido facilmente.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>
#include <sys/wait.h>

int main(int argc, char *argv[]) {
  int rc = fork();
  if (rc < 0) {
    // fork failed
    fprintf(stderr, "fork failed\n");
    exit(1);
  } else if (rc == 0) {
    // child: redirect standard output to a file
    close(STDOUT_FILENO);
    open("./p4.output", O_CREAT|O_WRONLY|O_TRUNC, S_IRWXU);

    // now exec "wc"...
    char *myargs[3];
    myargs[0] = strdup("wc");
    // program: wc (word count)
    myargs[1] = strdup("p4.c"); // arg: file to count
    myargs[2] = NULL;
    // mark end of array
    execvp(myargs[0], myargs); // runs word count
  } else {
    // parent goes down this path (main)
    int rc_wait = wait(NULL);
  }
  return 0;
}
```

```bash
prompt> ./p4
prompt> cat p4.output
  32  109 846 p4.c
prompt>
```