# Plan: issue #61, health check DB probe passes a raw SQL string

Issue: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61
Reproduction: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61#issuecomment-5984289682

## Diagnosis

The Postgres probe at `api/routes/health.py:32` calls:

```python
await db.execute("SELECT 1")
```

SQLAlchemy 2.x refuses a bare string at the statement-coercion layer, before
any dialect or driver is involved. My reproduction shows exactly that, with a
control that isolates the string as the only variable:

```
SQLAlchemy 2.1.3

--- FAILING RUN: exactly what health.py:32 does ---
ArgumentError: Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')

--- CONTROL: same probe wrapped in text() ---
OK, scalar result = 1
```

The control is what pins the cause. The same `AsyncSession`, the same engine
and the same query succeed when the only change is wrapping the string in
`text()`, so the session, the driver and the query are all ruled out and the
bare string is what fails.

The reason this surfaces as a false "database down" rather than as an error is
the handler at `api/routes/health.py:30-38`: the `try` catches `Exception`, so
the `ArgumentError` is logged as `postgres_health_check_failed` and the route
sets `postgres: unhealthy` and `status: unhealthy`. The database is reachable;
the probe never reaches it.

## Scope

**In scope:** the single call at `api/routes/health.py:32`, wrapped in
`sqlalchemy.text()`, plus a unit test that pins the behavior.

**Not in scope, with reasons:**

- **The Redis probe (#62).** `api/routes/health.py:46-51` reads
  `settings.redis_host` and `settings.redis_port`, which do not exist on
  `Settings`. That is a separate reported bug with its own issue and its own
  open pull requests. Fixing #61 alone will not turn `/health` green, and I
  say so rather than quietly widening this change to make the endpoint pass.
- **Keeping the `attr-defined` mypy suppression.** `pyproject.toml:152-154` is
  a per-module override for `api.routes.health` carrying
  `["attr-defined", "call-overload", "index"]`. `attr-defined` is #62's
  `settings.redis_host`, so it stays until that issue is fixed. `index` is not
  mine either. Only `call-overload` belongs to this fix, and removing it is
  now **in scope** (see Files below), because `docs/CONTRIBUTING.md:114-117`
  says "If you fix one of those, remove its suppression too".
- **Auditing other raw `execute()` calls.** I grepped: `health.py:32` is the
  only one in the repository outside my own repro script, so there is no
  sweep to do. Recording the check rather than leaving it as an open question.
- **Changing the catch-all `except Exception`.** Narrowing it would be a
  behavior change to error handling that the issue does not ask for.

## Files I will touch

- `api/routes/health.py` — add the `text` import, wrap the probe argument.
- `tests/unit/test_health.py` — new file. The directory already holds
  `test_keyword_search.py`, `test_pii_scrubber.py` and similar, so a unit test
  for the health route belongs alongside them.
- `pyproject.toml` — drop `call-overload` from the `api.routes.health` mypy
  override, leaving `attr-defined` (issue #62) and `index` in place.

## Approach

1. Import `text` from `sqlalchemy` in `api/routes/health.py`.
2. Change line 32 from `await db.execute("SELECT 1")` to
   `await db.execute(text("SELECT 1"))`.
3. Add `tests/unit/test_health.py` with a test that drives the real probe
   against an in-memory `sqlite+aiosqlite` `AsyncSession` and asserts the
   Postgres dependency comes back `healthy`. Using a real session rather than
   a mock is deliberate: a `MagicMock` session would accept a bare string and
   pass even with the bug still present, so it would not pin anything.
4. Remove `call-overload` from the `api.routes.health` mypy override in
   `pyproject.toml` and re-run mypy on the file.

## Test plan

The observable outcome is the `ArgumentError` disappearing and the probe
returning a result, run through the real code path rather than my stand-in
script.

**Before (already recorded in the repro comment):** running the probe's exact
call against a SQLAlchemy 2.x `AsyncSession` raises
`ArgumentError: Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')`.

**After:** the same call returns a row, and the new unit test passes. Concretely:

1. `pytest tests/unit/test_health.py -v` exits 0 with the new test passing.
   Expected: the health route's Postgres dependency reports `healthy` instead
   of `unhealthy`.
2. Re-run the repro script `repro/repro_61.py` unchanged. Expected: the failing
   run still raises, because that script deliberately calls the unwrapped form
   and documents the bug, so it is the control rather than the verification.
3. The deciding check is a direct one: call the real `health_check` handler
   with a live `sqlite+aiosqlite` session before and after the edit, and show
   `postgres: unhealthy` becoming `postgres: healthy`. This runs the actual
   changed code, which the stand-in script does not.

## Risks and unknowns

- **I have not run this against Postgres.** My environment uses
  `sqlite+aiosqlite`. The failure is in statement coercion, above the dialect,
  so the backend should not matter, but that is reasoning and not something I
  have demonstrated. If a reviewer has a Postgres stack up, that is the check
  I would most like confirmed.
- **The route will still report unhealthy overall** in any environment without
  Redis reachable, because of #62. A reviewer running `/health` end to end
  should expect `postgres: healthy` with `status: unhealthy`, and I would
  rather say that now than have it read as my fix not working.
- **Test placement.** I am assuming `tests/unit/` is right for a route-level
  test given the existing files there. If the project would rather this sat in
  `tests/integration/`, that is a cheap move.

## Deviations

The code change went as planned: one import and one argument at
`api/routes/health.py`, now line 33, plus the new `tests/unit/test_health.py`.
Four things differed from my first draft of this plan. The first is a
correction to the plan itself and is the one that matters.

**0. My first draft deferred the mypy suppression for a false reason, and I
was wrong.** I wrote that `pyproject.toml:154` was a global
`disable_error_code`, so removing an entry would surface errors repo-wide.
It is not global. Lines 152-154 are `[[tool.mypy.overrides]]` with
`module = "api.routes.health"`, scoped to this one file, and
`docs/CONTRIBUTING.md:114-117` says "If you fix one of those, remove its
suppression too". Checking the claim against the file showed it was wrong before the plan was
posted. So the suppression removal moved from
deferred to in scope, and `pyproject.toml` became a third touched file.

Running it down produced a result worth recording honestly. With
`call-overload` removed, `mypy api/routes/health.py` reports
`Success: no issues found in 1 source file`. But it reports the same on the
original unfixed code, so that run does not demonstrate the suppression was
doing any work. The reason is the handler signature: `health_check(db=Depends(get_db))`
leaves `db` unannotated, so it is `Any` and mypy never checks the call. A
scratch probe with the parameter typed confirms the suppressed error is
exactly this bug:

```
async def f(db: AsyncSession) -> None:
    await db.execute("SELECT 1")

error: No overload variant of "execute" of "AsyncSession" matches argument type "str"
```

So removing the entry is correct and CONTRIBUTING asks for it, and mypy stays
clean with it gone, but I cannot claim my run proved it was load-bearing. It
would start doing work the moment someone annotates `db`.

**1. The test needed `@pytest_asyncio.fixture`, not `@pytest.fixture`.** My
first version used a plain `@pytest.fixture` for the async session. It passed
when I ran it with `-o asyncio_mode=auto`, and errored under the repo's own
configuration, which sets only `testpaths` and therefore leaves pytest-asyncio
in its default strict mode. I caught this by re-running the way a reviewer
would, with no flags of my own. Switching the fixture decorator to
`@pytest_asyncio.fixture` fixed it, and both tests now pass under plain
`pytest tests/unit/test_health.py`.

**2. I wrote a second test I had not planned.** The plan described one test,
asserting the probe reports `healthy`. I added
`test_postgres_probe_executes_textual_sql` alongside it, which asserts that a
bare string still raises `ArgumentError` and that the `text()` form returns 1.
The first test proves the route works now; the second pins down why, and would
fail if someone later unwrapped the call. The extra test is in the same file
and touches nothing else, so it stays inside the scope I posted.

**3. Verification needed a small harness.** The plan said I would call the
real handler before and after. In practice `health_check` raises
`HTTPException(503)` whenever any dependency is down, and Redis is down here
because of #62, so the return value is never reached. The 503's `detail`
carries the per-dependency verdicts, so I read them from there. This is the
risk I listed under "the route will still report unhealthy overall", now
confirmed rather than predicted: after the fix the body reads
`postgres: healthy` with `status: unhealthy`, and the remaining `unhealthy` is
entirely #62. The harness (`verify_61.py`) is a local scratch file and is not
committed to the branch.

All four are recorded here before the plan comment went up, so the posted
comment states the corrected scope rather than the false deferral.
