---
name: handoff
description: Use when a work session is ending or pausing (the user says they are done for now, a deliverable has just shipped, or the context is about to be compacted).
---

# /handoff — end of a session

## Core

Write state back to the entry file, not to the conversation. The next session reads only files.

## Steps

0. **Find every project this session touched**; each gets its own write-back. If you did not read a project's entry file when the session started, find it now (from the working directory, or from the user's words and your memory index; ask once if it is unclear) and keep its three lines as the baseline. A project with no entry file yet gets one now, with the three lines from step 1.
1. **Entry file, top three lines.** Re-read it first and compare with the baseline. If another session changed it, merge instead of overwriting; if both changed the same line, keep the newer one, add a line "Disputed: …", and tell the owner. Then write:
   ```
   Status (YYYY-MM-DD): one sentence
   Next: starts with a verb, one sentence
   Ball: owner / builder / someone (waiting on what)
   ```
   Conditions use absolute dates ("if no reply by 10/29, use the fallback").
2. **File the outputs.** Handoff notes and reports go to the project's `reports/` (one page: done / not done / pitfalls / next); audit cards and verdicts to its `audits/`; repository records in the repository's own format. Anything worth keeping in scratch space moves now, because scratch is wiped.
3. **Memory.** The project's index line stays a pointer ("essence + entry → path"). Long-term facts that changed go to their topic files. After a deep conversation, offer to record new insights and list concrete candidates.
4. **Repository.** Changes are committed according to the repository's rules, or explicitly reported as uncommitted. No push without the owner's permission in this session.
5. **Reply in three lines**: where it stands, the next step, who has the ball.

## Checklist

- [ ] The entry file was modified today.
- [ ] The three lines contain no reference only this session understands ("see above") and no relative date.
- [ ] Every project touched was written back, not just the last one.
- [ ] Scratch space was checked.

## Common mistakes

| Symptom | Fix |
|---|---|
| The next session cannot find the audit verdict | It stayed in scratch space; step 2 |
| The memory index turns into a status board | Index lines are pointers; state lives in entry files |
| "Waiting on the owner" without saying for what | "Ball" always says what is awaited |
