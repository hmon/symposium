# symposium

A Claude Code skill that puts one decision to a panel of models from different vendors and merges their answers into one verdict with a vote tally. The panel can mix Claude subagents, Codex CLI, Cursor CLI and other CLIs.

Each seat does its own read-only research and writes a position. In the next round, each seat reads the others' positions and concedes or rebuts. The conductor (your Claude Code session) checks the claims that can be verified, then writes a verdict table with a tally and the dissent for each item. Nothing runs until you approve the lineup, and nothing changes until you approve the verdict.

## Install

As a plugin:

```
/plugin marketplace add hmon/symposium
/plugin install symposium@symposium
```

Or copy just the skill:

```
git clone https://github.com/hmon/symposium
cp -r symposium/skills/symposium ~/.claude/skills/
```

## Use

```
/symposium what model routing minimizes tokens for our workflow?
/symposium should we adopt X or Y? seats: codex:gpt-5.5 cursor:grok-4 claude:opus
```

## Requirements

- Claude Code. Claude subagents are always available.
- Optional: the `codex` CLI and the `cursor-agent` CLI, for seats from other vendors.

## Flow

frame → lineup gate → round 1 positions → settle facts → round 2 rebuttals → verdict → action gate

All round files are written to a scratch folder: `brief.md`, `r1-*.md`, `facts.md`, `r2-*.md` and `verdict.md`.

## License

MIT
