# CLAUDE-Skills

My custom Claude Code skills, packaged as a plugin marketplace so any machine can install them straight from GitHub.

## Install on a new machine

In a `claude` terminal session:

```
/plugin marketplace add kazoooich/CLAUDE-Skills
/plugin install my-skills@kazoooich-skills
```

Skills show up namespaced, e.g. `/my-skills:handover`.

The repo is public, so no GitHub sign-in is needed to install it.

## Get updates

```
/plugin marketplace update kazoooich-skills
```

## Add or change a skill

1. Add a folder under `skills/<skill-name>/` containing a `SKILL.md` (with `name` and `description` frontmatter).
2. Bump `version` in `.claude-plugin/plugin.json`.
3. Commit and push, then run the update command on other machines.

## Skills

| Skill | What it does |
|---|---|
| `handover` | Start-of-session catch-up and end-of-session HANDOVER.md notes for any repo. |
