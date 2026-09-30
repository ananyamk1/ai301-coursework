# Evidence guide: where evidence lives in a plan package

The map for the checks in `rubric.md`. `procedure.md` says when each
family gets gathered; this file says where it is and what counts as
having found it.

## Diagnosis and grounding

**Where it lives.** Two places, always read against each other. The
plan's cause statement is the first line or two of the
`## Candidate plan` section, usually opening `Cause:` or `Diagnosis:`.
The behavior it has to explain is in the `## Repro evidence` block: its
`Actual:` line, its artifact (a traceback, a timing matrix, a produced
value, a rendered output), and — the decisive part — any run the block
labels a **control**. Live mode: the diagnosis in `plan.md`, read
against the repro comment the author posted on the issue in week 2.

**What good looks like.** The stated cause explains the artifact, and
every control run in the block is consistent with it. Controls are what
falsify a diagnosis, so read them first and literally: a control that
removes the accused component and still shows the symptom rules that
component out, and a control that shows the symptom vanishing without
the accused component in play rules it out just as hard. The common
failure is not a vague diagnosis but a confident one the package's own
control already contradicts — a plan blaming the tokenizer when the
no-flag control parses fine, or blaming key bindings when the timing
matrix shows the cost with bindings unchanged. A thread comment
asserting a cause is not evidence for it; where a maintainer's stated
cause and the repro evidence disagree, the evidence decides. A
diagnosis that is supported but not fully proven is fine when the plan
says which part is inference.

## Scope

**Where it lives.** The plan's `Scope:` paragraph — its in-scope
sentence, its `Not in scope:` sentence, and the `Files:` list. Read
them against the single defect named in the `## Issue` section's title
and body.

**What good looks like.** Everything proposed serves the one defect the
issue reports, and the not-in-scope line actively fences off the
adjacent work a reader would otherwise expect. One bounded change can
touch several files and still be bounded; conversely a plan that names
few files can still be unbounded if one of them is "rewrite the
printer". What scope creep looks like concretely: a refactor, a
dependency migration, a new user-facing option or setting, a redesign
of the surrounding component, or a CI/test-harness change riding along
with the fix — the tell is work the issue never asked for, and it is
still scope creep when the fix buried inside it is correct. A
deliberate narrowing is the opposite signal: a plan that says "option 2
only, deferring option 1's rework because the maintainers may prefer it
long term" is better scoped than one that does both, not worse.

## Executability

**Where it lives.** The plan's `Approach:` section — its numbered
steps — plus the `Files:` list beside it.

**What good looks like.** A stranger could open the repo and start step
1 without asking the author anything: the layer is chosen, at least one
real file or call site is named, and each step reads as a decision
already taken rather than a question to answer later. The failure is
not brevity — a three-line approach naming one file and one change is
executable — it is deferral: no file named anywhere, the layer left
open ("gocui? tcell? not sure"), the work phrased as investigation
("profile and optimize", "poke around the editor code"), or an
unresolved either/or ("upstream or vendored, whichever is easier").
Every one of those moves the real decision to build time, which is
exactly the decision a plan exists to make.

## Test plan

**Where it lives.** The plan's `Test plan:` section, read against the
`## Repro evidence` block's steps and artifact.

**What good looks like.** It names something a reader could observe
that would read differently before and after the change: a specific
exit status, a printed path, a colour flip at a named step, a value, a
named regression case built from the issue's inputs. The strongest form
re-runs the repro's own steps and states what the artifact should say
instead. What fails is an outcome that a build fixing nothing would
also satisfy: "run the full test suite and make sure nothing
regresses", "nothing else should feel broken", "should feel fast". Note
that such a plan can be excellent everywhere else — bounded, grounded,
thread-aware — and still leave nobody able to tell whether it worked,
which is what this family exists to catch.

## Honesty

**Where it lives.** The plan's `Risk:` or risks-and-unknowns section,
and anywhere in the plan or comment an outcome is asserted — a
performance claim, a "this will fix it", a scope generalisation. Live
mode adds the `## Deviations` heading at the end of `plan.md`, where a
mid-build departure from the posted plan gets recorded.

**What good looks like.** What the plan has not established is marked
as open, in the plan's own words: an unmeasured cost flagged for
review, an alternative left to the maintainers, a question the author
would take to review. Terseness is not dishonesty — a plan that simply
claims nothing beyond what it has shown is honest, and needs no risks
section to be so. Dishonesty is a settled claim over an open question.
After a build, an honest deviation is one written into `## Deviations`
with what changed and why; a deviation that exists only in the diff is
not recorded at all, and "nothing changed" written in the author's own
words is a complete and honest entry.

## Comms

**Where it lives.** Three sources meeting at the
`## Candidate plan comment`. First, the `## Thread highlights` list —
read it for direction already given: a culprit located, a file or line
named, an approach proposed, an approach rejected, a patch posted, a
test requested. Second, the `contribution policy` line of the
`## Repo facts` block, read for what it requires of *issue comments*
specifically. Third, the issue body itself, for what the comment claims
about it. Live mode: the live thread, the repo's `CONTRIBUTING.md` and
any `AI_POLICY.md` / `AI_USAGE_POLICY.md`, plus `scope.md` for the Path
Review house rules.

**What good looks like.** Thread-aware means the comment shows it has
read the thread: it adopts the direction a maintainer gave, or departs
from it and says why, and it engages prior art (an open PR, a posted
patch) instead of racing it. The failure that looks most innocent is a
competent plan that routes around a culprit an owner already located
and posted a patch for, without ever mentioning it — competent, and
unpostable. On policy, read the rule's scope before applying it: a
policy binding pull requests and code says nothing about an issue
comment; a policy asking for comments in the contributor's own voice is
met by specific first-person prose; and a policy saying all AI usage in
any form must be disclosed is met only by an explicit disclosure
sentence naming the assistance — and these packages are AI-assisted, so
that sentence has to be there.
