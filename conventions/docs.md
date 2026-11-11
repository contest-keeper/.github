# Documentation

## READMEs

Every repository README has, in this order and kept short:

1. One or two sentences: what it is.
2. How to build and run it.
3. How to test it.
4. Links: conventions, contributing, license.

Crates and packages with their own folder get a short README too.

## Comments

- Comment sparingly and only where it really helps: explain why, not what. Skip comments that repeat the code.
  Link the plan step or issue when a choice needs background.
- `TODO(#123): ...` always with an issue number; no loose TODOs.
- Delete commented-out code; Git remembers it.

## Decision records

Choices that are hard to undo (language, protocol format, crypto, storage) get a decision record in
`ck-website/decisions/NNN-short-title.md`: context, decision, consequences. Never edit an accepted
record; write a new one that replaces it.

## Writing style

Plain, short sentences. No marketing words, no emoji. Write for someone new to the project.
