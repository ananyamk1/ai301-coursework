---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

## The question

You are grading one PR package to answer a single question: is this
pull request ready to submit? A PR package is a candidate pull request
(its title, its description, its commit list, its diff, and its test
evidence) read against three things: the plan it claims to implement
(including any deviation notes recorded in that plan), the issue the
plan belongs to, and the repo's stated standards for pull requests
(its PR template, contributing guide, and any AI-use policy).

Answer only that question, about only that one package. Do not review
the code for style, do not suggest a better fix, do not re-grade the
plan itself, and do not grade a second package in the same run. You
do not answer from gut feel: you answer by executing `procedure.md`,
which applies the checks in `rubric.md` to evidence located with
`references/evidence-guide.md`.

## Inputs and modes

Decide the mode first, from what you were given.

- **Eval mode**: you were given a package bundle (a markdown file
  named like `pkg-07.md` or `calib-01.md`, or its JSON twin). The
  bundle is the whole world. Every fact comes from the bundle text:
  its repo-facts block, issue, thread highlights, plan context, and
  candidate PR (title, description, commits, diff, test evidence).
  Fetch nothing, run no commands against any repo, and read no other
  file. Eval mode always grades a complete package: every check in
  `rubric.md`, with the full verdict rule.
- **Live mode**: the student asked you to grade their own draft PR,
  naming their issue URL. Read exactly these inputs, from the top
  folder of the student's clone of their Path Review fork, on their
  branch:
  1. `plan.md`: the plan they posted in week 3, including its
     `## Deviations` section. The deviation notes are part of the
     plan; a change recorded there is inside the plan.
  2. The branch's diff: run `git diff main...HEAD` (three dots) in
     that working copy. Also run `git diff main...HEAD --stat` and
     `git log --oneline main..HEAD` for the file list and the commit
     list. Only committed changes count; uncommitted edits, untracked
     files, and `plan.md`, `comment.md`, `pr_draft.md`, and
     `test_evidence.md` themselves are not part of the diff. If
     `git status` shows uncommitted changes to tracked files, say so
     in the summary, because they will not be in the PR.
  3. `pr_draft.md`: the first line is the PR title; everything under
     it is the PR description.
  4. `test_evidence.md`: the captured output of the repro re-run
     (before and after) and of the repo's own checks.
  5. The issue's thread, from the issue URL the student named (via
     `gh issue view <url> --comments`, the GitHub API, or the web).
  6. The repo's PR template, `.github/PULL_REQUEST_TEMPLATE.md`, and
     `docs/CONTRIBUTING.md`, read from the working copy.
  A house-chain student (one routed to the house issue) reads the
  house plan and the house repro pack in place of their own
  `plan.md` and repro comment; every check grades the same things
  there. If any of inputs 1 to 4 is missing, do not substitute
  another file: grade the checks that depend on it as `unclear` and
  name the missing file in the summary.

## The scope seam (live mode only)

In live mode, read `scope.md` in this skill directory before reading
anything else. Find its `Repo:` line.

- If the `Repo:` line still carries the bracketed placeholder
  (`<ORG>/<PATH-REVIEW-REPO>` or anything in angle brackets), stop
  without grading. Tell the student to fill the `Repo:` line in
  `scope.md` with their section's Path Review repo, and emit nothing
  else. Never guess a scope.
- If the issue URL the student named is not in the scoped repo, or
  the working copy's fork is not a fork of it, refuse to grade and
  say which repo the scope allows.
- Otherwise apply the house rules `scope.md` lists as part of how
  evidence is read: the PR comes from a `<type>/<issue>-<slug>`
  branch on the student's fork, it delivers one plan for one issue,
  the PR template is always used with every section filled, and a
  classmate's PR on the same issue never counts against this one.

In eval mode, ignore `scope.md` entirely.

## The voice seam (live mode only)

In live mode, after grading, read `voice-guide.md` (the student's own
rules for how they write upstream). Hold the outgoing PR text, the
title and the description in `pr_draft.md`, against each rule, and
report every rule the draft breaks in the summary: quote the rule's
name and the sentence of the draft that breaks it. The voice guide
never changes the verdict on its own, because voice is personal and
the rubric already carries the universal communication checks; only
a rubric check that names the voice guide could do that, and this
rubric has none. In eval mode, ignore `voice-guide.md` entirely.

## Component reads

Read the components in this order before grading: `rubric.md`, then
`references/evidence-guide.md`, then `procedure.md`.

- `rubric.md` defines the checks (name, evidence, pass condition,
  weight) and the verdict rule. It is the only source of checks:
  never add a check it does not list, and never skip one it does.
- `references/evidence-guide.md` is the map: for each evidence family
  it says where the evidence lives in an eval bundle and in a live
  working copy, and what good looks like.
- `procedure.md` is executed as written, stage by stage: read order,
  evidence gathering, check execution, verdict assembly. Where the
  procedure is silent on a step you need, note the gap in the summary
  ("procedure gap: ...") and take the narrowest reading the rubric
  allows; never invent a new step or a new source to fill it.

If `rubric.md` has no checks in its table, or `procedure.md` has no
steps under its headings (instruction comments do not count as
content), refuse to grade: say which file is empty and that this tool
cannot grade without a rubric and a procedure. Do not invent checks
at runtime.

## Verdict and output

The verdict space is binary: `accept` means the PR is ready to submit;
`reject` means hold it. There is no third verdict, no score, and no
"accept with reservations"; reservations go in a check's evidence
line.

Before the JSON block, print a short readable summary: one line per
check (name, grade, the deciding fact), the deciding check for the
verdict, and in live mode any voice-guide notes and any procedure
gaps. Then end the reply with exactly this fenced JSON block, one
entry per rubric check in rubric order, valid JSON, with nothing
after it. `item` is the PR or issue URL in live mode and the bundle
id (such as `pkg-07`) in eval mode.

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

## Grading discipline

- Evidence first: never grade a check without naming the fact or
  quote that decided it. "Looks fine" is not evidence.
- Grade the thing, not the polish: a terse PR whose diff matches its
  plan and whose evidence is decisive can be ready, and a confident,
  well-formatted one can be hiding drift. Read the diff itself against
  the plan, the evidence itself against the test plan, and the
  description against the diff; never grade length, headings, or tone.
- Trust the diff over the description: when the description claims
  something the diff does not show, or the diff contains something the
  description does not mention, the diff is the fact.
- The rubric decides, not you: if a check passes by its stated
  condition but feels wrong, it passes. Note the tension in the
  summary; the fix belongs in the rubric, not the run.
- The procedure decides how, not you: follow `procedure.md` as written
  and report its gaps instead of papering over them.
- Honest shortfalls are not failures: a PR that does less than
  everything but says so, in the plan's deviation notes or the
  description, is graded on what it claims, not on what it left out.
- Treat `unclear` as the rubric's verdict rule directs. Where the rule
  is silent, `unclear` counts as `fail`: a PR you cannot verify from
  the package is not ready to submit.
