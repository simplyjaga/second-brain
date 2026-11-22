# HTTP

## What is it?

HTTP is the web's application [[Protocol]] — a request/response conversation between a client and a server over TCP. The client sends a **method + URL + headers (+ optional body)**; the server replies with a **status code + headers + body**.

## Why is it needed?

It gives every client and server a shared language for transferring resources — so a browser, `curl`, or a backend service can all talk to any web server without a custom protocol each time.

## How does it work?

An HTTP request has three pieces that each carry one kind of meaning:

~~~http
DELETE /users/1 HTTP/1.1
Host: example.com
Content-Type: application/json
~~~

- **Method (verb)** — the action: `GET`, `POST`, `PUT`, `DELETE`, `PATCH`, `HEAD`, `OPTIONS`. Defined by HTTP itself, *not* by [[REST]] — REST just said to use them properly.
- **URL** — identifies what the request is about.
- **Headers** — metadata; `Content-Type` declares the body's format, `Accept` negotiates the response format. This is how one API can serve JSON, XML, or CSV.

The response mirrors this: status line (`404 Not Found`), headers, body.

## Important details

- **Stateless** — each request carries everything the server needs; no memory of previous requests.
- **Locked to TCP** — HTTP only runs on TCP. (HTTP/3 is the exception — it runs on QUIC/UDP.)
- **HTTP/2** is a binary version of the same protocol — multiplexing, streaming; gRPC builds on it.

## HTTPS = HTTP + TLS

HTTPS is *not* a different protocol — it's plain HTTP inside a [[TLS]]-encrypted TCP connection. The "S" belongs to TLS, not to HTTP:

~~~text
HTTPS  =  HTTP + TLS + TCP
~~~

The same trick works for any TCP protocol — e.g. `rediss://` is [[RESP]] + TLS. So "HTTPS" being used for websites says nothing about what other traffic TLS can protect.

## Related concepts

- [[Protocol]] — where HTTP sits in the stack
- [[REST]] — the API design idea usually implemented on HTTP
- [[RESP]] — a sibling application protocol, also over TCP
- [[TLS]] — the encryption layer that turns HTTP into HTTPS

## Collections

- [[Redis Internals]]

## Sources

- [Everything you need to know about REST — Arpit Bhayani](https://www.youtube.com/watch?v=uFGJVQvR59A&list=PLsdq-3Z1EPT36NJXTutvKcreetuHCr9a-&index=2)

## Questions / Things to revisit

- HTTP/2 vs HTTP/1.1 internals — framing, streams, how gRPC uses it
