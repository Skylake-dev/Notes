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

#### Concurrency control in practice
CSR checks is efficent but it only works *a posteriori*. Real DBMS schedulers need to make decisions *online*, managing the requests as they arrive.