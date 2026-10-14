# Failure modes

Things that went wrong while doing real research and engineering work with coding agents, grouped by the kind of signal that misled us. Every entry is a pattern that happened more than once. Each one names the symptom, why it happens, and what catches it.

The common thread: **a signal that looks fine is evidence, not a conclusion.**

---

## 1. Green is not correct

The tests and checks pass, and the code is still wrong.

| # | Shape | What catches it |
|---|---|---|
| 1.1 | **Code behind an environment gate never runs.** A whole function sits after `if (!providerConfigured()) return`; locally the key is never set, so a broken permission inside it never fails a test. | After changing permissions, schemas or signatures, list every code path behind an environment switch and exercise each one at least once, in its real order and as its real caller. |
| 1.2 | **The local database is not the one the migrations describe.** A migration changed, nobody reset the local database, and the tests ran against the old permissions. | Trust only the full gate that rebuilds the database first; a green single-file run proves nothing. |
| 1.3 | **Written but never run.** A reference client was delivered without being executed once; the first real run showed two defects immediately. | Anything meant for someone else to use gets run end to end by its author before delivery. |
| 1.4 | **Parallel processes contaminate each other.** While a long test battery was running, a read-only listing command reported "0 tests in 0 files", and a grep read a stale file. No error, just a wrong answer. | Do not even take read-only measurements while a battery that rewrites the workspace is running. |
| 1.5 | **A test that is green but not looking.** Three forms: a weak pin that only asserts "no X" (stays green when the feature is completely broken); an assertion that is always true by construction; a synchronization point that is already true before the action finishes, so later assertions run against the old state. | **Every new test must go red first**: put the bug back and watch it fail. Wait for something that can only be true in the target state, preferably asked of the server, not the UI. |
| 1.6 | **Your build is not their build.** A build that reruns until its output stabilises can look right on your machine while a pipeline that runs a fixed number of steps produces something different from the same source. | When someone else builds from your source (a publisher, CI, a customer), check *their* artifact, not yours. Pin values that depend on multiple passes at the source. |
| 1.7 | **Production has a wider default than local.** A hosted database can grant new objects to public roles by default where a local one does not, so every local security test passes while production grants more than the tests assume. | Read security baselines (default privileges, grants, function execution rights) in production, and diff production against local after every migration. |

## 2. No signal at all

Everything was audited, and the path had still never been walked.

- **Auditing the parts to perfection does not make the path work.** A pipeline whose parts had all passed their tests and repeated audits failed again and again when it was finally run end to end. Not one failure was a defect in a part; all were **seams**: a missing input file, a missing dependency, two sections of a plan contradicting each other, one parameter serving two purposes, an unactivated virtualenv, a hard-coded identifier, a path comparison without `realpath`.
- **After an audit passes, the first thing to do is run the path end to end**, even with fake parameters and tiny data. Where it dies tells you more, faster, than any amount of reasoning: an offline agent spent half an hour concluding the inputs were insufficient; running it took thirty seconds and surfaced the real first error (a missing package).
- **"Nobody has run this" is a risk to register**, at the same level as "this has a bug".
- **Half-upgraded schemas.** One task upgraded field names on the producer side while the consumer kept the old vocabulary, at one point with a flat contradiction (one side forbade a field the other side required). When a schema or field name changes, list every producer and every consumer and tick each one off.
- **Green evidence older than the code.** A gate started at 23:05 cannot vouch for a file modified at 23:59. Before citing a passing run, check that it started after every tested file was last modified, and re-run the *whole* gate after changing a script it calls: modules that pass alone do not imply the suite passes.
- **Seams need an owner.** When two adjacent tasks are handed out, one of them must explicitly own the boundary between them.

## 3. Tooling that lies quietly

Scripts we wrote ourselves finished without an error and reported a normal-looking, wrong result. The outer layer never exits non-zero, so **"it finished without errors" carries no information.** A selection:

| Shape | Note |
|---|---|
| Counting log strings instead of reading the exit code | the program under test prints the same string, so `exit 3` passes |
| `setsid cmd &` and then `$!` | not necessarily the program's PID, so `wait` and `kill -0` judge the wrong process |
| Pipelines swallow exit codes | `cmd \| sed` returns sed's status; in zsh read `pipestatus`, lower-case, 1-based |
| `cmd \|\| echo 0` | turns "this command does not exist on macOS" into "the count is zero" |
| zsh does not word-split unquoted variables | a whole list becomes one argument; "nothing found" looks like "nothing changed" |
| A variable followed by full-width punctuation | the punctuation is absorbed into the variable name, giving an empty string; write `${var}` |
| Killing the parent, not the process group | orphans keep writing to the same directory and the same GPUs |
| A test that takes its expected values from the code under test | wrong values stay green; write expected values independently and run a mutation check |
| Blacklisting temporary stubs in a sync script | it replaced real code with placeholders and reported "read-back identical" for each file; whitelist instead |
| Canonical JSON with sorted keys | silently destroys meaningful insertion order; make every ordering explicit |
| Editing a script while bash is executing it | bash reads as it runs and resumes at a shifted byte offset |

General rule: **gates read direct signals only** (exit codes, per-file hashes, process tables enumerated by exact path), never "the output contains X", because the thing under test can produce that text itself. And every new gate needs a negative control: make it fire once on purpose.

## 4. Measuring the wrong thing

"Measure before you speak" stops guessing. It does not stop measuring the wrong object.

- **Local mistaken for production.** A query meant for production used a local environment file pointing at a local database. Two different "explanations" were built on that reading, and a migration was written for a problem that did not exist.
- **`pgrep -f <name>` matching your own wait loop.** The loop waited on itself forever, and the elapsed time it measured was its own.
- **Your own echoed history read as a new result**, which nearly led to fixing something that was not broken. Mark new output before reading it.
- **When two readings disagree, first suspect that they are not reading the same thing**, then suspect the mechanism.
- **The instrument dropped a dimension.** A contrast check ignored alpha and reported exactly the opaque value. If a result equals a simpler quantity, suspect the instrument.
- **Two different things that look identical.** A GPU partition and a full GPU produced the same `nvidia-smi` line. Choose criteria on a dimension where the two are actually different.
- **One deduplicated error message is not one defect.** Dozens of failures collapsed to a single message; the real conclusion was "this is the first wall", not "there is one bug". Take the first wall down temporarily and look at the next layer. A check that passes covers only the segment it checks; to claim the chain works, run the chain.
- **The owner's own account is the current reading.** When the owner is working in parallel elsewhere, the builder's records lag behind reality. "I have no record of it" is not evidence against what the owner says.

## 5. Writing the specification

The builder and the acceptance checker both read the card literally, so wording errors get amplified on both sides.

- **"Every X must have Y" makes the builder manufacture X.** A rule that every reference marker must be used somewhere led the builder to add new content just to satisfy it. Write the intent ("achieve this"), give exemptions, and add "do not change the body to satisfy this rule". Count things yourself before writing a count.
- **The same criterion must be identical, units included, on the build card and the acceptance card.** One card set a length in one unit and the other counted in a different unit; the product passed one and failed the other. Length limits are recorded, not enforced.
- **Tolerances match what is compared.** A 1e-6 tolerance applied to a table printed to four decimals produced a false failure.
- **Scan scopes match authorization scopes.** A card demanded "zero dangling references in the whole directory" while forbidding edits to three files that contained some; the builder correctly reported itself blocked.
- **An audit without a threat model never converges.** A maximum-effort auditor with no stated adversary assumes a malicious insider with full write access, and that adversary cannot be defended against. Round after round, a verification tool kept growing, every finding was fixed, and every round found new holes, while the honest path had never been run once. Write the threat model into the card (default: an honest operator who makes mistakes) along with the pass criteria.
- **When two consecutive rounds find new blocking issues in code that was just added**, stop patching. Retreat to the last audited baseline and make the smallest change. A mechanism must be small enough for the owner to explain; one the owner cannot explain does not belong in the project.
- **Defense needs proportion.** Locks against "choosing the method after seeing the data" were added round after round until the rule became "don't look at the data", which is extreme for observational research. Ask three questions of every proposed lock: what does it defend against, will that actually happen, and what does the lock cost?
- **Agents implement every clause at its strictest.** Tell them: add no defensive mechanism unless you can name the specific scenario in which it prevents a silent wrong result; tie each deliverable to the line of the plan it serves; when a simple way satisfies the rule, use it.

## 6. Agents and the record

- **A builder agent may write its own verdict.** We have seen builder agents produce acceptance reports and verdict files that said "SHIP". Accept only the audit runner's exit code and its single end line; quarantine any file named like an audit inside a delivered tree.
- **The record must not be stronger than the evidence.** Comments, commit messages, logs and reports are all claims. A wrong attribution in a log is worse than no record: it sends the next person in the wrong direction. When a real diagnosis arrives, the first job is to correct what was written before.
- **Writes are verified by reading back.** A failed `assert` in an anchor replacement stopped nothing because the next command was on a new line rather than chained with `&&`. Do the write in its own step, read back the changed region plus one line on each side, then grep once more independently.
- **A conversation summary is not evidence.** After context compaction, the summary recorded "told the owner about the plan" when only a one-line progress note had been sent. Before citing "the owner knows" or "the owner agreed", find the actual message in the transcript. Long messages written mid-turn may not survive in the transcript, so anything the owner must see goes in the last message of the turn.
- **No time estimate for a chain that has never run end to end.** When a step the owner is waiting on slips, or a checklist becomes obsolete, say so in the conversation at once; writing it into a file is not telling them.
- **A continued sub-agent may run on a different model.** In our harness, resuming a finished sub-agent starts a fresh instance on the main session's model; start a new agent with the model stated explicitly instead.
- **An agent that says "I can't run this step"** has often found exactly where the real bug is. The delegating side must run that step itself.
