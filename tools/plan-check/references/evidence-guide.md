# Evidence guide: where evidence lives in a plan package

The map the rubric's checks read, and the procedure's gathering stage follows.
For each family: where to look, and what good looks like when you get there.

In an eval bundle the package is the whole world, and every family below sits
in one of its sections: `## Repo facts`, `## Issue`, `## Thread highlights`,
`## Repro evidence`, `## Candidate plan`, `## Candidate plan comment`. In live
mode the issue side comes from GitHub and the candidate side is the student's
`plan.md` and draft comment.

## Diagnosis and grounding

**Where it lives.** The plan's cause sits in the opening of
`## Candidate plan`, usually one or two sentences naming what is going wrong
and where. The thing it must answer to is `## Repro evidence`: its steps, its
output blocks, and above all any **control run**, a second run that changes one
variable to show what is and is not involved. Live: the student's posted repro
comment on the issue, read against the diagnosis in `plan.md`.

**What good looks like.** The stated cause explains the behavior the repro
pins down, and no artifact in that block rules it out. The control is what
decides most of these: if the repro shows the same failure with a component
removed, that component is not the cause, however confidently the plan names
it. Read the control first and write down what it excludes before reading the
diagnosis, because a polished plan is very good at making you read its cause
into evidence that does not support it. Watch for the plan that blames a stage
the evidence places the fault before or after: a parser blamed when the output
shows the value already wrong on the way in, a missing module blamed when a
sibling feature from the same bundle is shown working. A cause the evidence
supports without proving is fine; a cause the evidence contradicts is not.

## Scope

**Where it lives.** The in-scope and not-in-scope statements inside
`## Candidate plan`, plus the full list of work items it commits to and the
files or areas it names. The yardstick is `## Issue`: what was actually asked
for. Live: the scope section of `plan.md` against the issue body.

**What good looks like.** One bounded change that answers the issue and stops.
The reliable signal is a not-in-scope line that names something a reasonable
person might otherwise have bundled in. Creep looks like additional fronts the
issue never raised: a dependency migration, a module restructure, a new
user-facing option, a CI or test-harness overhaul, a redesign of the subsystem
with the actual fix as one bullet inside it. The trap worth naming is that
creep usually arrives inside a *good* plan: the core fix is correct and the
extra work is genuinely tempting while-you-are-in-there work.

Narrowing is the opposite of creep and must not be read as a failure. A plan
that deliberately handles less than the issue's full surface, names the part
it is deferring, and gives a reason is bounding its work, which is exactly what
this family wants. "Not in scope, stated deferrals: switching to direct
globset matching, a larger change" is a pass, not a hedge.

## Executability

**Where it lives.** The approach and work-order parts of `## Candidate plan`,
and wherever it names files, functions or call sites. Live: the same sections
of `plan.md`, which you can hold against the actual repo.

**What good looks like.** A stranger with the repo checked out could begin
without asking the author a question. That needs two things: a location
specific enough to find (a file path, a function, a named call site, or an
area like "the erase-scrollback branch") and one chosen approach rather than a
menu. Length is not the test; "add a saturating clamp at the two subtraction
sites the repro isolates" is more executable than three paragraphs of context.
The failure shape is the decision deferred to build time: no file named, the
layer undecided, the location given as "somewhere", two options listed with
neither chosen, or the whole plan described as investigation rather than
change.

## Test plan

**Where it lives.** The test or verification part of `## Candidate plan`, read
against the steps and artifacts in `## Repro evidence`. Live: the test plan in
`plan.md` against the student's posted repro comment.

**What good looks like.** The test names something a person could watch happen
and could disagree about: a specific command or case, and the result expected
after the fix. The strongest and most common good form is re-running the
repro's own steps and stating what changes, because the repro already produced
a concrete artifact and the test simply says what that artifact becomes. The
failure shape is an outcome nobody could check: "should feel fast", "nothing
else should feel broken", "run the full test suite" with no result named for
the fix itself. A full-suite run is fine as an addition and empty as the whole
plan, because it says nothing about the behavior the issue is about.

## Honesty

**Where it lives.** The risks and unknowns part of `## Candidate plan`, read
against how certain the rest of the plan sounds. After a build, the
`## Deviations` section of the student's own `plan.md` is where a mid-build
change of direction gets recorded.

**What good looks like.** The plan names something it does not yet know and
says how it will find out: an unknown with a next move attached, such as
whether a behavior holds tool-by-tool, or whether a variant can be tested at
all. False confidence looks like a plan where every step is settled and no
risk is named, particularly when the diagnosis rests on one artifact. For
deviations: an honest one says what changed and why, and it is recorded in the
plan rather than left to be discovered in the diff. "Nothing changed" is a
complete and honest deviation note when it is true and said plainly.

## Comms

**Where it lives.** Three places read together. `## Candidate plan comment`,
which is what the maintainer will actually see. `## Thread highlights`, for
what the maintainers have already said. The `contribution policy` and
contributing-asks lines in `## Repo facts`. Live: the draft comment, the live
issue thread, and `CONTRIBUTING.md` plus any `AI_POLICY.md` or
`AI_USAGE_POLICY.md`.

**What thread-aware looks like.** Scan the thread for comments whose author is
OWNER, MEMBER or COLLABORATOR, and treat as direction anything concrete: a
named culprit or file, a posted patch or test binary, a settled approach, a
request to test something, an open pull request on the same ground. A good
comment engages that direction, by adopting it, building on it, or saying why
it is departing from it. The failure shape is a plan that proceeds as though
the direction were not there, which reads to a maintainer as not having read
the thread, and is worst when the maintainer has already isolated the culprit
and the plan proposes a workaround elsewhere. A thread with no maintainer
direction, including one with no comments at all, asks nothing here.

**What policy-aware looks like.** Read the requirement and, just as carefully,
its **scope**. The policies in this set come in four shapes, and the last two
differ only in scope:

- **Silent.** No AI policy stated. Nothing to satisfy.
- **Permissive with responsibility.** AI tools welcome, you must understand
  and take responsibility for what you submit. Nothing to disclose.
- **Scoped disclosure.** Disclosure is required, but the ask is limited to the
  pull request and the code, and the policy may say outright that issue
  comments carry no disclosure ask. A plan comment satisfies this with no
  disclosure line.
- **Unscoped disclosure.** Disclosure is required for AI usage "in any form",
  which reaches comments. The comment must say AI assistance was used, and an
  otherwise excellent bounded plan still is not ready to post without it.

A separate ask some repos make: comments to maintainers must be in the
contributor's own words, with AI-generated comments liable to be hidden. That
wants human voice, not a disclosure line, and a comment that reads as a person
wrote it satisfies it.

Course work is AI-assisted, so these rules apply to it.
