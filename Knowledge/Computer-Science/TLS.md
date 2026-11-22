# TLS

## What is it?

TLS (Transport Layer Security, formerly SSL) is an encryption layer that wraps **any** TCP connection — it sits between TCP and whatever application [[Protocol]] runs inside. It is not a web-only thing.

## How does it work?

1. Client opens a TCP connection.
2. **TLS handshake** — server proves identity with certificates, both sides agree on encryption keys.
3. An encrypted tunnel now exists; the application protocol inside it travels unchanged, just unreadable to outsiders.

## Important details

The pattern repeats everywhere, which is why "S" suffixes appear in scheme names:

| Stack | Connection scheme |
|---|---|
| HTTP + TLS + TCP | `https://` |
| [[RESP]] + TLS + TCP (Redis) | `rediss://` |
| FTP + TLS | `ftps://` |

So HTTPS = "HTTP inside TLS," not a separate protocol — see [[HTTP]] for how the two relate.

## Related concepts

- [[HTTP]] — what HTTPS is made of
- [[RESP]] — a non-HTTP protocol secured by the same layer
- [[Protocol]] — TLS's place in the stack

## Questions / Things to revisit

- TLS handshake internals — certificates, key exchange, TLS 1.3 vs 1.2
