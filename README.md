# Hash Hash Hash

This project has 3 hash table implementation: Base, v1, and v2. Base implementation is the serial hash implementation while v1 and v2 are not serial. V1 should be slower than base while V2 should be significantly faster than V1 and base. The goal of this project is to add one mutex lock in V1 and as many mutex lock in V2. In V1, we only care about correctness while V2, we care about performance and correctness. So essentially, we want to make V2 and V1 safe from race conditions and do it in n-1 (cores) faster efficiency than just serial implementation.

## Building

```shell
make
```

## Running

```shell
./hash-table-tester -t 8 -s 50000
```

## First Implementation

In the `hash_table_v1_add_entry` function, I initalized one mutex and declared it as static just after the include statements.
I put the mutex lock at the beginning of the function so that no multiple threads can access the whole code of the hash table. So, setting locks around the entire thing causes the shared writes to be protected as well. I've also put unlock inside the if statement so that once you're done updating the value in the hash table, unlock the key and return. If I don't unlock the lock even after updating the value, that thread will just stay there "waiting" to be unlocked, so that can cause an error/bad programming practice. Otherwise when the value doesn't exist, add the new key and value onto the list and unlock the mutex afterwards to ensure that that thread has been locked and unlocked successfully.

### Performance

```shell
./hash-table-tester -t 8 -s 50000
```

Version 1 is slower than the base version because this version is only using a SINGLE mutex and therefore all 8 threads are waiting for the one thread to be done, causing the thread overhead to be significantly high. An example output is as follows:

-Hash table base: 1,206,335 usec: 0 missing
-Hash table v1: 1,655,196 usec: 0 missing

Also because locking the entire funtion would cause every addition to be done sequentially instead of concurrently. Which means the CPU is significantly low and more work are not getting done at the same time.

## Second Implementation

In the `hash_table_v2_add_entry` function, instead of adding a single static mutex before the include statement, I moved it into the struct hash_table_entry and added a mutex field called pthread_mutex_t foo_mutex. This will ensure that whenever I lock/unlock a mutex inside the hash_table_v2_add_entry function, it will call as many mutex (n mutexes) as it likes for each hash table entry and ensure maximum performance. In addition, most of the locking and unlocking place is almost the same in V2 as in V1, but I put the lock a line after struct list_entry \*list_entry = get_list_entry(hash_table, key, list_head). So in other words, the locking is done by locking the ability to update/create an entry, so no 2 threads can access the if statement, hence trying to update/add the same entry at the same time. So if the entries are of different values, the 2 threads are free to update concurrently. My reasoning is because the performance is not great when I lock starting from getting the list entry node, so I moved the lock to where it is starting to check if the value already exists or not. By doing so will increase the performance most of the time.

### Performance

```shell
./hash-table-tester -t 8 -s 50000
```

An example output of V2 compare to V1 and base:
-Hash table base: 1,050,660 usec: 0 missing
Hash table v1: 1,371,125 usec: 0 missing
-Hash table v2: 328,051 usec: 0 missing

-Generation: 72,491 usec
-Hash table base: 1,056,280 usec: 0 missing
-Hash table v1: 1,457,660 usec: 0 missing
-Hash table v2: 365,203 usec: 0 missing

-Generation: 73,833 usec
-Hash table base: 1,077,318 usec: 0 missing
-Hash table v1: 1,514,066 usec: 0 missing
-Hash table v2: 335,678 usec: 0 missing

Out of these 3 trial runs, 2 out of the runs were that V2 is faster than the base by 3 times. Initializing the mutex inside
the hash_table_entry struct really helps the performance by a lot because now the program is relies on how many mutex the
struct can hold, not just on a SINGLE mutex where multiple threads are waiting all at once. So now the thread can wait much
faster and use the critical section more efficiently and concurrently. Therefore, the speed is multiplied by 3x.

Note that both V2 and V1 passes the valgrind (so no memory leak)

## Cleaning up

```shell
make clean
```
