# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

---

## Selected issue

**Issue link**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/68

**Verdict output**

> TODO: paste the live-mode output here verbatim, ending with the fenced JSON block.
> Produced by: claude "issue-select: grade these candidate first issues: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/68 https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61 https://github.com/codepath/pathreview-ai301-fa26-s1/issues/53"

```
paste the output here, including the closing JSON block
```

---

## Eval iterations

**Run history**

> TODO: fill in from the real runs. Format: "Run 1: NN/20 ... Run 2: NN/20".
> The last score here must match the agreement line in the committed eval-run.txt.

**Issue analysis**

`issue-12` (bookwyrm-social/bookwyrm#1133). Gold label: `reject`.

This is the single item in the `policy` category, and it is the reason the rubric
carries an `ai-policy-permits` check at all. The bundle passes every other family:
`archived: no`, last push on the capture date itself (2026-08-12), a maintainer
first-response sample containing replies at 0.3 and 0.5 days, no assignee, no linked
PRs, and a bounded UI request with a maintainer already describing the implementation
path in the thread. The only claim comment is dated 2024-09-19, which is 693 days
before capture and therefore stale under the 180-day window in the `unclaimed` check.

What sinks it is the repo-facts contribution-policy line, which quotes
docs.joinbookwyrm.com section "Generative AI": "We do not accept AI-generated code or
documentation." That is an outright prohibition rather than a condition, so
`ai-policy-permits` grades `fail`, and the verdict rule turns a single required
failure into `reject`.

> TODO: state what YOUR run actually decided for issue-12 and whether it matched.

**Check rationale**

Quoted from `tools/issue-select/rubric.md` as currently written:

| maintainer-alive | The "last 5 default-branch commits" list and the "maintainer first-response sample" in the Repo facts block (live: the repo front page commit list, and first owner/member/collaborator replies on 5 recently updated issues). | At least one of: the newest default-branch commit is dated within 90 days of the capture date, or at least one issue in the first-response sample received a first owner/member/collaborator reply within 30 days of being opened. Shipping code and answering threads are each sufficient evidence of a live maintainer; a project whose maintainers commit steadily but leave issue threads unanswered still passes. | required |

The first version of this check required both halves: recent commits AND a maintainer
reply within 30 days. I changed it to a disjunction after checking the live Path Review
repo, where every comment on every candidate issue carries `author_association: NONE`.
There are no maintainer replies anywhere in that tracker, so the conjunctive form graded
`fail` on every candidate and the skill could not accept any issue at all.

Measuring the two halves separately against the three `dead-repo` bundles showed the
response-latency half was doing no work the commit half was not already doing:
issue-02 (last push 2024-06-19, 777 days before capture) and issue-07 (2025-02-27,
524 days) have no maintainer comment in any sampled thread, so both halves fail
regardless of how they are combined.

**Trade-offs**

The disjunction gives up the ability to reject a repo that ships code steadily but
ignores its contributors. That is a real category of dead end: a project with an active
core team whose outside PRs sit unreviewed.

issue-17 (jarun/googler) is the case that shows it. Its sampled response times are
0.2, 1.4 and 39.2 days, all fast, so under the disjunction it now *passes*
`maintainer-alive` where the conjunctive version failed it on commit recency
(last push 2021-11-13). Its verdict does not change, because `repo-in-use` fails it
independently on `archived: yes`. So the loosening costs nothing on this eval set, but
only because a second required check happens to cover the same repo. A repo that was
stale-but-unarchived with responsive maintainers would slip through, and I accept that
miss in exchange for the check working at all in a classroom tracker where maintainers
do not answer threads.

---

## Selection rationale

**Selection rationale**

> TODO: answer all three in your own words. Graded on being answered, not on quality
> or length. A short honest answer to each earns full marks.
>
> 1. The issue's fit to your interests and to the time available.
> 2. What the verdict identified correctly, and what you weighed that the rubric could not.
> 3. The anticipated difficulty in claiming it.

---

Related paths: `eval-run.txt` in this directory; the skill's files in `tools/issue-select/`.
