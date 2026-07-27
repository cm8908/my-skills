- upstream: https://github.com/b1rdmania/claude-plain-english-skill
- path: SKILL.md + REFERENCE.md (repo root)
- commit: 92090976c7d3ef8f7e655155b8c53520c1ac38f0
- license: **NONE declared upstream** — the repo ships no LICENSE file and the
  README's "License" section states no SPDX id. Vendored here for personal use
  only. Do not redistribute beyond this repo, and drop it if the author ever
  declares an incompatible license.
- vendored: 2026-07-27
- notes: Copied verbatim, no local edits. Upstream is a loose two-file skill (no
  `.claude-plugin/`), so vendoring is the only path — it cannot be referenced from
  the marketplace manifest.
  Kept here rather than in `cm8908-custom` because it is an operational
  audit/rewrite/edit tool, not a behavioral guideline skill.
  Overlaps `humanizer` and `avoid-ai-writing` (both referenced from the root
  marketplace manifest) on the AI-detox half. Its distinct value is the
  **Orwell/Gowers classical prose rules** — passive voice, abstract-noun subjects,
  Latinate padding, verbal false limbs, dying metaphors — which neither of the
  larger skills covers. Prefer this one when the goal is tighter prose rather than
  scrubbing AI tells.
  Update = re-copy both files at a newer commit and bump the SHA above.
