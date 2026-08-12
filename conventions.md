# Conventions

Read this once. Everything here applies to every module, and none of it is repeated in them.

Nothing that only one market needs belongs here. A market requirement goes in a
[market profile](project/markets/README.md) instead.

## Base URL and versioning

```
https://api.opin-vn.{tld}/v1
```

**The standard fixes the path, not the host.** Everything from `/v1` onward is what this document
governs, and it is the same on every implementation. The host in front of it belongs to whoever
deployed the API, so an implementation publishes its own and issues it to you with your credentials.
Do not build the host above from this page.

`{tld}` marks the part that varies by deployment. The version is a path prefix, so `/v1` and `/v2`
are separate surfaces and a caller pinned to one is never moved by a change to the other.

The form above carries `opin-vn`, a market name that no longer describes what is served under this
standard. It is kept because it is already in service, and changing a base URL breaks every caller,
which makes it a major change however small the edit looks. An implementation that has not shipped
yet is under no obligation to reproduce the name. See [`SCOPE.md`](SCOPE.md) for when held changes
of this kind ship.

## Authentication

OAuth 2.0 bearer tokens. Two scopes, and they divide on whether a call changes anything:

| Scope | What it permits |
| :--- | :--- |
| `opin-vn.developer` | Read. Every `GET` |
| `opin-vn.admin` | Write. Every `POST`, `PUT` and action endpoint |

There is no third scope and no per-resource permission. If you need finer control than read against
write, it belongs in your own authorisation layer rather than in the standard's.

Every endpoint in every module names the scope it requires.

**The division is what the standard fixes. The literal string is not.** An implementation may
namespace these names to itself, and the one you send is the one issued to you with your
credentials. When a module page says an endpoint needs `admin`, read that as the write scope your
implementation issued, whatever it is called there.

## Content type

`application/json` on every request and response body.

## Errors

Errors are [RFC 7807](https://www.rfc-editor.org/rfc/rfc7807) problem documents. That means a JSON
body with a stable machine-readable `type`, a human-readable `title` and `detail`, and the HTTP
status repeated inside it, rather than a bare status code and a free-text string.

The point of using a declared error format is that a client can branch on `type` without parsing
prose, and prose can then be improved without breaking that client.

## Pagination

Collection `GET`s are cursor-paged. Send `?cursor=` and follow the `Link` header for the next page.

Cursor paging is used rather than offset paging because a cursor stays correct while other callers
are writing. With `skip` and `limit`, a record inserted during your walk shifts everything after it,
so you silently see a record twice or miss it entirely. A cursor points at a position in the data
rather than a count from the start.

`skip` and `limit` remain accepted, so existing callers keep working.

## Idempotency

Send an `Idempotency-Key` header on `POST` and `PUT`.

The key lets you retry a request without wondering whether the first attempt landed. A replayed key
returns the original response instead of creating a second record. This matters most where a
duplicate is expensive rather than untidy: issuing one policy twice, or paying one claim twice.

## Action endpoints

State transitions are `POST /resource/{id}:action`, with a colon:

```
POST /motorCoverage/{id}:cancel
POST /claim/{id}:settle
```

Endorsements, cancellations, renewals, settlements, reopenings and refunds all take this form. They
are not create, read, update or delete. Each one is a specific transition with its own rules about
when it is legal, and forcing them into `PUT` would hide that behind a field change.

Which transitions are legal is drawn in each module's lifecycle diagram, and **those diagrams are
normative**. A transition a diagram does not draw is not one an implementation may make.

## Extensions

An implementation may carry fields the standard does not define. Two rules make that safe.

**An extension never reuses a name the standard defines, and never changes what a defined field
means.** An extension is additive or it is a fork.

**A caller that receives a field it does not recognise ignores it rather than failing.** This is
required in both directions, and it is what lets the standard add optional fields without breaking
anyone.

## Field names on the wire

**The API governs anything that travels. The data model governs what a field means.**

Several inherited field names are misspelled. They stay misspelled, because they are already on the
wire in every existing implementation and correcting them would break working integrations to make a
document tidier. The data model pages show the corrected name so a reader knows what was meant, and
what you send is the original.

Each module's data model lists the affected names for that module. The corrections ship together in
one major version rather than one at a time. See [`SCOPE.md`](SCOPE.md).

## Sources

- [OPIN Data Standard v1.2.1](https://docs.google.com/spreadsheets/d/1Y0Gk_LpTvTNEfoDMdIxeD7juv3E8FKcbE3mHUJNV5JY)
  (XLSX), published by the Open Insurance Initiative
- [OPIN API Specification v1.0](https://github.com/The-Open-Insurance-Initiative/API-spec/blob/main/Open-Insurance-io-Open_Insurance_API-1.0-resolved.json)
  (resolved JSON)
- [Mermaid syntax reference](https://mermaid.js.org/intro/), for the diagrams throughout
