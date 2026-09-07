# apollo-optimize

A skill that makes any AI coding agent treat optimization as the default way of working,
not a separate pass. The standard is the Apollo Guidance Computer: 2 KB of erasable
memory, 36 KB of program rope, and a crew to bring home. Every change is budgeted before
it is written, measured against a baseline, pushed down a seven-rung ladder from
"don't do it" to micro tightening, verified, and closed with a ledger of what is left.

## Layout

```
SKILL.md                  the discipline: loop, ladder, guardrails, ledger format
references/budgets.md     how to set resource ceilings, with realistic numbers
references/frontend.md    browser UI, bundles, rendering, assets
references/backend.md     servers, edge runtimes, APIs, caching, cold starts
references/database.md    queries, schema, indexes, writes, migrations
references/compute.md     algorithms, data structures, hot loops, memory
references/toolchain.md   build, CI, tests, dev loop, dependencies
references/agent-workflow.md  the agent's own tokens, tool calls, context, subagents
references/agc-lessons.md the Apollo lessons that set the standard
```

`SKILL.md` is always loaded when the skill triggers; the agent reads only the reference
files the task touches.

## Install

**Claude Code, one project:**

```bash
git clone https://github.com/pycarrot/apollo-optimize .claude/skills/apollo-optimize
```

**Claude Code, every project:**

```bash
git clone https://github.com/pycarrot/apollo-optimize ~/.claude/skills/apollo-optimize
```

**Other agents:** the files are plain Markdown with a YAML front matter. Point the agent's
instruction loader (`AGENTS.md`, a system prompt, a rules directory) at `SKILL.md` and
keep `references/` beside it.

## Contributing

Every technique in `references/` should say what it costs without the trick and what
measurement justifies it. Add a rung label (1–7) and keep each file under ~100 lines;
if a domain grows past that, split it and add a row to the table in `SKILL.md`.
