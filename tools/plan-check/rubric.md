# Rubric: is this plan ready to post and build from?

Grading assumption this rubric runs under: every package it grades is
AI-assisted work (the course's plans are drafted with an AI assistant in
the loop). So a repo policy that asks for AI disclosure is a policy this
package is subject to, not a hypothetical.

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| cause-grounded-in-repro | The plan's stated cause (its diagnosis sentence), read line by line against the repro-evidence block — its artifact, and especially any control run the block reports, which is the part that rules a candidate cause in or out. | Pass if the stated cause explains the behavior the repro evidence actually shows, and no control run in that block contradicts it. Fail if a control run rules the named cause out (the symptom persists with that component removed, or disappears without it), if the diagnosis rests on a mechanism the evidence never exercises, or if the plan adopts a cause asserted in the thread that the package's own evidence contradicts. A diagnosis the evidence supports but does not fully prove passes when the plan says so. | required |
| scope-bounded | The plan's in-scope and not-in-scope statements and its list of files or areas, read against the single defect the issue reports. | Pass if the work proposed is one bounded change addressing that defect, including when the plan deliberately narrows and names what it is deferring. Fail if the plan carries work the issue never asked for — a refactor, a migration, a dependency swap, a new option or setting, a redesign of the surrounding component, a CI or test-harness change — regardless of whether the core fix inside it is correct. An explicit deferral is the opposite of scope creep and never fails this check. | required |
| executable-by-stranger | The plan's approach section: the files or areas named, the change described at each, and the order of work. | Pass if someone who has not spoken to the author could open the repo and start the first step: the layer or component is chosen, at least one concrete file or call site is named, and the change at it is described as a decision already made. Fail if the real decisions are deferred to build time — no file named, the layer left open ("gocui? tcell? not sure"), the approach phrased as investigation ("profile it and see", "poke around"), or an either/or the plan never resolves ("upstream or vendored, whichever is easier"). | required |
| test-plan-decisive | The plan's test plan, read against the repro evidence's steps and artifact. | Pass if it names an observable outcome specific to this fix that a reader could check and that would differ before and after the change: a value, an exit status, a rendered colour or string, a named test case, a specific line of output. Fail if the only stated outcome is that an existing suite passes or nothing regresses, or if the outcome is subjective ("should feel fast", "nothing else should feel broken") — those are true of a build that does not fix anything. | required |
| engages-thread-direction | The issue's thread highlights, read for direction a maintainer or owner has already given — a culprit located, a file or line named, an approach proposed or rejected, a patch posted, a test requested — set against the plan and the plan comment. | Pass if there is no such direction in the thread, or the package engages it: adopts it, or takes a different route and says why. Fail if a maintainer has located the culprit or proposed an approach and the package proceeds as though the thread were empty — most plainly, a plan that routes around a named code-level culprit without ever acknowledging it. | required |
| comment-meets-repo-conventions | The repo-facts block's contribution-policy line, read for what it requires of issue comments specifically, checked against the text of the candidate plan comment. | Pass if the policy states no requirement reaching issue comments, or states one and the comment meets it: an explicit AI-disclosure sentence where the policy requires AI usage to be disclosed, or specific first-person prose in the contributor's own voice where the policy requires human-written comments. Fail only when the policy requires a statement or property of issue comments and the comment does not provide it — most plainly, a stated all-AI-usage-must-be-disclosed policy with no disclosure in the comment. A policy that binds only pull requests or code never fails this check. | required |
| unknowns-stated-honestly | Any place the plan commits to an outcome it has not established: performance claims, "this will fix it" phrasing, and the risks or unknowns the package does or does not record. | Pass if what the plan has not yet established is marked as open, or if the plan makes no claim beyond what its evidence carries. Fail if the plan asserts a settled outcome for something its own evidence leaves open. Never changes the verdict; a terse plan that simply claims nothing extra passes as fully as one with a risks section. | preferred |
| deferral-reasoned | The plan's not-in-scope line, where it defers a larger or alternative route. | Pass if each deferral carries a reason, or the plan defers nothing. Never changes the verdict; it records that a narrowing was a decision rather than an omission. | preferred |

## Verdict rule

Accept if every `required` check grades `pass`. Reject if any
`required` check grades `fail`. `unclear` on a required check counts as
`fail`: a plan whose grounding, scope, approach, or observable outcome
the grader cannot find in the package is a plan a maintainer cannot
find it in either, and it is not ready to build from. `preferred`
checks never change the verdict in either direction; they are recorded
in the output so the author can see what a stronger package would have
carried.
