# Evidence guide: where proof lives in a reproduction package

The map for the checks in `rubric.md`. Each family says where to look
for that proof and what counts as having found it.

## Environment

**Where it lives.** In an eval bundle: the opening lines of the
`## Candidate repro report`, usually an `Environment:` line or block
naming version, install method, OS, and runtime. Read it against two
other places in the same bundle — the `## Issue` section's stated
version and platform, and the `bug reports:` line of the
`## Repo facts` block, which names what the repo's template asks for.
Live mode: the `Environment:` line of the draft repro file, read
against the issue page's opening body and the repo's
`.github/ISSUE_TEMPLATE/` bug template.

**What good looks like.** The record names the version of the tool
under test and the OS or platform it ran on, and it names any extra
axis the issue itself makes decisive — the driver on a
driver-specific issue, the shell on a shell-specific issue, the
browser and language order on a browser issue, the build profile where
the issue says the build profile changes the failure. It does not have
to match the template field for field, and one dense line counts as
much as a table: the question is whether a reader could stand the
attempt up in the same place. An environment section that never names
a version, or never names a platform on a platform-specific issue, is
missing, not terse.

## Steps

**Where it lives.** The `Steps:` block of the repro report —
numbered actions, a fenced shell transcript, a code snippet, or a
config file quoted inline. Anything the steps consume (a config, a
fixture file, a playground link) has to be findable in the same
comment; if the steps mention a file, look for its contents nearby.
Live mode: the same block in the draft, plus any link it gives, which
you should be able to open.

**What good looks like.** A stranger with only this comment can go
from a clean machine to the trigger. Every input is inline or publicly
obtainable: command text they can paste, file contents they can
recreate, a public playground or gist URL, a release they can install.
The opposite is a step that points at something private — "our
internal monorepo", "my `.golangci.yml`", "the project set up as
usual" — or a step that jumps over a condition the issue named as the
trigger without saying why. Step count is not the measure: four lines
that reach the trigger beat twelve that do not.

## Behavior shown

**Where it lives.** The fenced blocks inside the repro report: shell
transcripts, tracebacks, log excerpts, produced output (CSS, JSON,
escape sequences), screenshots and their captions. Read each against
the `## Issue` section's own statement of the failure — its exact
error text, its exit status, its wrong value — and against any
`## Thread highlights` entry where a maintainer settled what correct
output looks like. The report's own `Actual:` sentence is narration,
not artifact; grade the block, not the sentence over it.

**What good looks like.** The artifact exhibits the failure the issue
names, in the same class: the issue's panic actually panics, the
issue's `Invalid path expression` is the message printed, the reported
exit 101 is the exit shown. It is an adjacent symptom — and a fail —
when the block shows a different error class from the same tool (an
argument-validation error where a capacity overflow was reported, a
compile error where a runtime error was reported, a graceful syntax
error where a crash was reported), when the tool is still alive in an
artifact presented as a crash, or when the block only proves the tool
runs at all (a version banner, a session list). A cannot-reproduce
report is read differently: its artifact should show the trigger being
attempted and the reported behavior failing to appear, which is a
result, not an absence of one.

## Honesty

**Where it lives.** At the seam between two parts of the same bundle:
every assertion in the claim comment and the report's prose —
"reproduced", "confirmed on two machines", "guaranteed reproducible",
"I verified this race condition", "this also affects the Store build"
— set beside the fenced artifacts that are actually present. Also the
report's `Result:` or opening line, where a cannot-reproduce declares
itself, and the "what differed" paragraph where an honest negative
names its own limits.

**What good looks like.** Each assertion has a block under it, or is
marked as a guess, a question, or a next step ("I'd like to check
whether…", "my hypothesis is…"). An evidenced cannot-reproduce is the
strongest honest form there is: it states the negative result, shows
the attempt, names what differed from the reporter's conditions, and
says what a triggering setup might need. What fails here is certainty
with nothing under it — a root cause named with no transcript, a
confidence adverb standing in for a run, or a finding stretched to
cover a build or platform that was never tested.

## Comms

**Where it lives.** Two places. The `## Candidate claim comment`, read
against the `## Issue` section's specifics — its symptom, version,
file, function, and any pointer a maintainer left in
`## Thread highlights`. And the `contribution policy` line of the
`## Repo facts` block, read for what it requires of *issue comments*
specifically, then checked against the text of both comments. Live
mode: the repo's `CONTRIBUTING.md`, `AI_POLICY.md` or
`AI_USAGE_POLICY.md`, plus `scope.md` in this skill directory for the
house rules that override the usual reading in the course's Path
Review repo.

**What good looks like.** A claim comment carries at least one detail
that could not be pasted onto another issue — the error it names, the
version it reproduced on, the function the thread pointed at — and
promises only investigation or reproduction. Boilerplate is the
inverse: warm, generic, transferable to any repo ("great project",
"kindly assign it to me"), and it usually arrives carrying a promise
the writer cannot keep, a fix or a deadline. On policy, read the scope
of the rule before applying it: a policy that binds pull requests and
code says nothing about an issue comment, a policy that asks for
comments in the contributor's own voice is met by specific first-person
prose, and a policy that says all AI usage in any form must be
disclosed is met only by an explicit disclosure sentence naming the
assistance — and these packages are AI-assisted, so that sentence has
to be there.
