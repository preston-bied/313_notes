# Thread Synchronization

## Atomic Operations

An *atomic operation* is an operation that always runs to completion or not at all
- It is indivisible: it cannot be stopped in the middle and its state connot be modified by someone else in the middle
- Fundamental building block: if no atomic operations, then we have no way for threads to work together

Each instruction in the instruction set is atomic.

An instruction fully finishes before the current process/thread can be preempted/interrupted.

## What is a mutex?

- A mutex is a *lock* that we set before using a shared resource and release after using it
- When the lock is set, *no other thread* can access the locked region of code
- It is used to protect data or other resources from concurrent access
- Mutex lock will only be released by the thread who locked it

## What is a critical section?

- A critical section is a segment of a program that multiple processes can access simultaneously
- The segment needs to be executed *atomically*, such as *accessing a resource* (file, input or output port, global data, etc.)
- It contains shared variables or resources which are needed to be *synchronized* to maintain the consistency of data variables
- In concurrent programming, if one thread tries to change the value of shared data at the same time as another thread tries to read the value (i.e. *data race* across threads), the result is unpredictable

## Mutex in C++ for Thread Safety

```c++
#include <iostream>
#include <thread>
#include <mutex>

using namespace std;

void func(int *p, int m, mutex* m) {
    // increment *p x times
    // lock it before the critical section
    m->lock();
    for (int i = 0; i < x; i++) {
        *p = *p + 1;
    }
    // unlock after critical section
    m->unlock();
}

int main(int ac, char** av) {
    int data = 0;
    int times = atoi(av[1]);
    // create a mutex and share it across the threads
    mutex m;

    thread t1(func, &data, times, &m);
    thread t2(func, &data, times, &m);

    t1.join(); // pauses until first finishes
    t2.join(); // pauses until second finishes
    cout << "data = " << data << endl;
}
```

## Mutex in C++ – Finer Locking

```c++
void func(int *p, int x, mutex* m) {
    // increment *p x times
    for (int i = 0; i < x; i++) {
        m->lock();
        *p = *p + 1;
        m->unlock();
    }
}
```

- The previous approach puts the entire thread under lock.
- This effectively makes the threads completely sequential.
- No threading/interleaving happens at all
- This is "coarse-grained" locking
- You can make locking finer
- The result is correct in both cases
- Locking and unlocking usually take time

## Producer-Consumer Synchronization

What if you want Thread1 to finish completely before you run Thread2?

E.g. Thread1 produces a result that Thread2 uses.

This is called the *producer-consumer* problem.

You need another synchronization primitive called the *conditional variable*.

### Producer-Consumer: Problems

```c++
// globals accessible to both threads
queue<int> list;

void Producer() {
    int data = getdatafrmsrc();
    list.push_back(data);
    cout << "I am a producer - I put: " << data << endl;
}

void Consumer() {
    while (list.size() == 0); // wait while empty
    int data = list.front();
    list.pop();
    cout << "I am a consumer – I got: " << data << endl;
}
```

- Produce operation has a race condition accessing the queue if there are multiple producers.
    - Push-back at a queue end requires size increment and we know that a variable increment operation itself has a race condition.
- If there are multiple consumers, more than 1 thread may think that there is something in the queue even when there is only 1 item in the queue.
    - 1 thread breaks out of the while loop, but before it pops the solitary, another consumer thread breaks out finding the same item. Whichever thread pops first will work fine, the other one(s) will crash.

You might attempt to solve multiple consumer's race condition by trying to do the following:

This prevents multiple consumers from running

```c++
void Consumer() {
    mutex.lock();
    while (list.size() == 0);
    int data = list.front();
    list.pop();
    mutex.unlock();
}
```

This leads to a *deadlock*, because the consumer is spinning with the lock held.

### Producer-Consumer: Correctly

```c++
// these are globals accessible to both threads
queue<int> list;
condition_variable cv;
mutex m;

void Producer() {
    int data = getdatafrmsrc();
    m.lock();
    list.push_back(data);
    m.unlock();
    cv.notify_all();
    cout << "I am a producer - I put: " << data << endl;
}

void Consumer() {
    unique_lock<mutex> ul(m);
    cv.wait(ul, []{return list.size() > 0;});
    int data = list.front();
    list.pop();
    ul.unlock();
    cout << "I am a consumer – I got: " << data << endl;
}
```

- **Step 1:** Declare a condition variable and a mutex
- **Step 2:** The producer:
    - produces data in a way such that the consumer can notice (e.g., pushing into the vector that the consumer checks)
    - calls `notify_one()`/`notify_all()` on the condition to wake up the consumers so that they can check again
- **Step 3:** The consumer(s) do the following:
    - calls `wait()` on the condition
    - `wait()` needs a "wrapped" lock (as `unique_lock`) and a predicate function
    - the "wrapper" also locks the lock
    - consume data
    - unlock the lock