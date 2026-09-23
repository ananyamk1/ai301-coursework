# Unit 1: Issue Selection

## Selected issue
https://github.com/codepath/pathreview-ai301-fa26-s1/issues/68 — "Keyword search raises ZeroDivisionError when the index is empty"

## Skill's verdict output
```json
{
  "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/68",
  "checks": [
    {"name": "Maintainer activity", "grade": "pass", "evidence": "Human commit by Andrew Burke on Sep 16, 2026, 6 days before today"},
    {"name": "Repository in use", "grade": "pass", "evidence": "Not archived; last push Sep 16, 2026, within 180 days"},
    {"name": "Bounded newcomer scope", "grade": "pass", "evidence": "Single bug: add empty-corpus guard to index(); existing xfail test is the acceptance criterion"},
    {"name": "Available to claim", "grade": "pass", "evidence": "No assignee, no linked PR; yulijasso's Sep 20 claim is a classmate comment — Path Review house rule makes it non-blocking"},
    {"name": "Contribution policy compatible", "grade": "pass", "evidence": "No CONTRIBUTING.md or AI policy file found; silence passes"},
    {"name": "Maintainer responsiveness", "grade": "unclear", "evidence": "5-issue sample shows no detectable maintainer comments from list view"}
  ],
  "verdict": "accept"
}
```

## Run history
1. Filled the empty `rubric.md` template with 5 required checks (maintainer activity, repository in use, bounded newcomer scope, available to claim, contribution policy) plus 1 preferred check (maintainer responsiveness), covering all 5 eval categories.
2. First full eval run: 15/20 agreement. Categories: claimed 4/4, clear-accept 4/8, dead-repo 3/3, policy 1/1, scope 3/4.
3. Used a diagnostic script to pull full evidence text on the 5 disagreements (issue-01, 04, 09, 19, 15) instead of guessing from pass/fail alone.
4. Found two check-wording problems: "Bounded newcomer scope" was rejecting multi-file tasks that were actually one coherent goal, and "Available to claim" was rejecting issues over old claim comments that were never followed by a PR.
5. Rewrote both pass conditions — scope now asks "is this one deliverable," and claim status now treats a stale (90+ day), unfollowed claim as non-blocking.
6. Re-ran just the 5 disputed issues: 5/5 agreement, including issue-15 (a false-accept) correcting to match gold too.
7. Ran the full clean eval: 20/20 agreement, all 5 categories perfect. Saved via `--save-run eval-run.txt`.
8. Ran live mode on 3 open Path Review issues (#37, #47, #68). All three accepted; selected #68.

## Issue analysis
issue-09 (gold: accept). Before the rubric fix, my rubric said reject — "Available to claim" failed because a user named MesaJonathan had commented "I'd like to take a swing at this" back in 2022, and that claim was never withdrawn or turned into a PR. My original check treated any unresolved claim comment as disqualifying, with no way to recognize that a 3-year-old claim with no follow-up is effectively abandoned. After rewriting the check to allow a stale, unfollowed claim to pass, the verdict corrected to accept, matching gold.

## Check rationale
The current wording of "Available to claim" in my rubric.md:
> Pass if no assignee, no open linked PR, AND any claim comment in the thread is either resolved (maintainer redirected it) or stale — 90+ days old with no PR ever following it and no newer activity in the thread. Fail only when a claim comment is unresolved AND still within 90 days, or a maintainer states the issue is currently being worked. A closed/merged linked PR alone never fails this check.

## Trade-offs
Loosening "Available to claim" to ignore stale claims risks missing someone who's genuinely still working slowly; loosening "Bounded newcomer scope" to judge by "one deliverable" instead of file count risks accepting an issue that's technically one goal but still too large for a true first-timer.

## Selection rationale
I picked #68 over #37 and #47 (both accepted docs issues) because it has an existing `xfail` test as its acceptance criterion — when the empty-index guard is added and the marker removed, the test turns green, so "done" is unambiguous going into Unit 2's reproduction step. The docs issues are fine but their scope is closer-ended by writeup quality rather than a hard test, which is harder to verify objectively as a first contribution.
