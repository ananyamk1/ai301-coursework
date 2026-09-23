# Rubric: is this a good first issue?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| Maintainer activity | Last 5 default-branch commits (dates + authors) and maintainer first-response sample in the repo-facts block; live mode: same signals from the repo and recent issues | Pass if a non-bot commit landed within 120 days of capture/today, OR a maintainer responded to a sampled issue within 60 days. Otherwise fail. | required |
| Repository in use | `archived:` flag, last push to any branch, latest release in repo-facts; live mode: archive flag, branch activity, Releases panel | Pass only if not archived AND (last push OR latest release within 180 days). Otherwise fail. | required |
| Bounded newcomer scope | Issue body, labels, comment thread, linked PRs line; live mode: Development box + PRs mentioned in thread | Pass if the issue describes one identifiable fix/change with enough detail to start (a named file, function, or symptom). Fail if it's an umbrella/tracking issue, a list of independently-titled sub-items, requires a codebase-wide or core-architecture change, is a support question rather than a change request, or is a net-new feature request with no maintainer/label backing. | required |
| Available to claim | `assignees` and `linked PRs` fields in repo-facts; every comment for claim language ("I'll take this," "working on it") and whether a maintainer redirected it; live mode: same via GitHub | Pass only if no assignee, no open linked PR, and no unresolved claim comment. A closed/merged linked PR alone doesn't fail it. | required |
| Contribution policy compatible | Repo-facts contribution policy line; live mode: CONTRIBUTING.md, .github/, AI policy files, PR template | Pass if the policy is silent or allows AI-assisted contributions with conditions (disclosure, review, tests). Fail on an outright ban on AI-generated code/docs. | required |
| Maintainer responsiveness | Maintainer first-response sample (repo-facts) or live recent-issues sample | Preferred pass if 3+ of 5 sampled issues got a maintainer response within 30 days. Never changes the verdict — informational only. | preferred |

## Verdict rule

Accept only if every required check passes. A `?` (not enough evidence) on a required check counts as a fail — don't guess on a first contribution. The preferred check never blocks acceptance; use it only to break ties when ranking multiple accepted candidates in live mode. Verdict is binary: `accept` or `reject`.