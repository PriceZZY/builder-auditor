# What we removed

The setup improved as much by deleting mechanisms as by adding them. Each entry says what it was, why it went, and what replaced it.

1. **A standing "lead" agent loop.** A long-lived lead agent dispatched work, held locks and was checked by a heartbeat every few rounds. It grew layers (lock files, lists of legacy locks, conventions for sharing between sessions) faster than it produced results, and each round of hardening found new holes in the code it had just added. *Replaced by* a checklist agreed with the owner before work (direction) and a budget (proportion).

2. **Double audits.** Two auditors cross-checking the same work cost twice as much and mostly disagreed about minor points. *Replaced by* one builder and one auditor.

3. **Home-grown dispatch scripts.** We wrote our own scripts to hand construction tasks to another agent's CLI, with sandboxes, worker pools and patch merging. *Replaced by* the vendor's official plugin. Use what the tools already provide before building your own.

4. **A separate acceptance layer.** An independent model re-checked every delivered task. Whenever the acceptance card's wording drifted from the build card's, it rejected correct work (see [failure modes §5](failure-modes.md#5-writing-the-specification)). *Replaced by* the builder verifying delegated work with one command of its own, while the auditor stays separate and is used when the owner asks for it.

5. **Hardening a tool round after round.** A verification tool kept growing over several audit rounds without converging. *Replaced by* a rule: when two consecutive rounds find new blocking issues in code that was just added, retreat to the last audited baseline and make the smallest change.

6. **"Don't look at the data" locks in observational research.** Locks against choosing the method after seeing the data were tightened round after round until nothing could be looked at before the method was frozen. *Replaced by* ordinary practice: look at the data, write down the preferred method, log every later change together with the feature of the data that motivated it, and report all plausible variants side by side.

7. **Heuristic memory compression.** Compressing the memory index by moving out "the longest lines" removed load-bearing rules three times in one day. *Replaced by* moving out only event narrative, then having a second agent compare the before and after versions line by line.

8. **Fixing every audit finding.** Treating every finding as must-fix produced rounds of defensive code that nobody needed. *Replaced by* triage: only silent errors, wrong external statements and irreversible actions must be fixed ([principles §7](principles.md#7-triage-audit-findings-with-three-questions)).
