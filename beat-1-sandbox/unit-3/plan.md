# Plan: issue #68 — `KeywordSearcher.index([])` raises `ZeroDivisionError`

Issue: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/68
My reproduction: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/68#issuecomment-5903186167

## Diagnosis

The crash is at index time, not search time, and the evidence that pins
that down is the top frame of the traceback in my repro comment:

> ```
> Traceback (most recent call last):
>   File "<string>", line 4, in <module>
>     s.index([])
>     ~~~~~~~^^^^
>   File ".../rag/retriever/keyword_search.py", line 25, in index
>     self.bm25 = BM25Okapi(tokenized_corpus)
>   File ".../site-packages/rank_bm25.py", line 52, in _initialize
>     self.avgdl = num_doc / self.corpus_size
>                  ~~~~~~~~^~~~~~~~~~~~~~~~~~
> ZeroDivisionError: division by zero
> ```

`index()` at `keyword_search.py:24-25` tokenizes the corpus and hands it
straight to `BM25Okapi`. With an empty list, `corpus_size` is 0 and
`_initialize` divides by it. The `print(s.search(...))` on the next line
of my repro never executes — the object never finishes constructing.

Two control runs from the same repro comment fix the boundaries of that
cause:

- **Control 1 (one chunk instead of zero)** — `index()` and `search()`
  both succeed and return a scored result. So nothing about the module,
  the install, or the tokenizer is broken; the corpus being *empty* is
  the trigger, not indexing as such.
- **Control 2 (`index()` never called)** — `search()` returns `[]`
  cleanly and logs `keyword_search_empty_index`. So the empty-result
  path `search()` already implements works; it is simply unreachable
  when `index([])` raises before `search()` is called.

Taken together those rule out "search mishandles an empty index" and
leave exactly one cause: `index()` has no empty-corpus branch, so an
empty list reaches a library that cannot accept one. This matches the
issue body's own statement that `search()` handles the empty case and
`index()` should not raise either.

## Scope

**In scope:** one guard in `KeywordSearcher.index()` for an empty chunk
list, and removal of the `xfail` marker on the test that covers it, as
the issue body directs ("The covering test is marked
`@pytest.mark.xfail` referencing manifest id H-01 — remove the marker as
part of the fix").

**Not in scope**, deliberately, with reasons:

- Changing `search()`. Control 2 shows its empty guard already behaves
  correctly; touching it would be a change the evidence does not ask for.
- Validating chunk shape in `index()` (missing `text` keys, non-dict
  entries). Real, adjacent, and not this issue — a `KeyError` on a
  malformed chunk is a different defect and belongs in its own report.
- `rag/retriever/hybrid.py`, which also consumes this searcher. I have
  not shown it can pass an empty list, so changing it would be
  speculative; noted below as an unknown instead.
- Patching or pinning `rank-bm25` upstream. The library's behavior on an
  empty corpus is arguably reasonable; the defect is that our caller
  hands it one.

## Files I will touch

- `rag/retriever/keyword_search.py` — the guard in `index()`.
- `tests/unit/test_keyword_search.py` — remove the `@pytest.mark.xfail`
  block on `TestKeywordSearcher::test_empty_index`.

## Approach

1. In `index()`, before building the index, branch on an empty `chunks`
   list: store `self.chunks = chunks`, set `self.bm25 = None`, log the
   empty-index case, and return.
2. Setting `self.bm25 = None` is the deliberate part. It makes a
   searcher that was handed an empty corpus land in exactly the state
   `search()`'s existing guard (`if not self.bm25 or not self.chunks`)
   already handles, so `search()` returns `[]` with no change to
   `search()` at all. It also clears a previously built index rather
   than leaving a stale one behind, so `index(chunks)` then `index([])`
   does not keep answering from the old corpus.
3. Remove the `@pytest.mark.xfail(strict=True, reason=...)` decorator
   from `test_empty_index`. Because the marker is `strict=True`, leaving
   it on after the fix makes CI fail with `XPASS(strict)`, so removing
   it is part of the fix, not a tidy-up.
4. Run the full `tests/unit/test_keyword_search.py` file to confirm the
   other 16 tests are unchanged.

## Test plan

Re-run the exact steps from my repro comment against the built change
and record before/after for each.

1. **The trigger.** `index([])` then `search('python')`, the snippet
   from my repro. Before: `ZeroDivisionError: division by zero` raised
   at `keyword_search.py:25`. After: no traceback, and the process
   prints `[]` — the return value `search()` was always supposed to
   give.
2. **Control 1 unchanged.** One chunk indexed, then searched. Before and
   after must both print the same one-result list with a `bm25_score`
   key. This is the check that the guard did not break ordinary
   indexing.
3. **Control 2 unchanged.** `search()` without `index()`. Before and
   after must both print `[]` and log `keyword_search_empty_index`.
4. **The seeded test flips.** `pytest tests/unit/test_keyword_search.py
   -q -rx`. Before: `16 passed, 1 xfailed`, with `test_empty_index`
   listed as XFAIL. After, with the marker removed: `17 passed`, no
   xfail line. If I removed the marker without fixing the bug, this
   would read `16 passed, 1 failed`, so the line distinguishes the fix
   from the marker removal.

The observable that decides success is step 1 printing `[]` where it
previously raised, and step 4 reading `17 passed`.

## Risks and unknowns

- **Is `None` the right empty state, or should `index([])` build a real
  empty index object?** I chose `None` because it reuses the guard that
  already exists and that Control 2 shows working. A maintainer may
  prefer `index()` to raise a clear `ValueError` instead of silently
  accepting an empty corpus; the issue body says it "shouldn't raise",
  so I am following that, but I will flag the alternative in the PR.
- **I do not know whether an empty chunk list reaches `index()` in
  normal operation.** I have not traced callers. If it only happens in
  tests, this is a robustness fix; if an ingestion run that produced no
  chunks can reach it, it is a live crash. Either way the fix is the
  same, so I am not blocking on the answer.
- **`rag/retriever/hybrid.py`** also uses `KeywordSearcher`. Untested by
  me; out of scope above, and listed here so it is not mistaken for
  something I checked.
- I have only observed this on macOS 26.6.2 / Python 3.13.9. The failing
  line is integer division inside a pure-Python library, so I do not
  expect platform or version sensitivity, but I have not verified it.

## Deviations

The built change matches the plan: the guard landed in
`KeywordSearcher.index()` exactly as described in Approach step 1-2
(`self.chunks = chunks`, `self.bm25 = None`, log, return), the `xfail`
marker came off `test_empty_index`, and no other file was touched. The
diff is 10 added lines and 4 removed, across the two files named under
"Files I will touch". Three things are worth recording anyway.

1. **I named the new log event `keyword_index_empty`.** The plan said
   only "log the empty-index case" and did not pick a name. I used a
   new event rather than reusing `keyword_index_built` with
   `chunk_count=0`, so an empty index is distinguishable in the logs
   from a one-chunk one. Small, but it was a decision the plan left
   open, so it is recorded here rather than only in the diff.

2. **I ran one check the test plan did not list.** Approach step 2
   claimed the guard "clears a previously built index rather than
   leaving a stale one behind", but my test plan never actually tested
   that claim — all four of its steps start from a fresh searcher. So I
   added a fifth run: `index([{...}])`, then `index([])`, then
   `search()`. It prints `[]`, confirming the stale index is cleared. In
   hindsight that check should have been in the test plan when I posted
   it, because the plan asserted the behavior; writing the claim and
   forgetting to test it is the gap this note is recording.

3. **The test plan's step 4 was narrower than "run the suite".** I ran
   `tests/unit/test_keyword_search.py` (17 passed) as planned, and also
   tried the whole `tests/unit` directory. That interrupts with 6
   collection errors — `ModuleNotFoundError` for `jose`, `pypdf`,
   `redis`, `sqlalchemy`, and `tiktoken`. I verified these are not mine:
   stashing my change and re-running on clean `main` gives the identical
   "6 errors during collection". They are missing dependencies in the
   minimal venv I deliberately built (see the repro comment's
   deviation 2), not a regression. Full-suite verification therefore
   still belongs to CI, and I have not claimed it here.

Nothing in the posted plan comment is now untrue, so no follow-up
correction comment is needed on the issue.
