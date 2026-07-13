# cm8908-custom — skills

Drop each standalone custom skill in its own folder here:

```
skills/
└── <skill-name>/
    ├── SKILL.md        # required — name + description frontmatter
    └── <scripts…>      # optional: run.sh, resolve.sh, assets, etc.
```

Claude Code **auto-discovers** every `skills/*/SKILL.md` in this plugin. You do
**not** edit `plugin.json` or the marketplace manifest when adding a skill —
just add the folder, commit, push, and run `/plugin marketplace update my-skills`.

`SKILL.md` frontmatter must have `name` and `description`. Keep the description
trigger-rich (the phrases that should invoke it) — that text is all Claude sees
when deciding whether to load the skill.

> This README is a placeholder so the empty `skills/` directory is tracked by
> git. Delete it once you add real skills, or leave it as a reminder.
