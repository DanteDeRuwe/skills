---
name: write-adr
argument-hint: '[decision summary]'
description: >-
  Record an Architecture Decision Record (ADR) documenting a technical decision, why it was made,
  and its consequences. Only use this skill when the user explicitly asks to write, create, or log
  an ADR (or "architecture decision record" / "decision log"). Do not trigger it automatically from
  general design or refactoring discussions.
---

# Write an ADR

Capture a technical decision as a short Architecture Decision Record. The value is in recording
**that** a decision was made and **why** — not in filling out sections. A single paragraph is a
perfectly good ADR.

## Where ADRs live

If the repository already keeps ADRs somewhere (look for an existing folder such as `docs/adr/`,
`doc/adr/`, `docs/decisions/` or `adr/`, or a path named in the README or contributing guide), use
that folder and follow its numbering and template. Otherwise write to `docs/adr/` at the repository
root, creating the folder if it doesn't exist.

The rest of this skill says `docs/adr/`; read it as whichever folder applies.

## Filename

`docs/adr/NNNN-kebab-case-summary.md`, where `NNNN` is a zero-padded 4-digit sequence number.

Pick the next number by looking at the highest existing `NNNN-*.md` in `docs/adr/` and adding one.
The first ADR is `0001`. Derive the kebab-case part from the decision, e.g.
`0007-use-outbox-pattern-for-event-publishing.md`.

## Author

Record who made the decision as `Author: Name`. You already know the current user from your
session context, so just use their name. Don't include an email address.

## Template

Keep it lean. Status and Date and Author on top, then prose. The two extra sections are optional —
add them only when they carry real information.

```markdown
# ADR NNNN: <short decision title>

- Status: Accepted
- Date: <YYYY-MM-DD>
- Author: <name>

<One or more paragraphs describing the context and the decision, and above all WHY it was made.
Name the specific problem and the specific choice. This alone is a complete ADR.>

## Considered options

- **<option>** — <why chosen or rejected>

## Consequences

- <what this makes easier, harder, or different going forward>
```

## Conventions

- `Status` is usually `Accepted`. Use `Proposed` for a decision still under discussion, or
  `Superseded by ADR NNNN` when a later ADR replaces it.
- Use today's date (`YYYY-MM-DD`).
- Write in plain, direct prose. Don't pad the sections to look complete — a one-paragraph ADR is fine.
- Never renumber or rewrite existing ADRs. To change a past decision, write a new ADR and mark the
  old one superseded.
