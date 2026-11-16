# Event Loop

## What is it?

A single-threaded loop that asks the OS "which sockets are ready?" and only then does I/O on those sockets — so one thread can serve many connected clients instead of being stuck on one.

## Why is it needed?

Triggering question: *can one idle client stop a server that has hundreds of clients?* With a naive sequential server — yes.

- A TCP server calls `accept()` to get a connected socket per client, then `recv()` to read from it.
- `recv()` **blocks** by default: the thread waits inside the read until that client sends data.
- Meanwhile the OS happily accepts other connections and buffers their data — but the application never gets to `accept()` them. One idle client freezes everyone.

## How does it work?

Two pieces make it work:

**1. Non-blocking sockets** — `setblocking(False)` makes `accept()`/`recv()` return immediately instead of waiting. Necessary but not sufficient alone: polling every socket in a loop wastes CPU.

**2. Readiness notification (epoll)** — instead of polling, ask the OS which sockets are ready *now*:
- `epoll_create1()` → create the watched set; `epoll_ctl()` → register/remove sockets and events; `epoll_wait()` → sleep until something is ready.
- Other OSes have equivalents (kqueue on BSD). Python's `selectors` wraps this; `DefaultSelector` uses epoll on Linux.

Then the loop:

1. Register the listening socket (non-blocking) for read-readiness. "Readable" on it = a connection waits in the queue → `accept()` and register that client socket.
2. Client socket readable → `recv()` a bounded chunk, buffer it, handle the request — a readable client can also mean the peer closed, which is how disconnects are detected.
3. Back to `select()`; newly ready sockets are handled next turn.

## Important details

- **Readiness ≠ complete message.** TCP is a byte stream: a "readable" event may be only part of a request. The app must buffer and detect message boundaries.
- **Fairness is the app's job.** Cap work per client per turn (e.g. read ≤ 1024 bytes) so one chatty client can't starve the loop.
- epoll is **level-triggered** by default — unread bytes keep the socket reported-ready, no need to drain everything per callback.
- **Slow or CPU-heavy work still blocks the whole loop.** Real servers offload such work to worker threads/processes, and run multiple loops or processes to use multiple CPU cores (this is how nginx, Redis, Node.js scale) — see [[Processes Threads and Cores]].

## Web server vs application server

- A **web server** like nginx is itself an event-loop program at the edge — it terminates client connections and can proxy requests onward.
- The **application server** behind it (running your code) may use threads for CPU/slow work the event loop shouldn't absorb.
- The event loop shines at the I/O boundary; worker pools handle the compute.

## Related concepts

## Collections

- [[Redis Internals]]

## Sources

- [[From Blocking Sockets to Event Loops]] (my blog post — has the full step-by-step Python `selectors` example)
- [[Redis Internals playlist]] (Arpit Bhayani)

## Questions / Things to revisit

- Edge-triggered vs level-triggered epoll — when to use which?
- How Redis's event loop ties into the rest of the engine (commands, persistence).
