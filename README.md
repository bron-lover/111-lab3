# Hash Hash Hash

TODO introduction

## Building

```shell
make
```

## Running

To run, you run the command:
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

## First Implementation

In the `hash_table_v1_add_entry` function, I added TODO

### Performance

```shell
TODO how to run and results
```

Version 1 is a little slower than the base version. As TODO

## Second Implementation

In the `hash_table_v2_add_entry` function, I TODO

### Performance

```shell
TODO how to run and results
```

TODO more results, speedup measurement, and analysis on v2

## Cleaning up

```shell
make clean
```
