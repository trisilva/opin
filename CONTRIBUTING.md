# Contributing

Open an issue. That is the whole process at this stage, and it is deliberate: a track with one
published version and no external implementers does not need governance machinery yet, and
inventing some would suggest more formality than exists.

## In scope

- **A correction against OPIN source.** If an entity, field or enum in a track does not match the
  OPIN sheet it claims to trace to, that is a defect. Cite the sheet.
- **A gap in a track.** Something an implementation needs to interoperate that the track does not
  settle. The most useful version of this names the two implementations that would disagree.
- **An OPIN-side defect.** Add it to `upstream/opin-concerns.md`, or raise it here and it will be
  added. These are more valuable than anything else on this list, because they are the ones that
  compound if left alone.
- **A market with no track.** Say which one and what makes it different. Not a promise that one
  will be written.

## Out of scope

- **Vendor product behaviour.** Distribution mechanics, commission ledgers, workflow sub-states,
  operational service-level measurement. These sit above a country track, in whatever platform an
  implementer builds. Proposals to pull them in will be closed with a pointer to the root README.
- **Redefining OPIN.** A country track extends the standard beneath it and never contradicts it. If
  OPIN is wrong, the fix is upstream, and `upstream/opin-concerns.md` is how it gets there.
- **Silent renames of misspelled fields.** They are misspelled on the wire in every existing
  implementation. Reporting them upstream is the fix; renaming them here is a compatibility break
  wearing a tidy-up's clothes.

## On disagreement

Anything marked `[OPIN-VN]` in a track is Trisilva's reading of an open question, not a settled
fact. Those are the places where an issue is most welcome, and where a well-argued objection should
be expected to change the document.

## Licence of contributions

Contributions are accepted under MPL 2.0, matching the repository.
