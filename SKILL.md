---
name: Scaffold agent harness
description: >-
  Use when someone wants to design, specify, or scaffold an agent harness (tools
  + call syntax + parse/run loop + result feedback + tool list), or to turn a
  product idea into a concrete world-class harness config before writing code.
---
# Scaffold agent harness

Help the user design and scaffold an **agent harness**: the loop that turns an LLM’s text into real actions. An **agent** = LLM + harness. Do not confuse the chat/UI with the harness.

Theo’s five ingredients define *what* a harness is. This skill also covers *how to operate* one at a world-class level: evals, traces, budgets, context, tool design, human loop, security, and optional multi-agent.

## Goal

Produce a concrete harness scaffold the user (or a coding agent) can implement: tools, call format, parse/run loop, result feedback, tool list, plus runtime quality, context rules, safety, and a ship checklist.

## Recipe (what every harness needs)

1. **Tools** that can actually do things (read file, edit file, run shell, search, HTTP, etc.)
2. **A text format** the model uses to call those tools (JSON tool calls, XML tags, or the host SDK’s format)
3. **A parser/runner** that spots that format and executes the matching code
4. **Result feedback** that sends tool output back into the next model turn
5. **A tool list** with short descriptions so the model knows what exists and how to use it

Optional but usually needed:
- **Instructions / system prompt** (role, constraints, domain)
- **Human approval** before risky tools run
- **MCP servers** for capabilities that live outside this process (remote APIs, other apps)
- **Interface** (CLI, chat UI, IDE) that sits *on top* of the harness, not inside it

## Steps

### 1. Interview (ask only what’s missing)

Collect briefly:
- **Job**: what should this agent accomplish for the user?
- **Environment**: where it runs (local machine, server, browser, IDE plugin, Slack bot, etc.)
- **Permissions**: what it may touch (files, shell, network, spend money, send messages)
- **Approval**: which actions need a human yes before running
- **Existing harness**: reuse Cursor / Claude Agent SDK / Codex / Pi / custom, or build new?
- **External systems**: APIs, DBs, Slack, email, GitHub, etc. (candidates for MCP)
- **Success bar**: what “world-class” means for this agent (latency, cost, accuracy, safety)

If the user already answered these, skip questions and proceed.

### 2. Separate the layers

Write three short lists for the user:
- **Interface** (what humans see): chat, CLI, buttons, approval prompts
- **Harness** (the loop): tools, call format, parse/run, result feedback, tool catalog
- **Model** (rented intelligence): which LLM, temperature, context limits

Refuse to mix “pretty UI” into the harness design. UI consumes harness events; it is not the harness.

### 3. Design the tool plane

For each capability, choose **local tool** vs **MCP**:
- **Local tool** = code that runs in this process / on this machine
- **MCP** = call something that lives elsewhere (don’t reimplement)

Prefer:
- Local tools for filesystem, shell, and tight loops
- MCP for third-party products and shared company systems

Output a table:
| Capability | Local tool or MCP | Risk (low/med/high) | Needs human approval? | Notes |

### 4. Spec the five harness pieces

Write a scaffold doc with these sections filled in for *this* agent:

**A. Tool catalog**
For each tool: `name`, one-line description, inputs, outputs, side effects, approval required yes/no.

**B. Call format**
Pick one and stick to it (prefer the host SDK’s native tool-calling if available). Show 1–2 example model messages that call a tool.

**C. Parse → run loop**
Pseudocode:
1. Send messages + tool list to model
2. If response has tool call(s) → validate → (optional approve) → execute
3. Append tool result(s) as messages
4. Repeat until model returns a final answer or hits a stop condition

**D. Result feedback**
How errors, stdout, and structured data are returned (keep payloads small; truncate large blobs).

**E. Instructions**
Short system prompt: goal, tool-use rules, what never to do, when to ask the human.

### 5. Tool design (world-class)

Require in the scaffold:
- Few sharp tools over one open-ended `do_anything`
- Typed inputs/outputs the model can’t half-parse
- Clear side-effect labels (read / write / send / spend)
- Dry-run or plan mode before mutating tools when feasible
- Sandboxing for shell and file writes (cwd allowlist, network allowlist)

### 6. Context engineering

Specify:
- What is injected every turn (instructions, tool list, memory, open files) and hard size limits
- Compaction rules when context fills (what to summarize, drop, or pin)
- Explicit memory write path: what is stored, when, retention; never store secrets

### 7. Runtime quality

Require:
- **Evals**: fixed tasks with pass/fail; run on every prompt or tool change
- **Traces**: model in → tool calls → results → final out (debuggable end-to-end)
- **Budgets**: max turns, max tool calls, wall-clock timeout, token/cost caps
- **Retries**: idempotent where possible; typed errors returned to the model

### 8. Human loop

Specify:
- Approval policy by **risk class**, not one global toggle
- Interrupt / cancel mid-loop
- Undo or rollback notes for mutating tools where possible

### 9. Trust and security

Always include unless the user explicitly opts out:
- Allowlist of tools in v0 (no open-ended “run anything”)
- Approval gate for shell, file deletes, sends, payments, and credential use
- Secrets only from env / vault; never in prompts or logs
- Treat tool results and MCP/web content as untrusted (prompt-injection aware)
- Least privilege per config; separate read-only vs can-send/spend configs
- Log tool name + args summary (never raw secrets)

### 10. Multi-agent (only if needed)

If the job needs subagents, specify:
- When to spawn a child vs stay in one loop
- What context the child inherits and what it may return
- Who owns approvals in a tree of agents

Skip this section for single-loop agents.

### 11. Deliverables

Hand the user:
1. **Harness one-pager** (layers + tool table + five pieces + sections 5–10 as relevant)
2. **Folder scaffold** (create files only if they asked to generate code or a repo layout), e.g.:
   - `harness/README.md` — one-pager
   - `harness/tools/` — one module per local tool
   - `harness/loop.ts` or `loop.py` — parse → run → feedback
   - `harness/catalog.json` — tool list for the model
   - `harness/instructions.md` — system prompt
   - `harness/mcp.json` — MCP server connections (if any)
   - `harness/approvals.md` — risk classes and what needs human yes
   - `harness/evals/` — golden tasks + expected outcomes
   - `harness/context.md` — injection and compaction rules
3. **v0 cut**: smallest tool set that proves the loop (usually 2–4 tools)
4. **Ship checklist**: allowlist, one golden-path demo, failure modes (timeout, tool error, user deny), log/trace export
5. **Next build step**: exact prompt they can give a coding agent to implement the loop

### 12. Naming check

Before finishing, rewrite any user language that says “the agent does X” into harness terms when useful:
- “Agent” → LLM + this harness config
- “Memory” → what you store and inject into context
- “Plugin” → local tool or MCP server

## Style

- Plain words. No kitchen/restaurant metaphors unless the user used them first.
- Lead with the scaffold, not theory.
- Prefer tables and short checklists over essays.
- If they only want the recipe reminder, give the five ingredients and stop.

## Non-goals

- Do not invent a new LLM or fine-tune as part of this skill.
- Do not build a full product UI unless asked.
- Do not claim a chat wrapper without tools/loop is a harness.
- Do not add multi-agent complexity unless the job clearly needs it.
