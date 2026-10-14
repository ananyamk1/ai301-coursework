# Procedure: how this tool grades a PR package

These are the operating steps. Follow them in order, exactly as
written. Where a step cannot be completed because the package does not
contain what it asks for, record that absence as the step says and
carry it into grading; do not improvise a substitute source.

## Read order

Read the whole package once, in this order, before grading any check.
The order matters because every check is a side-by-side: the diff held
against the plan, the evidence held against the test plan, the
description held against the diff. Read the description before the
diff and its claims become the lens you read the diff through, which
is exactly how silent drift gets past a reviewer. So the plan comes
first, the diff second, and the description last.

1. Read the **repo-facts block**. Write down, as a numbered list, every
   ask the pull-requests line and the contribution-policy line make of
   a PR: template sections by name, an issue-link line, checklists, a
   changelog or whatsnew entry, test or check commands, and any AI-use
   policy (what it requires, and whether it reaches PR text). Mark each
   ask as "PR can show" or "PR cannot show" (CLA, vouch flow, CI after
   opening).
2. Read the **issue** and **thread highlights**. Note in one line the
   single defect the issue reports, and any maintainer direction the
   thread gives that the plan adopted.
3. Read the **plan context**, before any of the candidate PR. Write
   down four lists: (a) the units of work the plan promises, one line
   each (each file and the change at it, plus any docs, tests, or
   changelog entries it names); (b) the not-in-scope line, verbatim;
   (c) each repro case, failure mode, and test command the test plan
   names, with its expected-after; (d) every deviation or deferral
   note, verbatim.
4. Read the **diff**, hunk by hunk. For each hunk, write one line: the
   file, and what the hunk does in plain words. Do this before reading
   the description.
5. Read the **commit list**.
6. Read the **test evidence**.
7. Read the **title and description** last.

In live mode the same order applies with these sources: read
`scope.md` first and stop if it says to; step 1 from
`.github/PULL_REQUEST_TEMPLATE.md`, `docs/CONTRIBUTING.md`, and the
house rules in `scope.md`; step 2 from the issue page; step 3 from
`plan.md` including `## Deviations`; step 4 from `git diff main...HEAD`;
step 5 from `git log --oneline main..HEAD`; step 6 from
`test_evidence.md`; step 7 from `pr_draft.md` (line 1 is the title).

## Evidence gathering

For each check in `rubric.md`, gather its evidence from the location
named below and write down the literal quote or fact before grading.
`references/evidence-guide.md` gives the full map.

1. **diff-matches-plan**: Put the hunk list (read-order step 4) next to
   the plan's units of work (step 3a). For each hunk, record the plan
   line or deviation note it maps to, or "UNPLANNED". Then for each
   plan unit of work, record the hunk that delivers it, or "MISSING",
   and for each MISSING, whether a deviation note or the description
   discloses it. Separately, record any hunk that touches something
   the not-in-scope line (step 3b) excludes.
2. **description-matches-diff**: List each claim the title and
   description make about what changed, one per line, quoting it. Next
   to each, record "in diff" (with the hunk) or "not in diff". Quote
   every fidelity claim ("exactly", "no changes beyond", "only") and
   record whether the UNPLANNED list from gathering step 1 makes it
   false.
3. **repro-rerun-decisive**: Put the test plan's repro cases (step 3c)
   next to the test evidence. For each case, quote the command run and
   its after-output, and record "re-run" or "not re-run". Record
   whether the case run is the one that broke or a control/unchanged
   path. If the evidence is prose with no output, quote the prose.
4. **repo-checks-run**: List each command the test plan (step 3c) and
   the repo asks (step 1) name. For each, quote its outcome line from
   the evidence, or record "no outcome shown".
5. **diff-free-of-debris**: Scan every `+` and `-` line of the diff.
   Record each debug print, commented-out line, dead or unused
   function, added TODO/FIXME, import reorder, and whitespace-only or
   re-indent change, with its file. Record the commit messages.
6. **repo-pr-asks-met**: For each "PR can show" ask from read-order
   step 1, quote the part of the description or diff that meets it, or
   record "absent". For template sections, record whether the section
   has real content or only the placeholder.
7. **ai-disclosure-per-policy**: Restate the AI policy from read-order
   step 1 (or "no AI policy stated"). Quote the description sentence
   that discloses AI use, or record that none exists. In live mode the
   course requirement always applies.
8. **commits-describe-change** and **title-names-the-change**: quote
   the commit messages and the title.

In eval mode every one of these comes from the bundle text and nothing
is fetched. In live mode the same facts come from the files and
commands named under Read order.

## Check execution

1. Grade the checks in the rubric table's order, top to bottom. Each
   check is decided only by the evidence gathered for it; do not let
   one grade influence another, even when the same fact appears in
   two checks.
2. Apply each pass condition as a decision rule to the written-down
   evidence. If you need to look again, reread only the part of the
   package the check's evidence column names, and update the written
   evidence first.
3. Grade `pass`, `fail`, or `unclear`, and attach the one quote or
   fact that decided it. A grade without a quote is not finished.
4. Grade `unclear` only when the evidence the check names is genuinely
   absent from the package (no test-evidence section at all, no plan
   context). Evidence that is present but weak is graded `pass` or
   `fail` on the pass condition as written.
5. A disclosure counts wherever it appears: a shortfall named in the
   plan's deviation note, or in the description, is disclosed. Search
   both before recording MISSING or "not re-run" as undisclosed.
6. Do not grade shape: section count, length, formatting, and
   confident tone are not evidence for any check. A terse PR that
   carries the facts passes; a polished one whose diff contradicts it
   fails.

## Verdict assembly

1. Split the graded checks into `required` and `preferred` by the
   rubric's weight column.
2. Convert every `unclear` on a required check to `fail`, per the
   rubric's verdict rule.
3. If every required check is `pass`, the verdict is `accept`. If any
   required check is `fail`, the verdict is `reject`. A count of
   passes never outweighs one required fail. Preferred grades never
   enter this step.
4. Name the deciding check in the summary. On a `reject`, it is the
   first failing required check in rubric order; quote its evidence
   line. On an `accept`, state that every required check passed.
5. Emit the JSON block from `SKILL.md`, with one entry per rubric
   check in rubric order, the grades and evidence as recorded, and the
   verdict from step 3. Nothing follows the JSON block.
