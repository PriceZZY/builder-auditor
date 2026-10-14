# builder-auditor

A working setup for doing research and engineering with coding agents: **one model builds, an independent model from another vendor audits blind, and the owner makes the decisions.** It comes out of months of daily use on research code, papers and software projects, and most of its rules exist because their absence cost something; the incidents are written down next to the rules.

## The loop

```
owner    ── agrees a checklist: deliverable, done criteria, out of scope ──▶ builder
builder  ── works end to end, then walks the whole path itself ──────────▶ deliverable
owner    ── asks for an audit (when the stakes warrant it) ───────────────▶ auditor
auditor  ── card + evidence only, zero shared context ──▶ VERDICT: SHIP | REWORK | BLOCK
builder  ── verifies each finding, fixes what must be fixed, re-audits ──▶ until no blocking finding
owner    ── decides and ships
```

State lives in files rather than in the conversation: each project has one entry file (status, next step, who has the ball), read at the start of a session and written back at the end.

## What's here

| Path | What |
|---|---|
| [docs/principles.md](docs/principles.md) | The rules: roles, a gate before work, state in files, three questions before handing work off, audit triage, walking the whole path, memory hygiene. |
| [docs/failure-modes.md](docs/failure-modes.md) | What went wrong and what catches it: green is not correct, no signal at all, tooling that lies quietly, measuring the wrong thing, writing the specification, agents and the record. |
| [docs/blind-audit.md](docs/blind-audit.md) | How the auditor is isolated, how a card is written, how verdicts are consumed and triaged. |
| [docs/what-we-removed.md](docs/what-we-removed.md) | Mechanisms we deleted, and why. |
| [templates/](templates/) | An audit card and a project entry file. |
| [skills/](skills/) | Claude Code skills: `kickoff` (with the entry template), `handoff`, `blind-audit` (with the card template). |

## Using it

- Copy the principles you agree with into your agent's rules file (for example `CLAUDE.md` or `AGENTS.md`).
- Create one entry file per project from [templates/project-entry.md](templates/project-entry.md).
- Install a skill by copying the whole `skills/<name>/` folder into your Claude Code skills directory (`~/.claude/skills/` for all projects, or `.claude/skills/` inside one project). Each skill folder carries the template it needs; `templates/` holds the same templates for use without the skills.
- For audits, write cards from [templates/audit-card.md](templates/audit-card.md) and run them through an auditor you can isolate; [docs/blind-audit.md](docs/blind-audit.md) lists what the runner has to guarantee. The runner scripts we use are not published here.

## Caveats

This is one person's setup around two particular tools, Claude Code as the builder and Codex CLI as the auditor, and both change quickly. The examples come from real incidents with identifying details removed; they are anecdotes, not benchmarks. Take the reasoning and adapt the rules.

## License

MIT (see [LICENSE](LICENSE)).
