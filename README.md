# two-reviewers

A Claude Code skill that reviews a design document, plan, spec or RFC with two subagents in
turn, until both approve the same version.

- The **thorough** reviewer checks every claim against the real code and hunts
  inconsistencies, race conditions and gaps.
- The **pragmatic** reviewer cuts over-engineering and weighs what each piece costs to build
  and maintain.

They fail in opposite directions: one adds a mechanism for every edge case, the other cuts
protections a real user needs. Claude coordinates: it checks the reviewers' facts, applies
each pass to the document, and hands each reviewer the other's changes and reasons, so each
undoes the other's excess.

## Install

```sh
ln -s "$PWD" ~/.claude/skills/two-reviewers
```

## Use

```
/two-reviewers designs/DESIGN_foo.md
/two-reviewers designs/DESIGN_foo.md --commit requirement: a long command never blocks the chat
```

Or ask in plain words: "review this plan with two reviewers until they agree".

- `--commit` commits the starting version and then each applied pass, so the history shows
  what each reviewer changed. Without it the document is edited in place and nothing is
  committed.
- `requirement:` names a decision you will not drop. Both reviewers get it up front, so
  neither cuts it.

## What you get

The reviewed document, and a report: a table of versions with each reviewer's verdict, what
the review removed and what it fixed, the decisions taken without you, and the next command
if one is due.

The loop ends when both reviewers say `VERDICT: AGREE` about the same version. If the diffs
are not shrinking after four full rounds, it stops and brings the disagreement to you.

## Files

- `SKILL.md`: the coordinator's instructions (the loop, the handoff between reviewers, when to
  stop, the final report).
- `references/thorough-brief.md`, `references/pragmatic-brief.md`: the first prompt for each
  reviewer.
- `evals/evals.json`: two test prompts with their expected outcomes.
