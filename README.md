# Agent Ping Pong

**Your OpenClaw is the brain. Codex is the hands. The clipboard is the protocol.**

Agent Ping Pong is a two-agent coding workflow where OpenClaw acts as Maestro and Codex does the build work. You relay structured blocks between them by copy-paste. No direct agent-to-agent connection. Just two windows and a clipboard.

The result: you ship real code to GitHub from a conversation.

---

## How It Works

```
YOU → OpenClaw:  describe what you want to build
OpenClaw → YOU:  spec block — copy this
YOU → Codex:     paste the spec
Codex → YOU:     PR opened — copy this
YOU → OpenClaw:  paste Codex's report
OpenClaw → YOU:  code review block — copy this
YOU → Codex:     paste the review
Codex → YOU:     fixes applied — copy this
YOU → OpenClaw:  paste the update
OpenClaw → YOU:  LGTM. Merge approved.
YOU → Codex:     Merge.
```

The agents write to each other. You are the relay, not the translator.

---

## What You Need

- [OpenClaw](https://openclaw.ai) — your Maestro
- [Codex Desktop](https://openai.com/codex) — free with ChatGPT Plus
- GitHub account — free
- Vercel account — free tier, for when you're ready to deploy

---

## Get Started

Read [SKILL.md](./SKILL.md) for the full setup guide, workflow loop, review format, and tips.

---

[@highnoonoffice](https://github.com/highnoonoffice)
