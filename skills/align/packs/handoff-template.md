# align — handoff document template

Fill this in at the end of an align session (see `skills/align/SKILL.md` §3).
Present the completed document to the user for a final confirmation pass before
treating alignment as complete.

Copy everything below the `---` into your reply and replace the bracketed fields.
Do not leave a bracket unfilled — if a field doesn't apply, write "n/a" and say
why, rather than leaving it blank (a blank field reads as "I didn't check").

---

# Handoff: [task short name]

## Task (one sentence)
[The aligned restatement of the task. No hedging words ("maybe", "might", "if
possible"). If you can't write it in one confident sentence, alignment is not
done yet — go back to §2.]

## What "done" looks like (observable)
[Concrete, verifiable conditions a stranger could check. Prefer the form
"running `<command>` produces `<output>`" or "the rendered state shows
<description>". List multiple conditions if needed. Avoid "it works" — that is
not observable.]

- [condition 1]
- [condition 2]

## In scope
- [explicitly what gets done]

## Out of scope
- [what the user did NOT ask for, especially things you might be tempted to
  touch — incidental refactors, "while I'm here" cleanups, related bugs]

## Assumptions (recommended defaults, not confirmed decisions)
[From §2-C. Every line here is something you proceeded on as a best guess. Flag
them so they can be revisited without re-deriving the whole alignment.]

- [assumption] — recommended because [reason grounded in exploration]

## Relevant context (from exploration)
[From §1. Enough that the next agent doesn't re-explore. Actual file paths,
conventions, prior-art. If you used a subagent, distill its conclusion here.]

- **Codebase area**: [paths + roles]
- **Structure / conventions**: [how that area works]
- **Prior art**: [how similar work is already done here, if any]
- **Observable entry points**: [tests / CLI / endpoints that can verify "done"]

## Open risks
[Where intent is aligned but reality might disagree. Not ambiguities — those
should be zero. These are the places where the aligned plan could still break
on contact with the code at execution time.]

- [risk] — [what could go wrong, and what to watch for]

## Confirmed by user
[Leave as a checkbox. Mark it only after the user has reviewed the whole
document and either agreed or corrected.]

- [ ] User confirmed this handoff
