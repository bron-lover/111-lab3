# Hash Hash Hash

In this lab, I modified a serial hash implementation to make it safe to use concurrently by multiple threads. The hash table used in this
lab was built using seperate chainging with singly linked list, where each each index of the hash table points to the head of a different linked list. I then implemented two different locking strategies using a mutex using the pthread_mutex functions and then compared the performance of using a single lock or multiple locks to the base hash table implementation provided.

## Building

To correctly build, run the command:

```shell
make
```

## Running

After building, run the command:
./hash-table-tester -t [# of threads] -s [# of insertions per thread]

Example:

```shell
./hash-table-tester -t 8 -s 50000

Generation: 87,988 usec
Hash table base: 1,522,957 usec
  - 0 missing
Hash table v1: 2,228,849 usec
  - 0 missing
Hash table v2: 437,942 usec
  - 0 missing
```

\*Missing: how many inserted keys were missing (due to a race condition)

## First Implementation

In the `hash_table_v1_add_entry` function, I added a single mutex that locks the entire function during each function, effectively
serializing every operation. This means that only one thread can add an entry at a time, regardless of what bucket the key hashes to.
This is gaurenteed by the single mutex implementation because when one thread enters the function, even if two threads are inserting into the same bucket at the same time, they must wait for each other as only one thread can possess the mutex at a given time. This effectively eliminates race conditions and things of that sort. This holds true even if two threads are not even inserting to the same bucket.
My implementation is correct because I locked at the start of the add function and unlocked right before any return statement, meaning no two threads could insert to the hash table at the same time. By doing this, I ensure that the state of the hash table cannot be modified concurrently, thus eliminating the chance for any race condition. However, this has a draw back when it comes to performance (I'll touch on this later in the report).

### Performance

Example run:

```shell
./hash-table-tester -t 1 -s 100000

...
Hash table base: 53,112 usec
  - 0 missing
Hash table v1: 57,020 usec
  - 0 missing
...

./hash-table-tester -t 8 -s 100000

...
Hash table base: 8,848,757 usec
  - 0 missing
Hash table v1: 11,411,704 usec
  - 0 missing
...
```

Version 1 is a little slower than the base version, as multiple threads contend for a single mutex. Due to the fact that unlocking and locking the mutex every time introduces some level of overhead, as we increase the number of threads (from 1 to 8 in the example I gave), we also increase mutex contention, context switiching, and overhead in general. That is why we see the difference in performance increase between the base and v1 when we increase the number of threads. In essence, instead of seeing a speedup due to parrelelizing and using more threads, we see a kind of road block as the single mutex implementation creates multi-threading overhead with the constant need to lock and unlock the mutex upon every insertion.

## Second Implementation

In the `hash_table_v2_add_entry` function, I a more specialized locking system where each bucket had its own mutex. So, locking would only occur when two threads try to insert to the same bucket (determined by taking the hash of the key), not upon any insertion that had no relation to one another. In simpler words, this allows multipe threads to actually operate concurrently as there is not reason to lock if we know that the threads are not altering the same bucket. Only threads targeting the SAME bucket will be in contention for the mutex.
My implementation of v2 is correct because the locking happens before editing / inserting a bucket's linked list, and only that specific bucket's mutex is actually locked. Then, similar to v1, the mutex is unlocked before any return statement. This prevents concurrent modifications to the same bucket, which is the main cause of race conditions such as an insert being over written, without the performance drawbacks of v1. So, main difference between v1 and v2 is that in v2, no two threads can simultaneously modify the same bucket, whereas in v1, no two threads can simultaneously modify the table as a whole. Looking at it this way, it becomes clear why v2 can outperform the base with multiple threads.

\* Test 1 Passed : Slower than hash-table-base --> 15 pts

### Performance

```shell
./hash-table-tester -t 1 -s 100000

...
Hash table base: 53,112 usec
  - 0 missing
...
Hash table v2: 62,196 usec
  - 0 missing

./hash-table-tester -t 8 -s 100000

...
Hash table base: 8,848,757 usec
  - 0 missing
...
Hash table v2: 2,765,496 usec
  - 0 missing
```

As you see, when we have a single thread, we are still slower than the base, which can be attributed to the overhead introduced by locking and unlocking mutexes. However, as we increase the number of threads, v2 significantly outperforms both base and v1 as threads are able to insert in parallel, only blocking when two try to insert to the same bucket. Essentially, v2 is faster than v1 becuase when v1 serializes all insertions whereas v2 allows parallel insertions into different buckets. Again, this happens because v2 only leads to contention when multiple threads hash to the same bucket. Since race conditions in the context of this lab occur when trying to insert to the same bucket and one insertion gets overwritten, v2 improves performance while also maintaining correctness as, again, we cannot simutaneously write to the same bucket.

\*Test 1 Passed : 2.7M <= 8.8M / (4 - 1) --> 2.9M --> 55 pts

## Cleaning up

To remove all binary files, run the command:

```shell
make clean
```
