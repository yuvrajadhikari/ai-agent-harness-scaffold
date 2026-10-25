# AI Agent Harness Scaffold

A reusable skill / playbook for designing and scaffolding an **agent harness**: the loop that turns an LLM’s text into real actions.

**Agent = LLM + harness.** The chat UI is not the harness.

Based on [Theo’s “What is a harness?”](https://x.com/theo/status/2113424203345265053) framing, plus operational practices for world-class agents (evals, traces, budgets, context, tool design, human approval, security).

## What’s in here

- [`SKILL.md`](./SKILL.md) — drop into Cursor / Claude Code / Codex-style skill folders, or paste as agent instructions
- This README — short overview you can share

## The five ingredients

1. **Tools** that can actually do things
2. **A text format** the model uses to call those tools
3. **A parser/runner** that executes those calls
4. **Result feedback** back into the next model turn
5. **A tool list** with short descriptions

Plus (for production agents): context rules, evals, traces, budgets, risk-class approvals, secret handling, and a ship checklist. See `SKILL.md`.

## How to use

1. Give `SKILL.md` to your coding agent as a skill or system doc.
2. Answer the interview questions (job, environment, permissions, approvals).
3. Get a one-pager + optional folder scaffold + coding-agent prompt for v0.

## License

MIT — use freely, share freely.
