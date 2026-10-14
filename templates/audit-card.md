# Audit card: blind audit of <one sentence>

You are an independent auditor with zero shared context, and the only auditor: do not spawn sub-agents; do all the work yourself.
Use only the files listed under "Objects". Do not trust any self-report, including "already tested" and anything this card attributes to the requester.
**Your last message is the whole verdict; nothing before it is kept.** Write every finding there in full, with evidence, line numbers and the fix. Do not write "see above".

## Background (facts only, at most five lines)
- What this is, who will read or use it, what happens next (for example: it becomes public and cannot be withdrawn).
- For a delta round: what the previous round found, what was fixed and how. The previous verdict is the previous auditor's call, not yours.

## Objects
1. <path or repository at commit …>
2. <reference material, read-only>
Read only these; read nothing else.

## Threat model
An honest operator who makes mistakes. <Say who will read or run the result after release.>

## What to audit
1. <decidable question>
2. <decidable question>
3. <decidable question>

## Contract numbers (read fresh by the requester; rerun the commands yourself)
- `<command>` = <value>
- `<command>` = <value>

## What to report
Only blocking findings (must be fixed before release: silent errors, wrong external statements, irreversible effects, leaks) and findings that would change the conclusion. Minor issues: one summary line at most, not itemised.

## Output format
One finding per entry: [blocking|major|minor|question] #n title — evidence (file:line) — why — how to fix
The last line must be the last non-empty line of the whole message, exactly one of `VERDICT: SHIP`, `VERDICT: REWORK` or `VERDICT: BLOCK`, with nothing after it.
