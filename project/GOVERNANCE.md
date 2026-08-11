# Governance

This document says who decides, how, and what a contributor can rely on. It describes the project
as it is today, not as it is intended to become.

## Where the project stands

Trisilva does the editorial work and holds the only editor seats. There are no other maintainers
yet and no external implementers on record. Saying otherwise would be more comfortable and would
not be true.

That is a real limitation and it is worth stating what it means for a reader. A standard maintained
by one organisation carries that organisation's blind spots. The protection against it is not a
promise, it is the record: every decision below leaves a written reason, so a reader can check the
reasoning rather than trust the maintainer.

## Roles

**Editors** decide what enters the standard.

| Name | Affiliation | Scope |
| :--- | :--- | :--- |
| Vu Tung Lam | Trisilva | The standard, and the Vietnam profile |

That is the whole list, and the limitation is described below rather than left to be inferred from
the length of the table.

**Profile owners** are accountable for what a profile claims about its market's regulation.

| Profile | Accountable |
| :--- | :--- |
| `vn` | Unassigned |

**Contributors** open issues and proposals. No agreement or membership is required.

**Implementers** build against the standard. An implementer who reports a defect that changes the
standard is the most useful contributor there is, and is listed as such in the changelog.

## How a change is decided

1. **An issue.** Anything: a defect, a gap, a question, a market with no profile.
2. **A proposal**, for anything that changes the standard. What breaks, who it affects, and what an
   existing implementation has to do about it. Small corrections skip this.
3. **A decision, with the reasoning written down.** Every accepted change records why, against the
   specific defect or gap it closes. Every rejected proposal records why, in the issue.
4. **A version.** See [`VERSIONING.md`](VERSIONING.md).

Nothing enters the standard without a written reason. That is the one procedural rule here, and it
is the rule that lets someone audit a single-maintainer project.

## When an editor disagrees with you

An editor can decline a proposal. What an editor cannot do is decline it silently or without a
reason, and a decision recorded as a preference rather than an argument is a defect in this process
that you should raise.

Anything marked `[added]` or `[OPIN concern]`, or otherwise flagged as a reading rather than a
settled fact, is explicitly open. Those are the places where this project made a call rather than
inherited one, and a well-argued objection to any of them should be expected to change the document.

## How editorship opens

Two independent implementers, or one sustained outside contributor, and editorship opens. This is
written here so it can be held against us rather than as an aspiration.

The reason it is not open already is that there is nobody to open it to. The reason it is written
down is that a standard governed by one company is worth less than one governed by several, and
saying when that changes is more useful than saying it will.

## Conflict of interest

Trisilva sells insurance operations software built on this standard. That is a real conflict and
the mitigation is a scope line rather than a declaration.

Nothing that only makes sense because of a vendor's product enters the standard. Distribution
mechanics, commission ledgers, workflow sub-states and operational service-level measurement sit
above it. This has bitten already: material previously filed as the Vietnam profile was base-standard
work, and material that was genuinely product behaviour was kept out of the profile at v0.2. The line
is enforced by asking one question of every proposal, including our own. Would a market with no
Trisilva software in it need this? If the answer is no, it does not go in.

## What this project does not have

No foundation, no legal entity, no membership, no funding, no trademark of its own, and no
ratification process. A future version may need some of these. None of them exist today.
