# Protocol

## What is it?

A protocol is a shared set of rules two parties follow to communicate — what messages look like, in what order they're sent, and what they mean. Without a protocol, bytes on the wire are just noise.

Protocols are **layered**. Each layer makes one decision and doesn't care about the others:

| Layer | Examples | What it decides |
|---|---|---|
| Transport | TCP, UDP | Bytes arrive reliably and in order (TCP) |
| Encryption | [[TLS]] | The connection is private and authenticated |
| Application protocol | [[HTTP]], [[RESP]], gRPC | The rules of the conversation (verbs, headers, syntax) |
| Data format | JSON, XML, Protobuf | How the payload bytes are structured |
| API design idea | [[REST]], GraphQL | Conventions for how the API is organized |

Two clarifications that untangle common confusion:

- **HTTP and RESP are siblings, not relatives of TCP.** Neither is "a type of TCP" — both *use* TCP as their delivery pipe, and both can be wrapped in TLS independently.
- **REST and JSON are not protocols.** REST is a *specification* (an idea for designing APIs, see [[REST]]); JSON is just a data format that travels inside a body. Neither defines a wire protocol.

## Why is it needed?

Machines can't infer meaning. The protocol is what lets a client send `GET /users/1` or `*3\r\n$3\r\nSET...` and have the server know exactly what was asked and how to reply.

## Important details

- **Any TCP protocol can be encrypted the same way.** HTTPS is just HTTP + TLS; `rediss://` is RESP + TLS. The "S" is TLS doing its job, not something HTTP-specific — see [[TLS]].
- **Layering is what lets you mix and match:** you can swap the data format (JSON → Protobuf), the API style (REST → gRPC), or add encryption (TCP → TCP + TLS) without the other layers changing.

> [!question] Things that confused me
> - "Is HTTPS coupled with REST? And does REST internally have to use JSON?" — No on both. They're at different layers: HTTPS is transport+security, REST is an API design idea, JSON is just one possible body format.
> - "So HTTP is the same line as RESP — both are types of TCP?" — Right instinct, wrong word. They're both *application protocols* that sit **on top of** TCP. TCP itself has no idea which language is inside.

## Related concepts

- [[HTTP]] · [[RESP]] · [[TLS]] · [[REST]]
- [[Event Loop]] — how servers handle many protocol conversations at once

## Collections

- [[Redis Internals]]

## Sources

- [Everything you need to know about REST — Arpit Bhayani](https://www.youtube.com/watch?v=uFGJVQvR59A&list=PLsdq-3Z1EPT36NJXTutvKcreetuHCr9a-&index=2)
