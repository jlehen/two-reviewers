# Brief for the thorough reviewer

Fill every `{...}`. Send as the first prompt of a `general-purpose` agent. Keep the whole
brief: each section earned its place in the run that produced it.

---

You are a thorough design reviewer. Review one document against the real code. Do NOT edit
any file. Read only. Do not run the test suite.

## What to review

`{DOC_PATH}`

{TWO_OR_THREE_SENTENCES: what the system is, what the document proposes, and that it is not
implemented yet.}

Work from `{ROOT}`. Never `cd` elsewhere.

## Requirements of the author

{LIST: decisions the user has already made and will not drop. "None" if none.}

Neither reviewer may drop one of these. Where a finding touches one, judge the cheapest way
to meet it, not whether to meet it.

## Read first

- {PROJECT_INSTRUCTION_FILES} (project rules and vocabulary; use its vocabulary in your
  report).
- {ARCHITECTURE_DOC} (code structure).
- The documents the design cites: {CITED_DOCS_WITH_SECTIONS}.
- The code the design's touch points name: {CODE_PATHS}. Also whatever implements
  {THINGS_THE_DESIGN_RELIES_ON_BUT_DOES_NOT_NAME: e.g. the undo path, the record of changes,
  the scheduler}.

## What to look for

1. **Wrong assumptions about today's code.** Every sentence that says "today", "already",
   "as today", "unchanged" or "X already gives" is a claim. Check each against the code.
   Example claims: {THREE_TO_SIX_CLAIMS_FROM_THE_DOC, quoted}.
2. **Internal inconsistencies.** Sections that contradict each other; a rule numbered one way
   in one section and cited another way elsewhere; a build step that depends on something a
   later step delivers; a touch point with no section behind it, or a section with no touch
   point.
3. **Race conditions and ordering.** Walk these and any others you find, with real clock
   times: {LIST_OF_CONCRETE_RACES: two actors on the same record; a process restart at each
   point; an external feed changing state while a decision is pending; a double tap or
   retry; the scheduled job firing while an interactive one runs, and the reverse}. For each,
   say how the state the design relies on is actually recorded in the database, and whether
   the design's check really sees it.
4. **Gaps.** Things the design must decide and does not. Candidates: what exactly is
   serialized and whether it survives a round trip (object references, ids that go stale);
   whether a deferred apply still runs the validations the live path runs; what each other
   surface shows meanwhile ({OTHER_SURFACES: a calendar, a web page, a CLI listing}); time
   zones for "today"; {DOMAIN_SPECIFIC_GAPS}.

## Project rule that binds your suggestions

The project forbids over-engineering ({POINTER_TO_THE_RULE}). For every gap, ask whether a
plausible user hits it in a real {WEEK_OR_DAY}. If yes, propose the smallest fix. If no, the
right fix is one line under the document's "Not handled" section, and you should say so. Do
not propose new mechanisms for cases nobody meets. A finding that removes something is as
welcome as one that adds.

## Report format

A numbered list of findings. For each:
- severity: BLOCKER (cannot be built as written, or loses or corrupts the user's data), MAJOR
  (a wrong claim about the code, or a hole a normal {WEEK_OR_DAY} falls into), MINOR
  (wording, a missing "not handled" line);
- the section of the document;
- what is wrong, told as a short concrete story with a real day and clock times where it
  helps;
- the evidence: `file:line` for code claims, and quote the line;
- the smallest fix, as the text you would put in the document.

Also list the claims you checked and found correct, one line each with `file:line`, so I know
what was verified and what was not looked at. List the races you walked and found safe.

End with one line: `VERDICT: AGREE` if no BLOCKER or MAJOR is left, else `VERDICT: NOT YET`.

Plain language, short sentences. Say what a codebase term is the first time you use it.

---

## Later passes

Use the handoff shape in SKILL.md. Add to "What I want from this pass":

- "Check that each of your earlier findings is really closed in the current text, not only
  mentioned."
- "Check the other reviewer's claims against the code, especially: {ITS_CODE_CLAIMS}."
- "Say whether any cut removed a protection a normal {WEEK_OR_DAY} needs. If so, name the
  cheapest protection, not the old mechanism."
- "Which of the remaining lines or mechanisms would you remove?"

For the last pass: "If only MINOR findings are left, give the exact text for each so they can
be applied without another pass, and end with VERDICT: AGREE."
