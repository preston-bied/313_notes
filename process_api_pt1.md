# Process API: Part 1

## Process API

**Common process operations provisioned in operating systems:**
- **Create**: When we type a command into the shell, or double-click on an application icon, the OS is invoked to create a new process to run the program you have indicated
- **Destroy:** Most programs terminate once they complete, but provision for terminating a runaway process is provided
- **Reaping:** Best example is a shell program within which another program(s) can be invoked
- **Miscallaneous Control:** Other operations besides the ones above (e.g. suspend, resume, etc.)
- **Status:** Check on status of process (e.g. runtime, memory, priority, state, etc.)

A process can run more than one program. The currently running program can ask that the OS loads a different program into the same process.

The new program inherits some reused process state, such as current directories, file handles, privleges, etc.

This is done at the system level, with only four syscalls:
1. `fork()`
2. `exec()`
3. `wait()`
4. `exit()`

## What happens in exec()?

The `exec` system call clears out the binary of the current program from the current process and then in the now empty process puts the code of the program named in the exec call and then runss the new program

`execvp` does not return if it succeeds

## Exec Family of Functions

The exec functions **execute a file**. They replace the current process image with a new process image. Even though they are similar, there are differences between them, and each one of them receives different information as arguments.

The first three are of the form `execl` and accept a variable number of arguments. To use this function, we must load the `<stdarg.h>` header file.

The latter three are of the form `execv` in which case the arguments are passed using an array of pointers to strings where the last entry is `NULL`.

### execl()

`execl()` receives the location of the executable file as its first argument. The next arguments will be available to the file when it's executed. The last argument must be `NULL`.

`int execl(const char *pathname, const char *arg, ..., NULL)`

```c++
#include <unistd.h>

int main(void) {
    char *file = "/usr/bin/echo";
    char *arg1 = "Hello world!";
    execl(file, file, arg1, NULL);
    return 0;
}
```

### execlp()

`execlp()` is similar to `execl()`. However, `execlp()` uses the PATH environment variable to look for the file. Therefore, the path to the executable file is not needed.

`int execlp(const char *file, const char *arg, ..., NULL)`

```c++
#include <unistd.h>

int main(void) {
    char *file = "echo";
    char *arg1 = "Hello world!";
    execlp(file, file, arg1, NULL);
    return 0;
}
```

### execle()

With `execle()`, we can pass environment variables to the function, and it will use them.

`int execle(const char *pathname, const char *arg, ..., NULL, char *const envp[])`

```c++
#include <unistd.h>

int main(void) {
    char *file = "/usr/bin/bash";
    char *arg1 = "-c"
    char *arg2 = "echo $ENV1 $ENV2!";
    char *const env[] = {"ENV1=Hello", "ENV2=World", NULL};
    execle(file, file, arg1, arg2, NULL, env);
    return 0;
}
```

### execv()

`execv()` receives a vector of arguments that will be available to the executable file. In addition, the last element of the vector must be NULL.

`int execv(const char *pathname, char *const argv[])`

```c++
#include <unistd.h>

int main(void) {
    char *file = "/usr/bin/echo";
    char *const args[] = {"usr/bin/echo", "Hello world!", NULL};
    execv(file, args);
    return 0;
}
```

### execvp()

`execvp()` looks for the program in the PATH environment variable.

`int execvp(const char *file, char *const argv[])`

```c++
#include <unistd.h>

int main(void) {
    char *file = "echo";
    char *const args[] = {"/usr/bin/echo", "Hello world!", NULL};
    execvp(file, args);
    return 0;
}
```

### execve()

We can pass environment variables to `execve()`. In addition, the arguments need to be inside a NULL-terminated vector.

`int execve(const char *pathname, char *const argv[], char *const envp[])`

```c++
#include <unistd.h>

int main(void) {
    char *file = "/usr/vin/bash";
    char *const args[] = {"/usr/bin/bash", "-c", "echo Hello $ENV!", NULL};
    char *const env[] = {"ENV=World", NULL};
    execve(file, args, env);
    return 0;
}
```

## Creating New Processes

The following program creates a new process by invoking the `fork()` system call.

It also uses system calls `getpid()` to get the calling process's ID and `getppid()` for the parent's ID.

```c++
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
    printf("Hello!! My ID is %d, my parent ID is %d.\n", getpid(), getppid());
    pid_t pid = fork();
    printf("Bye!! My ID is %d, my parent ID is %d.\n", getpid(), getppid());
    return 0;
}
```
```
Hello!! My ID is 75189, my parent ID is 74907.
Bye!! My ID is 75189, my parent ID is 74907.
Bye!! My ID is 75190, my parent ID is 75189.
```

Because of cloning, everything after the `fork()` is executed twice, once from the parent and once from the child.

The terminal (ID = 74907) is the parent during the first two prints.

Process IDs are assigned sequentially and recycled after process termination by a garbage collector. After the child is created, the schedule of the parent and children are independent (i.e., the parent or the child might be scheduled first).

## Creating and Terminating Processes

From a programmers perspective, think of a process as being in one of three states:
- **Running:** The process is executing on the CPU or waiting to be executed and will eventually be scheduled by the kernel.
- **Stopped:** The execution of the process is suspended and will not be scheduled. A process stops as a result of receiving a SIGSTOP, SIGTSTP, SIGTTIN, or SIGTTOU signal, and it remains stopped until it receives a SIGCONT signal at which point it becomes running again (a signal is a form of software interrupt).
- **Terminated:** The process is stopped permenantly. A process becomes terminated for one of three reasons:
    1. Receiving a signal whose default action is to terminate the process
    2. Returning from the main routine
    3. Calling the exit function