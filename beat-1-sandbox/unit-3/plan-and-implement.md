# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

Issue: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/68 — "Keyword search
raises `ZeroDivisionError` when the index is empty"

---

## Posted upstream

**GitHub username**

ananyamk1

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/68#issuecomment-5920731657

Following up on [my reproduction above](https://github.com/codepath/pathreview-ai301-fa26-s1/issues/68#issuecomment-5903186167) with the plan I intend to build, in case any of it should go differently before I start. Posting my own rather than piggybacking on the plans already in this thread, per the course's house rule.

**Diagnosis.** The two controls in my repro report are what settle this. Indexing one chunk works normally, so the module and the tokenizer are fine and *empty* is the trigger. Calling `search()` without ever calling `index()` gives:

```
[warning  ] keyword_search_empty_index
[]
```

so the empty-result path already works — it is just unreachable, because `index([])` raises at `keyword_search.py:25` before `search()` is ever called. That rules out "search mishandles an empty index" and leaves one cause: `index()` has no empty-corpus branch.

**Scope.** One guard in `KeywordSearcher.index()`, plus removing the `@pytest.mark.xfail` marker on `test_empty_index`, as the issue body directs. Deliberately not in scope: `search()` (control 2 shows it already correct), chunk-shape validation such as missing `text` keys (a different defect, and it deserves its own issue), and `rag/retriever/hybrid.py` (I have not shown it can pass an empty list, so changing it would be speculative).

**Approach.** When `chunks` is empty, `index()` records the empty list, sets `self.bm25 = None`, logs, and returns. Setting it to `None` is the deliberate bit: it lands the searcher in exactly the state `search()`'s existing guard (`if not self.bm25 or not self.chunks`) already handles, so `search()` needs no change at all, and it clears a previously built index rather than leaving a stale one answering queries.

**Test plan.** I re-run all three snippets from my repro comment and record before/after. The deciding observable is that the first one prints `[]` where it previously raised `ZeroDivisionError`. Then `pytest tests/unit/test_keyword_search.py -q -rx` should go from `16 passed, 1 xfailed` to `17 passed` — and because the marker is `strict=True`, that line also distinguishes an actual fix from merely deleting the marker.

**On the overlap with @yulijasso's plan above.** I reached the same guard — including `self.bm25 = None` to clear a stale index — independently from my own reproduction before reading theirs closely, and I am posting mine per the house rule rather than piggybacking. Worth saying out loud so it reads as convergence rather than a copy: if two of us landed on the same three lines from separate repros, that is probably the right shape for the fix.

**One open question for whoever reviews this.** I have `index([])` accept the empty corpus silently, because the issue says it "shouldn't raise". The alternative is for it to raise a clear `ValueError` instead of a library-internal `ZeroDivisionError`. I am going with the issue's reading, but say so if you'd rather have the explicit error. I also have not traced whether an empty chunk list reaches `index()` in normal operation or only in tests; the fix is the same either way, so I am not blocking on it. And I have only run this on macOS 26.6.2 with Python 3.13.9 — the failing line is plain division inside a pure-Python library, so I would not expect platform sensitivity, but I have not checked.

Disclosure, same as on my earlier comments: I am a student on CodePath's AI301 course and I draft with an AI assistant in the loop. I have run and checked everything I post here myself.

---

## Your branch

**Branch**

`fix/68-empty-index-guard`

**Evidence**

All four steps of the plan's test plan, re-run against the built change. Environment:
macOS 26.6.2 (arm64), Python 3.13.9, `rank-bm25` 0.2.2, `structlog` 26.1.0, `pytest`
9.1.1, in the fork clone at branch `fix/68-empty-index-guard` (base commit `f89c06f`).

### 1. The trigger — `index([])` then `search()`

Command (identical before and after):

```
$ .venv-repro/bin/python -c "
from rag.retriever.keyword_search import KeywordSearcher
s = KeywordSearcher(); s.index([]); print(s.search('python', top_k=10))"
```

Before:

```
  File ".../scratchpad/pathreview-fork/.venv-repro/lib/python3.13/site-packages/rank_bm25.py", line 27, in __init__
    nd = self._initialize(corpus)
  File ".../scratchpad/pathreview-fork/.venv-repro/lib/python3.13/site-packages/rank_bm25.py", line 52, in _initialize
    self.avgdl = num_doc / self.corpus_size
                 ~~~~~~~~^~~~~~~~~~~~~~~~~~
ZeroDivisionError: division by zero
```

After:

```
2026-09-30 17:18:21 [info     ] keyword_index_empty            chunk_count=0
2026-09-30 17:18:21 [warning  ] keyword_search_empty_index
[]
```

This is the deciding observable: the call that raised now prints `[]`, and it reaches
`search()`'s pre-existing guard, which logs `keyword_search_empty_index`.

### 2. Control 1 — one chunk indexed, must be unchanged

```
$ .venv-repro/bin/python -c "
from rag.retriever.keyword_search import KeywordSearcher
s = KeywordSearcher(); s.index([{'id': 1, 'text': 'python content'}]); print(s.search('python', top_k=10))"
```

Before:

```
2026-09-30 17:17:57 [info     ] keyword_index_built            chunk_count=1
2026-09-30 17:17:57 [info     ] keyword_search_complete        query_len=1 results_count=1
[{'id': 1, 'text': 'python content', 'bm25_score': -0.2746530721670274}]
```

After:

```
2026-09-30 17:18:21 [info     ] keyword_index_built            chunk_count=1
2026-09-30 17:18:21 [info     ] keyword_search_complete        query_len=1 results_count=1
[{'id': 1, 'text': 'python content', 'bm25_score': -0.2746530721670274}]
```

Identical, including the score. Ordinary indexing is untouched.

### 3. Control 2 — `search()` without `index()`, must be unchanged

```
$ .venv-repro/bin/python -c "
from rag.retriever.keyword_search import KeywordSearcher
print(KeywordSearcher().search('python', top_k=10))"
```

Before:

```
2026-09-30 17:17:57 [warning  ] keyword_search_empty_index
[]
```

After:

```
2026-09-30 17:18:21 [warning  ] keyword_search_empty_index
[]
```

Identical. `search()` was not modified.

### 4. The seeded test flips

```
$ .venv-repro/bin/python -m pytest tests/unit/test_keyword_search.py -p no:cacheprovider -q -rx
```

Before:

```
........x........                                                        [100%]
=========================== short test summary info ============================
XFAIL tests/unit/test_keyword_search.py::TestKeywordSearcher::test_empty_index - issue #68 (manifest H-01): BM25 keyword search raises ZeroDivisionError on an empty index
16 passed, 1 xfailed in 0.08s
```

After (with the `xfail` marker removed):

```
.................                                                        [100%]
17 passed in 0.06s
```

`16 passed, 1 xfailed` → `17 passed`. Because the marker was `strict=True`, removing it
without fixing the bug would have printed `16 passed, 1 failed`, so this line separates
the fix from the marker removal.

### 5. Extra check, added during the build (see Deviations in `plan.md`)

The plan claimed the guard clears a previously built index, but the test plan never
tested it — every step started from a fresh searcher. So:

```
$ .venv-repro/bin/python -c "
from rag.retriever.keyword_search import KeywordSearcher
s = KeywordSearcher()
s.index([{'id': 1, 'text': 'python content'}])
s.index([])
print(s.search('python', top_k=10))"
2026-09-30 17:18:21 [info     ] keyword_index_empty            chunk_count=0
2026-09-30 17:18:21 [warning  ] keyword_search_empty_index
[]
```

`[]`, not the stale one-chunk result. The claim holds.

### Note on the wider suite

`pytest tests/unit` interrupts with 6 collection errors (`ModuleNotFoundError` for
`jose`, `pypdf`, `redis`, `sqlalchemy`, `tiktoken`). These are not mine: stashing the
change and re-running on clean `main` gives the identical "6 errors during collection".
They are missing dependencies in the deliberately minimal venv, so full-suite
verification belongs to CI and I am not claiming it.

## Eval iterations

**Run history**

1. **`--only pkg-02,pkg-04,pkg-09,pkg-15,pkg-20` — 5/5 agreement** (about $1). A
   deliberate smoke run before spending on a full one, on the five packages I judged my
   rubric most likely to get wrong: a terse clear-accept that a structure-shaped rubric
   would reject (pkg-02), both members of the 2-package `thread-convention` category
   (pkg-04, ignoring the owner's located culprit; pkg-20, missing AI disclosure), an
   arguable accept that deliberately scopes itself down (pkg-09), and an arguable
   scope-creep reject whose core fix is correct (pkg-15). Categories printed
   `clear-accept 2/2  scope-creep 1/1  thread-convention 2/2`. Seeing the 2-package
   category clear at $0.20 each rather than at $4 was the point of running it first.
2. **Full run — 19/20 agreement, bar 18/20: PASS**, every category matched
   (`clear-accept 6/7  scope-creep 4/4  thread-convention 2/2  unbuildable 3/3
   wrong-cause 4/4`). This is the run saved in `eval-run.txt`. One disagreement, pkg-14,
   analysed below.

**Package analysis**

**pkg-14** (zellij-org/zellij#5174 — colour codes leaking into panes on SSH reattach).
Gold label: **accept**. My rubric said **reject**.

It failed one required check, `executable-by-stranger`, and the harness recorded the
quote that decided it:

> "exact functions to be pinned in the PR after tracing the query issuance with debug
> logs" — no concrete file or call site named, decision deferred to build-time
> investigation.

Everything else passed: the diagnosis is consistent with both controls (0.44.1 clean
across 5 cycles, and a cache-clear giving one clean attach before it leaks again), the
scope is bounded to the Unix reattach handshake with the Windows variant explicitly
deferred, and the test plan is decisively observable ("5 consecutive SSH reattach cycles
with no rgb strings in any pane").

My rubric read it that way because I wrote `executable-by-stranger` around a literal
test: at least one concrete file or call site must be named, and the change at it
described as a decision already taken. pkg-14 names *areas* — "the client
attach/reattach path in `zellij-server` (session connection handling) and
`zellij-client`'s terminal query issuance" — and then says the exact functions get
pinned later. Under my pass condition that is deferral, so it failed.

Rereading it, I think gold is right and my check is a shade too literal here. Naming two
crates and the subsystem inside each, plus a working method for finding the site (the
leak's origin is visible in `zellij --debug`), is meaningfully different from pkg-17's
"gocui? tcell? not sure" or pkg-18's "recover() somewhere". pkg-14's author has chosen
the layer and knows how to locate the line; pkg-17's and pkg-18's have not chosen
anything. My check cannot currently tell "area named, line not yet pinned" apart from
"layer not chosen", and the `unbuildable` category is built around the second one.

**Check rationale**

Quoting `test-plan-decisive` from the `rubric.md` I uploaded to `tools/plan-check/` (the
file fingerprinted `sha256:ca13d0d41ba6d7c3` in the committed `eval-run.txt` header):

> **Evidence:** The plan's test plan, read against the repro evidence's steps and
> artifact.
>
> **Pass condition:** Pass if it names an observable outcome specific to this fix that a
> reader could check and that would differ before and after the change: a value, an exit
> status, a rendered colour or string, a named test case, a specific line of output. Fail
> if the only stated outcome is that an existing suite passes or nothing regresses, or if
> the outcome is subjective ("should feel fast", "nothing else should feel broken") —
> those are true of a build that does not fix anything.

It reads that way because my first draft asked whether the plan "has a test plan that
follows from the repro evidence", and `calib-04` showed me that condition is worth
nothing. That package is a genuinely good plan — bounded to the absolute-prefix case,
grounded in the owner's analysis, naming `crates/core/flags/hiargs.rs`, engaging the
thread's note that the deep fix is hard — and its entire test plan is "Run the full test
suite (`cargo test --workspace`) and make sure nothing regresses." That *does* follow
from the repro evidence in any loose sense, and it would have passed my first draft. It
is also exactly true of a build that changes nothing at all.

So I rewrote the condition around a single question: is there something stated here that
would read *differently* before and after the fix? That reframing is what makes
`calib-04` and pkg-10 ("should feel fast") and pkg-17 ("nothing else should feel broken")
fail under one rule, while pkg-02's "repro re-run with expected exit 0" and pkg-13's
script-based check against a reference terminal pass. The clause naming the failing forms
explicitly — a suite passing, nothing regressing, subjective phrasing — is there because
those are the three shapes that look like a test plan without being one.

It also shaped my own plan: my test plan's deciding observable is the repro snippet
printing `[]` where it raised, and `16 passed, 1 xfailed` becoming `17 passed` — and,
because the marker is `strict=True`, that second line distinguishes a real fix from
merely deleting the marker.

**Trade-offs**

The trade-off I kept is the one that costs me pkg-14. `executable-by-stranger` demands a
named file or call site, and that literalism is the whole of my 19/20 rather than 20/20.

I could loosen it — "a named file **or a named subsystem plus a stated method for
locating the site**" — and pkg-14 would flip to accept. I did not, for two reasons.
First, that clause is close to what pkg-18 already says: "fix upstream or vendored,
whichever is easier", "maybe also check other linters" is a stated method of sorts, and
`unbuildable` is a 3-package category I currently match 3/3. The distance between "I know
which crate and how to find the function" and "I'll work it out at build time" is real
but narrow, and I could not write a pass condition that separates them without importing
a judgment call about how confident the author sounds — which is the kind of check that
makes two graders disagree. Second, the literal version is what made my own plan better:
it is why I named `rag/retriever/keyword_search.py` and the exact `index()` branch instead
of writing "the retriever's indexing path".

Per the eval README's canary rule, the honest way to test that loosening would be a cheap
`--only pkg-14,pkg-18,pkg-10,pkg-17,pkg-04` run — the three `unbuildable` packages the
change could flip, plus one `thread-convention` canary — before spending another $4 on a
confirming full run. I am recording that as the move I would make rather than one I made:
the bar and the category floor were already clear at 19/20, and the run I committed is a
rubric whose single miss I can explain.

What it costs in real terms: my rubric will hold a plan that has genuinely chosen its
layer but has not yet opened the file to find the function name. I accept that. For a
first contribution the cost of being asked to name the line before posting is small, and
the cost of a plan that defers the real decision is a week of nobody being able to review
it.

---

Related paths: `plan.md` and `eval-run.txt` in this directory; your skill's files in
`tools/plan-check/`.
