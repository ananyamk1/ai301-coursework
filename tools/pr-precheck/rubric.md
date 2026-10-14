# Rubric: is this pull request ready to submit?

Grading assumption this rubric runs under: every package it grades is
AI-assisted work (the course's PRs are built with an AI assistant in
the loop). So a repo policy that asks for AI-use disclosure is a
policy this package is subject to, not a hypothetical.

What counts as "inside the plan" for every check below: the plan's
scope, its files list, its approach, its test plan, and its deviation
notes. Tests the test plan calls for, and an entry the repo's stated
asks require (a changelog or whatsnew line), are inside the plan even
when the files list does not name them.

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| diff-matches-plan | The diff's changed files and hunks, read one hunk at a time against the plan context's scope, files list, approach, not-in-scope line, and deviation notes; then the plan's promised units of work read back against the diff. | Pass if every hunk is a unit of work the plan (or a deviation note) names, AND every unit of work the plan promises is in the diff or is disclosed as deferred or dropped (in the plan's deviation note or the PR description). Fail if any hunk does work the plan does not name: a rename or refactor pass, a new flag, option, or setting, a rewrite of a function the plan does not touch, a change in a file the plan never mentions, or anything the plan's not-in-scope line excludes. Fail if a promised unit of work is missing with no disclosure. A shortfall that is disclosed passes; a correct core fix does not rescue an unplanned extra. | required |
| description-matches-diff | Each claim the PR title and description make about what the change does, read against the diff: especially fidelity claims ("exactly as planned", "no changes beyond the plan", "no functional changes outside X", "now documents Y"). | Pass if every claim about what changed is true of the diff, and the description does not claim fidelity the diff contradicts. Fail if the description claims something the diff does not contain (docs updated, a behavior or case handled, a file changed) or claims the diff stays inside the plan when it does not. The claims graded here are about behavior, files, docs, and scope; how the description counts or groups the tests it added is not graded here (whether testing is enough is the test checks' job). Omitting a small detail is not a false claim; asserting work that is not there is. | required |
| repro-rerun-decisive | The test-evidence section (and any before/after in the description), read against the plan's test plan: each repro step, failure mode, or input the test plan names, and the expected-after it states. | Pass if the evidence re-runs the plan's repro on the path the fix changes and shows the observed after-result for it, with the before-result shown or quoted from the repro, for every broken case or failure mode the test plan names (a case the description or plan explicitly defers is exempt). Control runs (the cases that already worked, kept as regression guards) may be reported in a sentence; the output requirement applies to the cases that broke. Fail if the evidence is a claim instead of an output ("tested locally, works now", "verified on my machine"), if it runs only a control or an unchanged path (the case that never broke), if its added test exercises something other than the fix, or if any repro case the test plan names is silently not re-run. | required |
| repo-checks-run | The test-evidence section, read against the test or check commands the plan's test plan and the repo's stated asks name (a test suite, lint, typecheck, fmt). | Pass if the outcome of each named command is shown (a pass count, "passes", or a failing run reported honestly with its reason), or if neither the plan nor the repo names any command. Fail if a named command's outcome is absent, or the evidence says only "tests pass" with no command named where the repo names one. An honestly reported failure that is not caused by the diff passes. | required |
| diff-free-of-debris | Every added and removed line of the diff, plus the commit list. | Pass if every hunk serves the fix or its tests, and no line is debris. Fail on any one of: a debug print or log left in (`eprintln!("DEBUG`, `print(` used for tracing), commented-out code (an old attempt or a disabled line), a dead function or `allow(dead_code)` / unused helper, a TODO or FIXME added, import reordering or restructuring the fix does not need, whitespace/re-indent/reformat churn on lines whose content did not change, or an unrelated drive-by edit. A comment that explains the new code is not debris. | required |
| repo-pr-asks-met | The repo-facts block's pull-requests line and contribution-policy line (live: `.github/PULL_REQUEST_TEMPLATE.md` and `docs/CONTRIBUTING.md`), each stated ask read against the PR title, description, and diff. | Pass if every ask the PR itself can satisfy is satisfied: each named template section present with real content (not left as the template's placeholder), an issue link where the template asks for one (`Closes #`, `fixes #`), each named checklist answered, and a changelog or whatsnew entry in the diff where the repo asks for one on bug fixes. Fail if any such ask is visibly unmet. Asks the PR text cannot show (a signed CLA, a vouch flow, CI that only runs after opening) and asks that do not apply to this change (screenshots for a non-UI fix, annotations when no argument was added) never fail this check; test-related asks are graded by the two test checks above, not here. A repo with no template and no stated ask passes. | required |
| ai-disclosure-per-policy | The repo-facts block's contribution-policy line, read for an AI-use policy (live: `docs/CONTRIBUTING.md`, any AI policy file, and the course rule in `scope.md` that the disclosure is always made), checked against the PR description. | Pass if no AI-use policy is stated, or the description meets the stated policy: an explicit sentence saying AI assisted the work, naming the tool or the extent, where the policy requires disclosure; first-person, specific prose where the policy requires human-written text. Fail if a policy requires AI usage to be disclosed and the description contains no disclosure sentence. In live mode the course always requires the disclosure, so a live description with no AI-use sentence fails. | required |
| commits-describe-change | The commit list. | Pass if each commit message says what that commit changes. Fail on messages that say nothing about the change ("wip", "fix", "misc", "stuff"). Never changes the verdict. | preferred |
| title-names-the-change | The PR title, read against the diff. | Pass if the title names the behavior the diff changes, specifically enough that a maintainer scanning a list would know what it fixes. Never changes the verdict. | preferred |

## Verdict rule

Accept if every `required` check grades `pass`. Reject if any
`required` check grades `fail`. `unclear` on a required check counts
as `fail`: a PR whose fidelity, evidence, cleanliness, or compliance
the grader cannot find in the package is a PR a maintainer cannot
find it in either, so it is not ready to submit. `preferred` checks
never change the verdict in either direction; they are reported so
the author can see what a stronger PR would carry. A disclosed
shortfall (a deferred case, a dropped step, an honestly reported
failing check) is never by itself a reason to reject: it passes the
check it touches as long as it is disclosed.
