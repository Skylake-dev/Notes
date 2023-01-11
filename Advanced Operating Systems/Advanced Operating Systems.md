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
