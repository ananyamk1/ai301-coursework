# Procedure: how this skill grades a plan package

These are the operating steps. Follow them in order, exactly as
written. Where a step cannot be completed because the package does not
contain what it asks for, record that absence as the step says and
carry it into grading; do not improvise a substitute source.

## Read order

Read the whole package once, in this order, before grading any check.
The order matters because every check downstream is decided by holding
the plan against something read earlier, and a grader who reads the
plan first will rationalise the evidence to fit it.

1. Read the **issue context** (title, body, labels). Note in one line:
   the single defect being reported, and the behavior the reporter
   says is wrong.
2. Read the **repo-facts block**. Note: what the contribution-policy
   line requires of *issue comments* specifically (an AI-disclosure
   sentence, comments in the author's own voice, or nothing), and
   whether the policy's scope is issue comments or only pull requests
   and code.
3. Read the **thread highlights**. Note every piece of maintainer or
   owner direction: a culprit located, a file or line named, an
   approach proposed, an approach rejected, a patch posted, a test
   requested. If there are no comments, record "no thread direction"
   explicitly — that is a finding the comms check needs, not a blank.
4. Read the **repro-evidence block**, and read it before the plan.
   Note: the environment, the steps, what the artifact shows, and —
   separately and by name — every **control run** it reports and what
   each control rules in or out. The controls are the single most
   decisive evidence in a plan package, because they are what can
   falsify a stated cause.
5. Read the **candidate plan**: diagnosis, scope, files, approach,
   test plan, risks.
6. Read the **candidate plan comment** last, on its own terms, as the
   only part of the package a maintainer on the thread will actually
   see.

In live mode, the same order applies with these sources: step 1-3 from
the issue page and the repo's `CONTRIBUTING.md` / AI policy files, step
4 from the student's own posted repro comment on that issue (or, on the
house issue, the repro pack as quoted in the drafts — if the drafts
quote no repro evidence, record that absence and let the grounding
check grade it), steps 5-6 from `plan.md` and `comment.md`. Read
`scope.md` before any of it and stop if the issue is outside the scoped
repo.

## Evidence gathering

For each check in `rubric.md`, pull its evidence from the location
named below and write down the literal quote or fact before grading.
`references/evidence-guide.md` gives the full map; these are the
gathering moves.

1. **cause-grounded-in-repro** — Quote the plan's diagnosis sentence.
   Then quote each control run from the repro-evidence block. For every
   control, write one line: does this control rule the plan's named
   cause IN, OUT, or leave it untouched? Record the verdict of that
   comparison, not an impression of the diagnosis.
2. **scope-bounded** — Quote the in-scope statement, the not-in-scope
   statement (record "none stated" if absent), and the list of files or
   areas. Then list each distinct unit of work the approach section
   proposes, one line each. Count them against the one defect from
   read-order step 1.
3. **executable-by-stranger** — From the approach section, record
   three facts: the layer or component chosen, the concrete files or
   call sites named, and whether each step is phrased as a decision
   made or as investigation to be done. If no file is named anywhere in
   the plan, record that.
4. **test-plan-decisive** — Quote the test plan verbatim. Next to it,
   write the one observable it names that would differ before and after
   the fix. If the only observable you can extract is "a suite passes"
   or a subjective phrase, write that down as the extraction — do not
   supply an observable the plan did not state.
5. **engages-thread-direction** — From the thread notes in read-order
   step 3, list each piece of maintainer direction. For each, search
   the plan and the comment for any mention of it, and record: adopted,
   departed-from-with-reason, or unaddressed.
6. **comment-meets-repo-conventions** — From the repo-facts note in
   read-order step 2, state the requirement reaching issue comments.
   Then quote the sentence in the plan comment that meets it, or record
   that no such sentence exists.
7. **unknowns-stated-honestly** — Quote any outcome the plan asserts as
   settled, and any risks or unknowns it records.
8. **deferral-reasoned** — Quote each deferral and the reason beside
   it, or record that the plan defers nothing.

In eval mode, every one of these comes from the bundle text and nothing
is fetched. In live mode, the same facts come from the locations in
`references/evidence-guide.md`.

## Check execution

1. Grade the checks in the table's order, top to bottom. Do not let an
   earlier grade influence a later one: each check is decided only by
   the evidence gathered for it in the previous stage.
2. For each check, apply its pass condition as a decision rule to the
   evidence already written down. The evidence is fixed at this point —
   if grading a check makes you want to go back and reread the package,
   reread only the one section the check's evidence line names, and
   update the written evidence before grading.
3. Grade `pass`, `fail`, or `unclear`, and attach the one quote or fact
   that decided it. A grade with no quote attached is not finished;
   go back and gather it.
4. Grade `unclear` only when the evidence the check names is genuinely
   absent from the package — not when it is present and weak. Present
   and weak is a `fail` or a `pass` on the pass condition as written.
5. Where the plan and the plan comment disagree on a fact, grade the
   check on whichever part its evidence line names, and note the
   disagreement in the summary.
6. Do not grade a check on the write-up's shape. Section count, length,
   headings, and polish are not evidence for any check in the rubric; a
   terse plan that carries the facts passes, and a long confident one
   that does not, fails.

## Verdict assembly

1. Separate the graded checks into `required` and `preferred` by the
   weight column in `rubric.md`.
2. Convert every `unclear` on a required check to `fail`, per the
   rubric's verdict rule.
3. If every required check is `pass`, the verdict is `accept`. If any
   required check is `fail`, the verdict is `reject`. There is no third
   verdict, and a count of passes never outweighs a single required
   fail.
4. `preferred` grades are reported in the output and are excluded from
   step 3 entirely. Never let a preferred fail move a verdict.
5. Name the deciding check in the summary. On a `reject`, that is the
   first required check that failed, and quote the evidence line that
   decided it. On an `accept`, state that every required check passed.
6. Emit the JSON block in `SKILL.md`'s format with one entry per check
   in rubric order, grades and evidence as recorded, and the verdict
   from step 3. Nothing follows the JSON block.
