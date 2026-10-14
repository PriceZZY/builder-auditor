---
name: blind-audit
description: Use when a deliverable needs a zero-shared-context audit before it reaches the owner or leaves the house (code, public text, a report, a release), when the owner asks for an audit, or when a previous round returned blocking findings and the fix needs a delta re-audit.
---

# /blind-audit

One builder, one auditor: a model from a different vendor, isolated from the builder's context. The owner decides whether to audit at all. The reasoning behind each step is in `docs/blind-audit.md` of the builder-auditor repository.

## Steps

1. **Write the card** from `audit-card.md` in this skill's directory into the project's `audits/` as `YYYY-MM-DD-Rn-<topic>-card.md`. Read every contract number fresh, with its command. State the threat model. Ask only for blocking findings.
2. **Run it** through your isolated runner in the background, and record the entry process ID. Wait on that process (one notification), not on the output file.
3. **Consume the result.** Exit 0 and exactly one END line is this round's verdict; anything else means the audit did not happen. Save the verdict next to the card as `…-verdict.md`.
4. **Verify each finding yourself**, then triage: fix silent errors, wrong external statements and irreversible effects; log everything else in one line each.
5. **Fix with tests that go red on the old code first**, and mutation-test the new tests.
6. **Run a delta round** (R(n+1)): say what was found, what was fixed and how, and what does not need re-auditing. Stop when no blocking finding remains.

## Don't

- run a second auditor on the same work;
- read exit code 0 as "SHIP";
- copy an auditor's factual claim into a record without checking it;
- use offensive-security wording in a card (it can trip the vendor's safety filter).
