# cm8908-thirdparty — vendored skills

Use this plugin **only** for third-party skills that upstream ships as loose
`SKILL.md` files (not as an installable plugin/marketplace). If upstream *is* a
plugin or marketplace, reference it from the root `.claude-plugin/marketplace.json`
instead (see the repo README, "Referencing" path) — that stays auto-updatable.

Layout for each vendored skill:

```
skills/
└── <skill-name>/
    ├── SKILL.md        # copied verbatim from upstream
    ├── SOURCE.md       # required — provenance (see template below)
    └── <assets…>       # any scripts/resources the skill needs
```

`SOURCE.md` template — so a future you knows what to re-sync and under what license:

```markdown
- upstream: https://github.com/<owner>/<repo>
- path: <path/inside/repo>
- commit: <full 40-char SHA vendored from>
- license: <SPDX id> (copy the upstream LICENSE alongside if required)
- vendored: <YYYY-MM-DD>
- notes: <local modifications, if any>
```

Update = re-copy from upstream at a newer commit and bump the SHA in `SOURCE.md`.

## Vendored here

- `plain-english/` — Orwell/Gowers plain-prose rules + AI-detox audit/rewrite/edit.
  From `b1rdmania/claude-plain-english-skill`. **No upstream license** — personal
  use only, see its `SOURCE.md`.
