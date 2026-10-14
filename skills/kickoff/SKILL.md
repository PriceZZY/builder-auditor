---
name: kickoff
description: Use when a session starts or resumes work on a project (the user names a project, says "continue" or "start", or asks what's next for it) and before making the first edit.
---

# /kickoff — start of a session

## Core

State lives in the project's entry file, not in the conversation and not in memory. Read first, then act.

## Skip this for

One-off questions, small text edits, configuration lookups and conversation. Only projects whose state spans sessions have an entry file.

## Steps

1. **Find the project.** From the working directory (keep a mapping from repositories to entry files) or from the user's words and the memory index. If it cannot be mapped, ask once; do not guess.
2. **Read the entry file**: the three lines at the top (Status / Next / Ball) and the body. No entry file: create one from `project-entry.md` in this skill's directory, with "unknown" in the three lines and tell the user.
3. **Read the repository's own discipline files** (CLAUDE.md, AGENTS.md, decision logs, handoff notes) as procedure, and note what they require at the end of a session.
4. **Check the state.** Has the "Next" step already been done (`git log`, file times, what the user just said)? Conditions must use absolute dates; convert relative ones before judging. If the entry and memory disagree, the entry wins; point out the conflict.
5. **Reply in at most five lines**: where it stopped, what this session will do, who has the ball, how you will start. For non-trivial work, stop here and agree the checklist first (deliverable, done criteria, out of scope).

## Don't

- edit before reading;
- use an old status from memory instead of the entry file;
- touch the repository while a long test battery is running in it.

## Common mistakes

| Symptom | Fix |
|---|---|
| "Last time we planned X", but X is already merged | Step 4: check `git log` |
| A decision log written in the wrong format | Step 3: read its header first |
| Opening with "what's the situation?" | Read the entry first; ask only what it cannot answer |
