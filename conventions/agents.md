# House rules for people and agents

These rules apply to everyone who changes a Contest Keeper repository: people and coding agents alike.
Each repository copies the block below into its `AGENTS.md`, between the two marker lines, and adds a
"This repository" section of its own. `CLAUDE.md` in each repository contains only `@AGENTS.md`.

The `house-rules` workflow compares the block in a repository with this one by sha256. **Never edit the
copied block in a repository.** Change it here, in a PR to the `.github` repository, then copy it over.
The hash covers everything between the markers, without the marker lines themselves.

<!-- ck-house-rules:start -->
## House rules

1. **A person is accountable.** Every change has a human owner who reads the diff and answers for it.
   An agent proposes; it does not decide on its own what the product should do. Decisions live in the
   decision records, not in a change.
2. **Check, then claim.** Look at the code, the repository state and the open PRs before you say a thing
   exists, is missing or is broken. Say where you looked.
3. **Verify, don't guess.** Run the code or the check that answers the question. A name, flag, path or
   API you have not seen in this repository is a guess; open the file.
4. **Narrow, tested fixes.** One change, one reason. A fix comes with a test that fails without it.
   Moves and behaviour changes go in separate PRs. Don't fix unrelated problems in passing; list them.
5. **Review the diff before you ask for review.** Read your own diff line by line. Remove debug output,
   dead code and unrelated edits. Run the formatter, the linter and the tests the repository lists.
6. **Report only what ran.** Say which commands you ran and what they printed. A check you did not run,
   or could not run, is reported as not run. Never write "should work" for something you could test.
7. **No pushes to `main` or `develop`.** Work on a branch, open a PR, let the checks pass, squash-merge.
   Never force-push a shared branch, skip a hook or bypass a required check.
8. **Secrets stay out.** Never store, log, print or send passwords, cookies, session ids, tokens or the
   Wi-Fi password. Fixtures made from real traffic are stripped first. The user types credentials.
9. **Ask before the risky step.** Ask before deleting data, branches or folders, force-pushing, deploying,
   or anything that leaves the machine.
10. **Guards say why.** A check that rejects, blocks or fails closed carries a short comment saying what it
    prevents, so nobody removes it as clutter. Other comments explain why, never what.
11. **Tests earn their place.** Keep a test only if it protects a rule, a realistic edit would break it, and
    no other test would catch that edit. Tests never pass silently: a missing resource is a printed skip,
    and a failure when its `*_REQUIRED` variable is set. Code that reads outside input gets a hostile-input
    sweep. See `conventions/testing.md`.
12. **Commits and PRs.** Branch from `develop`; title is `type: lowercase imperative subject`; no
    `Co-Authored-By`, "Generated with" or other tool attribution in commits or PR text.
<!-- ck-house-rules:end -->

## What is left out on purpose

Other projects' house rules include a clean-room rule for GPL code, translation rules and desktop
accessibility rules. They are not here: Contest Keeper ships GPL programs (see the decision records),
translations come later, and the Forge window follows its own plan.

## Computing the hash

```sh
sed -n '/<!-- ck-house-rules:start -->/,/<!-- ck-house-rules:end -->/p' conventions/agents.md \
  | sed '1d;$d' | sha256sum
```

The `house-rules` workflow does exactly this on `conventions/agents.md` from `main` of `.github` and on
the repository's `AGENTS.md`, and fails when the two differ.
