# two-reviewers

An [Agent Skill](https://agentskills.io) that reviews a design document, plan, spec, RFC or
code change with two subagents in turn, until both approve the same version. It is written for
any coding agent that loads skills and can start subagents (Claude Code, Codex, Antigravity and
others): it names no agent-specific tool.

- The **thorough** reviewer checks every claim against the real code and hunts
  inconsistencies, race conditions and gaps.
- The **pragmatic** reviewer cuts over-engineering and weighs what each piece costs to build
  and maintain.

They fail in opposite directions: one adds a mechanism for every edge case, the other cuts
protections a real user needs. The agent you ask coordinates: it checks the reviewers' facts,
applies each pass to the document or code, and hands each reviewer the other's changes and
reasons, so each undoes the other's excess.

## Install

Link or copy this directory into the place your agent loads skills from:

```sh
ln -s "$PWD" ~/.claude/skills/two-reviewers                  # Claude Code
ln -s "$PWD" /path/to/project/.agents/skills/two-reviewers   # Codex, Antigravity
```

For another agent, or for its per-user directory, see its documentation.

## Use

Ask in plain words:

```
Review designs/DESIGN_foo.md with two reviewers until they agree.
Review designs/DESIGN_foo.md with two reviewers and commit each version. Requirement: a long
command never blocks the chat.
Review this branch with two reviewers until they agree.
Review PR 123 with two reviewers, thorough and pragmatic.
```

Where your agent invokes a skill by name, pass the same as arguments. In Claude Code:

```
/two-reviewers designs/DESIGN_foo.md --commit requirement: a long command never blocks the chat
```

- `--commit`, or "commit each version", commits the starting version and then each applied
  pass, so the history shows what each reviewer changed. Without it the document or code is
  edited in place and nothing is committed.
- `requirement:` names a decision you will not drop. Both reviewers get it up front, so
  neither cuts it.

A code review edits your working tree, so it only starts when you ask for two reviewers, not on
a plain "review my code". It never pushes, and never comments on or approves a PR, unless you ask.

## What you get

The reviewed document or code, and a report: a table of versions with each reviewer's verdict,
what the review removed and what it fixed, the decisions taken without you, and the next command
if one is due. A code review also lists problems found outside the change, and, without
`--commit`, how to undo it.

The loop ends when both reviewers say `VERDICT: AGREE` about the same version; for code, its
test result must also be no worse than before the review. If the diffs are not shrinking after
four full rounds, it stops and brings the disagreement to you.

## Files

- `SKILL.md`: the coordinator's instructions (the loop, the handoff between reviewers, when to
  stop, the final report).
- `references/thorough-brief.md`, `references/pragmatic-brief.md`: the first prompt for each
  reviewer of a document.
- `references/thorough-code-brief.md`, `references/pragmatic-code-brief.md`: the same for code.
- `evals/evals.json`: four test prompts with their expected outcomes.
