# Rubric: is this a good first issue?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| Maintainer activity | Last 5 default-branch commits (dates + authors) and maintainer first-response sample in the repo-facts block; live mode: same signals from the repo and recent issues | Pass if a non-bot commit landed within 120 days of capture/today, OR a maintainer responded to a sampled issue within 60 days. Otherwise fail. | required |
| Repository in use | `archived:` flag, last push to any branch, latest release in repo-facts; live mode: archive flag, branch activity, Releases panel | Pass only if not archived AND (last push OR latest release within 180 days). Otherwise fail. | required |
| Bounded newcomer scope | Issue body, labels, comment thread, linked PRs line; live mode: Development box + PRs mentioned in thread | Grade whether this is ONE deliverable, not how many files it touches. Test: if only one listed sub-item were done and the rest left undone, would the issue still be unresolved as a whole? If yes — every item is a necessary part of one goal — pass. Fail only if: (a) the issue calls itself an umbrella/tracking/meta/mega issue, or lists items that are independently valuable and each meant to ship as its own separate PR (not steps toward one goal); (b) the body or a maintainer states the work is codebase-wide or requires a core-architecture change; (c) it's a usage/support question, not a change request; (d) two or more people actively disagree on expected behavior with no maintainer resolution; (e) the issue is 2+ years old with 2+ closed-unmerged linked PRs; (f) it's a net-new feature with no maintainer/label backing. | required |
| Available to claim | `assignees` and `linked PRs` in repo-facts; every comment for claim language and whether a maintainer redirected it; live mode: same via GitHub | Pass if no assignee, no open linked PR, AND any claim comment in the thread is either resolved (maintainer redirected it) or stale — 90+ days old with no PR ever following it and no newer activity in the thread. Fail only when a claim comment is unresolved AND still within 90 days, or a maintainer states the issue is currently being worked. A closed/merged linked PR alone never fails this check. | required |
| Contribution policy compatible | Repo-facts contribution policy line; live mode: CONTRIBUTING.md, .github/, AI policy files, PR template | Pass if the policy is silent or allows AI-assisted contributions with conditions (disclosure, review, tests). Fail on an outright ban on AI-generated code/docs. | required |
| Maintainer responsiveness | Maintainer first-response sample (repo-facts) or live recent-issues sample | Preferred pass if 3+ of 5 sampled issues got a maintainer response within 30 days. Never changes the verdict — informational only. | preferred |

## Verdict rule

Accept only if every required check passes. A `?` (not enough evidence) on a required check counts as a fail — don't guess on a first contribution. The preferred check never blocks acceptance; use it only to break ties when ranking multiple accepted candidates in live mode. Verdict is binary: `accept` or `reject`.
