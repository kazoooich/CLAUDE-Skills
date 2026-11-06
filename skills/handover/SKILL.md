---
name: handover
description: Start-of-session catch-up and end-of-session handoff notes for any project, via /handover. Use when the user runs /handover, or asks to "read HANDOVER.md", "catch me up", "what's the state of this project", "start the session" — OR at the end of work asks to "wrap up", "write a handover", "update HANDOVER.md", "end the session". Reads project docs (HANDOVER.md, README.md, PROJECT.md, docs/) to summarize state, highlight pending tasks, and flag setup friction at session start. Writes an updated HANDOVER.md with a dated session summary, refreshed next-steps, and any new friction notes at session end, then commits — and pushes, if the repo has a remote — the doc changes.
---

One command, two modes, works in any repo. It has no app to drive —
the "harness" here is git plus the project's own docs, read and
written fresh each time rather than assumed from one project's layout.

## Mode detection

- Explicit arg `start` / `begin` → START.
- Explicit arg `end` / `finish` / `wrap` / `done` → END.
- No arg: if this is early in the conversation and no substantive work
  has happened yet this session, treat it as START. If real work has
  already happened in this conversation, treat it as END. If genuinely
  ambiguous, ask rather than guessing.

## Locating the docs

Find the repo root first (nearest ancestor with `.git`; fall back to
cwd if none). Then look, in order, for the handover doc:

1. `HANDOVER.md` at repo root
2. `docs/HANDOVER.md`, `Docs/HANDOVER.md`
3. `Claude Docs/HANDOVER.md`, `claude-docs/HANDOVER.md`, `.claude/docs/HANDOVER.md`
4. Fallback: case-insensitive recursive search, depth ≤ 3, excluding
   `node_modules`, `.git`, `dist`, `build`, for a file named `handover.md`

Also note the location of `README.md` and `PROJECT.md` (or equivalent
top-level overview docs) if present — same search order. Record
whatever paths you find; reuse the same paths in END so both modes
target the same files.

**First time in a project (nothing found):** say so plainly. In START,
don't invent a summary from nothing — say there's no existing handover
doc and offer to create one. In END, create `HANDOVER.md` at repo root
unless the user says otherwise.

## START procedure

1. Confirm git state: `git rev-parse --is-inside-work-tree`,
   `git remote -v`, `git status`, `git log -5 --oneline`.
2. Read `HANDOVER.md` in full — it's the authoritative handoff doc.
   Read `README.md` and `PROJECT.md` if present for broader context.
   Only skim other root docs (DEPLOY.md, CONTRIBUTING.md, CHANGELOG.md)
   if HANDOVER.md references them or clearly leaves a gap.
3. Give the user a short summary: what the project is, what state
   it's in, what the last session did. A few sentences, not a re-read
   of the whole doc back at them.
4. Pull out pending work and present it as a short list of candidate
   tasks to pick up. This isn't always a clean "Next steps" section —
   in practice it's often scattered: items marked "not yet fixed" /
   "still not fixed" / "TODO" / "revisit if..." inline through the
   doc (e.g. a code-review findings list with some items unresolved).
   Scan for those too, not just a dedicated header. If there's
   genuinely nothing pending, say so — don't invent work.
5. Run the friction checklist below and report anything real you find,
   each with a one-line suggested remedy. Don't silently fix things,
   just surface them.
6. Ask which task to start on, or wait for direction. Don't start
   coding unprompted.

### Friction checklist

Only report what you actually verified — an invented friction point is
worse than none.

- Dependencies installed? (`node_modules` present if there's a
  `package.json`, equivalent check for other stacks)
- Any setup step the docs describe that hasn't actually been done
  (missing config/data files, a migration that was never run, env vars
  referenced but no `.env`)
- Any script/tool the docs mention that isn't actually there
  (package.json script, Makefile target, CLI dependency)
- Git: leftover uncommitted changes from a previous session, unpushed
  commits, or push failing (auth/remote misconfigured) — surface these,
  don't carry them silently into new work
- Anything HANDOVER.md already flagged as unresolved friction — carry
  it forward instead of rediscovering it from scratch

## END procedure

1. Reconstruct what actually happened this session: review the
   conversation, and ground it in `git status` / `git diff` / `git log`
   since the commit that was HEAD at session start (or since
   HANDOVER.md's last recorded session, if that's what's available).
   The summary must match real file changes, not just narrative memory.
2. Open the HANDOVER.md located earlier (or create one at repo root if
   none existed). Follow its existing structure and conventions —
   don't impose a new format on a doc that already has one. Update:
   - A dated session-summary entry — write it for a fresh Claude
     instance with zero memory of this conversation: what changed,
     why, and anything a human teammate would already know but an
     agent wouldn't.
   - The next-steps / pending-tasks section, reflecting what's
     actually left, not what was left before this session.
   - Any new friction or gotchas hit this session (setup surprises,
     tool gaps, deploy quirks) — this is exactly the detail that saves
     the next agent from repeating the same investigation.
3. Update other docs (README, PROJECT.md, etc.) only if this session's
   work actually changed something they describe. Don't touch docs the
   session didn't affect.
4. If this is a git repo:
   - Stage only the doc files just modified (HANDOVER.md + anything
     touched in step 3). If unrelated uncommitted code changes exist,
     flag them to the user rather than sweeping them into this commit.
   - Commit with a message describing the session's actual work, not
     just "update HANDOVER.md". Follow normal commit hygiene (HEREDOC
     message, no `--no-verify`).
   - Push. If it fails (no remote, auth, diverged), report the exact
     error — don't retry blindly or force-push.
   - Not a git repo: just confirm the file was saved, no commit step.
5. Close with a short summary: what was recorded, whether it was
   committed/pushed, and anything that needs the user's attention
   before the next session.

## Notes

- Generic by design — discovers docs and conventions at run time
  rather than assuming any one project's layout.
- Never fabricate a next-steps item or friction point that wasn't
  actually verified.
- Keep START tight (a few sentences + a task list). Spend the effort
  on END — that's the doc a fresh agent will actually depend on.
