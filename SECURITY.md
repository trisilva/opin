# Security

## What this repository is

A specification. There is no running service here, no deployed endpoint, and no code that executes.
The base URL in the API design is a convention for implementers, not an address that answers.

So a vulnerability in this repository means a defect in the standard that would cause a correct
implementation to be insecure. That is a real category and it is worth reporting.

## What counts

- An authentication or authorisation pattern in the API design that is unsafe as specified.
- A field or flow that would require an implementation to transmit or store data in a way that
  breaches a market's data protection law.
- An idempotency, pagination or lifecycle rule with a race or replay consequence.
- A worked example that would teach an implementer an unsafe pattern.

## What does not count here

A vulnerability in someone's implementation of the standard. Report that to whoever runs it. We
cannot reach it and publishing it here does not help them.

## How to report

Use GitHub private vulnerability reporting on this repository. If that is unavailable, open a
regular issue describing the class of problem without the exploitable detail, and ask for a private
channel.

There is no bounty and no service level attached to this. What there is: a reply, and the reasoning
for whatever is decided, the same as any other change.

## Disclosure

A standards defect is not an active exploit against a live system, so the usual embargo logic mostly
does not apply. The default is to work in the open. Where a defect would expose live
implementations before they can update, the fix is prepared privately and published together with
the advisory.
