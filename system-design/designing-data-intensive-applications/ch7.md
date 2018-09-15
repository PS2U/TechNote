<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [The Slippery Concept of a Transaction](#the-slippery-concept-of-a-transaction)
  - [The Meaning of ACID](#the-meaning-of-acid)
    - [Atomicity](#atomicity)
    - [Consistency](#consistency)
    - [Isolation](#isolation)
    - [Durability](#durability)
  - [Single-Object and Multi-Object Operations](#single-object-and-multi-object-operations)
    - [Single-object writes](#single-object-writes)
    - [The need for multi-object transactions](#the-need-for-multi-object-transactions)
    - [Handling errors and aborts](#handling-errors-and-aborts)
- [Weak Isolation Levels](#weak-isolation-levels)
  - [Read Committed](#read-committed)
    - [No dirty reads](#no-dirty-reads)
    - [No dirty writes](#no-dirty-writes)
    - [Implementing read committed](#implementing-read-committed)
  - [Snapshot Isolation and Repeatable Read](#snapshot-isolation-and-repeatable-read)
    - [Implementing snapshot isolation](#implementing-snapshot-isolation)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

In the harsh reality of data systems, many things can go wrong:

- The database software or hardware may fail at any time (including in the middle of a write operation).
- The application may crash at any time (including halfway through a series of operations).
- Interruptions in the network can unexpectedly cut off the application from the database, or one database node from another.
- Several clients may write to the database at the same time, overwriting each other’s changes.
- A client may read data that doesn’t make sense because it has only partially been updated.
- Race conditions between clients can cause surprising bugs.

A transaction is a way for an application to group several reads and writes
together into a logical unit: either the entire transaction succeeds (commit) or it fails (abort, rollback). If it fails, the application can safely retry. 

By using transactions, the application is free to ignore certain potential error scenarios and concurrency issues, because the database takes care of them instead (we call these safety guarantees).

# The Slippery Concept of a Transaction

Almost all relational databases today, and some nonrelational databases, support transactions. Many NoSQL databases abandon transactions entirely, or redefine the word to describe a much weaker set of guarantees for good performance and high availability.

## The Meaning of ACID

In practice, one database’s implementation of ACID does not equal
another’s implementation. For example, as we shall see, there is a lot of ambiguity around the meaning of isolation. 

Systems that do not meet the ACID criteria are sometimes called BASE, which stands for Basically Available, Soft state, and Eventual consistency. This is even more vague than the definition of ACID.

### Atomicity

In the context of ACID, atomicity is not about concurrency.

The ability to abort a transaction on error and have all writes from that transaction discarded is the defining feature of ACID atomicity.

### Consistency

The word consistency is terribly overloaded:

- Replica consistency.
- Consistent hashing.
- Consistency in CAP theorem means linearizability.
- Consistency in ACID refers to an application-specific notion of the database being in a "good state".

Atomicity, isolation, and durability are properties of the database, whereas consistency (in the ACID sense) is a property of the application. The application may rely on the database’s atomicity and isolation properties in order to achieve consistency, but it’s not up to the database alone.

### Isolation

Isolation in the sense of ACID means that concurrently executing transactions are isolated from each other: they cannot step on each other’s toes. 

The classic database textbooks formalize isolation as serializability, which means that each transaction can pretend that it is the only transaction running on the entire database. The database ensures that when the transactions have committed, the result is the same as if they had run serially (one after another), even though in reality they may have run concurrently.

![](/img/ch7-race-condition-of-two-clients.jpg)

However, in practice, serializable isolation is rarely used, because it carries a performance penalty.

### Durability

Durability is the promise that once a transaction has committed successfully, any data it has written will not be forgotten, even if there is a hardware fault or the database crashes.

In a replicated database, durability may mean that *the data has been successfully copied to some number of nodes*. In order to provide a durability guarantee, a database must wait until these writes or replications are complete before reporting a transaction as successfully committed.

## Single-Object and Multi-Object Operations

Multi-object transactions require some way of determining which read and write operations belong to the same transaction. In relational databases, that is typically done based on the client’s TCP connection to the database server: on any particular connection, everything between a BEGIN TRANSACTION and a COMMIT statement is considered to be part of the same transaction.

On the other hand, many nonrelational databases don’t have such a way of grouping operations together. Even if there is a multi-object API, that
doesn't necessarily mean it has transaction semantics: the command may succeed for some keys and fail for others, leaving the database in a partially updated state.

### Single-object writes

Atomicity can be implemented using a log for crash recovery, and isolation can be implemented using a lock on each object (allowing only one thread to access an object at any one time).

Some databases also provide more complex atomic operations, such as an increment operation, which removes the need for a read-modify-write cycle . Similarly popular is a compare-and-set operation, which allows a write to
happen only if the value has not been concurrently changed by someone else.

### The need for multi-object transactions

- In a relational data model, a row in one table often has a foreign key reference to a row in another table. Multi-object transactions allow you to ensure that these references remain valid.
- In a document data model, no multi-object transactions are needed when updating a single document. However, document
databases lacking join functionality also encourage denormalization. When denormalized information needs to be updated, you need to update several documents in one go. 
- In databases with secondary indexes, the indexes also need to be updated every time you change a value. 

### Handling errors and aborts

ACID databases are based on this philosophy: if the database is in danger of violating its guarantee of atomicity, isolation, or durability, it would rather abandon the transaction entirely than allow it to remain half-finished.

Retrying an aborted transaction is not perfect:

- If the transaction actually succeeded, but the network failed while the server tried to acknowledge the successful commit to the client.
- If the error is due to overload, retrying the transaction will make the problem worse, not better.
- If the transaction also has side effects outside of the database, those side effects may happen even if the transaction is aborted.
- If the client process fails while retrying, any data it was trying to write to the database is lost.

# Weak Isolation Levels

Concurrency issues (race conditions) only come into play when one transaction reads data that is concurrently modified by another transaction, or when two transactions try to simultaneously modify the same data.

Isolation is unfortunately not that simple. Serializable isolation has a performance cost, and many databases don’t want to pay that price. It’s therefore common for systems to use weaker levels of isolation, which protect against some concurrency issues, but not all.

Rather than blindly relying on tools, we need to develop a good understanding of the kinds of concurrency problems that exist, and how to prevent them. Then we can build applications that are reliable and correct, using the tools at our disposal.

## Read Committed

The most basic level of transaction isolation is read committed. It makes two guarantees:

1. When reading from the database, you will only see data that has been committed (no dirty reads).
2. When writing to the database, you will only overwrite data that has been committed (no dirty writes).

### No dirty reads

Any writes by a transaction only become visible to others when that transaction commits.

![](/img/ch-no-dirty-reads.jpg)

There are a few reasons why it's useful to prevent dirty reads:

- If a transaction needs to update several objects, a dirty read means that another transaction may see some of the updates but not others.
- If the database allows dirty reads, that means a transaction may see data that is later rolled back.

### No dirty writes

By preventing dirty writes, this isolation level avoids some concurrency problems if transactions update multiple objects.

### Implementing read committed

Most commonly, databases prevent dirty writes by using row-level locks: when a transaction wants to modify a particular object (row or document), it must first acquire a lock on that object.

How do we prevent dirty reads? One option would be to use the same lock, and to require any transaction that wants to read an object to briefly acquire the lock and then release it again immediately after reading.

However, the approach of requiring read locks does not work well in practice, because one long-running write transaction can force many read-only transactions to wait until the long-running transaction has completed.

## Snapshot Isolation and Repeatable Read

![](/img/ch7-alice-read-skew.jpg)

Read skew is considered acceptable under read committed isolation: the account balances that Alice saw were indeed committed at the time when she read them.

However, some situations cannot tolerate such temporary inconsistency:

- Backups
- Analytic queries and integrity checks

*Snapshot isolation* is the most common solution to this problem. The idea is that each transaction reads from a consistent snapshot of the database—that is, the transaction sees all the data that was committed in the database at the start of the transaction. Even if the data is subsequently changed by another transaction, each transaction sees only the old data from that particular point in time.

### Implementing snapshot isolation

Like read committed isolation, implementations of snapshot isolation typically use write locks to prevent dirty writes, which means that a transaction that makes a write can block the progress of another transaction that writes to the same object. However, reads do not require any locks.

Readers never block writers, and writers never block readers. This allows a database to handle long-running read queries on a consistent snapshot at the same time as processing writes normally, without any lock contention between the two.

The database must potentially keep several different committed versions of an object, because various in-progress transactions may need to see the state of the database at different points in time. This technique is known as multi-version concurrency control (MVCC).

Storage engines that support snapshot isolation typically use MVCC for their read committed isolation level as well. A typical approach is that read committed uses a separate snapshot for each query, while snapshot isolation uses the same snapshot for an entire transaction.

![](/img/ch7-snapshot-isolation-multi-version.jpg)




