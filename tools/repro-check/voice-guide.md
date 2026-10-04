# Voice guide: how I talk upstream

## Who I am in threads

I am a student contributor working through a course, and I say so rather than
performing seniority I do not have. What I am doing in this repo is narrow: I
take one issue, reproduce it in my own environment, and post what I actually
observed. Readers can expect me to show my commands and my output, to say
plainly when something did not work or I did not understand it, and never to
describe an outcome I did not see. Being a beginner is not an excuse for a bad
comment, but it is a reason to promise less.

## Rules I write by

### Rule: Promise the investigation, never the fix

When I claim an issue I say what I am going to look at and what I will post
back. I do not promise a fix, and I never give a date, because I do not yet
know what the fix is or whether I can write it.

- Wrong: "I'll take this one and have a PR up by Friday."
- Right: "I'd like to work on this. Next I'll reproduce it on 1.3.1 and post
  the environment and output before I propose anything."

### Rule: Every claim about behavior comes with the output that shows it

If I say something happened, the command and its result are in the comment. If
I have no artifact, I do not make the statement. This is the rule I break when
I am tired and want to sound useful.

- Wrong: "Confirmed, this is definitely a race condition in the debounce
  handler."
- Right: "Reproduced on my machine; transcript below. I haven't isolated the
  cause yet, so I'm not going to guess at one."

### Rule: Name what differed from the reporter's setup

If my version, OS, or configuration is not the one the issue targets, that
goes in the comment where the reader cannot miss it, not buried or omitted.
An unstated deviation makes my evidence worthless to the person reading it.

- Wrong: "Reproduced, here's the traceback." (on an older version than the
  issue reports)
- Right: "Reproduced on 1.5.3; the issue reports this on main, so this may be
  the older behavior rather than the same bug. Traceback below."

### Rule: A cannot-reproduce is a real result, posted like one

If the bug does not appear, I post that with the same evidence I would post
for a success. I do not quietly drop the issue or keep retrying until I get
something postable.

- Wrong: (saying nothing, and moving to a different issue)
- Right: "I could not reproduce this on Linux/zsh with the exact config from
  the issue; prompt output below. The reporter is on macOS/fish, and I suspect
  the PWD resolution differs there."

### Rule: Write it myself, then say the tool was in the room

The comment is in my own words, not generated text I pasted. Where the repo's
policy asks for disclosure of AI assistance, I say so plainly in one line
rather than hoping nobody asks.

- Wrong: a polished paragraph I did not write, posted with no note.
- Right: "I used an AI assistant to help organize this report. I ran every
  step myself and I understand what I'm reporting."

## Things I never post

- A promised delivery date, or a promised fix, for work I have not done.
- "Same as above", "+1", "can confirm" with nothing of my own underneath.
- A root cause I have not isolated with an experiment.
- A confident summary standing in for an artifact I did not capture.
- "Any update on this?" to a maintainer who owes me nothing.
- A comment asking to be assigned that says nothing about what I will do.
