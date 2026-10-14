# Unit 4 — Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

Record of the pull request you opened against the Path Review repo, and of the evaluation
runs that produced `eval-run.txt`. This file is graded at the path above; a copy kept
anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your pull request

**Pull request**

[PR_LINK_PENDING](https://github.com/codepath/pathreview-ai301-fa26-s1/pull/203)

**Branch**

`fix/68-empty-index-guard`

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

1. **Full run: 18/20 agreement, bar 18/20: PASS.** Every category matched
   (`clear-accept 5/7  not-tested 4/4  silent-drift 4/4  standards-wall 2/2
   unreviewable 3/3`). Both misses were clear-accepts my rubric rejected:
   pkg-11 on `description-matches-diff` and pkg-19 on `repro-rerun-decisive`.
   (An attempt before this one crashed at startup under the system Python 3.9
   before grading anything. It produced no score, and I re-ran under
   Python 3.13.)
2. **`--only pkg-11,pkg-19,pkg-07,pkg-14,pkg-17,calib-04 --include-calibration`:
   5/5 scored agreement**, plus calib-04 agreeing. These were the two misses
   after I loosened the two checks, plus canaries for each loosening: pkg-07
   and pkg-14 (not-tested: control-only and unchanged-path evidence), pkg-17
   (silent-drift: a false docs claim in the description), and calib-04 (one of
   two named failure modes never re-run). Partial run, so no bar verdict.
3. **Full run: 19/20 agreement, bar 18/20: PASS.** Every category matched
   (`clear-accept 6/7  not-tested 4/4  silent-drift 4/4  standards-wall 2/2
   unreviewable 3/3`). This is the run saved in `eval-run.txt`. The one miss
   is pkg-11, analysed below.

**Package analysis**

**pkg-11** (BurntSushi/ripgrep#3477: escaped trailing space in `.gitignore`).
Gold label: **accept**. My rubric's verdict in the committed run: **reject**.

It failed one required check, `repro-rerun-decisive`, with this evidence line:

> `$ rg hi   # after: searches normally, no error` is a comment annotation, not
> captured stdout/exit-code output, for the case that actually broke

Every other required check passed, including the one that sank it in run 1.

My rubric read it that way because the pass condition asks the evidence to show
"the observed after-result" for every broken case. pkg-11's evidence is a real
transcript of commands, but its after-results are written as shell comments
beside the commands (`# after: searches normally, no error`, `# after: 'foo '
not listed (ignored)`), not printed output. A strict reading of "observed"
treats an annotation as a claim, and my fail list does name "a claim instead of
an output".

I think gold is right and my check is too literal for this package. "Searches
normally, no error" is the whole observable here: the bug was an error line, and
its absence can't be printed. The second annotation names a specific result
(`'foo '` not listed, matching `git check-ignore`). That is very different from
pkg-04's "tested locally, colors work now" or pkg-10's "verified working on my
machine for a full day", which name no command at all. This package also flipped
between runs. In run 1 it passed this check and failed `description-matches-diff`
instead, over how many test cases the description counted. So it sits right on
my rubric's boundary, and the grader's reading varies from run to run. A rubric
that is stable on it would need to say outright that a command shown with its
stated after-result counts as observed.

**Check rationale**

Quoting `repro-rerun-decisive` from the `rubric.md` I uploaded to `tools/pr-precheck/`
(fingerprinted `sha256:dc4ec1636315485b` in the committed `eval-run.txt` header):

> **Evidence:** The test-evidence section (and any before/after in the description), read against the plan's test plan: each repro step, failure mode, or input the test plan names, and the expected-after it states.
>
> **Pass condition:** Pass if the evidence re-runs the plan's repro on the path the fix changes and shows the observed after-result for it, with the before-result shown or quoted from the repro, for every broken case or failure mode the test plan names (a case the description or plan explicitly defers is exempt). Control runs (the cases that already worked, kept as regression guards) may be reported in a sentence; the output requirement applies to the cases that broke. Fail if the evidence is a claim instead of an output ("tested locally, works now", "verified on my machine"), if it runs only a control or an unchanged path (the case that never broke), if its added test exercises something other than the fix, or if any repro case the test plan names is silently not re-run.

The sentence about control runs is the one I added after run 1. My first version
asked for shown output "for every repro case or failure mode the test plan
names", and pkg-19 failed on it. pkg-19's test plan names the issue's file plus
two controls (the double-quoted twin, and single-quoted without the trailing
blank line). Its evidence shows a full before/after and a `diff ... && echo
IDENTICAL` for the broken case, but reports the controls in one sentence:
"unchanged". My check counted the controls as two of three cases with no output
and failed it. That is the wrong target. Controls are the cases that already
worked, so the risk in skipping their output is a regression, and the suite run
covers that. The not-tested failure is about the broken case never being shown.

I kept the rest of the condition strict on purpose, because the not-tested
category is built from near-misses. "If it runs only a control or an unchanged
path" is there for pkg-07, whose only evidence is the single-file control that
never broke, and for pkg-14, which re-runs a GET when the bug is in a POST.
"Every broken case or failure mode the test plan names" is there for calib-04,
whose plan names two failure modes and whose evidence re-runs only one. The
deferral exemption is there for pkg-13 and pkg-16, whose shortfalls are
disclosed. A rubric that equates "less than everything" with "hold" fails those
two.

**Trade-offs**

Loosening `repro-rerun-decisive` for controls gives up something real. A PR
could now say "controls unchanged" without showing them, and a regression in a
control would be visible only in the suite run. I accepted that because the
alternative rejected pkg-19, a clean, decisively tested clear-accept. Before
spending on a full run, I checked the loosening didn't open a hole with a
canary run (`--only pkg-11,pkg-19,pkg-07,pkg-14,pkg-17,calib-04
--include-calibration`). pkg-07 and pkg-14 still rejected. Those are the two
not-tested packages the change could most plausibly flip, since their evidence
runs a control or an unchanged path. calib-04 also still rejected on the
skipped failure mode. The confirming full run kept not-tested at 4/4.

The other loosening, telling `description-matches-diff` not to grade how a
description counts its tests, risked the silent-drift category. So pkg-17 (a
description claiming docs the diff doesn't contain) was in the same canary run
and still rejected, and silent-drift stayed 4/4 in the full run.

The case I accept missing is pkg-11's: evidence that states each after-result
as an annotation instead of printing it. I could add "a command shown with its
stated after-result counts as observed", and pkg-11 would very likely pass. I
didn't, because that sentence is close to what pkg-04 and pkg-10 do in prose.
Separating "command plus stated result" from "stated result with no command"
is a narrow line. I'd want a canary run on all four not-tested packages before
trusting it, and the bar was already cleared.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/pr-precheck/`.
