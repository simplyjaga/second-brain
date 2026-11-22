# RESP

## What is it?

RESP (Redis Serialization Protocol) is Redis's wire [[Protocol]] — the language client and server speak over a raw TCP connection. It is deliberately tiny and fast to parse. It is *not* [[HTTP]] and *not* [[REST]] — it doesn't know what `GET` or `POST` mean in the HTTP sense.

## How does it work?

RESP prefixes every element with a type marker. A `SET key value` command on the wire looks like:

~~~text
*3\r\n$3\r\nSET\r\n$3\r\nkey\r\n$5\r\nvalue\r\n
~~~

- `*3` — an array of 3 elements follows
- `$3` — a bulk string of 3 bytes (`SET`), `$5` — 5 bytes (`value`)
- `\r\n` — delimiters between elements

Same pipe as HTTP, different language inside: HTTP and RESP are siblings — both **use** TCP, neither **is** TCP.

## The "HTTPS equivalent" for Redis

"HTTPS" is [[HTTP]] + [[TLS]]. TLS isn't married to HTTP — it's an independent encryption layer that can wrap **any** TCP connection, and RESP is the concrete example:

~~~text
HTTPS          =  HTTP + TLS + TCP
Secure Redis   =  RESP + TLS + TCP
~~~

With TLS enabled, the client opens TCP, does the TLS handshake (certificates, keys), then speaks plain RESP inside the encrypted tunnel. No HTTP anywhere. You wouldn't run RESP *inside* HTTP either — the headers would mean nothing to Redis (unless you deliberately put an HTTP proxy in front).

The connection string tells the client which mode:

| URL | Meaning | When |
|---|---|---|
| `redis://user:pass@host:6379` | Plain TCP, RESP unencrypted | localhost, isolated private network |
| `rediss://user:pass@host:6379` | TLS handshake first, RESP encrypted | production, cloud Redis |

These go to a client library (ioredis, redis-py, go-redis) in backend code — not a browser.

> [!question] Things that confused me
> - "If HTTPS isn't used, how is RESP secure over raw TCP?" — The "S" in HTTPS is TLS, which wraps any TCP connection. RESP + TLS is exactly what `rediss://` does.
> - "So when I call my Redis server I use `redis://`?" — Yes, or `rediss://` (double-s) when TLS is enabled.

## Related concepts

- [[Protocol]] — RESP's place in the stack
- [[TLS]] — the layer that makes `rediss://` possible
- [[HTTP]] — sibling protocol over the same TCP pipe
- [[Event Loop]] — how Redis serves many RESP conversations on one thread

## Collections

- [[Redis Internals]]

## Sources

- [[Redis Internals playlist]] (Arpit Bhayani)
- [Everything you need to know about REST](https://www.youtube.com/watch?v=uFGJVQvR59A&list=PLsdq-3Z1EPT36NJXTutvKcreetuHCr9a-&index=2) — the HTTP vs RESP layering discussion
