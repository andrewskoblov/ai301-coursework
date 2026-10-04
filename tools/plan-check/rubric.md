# Rubric: is this plan ready to post and build from?

Six required checks, one per failure family that gets bad plans posted: the
diagnosis contradicts the evidence, the change is unbounded, a stranger could
not start, the test proves nothing observable, the comment ignores what the
thread already settled, and the words ignore the repo's stated rules. Two
preferred checks rank plans that already pass.

Every check reads the thing itself against the issue and its repro evidence.
None grades the write-up's shape, its length, or its headings.

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| cause-grounded | The plan's stated cause, read against what the Repro evidence block actually shows: its steps, its artifacts, and especially any control run that isolates one variable. | The stated cause explains the behavior the repro evidence pins down, and nothing in that evidence rules it out. Fail when a control run already excludes the named cause (the behavior still occurs with that component removed, or does not occur with it present), when the evidence shows the fault occurring at a different stage than the plan names, or when the plan asserts a cause the package's evidence never touches. A cause the evidence supports but does not prove passes; a cause the evidence contradicts does not. | required |
| scope-bounded | The plan's in-scope and not-in-scope statements, the files or areas it names, and the work items it lists, read against what the issue asks for. | The plan changes what the issue needs and stops. Fail when it adds work the issue never asked for: a drive-by refactor, a dependency migration, a module restructure, a new user-facing option, a CI or test-harness overhaul, or a redesign with the bounded fix buried inside it. Narrowing is not creep: a plan that deliberately does less than the issue's full surface and says which parts it defers and why passes this check, because a stated deferral bounds the work rather than expanding it. | required |
| buildable | The plan's approach, the files or areas it names, and the order of work, read as a stranger holding the repo would read them. | A stranger could start executing without asking the author anything. The plan names where the change goes (a file, a function, a named call site, or an area specific enough to locate) and commits to one approach. Fail when the real decisions are deferred to build time: no files named, the layer still undecided ("gocui? tcell? not sure"), the location given as "somewhere", the approach offered as alternatives with no choice made, or the work described only as investigation. | required |
| test-decisive | The plan's test plan, read against the Repro evidence block's steps and artifacts. | The test names an outcome someone could observe and disagree about: a specific command or case to run, and what the result should be after the fix, tied to the behavior the repro pinned down. Re-running the repro's own steps with the expected post-fix result stated is the strongest form and passes outright. Fail when the stated outcome is unobservable or generic: "should feel fast", "nothing else should feel broken", or "run the full test suite" with no result named for the fix itself. | required |
| thread-engaged | The Thread highlights section, specifically comments from an OWNER, MEMBER or COLLABORATOR, read against the candidate plan and plan comment. | Where the thread carries explicit maintainer direction (a named culprit or file, a posted patch or test binary, a settled approach, a request to test something, or an open pull request covering the same ground), the plan or the comment engages it: adopting it, building on it, or giving a reason for departing from it. Fail when such direction exists and the drafts proceed as if it does not. Pass when the thread carries no maintainer direction, including a thread with no comments. | required |
| conventions-and-disclosure | The repo-facts block's contribution policy and contributing asks, read against the candidate plan comment. Treat the package as AI-assisted work. | Pass when the comment satisfies the repo's stated rules about how contributors write. Specifically: (a) if the policy requires disclosing AI assistance in a scope that reaches issue comments, meaning it is stated for comments or applies to AI use "in any form" without limiting the ask to pull requests and code, then the comment states that AI assistance was used; (b) if the policy requires comments be in the contributor's own words, the comment reads as human-written rather than generated boilerplate; (c) if the policy states no AI requirement, or limits its disclosure ask to pull requests and code, the check passes with no disclosure needed. | required |
| unknowns-stated | The plan's risks and unknowns, read against the confidence of its other sections. | The plan names at least one thing it does not yet know and says how it will find out, rather than presenting every step as settled. | preferred |
| prior-art-engaged | The Thread highlights and the issue context, for existing pull requests or earlier attempts at the same fix. | Where prior art exists, the plan says how its change relates to it rather than silently duplicating it. | preferred |

## Verdict rule

Accept if and only if all six `required` checks grade `pass`. Any required
check grading `fail` produces `reject`: the plan is held, not posted.

`unclear` counts as `fail` on a required check. A plan whose grounding, scope,
buildability, test, thread-awareness or conventions cannot be verified from the
package is a plan that is not ready to build from.

The two `preferred` checks never change a verdict. They rank plans that already
pass: among ready plans, one that states its unknowns and engages prior art is
the stronger contribution.
