---
name: mre
description: Use when writing or improving a minimal reproducible example (MRE) for a bug report, GitHub issue, or PR — distilling a bug into the smallest demo a maintainer can paste and run. Triggers include "write an MRE / repro", "minimal example", "repro for this bug", a flaky/timing bug you must make deterministic, a tracker asking for a playground/version/bisect, or filing/sharpening a bug issue.
---

# Writing an MRE

An MRE is a **communication artifact for someone else** — a maintainer who will paste it, run it, and watch it fail. It is judged by *them*, not by whether you proved the bug to yourself. **A good writeup never substitutes for a runnable minimal repro:** trackers routinely close "incredible bug reports" that lack one. So: smallest **real-public-surface** program that shows **one observable symptom**, runnable (or a hosted repro link), with exact version — and you ran it.

## The recipe

1. **Runnable, real public surface, right context.** *(The load-bearing rule.)* The smallest program a maintainer can paste and run, using the public API/CLI the way a user does, in the context the bug needs (the page/request, or the right flags/target/build-profile). Where an ecosystem expects it, a **hosted repro / playground link counts as "runnable" — and several trackers auto-close issues without one.**
2. **One observable symptom, machine-checkable.** Do X → see Y, one symptom. Prefer a check over an eyeball: pipe output through a runtime (`… | node` → a real `TypeError`), assert an invariant (`Parse(u.String()) == u`), emit the IR — so "it's wrong" isn't a judgment call.
3. **Expected vs actual — ideally a works/broken differential.** State both, or let a self-evident invariant stand in. Strongest form: put the nearest *passing* variant beside the failing one (a one-line toggle that flips the bug). It localizes the cause and doubles as a determinism control.
4. **Exact version + platform; bisect if you can.** Give the exact version (and OS/arch where it matters) — most trackers require it. If you can, bisect to the breaking commit/PR; tooling exists (`cargo-bisect-rustc`, pinned per-run installs, CI version sweeps). A works-on-X / broken-on-Y contrast is the strong form.
5. **Engineer determinism for non-deterministic bugs.** Timing/async/order/memory bugs need a device that forces the failure every run — forced GC + finalizer order, concurrency volume, a fixed seed, payload padding, an ordering toggle. The device varies by ecosystem; the requirement doesn't. Skip for ordinary deterministic bugs.
6. **Run it; show real output honestly.** Paste the actual failure — traceback, panic, wrong output, IR diff. If you couldn't run it (needs a GUI/hardware), say so plainly; never imply you ran it when you didn't.
7. **Smallest that still triggers it.** Strip to one file, drop external deps, cut every non-load-bearing line. Several trackers auto-close non-minimal reports regardless of prose quality.

Keep internals, mechanism, and source links in the prose *around* the code — never in the runnable repro.

## Quick reference

| Do | Not |
|---|---|
| Real public surface in the repro; runnable or hosted/playground link | Private/internal calls; a fragment that won't run |
| Exact version + platform; bisect to the culprit PR/commit | No version; vague "latest" |
| Nearest passing variant beside the failing one (works/broken toggle) | Only the failing case, cause un-localized |
| Machine-checkable symptom (assert / pipe / IR) | "Looks wrong" by eyeball |
| Engineer determinism for flaky/timing bugs | A repro that fires ~half the time |
| Run it; paste real output — or say you didn't | "This raises…" implied from reading code |
| Smallest: one file, deps dropped | Kitchen-sink; extra widgets/routes/packages |

## Real public surface in the repro; internals only in the diagnosis

Internal-method calls *feel* like a tighter proof, but a maintainer can't paste-and-run them, and — worse — they can **"prove" a bug via a code path no real call ever hits.** Drive the public surface the way a user does. If the symptom *only* appears via internals or a path you can't run normally, **state that explicitly** ("I could only trigger this by calling `_match_path` directly") — it changes the bug; don't dress it up as an MRE.

## Engineer determinism (the move most people skip)

A repro that fires half the time wastes the maintainer's run. For any non-deterministic bug, build in a device that forces the failure — examples seen across ecosystems:

- **Force GC / finalizer ordering** — `runtime.SetFinalizer` + a double `runtime.GC()` to make a liveness bug deterministic; inject a dead `weakref` into the GC's object list to force a Python race.
- **Pad / shape the payload** — inflate a streamed frame so the client paints it immediately and reliably.
- **Fix async ordering** — `await asyncio.sleep(0)` so a waiter subscribes before the event fires; a short reconnect/timer timeout to provoke a teardown race.
- **Surface invisible state** — multi-cycle loop + `tracemalloc`/`gc` or `objgraph.by_type(...)` counts for leaks; a fixed seed; an emitted-IR diff.

## Example

A near-identical works/broken pair makes the bug undeniable and pre-localized — here the failure appears only inside a container, so the passing case sits right beside it:

```python
from nicegui import ui

@ui.page('/')
def index():
    ui.button('works', on_click=do_refresh)          # fine
    with ui.card():
        ui.button('breaks', on_click=do_refresh)     # same handler, raises here
```

State the exact version, paste the real traceback, and (if known) name the culprit PR — in the prose, not the snippet.

## Common mistakes

- **Internal-method crutch in the repro.** A "minimal proof without running a server" calling private functions. Move internals to the diagnosis; drive the public surface. If only internals reproduce it, say so — it changes the bug.
- **Prose instead of a repro.** A polished root-cause writeup with no runnable minimal repro gets closed regardless of how good it reads. The repro is load-bearing.
- **Claim-verified, not run.** "Confirmed empirically, reproduces as shown" with no pasted output and no run. Run it and paste real output, or disclaim that you didn't.
- **No hosted repro where the tracker requires one** → auto-closed (`needs reproduction`).
- **No version / "latest".** Give the exact version + platform; bisect if you can.
- **Eyeballed symptom.** "Looks wrong" instead of an assert / pipe-to-runtime / invariant check.
- **Non-deterministic repro / synthetic setup that bypasses the real path** (a gc monkeypatch, `dispatchEvent`). Engineer real determinism; use the real trigger.
- **Wrong context.** Code that needs a page/request/event-loop — or the right flags/target/profile — run in the wrong one fails differently or not at all.
- **Kitchen-sink.** Every non-load-bearing line is something the maintainer must rule out.

## Related

- **Run it** — recipe step 6: run it and observe before claiming.
- **`supercomment`** — how to present it (glanceable symptom up top, full traceback folded as evidence, version + mechanism + culprit-PR links below).
- A diagnostic-MRE is a sibling form: an instrument you hand the *reporter* to gather data, not a confirmed repro — label it as such.
