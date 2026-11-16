# Redis Internals

## What I am learning

How Redis works internally — starting from how one server handles many clients using an event loop.

## Scope

- Event-driven networking: blocking → non-blocking → event loop
- How real servers structure their concurrency (nginx, workers, multiple cores)

## Knowledge concepts

- [[Event Loop]]

## Learning resources

- [[Redis Internals playlist]] — primary source (Arpit Bhayani)
- [[From Blocking Sockets to Event Loops]] — my own write-up consolidating the networking part

## Understanding / Things to revisit

- How the rest of Redis (dict, expiry, persistence) builds on top of the event loop
