# Concurrency And Threads

## What is concurrency?

Concurrency is the execution of a set of multiple instruction sequences at the same time. This occurs when there are several process threads running in parallel.

## What is a thread?

- A thread is a single sequential flow of control within a program
- A UNIX process can be thought of as having a single thread of control: each process is doing only one thing at a time
- With multiple threads, a program can do more than one thing at a time within a single process, with each thread handling a seperate task

### Motivation for Threads

1. Processes are expensive to create
2 It takes time to switch between processes
3. Communication between processes must be done through an external kernel structure: files, pipes, shared memory
4. Synchronization between processes is cumbersome

## Process vs. Thread

Each process has its own:
- program counter
- stack
- stack pointer
- address space

Processes may share:
- open files
- pipes

Each thread has its own:
- program counter
- stack
- stack pointer

Threads share:
- address space
    - variables
    - code
- open files

## Threads

- The communication between threads is cheap
- Threads can share variables
- Threads are "lightweight"
    - faster to create
    - faster to switch between
- Thread synchronization avoids the kernel

## Threads Applications

- **Manager/Worker:** A single manager thread assigns work to other threads, the workers. The manager handles all input and assigns the workers.
- **Pipeline:** A task is split into suboperations, which are handled in series, but concurrently, by a different thread.
- **Peer:** Similar to the manager/worker model, but after the main thread creates other threads, it participates in the work

## Thread Execution

Creating a thread is similar to calling a function directly, with a slight difference shown below:
- A regular function call is *blocking*, i.e., the caller waits for the callee
- A thread call is *non-blocking*. So, this is like a fork(); we create the thread, call the function inside, but do not wait.

## The Race Condition Problem

**Race Condition:** The output of a concurrent program depends on the *order of operations* between threads.

We cannot make any assumptions about the relative speeds of threads.

*Non-determinism* is omnipersent:
- Schedulers decision depends on many factors
- Processor architecture

## Race Condition for Out-of-Order Execution

Most architectures support this feature. Pipelined processors cannot achieve peak performance without it.

Compilers may reorder "unrelated" instructions like the following:

```c++
// global variables
bool done = false;
int data = -1;

Thread1() {
    // takes a long time
    data = longfunction();
    done = true;
}
```

```c++
// global variables
bool done = false;
int data = -1;

Thread1() {
    done = true;
    // takes a long time
    data = longfunction();
}
```

## Out-of-Order Execution Causing Race Condition

The problem arises when there is an inter-thread dependency.

Because of Thread2's wait, the 2 lines in Thread1 are no longer independent.

However, the compiler has no way to tell that.

```c++
Thread1() {
    done = true;
    data = longfunction();
}

Thread2() {
    while (!done); // wait
    int newdata = compute(data);
}
```

## pthread (POSIX Thread) Creation

With pthreads, when a program runs, it also starts out as a single process with a single thread of control.

Additional threads can be created by calling the `pthread_create` function.

```c
#include <pthread.h>

int pthread_create(pthread_t *restrict tid, const pthread_attr_t *restrict attr, void *(*func) (void *), void *restrict arg);
```

-`tid`: Uniquely identifies a thread within a process and is returned by the function
- `attr`: Sets attributes such as priority, initial stack size (can be specified as `NULL` to get defaults)
- `func`: The function to call to start the thread, accepts one `void *` argument, returns `void *`
- `arg`: The argument to `func`

`pthread_create` returns 0 if successful, a positive error code if not.

## `pthread_join`

```c
int pthread_join(pthread_t tid, void **status)
```

-`tid`: The tid of the thread to wait for, cannot wait for any thread (as in `wait()`)
- `status`: If not `NULL`, returns the `void *`, returned by the thread when it terminates

A thread can terminate by:
- returning from `func`
- the `main()` function exiting or `exit()` called
- `pthread_exit()`
- `pthread_cancel()`

## More Functions

`void pthread_exit(void *status)`
- A second way to exit, returns status explicitly
- Status must not point to an object local to the thread, as these disappear when the thread terminates

`int pthread_detach(pthread_t tid)`
- If a thread is detached its termination cannot be tracked with `pthread_join()`
- It becomes a daemon thread

`pthread_t pthread_self(void)`
- Returns the thread ID of the thread which called it
- Often see `pthread_detach(pthread_self())`

## Thread API vs Process API

| Process | Thread               | Description                            |
| ------- | -------------------- | -------------------------------------- |
| fork    | pthread_create       | create a new flow of control           |
| exit    | pthread_exit         | exit from an existing flow of control  |
| waitpid | pthread_join         | get exit status from flow of control   |
| atexit  | pthread_cleanup_push | register function to be called at exit |
| getpid  | pthread_self         | get ID for flow of control             |
| abort   | pthread_cancel       | request abnormal termination           |

## Thread API Example (C++)

```c++
#include <iostream>
#include <thread>
#include <unistd.h>

using namespace std;

void foo() {
    sleep(3);
    cout << "foo done" << endl;
}

void bar(int x) {
    sleep(1);
    cout << "bar done" << endl;
}

int main() {
    thread foothrd(foo); // calls foo() in a thread
    thread barthrd(bar, 0); // calls bar(x) in a thread

    cout << "main, foo, and bar would now execute concurrently..." << endl;
    // synchronize threads
    foothrd.join(); // pauses until foo finishes
    barthrd.join(); // pauses until bar finishes
    cout << "foo and bar completed" << endl;
    return 0;
}
```

## Race Condition Demonstration

Start 2 (or more) threads that increment a shared variable.

Use a pointer to the data so that the threads share it.

Use a large number for x to ensure adequate overlap.

Notice that the output is different every time.

```c++
void func(int *p, int x) {
    // increment *p x times
    for (int i = 0; i < x; i++) {
        *p = *p + 1;
    }
}

int main(int ac, char** av) {
    int data = 0;
    int times = atoi(av[1]);

    // start 2 threads to increment
    thread t1(func, &data, times);
    thread t2(func, &data, times);

    t1.join();
    t2.join();
    cout << "data=" << data << endl;
    return 0;
}
```