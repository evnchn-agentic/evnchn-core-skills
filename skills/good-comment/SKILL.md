---
name: good-comment
description: Use when drafting a reply on someone else's issue or pull request — a maintainer asking which option to take, a first-time contributor's PR, a thread where you are a guest rather than the owner. Also use when a draft reply opens with a verdict, argues a defect in prose, or reads as one long paragraph.
---

# Good comment

A reply that lands shrinks four things: the decision, the ambiguity, the default voice, and the bar.

The baseline failure is not incompetence. A fresh agent given a real five-option maintainer question
produced a well-grounded, code-verified draft that violated all four rules at once (see
`reference/baseline.md`). Good prose is the failure mode.

## 1. Never outsource the decision — simplify it

The thread owner decides. Your job is to make the decision small enough to make, one step at a time.

- Lay the options out side by side so they can be compared at a glance.
- Kill a whole class of them with one stated rule ("nothing that widens a newcomer's PR").
- Then ask the single question that decides the rest ("can we meaningfully reproduce A?").
- Answer that question with evidence, and the choice makes itself.

**Never** open with a bald verdict and argue for it. A lean short-circuits the ladder: they
now have to un-convince you before they can think. If you hold a view, put it after the options, as
one line, marked as yours.

## 2. Never talk a bug — code it out

English is free-syntax. "It still leaks if a timer holds the map after the client is gone" leaves
*leaks*, *holds* and *gone* undefined, and a competent reviewer can argue the exact opposite in
equally fluent English — one did, and was wrong.

Write the smallest runnable thing that fixes each word's meaning:

```python
user.client.delete()                  # "gone"
held[0].marker(latlng=(51.5, -0.09))  # the late access
held.clear()                          # "holds" ends here
gc.collect()
assert len(maps) == 0                 # "leaks" means exactly this
```

Then run it on both trees and post the result as a table. A maintainer, a linter and the next agent
can all use that; none of them can use a paragraph. **REQUIRED SUB-SKILL:** use `mre` to shrink it,
and actually run it — the run settles it, not the argument.

## 3. Never trust the sampler — convert prose into things that carry themselves

The default output is flowing paragraphs. Measured on the baseline draft: 1,846 chars, 16 prose
sentences, **zero** bullets, code blocks, tables or folds. Measured on the reply that landed: prose
is 29% by character; bullets 43%, code 20%, one `<details>` holding another 20%.

Convert, do not compress:

| Prose doing this | Becomes |
|---|---|
| listing properties or steps | bullets, one idea per line |
| describing what the code does | the code |
| comparing two states | a two-column table |
| evidence they may not want | one `<details>` fold |

Length is not the enemy — the reply that landed was 33% *longer* than the wall of text it answered.
Restructuring the explainer cost −29 characters and changed the reading cost completely.

## 4. Never raise the contributing bar — lower it

Public threads are read by people deciding whether to contribute at all.

- Frame advanced topics as optional depth: "something you eventually stumble into once you get far
  enough into the weeds, not something anyone needs to know to contribute." Never "worth learning
  early", never "every contributor should know".
- Do not pile widenings on a first PR. Small ones that still fix the reported bug are fine; design
  changes go to a separate follow-up.
- Leave the follow-up **unowned**. Delete "happy to take that one" — the project needs new
  contributors more than it needs another you.
- Teach the thing that blocked them, in the same comment that answers them.

## Red flags — stop and restructure

- The draft opens with a letter, a verdict or a recommendation.
- A defect claim has no runnable form anywhere in the comment.
- Scroll the draft: no bullet, no code block, no table, no fold.
- The words "worth learning", "should know", "basic", "obviously", "just".
- "Happy to take that one."
- You are about to tell a first-time contributor their PR should also do X.

## Rationalizations

| Excuse | Reality |
|---|---|
| "They asked me which option, so I should answer" | They asked which option *is best*. Narrowing beats asserting; give them the evidence that decides it. |
| "A test is overkill for a review comment" | The test is what stops a fluent wrong counter-argument. One was filed here and was refuted by a run. |
| "Bullets would look choppy / unprofessional" | The reply that landed is 71% non-prose and got "a better introduction than I would have managed". |
| "Mentioning that they should learn X is helpful" | It reads as a bar to entry to every future contributor scrolling the thread. |
| "I'll just do the follow-up, it's faster" | Faster once, and the newcomer never comes back. Here the newcomer shipped it the same morning. |

## Worked example

The example thread is zauberzeug/nicegui#6345.
`reference/landed.md` is the comment that resulted (contributor handle redacted); `reference/baseline.md` is what a fresh agent
wrote for the same thread with none of this. Read both before writing a hard reply.

Outcome on that thread: maintainer replied "Agreed, C it is… a better introduction to weak
references than I would have managed", then handed the change to the first-time contributor, who
shipped it — including the half the comment left as an exercise — within 25 minutes.
