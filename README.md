# evnchn core skills

Six Claude Code skills in one plugin, for agents that comment on GitHub, review work, prove things work, and show a finding as one image. A small core on purpose. Each started as a skill used on one person's setup and was rewritten here without the setup-specific parts.

## Install

```
/plugin marketplace add evnchn-agentic/evnchn-core-skills
/plugin install evnchn-core-skills@evnchn-core-skills
```

Or from a shell: `claude plugin marketplace add evnchn-agentic/evnchn-core-skills`, then `claude plugin install evnchn-core-skills@evnchn-core-skills`.

## Skills

| Skill | What it does |
|---|---|
| `supercomment` | GitHub comments and PR bodies with a self-sufficient TL;DR on top and the evidence folded in `<details>` for whoever reads the thread later. |
| `good-comment` | Replying as a guest on someone else's issue or PR: narrow the decision instead of handing down a verdict, show the bug as code, keep the bar low for newcomers. |
| `mre` | The smallest runnable, public-API repro a maintainer can paste and watch fail, with exact version and a works/broken pair. |
| `calibrated-adversarial-review` | A reviewer prompt that finds the real problems without burying them in nitpicks: severity tiers, "most of this is sound" allowed, a 0–10 closer. |
| `empirical-verify` | A per-repo verify recipe that boots the real app and drives it like a user, plus a planted bug that proves the recipe can go red. |
| `explain-with-image` | Turn a verified finding, bug or design into one 16:9 image for an MR, issue or chat message: thesis as the title, evidence you reproduced first, checked by looking at the render. |

## If you already use another pack

These are comparisons, not lineage. None of these skills is derived from the packs below, except the one credited borrowing noted.

| Skill | Nearest counterpart |
|---|---|
| `supercomment` | A bit like Matt Pocock's [`pr`](https://github.com/mattpocock/skills/blob/49dd158d1076/skills/engineering/pr/SKILL.md) (a PR-body template), plus a TL;DR and folded-evidence shape for any comment, and a one-line "mini" variant. |
| `good-comment` | A bit like pstack's [`technical-writing`](https://github.com/cursor/plugins/blob/d457567332aa/pstack/skills/technical-writing/SKILL.md), but only for replies on threads you don't own. No equivalent in [gstack](https://github.com/garrytan/gstack/tree/2d42e4ece834) or [ECC](https://github.com/affaan-m/ECC/tree/76aaa77df6ad). |
| `mre` | A bit like Matt Pocock's [`diagnosing-bugs`](https://github.com/mattpocock/skills/blob/49dd158d1076/skills/engineering/diagnosing-bugs/SKILL.md), but the repro is written for a maintainer to paste, not to find the cause yourself. |
| `calibrated-adversarial-review` | A bit like gstack [`review`](https://github.com/garrytan/gstack/blob/2d42e4ece834/review/SKILL.md) / ECC [`santa-method`](https://github.com/affaan-m/ECC/blob/76aaa77df6ad/skills/santa-method/SKILL.md), but this one shapes the reviewer's prompt rather than running the review. **Credit:** its optional code-smell lens is adapted from Matt Pocock's [`code-review`](https://github.com/mattpocock/skills/blob/49dd158d1076/skills/engineering/code-review/SKILL.md). |
| `empirical-verify` | A bit like pstack [`create-verification-skill`](https://github.com/cursor/plugins/blob/d457567332aa/pstack/skills/create-verification-skill/SKILL.md) / gstack [`qa`](https://github.com/garrytan/gstack/blob/2d42e4ece834/qa/SKILL.md), plus running the recipe on the change and the planted-bug check. |

Counterparts were compared at these commits (links above are pinned to them): [gstack `2d42e4e`](https://github.com/garrytan/gstack/commit/2d42e4ece834a7fa2c16edda87ec136af6db4c26), [pstack `d457567`](https://github.com/cursor/plugins/commit/d457567332aa6989e753ea289f30ea2bfbc80bc0) (the `pstack/` directory of `cursor/plugins`), [ECC `76aaa77`](https://github.com/affaan-m/ECC/commit/76aaa77df6adc5b889e689007bf99b9a4f74af34), [Matt Pocock's skills `49dd158`](https://github.com/mattpocock/skills/commit/49dd158d1076134a641b33efb035946536778336). Packs change, so check theirs before quoting this table.

## Standalone repos

Three of these also live in their own repos: [supercomment](https://github.com/evnchn-agentic/supercomment), [mre-skill](https://github.com/evnchn-agentic/mre-skill), [calibrated-critique](https://github.com/evnchn-agentic/calibrated-critique). More at [agentic.evnchn.io](https://agentic.evnchn.io).

## License

MIT. Third-party notice: see `THIRD-PARTY-NOTICES.md`. The code-smell list in `calibrated-adversarial-review` follows Fowler's *Refactoring*.
