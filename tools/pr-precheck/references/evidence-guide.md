# Evidence guide: where evidence lives in a PR package

The map for the checks in `rubric.md`. `procedure.md` says when each
family gets gathered; this file says where it is and what counts as
having found it. In an eval bundle every location is a section of the
package file. In live mode the locations are files in the student's
working copy on their branch, plus the issue page.

## Plan fidelity (harness category: silent-drift)

Checks: `diff-matches-plan`, `description-matches-diff`.

**Where it lives (eval bundle).** The plan's side is the
`## Plan context` block: the `Plan:` paragraph's `Scope, one bounded
change:` sentence, its `Not in scope:` sentence, its `Files:` list,
and any deferral or "known limit" note. The PR's side is the
`### Diff` section under `## Candidate PR` (each `--- a/` / `+++ b/`
file header and each `@@` hunk), and, for the claims, the `### Title`
and `### Description` sections.

**Where it lives (live).** The plan's side is `plan.md`: its
`## Scope`, `## Files I will touch`, `## Approach`, and `## Deviations`
sections. The PR's side is `git diff main...HEAD` (and `--stat` for
the file list), and the claims are in `pr_draft.md` (line 1 is the
title, the rest is the description, especially its Summary and
Changes sections).

**What good looks like.** Every hunk maps to a unit of work the plan
or a deviation note names, and every unit of work the plan promises
has a hunk, or is named as deferred. Tests the test plan calls for and
a changelog entry the repo asks for are inside the plan. Drift runs in
two directions. More than the plan: a one-line fix that arrives with a
rewrite of a neighbouring function, a renaming pass, or a new config
option or flag, especially one the not-in-scope line excluded. Less
than the plan: a promised docs update or second case missing with no
note. Either becomes drift when the description papers over it ("implements
the posted plan exactly", "no functional changes outside the fix",
"docs now document the limitation"). A description that names a
shortfall ("`%VAR%` expansion is NOT handled here; the plan's deferral
note covers it") is the opposite of drift and passes.

## Test evidence (harness category: not-tested)

Checks: `repro-rerun-decisive`, `repo-checks-run`.

**Where it lives (eval bundle).** The `### Test evidence` section
under `## Candidate PR`, plus any before/after block inside
`### Description`, read against the `Test plan:` sentence of the
plan context and the `Repro evidence` paragraph (which says what the
broken path is and what the controls were). Commands the repo asks
for are on the repo-facts block's pull-requests and
contribution-policy lines.

**Where it lives (live).** `test_evidence.md` (the repro re-run on
`main` for the before and on the branch for the after, then the
output of `make test-unit`, `make test-integration`, `make lint`, and
`make typecheck`, the four the PR template lists, plus `black --check .`
and the frontend tests, because `docs/CONTRIBUTING.md` asks for
`make check` (lint, format, typecheck) and CI runs five jobs), read against `plan.md`'s `## Test plan`. The
Testing section of `pr_draft.md` should carry the same output.

**What good looks like.** Each repro case the test plan names is
re-run on the path the fix changes, and the evidence shows its output
after the change next to the before (or the repro's quoted before),
with the expected-after the plan stated (controls, the cases that
already worked, can be reported in a sentence; the broken cases need
output): `16 passed, 1 xfailed` then
`17 passed`; `len == 99999` then `100000`; exit 0 and a printed error
where there was silence. Then each named check command has its
outcome line. What fails: prose instead of output ("tested locally,
colors work now", "verified working for a full day"); a run of only
the control or of an unchanged path (the single-file case that never
broke, a GET when the bug is in a POST); one of two named failure
modes re-run and the other silently skipped; "tests pass" with no
command. A failing check reported honestly with its cause (Path
Review's integration target stopping at "no tests ran") is evidence,
not a gap.

## Diff quality (harness category: unreviewable)

Check: `diff-free-of-debris` (and the preferred
`commits-describe-change`).

**Where it lives (eval bundle).** Every `+` and `-` line of the
`### Diff` section, and the `### Commits` list.

**Where it lives (live).** `git diff main...HEAD` line by line, and
`git log --oneline main..HEAD`.

**What good looks like.** Every changed line serves the fix or its
test; a reviewer reading the diff sees the fix and nothing else.
Debris tells, any one of which fails: a debug print or `eprintln!` /
`print()` left in; commented-out code (a first attempt, a disabled
line); a dead or unused function, often behind `allow(dead_code)` or
named `_unused_...`; an added TODO or FIXME; import reordering or
restructuring the fix does not need; identical lines removed and
re-added (re-indent, reformat, whitespace churn); a drive-by edit to
unrelated code. Commit messages like `wip`, `fix`, `fmt + cleanup`,
or `misc cleanups while debugging` are a symptom that usually travels
with debris. A comment that explains the new code is not debris.

## Standards and comms (harness category: standards-wall)

Checks: `repo-pr-asks-met`, `ai-disclosure-per-policy` (and the
preferred `title-names-the-change`).

**Where it lives (eval bundle).** The `## Repo facts` block: its
`pull requests:` line (the template's sections, checklist, issue-link
line, and any changelog/whatsnew ask) and its `contribution policy:`
line (contributing guide and any AI policy). The PR's answers are in
`### Title`, `### Description`, and, for a changelog entry, the
`### Diff`.

**Where it lives (live).** `.github/PULL_REQUEST_TEMPLATE.md` (Path
Review: Summary, Issue with `Closes #`, Changes, Testing checklist,
Screenshots / Demo, Notes for Reviewers), `docs/CONTRIBUTING.md`
(branch naming, conventional commits, run the checks, fill the
template completely, remove the seeded-bug `xfail` marker), and the
house rules in `scope.md`. The course always requires an AI-use
disclosure, which Path Review's template has no section for, so it
goes under Notes for Reviewers. The answers are in `pr_draft.md`.

**What good looks like.** Every ask the PR text can show is shown:
each template section present with real content rather than the
placeholder, `Closes #<n>` or `fixes #<n>` where asked, the checklist
answered truthfully (a box ticked only if true; the CI box left
unticked before CI has run is honest, not missing), a changelog or
whatsnew entry in the diff where the repo asks for one. Where an AI
policy requires disclosure, the description has an explicit sentence
saying AI assisted and to what extent ("this change was AI-assisted
with me reviewing and understanding every line, and this text is in
my own words"). The walls: a required checklist with no `closes #`,
no checklist, and the stated whatsnew entry absent from the diff; an
excellent fix in a repo whose policy says all AI usage must be
disclosed, with no disclosure anywhere. Asks a PR cannot show (CLA,
vouch flow) are not graded.
