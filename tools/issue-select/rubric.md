# Rubric: is this a good first issue?

Five required checks, one per failure family that kills first contributions:
the maintainer is alive, the repo is in use, the scope fits a newcomer, nobody
else is already on it, and the project's rules permit the way I work. Two
preferred checks rank the issues that survive.

Every recency threshold is measured against the bundle's stated capture date in
eval mode, and against today in live mode.

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| maintainer-alive | The "last 5 default-branch commits" list and the "maintainer first-response sample" in the Repo facts block (live: the repo front page commit list, and first owner/member/collaborator replies on 5 recently updated issues). | At least one of: the newest default-branch commit is dated within 90 days of the capture date, or at least one issue in the first-response sample received a first owner/member/collaborator reply within 30 days of being opened. Shipping code and answering threads are each sufficient evidence of a live maintainer; a project whose maintainers commit steadily but leave issue threads unanswered still passes. | required |
| repo-in-use | The `archived:` flag on the repo line, plus the "latest release" and "last push to any branch" lines (live: the archived banner, the Releases sidebar box, the front-page commit date). | `archived: no`, AND at least one of: latest release dated within 365 days of capture, or last push dated within 90 days of capture. A repo with no published release passes on push recency alone. | required |
| scope-bounded | The issue title, body, labels, and the full comment thread. | The issue asks for one bounded change. Fail if any of: the body or thread calls it an umbrella, tracking, meta, or mega issue, or lists sub-items meant to be split into separate work; the thread shows a design question still being debated that no maintainer has settled; a maintainer states the fix requires changes to core internals; the issue is a usage or support question rather than a change; or it is a feature request with no stated spec or acceptance criteria and an unresolved product decision inside it. Issue age, a terse body, and a missing reproduction do not fail on their own. | required |
| unclaimed | The "this issue: assignees:" and "linked PRs:" lines, plus claim language in the comment thread (live: the Assignees box, the Development box, and the thread). | No assignee is set, AND no linked PR is in `open` state (a `closed` or `merged` linked PR is an abandoned or finished attempt and does not fail this check), AND no comment claiming the work ("I'll take this", "working on this", "can I work on this") is dated within 180 days of capture, unless a maintainer has since invited new takers in the thread. | required |
| ai-policy-permits | The "contribution policy" line under Repo facts (live: `CONTRIBUTING.md` in the repo root or `.github/`, the docs it links to, and any `AI_POLICY.md`). | The stated policy does not outright prohibit AI-assisted contributions. An explicit prohibition ("we do not accept AI-generated code or documentation") fails. Conditions pass: disclosure requirements, human-review requirements, personal-understanding and testing requirements, and discouragement that stops short of prohibition. No stated policy passes. | required |
| maintainer-filed | The `author_association` of the issue opener. | The opener is OWNER, MEMBER, or COLLABORATOR. | preferred |
| labelled-friendly | The issue's labels. | The issue carries a "good first issue" (or "good-first-issue") label. | preferred |

## Verdict rule

Accept if and only if all five `required` checks grade `pass`. Any `required`
check grading `fail` produces `reject`.

`unclear` counts as `fail` on a required check: a first issue whose liveness,
scope, claim state, or contribution policy I cannot verify from the named
evidence is not a first issue I should take.

The two `preferred` checks never change a verdict. They rank the accepted
issues: among accepted candidates, one with more preferred checks passing is
ranked above one with fewer, because a maintainer-filed issue carrying a
good-first-issue label is the safest of the issues that already cleared the
required bar.
