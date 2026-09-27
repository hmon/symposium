---
name: symposium
description: Put one decision question to a panel of models from different vendors (Claude subagents, Codex CLI, Cursor CLI, others). Each seat researches on its own, then concedes or rebuts against the others. The result is one verdict with a vote tally. Use for architecture choices, model and tool routing, plan reviews, and risk calls, when the user asks for a "symposium", a "panel", or several models to debate something.
---

# Symposium

Run a panel of models to reach one verdict on one question. You are the **conductor**: you write the brief, dispatch the seats, check facts and merge the result. You never vote.

Arguments: the question, plus an optional seat list (e.g. `codex:gpt-5.5 cursor:grok-4 claude:opus`).

## Rules

- **The user approves the lineup before any model call.** That includes probe calls that check a model id. Rejecting the lineup means zero calls.
- **Seats are read-only.** Use the harness's read-only flag. A seat that writes files, or makes model calls of its own, is invalid; report it as such.
- **Positions live in files.** Read the files, not a chat recap.
- **A failed seat is reported as absent.** Never replace it silently.
- **Show the tally for every item.** A 2–1 split is never presented as unanimous, and each dissent keeps one line.
- **No action without a second approval.** The verdict proposes exact changes. Apply them only after the user says yes, in one commit.
- **Cap the rounds.** Round 1 is positions and round 2 is rebuttals. Run round 3 only for a split that still matters. Never run more than 3.

## Flow

### 1. Frame

Create `<scratch>/symposium/<slug>/`, using the session scratchpad when the system prompt lists one. Write `brief.md` with:
- the question
- the goal and constraints
- the facts already established
- the files and sources the seats must read
- the deliverable: numbered sections, a word cap (default 700), and a required one-line verdict

Add "READ-ONLY: never modify files, never call other models."

### 2. Lineup gate

Detect harnesses with `command -v codex cursor-agent` and similar checks. Claude subagents are always available. The default is one strong model per available vendor, 3 seats. Propose each seat's model, harness and effort, the number of rounds and a rough token cost. **Wait for a yes.**

### 3. Round 1: positions

Run all seats in parallel, each on the same brief, with output going to `r1-<seat>.md`. See the adapter table below. Run long CLI calls in the background, since they can take over 10 minutes.

### 4. Settle facts

Pull out the claims that can be checked: model availability, versions, receipts, file contents. Verify them locally with cheap reads. Write `facts.md`, keeping the settled facts separate from the open disputes.

### 5. Round 2: rebuttal

Write `r2-prompt.md`. It tells each seat to read all the `r1-*.md` files and `facts.md`. It lists each dispute by name. For every dispute the seat concedes or rebuts in one or two sentences, with evidence. The seat then states its final answer or config and a verdict line. The word cap is half of round 1 or less. Output goes to `r2-<seat>.md`.

### 6. Verdict

Write `verdict.md` and show it to the user:
- a table of item → decision → vote tally → dissent
- where the savings or gains come from
- the risks, meaning where the verdict is weakest
- the exact changes proposed, as files and values

Report the tokens each seat used, when the harness exposes them.

### 7. Action gate

Ask the user. Apply the changes only after a yes.

## Seat adapters

| Harness | Read-only invocation | Model / effort |
|---|---|---|
| Claude subagent | Agent tool, `general-purpose`, the prompt says read-only | `model` param |
| Codex CLI | `codex exec -s read-only --skip-git-repo-check -o <out> "<prompt>" </dev/null` | `-m <id>`, `-c model_reasoning_effort=<level>` |
| Cursor CLI | `cursor-agent -p --mode ask --model <id> "<prompt>" > <out>` | effort is part of the model id |
| Other CLI | its non-interactive mode, its read-only flag, stdout to a file | per CLI |

For a CLI seat, pass the folder path inside the prompt so it can read the round files.

## Failure handling

- **A seat returns nothing, or errors:** retry once. If it fails again, mark it absent in the verdict.
- **A seat seems stuck:** check its process and output file directly. Don't wait on a job that isn't running.
- **The account rejects a model id:** go back to the lineup gate with alternatives.
