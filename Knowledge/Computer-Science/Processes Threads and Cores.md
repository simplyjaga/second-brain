# Processes, Threads, and Cores

## What is it?

A **process** is a running program with its own memory space. **Threads** are independent units of execution *inside* a process — they share the process's memory and resources. **Cores** are the physical CPUs that actually execute threads.

## Why is it needed?

Triggering question: *can two threads of one process run in parallel on two different cores?* **Yes.**

- The OS scheduler treats each thread as an independent unit — 2 threads on a 3-core machine can run truly in parallel, on 2 different cores.
- Threads don't have to share a core; the scheduler distributes them across whatever is free.
- Different cores ≠ different processes: the threads still share one memory space and one parent process.

## How does it work?

- **Concurrency** ≠ **parallelism**: one core can interleave many threads (concurrent but not parallel); true parallelism needs multiple cores.
- One thread, one flow of control — which is why a single-threaded [[Event Loop]] can only use one core. Real servers (nginx, Redis) scale to multiple cores by running multiple event-loop processes or worker threads.

## Important details

- Shared memory between threads is a feature (cheap communication) and a hazard (data races → need locks/atomics).
- Threads are cheaper to create than processes, but process isolation is stronger.

## Related concepts

- [[Event Loop]]

## Collections

- [[Redis Internals]]

## Sources

## Questions / Things to revisit

- GIL in Python: threads in CPython can't run Python bytecode in parallel — relevant to why the blog's server examples use processes/workers.
