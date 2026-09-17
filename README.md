# Agent Ping Pong

**A protocol for handing work between two agents through a human clipboard — spec, build, review, merge-gate.**

One agent does **Judgment**: it specs the task, reviews the result, and recommends merge. The other does **Build**: it implements, opens a PR, and waits. A human relays structured `[AGENT_HANDOFF]` blocks by copy-paste and is the **sole merge authority**. No direct agent-to-agent connection required — two windows and a clipboard.

**v3.0** is vendor-neutral. OpenClaw + Codex/Claude Code remains a worked reference setup in [SKILL.md](./SKILL.md), not the definition of the protocol.

---

## How It Works

```
YOU → Judgment:   describe what you want
Judgment → YOU:   full spec block — copy this
YOU → Builder:    paste the spec
Builder → YOU:    delivery (PR) — copy this
YOU → Judgment:   paste the delivery
Judgment → YOU:   short review stamp — copy this
YOU → Builder:    paste the review
Builder → YOU:    fixes (if any) — copy this
YOU → Judgment:   paste the update
Judgment → YOU:   verdict: go
YOU → Builder:    Merge.
```

You are the relay, not the translator. LGTM ≠ merge.

---

## Two Block Shapes

Same wrapper. Different field weight:

- **Full** — `spec` / `delivery` / `decision` / `diagnostic` (goal, done-when, scope, constraints, verification, …)
- **Short** — `review` / `ack` / `status` (verdict + thread; optional PR/note)

Guidance, not a linter. Details and type map: [SKILL.md](./SKILL.md).

---

## Default Repo Pattern

Builder opens a **PR on the target repo**. Judgment reviews. **Human merges.**

A separate sandbox repo is optional (legacy OpenClaw+Codex port flow) — see Reference setups in SKILL.md.

---

## Works With Any Agent Pair

1. A Judgment agent that can spec, review a diff, and stop at the human merge gate.
2. A Builder agent that can accept a block, work in-repo, open a PR, and return a clean block without prose wrapping the fence.

`from` / `to` are roster names — not hard-coded vendors.

---

## What You Need

- Judgment agent (any orchestrator that can hold the bar)
- Builder agent (any coding agent that can open PRs)
- GitHub
- This skill's block format ([SKILL.md](./SKILL.md))

---

## Get Started

Read [SKILL.md](./SKILL.md) for gates, short/full shapes, merge protocol, and reference setups.

---

[@highnoonoffice](https://github.com/highnoonoffice)
