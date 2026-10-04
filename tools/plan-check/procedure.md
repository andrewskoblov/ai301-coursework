# Procedure: how this skill grades a plan package

## Read order

Read the whole package before grading anything. The order matters because
three of the six checks compare the plan against something that must already
be in your head when you reach it.

1. **Read the issue context.** Note what is actually being asked for, in one
   sentence. This is the yardstick `scope-bounded` measures against later.
2. **Read the repo-facts block.** Note two things verbatim: the contribution
   policy, including its AI rules **and the scope those rules apply to**, and
   any stated contributing asks or templates. Record the policy as one of four
   shapes: silent, permissive-with-responsibility, scoped disclosure (limited
   to pull requests or code), or unscoped disclosure ("in any form").
3. **Read the Thread highlights.** List every comment whose author is OWNER,
   MEMBER or COLLABORATOR, and for each, note whether it carries direction: a
   named culprit or file, a posted patch, a settled approach, a request to
   test, or an open pull request. If there are none, record "no maintainer
   direction" explicitly, because that is a pass condition later, not a gap.
4. **Read the Repro evidence block before the plan, never after.** Note what
   behavior it pins down and, separately, what any control run rules out. A
   control is the single most decisive artifact in the package: it is what
   turns `cause-grounded` from an opinion into a check. Write down the
   excluded causes before you have read the plan's diagnosis, so the plan
   cannot anchor your reading of the evidence.
5. **Read the candidate plan**: its stated cause, its in-scope and
   not-in-scope statements, the files or areas it names, its approach, its
   test plan, and its risks.
6. **Read the candidate plan comment** last, as a maintainer on the thread
   would read it.

## Evidence gathering

For each check, pull the fact from exactly this place and record it as a
quotable line.

1. **cause-grounded.** From step 4, the list of behaviors the repro pins down
   and the causes its controls exclude. From step 5, the plan's stated cause
   as a single sentence. Put the two side by side before grading.
2. **scope-bounded.** From step 5, every work item the plan commits to, listed
   out. Mark each as either answering the issue from step 1, or as additional.
   Record separately any item the plan explicitly defers and the reason given,
   because deferrals are evidence of bounding, not of creep.
3. **buildable.** From step 5, the named files, functions or call sites, and
   the chosen approach. Record the specific location strings. If the plan
   offers alternatives, record whether one is chosen.
4. **test-decisive.** From step 5, the test plan verbatim. From step 4, the
   repro's steps and artifacts. Record whether the test names a command or
   case and a result that someone could observe and dispute.
5. **thread-engaged.** From step 3, the maintainer direction list. From steps
   5 and 6, any sentence in the plan or comment that adopts, builds on, or
   argues against each item of direction. Record direction with no
   corresponding sentence.
6. **conventions-and-disclosure.** From step 2, the policy shape. From step 6,
   any sentence disclosing AI assistance, and whether the comment reads as
   written by a person.

In live mode, gather the same families from: the issue page and its thread for
steps 1 and 3; `CONTRIBUTING.md`, any `AI_POLICY.md` and the issue template for
step 2; the student's own posted repro comment on the issue for step 4; and
`plan.md` and the draft comment for steps 5 and 6. `references/evidence-guide.md`
gives the exact locations for each.

## Check execution

1. Execute the checks in this order: `cause-grounded`, `scope-bounded`,
   `buildable`, `test-decisive`, `thread-engaged`,
   `conventions-and-disclosure`, then the two preferred checks. Grounding runs
   first because a plan built on an excluded cause is not worth grading for
   buildability, though you still grade every check and report all of them.
2. Grade each check only from the evidence recorded in the gathering stage. Do
   not re-read the package for a check whose evidence is already recorded; if
   a check's evidence was not recorded, go back and record it before grading,
   rather than grading from memory.
3. Apply the rubric's pass condition literally. Where the condition names a
   failure shape, the check fails only if the package matches one of those
   shapes. Where it names an exemption (a stated deferral under
   `scope-bounded`, a thread with no maintainer direction under
   `thread-engaged`, a policy limited to pull requests under
   `conventions-and-disclosure`), apply the exemption rather than the general
   rule.
4. When evidence for a check is genuinely absent from the package, grade it
   `unclear` and say what was missing. Absent means you looked in the place
   the evidence guide names and it is not there. It does not mean the evidence
   is present but weak: weak evidence gets graded `pass` or `fail` on the
   rubric's condition.
5. Record one line of evidence for every grade: the quote or fact that decided
   it, not a restatement of the check's name.
6. Grade the thing, never the polish. A terse plan that names a file, a cause
   the evidence supports, and a re-run of the repro passes every check it
   needs to. A long confident plan with headings and no named file does not.

## Verdict assembly

1. Collect the six required grades. Treat every `unclear` on a required check
   as a `fail`, per the rubric's verdict rule.
2. If all six required checks are `pass`, the verdict is `accept`. If any one
   is `fail`, the verdict is `reject`. There is no third verdict and no
   weighing of how many passed: one required failure holds the package.
3. The preferred checks never enter the verdict. Report their grades and, on
   an accepted package, mention them in the summary as reasons to prefer this
   plan over another that also passed.
4. Name the deciding check in the summary. On a reject, that is the first
   required check that failed, and the output quotes the evidence line that
   failed it. On an accept, the summary says which checks were closest to
   failing, so the student knows where the package is thin.
5. Emit the fenced JSON block last, with every check's name, grade and
   evidence line, and the verdict. Nothing follows the JSON block.
