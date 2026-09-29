# Rubric: is this reproduction package ready to post?

Grading assumption this rubric runs under: every package it grades is
AI-assisted work (the course's packages are drafted with an AI
assistant in the loop). So a repo policy that asks for AI disclosure is
a policy this package is subject to, not a hypothetical.

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| environment-recorded | The environment record in the repro report (the line or block naming OS, tool/package version, and any install method, runtime, or platform detail the issue's behavior depends on), read against the issue's own stated version/platform and the repo-facts block's bug-report template asks. | Pass if the report names, somewhere in its own text, the tool version under test AND the operating system or platform it ran on, plus any further axis the issue itself makes decisive (e.g. the driver on a driver-specific issue, the shell on a shell-specific issue, the browser on a browser issue). Pass an issue stated to apply to all platforms on version alone. Fail if the version or the platform is absent, or if the issue singles out a variable the report never states. Terseness is not a fail: one line carrying the facts passes as fully as a table. | required |
| steps-rerunnable | The steps section of the repro report: the commands, code, config files, or UI actions between the starting state and the trigger, plus any inputs they depend on. | Pass if a stranger holding only this comment could re-run the attempt: every input the steps consume is either given inline (command text, file contents, snippet, public URL, public playground link) or is a resource anyone can obtain. Fail if any step depends on something the reader cannot get (a private repo, an unshared config, a local file never shown, "set up the project" with no commands), or if a step the issue names as decisive is skipped without comment. | required |
| artifact-shows-issue-behavior | The report's artifacts (output excerpts, tracebacks, logs, produced CSS/JSON, screenshots) read line by line against the specific failure the issue describes — its error text, exit status, or wrong value — not against the report's own narration of them. | Pass if the artifact exhibits the failure mode the issue names. If the report's stated result is a cannot-reproduce, pass instead when the artifact shows a real attempt at the issue's trigger and records the absence of the reported behavior. Fail when the artifact shows a different failure than the issue's (a different error class, a graceful error where a crash was reported, a survivable state narrated as a crash), when it shows only that the tool runs, or when there is no artifact at all. | required |
| conditions-match-or-deviation-named | The version, branch, platform, and configuration the report actually tested, compared against the version/branch/platform the issue and its thread establish the bug on (including any "confirmed on latest and main" template ticks in the issue context). | Pass if the tested conditions fall inside the range the issue establishes, OR if they differ and the report itself names the difference in its own text. A stated delta ("filed against 13.0.0, unchanged on 15.2.0") passes. Fail if the report tested a version or platform outside the issue's established range and never says so, so the reader cannot tell whether the artifact is about the reported bug at all. | required |
| claims-matched-by-evidence | Every assertion the two comments make about what is true — "reproduced", "confirmed", "guaranteed reproducible", a named root cause, a scope generalization ("affects the Store build too") — set beside the artifacts actually shown in this package. | Pass if each such assertion is backed by a shown artifact, or is marked as a hypothesis, question, or next step. An explicitly evidenced cannot-reproduce passes in full: saying what did not happen, with the attempt shown, is an honest outcome. Fail if the package asserts more than its artifacts support — certainty with no artifact, a root cause declared without a transcript, or a claim generalized past what was run. | required |
| claim-comment-specific-and-honest | The candidate claim comment on its own, read against the issue's specifics and against what the package can actually promise. | Pass if the claim names something particular to this issue (its symptom, version, file, function, or the thread's pointer) AND states an intention limited to investigation or reproduction. Fail if it is interchangeable boilerplate that would fit any issue ("great project, please assign me"), a bare +1 with no stated intent, or if it promises a fix, a deadline, or a guaranteed outcome the package has not earned. | required |
| repo-conventions-satisfied | The repo-facts block's contribution-policy line, read for what it requires of issue comments specifically, checked against the text of both candidate comments. | Pass if the policy states no requirement reaching issue comments, or states one and the comments meet it: an explicit AI-disclosure statement where the policy requires AI usage to be disclosed, or specific first-person prose in the contributor's own voice where the policy requires human-written comments. Fail only when the policy requires a statement or property of issue comments and neither comment provides it — most plainly, a stated all-AI-usage-must-be-disclosed policy with no disclosure anywhere in the comments. A policy that binds only pull requests or code never fails this check. | required |
| control-or-contrast-run | Any second run in the report shown for contrast: the working case, the pre-trigger state, or the variant that isolates the decisive variable. | Pass if the report shows a contrast run alongside the failing one, or says why one is not possible. Never changes the verdict; it records that the report isolated its variable rather than only exhibiting it. | preferred |

## Verdict rule

Accept if every `required` check grades `pass`. Reject if any
`required` check grades `fail`. `unclear` on a required check counts as
`fail`: evidence the grader cannot find is evidence a stranger reading
the thread will not find either. `preferred` checks never change the
verdict in either direction.

One exception, live mode only: on a claim-only draft, the checks whose
evidence is the repro report (`environment-recorded`,
`steps-rerunnable`, `artifact-shows-issue-behavior`,
`conditions-match-or-deviation-named`, `control-or-contrast-run`) are
reported `unclear` with evidence `not yet applicable: claim-only
draft` and are left out of the verdict. `claims-matched-by-evidence`
still applies to the claim comment's own assertions;
`claim-comment-specific-and-honest` and `repo-conventions-satisfied`
apply as written; and the verdict answers only whether the claim
comment is ready to post.
