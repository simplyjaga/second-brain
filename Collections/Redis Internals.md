# Redis Internals

## What I am learning

How Redis works internally — starting from how one server handles many clients using an event loop.

## Scope

- Event-driven networking: blocking → non-blocking → event loop
- How real servers structure their concurrency (nginx, workers, multiple cores)
- Protocols and API styles: HTTP vs REST vs RESP, and where TLS fits

## Knowledge concepts

- [[Event Loop]]
- [[Processes Threads and Cores]]
- [[Trie]]
- [[Radix Trie]]
- [[Redis Persistence]]
- [[Protocol]]
- [[HTTP]]
- [[REST]]
- [[RESP]]
- [[TLS]]

## Learning resources

- [[Redis Internals playlist]] — primary source (Arpit Bhayani)
- [[From Blocking Sockets to Event Loops]] — my own write-up consolidating the networking part

## Understanding / Things to revisit

- How the rest of Redis (dict, expiry, persistence) builds on top of the event loop
