# Thread Synchronization With Condition Variables and Semaphores

## Condition Variables

We can use a condition variable to solve the initial and simpler Producer-Consumer problem:

```c++
bool done = false;
int data = -1;
condition_variable cv;
mutex m;

void Thread1() {
    data = longfunction();
    done = true;
    cv.notify_all();
}

void Thread2() {
    unique_lock<mutex> l(m);
    cv.wait(l, []{return done == true;});
    temp = readanduse(data);
    l.unlock();
}
```

## A Few More Points on `wait()`

First, the `cv.wait()` function wakes up the waiting threads *sequentially*, not all at once.
- This provides a critical section until `unlock()` function is called on the lock later on
- As a result, every awake thread gets to do something wuthout running into a race condition

The `wait()` function internally unlocks the lock before going to `sleep()`.
- Otherwise, even the producer cannot produce, which requires the lock itself
- However, when returning from `wait()`, the lock is grabbed again and that causes the "one-at-a-time" safety

*Spurious wake ups* are possible.
- Waiting threads may wake up even when the predicate is not true
- That's why using the "predicate" is absolutely necessary

## Semaphores

A *semaphore* is a non-negative integer variable, shared among multiple processes. They are used for access control for a common resource in a concurrent environment.

A semaphore supports the following two *atomic* operations:
- `P()`: Waits until value > 0, then *decrements it by 1*
    - Think of this as the `wait()` operation, or the `lock()` operation
- `V()`: Increments value by 1
    - May wake up a waiting `P`, if any
    - Analogous to the `signal()` function or the `unlock()` function

A semaphore is like a *key* that allows a task to carry out some operation or to access a resource.

The only operations allowed are `P()` and `V()`.

Operations must be atomic
- Two `P`'s together can't decrement a value below zero
- Similarly, thread going to sleep in `P` won't miss wakeup from `V`, even if they both happen at the same time

## Semaphore Implementation

```c++
class Semaphore {
    private:
        int value;
        mutex m;
        condition_variable cv;
    public:
        Semaphore(int _v) : value(_v) {}
        void P() {
            unique_lock<mutex> l(m);
            // wait until value > 0, so it can be decremented
            // then decrement it
            cv.wait(l, [this]{return value > 0;});
            value--;
        }
        void V() {
            unique_lock<mutex> l(m);
            value++;
            // always notify on the way out
            // notify_all() is correct, but would lead to spurious wake ups
            cv.notify_one();
        }
};
```

## Semaphore: Simpler Code

```c++
Semaphore s(0);
int data = -1;

Thread1() {
    // takes a long time
    data = longfunction();
    s.V();
}

Thread2() {
    s.P();
    temp = readanduse(data);
}
```

## Two Uses of Semaphores

**Mutual Exclusion** (initial value = 1)
- Also called "Binary Semaphore"

```c++
semaphore.P();
// critical section goes here
semaphore.V();
```

**Scheduling Constraints** (initial value = 0)
- Allow thread1 to wait for a signal from thread2, i.e., thread2 *schedules* thread1 when a given *constraint* is satisfied
- E.g., suppose you had to implement `ThreadJoin` which must wait for a thread to terminate

```c++
// initial value of semaphore = 0
ThreadJoin {
    semaphore.P();
}
ThreadFinish {
    semaphore.V();
}
```

## Producer-Consumer with a Bounded Buffer

Problem Definition
1. A producer puts data into a shared buffer
2. A consumer takes them out
3. We need synchronization to coordinate producer/consumer

We do not want producer and consumer to have to work in lockstep, so we put a fixed-size buffer between them
1. We need to synchronize access to this buffer
2. The producer needs to wait if buffer is full
3. The consumer needs to wait if buffer is empty

## Correctness Constraints

**Correctness Constraints**
1. Consumer must wait for producer to fill slots, if empty (scheduling constraint)
2. Producer must wait for consumer to make room in buffer, if all full (scheduling constraint)
3. Only one thread can manipulate buffer queue at a time (mutual exclusion using lock)

**Rate Control**
- Consumer is limited by production rate
- Producer is limited by buffer size and consequently consumption rate

## BoundedBuffer Implementation

The following is a skeleton for the `BoundedBuffer` class:

**Unsafe:** because multiple threads can push into and pop from it simultaneously, leading to a race condition

**Unbounded:** because nothing stops the buffer from growing to infinity when producer thread(s) are much faster than the consumer(s)

We need to implement thread-safety and bounds on overflow and underflow

We also want *efficient wait* until the overflow/underflow conditions are gone, so Producer/Consumer do not have to "retry"

```c++
// a safe and unbounded buffer
class BoundedBuffer {
    private:
        queue<string> q;
        int maxcap; // max capacity
        // add sync. variables here
    public:
        BoundedBuffer(int _cap) : maxcap(_cap) {}
        string pop() {
            string data = q.front();
            q.pop();
            return data;
        }
        void push(string data) {
            q.push(data);
        }
};
```

## Full Solution to Bounded Buffer
```c++
Semaphore fullSlots = 0; // initially no coke
Semaphore emptySlots = bufSize; // initially all empty
Semaphore mutex = 1; // no one using machine

Producer(item) {
    emptySlots.P(); // wait until space
    mutex.P(); // wait until machine free
    enqueue(item);
    mutex.V();
    fullSlots.V(); // notify there is more coke
}

Consumer() {
    fullSlots.P(); // check if there's a coke
    mutex.P(); // wait until machine free
    item = dequeue();
    mutex.V();
    emptySlots.V(); // tell producer need more
    return item;
}
```