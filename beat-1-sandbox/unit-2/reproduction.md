# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

---

## Your identity upstream

**GitHub username**

andrewskoblov

---

## Posted upstream

**Claim comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61#issuecomment-5984283473

I'd like to work on this one.

Reading `api/routes/health.py:32`, the Postgres probe calls `await db.execute("SELECT 1")` with a bare string rather than `sqlalchemy.text("SELECT 1")`. The surrounding `try` block catches every exception and sets `postgres: unhealthy`, so if SQLAlchemy 2.x rejects the raw string the route would report the database as down and swallow the real `ArgumentError` into a log line, which matches what this issue describes.

Next I'll reproduce that against a real SQLAlchemy 2.x async session, record the environment and the exact output including a control run with `text()`, and post the report here. I'm not promising a fix or a date yet, only the reproduction and what it shows.

I used an AI assistant to help organise this comment. I'll run every step myself and I understand what I'm posting.

**Reproduction comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61#issuecomment-5984289682

Reproduced. The `text()`-less call fails as described, and the error text matches the one in the issue.

**Environment**

- OS: Windows 10 (AMD64)
- Python: 3.11.9
- SQLAlchemy: 2.1.3
- aiosqlite: 0.22.1 (see the deviation note below)
- Repo: my fork of `codepath/pathreview-ai301-fa26-s1` at commit `f89c06fc3ff292df2a04a39ac51319d32a76b779`, `main`, unmodified working tree

**Deviation from the issue's target, stated up front:** I ran the probe against
a SQLAlchemy `AsyncSession` backed by `sqlite+aiosqlite` in memory, not against
the project's Postgres stack. I did this because the failure happens in
SQLAlchemy's statement-coercion layer before any dialect or driver is involved,
so the backend should not matter to it, but that is my reasoning and not
something this run proves. I have not run `GET /health` against a live Postgres
container.

**Steps**

1. Create a virtualenv and install the versions above:
   `pip install "sqlalchemy>=2.0.0" aiosqlite greenlet`
2. Save this as `repro_61.py`. The failing call is the same one at
   `api/routes/health.py:32`, `await db.execute("SELECT 1")`:

```python
import asyncio, sqlalchemy
from sqlalchemy import text
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker

async def main():
    print(f"SQLAlchemy {sqlalchemy.__version__}")
    engine = create_async_engine("sqlite+aiosqlite:///:memory:")
    Session = async_sessionmaker(engine, expire_on_commit=False)

    print("\n--- FAILING RUN: exactly what health.py:32 does ---")
    async with Session() as db:
        try:
            await db.execute("SELECT 1")
            print("UNEXPECTED: raw string accepted")
        except Exception as exc:
            print(f"{type(exc).__name__}: {exc}")

    print("\n--- CONTROL: same probe wrapped in text() ---")
    async with Session() as db:
        result = await db.execute(text("SELECT 1"))
        print(f"OK, scalar result = {result.scalar()}")

    await engine.dispose()

asyncio.run(main())
```

3. Run it: `python repro_61.py`

**Output**

```
SQLAlchemy 2.1.3

--- FAILING RUN: exactly what health.py:32 does ---
ArgumentError: Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')

--- CONTROL: same probe wrapped in text() ---
OK, scalar result = 1
```

**Expected:** the probe executes and the route reports `postgres: healthy`.

**Actual:** the bare string raises
`ArgumentError: Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')`,
which is the error named in the issue. The control run with `text()` returns `1`
on the same session, so the session and driver are fine and the raw string is
the only difference between the two runs.

**Why this surfaces as a false "database down":** the `try` at
`api/routes/health.py:30-38` catches `Exception`, so the `ArgumentError` never
propagates. It is logged as `postgres_health_check_failed` and the handler sets
`postgres: unhealthy` and `status: unhealthy`. That is the handler path as
written in the source; I have not observed the 503 response itself, for the
reason in the deviation note.

**Next step:** wrap the probe in `sqlalchemy.text()` and add a unit test that
calls the health route's database check against a mocked session, so the
regression is pinned without needing the Postgres container. I'll also check
whether the `call-overload` suppression at `pyproject.toml:154` can come out
once the call is typed properly.

I used an AI assistant to help organise this report. I ran every command above
myself and the output is from my own run.

## Eval iterations

**Run history**

1. Full run, 20 scored packages: **18/20**, bar PASS, with the category floor
   held in every category (clear-accept 6/8, disclosure 1/1, no-evidence 4/4,
   unfollowable-comms 3/3, wrong-target 4/4). This is the run saved in
   `eval-run.txt`.

One run. Both disagreements were clear-accepts my rubric held rather than
posted (pkg-03 and pkg-05), which is the conservative direction to be wrong in
for a tool whose job is to stop a bad comment going out, and both fell inside
the four calls the set describes as genuinely arguable. I did not spend a
second full run, because the bar reads the committed run and the floor, and
both were already satisfied; a revision loosening `outcome-honest` or
`steps-followable` enough to flip those two would have risked the
single-package `disclosure` category that the floor depends on, and that trade
is examined under Trade-offs below.

**Package analysis**

`pkg-20` (ghostty-org/ghostty#13604). Gold label: `reject`. My rubric also
decided **reject**, on `conventions-and-disclosure` alone.

This package is the reason that check exists. Every proof check passes, and
passes easily. The environment record is complete
(`ghostty 1.3.1 (release build, Fedora 42 RPM), GTK backend, GNOME 48
(Wayland), dark system scheme`). The steps are exact, the config is a single
named line, and the artifact is a real terminal transcript showing
`^[[?997;2n` where the issue says the response should be `997;1`. There is even
a control run with the conditional theme pair returning `^[[?997;1n`. Read as
proof, it is one of the best packages in the set.

What holds it is the repo-facts line
`contribution policy (CONTRIBUTING.md + AI_POLICY.md): strict AI rules. All AI
usage in any form must be disclosed, stating the tool used and the extent of
the assistance`. Course packages are AI-assisted work, and neither the claim
comment nor the repro report says so anywhere. The phrase that decides it is
"in any form": the requirement is not limited to pull requests or to code, so
it reaches an issue comment, and an undisclosed comment is not ready to post no
matter how good the evidence under it is.

The reason I trust the check rather than just the outcome is that `pkg-09`
tests the opposite case and my rubric passed it. Its policy also demands the
tool and the extent of its use, but it scopes that to the pull request and then
says outright that "the policy states no disclosure ask for issue comments".
`pkg-09` is a gold accept and discloses nothing, correctly. A check that fired
on the word "disclose" would have taken a clear-accept down with it; the check
had to read the policy's scope, not just its verb.

**Check rationale**

Quoted from `tools/repro-check/rubric.md`, exactly as it reads now:

| outcome-honest | Every statement the claim comment and the repro report make about what was established, read against what the artifacts actually show, plus any difference between the setup tested and the target the issue names. | The package asserts no more than its artifacts support, and any deviation from the issue's stated target (a different version, operating system, build, or configuration) is stated in the text rather than left silent. An evidenced cannot-reproduce passes in full: reporting honestly that the bug did not appear, and naming what differed, is a correct outcome. Fail on certainty the artifacts do not carry ("guaranteed reproducible", "I verified the race condition") and on a silent deviation from the issue's target. | required |

Two decisions are in that wording. The first is the final sentence. An
evidenced cannot-reproduce had to be an explicit pass, not something left for
the grader to infer, because two of the eight clear-accepts (`pkg-09` and
`pkg-10`) are cannot-reproduce reports. Both show a genuine attempt, name what
differed from the reporter's setup, and say the bug did not appear. Under a
check that asked "did the package establish the bug?", both would fail and the
clear-accept category would lose a quarter of its members to a rubric that was
working as written.

The second is the deviation clause. I rejected a version-matching check, which
was my first instinct, because it cannot tell `pkg-16` from `pkg-03`, `pkg-07`
and `pkg-12`. All four ran against something other than the issue's target.
The three accepts say so in the text; `pkg-16` tests pandas 1.5.3 against an
issue confirmed on main and never mentions it. The difference that matters is
not whether the versions matched but whether the reader is told they did not,
so the check grades the disclosure of the deviation rather than the deviation
itself.

**Trade-offs**

The clause requiring that the package assert no more than its artifacts support
is what cost me `pkg-03`, a gold accept my rubric rejected. The run's note was
`Asserts that dropping -r reports 1,4,7,10 correctly, but no output for that
run is shown; the assertion exceeds the artifacts`. That reading is literally
true: the main reproduction is fully evidenced, but the contrasting
minus-replace run is described in prose with no transcript. My check treats a
stated result with no artifact behind it as over-claiming, and it does not
distinguish the central claim of a report from a supporting aside.

I accept that miss deliberately. The alternative is exempting secondary claims
from the evidence requirement, and the packages that most need catching fail in
exactly that shape: `pkg-15` asserts "I verified this race condition" as a
supporting observation with no transcript, and `pkg-13` offers "guaranteed
reproducible" the same way. A clause loose enough to let `pkg-03` through is
loose enough to let those two through, and they are two of the four
`no-evidence` packages. Trading one false reject in an eight-package category
for four true rejects in a four-package category is the right side of that
exchange, and holding a good comment back for a missing transcript is a cheaper
error than posting a confident empty one.

The same strictness is doing no harm anywhere else, and the run shows it:
`outcome-honest` was the sole cause of exactly one disagreement. The other
seven clear-accepts passed it, including both cannot-reproduce reports and all
three packages carrying an acknowledged version delta.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/repro-check/`.
