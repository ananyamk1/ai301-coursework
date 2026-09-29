# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

Issue: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/68 — "Keyword search
raises ZeroDivisionError when the index is empty" (selected in Unit 1)

---

## Your identity upstream

**GitHub username**

ananyamk1

---

## Posted upstream

**Claim comment**

<!-- TODO: paste the permalink of your posted claim comment here (Copy link on the
comment's ••• menu), replacing this line. The text below is what I posted. -->

I'd like to pick this one up as a first contribution. (Posting my own claim per the course's house rule, rather than waiting on the claims already in this thread.)

To confirm I am reading the issue the way it was filed: the failure is at index time, not search time. `search()` already guards the empty case —

```python
if not self.bm25 or not self.chunks:
    logger.warning("keyword_search_empty_index")
    return []
```

— which is why `test_index_not_called_returns_empty` passes today, while `index([])` hands an empty `tokenized_corpus` to `BM25Okapi` and never reaches that guard. So a fix belongs in `index()`, not in `search()`.

Next step for me: set up the repo locally, reproduce the `ZeroDivisionError` on my own machine, and post the full reproduction back in this thread — environment, exact steps, and the traceback — before I open anything. `tests/unit/test_keyword_search.py::TestKeywordSearcher::test_empty_index` carries the `xfail(strict=True)` marker naming this issue, so per `docs/CONTRIBUTING.md` I am treating that test turning green, with the marker removed, as the acceptance criterion.

I am not promising a fix or a date yet — I want to see the failure myself first, and I will report back either way, including if I cannot reproduce it.

For transparency: I am a student working through CodePath's AI301 course, and I draft with an AI assistant in the loop. Every command and output I post is one I ran and checked myself.

**Reproduction comment**

<!-- TODO: paste the permalink of your posted reproduction comment here, replacing this
line. The text below is what I posted. -->

Reproduced. The `ZeroDivisionError` is raised by `index()`, before `search()` is ever called, so `search()`'s empty guard never gets a chance to run.

**Environment**

- macOS 26.6.2 (arm64, Apple Silicon)
- Python 3.13.9 in a fresh venv
- pathreview at commit `f89c06f` (`main`)
- `rank-bm25` 0.2.2, `numpy` 2.5.3, `structlog` 26.1.0, `pytest` 9.1.1

Two deviations from `docs/SETUP.md`, both deliberate, both flagged in case either matters:

1. `docs/SETUP.md` lists Python 3.11 and `pyproject.toml` pins `requires-python = ">=3.11"`; I ran 3.13.9. The failing line is plain integer division inside `rank_bm25`, so I would not expect the interpreter version to change it, but I have only observed 3.13.9.
2. I did not run `make setup` or bring up Docker. `rag/retriever/keyword_search.py` imports only `structlog` and `rank_bm25`, and nothing on this path touches the database, the API, or any service, so I installed just those into a venv:

```
$ python3.13 -m venv .venv-repro
$ .venv-repro/bin/pip install "rank-bm25>=0.2.2" pytest structlog
```

**Steps**

From the repo root, with that venv:

```
$ .venv-repro/bin/python -c "
from rag.retriever.keyword_search import KeywordSearcher
s = KeywordSearcher()
s.index([])
print(s.search('python', top_k=10))
"
```

**What happened**

```
Traceback (most recent call last):
  File "<string>", line 4, in <module>
    s.index([])
    ~~~~~~~^^^^
  File ".../rag/retriever/keyword_search.py", line 25, in index
    self.bm25 = BM25Okapi(tokenized_corpus)
                ~~~~~~~~~^^^^^^^^^^^^^^^^^^
  File ".../site-packages/rank_bm25.py", line 83, in __init__
    super().__init__(corpus, tokenizer)
    ~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^
  File ".../site-packages/rank_bm25.py", line 27, in __init__
    nd = self._initialize(corpus)
  File ".../site-packages/rank_bm25.py", line 52, in _initialize
    self.avgdl = num_doc / self.corpus_size
                 ~~~~~~~~^~~~~~~~~~~~~~~~~~
ZeroDivisionError: division by zero
```

Expected: `search()` returns `[]` on an empty index, the way it already does when `index()` was never called.

Actual: the traceback above. Note the top frame — it is `s.index([])`, not `s.search(...)`. The `print(s.search(...))` line never executes. `corpus_size` is 0, so `num_doc / self.corpus_size` divides by zero while the index is being built.

**Two control runs**

Same venv, same commit, changing one thing each time.

Control 1 — one chunk instead of zero. Indexing and searching both succeed, so nothing about the setup is broken:

```
$ .venv-repro/bin/python -c "
from rag.retriever.keyword_search import KeywordSearcher
s = KeywordSearcher()
s.index([{'id': 1, 'text': 'python content'}])
print(s.search('python', top_k=10))
"
2026-09-29 13:51:52 [info     ] keyword_index_built            chunk_count=1
2026-09-29 13:51:52 [info     ] keyword_search_complete        query_len=1 results_count=1
[{'id': 1, 'text': 'python content', 'bm25_score': -0.2746530721670274}]
```

Control 2 — skip `index()` entirely and search. This is the case `search()`'s guard covers, and it returns cleanly:

```
$ .venv-repro/bin/python -c "
from rag.retriever.keyword_search import KeywordSearcher
print(KeywordSearcher().search('python', top_k=10))
"
2026-09-29 13:51:52 [warning  ] keyword_search_empty_index
[]
```

Together those two isolate the trigger: it is not an empty *search*, it is `index()` being handed an empty list. An index built from zero chunks is unreachable through the existing guard because the object never finishes constructing.

**The seeded test**

`tests/unit/test_keyword_search.py::TestKeywordSearcher::test_empty_index` is the test that covers this, and it currently xfails exactly as its marker says:

```
$ .venv-repro/bin/python -m pytest tests/unit/test_keyword_search.py -q -rx
........x........                                                        [100%]
=========================== short test summary info ============================
XFAIL tests/unit/test_keyword_search.py::TestKeywordSearcher::test_empty_index - issue #68 (manifest H-01): BM25 keyword search raises ZeroDivisionError on an empty index
16 passed, 1 xfailed in 0.15s
```

**What I did not check**

I have not looked at callers, so I do not know whether an empty chunk list reaches `index()` in normal operation (say, from an ingestion run that produced no chunks) or only in tests. I also have not tried this on Linux or on Python 3.11, and I have not touched `rag/retriever/hybrid.py`, which also uses this searcher.

Next I want to work out where the guard belongs — an early return in `index()` that leaves `self.bm25` as `None` so the existing `search()` guard catches it looks like the smallest change, but I would rather ask than assume: is leaving `self.bm25 = None` on an empty corpus the behavior you want, or should `index()` raise something more informative than a library-internal `ZeroDivisionError`?

Disclosure, same as in my claim above: I am a student on CodePath's AI301 course and I draft with an AI assistant in the loop. Every command and every output pasted here is one I ran on this machine.

## Eval iterations

**Run history**

1. **`--only pkg-09,pkg-11,pkg-16,pkg-17,pkg-20` — 5/5 agreement** (about $1). Not a
   revise loop: a deliberate smoke run on the five packages I expected my rubric to be
   most fragile on, before spending $4. I picked one honest cannot-reproduce that must
   come out `accept` (pkg-09), one terse-but-complete accept (pkg-11), one silent version
   deviation that must come out `reject` (pkg-16), one polished wrong-target (pkg-17), and
   the single disclosure-wall package that the category floor exists for (pkg-20).
   Categories printed `clear-accept 2/2  disclosure 1/1  wrong-target 2/2`. Seeing
   disclosure 1/1 at $0.20 rather than at $4 is the whole reason I ran this first.
2. **Full run — 19/20 agreement, bar 18/20: PASS**, all five categories matched
   (`clear-accept 7/8  disclosure 1/1  no-evidence 4/4  unfollowable-comms 3/3
   wrong-target 4/4`). This is the run saved in `eval-run.txt`. One disagreement, pkg-05,
   analysed below. I stopped here deliberately rather than chasing 20/20: the change that
   would flip pkg-05 loosens a check in a direction I do not want (see **Trade-offs**),
   and the bar and the category floor were both already clear.

**Package analysis**

**pkg-05** (conda/conda#16543 — the `EnvironmentSectionNotValid` warning printed to stdout
and breaking `--json` output). Gold label: **accept**. My rubric said **reject**.

It failed on exactly one required check, `steps-rerunnable`, and the harness recorded why:

> Steps say only 'wrote a minimal env.yml containing a valid dependencies: list plus a
> category: section' — the actual file contents/syntax are never quoted, so a stranger
> cannot recreate the exact trigger file, only a paraphrase of it.

(It also failed my `control-or-contrast-run` check — no run without the `category:` section
— but that one is `preferred` and never moves the verdict, which is the point of having it
as preferred.)

My rubric read it that way because I wrote `steps-rerunnable` around a literal test: every
input the steps consume is *given inline* or publicly obtainable. pkg-05's report consumes a
file it never shows. The other seven checks all passed — the artifact is the issue's exact
error text above the JSON blob, plus a `json.tool` parse failure, and the version is the
reporter's own 26.7.0 — so this is a single check deciding the whole verdict.

Having reread the bundle, I think the gold label is right and my check is slightly too
literal here. "A valid `dependencies:` list plus a `category:` section" is, for conda
specifically, close to a recipe: any reader who knows the format can write that file in
four lines, and the artifact shows the file path (`env.yml`) and the exact command. The
distance between that and a pasted file is smaller than the distance between it and
pkg-18's "our internal monorepo with an unshared config", which is what I built the check
to catch. My check cannot currently tell those two apart, and it should.

**Check rationale**

Quoting `conditions-match-or-deviation-named` from `tools/repro-check/rubric.md` as it now
reads (the `rubric.md` fingerprinted `sha256:9083bae528ba9686` in the committed
`eval-run.txt` header):

> **Evidence:** The version, branch, platform, and configuration the report actually
> tested, compared against the version/branch/platform the issue and its thread establish
> the bug on (including any "confirmed on latest and main" template ticks in the issue
> context).
>
> **Pass condition:** Pass if the tested conditions fall inside the range the issue
> establishes, OR if they differ and the report itself names the difference in its own
> text. A stated delta ("filed against 13.0.0, unchanged on 15.2.0") passes. Fail if the
> report tested a version or platform outside the issue's established range and never says
> so, so the reader cannot tell whether the artifact is about the reported bug at all.

It reads that way because the first version I wrote was "pass if the version tested matches
the version the issue was filed against", and reading the packages showed that condition
punishes good reports and misses the bad one. pkg-03 (ripgrep) tested 15.2.0 against an
issue filed on 13.0.0, and pkg-12 (prettier) and pkg-07 (p5.js) both tested newer releases
than the issue named — all three are gold `accept`, because each one says so in the report
("filed against 13.0.0; behavior is unchanged on 15.2.0"). Meanwhile pkg-16 (pandas) is
gold `reject` for testing 1.5.3 against an issue whose template ticks confirm it on latest
and main — and 1.5.3 *is* a version the bug could plausibly be discussed on, so a
matching-versions rule would not catch it either.

So the thing being graded is not the version number at all. It is whether the report
**discloses its own deviation**, because a stated delta lets the reader judge the artifact
and a silent one does not. I rejected "versions must match" in favour of "the deviation is
named", and that single rewrite is what makes pkg-03, pkg-07, pkg-12 and pkg-16 all come
out right under one rule instead of four exceptions. It also decided how I wrote my own
repro comment above: I ran Python 3.13.9 against a repo pinned at `>=3.11`, so I flagged it
rather than quietly hoping it did not matter.

**Trade-offs**

The trade-off I chose to keep is the one that costs me pkg-05. `steps-rerunnable` demands
that inputs be *given*, not *described*, and that is why my 19/20 is not 20/20.

I could loosen it — "or described precisely enough that a reader familiar with the format
could recreate it" — and pkg-05 would flip to `accept`. I did not, for two reasons. First,
that clause is the only thing standing between my rubric and pkg-18 (golangci-lint), whose
repro runs in a private monorepo with an unshared config and is gold `reject`; "described
precisely enough" is exactly the sentence a confident report about a private setup would
satisfy, and `unfollowable-comms` is a 3-package category I currently match 3/3. Second,
it would push a judgment call ("familiar with the format") into a check whose whole value
is that it is mechanical: a stranger either has the input or does not.

Per the eval README's canary rule, the honest way to test that loosening is a cheap
`--only pkg-05,pkg-18,pkg-06,pkg-20` run — pkg-18 and pkg-06 as the `unfollowable-comms`
canaries the change could flip, pkg-20 as the single-package `disclosure` category — before
spending another $4 on a confirming full run. I am recording that as the move I would make
rather than one I made: the bar and the category floor were already clear at 19/20, and I
would rather submit a rubric whose one miss I can explain than buy one agreement by
blurring the check that earns me a whole category.

What it costs in real terms: my rubric will reject an otherwise-good report that
paraphrases a small config file instead of pasting it. I accept that, and the cheap fix is
on the writer's side, not the grader's — paste the four lines.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/repro-check/`.
