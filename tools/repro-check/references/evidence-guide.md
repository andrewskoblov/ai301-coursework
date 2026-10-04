# Evidence guide: where proof lives in a reproduction package

The map the rubric's checks read. For each proof family: where to look, and
what good looks like when you get there.

Two reminders that apply throughout. In an eval bundle the package is the
whole world, so every signal below is somewhere in that one file. In live
mode the issue side comes from GitHub and the candidate side is the student's
draft. And recency or formatting is never the signal: a four-line report with
a command and its output carries more proof than a templated page with
headings and no artifact.

## Environment

**Where it lives.** In an eval bundle: the opening lines of the Candidate
repro report, usually an `Environment:` line or a short block before the
steps. The target it must be read against is in two other places: the issue
body (the version and platform the reporter used) and the repo-facts block's
`bug reports:` line, which records what the repo's own template asks
contributors to supply. In live mode: the top of the student's draft report,
read against the issue page and the repo's issue template.

**What good looks like.** The record names the software version and the
platform, plus whatever component the issue itself makes deciding. Which
component that is comes from the issue, not from a fixed list: a
Windows-only issue needs the OS and the driver, a shell-prompt issue needs
the shell, a panic that only appears in release builds needs the build
profile, a translation issue needs the locale. One line is enough
(`ghostty 1.3.1 release build, Fedora 42, GTK/Wayland`). What is not enough
is an environment that goes unmentioned entirely, or a report that omits the
one component the issue is about.

## Steps

**Where it lives.** In an eval bundle: the numbered or bulleted sequence in
the Candidate repro report between the environment record and the artifacts.
Read it together with the inputs it references, which may be inline in the
same report (a config file, a sample document) or named but absent. In live
mode: the same section of the draft.

**What good looks like.** A stranger with the named environment can run every
step from the text alone and arrive at the same place. The starting state is
stated (a clean config, an empty directory, a specific file), the commands are
given verbatim rather than described, and every input the steps consume is
either pasted inline or publicly fetchable by name and version. The failure
mode to catch is a run that cannot leave the author's machine: a private
repository, an unshared config, a dataset referred to but never provided, or
a step that says what was done without saying how. Length is not the test.
Four exact commands are followable; twelve prose paragraphs around a private
monorepo are not.

## Behavior shown

**Where it lives.** In an eval bundle: the fenced output blocks, log
excerpts, and described screenshots inside the Candidate repro report. The
thing to read them against is the issue's own description of the symptom,
in the Issue section, and sometimes a maintainer's narrowing in the Thread
highlights. In live mode: the draft's output blocks against the issue page.

**What good looks like.** The artifact exhibits the issue's symptom, and it
was produced by the issue's actual trigger. Both halves matter, and the second
is where good-looking packages fail. Check the trigger first: if the issue is
about an offset-from-end range and the artifact ran a prefix range, if the
issue is about `=` and the input used `:`, if the issue is about an unbound
variable path and the expression was edited, then whatever the output shows,
it is not about this issue. Then check the symptom: the error text, the exit
status, and the observable behavior should be the ones the issue reports. A
graceful argument-validation error with exit 1 is not a capacity-overflow
panic with exit 101. Garbled escape output from a terminal still running is
not a crash. A version banner and a session list prove the program starts and
say nothing about a blank pane.

An artifact for a cannot-reproduce is read differently: there, good means the
artifact shows the genuine attempt and whatever actually happened, which is
the absence of the symptom.

## Honesty

**Where it lives.** At the seam between two places that must be compared: the
assertions, in the Candidate claim comment and in the repro report's
`Expected`/`Actual` lines and closing summary; and the backing, in the
artifacts above. Also compare the environment record against the issue's
stated target, because an unstated difference between them is a dishonesty
even when nothing in the prose is false.

**What good looks like.** Every statement is one the shown artifacts can
carry. The report says what happened and stops. Where the setup departed from
what the issue targets, the departure is named in the text rather than left
for the reader to notice: "reproduced on 3.0.3, the issue reports 3.1.0" is
honest, and testing an old version against an issue confirmed on main without
saying so is not, even when the output is real. An honest cannot-reproduce is
a full pass and one of the most useful things to post: it shows the attempt,
states that the bug did not appear, names what differed from the reporter's
setup, and offers what a triggering setup would likely need. What fails is
certainty the artifacts do not support: "guaranteed reproducible" with no
measurements, "I verified this race condition" with no transcript, a
root-cause diagnosis asserted where no experiment isolated the cause.

## Comms

**Where it lives.** Three sources read together. The Candidate claim comment,
read against the issue it sits on. The repo-facts block's
`contribution policy` line, which records the repo's stated rules including
any AI policy and the scope that policy applies to. The repo-facts block's
`bug reports:` line for what the template asks. In live mode these are
`CONTRIBUTING.md`, any `AI_POLICY.md` or `AI_USAGE_POLICY.md`, and the issue
template in `.github/`.

**What good looks like, for the claim.** It is specific to this issue and
could not be pasted onto another one: it names the symptom, the file, the
configuration, or the version. It says what the author will do next, and what
it promises is investigation and a report, never a fix and never a date. The
failure modes are a bare "+1" or "me too" with no stated intent, an
interchangeable request to be assigned, and a confident promise of a fix in
two days.

**What good looks like, for the policy.** Read the policy's requirement and,
just as carefully, its scope. Policies in this set come in four shapes, and
the difference between the last two is the whole game:

- **Silent.** No AI policy stated. Nothing to satisfy.
- **Permissive with responsibility.** AI tools welcome, you must understand
  and take responsibility for what you submit. Nothing to disclose.
- **Scoped disclosure.** Disclosure is required, but the policy limits the ask
  to pull requests and code, and may say outright that issue comments carry no
  disclosure ask. An issue comment satisfies this with no disclosure at all.
- **Unscoped disclosure.** Disclosure is required for AI usage "in any form",
  which reaches comments. Here the comment must state that AI assistance was
  used, and a package that is excellent on every proof check still is not
  ready to post without it.

A separate requirement some repos add: comments to maintainers must be written
by the contributor in their own words, with AI-generated comments liable to be
hidden. That asks for human voice, not for a disclosure line, and a comment
that reads as a person wrote it satisfies it.

Course work is AI-assisted, so these rules apply to it.
