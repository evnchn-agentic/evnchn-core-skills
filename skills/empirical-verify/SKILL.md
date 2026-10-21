---
name: empirical-verify
description: Use when asked for "empirical" verification, testing or evidence (including typos like empircal, emperical, imperical), and before claiming that a change to a runnable project works, that a PR review is done, or that upstream work is ready. Builds a per-repo verify recipe, runs it on the change, and proves the recipe can go red with a planted bug.
---

# Empirical verify

"Verify before you claim" is the rule. This skill is the **procedure**: a recipe that persists per repo, a run on the change, and a planted bug that proves the recipe can fail.

## 1. Find or make the recipe
- Look for an existing recipe for this repo (a `verify-<repo>` skill or script in your own config). If it exists, use it and go to step 2.
- Otherwise create one in your own config, **never inside the repo tree**: an upstream checkout is not yours to litter.
- A recipe has these parts, and nothing else:
  1. **Boot**: the real entrypoint, started from **that checkout's own** environment. Add a guard that the import path or binary resolves inside the checkout, not to some other installed copy.
  2. **Act as a user**: a browser driver, a CLI invocation, or a serial/HTTP client. Keep the driver's environment outside the repo.
  3. **Evidence**: one PASS/FAIL line per feature carrying the *observed* value, plus screenshots or logs, the commit SHA and the dirty-file count.
  4. **3–5 standing features** with pass criteria, plus **one change slot** for whatever is being verified now.
  5. One script that boots, drives and always tears down: `trap … EXIT` plus HUP/INT/TERM, and a bounded stop (TERM, wait ~5 s, then KILL).

## 2. Run it on the change
Rewrite the change slot so it asserts what the change claims, at the level a user sees (DOM, HTTP response, device output), not internals. Run it against the change in a disposable checkout or worktree.

## 3. Prove the recipe can fail (planted bug)
- **New recipe:** pick a recent merged fix in the same area, revert only the fix (`git show <sha> -- <src paths> | git apply -R`), and run. It must go red *for the stated reason*, so print the observed state on failure. Then restore it by re-applying the same patch (`git show <sha> -- <src paths> | git apply`). Do not use `git checkout -- <paths>`: it also discards any uncommitted fix in those files. Run it green again.
- **Verifying your own fix:** save it first (`git diff > fix.patch`), revert, see red, then `git apply fix.patch`.
- If the recipe stays green on the plant, the recipe is blind. Say so, and fix the change slot before you trust any green result.
- Never commit the plant. Use a throwaway worktree.

## 4. Report
Report the SHA, the PASS/FAIL lines, the planted-bug result (red → green), and the screenshot or log paths. In a PR comment these go in a folded evidence block. If something could not be exercised (no hardware, a platform-conditional test skip), name it. Do not let a neighbouring green result stand in for it.
