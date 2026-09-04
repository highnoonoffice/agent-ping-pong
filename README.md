# Agent Ping Pong

**A protocol for handing work between two agents through a human clipboard — spec, build, review, merge-gate.**

One agent does judgment: it specs the task, reviews the result, and holds the merge gate. The other agent does volume: it builds, opens a PR, and waits. A human relays structured blocks between them by copy-paste. No direct agent-to-agent connection required — just two windows and a clipboard.

The result: you ship real, reviewed code to GitHub from a conversation — with any orchestrator and any builder that can speak the block format.

**Reference implementation:** OpenClaw as the judgment agent, Codex or Claude Code as the build agent. That's the pairing this repo was built and proven on daily. But nothing about the protocol is OpenClaw-specific or Codex-specific — see [Works With Any Agent Pair](#works-with-any-agent-pair) below.

---

## How It Works

```
YOU → Judgment agent:  describe what you want to build
Judgment agent → YOU:  spec block — copy this
YOU → Build agent:     paste the spec
Build agent → YOU:     PR opened — copy this
YOU → Judgment agent:  paste the build agent's report
Judgment agent → YOU:  code review block — copy this
YOU → Build agent:     paste the review
Build agent → YOU:     fixes applied — copy this
YOU → Judgment agent:  paste the update
Judgment agent → YOU:  LGTM. Merge approved.
YOU → Build agent:     Merge.
```

The agents write to each other. You are the relay, not the translator. The judgment agent directs and reviews the work. The build agent never merges without explicit human approval.

---

## Works With Any Agent Pair

The protocol has exactly two structural requirements:

1. **A judgment agent** that can hold context, produce a spec block, review a diff, and gate the merge decision.
2. **A build agent** that can accept a spec block, do the work, and return a structured report — without wrapping it in prose that breaks the copy-paste.

That's it. Nothing in the `[AGENT_HANDOFF]` schema references any specific vendor.

**Judgment-agent options:**
- **OpenClaw** — the reference implementation for this repo
- **Hermes Agent** (Nous Research) — an open-source, provider-agnostic agent harness that can take the judgment role
- **Grok CLI** — a CLI-based Grok harness that can take the judgment role when configured with the protocol
- Any harness that can hold a conversation, write a structured block, and refuse to merge without your say-so

**Build-agent options:**
- **Codex** (OpenAI) and **Claude Code** (Anthropic) — the two build agents this repo was built and tested against
- **Grok Build / `grok-code-fast-1`** (xAI) — xAI's terminal coding agent and coding model
- **Qwen3-Coder**, **DeepSeek**, and **Kimi** — coding-capable model families that can fill the build role through an agent harness

The protocol does not claim identical behavior across these tools. Each pairing still needs a harness that can honor the block format, work inside the intended repository boundary, and stop at the human merge gate.

If your two agents can each hold up their end — write a block, keep it self-contained, wait for the human to hit send — this protocol works regardless of which lab built either one.

---

## What You Need

- A judgment agent — [OpenClaw](https://openclaw.ai), Hermes Agent, or any orchestrator that can spec, review, and gate
- A build agent — [Codex](https://openai.com/codex), Claude Code, Grok Build, or any agent that can accept a spec and open a PR
- GitHub account — free
- Vercel account — free tier, for when you're ready to deploy

---

## Get Started

Read [SKILL.md](./SKILL.md) for the full protocol spec, setup guide, workflow loop, review format, and tips.

---

[@highnoonoffice](https://github.com/highnoonoffice)
