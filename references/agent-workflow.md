# Agent workflow catalogue

Load for every task. This is the ladder applied to the AI's own work: tokens, tool calls,
context, and agents are budgets too, and they are the ones most often spent carelessly.

## Don't do it

- Do not read a whole file to change one function. Locate with search, read the range,
  edit the range.
- Do not re-derive what the conversation already established. Facts, decisions, and
  measurements from earlier in the session are cached; use them.
- Do not spawn an agent for a lookup you can do in one tool call. Agents cost a fresh
  context and a summary round trip.
- Do not run the whole test suite when a filtered run answers the question, then run the
  whole suite once before finishing.
- Do not narrate what you are about to do at length; do it, then report the result.

## Do it once

- Read the project's instruction file and the relevant reference file once, early, and
  act on it rather than re-checking.
- Write a helper script the first time a multi-step check is needed; run it thereafter.
- Record measurements in the ledger so they are not re-taken.

## Do it later, in bulk

- Batch independent tool calls into one turn: several reads, several searches, several
  independent edits. Round trips are the dominant latency.
- Group related edits to a file into a single edit where the ranges are adjacent.
- Collect questions for the user and ask them once, at the moment the answer is needed,
  not one per turn.

## Better algorithm

- Search before read: `grep`/`rg` with a tight pattern narrows to the lines that matter.
- Read the test for a module before the module; tests state the contract in fewer lines.
- Use the type checker and the test runner as oracles instead of reasoning through
  every call site by hand.
- When the task spans many files, delegate the *search* to an explore agent and keep the
  *decision* in the main context; do not delegate decisions that need the full picture.

## Smaller

- Keep tool outputs small: `head`, `tail`, line ranges, `--quiet`, filtered logs. A
  10,000-line output that you scan for one line has cost 10,000 lines of context.
- Prefer diffs to full-file rewrites in edits; prefer `Edit` to `Write` for existing files.
- Summaries in your own notes, not verbatim pastes.

## Parallel

- Independent agents for independent workstreams, launched in the same turn, each with a
  self-contained brief. Merge their conclusions, not their transcripts.
- Never parallelize agents whose work touches the same files without a plan for merging.

## Tighter

- Verify with the cheapest oracle first: typecheck before tests, a single test before the
  suite, a `curl` before a browser session.
- Stop when the task is done. A closing recap and the ledger; no restatement, no offers.

## Measure

- Count tool calls and estimate tokens per task phase; the ledger should note where the
  budget went when a task ran long.
- Note which reads turned out unnecessary; adjust the next task's approach.
