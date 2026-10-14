# Blind audit

How the auditor is set up, how an audit card is written, and how verdicts are consumed. The template is in [templates/audit-card.md](../templates/audit-card.md); the procedure as a Claude Code skill is in [skills/blind-audit](../skills/blind-audit/SKILL.md).

## Roles

- **One builder, one auditor.** The auditor is a model from a different vendor, reached through its own CLI (in our case Codex CLI at maximum reasoning effort). It gets the audit card and the evidence, never the builder's conversation.
- **No double audits.** One auditor per round. Cross-checking the same work with a second auditor doubled the cost and mostly produced disagreements about minor points.
- **The auditor's statements are claims too.** Before a fact from a verdict goes into any record, the builder re-runs the number or reads the line itself. Auditors read code and can be wrong, but a green test from the builder does not refute them either; when they conflict, write a probe that measures the thing directly.

## The card

A good card is short and decidable:

1. **Preamble.** You are the only auditor, so do not spawn sub-agents. Use only the listed evidence and trust no self-report, including "already tested". Your last message is the whole verdict: write every finding there in full.
2. **Background.** At most five lines, facts only.
3. **Objects.** Exact paths, and an instruction to read nothing else.
4. **What to audit.** Three to six questions the auditor can actually decide.
5. **Threat model.** By default, *an honest operator who makes mistakes*. Without one, a maximum-effort auditor assumes a malicious insider with full write access, and the audit never converges (see [failure modes §5](failure-modes.md#5-writing-the-specification)).
6. **Contract numbers.** Counts, hashes and commit IDs, read fresh from the repository, each with the command that produced it. Hand-written numbers that have gone stale are the most common card error.
7. **What to report.** Only blocking findings and findings that would change the conclusion; minor ones get one summary line at most.
8. **A strict last line**: `VERDICT: SHIP`, `VERDICT: REWORK` or `VERDICT: BLOCK`, with nothing after it.

**Delta rounds (R2 onward)** state what the previous round found, what was fixed and how, and what does not need re-auditing. Previous findings are presented as "the previous auditor's call, not yours", so the new round checks them independently instead of rubber-stamping them.

**Wording pitfall:** offensive-security vocabulary in a card ("exploit", "bypass") can trip the vendor's safety filter and kill the run before it starts. Phrase the same questions as "missed detection", "misclassification", "integrity check".

## Isolation: what the runner must guarantee

The auditor must not inherit the builder's context or the owner's personal setup:

- an environment allowlist for the model process;
- a separate home directory, so personal skills, memories and rules are not loaded;
- user configuration and rules ignored; the audited repository's own agent instruction files not loaded; project-level skills disabled; memories off; web search off;
- a read-only sandbox without network access;
- for git repositories, a detached worktree at an exact commit, so only committed content is audited.

State the residual risk plainly: a read-only sandbox blocks writes and network, not reads. Unless the auditor runs in a container, the boundary on what it reads rests on the card's instructions.

## The exit protocol: consuming a run

Silent misreadings of audit output are expensive, so the runner's contract is strict:

- **Exit 0 means a valid run that produced a verdict, not "SHIP".** Read the verdict.
- Exit 3 means the audit did not run (the reason is in a marker line); exit 2 means bad arguments.
- The output file opens with a *pending* header. A run counts only if the exit code is 0 **and** the file ends with exactly one END line carrying `result=valid verdict=<X>`. Old content at the same output path is never this round's result.
- **Wait on the entry process, not on the output file.** At maximum effort the model can reason for many minutes without emitting an event; about 20 minutes without new events is the point to *consider* interrupting, not proof that the run is dead.
- Interrupt through the entry process, so it can clean up its children, then confirm that every recorded child process is gone before re-running.

## After the verdict

1. **Verify each finding yourself**: re-run the numbers, read the cited lines, reproduce the counterexample.
2. **Triage** with the three classes and three questions from [principles §7](principles.md#7-triage-audit-findings-with-three-questions): fix silent errors, wrong external statements and irreversible actions; log the rest in one line each.
3. **Defects the code already had still count.** An audit before publishing often finds problems that predate the change under review; publishing them is still publishing them.
4. **Pin each fix with a test that goes red on the old code first**, and mutation-test the new tests: revert one fix at a time and confirm a test catches it.
5. **Re-audit as a delta round**, and stop when no blocking finding remains.
