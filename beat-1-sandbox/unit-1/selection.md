# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

---

## Selected issue

**Issue link**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61

**Verdict output**

```
Only #61 is worth taking. #62 and #57 each already have an open pull request linked to them, which fails the `unclaimed` check.

**Ranked read-out**

1. **#61 (accept).** The health check's database probe passes a raw `"SELECT 1"` string, which SQLAlchemy 2.x rejects.
   - **Why it fits you:** it's a backend Python bug in your SQLAlchemy and FastAPI territory, a one-line fix in `api/routes/health.py` with a clear reproduction and a small blast radius.
   - **Tests:** the issue gives you a real root cause to confirm, and a test that pins the `text()` behavior is a good exercise. You can write it against a mocked session without Docker. Only the end-to-end `/health` check needs the Postgres container.
   - **Claim comments:** the thread has many classmate claims, but under the Path Review house rule they don't block it. No assignee is set and no PR is linked.
   - **Your PR:** per `docs/CONTRIBUTING.md`, it should also drop the `call-overload` suppression for this module in `pyproject.toml`.
   - **Overlap with #62:** fixing #61 alone won't turn `/health` green. The Redis check (#62) fails separately.
2. **#62 (reject).** Open PRs #76 and #86 are linked to it, and the thread has about 26 comments.
3. **#57 (reject).** Open PR #82 from `whitgunt77` is linked to it.

I read `docs/CONTRIBUTING.md` and the PR template and found no AI policy or prohibition. The repo has no `AI_POLICY.md` or `AGENTS.md`. I did not sample maintainer first-response times, since the recent commits already settle `maintainer-alive`. I also didn't fetch #61's `author_association`. I assumed COLLABORATOR because the same opener, `Aburke225`, is COLLABORATOR on #62, #68, #72 and #26.

```json
[
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61",
    "checks": [
      {"name": "maintainer-alive", "grade": "pass", "evidence": "Newest default-branch commit 2026-09-16 (18 days before today), authored by Aburke225"},
      {"name": "repo-in-use", "grade": "pass", "evidence": "archived: false; no releases; last push 2026-09-16, within 90 days"},
      {"name": "scope-bounded", "grade": "pass", "evidence": "Single bug, one-line fix: wrap 'SELECT 1' in sqlalchemy.text() at api/routes/health.py:32; labelled good first issue, tier-1"},
      {"name": "unclaimed", "grade": "pass", "evidence": "assignees: []; closedByPullRequestsReferences: []; many claim comments but ignored per Path Review house rule"},
      {"name": "ai-policy-permits", "grade": "pass", "evidence": "docs/CONTRIBUTING.md and PR template state no AI policy; no AI_POLICY.md or AGENTS.md"},
      {"name": "maintainer-filed", "grade": "pass", "evidence": "Opener Aburke225 (assumed COLLABORATOR; not fetched for #61, COLLABORATOR on #62, #68, #72, #26)"},
      {"name": "labelled-friendly", "grade": "pass", "evidence": "Labels: bug, good first issue, api, tier-1"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/62",
    "checks": [
      {"name": "maintainer-alive", "grade": "pass", "evidence": "Newest default-branch commit 2026-09-16"},
      {"name": "repo-in-use", "grade": "pass", "evidence": "archived: false; no releases; last push 2026-09-16"},
      {"name": "scope-bounded", "grade": "pass", "evidence": "Single bug: Redis probe uses settings.redis_host/redis_port, which Settings does not define; good first issue label"},
      {"name": "unclaimed", "grade": "fail", "evidence": "Linked PRs #76 (open, Tiyatrotist) and #86 (open, alexh30486-ui, 2026-10-04)"},
      {"name": "ai-policy-permits", "grade": "pass", "evidence": "No AI policy stated in docs/CONTRIBUTING.md or PR template"},
      {"name": "maintainer-filed", "grade": "pass", "evidence": "Opener Aburke225, COLLABORATOR"},
      {"name": "labelled-friendly", "grade": "pass", "evidence": "Labels: bug, good first issue, api, tier-1"}
    ],
    "verdict": "reject"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/57",
    "checks": [
      {"name": "maintainer-alive", "grade": "pass", "evidence": "Newest default-branch commit 2026-09-16"},
      {"name": "repo-in-use", "grade": "pass", "evidence": "archived: false; no releases; last push 2026-09-16"},
      {"name": "scope-bounded", "grade": "pass", "evidence": "Single bug: tech_detector _should_skip_file misses root-level node_modules/ and build/ paths; good first issue label"},
      {"name": "unclaimed", "grade": "fail", "evidence": "Linked PR #82 (open, whitgunt77, 2026-10-02)"},
      {"name": "ai-policy-permits", "grade": "pass", "evidence": "No AI policy stated in docs/CONTRIBUTING.md or PR template"},
      {"name": "maintainer-filed", "grade": "pass", "evidence": "Opener Aburke225 (COLLABORATOR on sibling issues)"},
      {"name": "labelled-friendly", "grade": "pass", "evidence": "Labels: bug, good first issue, agent, tier-1"}
    ],
    "verdict": "reject"
  }
]
```
```

---

## Eval iterations

**Run history**

1. Full run, 20 scored items: **17/20**, below the bar. The category floor was already
   met (claimed 4/4, clear-accept 6/8, dead-repo 3/3, policy 1/1, scope 3/4). All three
   disagreements were the same check, `scope-bounded`: issue-04 and issue-19 graded
   reject against gold accept, and issue-20 graded accept against gold reject.
2. Partial canary, `--only issue-04,issue-19,issue-20`: **3/3**. Run after rewriting
   `scope-bounded`, to confirm the rewrite flipped exactly the three items it was aimed
   at before paying for another full run. Partial runs are not scored and this one did
   not write `eval-run.txt`.
3. Full run, 20 scored items: **20/20**, bar PASS, every category complete (claimed
   4/4, clear-accept 8/8, dead-repo 3/3, policy 1/1, scope 4/4). This is the run saved
   in `eval-run.txt`.

One change was made before run 1 rather than between runs, so it has no score of its
own: `maintainer-alive` originally required recent commits AND a maintainer reply
within 30 days. Every comment in the Path Review tracker carries
`author_association: NONE`, so the conjunctive form graded fail on every live candidate
and the skill could not have accepted anything. It became a disjunction.

**Issue analysis**

`issue-20` (excalidraw/excalidraw#11811, "Add company logo shape to the toolbar").
Gold label: `reject`. My rubric graded it **accept on run 1** and **reject on run 3**.

On run 1 the issue passed all five required checks. It deserved to pass four of them:
excalidraw is not archived, pushed 2026-08-04 one day before capture, has release
v0.18.1, no assignee, no linked PRs, no claim comments, and a CONTRIBUTING.md with no
statement on AI. The check that should have caught it was `scope-bounded`, and in its
first form it did not, because the body is not obviously unbounded. It has a filled-in
feature-request template, a "Success looks like" line, a named likely surface
(`packages/excalidraw`), and an explicit out-of-scope note. By the wording I had
written, that read as a specified piece of work.

What the first wording could not see is who wanted it. The issue was opened by
`cursor[bot]` with `author_association: NONE`, carries **no labels at all**, and has
zero comments, so no maintainer has ever said excalidraw wants a company-logo tool in
the toolbar. The product decision is still open, and the polish of the write-up hides
that. The fix was to make `scope-bounded` fail a feature request that no maintainer has
endorsed. That same clause has to leave `issue-09` alone, which is also a feature
request and a gold accept: it was opened by a MEMBER, carries a good-first-issue label,
and a maintainer answered a would-be contributor with "Think you can just give it a
try." Endorsement, not write-up quality, is what separates the two.

**Check rationale**

Quoted from `tools/issue-select/rubric.md` as currently written:

| scope-bounded | The issue title, body, labels, the opener's author_association, and the full comment thread. | The issue asks for one change a newcomer could ship in a single pull request. Fail if any of: (a) the issue is explicitly an umbrella, tracking, meta, or mega issue, or its body is a checklist of independently shippable items meant to be split across separate pull requests; (b) the thread shows a design question still being debated that no maintainer has settled; (c) a maintainer states the fix requires changes to core internals; (d) it is a usage or support question rather than a change; (e) it is a feature request that no maintainer has endorsed, where endorsement means the opener is OWNER, MEMBER or COLLABORATOR, or a maintainer comment invites someone to take it, or the project has applied a good-first-issue label. The following are explicitly NOT failures: an illustrative list of examples inside one change ("including X, Y, Z, etc."); a maintainer's list of candidate causes or possible approaches for a single bug; the issue's age; a terse body; a missing reproduction. | required |

Clause (e) and the closing exemption list are both products of run 1. Clause (e) exists
for issue-20 above. The exemption list exists for the opposite failure in the same run:
issue-04 and issue-19 are both gold accepts that my first wording rejected, because it
failed anything that "lists sub-items meant to be split into separate work". issue-04
says the change covers rewrites "including remove identity, fuse spiders, remove self
loops, etc.", and issue-19 is a maintainer-diagnosed performance bug naming two causes
and three possible approaches. Neither is a list of separate pull requests. One is a
list of examples inside a single change, the other is a maintainer showing their
diagnostic work. Both are signs of a well-described issue, and the first wording
punished them for it. Naming the exemptions explicitly was the only way to keep the
umbrella clause strict for issue-05 (a codebase-wide type-annotation umbrella) and
issue-10 (a self-described megaissue) without it firing on ordinary enumeration.

**Trade-offs**

Clause (e) gives up unendorsed feature requests that are nonetheless good work. A
newcomer-sized, well-specified feature filed by an outside contributor that no
maintainer has labelled or answered yet now fails, even when the change itself would be
a fine first contribution. That is a real cost, and it is one I chose: on a feature
request, silence from the maintainers is the risk, because the work can be perfect and
still be closed as unwanted. For a bug the same silence matters much less, which is why
clause (e) is scoped to feature requests only and does not touch issue-01, issue-06,
issue-11, issue-14, issue-16 or issue-19.

I verified the rewrite changed only what it was meant to change rather than assuming it.
After editing `scope-bounded` I re-ran the three disagreements as a canary with
`--only issue-04,issue-19,issue-20`, which returned 3/3: issue-04 and issue-19 flipped
to accept and issue-20 flipped to reject. The full run that followed came back 20/20
with no new disagreements anywhere else, so the other 17 items were unaffected.

---

## Selection rationale

**Selection rationale**

1. **Fit and time.** #61 is a backend Python bug in the part of the stack I have
   actually used: the health route passes a raw `"SELECT 1"` string to SQLAlchemy 2.x,
   which rejects it, and the fix is wrapping it in `text()` in `api/routes/health.py`.
   It is one line of production change, so almost all of the work is confirming the
   failure and writing a test that pins it, which is the part I wanted practice at. I
   can run that test against a mocked session without standing up Docker, which matters
   because I did not want a first issue that depends on infrastructure I cannot run
   locally.

2. **What the verdict got right, and what it could not weigh.** The rubric was right
   that #61 is live, bounded, policy-clear and genuinely unclaimed: no assignee and no
   linked PR, while #62 and #57 both already have open PRs against them (#76, #86 and
   #82). That open-PR distinction is the thing I would have gotten wrong by eye, since
   all three issues look equally busy in the comment thread and the house rule says to
   ignore classmate claim comments. What the rubric cannot weigh is that #61 does not
   fully fix `/health` on its own: the Redis probe in #62 fails separately, so the
   endpoint stays red until both land. I am taking that as acceptable because the issue
   is scoped to the database probe rather than to making the endpoint green, but a
   rubric grading one issue in isolation has no way to see that coupling.

3. **Anticipated difficulty in claiming it.** The thread is crowded, with roughly a
   dozen classmates having already commented that they intend to work on it. Under the
   Path Review house rule that does not block me, and credit attaches to the PR I open
   rather than to whether it merges, so the real risk is not being blocked but being
   third or fourth to open a near-identical one-line diff. I plan to make the test the
   distinguishing part of my PR, and to follow the `docs/CONTRIBUTING.md` instruction to
   also drop the `call-overload` suppression for this module in `pyproject.toml`, which
   several of the claim comments do not mention.

---

Related paths: `eval-run.txt` in this directory; the skill's files in `tools/issue-select/`.
