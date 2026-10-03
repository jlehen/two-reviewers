---
name: two-reviewers
description: Review a design document, plan, spec or RFC with two subagents in turn — a thorough reviewer that checks every claim against the real code and hunts inconsistencies, race conditions and gaps, and a pragmatic senior reviewer that cuts over-engineering and weighs maintenance cost — applying each pass to the document and alternating until both agree on the same version. Use this whenever the user asks for a design review, says "two reviewers", "thorough and pragmatic", "review until they agree", "technical vs pragmatic", or wants a DESIGN_*.md, plan or proposal checked before it is implemented. Pass `--commit` to commit each version.
argument-hint: <path-to-document> [--commit] [requirement: <one the author will not drop>]
---

You coordinate two reviewers over one document until both approve the same version. You are
not a mailbox between them: you check their facts, apply their findings, overrule them where
they are wrong, and carry each one's reasoning to the other.

The two reviewers fail in opposite directions, which is the point. The thorough one adds a
mechanism for every edge case it finds. The pragmatic one cuts protections a real user needs.
Alternating them, with the diff of the previous pass in hand, lets each undo the other's
excess. The loop ends when neither has anything left to add or cut.

## Before the first pass

1. Read the document yourself, fully. You will be judging findings against it all day.
2. Write down the author's requirements: the decisions the user has already made and will not
   drop, whether they are in the document, in the conversation, or in memory. Both reviewers
   get them up front. In the run that produced this skill the pragmatic reviewer cut a whole
   build step because nobody told it the user required it, and a round trip was spent putting
   it back. If you cannot name the requirements, ask the user before spawning anyone.
3. Find what the reviewers must read: the project's instruction files (vocabulary, design
   rules, style), every document the design cites, and the code behind each touch point the
   design names. List them by path. A reviewer told "check the code" finds less than one
   handed the files.
4. With `--commit`: commit the document as it stands, message "<file>: <title>" plus "This is
   the version before review." Each applied pass then gets its own commit, so the history
   shows what each reviewer changed. Without `--commit`, edit in place and never commit; most
   projects reserve commits for the user.

## The loop

```
thorough pass → apply → pragmatic pass → apply → thorough pass → … until both AGREE on
the same version
```

Spawn each reviewer once with the Agent tool (`general-purpose`, not a fork: a fresh agent
reads the code without your conclusions in its head). Keep both agent ids. Every later pass
goes to the same agent through SendMessage, so it keeps what it read and can check that its
own findings were really closed rather than merely mentioned.

The first brief for each reviewer is in `references/thorough-brief.md` and
`references/pragmatic-brief.md`. Fill the placeholders; do not trim the sections. Each brief
carries the same spine: what to review, what to read first, what to look for, the project rule
that binds its suggestions, the report format, and the verdict line.

### Applying a pass

Before editing, spot-check the code facts the fixes rest on (a `grep`, a `sed -n` on the
lines the reviewer cited). Reviewers cite confidently and are usually right; the one time they
are not, you would otherwise write their error into the design.

Apply findings as design text, in the document's own voice and wrapping. Where a reviewer's
wording is wrong or its fix is larger than the problem, depart from it and tell the next
reviewer what you changed and why, in the handoff. Both reviewers in the original run
accepted such departures when given the reason; neither would have accepted a silent one.

With `--commit`, commit after each pass: "<file>: Apply the <thorough|pragmatic> review,
pass <n>", then one short paragraph per substantive change, in plain words.

### The handoff

Every message to a reviewer after its first pass has this shape:

```
<Nth> pass wanted on <file>: <one line on why it changed>. Read only, no edits, same report
format and verdict line.

History: <sha you reviewed> → <sha with your pass applied> → <sha current>. `git diff a b`
shows each step. Review the whole current file, not only the diffs.

What happened to your findings: <numbered; each one "applied", "narrowed: <how>",
"reverted: <why>" or "departed: <why>">.

What the other reviewer changed beyond that: <bullets, with its reasons and its evidence>.

What I want from this pass: <numbered, specific: claims to check, races to walk, cuts to
judge>.

End with VERDICT: AGREE or VERDICT: NOT YET.
```

"Review the whole file" is not politeness. In the original run the thorough reviewer missed a
data-losing hole in pass 1 and found it in pass 2 only because it reread everything; a
diff-only pass would have signed off on it.

### When the reviewers contradict each other

Do not pick a side and do not average. Send one of them both arguments with both pieces of
evidence, state that both failures are real, and ask for the cheapest design that has
neither. Offer two or three candidate shapes, say "to judge, not to accept", and ask for the
cost of each from the code. A reviewer that gets the other's evidence usually finds a third
shape; one that gets only "the other reviewer disagrees" digs in.

The same message is where you deliver a requirement a reviewer violated: "Finding X meets a
requirement you were not told about: <it>. Judge again with that as a given." Expect it to
change position and say so; that is honest review working, not a reviewer caving.

## Stopping

The loop ends when both reviewers have said AGREE about the **same commit or file state**. A
verdict on an earlier version does not count, so after the last substantive pass, send the
other reviewer the final diff to confirm, even when it is small.

Steer the last passes toward exact text. When a reviewer reports only MINOR findings, ask it
to give the replacement sentences so no further pass is needed, and let the other reviewer
confirm those lines in one short message. Severity discipline for the final pass: BLOCKER
and MAJOR only for what cannot be built as written, loses or corrupts the user's data, or is a
wrong claim about the code that a normal use falls into. Wording is MINOR.

Convergence shows in the diffs shrinking pass over pass (the original run went +112/−53,
+94/−105, +51/−46, +11/−7, +19/−14). If after four full rounds the diffs are not shrinking,
stop, and bring the live disagreement to the user with both reviewers' last arguments; it is
a decision, not a review finding.

## Report to the user

End with, in this order:

1. A table of versions: commit, which pass produced it, lines added and removed, which
   reviewer gave which verdict on it.
2. What the review removed from the first version, each item told as a short story with a
   real day and clock times, so the cost of the cut is visible.
3. What the review fixed, the same way.
4. **Decisions taken without the user**: every choice a reviewer made that the user had been
   holding, or that touches a requirement, so they can overrule it before the build.
5. The next command, if one is due (rebase, land, squash).

Write the report in plain language. Say what a codebase term is the first time you use it.
The user reads this without the document open.
