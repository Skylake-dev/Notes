# AOS

## Multiprocessing and IPC

### Definitions

- program: sequence of instruction stored somewhere, not in execution
- application: user-oriented concept of program, describe programs with GUI
- process: instance of a program currently executing
- thread: smallest schedulable unit of execution
  - process can contain multiple threads of execution (see later)
- task: not unique definition, usually synonym of thread, but depends on contex

### Process

- instance of a program currently in execution
- has an isolated address space
- can contain one or more threads

### Threads

- smallest schedulable execution unit
- belong to the memory address space of a process
- share memory address space with other threads of the same process
  - can synchronize each other
  - can access global aread (heap, data, ..)
  - can access variables in other thread if address is known

## Communication betweeen processes

How can we have different processes communicate with each other? They have different address space.

OS offers inter process communication. There are different ways of doing . First we see how to spawn new processes and then how to communicate.

### Forking

`fork()` spawns a new child process which is a copy of itself. The child program will be exactly the same and will start from the instruction following the fork, except from the return value of the `fork()`:

- 0 in the child process
- child PID in the parent process

The two processes will have a different address space, the memory is not actually copied until one of the two processes writes on it (copy-on-write) for efficiency.

We have no control over which one of the two processes will execute first a certain part, they are concurrent.

Each process has:

- exactly one parent
- zero or more child processes
- PID: Process IDentifier
  - `pid_t`, currently 32 bit integer
  - limeted by `/proc/sys/pid_max`
- PPID: Parent PID

### Executing

Load a new program and replace the current process image with it.

It uses the `execve` syscall, C offers several interface to this with some slight variations:
`exec,execlp,execle,execv,execvp,execve`

structure is exec + options

- -l accepts list of parameters, NULL terminated
- -v accepts array of NULL terminated strings
- -p search the PATH environment variable
- -e allows to specify new environment variables

### Inter-Process Communication (IPC)

We will follow the POSIX standard (no SystemV):

- newer
- IEEE 1003.1-2017 follow posix standard
- thread-safe

We will see the following POSIX components:

- signals
- pipes (also called unnamed pipes)
- FIFO (also called named pipes)
- message queues
- shared memory
- semaphores

#### Signals

- unidirectional
- no data transfer, contain only the signal type
- asynchronous

Used to signal some event (I/O operation done, exceptions):

- SIGCHLD, child sends to parent when it terminates
- SIGILL, sent to a process by the OS if it tries to executes an illegal instruction
- many other

Most signals by default cause the termination of the process, but the behaviour can be overridden with a custom signal handler.

Sending signal

```C
#include <signal.h>
#include <sys/types.h>
int kill(pid_t pid, int sig);
// pid: pid of the receiving program
// sign: the signal to send
// returns 0 on success -1 on fail
```

Handling signal

```C
#include <signal.h>
int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
// signum: the signal to handle
// act: handler to call when receiving the signal
// oldact: saves old handler (can be NULL)
// returns 0 on success -1 on fail
```

The `act` structure contains all the info on handling the signal

```C
struct sigaction {
void (*sa_handler)(int);
void (*sa_sigaction)(int, siginfo_t *, void *);
sigset_t sa_mask;
int sa_flags;
void (*sa_restorer)(void);
};
```

- `sa_handler`: function to handle the signal (or `SIG_IGN` to ignore it)
- `sa_sigaction`: alternative handler that accepts input data
- `sa_mask`: set a mask to block certain signals (Not all signals can be blocked or ignored)*
- `sa_flags`: various options, see docs
- `sa_restore`: not in POSIX, not for user use, don't use

*masked signals are queued for later processing (SIGKILL and SIGSTOP cannot be masked)

#### Pipes

Based on the producer/consumer pattern, they are unidirecitonal queues where data is written/read FIFO.

In Linux, the OS guarantees that only one process at a time can access the pipe.

Creating a pipe

```C
#include <unistd.h>
#include <fcntl.h>

int pipe(int pipefd[2]);
int pipe2(int pipefd[2], int flags);
```

`pipefd` will be filled with two file descriptors:

- `pipefd[0]` the read end of the pipe
- `pipefd[1]` the write end of the pipe

`flags` can be various options (e.g. non blocking if full/empty)

To use a pipe one can directly do r/w operations on them with low level `read` and `write` or open them as a stream (like a file).

#### FIFO

Similar to pipes, they are based on special files in the filesystem, but no actual I/O is done, the OS passes the data. (It sorts of pretend that it is a file, usually under `/tmp/something`).

Creating a FIFO

```C
#include <sys/types.h>
#include <sys/stat.h>

int mkfifo(const char *pathname, mode_t mode);
```

#### Message queues

Suitable for multiple writers and readers, based on a priority queue. Can be accessed via special files in the `/dev/mqueue/` directory. (requires POSIX real-time extension library)

Creating a message queue

```C
#include <mqueue.h>
mqd_t mq_open(const char *name, int oflag, mode_t mode, struct mq_attr *attr);
```

- `name`: a unique name for the message queue, starting with `/`
- `oflag`: opening flag O_RDONLY, etc
- `mode`: the permission to give the file
- `attrs`: attributes (see struct)

  ```C
  struct mq_attr {
  long mq_flags; // 0 or NON_BLOCK
  long mq_maxmsg; // max nr. messages in the queue
  long mq_msgsize; // max message size in bytes
  long mq_curmsgs; // nr. messages currently in the queue
  };
  ```

A process can use the queue using these functions (args are self explainatory, priority is 0-31, same priority messages follow FIFO logic)

```C
#include <mqueue.h>
int mq_send(mqd_t mqdes, const char *msg_ptr, size_t msg_len, unsigned int msg_prio);

int mq_receive(mqd_t mqdes, char *msg_ptr, size_t msg_len, unsigned int *msg_prio);
```

#### Shared memory

Allow processes to share a memory segment, based on the *memory mapping* concept (require POSIX real-time extension library).

NOTE: shared memory access needs to be properly sinchronized, see later.

To use shared memory:

- create a shared memory area (args are the same as mqueue, returns a file descriptor)
  
  ```C
  #include <sys/mman.h>
  #include <sys/stat.h>
  #include <sys/fcntl.h>
  
  int shm_open(const char *name, int oflag, mode_t mode);
  ```

- specify the size of the special object with `fd_truncate`
- `mmap` the fd to an area of the memroy of the process

When the process is done, some cleanup is needed

```C
#include <sys/mman.h>

// delete the mapping in the process memory
int munmap(void *addr, size_t length);

// removes shared memory object created with the open
int shm_unlink(const char *name);
```

### Synchronization

The simplest mechanism is the `wait()` (and `waitpid(pid_t pid)`) primitive:

- wait:  suspends the execution until one child completes
- waitpid:  suspends execution until that specific child completes

signatures:

```C
pid_t wait(int *status);
pid_t waitpid(pid_t pid, int *status, int options);
```

status is used to collect the return value from the child process (can be NULL if i don't care)

#### zombie processes

If the parent process does't call `wait` before exiting the child processes will become zombies (they are ended but are waiting to return the exit value) and consume resources.

All orphan children are adopted by init that calls wait and frees memory and PID number.

Zombie processes are a sign of bad programming.

#### Semaphores

The approach based on `wait` is very limited. POSIX provides inter-process semaphores. The general logic is simple:

- semaphore counter == 0 ---> WAIT
- semaphore counter > 0 ---> proceed
  - if counter can only be 0 or 1 then it's a binary semaphore

There are 2 atomic functions to interact with the semaphore:

- `wait()` blocks until counter > 0, then decrements and proceed
- `post()` increment counter

Similarly to pipes, they can be named or unnamed. `pthread` library is required to use those.

```C
#include <semaphore.h>

// unnamed semaphores
// create
int sem_init(sem_t *sem, int pshared, unsigned int value);
// destroy
int sem_destroy(sem_t *sem);

// named semaphores
// create
sem_t * sem_open(const char *name, int oflags);
sem_t * sem_open(const char *name, int oflags, mode_t mode, unsigned int value);
// destroy
int sem_close(sem_t *sem);
int sem_unlink(const char *name);

// interacting with the semaphore
int sem_wait(sem_t *sem);
int sem_trywait(sem_t *sem);  // non blocking alternative
int sem_timedwait(sem_t *sem, const struct timespec *timeout);

int sem_post(sem_t *sem);
```

All functions return 0 on success and -1 on error.

#### Final notes

Choosing which IPC method do use is not trivial and can impact performance. The best choice depends on several variables:

- data size and type
- machine architecture
- system workload
- ...

See [this link](https://www.cl.cam.ac.uk/research/srg/netos/projects/ipc-bench/) for a study on IPC performance.

## Task scheduling

Scheduler: OS component responsible of establishing the order of execution of the tasks. The ordering alg. is called scheduling policy.

Idea:

- tasks are preeempted by the OS in order to dispatch another task
  - can be task triggered also but it's unusual
  - the OS also dispatches another task when the current one is blocked waiting for I/O operations
- preemption is performed via context switch: save current registers and load back the other task

Tasks can be in different states:

![task_state_diagram](assets/task_state_diagram.png)

- new: task just created
- ready: task can be dispatched at any time
- running: currently using the CPu
- blocked: waiting on an I/O request
- terminated: the task has exited

In reality there are much more states defined in the Linux kernel, but those are the major ones.

### Task model

|[task_model](assets/task_model.png)

- `a_i` arrival time, when the tasks is ready to be scheduled
- `s_i` start time, starts execution
- `W_i` wait time, time spent waiting in the queue `W_i = s_i - a_i`
- `f_i` finishing time, when the execution terminates
- `C_i` computation time (or burst time), time necessary for the processor to execute the task (without interruptions)
- `Z_i` turnaround time, total time taken from whne the task is ready to when it completes `Z_i = f_i - a_i` (NOTE: not the same as `W_i + C_i` because there can be interruptions)

Based on the operation done by the task we distinguish between:

- CPU-bound
  - spends most of the time executing stuff
  - `Z_i ~ W_i + C_i`
- I/O bound
  - spends most of the time waiting for I/O operation
  - `Z_i >> W_i + C_i`

### Platform model

A computing system is composed of:

- `m` processing elements (PE) CPU
  - to each CPU, at each time `t`, it is assigned zero or one task
- `s` additional resources R
  - each resource, at each time `t` is assigned to zero or more tasks

### Problem statement

Given

- a set of `n` tasks
- a set of `m` PE
- a set of resources `R`

Compute an optimal schedule and allocation. (NP-complete problem)

This is very difficult: what does optimal mean? There can be different objective that require different policies to be in place in order to achieve it:

- maximize processor utilization
- maximize throughput: number of tasks completing per time unit
- minimize waiting time: time spent ready in the wait queue
- ensure fairness
- minimize scheduling overhead
- minimize turnaround time
- and many more: energy, power, temps, ....

These objective are generally in contrast with each other, we need to find a good balance.

#### Starvation

Undesirable condition in which one or more task cannot execute due to lack of resources (e.g. low priority task that always get pushed back in favor of higher priority tasks)

### Scheduling algorithms classification

- preemptive vs not preemptive
  - preemptive schedulers, can interrupt tasks to allocate CPU to another task, required for responsive systems
  - not preemptive, a task when it's scheduled runs until completion. Has minimum overhead but bad for responsiveness
- static vs dynamic
  - static, scheduler decisions are based on fixed parameters known before task activation (not very realistic in general)
  - dynamic, scheduler decisions are based on parameters that change at run time and new tasks can be added
- offline vs online
  - offline, run once before activation, when the schedule is decided it doesn't change
  - online, executed at run time during task execution, new tasks can be added
- optimal vs heuristic
  - optimal can give some guarantees but usually has much higher overhead
  - heuristic has no guarantees but in practice it can be almost as good with much lower overhead

### Scheduling algorithms

Simple algorithms that target systems with only one processor.
#### First-In-First-Out (FIFO)

First-In-First-Out (FIFO) scheduler:

- tasks scheduled in order of arrival
- non preemptive

The advantage is that it is very simple to implement and doesn't need to know anything about processes but it is not good for responsive systems (long tasks monopolize the CPU, short tasks are penalized)

![FIFO_scheduler](assets/FIFO_scheduler.png)

#### Shortest Job First (SJF)

Shortest Job First (SJF) scheduler:
 
- tasks scheduled in ascending order of computation time `C_i`
- non preemptive

It is the optimal non preemptive scheduler w.r.t. minimizing waiting time, but there is the risk of starving long tasks. Moreover, we need to know the time required to complete ahead of time.

![SJF_scheduler](assets/SJF_scheduler.png)

#### Shortest Remaining Time First (SRTF)

Shortest Remaining Time First (SRTF) scheduler:

- preemptive variant of SJF
- it uses the remaining execution time instead of the total to schedule next task

It is more responsive than SJF but shares the same disadvantages

![STRF_scheduler](assets/STRF_scheduler.png)

The preemption occurs when a new tasks arrives but only if the new tasks has a shorter remaining time than the current task, otherwise the current task continues.

#### Highest Response Ratio Next (HRRN)

Highest Response Ratio Next (HRRN) scheduler:

- select task with the highest response ration, computed as
  - `RR_i = (W_i + C_i) / C_i`
- not preemptive

It prevents starvation of longer tasks w.r.t. SJF but again, we need to know `C_i` in advance.

![HRRN_scheduling](assets/HRRN_scheduling.png)

#### Round Robin (RR)

Round Robin (RR) scheduler:

- task are scheduled to run for a given amount of time `q` (quantum or time slice)
- preemptive

This approach has several advantages:

- computable maximum waiting time `(n - 1) * q`
- no need to know `C_i` in advance
- good to achieve fairness and responsiveness goals
- no starvation

However, it has a worse turnaround than SJF.

![RR_scheduler](assets/RR_scheduler.png)

Preemption occurs at the end of the time quantum that the task has allocated (or before if the tasks ends). The preempted task is put back at the end of the queue. New tasks are added to the ready queue in a FIFO fashion.

NOTE: if a task is preempted at the same time a new arrives the order of operation is

- add new tasks at the end of the queue
- preempt the current task and put at the end of queue

The value of the quantum of time is very important:

- long quantum:
  - tend to FIFO scheduler
  - favors CPU-bound tasks
  - low overhead (less context switches)
- short quantum:
  - reduce average waiting time
  - favors I/O-bound tasks
  - good for responsiveness and fair scheduling
  - higher overhead (more context switches have to occur)

In Linux the default time quantum for RR scheduler is stored in `/proc/sys/kernel/sched_rr_timeslice_ms` (default 100ms)

### Priority-based scheduling and multi-level scheduling

Can assign a priority to each task to specify its importance. Can be fixed at design time or change dynamically at run time. It is usually expressed using an integer value, the lower the value the higher the priority.

In a priority-based scheduler usually there are multiple ready queues, divided by priority. The first task to schedule is picked from the topmost non-empty queue. Tasks can be preempted if a higher priority task arrives.

![priority-based_scheduling](assets/priority-based_scheduling.png)

For each of these queues i can use a different scheduling algorithm. 

Of course this approach could cause starvation of tasks with lower priority, so we need to use correctly the scheduling algorithms at the different queues.

Example:  
use RR for each queue, but use a longer quantum for lower priority tasks to compensate for the long wait. Then we can assign lower priority to CPU-bound tasks (so they get a longer time slice) and high priority for I/O bound tasks (increase responsiveness).

How can we determine if a task is CPU-bound?

- information provided by the user
- use some feedback mechanism in the scheduler using a dynamic priority (Multi-Level Feedback Queue Scheduling)
  - new tasks are put in the highest priority
  - if the tasks uses the whole quantum, then when it's preempted move it to a lower priority queue
    - the idea is that CPU-bound tasks will use more CPU, so they will be moved to queues with longer quantums, while I/O-bound tasks, since they will block before finishing the quantum, will remain in high priority

This doesn't solve the problem of starvation. To do so we need to do time slicing between all the queues, for instance:

- i have a 100ms time window
- 80ms assigned to first queue (high priority)
- 15ms assigned to intermediate priority
- 5ms assigned to lower priority

This way i can guarantee that all queues get at least a certain percentage of CPU time.

Another option to prevent starvation is the concept of *aging*: the more a task spends time in the ready queue the more its priority is increased. This prevents it from being postponed indefinitely by the arrival of newer tasks.

### Multi-processor scheduling

This problem is way harder, the scheduler also has to choose which CPU to assign the task:

- task synchronization my occur across parallel executions
- difficult to achieve high utilization of all CPU cores
  - need to migrate tasks across cores to balance the load, but this leads to cache miss penalties (the data needs to be loaded in the cache of the new core)
- simultaneous access to shared resources (e.g. chache memory)
  - task scheduled on the same core may trash each other data from the cache (i.e. they need two resources that maps on the same cache address) slowing each other down, it would be more efficient to schedule those on different cores
  - they could still interfere on higher level caches that are shared across cores...

There could be different design choiches for the scheduler:

- single queues vs multiple queues
  - single queue:
    - all tasks wait in a global queue
    - simple design
    - good fairness
    - good for managing CPU utilization
    - issues with scalability, the scheduler runs in any core and require synchronized access to the ready queue (semaphore/mutexes)
  - multiple queues:
    - ready queue for each processor
    - more scalable
    - easier to exploit more easily data locality in caches
    - can use one global scheduler or a per-CPU scheduler
    - potentially more overhead (more data structures to manage)
    - needs load balancing --> rebalance queues if necessary to achieve good utilization and reduce waiting time (i.e. move waiting tasks to idling or less loaded cores)
      - also useful to manage thermals (spread load across cores to avoid hot spots)
      - this has also impact on power consumption and reliability
      - task migration may happen in two ways:
        - push model: a dedicated task periodically checks each queue and rebalances if necessary
        - pull model: a processor may notify an empty queue and pick tasks from other queues
- single scheduler vs multiple per-processor scheduler
- can also use a hierchical queue: global queue that dispatches tasks to the queue of each CPU:
  - better utilization and load balancing
  - good scalability
  - more complex to implement

## Concurrency

Concurrency is when a program is composed by activities where one activity can start before the previous one has finished. This is what we will see:

- thread execution model
  - need context switching to save state between threads
  - hw parallelism (multicore)
  - sw parallelism (timesharing)
- lightweight execution model
  - do not require context switches, supported by programming languages
  - coroutines
  - generators
  - event-based models
    - continuation passing (callbacks)
    - async/await constructs

Why concurrency if it is harder? Because we are reaching the limits of single core improvements

![microprocessor_evolution](assets/microprocessor_evolution.png)

### Properties and issues

We characterize concurrent programs with two properties:

- safety (correctness): never reach error states. Possible issues are
  - data race: program behaviour depends in an uncontrolled way from the memory model and the interleaving of threads. If it is not what the programmer wants it is a bug
  - atomicity violation: operation supposed to be atomic in reality they are not and can lead to problems
  ```C
    // Thread 1::
  if (thd->proc_info) {
  fputs(thd->proc_info, ...);
  // proc_info can become NULL after the check if interleaves with thread 2
  }
  ...
  // Thread 2::
  thd->proc_info = NULL;
  ```
  - order violation: assume a specific order of execution when there is no guarantee that it will be that way
- liveness (progress): eventually all activities will be able to finish. Issues that concern liveness are
  - deadlocks: no task can take action because it is waiting for another task to take action (e.g. t1 waits for t2 and t2 waits for t1)
    - mutual exclusion, only one can access a specific resource
      - preventable if there are some atomic instructions to read and act on the data (need to be supported by CPU)
    - hold-and-wait, threads hold some resource and wait for the release of another
      - preventable by using appropriate API to release a resource if the acquisition of further resources fails
      ```C
      lock(M1);  // blocks until the lock is obtained
      if(!try_lock(M2)) {  // doesn't block, if resource not available it returns
      release(M1);  // if i cannot get M2, release the first resource
      }
      else {
      update(R1);  // when i get them both, use them
      update(R2);
      release(M1,M2);  // and release those in the end
      }
      ```
    - no preemption, resources cannot be forcibly take away from a thread that is holding them
    - circular wait, at the end this is the cause of all deadlocks
      - need to ensure a certain order in the locking of the resources (or exclude unwanted orders)
  - priority inversion: this causes higher priority tasks to be delayed because they need to wait for a lower priority task to release a lock on a resource. This doesn't cause deadlocks but dealys a high priority task and can lead to missing some deadline (see [1997 pathfinder mission](https://people.cs.ksu.edu/~hatcliff/842/Docs/Course-Overview/pathfinder-robotmag.pdf)). This can be solved in different ways:
    - highest locker priority protocol: raise the priority of a task holding a lock to the highest priority task that holds that resource.
    - priority inheritance protocol: similar as before but the lower priority task gets his priority increased only when the high priority task tries to enter the critical section.
    - priority ceiling protocol: a priority ceiling for a semaphore is the highest priority among the tasks that could lock it. In this context a task is allowed to enter a critical section only if its priority is higher than all the priority ceilings of the resources currently locked by other tasks that it has to access.

### Linux user space concurrency

Based on the concept of *futex* (fast user-level lock) and have the following objectives:

- avoid unnecessary system calls (they are expensive)
- avoid unnecessary context switches
- avoid thundering herd problem (wake up multiple threads but only one can run)

![futex_vs_traditional_lock](assets/futex_vs_traditional_lock.png)

futex allow uncontended locks (i.e. only one thread is trying to use the resource) to be locked/unlocked without switching to kernel mode. If a thread is holding the lock and another thread tries to lock it (contended) it is necessary to have the kernel intervene and put the thread in a wait queue. If uncontended access is frequent this approach allows to avoid lots of system calls.

For this to work the lock needs to stay in the runtime instead of the kernel, it is usually implemented as a 32 bit integer managed with **atomic** instructions:

- 31 bits encode the number of waiters
- 1 bit flags the state (lock/unlocked), it is the MSB
  - check if set
    - not set --> set the bit
    - set --> syscall to put the thread in waiting, call `futex` with `FUTEX_WAIT` flag
  - remember that the check and set is done in an atomic operation (depends on the platform)

This functionality is implemented like

```C
void futex_based_lock(int *mutex) {
  int v;
  if (atomic_bit_test_set(mutex, 31) == 0) 
    return;
  atomic_increment(mutex);
  while (1) {
  if (atomic_bit_test_set(mutex, 31) == 0) {
    atomic_decrement(mutex);
  return;
  }
  v = *mutex;
  // technicality, if 32th bit is set the number is negative
  // i want to call the syscall only if i'm sure that the bit is set
  // v > 0 --> unlocked 
  // v < 0 --> locked
  if (v >= 0) continue;
    futex(mutex, FUTEX_WAIT, v); /* sleeps only if mutex still has v */
  }
}

void futex_based_unlock(int *mutex) {
  // unlock and if the mutex is zero return
  // the add is basically adding 1 to the 32 bit
  // if the result is zero it means that
  // - 32th bit is 0 --> unlocked
  // - other bits are 0 --> no more waiters
  // so it wakes up a thread only if there is something waiting
  if (atomic_add_zero(mutex, 0x80000000))
    return;
  futex(mutex, FUTEX_WAKE, 1); // wake up only one thread
}
```

NOTE: futexes are also used to implement condition variables and try to avoid thundering herd problem.

#### Event-based concurrency

DIfferent style of concurrent programming. Multiple activity whose progress is triggered by external events. Locks in these cases are not the optimal way to go about it. 
The classical example is some GUI-based program waiting for user interaction or internet based services.

The general idea is:
- wait for some event to occur
- check what type of event arrived and identify which activity it belongs
- do the small amount of work it requires (I/O requests, other events, ...)
- repeat

We want to avoid the cost of context switches if not necessary. To implement this we use an *event loop*: a single thread that blocks on all events and calls other activities as functions... sort of, because they need to restore the state that the activity had before. This is done using callbacks (continuation passing style)

The advantages of this approach is that:
- when a handler processes an event, it is the only activity taking place in the system (in the event loop thread)
  - no locking needed
  - no context switch needed
- network I/O via the `select` or `poll` API
- ability to run blocking I/O in a separate thread pool and register a callback when to call when the data is ready (avoid blocking entire app). `libuv` offers an interface to do so

Limits of event loops --> it runs in a single thread, it is difficult to extend to multicore  

### Linux kernel space concurrency

There are multiple sources of concurrency in the kernel:

- interrupts
- multiple processors
- kernel preemption: multiple thread in the kernel can share the same resources

#### Interrupts

An interrupts can occur asynchronously almot at any time, interrupting the code currently executing in kernel mode. If the interrupt and the interrupted task need to use the same resource then access must be regulated.

Code that is safe from concurrenct access from interrupt handler is said to be interrupt-safe (example the global variable `jiffies` that keeps track of the uptime).

#### Multiprocessors

Kernel code must be able to simultaneously run on multiple processors, therefore there is the need to regulate access to resources that are shared (true concurrency).

Code that is safe from true concurrency on symmetrical multiprocessor machines is sai to be SMP-safe.

Do all processor see the memory in the same way?

#### Kernel preemption

![preemptive_kernel](assets/preemptive_kernel.png)

Scheduling can be called during some kernel mode code execution and switch to another activity that is deemed more important. This allows for increased responsiveness. 

From 2.6, the Linux kernel became preemptable and the preemption points are:

- end of an interrupt handling, when `TIF_NEED_RESCHED` flag is set in the thread (forced context switch)
- if a task in the kernel explicitly blocks, which causes `schedule()` to be called (planned context switch)

How can we ensure that a context switch occurs only if it is safe to do so?

A variable is used, called `preempt_count`, that keeps track of the preemptions:

- set at 0 when the process enters kernel mode
- increase by 1 on lock acquisition (critical section)
- increase by 1 on interrupt

![preemption_count](assets/preemption_count.png)

As long as `preempt_count > 0` the kernel cannot switch.

NOTES:

- can compile kernel to avoid preemption
- can also do the opposite, increase the responsiveness of the kernel to be suitable for more "real-time" operations:
  - apply the `PREEMPT_RT` patch that tries to maximize the preemptability of the kernel
    - preempt critical sections in the kernel
    - manage interrupt in their own separate thread

### Kernel synchronization

#### Spinlocks

Locking operation doesn't put to sleep the activity that is trying to lock, it's a busy wait. The thread keeps "spinning" on the lock variable until it can get the lock. It requires platform support to do the atomic operation on the locking variable.

For instance in [x86](https://en.wikipedia.org/wiki/Spinlock)

```asm
; Intel syntax

locked:                      ; The lock variable. 1 = locked, 0 = unlocked.
     dd      0

spin_lock:
     mov     eax, 1          ; Set the EAX register to 1.
     xchg    eax, [locked]   ; Atomically swap the EAX register with
                             ;  the lock variable.
                             ; This will always store 1 to the lock, leaving
                             ;  the previous value in the EAX register.
     test    eax, eax        ; Test EAX with itself. Among other things, this will
                             ;  set the processor's Zero Flag if EAX is 0.
                             ; If EAX is 0, then the lock was unlocked and
                             ;  we just locked it.
                             ; Otherwise, EAX is 1 and we didn't acquire the lock.
     jnz     spin_lock       ; Jump back to the MOV instruction if the Zero Flag is
                             ;  not set; the lock was previously locked, and so
                             ; we need to spin until it becomes unlocked.
     ret                     ; The lock has been acquired, return to the calling
                             ;  function.

spin_unlock:
     xor     eax, eax        ; Set the EAX register to 0.
     xchg    eax, [locked]   ; Atomically swap the EAX register with
                             ;  the lock variable.
     ret                     ; The lock has been released.
```

Of course if the wait is long the spinlock becomes a wasteful way of locking since the thread keeps the CPU without doing anything except waiting. On uniprocessor machines this is just a call to `preempt_disable`, only on multiprocessor system the thread actually spins.

Variants:

- Readwrite locks

  Distinguish between readers and writers, multiple readers can access the object but only a writer at a time is allowed. 
  This is used to maximize concurrent access when there is no risk of concurrent modification.

- Seqlocks

  Similar to readwrite locks but want to prevent the starvation of writers. The idea is

  - when writer acquire lock it increments a counter (starting from 0)
  - when it releases it, the counter is incremented again

  This means that if the counter is even there are no writes going on, while if it is odd a write is currently holding the lock.

  Readers check the counter when trying to lock:
  - odd: busy wait
  - even: return counter
    - do work but before releasing check if the counter changed, if it did redo the work
  
  ```C
  // what the write does
  write_seqlock(&mr_seq_lock); // increment seq. counter
  /* write lock is obtained... */
  write_sequnlock(&mr_seq_lock); // increment seq. counter
  
  // what the read does
  do {
  // loops if seq. counter odd
  seq = read_seqbegin(&mr_seq_lock);          // ^
  /* read/copy data here ...              | check if seq. counter equal. */
  } while (read_seqretry(&mr_seq_lock, seq)); // V
  ```

  Example, the `jiffies` variable is frequently read by interrupt handler. For machines that do not have atomic 64 bit reads, it is read using a seqlock.

#### Sleeping locks

Tasks trying to lock and already locked resource are put to sleep. Implemented by a semaphore. Better choice if the time that we have to wait is unknown or expected to be long.

MEMO: semaphore, basically a counter, lock decrements, unlock increments, blocked when trying to lock a 0 semaphore.

```C
/* define and declare a semaphore, named mr_sem, with a count of one */
static DECLARE_MUTEX(mr_sem);
/* attempt to acquire the semaphore (can also specify interruptible) ... */
if (down_interruptible(&mr_sem)) {
/* signal received, semaphore not acquired ... */
}
/* critical region ... */
/* release the given semaphore */
up(&mr_sem);
```

#### Read copy update locks (RCU)

The goal is to have low latency reads to shared data that is read often but updated infrequently. The idea works as follow and it's an improvement over seqlocks:

- readers
  - avoid locks
  - should tolerate concurrent writers
  - might see old version of the data for a short time
- writers
  - create a new copy of the data structure
  - publish new version with a single atomic operation

Basically the idea is that writers update the data "offline" and then commit the changes atomically when they are done. The reader can read old data while the writer is modifying but not committed the data.

Of course, the writer needs to wait some *grace period* to allow all readers of the old structure to finish their work on the old struct before deallocating it.

![read_copy_update_locks](assets/read_copy_update_locks.png)

So to recap:

- avoid deadlocks
- read locks acquisition are easy (they don't lock at all)
- writer might delay the completion of destructive operations

#### Locks and multiprocessing

In a SMP system, an attempt to acquire a lock requires moving the cache line containing that lock to the local CPU cache. 
If another CPU tries to check the lock it forces the value held in the cache of the first CPU to be written back to memory (to ensure consistency). If multiple processors are spinning on the same lock there is a overhead because of the need to ensure memory consistency

Mellor-Crummey and Scott lock (MCS or `queued spinlocks`) are a way to better implement spinlocks on multicore systems to avoid this overhead.

![MCS_lock](assets/MCS_lock.png)

The idea is:

- start with a spinlock that is not taken
- P1 comes and takes the lock
  - when the lock is taken, it allocates a processor specific data structure (each processor has it's own version of that data structure)
  - the lock points to that specific data structure
- if P3 arrives an tries to take the lock
  -  gets it's own copy of the struct
  - the P1 struct will point to P3 as `next locker`
- this way we have two advantages
  - each CPU spins on its own lock variable
  - when one CPU releases the lock it notifies the next one in line
  - no coherency needs to be enforced because each CPU is on a different variable and are given the lock in order on release

### Memory models

Defines the behaviour and the consistency of how operations done by one thread are seen by the other threads in a multiprocessor system. Due to write buffering, speculative execution and cache coherency the order of access of the memory may not be seen in the same way by another thread.
This depends on the way the architecture manages memory, for instance:

```C
// thread 1
x = 1;
done = 1;

// thread 2
while(done == 0) { /* loop */ }
print(x);
```
On an x86 machine it will always print `1` while on and ARM processor it can also print `0`.

We can distinguish 2 types of ordering:

- program order: order in which a thread does the memory accesses, denoted by `<_p` (happens before in program order)
- memory order: order in which the operation are seen by the shared memory, denoted by `<_m` (happens before in memory order)

We will cover 3 models:

- sequential: strongest possible and more difficult to achieve, do not exist in practice
  - if `i <_p j` then `i <_m j`, operations of each processor appear in the same order specified by the program that is running.
- total store order (TSO): model of x86

  ![total_store_order](assets/total_store_order.png)
  - each thread interfaces with the shared memory through a store buffer, so writes can be seen "delayed"
    - if the thread wants to read data that is still in the store buffer then it will read it from there --> the processor sees the data that it has written, even if other threads have not yet seen it
    - otherwise the data is pulled from the shared memory
    - all other threads see the write at the same time when it is written back to the shared memory
  - there is a `mfence` instruction that forces the flush of the write buffer. There are also `.L` (locked) instructions that require to get a global lock on the resource before executing (unlocking flushes the write buffer)
  - reads may be issued before writes due to speculation
- partial store order (PSO): model of ARM, much weaker model

  ![partial_store_order](assets/partial_store_order.png)
  - each processor reads from and writes as if it had its own complete copy of the memory
    - writes can be reordered by out-of-order execution in the same processor
    - no mechanism to ensure that all other processors see the writes at the same time

#### Data races

The PSO model can allow something like this to happen

![data_race_in_PSO](assets/data_race_in_PSO.png)

Because there is no enforcement of an happens before operation. We need to use some synchronization instructions. Not quite the same thing as a memory barrier, we just enforce that after a specific write, all the writes that happened before have been committed to the shared memory (note: doesn't mean that all the previous writes are committed in order, like TSO, just that they have been committed).
That specific write is called a *release* and to exploit it when reading it is needed to do an *acquire* operation.

This eliminates the data race.

If all data races can be removed then the program will appear as if it was sequentially consistent.

#### Software memory models

Compilers can introduce additional reordering of instructions that might appear as if the machine had a weaker memory model. Higher level languages must give programmers the ability to enforce some happen-before relationships. This is called language memory model.

If you write a data race free program then it will behave like a sequentially consistent one.

(see slides for C++ memory model)

##### Linux Kernel Memory Model (LKMM)

The model is essence the lowest common denominator of the guarantees of all the CPU families where the kernel can run.

- happens-before relationships can be enforced using `smp_store_release` and `smp_load_acquire`
- provides `atomic_t` and `atomic64_t`. Operations on this type are guaranteed to be non interruptible
  
  ```C
  atomic_t v = ATOMIC_INIT(0);
  atomic_set(&v, 4);
  atomic_add(2, &v);
  atomic_inc(&v);
  ```
## Virtual memory

Linux (as any other modern OS) uses virtual memory.

Physical pages are mapped to virtual pages, each process has a memory structure `mm_struct` that is used by the kernel to keep track of the mapping. 
Each task has it's own value for the register that points to the base of the page directory (`RC3` in x86).

Both kernel and user space code use virtual memory:

- higher memory addresses for the kernel
- lower memory addresses for user space programs

How it this managed?

- `KERNEL VIRTUAL` used for remapping some pages (e.g. pages that need to be contiguous in the virtual space but not necessarily in the physical). Allocated by `vmalloc`.
- `KERNEL LOGICAL` direct image of all the physical memory (offsetted by some address in order to live in the kernel memory area). Contiguous pages are also contiguous in the physical memory. Allocated by `kalloc`. Typically used for DMA.

Virtual pages can be mapped:

- directly to a physical memory pages
- not mapped to physical (e.g. swapped out to disk)
  - used to manage data that doesn't fit in memory
- neither of those, for instance pages allocated via `brk` to expand some area that are not mapped until they are used

A user mode process memory is composed of many Virtual Memory Areas (VMAs) and are stored in the `mm_struct` of the process and can be inspected in the `/proc/<name>/maps`.

![process_VMA](assets/process_VMA.png)

A VMA can be:

- anonymous: exists only in memory, not corresponding to a file (until they need to be swapped)
- file_backed (backing store): there is a corresponding file on disk, for instance the code for a program, a library

Also there are flags to specify what action can be done on them (read/write/exec/...)

Multiple VMA can point to the same physical pages to share resources (for instance library code)

### Page fault management

What happens when a process tries to access a page that is not present in memory? The page table contains only the pages that are present in memory, but the process can have more pages that are not present, managed in the VMAs.
There are 2 possibilities:

- page valid but it's not mapped
  - if so it needs to be loaded
- page is invalid
  - segmentation fault

To add pages to the VMA a process needs to call `brk()` (but pages are actually allocated only on access --> demand pages)

![page_fault_management](assets/page_fault_management.png)

The `find_vma` function uses a RB-tree to find the corresponding VMA (uses tree instead of list for efficiency).

- not found: SIGSEGV
- found:
  - check permissions
    - SIGSEGV if doesn't have permission (e.g. write to a read only area)
  - `handle_mm_fault` allocates a page table entry if needed
  - `handle_pte_fault` load from disk

New VMAs can be created by a process by calling `mmap()` on already open file descriptors. This allows to avoid copy of files in the space of the each process and share access to a *file cache* of already opened files.

### Physical address space

#### NUMA

Physical memory can be in different NUMA nodes (e.g. on multiprocessor systems). Linux keeps a data structure `pg_data_t` for each node in a list (If a single NUMA node is present there is only one element). 

![NUMA_nodes](assets/NUMA_nodes.png)

For each node there is several information stored:

- the memory of each node id divided in 3 zones:
  - `ZONE_DMA`: what zones can be used for DMA by other devices
  - `ZONE_NORMAL`: 
  - `ZONE_HIGHMEMORY`: not considered here
  Each zone has 
    - `free_area` structure that keeps track of the areas that are free, grouped by the number of contiguous pages available
    - `watermarks` 3 possible values (high, low, min), stating how many pages are free.
      ![memory_zones_watermarks](assets/memory_zones_watermarks.png) 
      - above `high`, pages are consumed by the buddy allocator
      - between `high` and `low`, the allocator wakes up the `kswapd` to start freeing up pages
      - between `low` and `min`, the allocator will swap pages itself
      - below `min`, normally not allowed, only in specific cases
- `node_mem_map` stores the whole available memory pages in that node
- `lruvec` keeps track of the activity of each pages, used to decide which pages to swap in case of necessity

#### Buddy allocator

The idea is:

- do not fragment contiguous blocks of pages too much
- compact free large blocks with little overhead

To allocate the blocks it works like this:

![buddy_allocator](assets/buddy_allocator.png)

- recursively split the block in half until the requested size is reached (minimum size is 1 page)
- the unused half of each block is the *buddy* of the other
- when merging back pages, a page can only be merged with its buddy

NOTE: blocks are always a power of 2 number of pages and are tracked in the `free_area`

### Page cache

Set of pages that are:

- in memory
- swappable
- file backed (i.e. have an associated backing store)

The idea is to efficiently use memory by sharing some physical pages and mapping them on multiple processes' VMAs (e.g. shared library code, copy-on-write child process pages).

Each page needs to keep a reference count (how many VMAs map to that page).

```C
struct page {
  unsigned long flags;
  atomic_t _count;
  atomic_t _mapcount;
  struct address_space *mapping;
  pgoff_t index;
  struct list_head lru;
};
```

#### Page frame reclaim algorithm (PFRA)

Based on the idea of the *clock algortithm*, an approximation of the LRU algorithm.

- circular list of pages
  - the clock head goes around the list to find some page to evict
- each page has a reference bit
  - R=1 --> recently used, don't evict but put R=0
  - R=0 --> choose to evict
    - may want to check if dirty to avoid writes if there are other candidates

In Linux we need a more complex way of managing pages, using the `lruvec`.

- INACTIVE_ANON and ACTIVE_ANON
- INACTIVE_FILE and ACTIVE_FILE
- UNEVICTABLE

active and inactive lists, the top of the inactive list is the candidate page to be evicted. A page stays in the active as long as it gets referenced in by some process.

![active_inactive_pages](assets/active_inactive_pages.png)

### Object allocation

Fast allocator for objects that are smaller. In general in the kernel fixed size data structures are ofter allocated and deallocated.
The buddy system doesn't scale well, it will lead to fragmentation and inefficient use of the memory.

There are two fast allocators in the kernel:

- quicklists, used only for paging
- slab allocator, used for other buffers
  - for smaller structure, store many object in a single page
  - for efficiency we have that frequently used structures are prepared and already initialized

![slab_allocator_implementation](assets/slab_allocator_implementation.png)

A `kmem_cache` structure is present for each type of data structure. It manages multiple caches, one for each CPU. When the page gets full it is swapped to another cache slab.
`kmalloc` will look in the slab cache to find a suitable block.

For more info can check `/proc/slabinfo`

### Linux memory security

Mitigation against common type of vulnerabilities.

#### Address Space Layout Randomization (ASLR)

Randomize the base address of the sections in memory to make it difficult to find code to execute. Also available in the kernel (KASLR) to randomize the .text section of the kernel at startup.

#### kernel Page Table Isolation (KPTI)

To protect against new attacks based on **Meltdown** that exploit the processor speculative execution to leak information.

The idea is to use different PGDs for user mode and kernel mode. The two page tables are adjacent so that to switch mode it is sufficient to change the base.

![KPTI](assets/KPTI.png)
