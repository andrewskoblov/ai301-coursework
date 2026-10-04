# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

---

## Posted upstream

**GitHub username**

andrewskoblov

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61#issuecomment-5984450563

Posting my plan, following up on [my reproduction above](https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61#issuecomment-5984289682).

**Cause.** `api/routes/health.py:32` calls `await db.execute("SELECT 1")` with a bare string. SQLAlchemy 2.x rejects that in statement coercion, before the driver is involved. My control run pins it: the same session and query succeed when the only change is wrapping the string in `text()`.

```
--- FAILING RUN: exactly what health.py:32 does ---
ArgumentError: Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')

--- CONTROL: same probe wrapped in text() ---
OK, scalar result = 1
```

It surfaces as a false "database down" because the `try` at lines 30-38 catches `Exception`, so the error is logged and the route sets `postgres: unhealthy` while the database is actually reachable.

**What I'm changing.** `api/routes/health.py`: import `text` and wrap the probe argument. Plus a new `tests/unit/test_health.py` that drives the real probe against an in-memory `sqlite+aiosqlite` session. Note this departs from my repro comment, where I said I'd use a mocked session: a `MagicMock` would accept the bare string and pass with the bug still in place, so it would pin nothing. A real session is the only version of that test worth having.

**What I'm not changing, and why.**

- The Redis probe (#62). `settings.redis_host` doesn't exist on `Settings`, but that's a separate issue with its own open PRs. Worth flagging: fixing #61 alone won't turn `/health` green, and I'd rather say that than widen this change to make the endpoint pass.
- Other raw `execute()` calls. There's no sweep to do, line 32 is the only one:

```
$ grep -rn 'execute("' --include=*.py . | grep -v '.venv'
./api/routes/health.py:32:        await db.execute("SELECT 1")
```

**One thing I am changing that I first said I wouldn't.** In my repro comment I said I'd check the `call-overload` mypy suppression, and my first draft of this plan deferred it on the grounds that `pyproject.toml:154` was a global `disable_error_code`. That was wrong. Lines 152-154 are a `[[tool.mypy.overrides]]` block scoped to `module = "api.routes.health"`, and `docs/CONTRIBUTING.md:114-117` says "If you fix one of those, remove its suppression too". So removing `call-overload` from that override is part of this fix. `attr-defined` stays, since that one is #62.

Worth flagging honestly: with `call-overload` removed, mypy is clean,

```
$ mypy api/routes/health.py
Success: no issues found in 1 source file
```

but it is also clean on the unfixed code, so that run doesn't prove the entry was doing anything. `health_check(db=Depends(get_db))` leaves `db` unannotated, so mypy never checks the call. Typing the parameter in a scratch probe produces the error the entry was named for:

```
error: No overload variant of "execute" of "AsyncSession" matches argument type "str"
```

**How I knew it worked.** I've built this on `fix/61-health-db-probe-text`, so here is the check rather than a promise of one. Calling the real `health_check` handler with a live `sqlite+aiosqlite` session, before and after the edit:

```
BEFORE
[error] postgres_health_check_failed   error="Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')"
postgres dependency : unhealthy

AFTER
[debug] postgres_health_check_passed
postgres dependency : healthy
```

And the new tests:

```
$ pytest tests/unit/test_health.py -v
tests/unit/test_health.py::test_postgres_probe_reports_healthy_on_a_reachable_database PASSED
tests/unit/test_health.py::test_postgres_probe_executes_textual_sql PASSED
2 passed
```

As predicted above, overall `status` stays `unhealthy` after the fix, because the Redis probe (#62) is still failing separately. The Postgres dependency is the part this issue is about, and that one flips.

**Prior art on this thread.** smtanaka00 posted a plan on 3 Oct that reaches the same `text()` fix and also proposes dropping the `call-overload` entry, so I'm not claiming novelty here. The differences in mine are the regression test that asserts a bare string still raises, so the call can't be silently unwrapped later, and the mypy note above about the entry not currently being load-bearing. Under the course's house rules a classmate's plan doesn't block mine, so I'm posting my own rather than piggybacking.

**Unknown I haven't closed.** I haven't run this against Postgres, only SQLite. The failure is above the dialect layer so the backend shouldn't matter, but that's reasoning rather than something I've shown. If anyone has a Postgres stack up, that's the check I'd most like confirmed.

I used an AI assistant to help organise this comment. I ran every command myself and understand what I'm proposing.

---

## Your branch

**Branch**

`fix/61-health-db-probe-text`

**Evidence**

My Unit 2 reproduction drove the probe's exact call against a SQLAlchemy 2.x
`AsyncSession`. Re-run against the built change, through the real
`health_check` handler rather than the stand-in script, using a live
in-memory `sqlite+aiosqlite` session.

**Before** (`api/routes/health.py:32` = `await db.execute("SELECT 1")`):

```
$ ./.venv-repro/Scripts/python.exe verify_61.py
2026-10-04 17:07:05 [error    ] postgres_health_check_failed   error="Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')"
2026-10-04 17:07:05 [error    ] redis_health_check_failed      error="No module named 'redis'"
2026-10-04 17:07:05 [debug    ] vector_db_health_check_passed
postgres dependency : unhealthy
redis dependency    : unhealthy (separate issue #62)
overall status      : unhealthy
```

**After** (`api/routes/health.py:33` = `await db.execute(text("SELECT 1"))`):

```
$ ./.venv-repro/Scripts/python.exe verify_61.py
2026-10-04 17:07:31 [debug    ] postgres_health_check_passed
2026-10-04 17:07:31 [error    ] redis_health_check_failed      error="No module named 'redis'"
2026-10-04 17:07:31 [debug    ] vector_db_health_check_passed
postgres dependency : healthy
redis dependency    : unhealthy (separate issue #62)
overall status      : unhealthy
```

The Postgres dependency flips from `unhealthy` to `healthy` and the
`ArgumentError` log line is gone. `overall status` stays `unhealthy` because
the Redis probe fails separately under issue #62, which my plan scoped out and
predicted would leave the endpoint red.

**The new regression tests**, run with no flags of my own so they exercise the
repo's own pytest configuration:

```
$ ./.venv-repro/Scripts/python.exe -m pytest tests/unit/test_health.py -v
collected 2 items

tests/unit/test_health.py::test_postgres_probe_reports_healthy_on_a_reachable_database PASSED [ 50%]
tests/unit/test_health.py::test_postgres_probe_executes_textual_sql PASSED [100%]

======================== 2 passed, 1 warning in 1.91s =========================
```

**mypy, after dropping `call-overload` from the `api.routes.health` override:**

```
$ ./.venv-repro/Scripts/python.exe -m mypy api/routes/health.py
Success: no issues found in 1 source file
```

## Eval iterations

**Run history**

1. Full run, 20 scored packages: **20/20**, bar PASS, every category complete
   (clear-accept 7/7, scope-creep 4/4, thread-convention 2/2, unbuildable 3/3,
   wrong-cause 4/4). This is the run saved in `eval-run.txt`.

One run. The rubric agreed with every gold label on the first full run, so
there were no disagreements to re-grade with `--only` and no second run to
make. What that cost is examined under Trade-offs: a first-run 20/20 means the
rubric was never tested against a case it got wrong, so I have less evidence
about its edges than a messier run would have given me.

**Package analysis**

`pkg-09` (sharkdp/fd#2067). Gold label: `accept`. My rubric also decided
**accept**, and it is the package I was most worried about, because it is the
one my `scope-bounded` check could most easily have rejected for doing the
right thing.

The plan handles only part of the issue's surface. It picks option 2,
normalizing the path separator, gates that to Windows, and explicitly declines
option 1, the larger rework to direct globset matching. The package states it
as `In scope: the glob-mode full-path match path on Windows. Not in scope,
stated deferrals: option 1 (switching to direct globset matching, a larger
change)`. A scope check written as "does the plan address the issue" would
fail this, because it demonstrably does not address all of it. The gold note
even concedes the point: "arguable on the deferral, ready as scoped".

My rubric passed it because of the sentence in `scope-bounded` that reads
"Narrowing is not creep: a plan that deliberately does less than the issue's
full surface and says which parts it defers and why passes this check, because
a stated deferral bounds the work rather than expanding it." The direction of
travel is what the check measures. `pkg-06`, `pkg-12`, `pkg-15` and `pkg-19`
all add fronts the issue never raised, which is work growing past its bounds.
`pkg-09` and `pkg-14` both subtract, and say so. Those are opposite motions,
and a check that only counts "is the plan smaller or larger than the issue"
would get one of the pairs wrong whichever way it was tuned.

**Check rationale**

Quoted from `tools/plan-check/rubric.md`, exactly as it reads now:

| cause-grounded | The plan's stated cause, read against what the Repro evidence block actually shows: its steps, its artifacts, and especially any control run that isolates one variable. | The stated cause explains the behavior the repro evidence pins down, and nothing in that evidence rules it out. Fail when a control run already excludes the named cause (the behavior still occurs with that component removed, or does not occur with it present), when the evidence shows the fault occurring at a different stage than the plan names, or when the plan asserts a cause the package's evidence never touches. A cause the evidence supports but does not prove passes; a cause the evidence contradicts does not. | required |

The clause doing the work is "especially any control run that isolates one
variable", and it is there because of how the `wrong-cause` packages fail.
They are not sloppy. `pkg-16` is described in the gold notes as "long and
confident", and `calib-03` as "polished... beautifully formatted". Read as
prose, each diagnosis is plausible. What refutes them is a single artifact
already sitting in the package's own repro evidence: `pkg-01`'s control runs
the same items without `-v` and parses fine, ruling out the tokenizer the plan
blames. `pkg-11`'s control shows `[.a] | length == 1` at top level, so collect
already synthesizes null outside condition contexts. `pkg-07`'s control shows
a sibling friendly error printing from the same bundle the plan says was
tree-shaken away.

I rejected the first version I drafted, which asked whether the diagnosis was
"supported by the evidence". That phrasing invites reading the evidence
through the plan's eyes, and these packages are built to make that feel
reasonable. Naming the control specifically, and pairing it in
`procedure.md` with a step that writes down what the control excludes
**before** reading the plan's diagnosis, turns the check from an impression
into a comparison. The last sentence does the remaining work: "A cause the
evidence supports but does not prove passes; a cause the evidence contradicts
does not." Without it the check would start rejecting accepts like `pkg-02`
and `pkg-13`, whose causes are correct but not formally proven by the repro.

**Trade-offs**

What `cause-grounded` gives up is any case where the true cause is one the
package's evidence cannot reach. The check is built to trust the control run
over the plan's reasoning, and that is exactly backwards when a plan is right
for a reason the repro never exercised: a correct diagnosis resting on reading
the source rather than on an artifact would be graded `unclear`, and the
verdict rule turns `unclear` into `fail` on a required check. It would hold a
good plan.

That cost is real, and I accepted it because the failure it prevents is more
common and more expensive. Four of the twenty packages are `wrong-cause`, and
all four are confident, well-written plans that would have been built from. A
plan held for thin evidence costs one round trip; a plan built on an excluded
cause costs the build.

Nothing else moved, and the run shows how I know. This was a single full run
at 20/20, so no check caused a disagreement anywhere, `cause-grounded`
included: all seven clear-accepts passed it, including `pkg-09` and `pkg-14`
whose plans narrow their scope, and all four `wrong-cause` packages failed it
rather than failing on some unrelated check that happened to catch them. That
is the result I wanted, but it is also the limit of what I can claim. A rubric
that agrees everywhere on its first run has not been probed at its edges, and
the honest version of this field is that I know `cause-grounded` separates
these twenty packages correctly and I do not yet know where it breaks. The
`--only` canary loop exists for revisions, and I never had a revision to make,
so I never used it.

---

Related paths: `plan.md` and `eval-run.txt` in this directory; your skill's
files in `tools/plan-check/`.
