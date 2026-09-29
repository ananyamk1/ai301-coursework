# Voice guide: how I talk upstream

## Who I am in threads

I am a student contributor making my first contributions to open
source, and I say so rather than performing seniority I do not have. In
a repo I am there to do one small, concrete thing: reproduce a reported
bug and, if that goes well, propose a narrow fix. What a maintainer can
expect from me is that anything I state as fact, I ran myself and can
show, and anything I am unsure about is labelled as unsure.

## Rules I write by

### Rule: promise the investigation, never the outcome

I commit only to what is inside my control — looking, reproducing,
reporting back. I do not commit to a fix, a date, or a result, because
I do not yet know what the fix costs, and a missed promise costs a
maintainer more than no promise would have.

- Wrong: "I'll take this one and have a PR up by Friday."
- Right: "I'd like to investigate this one. I'll reproduce it first and
  report back here with what I find before I open anything."

### Rule: every claim carries its receipt in the same comment

If I write that something happens, the command and the output that show
it happening are in the comment I am posting, not in my terminal
history or my memory of last night. If I have no artifact, I write it
as a question or a hypothesis instead of as a finding.

- Wrong: "Confirmed, this definitely crashes on an empty index."
- Right: "On an empty index this raises `ZeroDivisionError`; full
  transcript below, and the same call on a one-document index returns
  normally."

### Rule: say what I did not do

Scope creeps quietly in a write-up. If I tested one of two cases, one
platform, one version, I name the boundary out loud so nobody reads my
result as wider than it is.

- Wrong: "Reproduced — this affects all platforms."
- Right: "Reproduced on macOS 15 / Python 3.12; I have not tried this
  on Windows, and case 1 in the report (`--batch-size`) I did not test
  at all."

### Rule: a negative result is a result, and I post it plainly

If I cannot reproduce something, I say cannot-reproduce in the first
line, show the attempt anyway, and name what differed from the
reporter's setup. I do not hedge it into sounding like a partial
success, and I do not quietly drop the issue instead.

- Wrong: "Hmm, I mostly got it working, might just be my machine, not
  sure this is real?"
- Right: "I could not reproduce this. Attempt and environment below;
  the likely difference is that my `ARG_MAX` is 2 MiB, so my two
  commands flush at the same boundary."

### Rule: disclose the AI assistance where the repo asks for it

I draft with an AI assistant in the loop. Where a repo's policy asks
for AI usage to be disclosed, I disclose it in the comment itself, in
one plain sentence naming the tool and what it did — and I only post
what I have actually run and understood.

- Wrong: posting a polished report with no mention of the assistant,
  in a repo whose AI policy says all AI usage must be disclosed.
- Right: "Disclosure per the contributing guide: I used Claude to help
  organise this write-up. I ran every command shown here myself and I
  understand what it reports."

## Things I never post

- A deadline, an ETA, or "guaranteed".
- "Fixing this now" before I have reproduced the bug.
- "Same here" / "+1" / "any update on this?" with nothing added.
- A root cause I have not seen in a transcript, stated as if I had.
- Flattery as an opener ("great project, love your work") to soften an
  ask — it is filler, and it makes the ask the real content.
- A wall of headings and emoji over three facts; if it is three facts,
  I post three lines.
- Anything about someone else's comment that I would not say with them
  reading it, because they are.
