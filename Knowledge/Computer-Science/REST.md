# REST

## What is it?

REST (Representational State Transfer) is a **specification** — an architectural style for designing APIs, introduced in Roy Fielding's 2000 PhD dissertation. It is *not* a [[Protocol]]: it enforces no wire format, no storage engine, no payload type. Its one big idea: **everything is a resource** (a student, a book, an order), and client and server exchange *representations* of those resources.

## Why is it needed?

Before REST, most web APIs were remote procedure calls tunneled through [[HTTP]] — the action lived in the URL or body (`/getUser`, `/updateStudent`), `POST` was used for everything, and SOAP services returned `200 OK` with a `<Fault>` inside the XML. REST's contribution: *use HTTP the way it was meant to be used.*

## How does it work?

- **URL identifies the resource, method names the action.** `DELETE /users/1` reads literally as "for user identified by 1 — delete it."
- **Multiplexing on one URL** — `/students/1` accepts `GET` (read), `PUT`/`PATCH` (update), `DELETE` (remove). Same URL, different method, different meaning.
- **Litmus test:** `POST /getStudent` with the ID in the body is a working HTTP endpoint but is *not* REST — the URL names an action instead of a resource.
- **Format-agnostic** — `Content-Type`/`Accept` headers negotiate the representation: JSON, XML, CSV, Protobuf. JSON is only the most popular choice because it's easy to read, not because it's compact.

## Important details

- **"Representational state transfer" is literal:** a resource has an *internal representation* (DB rows, documents) and an *external representation* (the JSON you serve). REST only specifies the second. Store however you want; serve however the client asks.
- **REST doesn't even require HTTP** — in principle it could sit on another transport. HTTP just fits it beautifully (methods, URLs, status codes, content negotiation already existed), which is why "HTTP API" and "REST API" feel synonymous. They aren't.
- **Alternatives worth exploring separately:** GraphQL (client queries exactly the fields it needs), gRPC (action-oriented RPC style with binary Protobuf over HTTP/2 — a direct REST alternative), SOAP (the pre-REST XML style).

> [!question] Things that confused me
> - "Is HTTPS coupled with REST?" — No. HTTPS is transport security ([[TLS]]); REST is an API design idea. REST usually rides HTTPS in production, but neither requires the other.
> - "Does REST have to use JSON?" — No. Any format `Content-Type` can name.
> - "Are GET/POST/DELETE part of REST?" — No, they're built into [[HTTP]]. REST only defines *how to use* them.

## Related concepts

- [[HTTP]] — the protocol REST is almost always implemented on
- [[Protocol]] — the layering view that separates REST from HTTP/JSON/TLS
- [[RESP]] — a protocol that is *not* REST at all

## Projects / Journeys

- [[Redis Internals]]

## Sources

- [Roy T. Fielding's dissertation — Chapter 5 introduces REST (2000)](https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
- [Everything you need to know about REST — Arpit Bhayani](https://www.youtube.com/watch?v=uFGJVQvR59A&list=PLsdq-3Z1EPT36NJXTutvKcreetuHCr9a-&index=2)

## Questions / Things to revisit

- gRPC in depth — stubs, Protobuf, HTTP/2 streaming (separate note later)
- What makes an API "RESTful" vs merely "uses HTTP" — Fielding's actual constraints (HATEOAS etc.)
