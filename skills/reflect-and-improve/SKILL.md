---
name: reflect-and-improve
description: Use after completing significant work, recovering from errors, or at session breakpoints to review actions, synthesize learnings, and write persistent improvements to memory files. Reduces repeated mistakes and user friction over time.
---

# Reflect and Improve

## Overview

A self-improvement loop for Claude Code. After significant work, this skill triggers a structured review of what happened — errors made, permissions granted, user corrections, implicit preferences — and writes confirmed learnings to persistent memory files so future sessions start smarter.

**Goal:** Every session leaves Claude Code slightly better calibrated to the user and codebase.

## When to Invoke

- After completing multi-file edits, project setup, or debugging sessions
- After making an error and recovering from it
- When the user explicitly runs `/reflect-and-adapt`
- At natural session breakpoints (switching tasks, wrapping up)
- After a tool call is denied and you had to adjust approach

## The Reflection Framework

Run these 6 steps in order. Skip any step that has no findings.

---

### 1. Review Checkpoint

Scan recent actions in the current session for signals worth learning from.

**Look for:**
- Tool calls that failed or were retried
- User corrections ("no, I meant...", "not that file", "use X instead")
- Permission denials (user clicked "deny" on a tool call)
- Approaches that were abandoned mid-way
- Tasks that required multiple attempts

**Output format:**
```
## Review Checkpoint — [date]
- [RETRY] Ran tests 3 times before realizing venv wasn't activated
- [CORRECTION] User said "use pnpm, not npm" — package manager preference
- [DENIED] User denied `git push` — wants to review before pushing
- [ABANDONED] Started editing wrong file, had to switch to correct one
```

**Action:** If you find 0 signals, skip to step 6. Don't force findings.

---

### 2. Permission Patterns

Track which operations the user approved or denied to reduce future permission prompts.

**Capture:**
- Operations the user said "yes" to (especially repeated ones)
- Operations the user denied (and why, if stated)
- Implicit permissions (user asked you to do X, so Y is clearly fine too)

**Write to:** `permissions.md` in the project memory directory

**Format:**
```markdown
# Permission Patterns
Last updated: YYYY-MM-DD

## Approved
- git commit with co-author — always approved
- npm install — approved for this project
- file creation in src/ — approved

## Denied
- git push — user wants to review first
- modifying .env — never touch without asking

## Implicit
- if user asks to "set up tests", creating test files is implicitly approved
```

**Rules:**
- Only record permissions observed across 2+ instances OR explicitly stated by user
- Never persist token values, passwords, or secrets
- Update existing entries rather than appending duplicates

---

### 3. Error Catalog

Classify errors you made and record the fix so you don't repeat them.

**Error categories:**
| Category | Example |
|----------|---------|
| Wrong assumption | Assumed Python 3.10 but project uses 3.8 |
| Missing context | Didn't read package.json before running npm commands |
| Tool misuse | Used `cat` instead of Read tool |
| Wrong file | Edited the test file instead of the source file |
| Stale knowledge | Used deprecated API method |
| Ordering error | Ran tests before installing dependencies |

**Write to:** `errors.md` in the project memory directory

**Format:**
```markdown
# Error Catalog
Last updated: YYYY-MM-DD

## [date] Wrong assumption — Python version
- What happened: Used match/case syntax, but project requires Python 3.8
- Root cause: Didn't check pyproject.toml before writing code
- Fix: Always read pyproject.toml or setup.cfg for Python version constraints
- Prevention: Before writing Python code, check version requirements

## [date] Missing context — Package manager
- What happened: Ran `npm install` but project uses pnpm
- Root cause: Didn't check for pnpm-lock.yaml
- Fix: Check for lock files (pnpm-lock.yaml, yarn.lock, package-lock.json) first
- Prevention: Before any install command, check which lock file exists
```

**Rules:**
- Only log errors that are likely to recur (not one-off typos)
- Include the prevention step — that's the real value
- After 30 days with no recurrence, consider archiving old entries

---

### 4. User Preference Signals

Capture implicit preferences the user reveals through their behavior and corrections.

**Signal types:**
- **Coding style**: tabs vs spaces, single vs double quotes, semicolons, naming conventions
- **Communication style**: prefers short answers, wants explanations, likes/dislikes emojis
- **Tool choices**: preferred package manager, test framework, linter, editor
- **Workflow patterns**: commits frequently vs batches, reviews diffs vs trusts agent
- **Architecture preferences**: monorepo vs multi-repo, OOP vs functional, etc.

**Write to:** `preferences.md` in the project memory directory

**Format:**
```markdown
# User Preferences
Last updated: YYYY-MM-DD

## Coding Style
- TypeScript: single quotes, no semicolons, 2-space indent
- Python: black formatter, type hints preferred
- Prefers functional style over classes

## Communication
- Likes concise responses — no fluff
- Doesn't want time estimates
- Prefers bullet points over paragraphs

## Tools
- Package manager: pnpm
- Test framework: vitest (not jest)
- Linter: biome

## Workflow
- Wants to review git diffs before committing
- Prefers small, focused commits over large batches
```

**Rules:**
- Only record preferences confirmed by user behavior (2+ instances) or explicit statement
- Don't record preferences that are already in CLAUDE.md or project config files
- If a preference contradicts project config (e.g., .prettierrc), project config wins

---

### 5. Memory Synthesis

Write confirmed learnings to persistent memory files. This is the actual "getting smarter" step.

**Target directory:** The project-level memory directory
- For project-scoped learnings: `~/.claude/projects/<project>/memory/`
- For global learnings: `~/.claude/memory/` (but be conservative — most things are project-scoped)

**What qualifies as a confirmed learning:**
- Pattern observed 2+ times in this session
- Explicit user instruction ("always do X", "never do Y")
- Error that was fixed and has a clear prevention rule
- Permission pattern that's clearly stable (not a one-off)

**What does NOT qualify:**
- Speculation or single observations
- Anything already in CLAUDE.md or project docs
- Session-specific context (current task, temporary state)
- Sensitive data (tokens, passwords, personal details)

**Process:**
1. Check if a relevant topic file already exists (permissions.md, errors.md, preferences.md)
2. If yes: use Edit tool to update existing entries, add new ones, remove outdated ones
3. If no: create the topic file with clear structure
4. If MEMORY.md needs a link to a new topic file, add it (keep MEMORY.md under 200 lines)

**Anti-patterns to avoid:**
- Append-only logging (leads to bloat — edit and consolidate instead)
- Saving everything (only save what changes future behavior)
- Duplicating project docs (if it's in .eslintrc, don't also put it in preferences.md)

---

### 6. Friction Audit

Identify moments where the user experience could be smoother.

**Look for:**
- User had to repeat themselves (said the same thing twice)
- User had to correct a misunderstanding
- User had to provide context that should have been known
- A task took multiple rounds when it could have been done in one
- User seemed frustrated or impatient

**Output format:**
```
## Friction Audit — [date]
- User corrected package manager twice → saved to preferences.md
- Had to re-read config file I should have cached → added to review checklist
- User provided project structure context that exists in README → read README first next time
```

**Actions:**
- For each friction point, identify if there's a systemic fix (memory update, workflow change)
- If the fix is a memory update, do it now (step 5)
- If the fix requires a workflow change, note it but don't over-engineer

---

## Memory File Hygiene

### Staleness Prevention
- Every entry should have a "last confirmed" or "last updated" date
- During reflection, scan for entries older than 90 days — flag for review
- If a preference has been contradicted by recent behavior, update or remove it

### Size Limits
- Each topic file: aim for under 100 lines
- MEMORY.md: must stay under 200 lines
- If a file grows too large, archive old/low-value entries

### File Organization
```
~/.claude/projects/<project>/memory/
  MEMORY.md          # Index + high-level context (loaded into system prompt)
  permissions.md     # Approved/denied operations
  errors.md          # Error catalog with prevention rules
  preferences.md     # User style and tool preferences
```

## Example: Full Reflection Output

```
=== Reflect and Improve — 2026-02-17 ===

## Review Checkpoint
- [CORRECTION] User said "use vitest not jest" — test framework preference
- [RETRY] Failed to run tests — forgot to install deps first
- [DENIED] git push denied — user reviews before pushing

## Permission Patterns
→ Updated permissions.md: added "git push — review first"

## Error Catalog
→ Updated errors.md: "Always run install before test commands"

## User Preferences
→ Updated preferences.md: "Test framework: vitest (not jest)"

## Memory Synthesis
→ Wrote 3 entries across 3 topic files
→ No MEMORY.md changes needed (topic files already linked)

## Friction Audit
- 1 friction point: had to be told test framework twice
  → Fixed: saved to preferences.md, won't happen again
```

## Important Principles

1. **Less is more.** A few high-quality memory entries beat a bloated log.
2. **Edit, don't append.** Keep files clean by updating existing entries.
3. **Confirm before persisting.** Don't save assumptions — save observed facts.
4. **Privacy first.** Never persist secrets, tokens, or personal data.
5. **Respect existing docs.** Don't duplicate what's in CLAUDE.md or project config.
6. **Date everything.** Every entry needs a timestamp for staleness tracking.
