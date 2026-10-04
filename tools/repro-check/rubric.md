# Rubric: is this reproduction package ready to post?

Seven required checks covering the proof families that get bad packages
posted: the environment is recorded, the steps are followable, an artifact
exists, the artifact shows what the report says it shows, the outcome is
stated honestly, the claim is specific, and the words respect the repo's
stated rules. Two preferred checks rank packages that already pass.

Every check reads the thing itself. None of them grades the write-up's shape,
its length, or whether it used the template's headings.

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| env-recorded | The repro report's environment record, read against the issue's stated target and the repo-facts block's bug-report template asks. | The report names the software version and the platform it ran on, plus any component the issue itself singles out as deciding (a driver, a shell, a build profile, a backend, a locale). A single terse line carries this as well as a table does. Fail when there is no environment record at all, or when the issue is specific to a platform or component the report never names. | required |
| steps-followable | The repro report's steps, and every input those steps reference. | A stranger holding the named environment could re-run each step from the report's text alone. Inputs the steps depend on (a config file, a sample document, a command, a dataset) are either shown inline or publicly fetchable. Fail when any step depends on something private, unshared, or never named, so that the run cannot be repeated outside the author's machine. | required |
| artifacts-present | The repro report's shown output: command transcripts, log excerpts, error text, screenshots. | At least one artifact produced by the author's own run is shown in the report. A description of what happened, a summary of the outcome, or a diagnosis of the cause is not an artifact, however confident or detailed it is. | required |
| artifact-matches-narration | The artifacts, read against both the issue's described behavior and the report's own statement of what it established. | The artifacts show what the report says they show. If the report says the issue reproduced, the artifact exhibits the issue's described symptom (the same error, exit status, or observable behavior), produced by the issue's actual trigger rather than an altered or adjacent one. If the report says it could not reproduce, the artifact shows the genuine attempt and its real outcome. Fail when the shown output is a different failure from the reported one, when it only demonstrates that the software runs, or when the trigger was changed so the artifact cannot bear on the issue. | required |
| outcome-honest | Every statement the claim comment and the repro report make about what was established, read against what the artifacts actually show, plus any difference between the setup tested and the target the issue names. | The package asserts no more than its artifacts support, and any deviation from the issue's stated target (a different version, operating system, build, or configuration) is stated in the text rather than left silent. An evidenced cannot-reproduce passes in full: reporting honestly that the bug did not appear, and naming what differed, is a correct outcome. Fail on certainty the artifacts do not carry ("guaranteed reproducible", "I verified the race condition") and on a silent deviation from the issue's target. | required |
| claim-specific-and-honest | The claim comment, read against the issue it is posted on. | The claim names something specific to this issue (its symptom, the file or function involved, the configuration, the version) that could not be pasted unchanged onto a different issue, and it says what the author will do next. It promises investigation and a report, not a fix and not a delivery date. Fail on interchangeable boilerplate, on a bare "+1" or "me too" with no stated intent, on a request to be assigned carrying no concrete next step, and on a promised fix or deadline. | required |
| conventions-and-disclosure | The repo-facts block's contribution policy and bug-report template asks, read against both comments. Treat the package as AI-assisted work. | Pass when the comments satisfy the repo's stated rules about how contributors write. Specifically: (a) if the policy requires disclosing AI assistance in a scope that reaches issue comments, meaning it is stated for comments or applies to AI use "in any form" without limiting the ask to pull requests, then at least one comment states that AI assistance was used; (b) if the policy requires comments be written by the contributor in their own words, the comments read as human-written rather than generated boilerplate; (c) if the policy states no AI requirement, or limits its disclosure ask to pull requests and code, the check passes with no disclosure needed. | required |
| control-run-shown | The repro report's artifacts. | A contrasting run is shown alongside the failing one: the working case, the unaffected version, or the input that does not trigger the bug. | preferred |
| next-step-concrete | The closing of the claim comment or the repro report. | The author names a specific next action (a function to read, a patch to test, a configuration to try) rather than a general offer to help. | preferred |

## Verdict rule

Accept if and only if all seven `required` checks grade `pass`. Any required
check grading `fail` produces `reject`: the package is held, not posted.

`unclear` counts as `fail` on a required check. Proof a reader cannot verify
is proof that is not ready to go upstream.

The two `preferred` checks never change a verdict. They rank packages that
already pass: among ready packages, one showing a control run and naming a
concrete next step is the stronger contribution.

**Claim-only drafts (live mode).** When the package is a claim comment with no
repro report yet, the five checks whose evidence is the repro report
(`env-recorded`, `steps-followable`, `artifacts-present`,
`artifact-matches-narration`, and the report half of `outcome-honest`) are
reported `unclear` with evidence `not yet applicable: claim-only draft` and
left out of the verdict. The verdict then rests on `claim-specific-and-honest`,
`conventions-and-disclosure`, and the honesty of the claim comment's own
assertions, and it answers only: is this claim comment ready to post?
