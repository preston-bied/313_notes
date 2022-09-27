# Process API: Part 2

## Process Synchronization

The `waitpid` function makes one process wait for another one.

`pid_t waitpid(pid_t pid, int *status, int options)`

The three arguments are:
- The *target* process's ID
- The address of an integer where termination information (i.e., exit code) can be placed. You can pass `NULL` if we don't care for that.
- A collection of bitwise-or'ed flags. Use 0 to block until the target's termination.

A simpler way to wait for any child process is to use `wait`.

`pid_t wait(int *status)`

- This function waits until any of the child processes finish
- Excellent when there are many children and you want to wait for them in the order they finish

### Example

```c++
int value = 5;
int pid = fork();
if (!pid) {
    waitpid(getppid(), 0, 0) // wait for the parent
    value += 5;
    cout << "Child has value=" << value << endl;
} else {
    value += 10;
    cout << "Parent has value=" << value << endl;
}
```
```
Parent has value=15
Child has value=10
```

```c++
int status;
int value = 5;
int pid = fork();
if (!pid) {
    value += 5;
    cout << "Child has value=" << value << endl;
    exit(100); // trying to tell something to the parent
} else {
    waitpid(pid, &status, 0); // wait for the child
    value += 10;
    cout << "Child terminated, status=" << WEXITSTATUS(status) << endl;
    cout << "Parent has value=" << value << endl;
}
```
```
Child has value=10
Child terminated, status=100
Parent has value=15
```

## Zombie Processes

1. After a `fork()`, the child process executes independently from the parent.
2. If the parent does not wait for the child to terminate and executes subsequent tasks, then, at the termination of the child, the exit status is not captured by the parent.
3. A *PCB (Process Control Block)* entry remains the process table even after the termination of the child.
4. The orphaned child becomes a *zombie process*.
5. Multiple zombies can result in memory leaks because the PCBs take memory and are *not deallocated*.

### Example 1

Multiple zombie processes in OS are harmful.

The OS has one process table of finite size, so lots of zombie processes will result in a full process table.

Example: the fork bomb

A program creates infinite zombie processes because the parent does not wait for the child processes.

```c++
#include <unistd.h>

int main() {
    while (1) {
        fork();
    }
    return 0;
}
```

### Example 2

```c++
int main() {
    int i;
    int pid = fork();
    if (pid == 0) {
        for (i = 0; i < 5; i++) {
            printf("I am Child\n");
        }
    } else {
        printf("I am parent\n");
        while(1);
    }
}
```
```
I am parent
I am Child
I am Child
I am Child
I am Child
I am Child
```

## Zombie Prevention Methods

1. Using `wait()` system call
2. Ignoring the `SIGCHLD` signal
3. Using a signal handler

### Zombie Precention Using `wait()`

```c++
int i;
int pid = fork();
if (pid == 0) {
    for (i = 0; i < 5; i++) {
        printf("I am Child\n");
    }
} else {
    wait(NULL);
    printf("I am parent\n");
    while(1);
}
```
```
I am Child
I am Child
I am Child
I am Child
I am Child
I am Parent
```

- When the parent process calls `wait()`, after the creation of a child, it will wait for the child to complete and it will reap the exit status of the child.
- The parent process is suspended (waits in a waiting queue) until the child is terminated.
- During the wait period, the parent process *does nothing else*.

### Zombie Prevention Using `SIGCHLD` and `SIG_IGN`

```c++
int main() {
    int i;
    int pid = fork();
    if (pid == 0) {
        for (i = 0; i < 5; i++) {
            printf("I am Child\n");
        }
    } else {
        signal(SIGCHLD, SIG_IGN);
        printf("I am Parent\n");
        while(1);
    }
}
```
```
I am Child
I am Child
I am Child
I am Child
I am Child
I am Parent
```

- When a child is terminated, a corresponding `SIGCHLD` signal is delivered to the parent.
- When calling the `signal(SIGCHLD, SIG_IGN)`, the `SIGCHLD` signal is ignored by the system.
- The child process entry is deleted from the process table and no zombie is created.

### Zombie Prevention Using `SIGCHLD`

```c++
void func(int signum) {
    wait(NULL);
}

int main() {
    int i;
    int pid = fork();
    if (pid == 0) {
        for (i = 0; i < 5; i++) {
            printf("I am Child\n");
        }
    } else {
        signal(SIGCHLD, func);
        printf("I am Parent\n");
        while(1);
    }
}
```
```
I am Child
I am Child
I am Child
I am Child
I am Child
I am Parent
```

- The parent process *installs a signal handler* for the `SIGCHLD` signal. The signal handler calls the `wait()` system call within it.
- When the child terminates, the `SIGCHLD` is delivered to the parent.
- On receipt of `SIGCHLD`, the corresponding handler is activated, which in turn calls the `wait()` system call.
- The parent collects the exit status almost immediately and the child entry in the process table is cleared.