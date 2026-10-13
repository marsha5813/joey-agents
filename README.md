# joey-agents

A two-agent engineering pair for [Claude Code](https://claude.com/claude-code):

| Agent | Model | Role |
|---|---|---|
| `coder` | Opus | Lead engineer. Plans, builds, commits locally, and calls the reviewer at milestones. |
| `code-reviewer` | Sonnet | Read-only reviewer. Returns Blockers / Concerns / Nits with `file:line` citations. Never edits, commits, or approves. |

The coder asks for review at three milestones: **design** (before substantial code), **first coherent unit**, and **final pass**. It defaults to plan mode for architectural work, and reviews the design memo before presenting the plan, so you see a revised plan rather than a plan plus amendments.

## Install

```
claude plugin marketplace add marsha5813/joey-agents
claude plugin install joey-agents@joey-agents
```

Then start it from Agent View or with `claude --agent joey-agents:coder`. The reviewer can also be used on its own for a one-off review of a diff, PR, or files.

## Using the coder from an orchestrating agent

Subagents can't spawn subagents, so when another agent dispatches `coder`, it can't call the reviewer itself. Instead it emits a marker at each milestone and stops:

```
<review-requested>
milestone: design | first-coherent-unit | final-pass
path: <absolute path to review>
focus: <what to flag>
conventions: <CLAUDE.md or doc paths>
known-rough: <skip list>
</review-requested>
```

The orchestrator should then:

1. Check that all five fields are present and `milestone` is valid. If not, resume the coder asking it to re-emit.
2. Dispatch `joey-agents:code-reviewer` with `path`, `focus`, `conventions`, and `known-rough` passed through verbatim.
3. Resume the coder with `SendMessage`, pasting the findings unfiltered. The coder triages them and continues to the next milestone.

Don't run the reviewer without a marker, and don't skip one the coder emits.

## Project context

Neither agent carries project-specific knowledge. They read the `CLAUDE.md` in or above the working directory, so put conventions, secret-handling rules, and test commands there.

## License

MIT
