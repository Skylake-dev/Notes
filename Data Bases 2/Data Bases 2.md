# Data Bases 2

## Transactions
Atomic unit of work performed by an application. Conceptually, each  transaction is encapsulated within:
- begin of transaction (BoT)
- end of transaction (EoT)  

At the end of the transaction only one of the following commands is executed:
- commit-work: save the changes to the database permanently
- rollback-work: discard changes  

This is done to guarantee the ACID properties:
- **Atomicity**: all or nothing semantics for transaction, it either completes or fails, no partial execution.
- **Consistency**: enforce integrity constraints on data, DBMS can automatically rollback transactions that do not satisfy these constraints.
- **Isolation**: the execution of transaction should not interfere with other concurrent transactions, the result must be the same as if the transactions were issued in order.
- **Durability**: the effect of committed transactions must be stored long term.

These properties are enforced by different modules of the DBMS:
- Atomicity and durability -> reliability manager
- Isolation -> concurrency control system
- Consistency -> integrity control system at query execution time  

![DBMS_modules](assets/DBMS_modules.png)  

## Concurrency control
How can we serve multiple concurrent transactions while maintaining the ACID properties?  
Serializing (i.e. execute one at a time) is not an option because it would create a bottleneck and lose performance. We want to serve the transactions concurrently. Concurrency can lead to have a series of *anomalies* if we are not careful in accessing the data.

### Anomalies
Explained using examples. Assume transactions occur concurrently without each of them knowing about the others and can interleave.

#### Lost update  
The update of a transaction is overwritten by another transaction. The sequence of operation to have this occur is:
- T1 reads data `x`
- T2 reads the same data `x`
- T1 and T2 both update the value of `x`
- T1 writes `x`
- T2 writes `x`  

The result is that the update made by T1 is overwritten by T2 because T2 read the same starting value as T1. Clearly this would not have happened if T1 and T2 happened in sequence (T1 and then T2) since T2 would have read the value of `x` written by T1 and update over it.

#### Dirty read
A transaction reads an uncommitted value written by another transaction that eventually rolls back. The sequence is:
- T1 updates the value of `x`
- T2 reads `x`
- T1 aborts, rollback the value of `x`
- T2 retains the value of `x` that it has read before but have been discarded
- T2 updates the value and writes it.  

The result is that the update made by T2 is wrong because it is based on a value that never happened in the database since it was rolled back.

#### Non-repeatable read
A transaction reads two times the same piece of data but it obtains a different value. The sequence is:
- T1 reads value of `x`
- T2 reads and update the value of `x`
- T1 reads again `x` <-- the value has changed even if T1 did not update it.

From the point of view of T1 the value of `x` has changed by itself because it did not make any modification itself.

#### Phantom update
Let's suppose that we have a constraint on some data, for example `A + B + C = 100`
- T1 reads `A` and `B`
- T2 reads `B` and `C`
- T2 updates them consistently with the constraint, for example `B = B + 10` and `C = C -10`
- T2 writes the updated values (constraint still holds)
- T1 reads `C`
- For T1 the constraint doesn't hold

The problem here is that T1 has both old and new data because it read only part of the data involved in the constraint. For T1 the sum is no longer 100 but the data in the database is still correct.

#### Phantom insert
Another transaction inserts a new record that changes the value of an aggregate operation.  
- T1 computes an aggregate operation on part of the data, for example computes the average value of a column
- T2 inserts a new record, adding a new value in the column read by T1
- T1 when it recomputes the same average will obtain a different result because there is a new record that previously wasn't there

#### Recap
Mnemonics to remember the anomalies
- Lost update: R1 R2 W1 W2
- Dirty read: R1 W1 R2 abort1 W2
- Non-repeatable read: R1 R2 W2 R1
- Phantom update: R1 R2 W2 R1
- Phantom insert: R1 insert2 R1

### Modeling concurrency
First, some definition and notation:
- operation: read or write on a specific piece of data
  - R1(x): transaction 1 read the value of `x`
  - W2(x): transaction 2 update value `x`
- schedule: sequence of operation performed by concurrent transactions such that the order of operations in each transaction is respected (i.e. possible interleavings fo different transactions).  
Example:
  - T1: R1(x) W1(x)
  - T2: R2(x) W2(x)
  - possible schedule: R1(x) R2(x) W1(x) W2(x)

How many schedules are possibles? A LOT, it's factorial w.r.t. the number of operation and transactions --> cannot enumerate them and pick the ones without anomalies.  
Only the serial schedules (excute the transactions in order T1, T2, ..., Tn) are already `n!`, but a serial schedule has very low throughput and high latency (all transactions need to wait).

GOAL: reject schedules that cause anomalies. The scheduler is the component that accepts or rejects operations requested by transactions.

#### Serializable schedule
Schedule that leaves the database in the same state as *some* serial schedule of the same transactions. This ensures correctness.  
In this simple model we have very strict assumptions:
- all transactions have committed (commit-projection)
- the schedule is evaluated *a posteriori*, after the operations have been performed when i can observe the whole sequence.

![serializable_schedules](assets/serializable_schedules.png)

In order to understand when a schedule is serializable we need to define a notion of equivalence between schedules.

#### View serializability
Definitions:
- reads-from: Ri(x) reads from Wj(x) is a schedule S when
  - Wj(x) precedes Ri(x)
  - there aren't any other writes in between
- final write: Wi(x) is the final write in a schedule if it is the last write on item x  

Two schedules `S_i` and `S_j` are *view-equivalent* if they have:
- the same operations
- the same reads-from relations
- the same final writes  

A schedule is *view serializable* if it is view-equivalent to a serial schedule for the same transactions. The class of serializable schedules is called **VSR**.

NOTE: two serial (serializabile) schedules can leave the database in different states in the end. This is not the point, the point is to avoid anomalies.  

PROBLEM WITH VIEW SERIALIZABILITY  
Checking if two schedules are view equivalent is polynomial in time, but checking if a generic schedule is in VSR is an NP-complete problem because it requires to consider all reads from and final writes of serial schedules (combinatorial).  
What can we trade in order to get more performance? Accuracy, we can define a looser but easier to check model. This may leave out some good (i.e. that do not cause anomalies) schedules but makes the check feasible.

#### Conflict serializability
Two operations are in conflict if they address the same resource and at least one of them is a write:
- read-write conflict (r-w or w-r)
- write-write conflict (w-w)

Two schedules `S_i` and `S_j` are *conflict-equivalent* if:
- they contain the same operations
- for all conflicting operation pairs transactions occur in the same order

A schedule is *conflict serializable* if it is conflict-equivalent to a serial schedule. The class of serializable schedules is called **CSR** and we can prove that it is strictly included in VSR.

Theorem: `CSR ⊂ VSR`

How to test conflict serializability: build conflict graph
- one node for each transaction
- one arc from `T_i` to `T_j` if there is at least one conflict between an operation `o_i ∈ T_i` and `o_j ∈ T_j` and `o_i` precedes `o_j`

Theorem: a schedule is conflict serializable if the conflict graph is acyclic.

NOTE: the opposite is not true, if the graph contains a cycle we cannot conclude neither that is is CSR nor that it isn't.

Graphical recap:  
![VSR_CSR_schedules](assets/VSR_CSR_schedules.png)  

### Concurrency control in practice
CSR checking is efficent but it only works *a posteriori*. Real DBMS schedulers need to make decisions *online*, managing the requests as they arrive.  
We can use two approaches:
- pessimistic, lock based approaches, higher isolation
- optimistic, timestamps and versioning, higher throughput

#### Locking
A transaction is well formed w.r.t. locking if:
- read operations are preceded by a `R_LOCK` (shared lock) and followed by `UNLOCK`
- write operations are preceded by a `W_LOCK` (exclusive lock) and followed by `UNLOCK`  

Transactions that first read and then write can acquire an exclusive lock directly or acquire a shared lock and upgrade it later.

The objects in a database can have 3 states:
- `FREE`
- `R_LOCKED`
- `w_LOCKED`  

| REQUEST | FREE | R_LOCKED | W_LOCKED |
| -- | -- | -- | -- |
| R_LOCK | OK -> R_LOCKED | OK -> R_LOCKED* | NO -> W_LOCKED |
| W_LOCK | OK -> W_LOCKED | NO -> R_LOCKED | NO -> W_LOCKED |
| UNLOCK | ERROR | OK -> R_LOCKED/FREE* | OK -> FREE |  

NOTE: * multiple transactions can have a read lock on the same resource (it's a shared lock), the DBMS keeps track of how many transactions are sharing the lock.

Following this rules, some transactions will have to wait until the resources they need are free to use. The arrival sequence can be different from the schedule a posteriori because some transactions can be delayed.

##### Implementation: lock tables
Lock tables are hash tables that index the lockable items by hashing them.  
every node has a linked list with all the transactions that requested a lock for that resource, the lock mode (shared/exclusive) and the status (granted/pending). When a transaction ends it is removed from the list.

![lock_table](assets/lock_table.png)  

##### Locks and serializability
Is respecting locks enough to guarantee serializability? NO, some anomalies may still occur. This is caused by the fact that transactions may release locks early and reacquire it later.

Example:
- T1 locks resource `x` with a shared lock to read its value
- T1 releases the lock after the read
- T2 locks `x` with an exclusive lock
- T2 updates `x`
- T2 releases the lock
- T1 reacquires the shared lock on `x` to read it
- T1 finds that the value has changed --> non-repeatable read

#### 2PL
Aims at preventing non repeatable reads.  
A transaction cannot acquire more locks after it has released a lock. This simple rule not only prevents non-repeatable reads but also ensures serializability.  

![2PL](assets/2PL.png)  

Theorem: 2PL is strictly contained into CSR.  

![2PL_CSR_VSR_schedules](assets/2PL_CSR_VSR_schedules.png)  

What about other anomalies? The ones that cause problems are:
- phantom inserts because it requires to lock data in a "future-proof" way, not only the data that i am currently fetching. We will solve this issue using predicate locks.
- dirty reads because it requires to deal with aborts which so far we have not done in our model.

##### Strict 2PL
Releasing a lock before committing exposes uncommitted data. Other transactions can then read this data and use it and this leads to anomalies in case of abort. The solution is pretty simple.  
A transaction hold all its locks until the decision point (commit/rollback). This way nobody can read data that was not committed.

![strict_2PL](assets/strict_2PL.png)

##### Predicate locks
To prevent phantom inserts we need, as said before, to "lock future data", that is block the insertion of data that would satisfy a previous query made by a transaction.  
Let's suppose that we have a table with two columns `A` and `B`  

| A | B |
| -- | -- |
| data | data |
| .. | .. |  

Transactions T1 updates the data with the following command (pseudocode):  
`T1: update B where A < 1`  
The concept of predicate lock disallow other transactions to insert, delete or update any record that satisfies the predicate `A < 1`. This prevents phantom inserts because no other transaction can modify the data that T1 is working with.

#### Isolation levels
DBMS in practice allow to specify so called *isolation levels* for the transactions. These specify what anomalies we allow in order to achieve more concurrency and performance.  
Not all transactions need to have the same isolation level, it can be set on a per-transaction basis.  
These levels do not affect write locks that are **always kept until the decision point** (strict 2PL on write locks), regardless off the isolation level. If this doesn't occur we cannot ensure rollback in case of aborts, jeopardizing consistency (dirty write).  
On the other hand we can have different levels for read locks:
- READ UNCOMMITTED, no read locks  
allowed anomalies: dirty reads, non-repeatable reads, phatom inserts, phantom updates
- READ COMMITTED, read locks but not 2PL  
allowed anomalies: non-repeatable reads, phantom inserts, phantom updates
- REPEATABLE READ, read locks with 2PL  
allowed anomalies: phantom inserts
- SERIALIZABLE, read locks with 2PL and predicate locks  
allowed anomalies: none (only advised for distributed transactions)

Recap table
| ISOLATION LEVEL | DIRTY READ | NON-REPEATABLE READ | PHANTOM UPDATE | PHANTOM INSERTS |
| -- | -- | -- | -- | -- |
| READ UNCOMMITTED | YES | YES | YES | YES |
| READ COMMITTED | NO | YES | YES | YES |
| REPEATABLE READ | NO | NO | NO | YES |
| SERIALIZABLE | NO | NO | NO | NO |  

#### Problems of locking: deadlocks and starvation
##### Deadlocks
Deadlocks happen when a transaction holds a lock that the another transaction needs and viceversa, blocking both transactions in a infinite wait. Of course more than two transactions can be involved in a deadlock, the idea is that there is a circular dependence between them (i.e. T1 waits for T2 that waits for T3 that waits for T1).  
There are different approaches to deal with deadlocks:
- timeouts: kill transactions that wait for longer than a certain threshold.  
Impractical approach because it is difficult to correctly balance the timeout, transactions may require very different time to execute even within the same application.
- prevention: killing transactions that could cause deadlocks.  
Transactions are assigned an ID as they arrive. The heuristic that is used is that an older transaction should not wait for a younger transaction. If this happens the younger transaction is killed and restarted. Also this approach is inefficient because waiting does not imply that there is a deadlock so this leads to a lot of unnecessary kills.
- detection: let deadlocks happen and resolve them when they occur.  
Build the *waits-for* graph to detect cycles of waits and choose a transaction to kill according to some policy. Works in local DBMS.

NOTE: detection in a distributed DBMS  
OBERMARK'S ALGORITHM  
Recontructs the whole dependency graph using only the view of a single node. Each node needs to exchange only minimal amount of information, node A sends info to node B only if:
- A contains a transaction `T_i` that is waited for from another transaction and waits for `T_j` on B
- `i < j` (or viceversa, it doesn't have a semantics) to ensure information is propagated only one way

The algorithm is executed periodically at each node and consists in:
- get graph info from previous nodes
- update local graph with information from remote nodes
- check if there are cycles
  - yes: kill one in the cycle following some policy
  - no: do nothing
- send update to other nodes

##### Deadlocks in practice
If i have `n` records, assuming uniform distribution for accessing:
- conflict probability is `O(1/n)`
- deadlock probability is `O(1/n^2)`

Still they do occur (once in a minute in a medium sized database) and long running transactions are more likely to deadlock.  
There are techniques to limit the frequency of deadlocks:
- update lock
- hierarchical lock

##### Update locks
Most deadlocks involves only 2 transaction and happen because they lock some resource in a shared way with the intention of upgrading it later.  
Update locks allow to state the intention of upgrading the lock later in order to prevent other transactions to express the same will.  
There are 3 possible lock request:
- SHARED LOCK (SL): express the intention to read only
- UPDATE LOCK (UL): express the intention to read and then upgrade to an exclusive lock later
- EXCLUSIVE LOCK (XL): express the intention to write

The interactions work as follows  
| REQUEST | SL | UL | XL |
| -- | -- | -- | -- |
| SHARED LOCK | YES | YES | NO |
| UPDATE LOCK | YES | **NO** | NO |
| EXCLUSIVE LOCK | NO | NO | NO |

Update locks are requested using the SQL statement SELECT FOR UPDATE.

##### Hierarchical locks
Locks can be specified with different granurality.  
Objectives:
- lock minimum amount of data
- recognize conflicts as soon as possible  

The idea that a transaction expresses the intention to lock a finer level of granularity (for instance: table -> page -> tuple -> field).  
In addition to SL and XL there are 3 more locks:
- ISL: intention to lock a sub-element of the current element in shared mode
- IXL: intention to lock a sub-element of the current element in exclusive mode
- SIXL: lock in shared mode with the intention to lock a sub-element in exclusive mode (basically SL + IXL)

The protocol to lock is the following:
- lock request starts from the root of the hierarchy and go down the levels
- locks are released in reverse order, going up in the hierarchy
- to request a lock on a given level the transaction must hold an equal or stricter lock at an higher level
- locks requests are granted following this decision table

| REQUEST | ISL | IXL | SL | SIXL | XL |
| -- | -- | -- | -- | -- | -- |
| ISL | YES | YES | YES | YES | NO |
| IXL | YES | YES | NO | NO | NO |
| SL | YES | NO | YES | NO | NO |
| SIXL | YES | NO | NO | NO | NO |
| XL | NO | NO | NO | NO | NO |

Very useful approach to avoid locking an entire table if i only need to act on a small subset of the records. Of course when i want to perform the action i have stateted my intention to perform i have to get an actual SL or XL.

#### Timestamp-based approaches
A timestamp is a unique identifier that defines a total ordering of the events in a system. In a centralized system it is easy because all requests are on the same node, in a distributed system we can use the Lamport clocks to assign timestamps without a global clock.  
For each element `x` the scheduler keeps two timestamps:
- `RTM(x)` timestamp of the last transaction that read `x`
- `WTM(X)` timestamp of the last transaction that wrote `x`

The scheduler receives R/W requests with the timestamp `ts` of the transaction that issued it. The algorithm to grant access is the following:  

```python
read(x, ts):
  if ts < WTM(x):
    #reject and kill, the request is obsolete the value was updated
  else:
    #granted and update RTM if ts is more recent
    RTM(x) = max(RTM(x), ts)

write(x, ts):
  if ts < RTM(x) or ts < WTM(x):
    #reject and kill, someone younger already read the value or updated it
  else:
    #granted and update WTM
    WTM(x) = ts
```

The schedules produced by this algorithm belong to the TS class. How do these schedules relate to the other?
- 2PL: not comparable, there can be some schedules that are 2PL but not TS and viceversa.
- CSR: TS is strictly included in CSR

![VSR_CSR_2PL_TS_schedules](assets/VSR_CSR_2PL_TS_schedules.png)  

We can have some optimizations to the basic TS algorithm to avoid killing transactions unnecessarily. For example Thomas rule changes condition to kill a transaction in writing:

```python
write(x, ts):
  if ts < RTM(x):
    #kill, a younger transaction already read the value
  else if ts < WTM(x):
    #the write has already been overwritten, can be skipped without killing
  else:
    #granted and update WTM
    WTM(x) = ts
```

TS with Thomas rule is "bigger" than TS and sometimes can produce schedules that are not in CSR but still VSR.

NOTE: TS and dirty writes  
To prevent dirty reads a transaction that reads or writes and has a timestamp greater than WTM (`ts > WTM(x)` i.e. acceptable) must wait until the transaction that wrote that value has committed or aborted.

#### Multiversion concurrency control
IDEA: writes generate new copies (new versions) of data, a read request accesses the "right" value w.r.t. its timestamp.  
Each write generates a new copy with its own timestamp for the write. In the end there are N copies each with its WTM_N. Old copies needs to be discarded when all transaction that could access it have completed.  
With this mechanism the reads are always allowed (avoid killings) following the protocol:

```python
read(x, ts):
  #always accepted and update the RTM
  RTM(x) = ts
  if ts < WTM(x):
    #get latest value
  else:
    #get value k such that WTM_k(x) <= ts <= WTM_k+1(x)

write(x, ts):
  if ts < RTM(x):
    #reject the request
  else:
    #new version and set WTM
    WTM_n(x) = ts
```

This avoids the problem of killing long running transactions only because a newer and faster transaction has written some value in the database (which is a very common occurrence).

The drawback of this approach (denoted as TS-multi) is that it can generate some schedules that are not even in VSR, so some long running transaction may not get the most up to date value but a value that was current when they started (the older version). 

![TS_multi_schedules](assets/TS_multi_schedules.png)

##### Snapshot isolation
The typical implementation of multiversion in DBMS. It does not use RTM but only WTMs for each version.  
Every transaction is assigned a timestamp and when it reads the DBMS gives it a value consistent with the timestamp (i.e. the value that there was when the transaction started).

Snapshot isolation does not guarantee serializability. For example:

T1: UPDATE Balls set Color=White where Color=Black  
T2: UPDATE Balls set Color=Black where Color=White

A serializable execution of these two transaction will leave all balls either white or black. Under snapshot isolation instead the result will be that the two colors have swapped. This anomaly is called **write skew**.
