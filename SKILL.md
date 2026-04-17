---
title: "Agent Ping Pong"
version: 1.0.0
created: 2026-04-16
modified: 2026-04-16
tags: [codex, github, workflow, agents, skill]
status: draft
owner: highnoonoffice
---

# Agent Ping Pong

**Your OpenClaw is the brain. Codex is the hands. The clipboard is the protocol.**

Agent Ping Pong is a two-agent coding workflow where OpenClaw acts as Maestro — speccing, reviewing, and directing — while Codex does the build work. You relay structured blocks between them by copy-paste. No direct agent-to-agent connection required. Just two windows and a clipboard.

The result: you ship real code to GitHub from a conversation. Your agent reviews the PR. You approve the merge. Repeat.

---

## What You Need

- **OpenClaw** — your Maestro. Higher-level intelligence. Specs the work, reviews PRs, sends critique.
- **Codex Desktop** — free with ChatGPT Plus. Does the build. Opens PRs. Never merges without approval.
- **GitHub account** — free. Source of truth. Where the code lives.
- **Vercel account** — free tier. Deploy when something's ready to go public.
- **A repo scoped PAT token** — GitHub Personal Access Token with access to one repo only. This is how Codex touches your GitHub without having the keys to everything.

---

## One-Time Setup

### 1. Install Codex Desktop
Download from [openai.com/codex](https://openai.com/codex). Requires ChatGPT Plus subscription. Install, sign in.

### 2. Create a GitHub repo for your project
Free account at github.com. One repo per project. Keep it focused.

### 3. Generate a scoped PAT token
In GitHub: Settings → Developer Settings → Personal Access Tokens → Fine-grained tokens.

Scope it to **one repo only**. Permissions needed:
- Contents: Read & Write
- Pull Requests: Read & Write
- Metadata: Read

This is the key insight: Codex only gets access to what you give it. Your other repos are untouched.

### 4. Connect Codex to your repo
In Codex Desktop: add your repo, paste the PAT when prompted. Codex can now read your code and open PRs against it.

### 5. Tell Codex one hard rule
In your first Codex message, establish the protocol:

```
Hard rule: open a PR against main for every build. Do not merge. Wait for review.
```

Say it once. It holds for the session.

---

## The Workflow Loop

This is the ping pong. Each volley is a structured block you copy-paste from one agent to the other.

```
YOU → OpenClaw:  "Here's what I want to build: [describe it]"

OpenClaw → YOU:  Spec block. Exact requirements, edge cases, constraints.
                 Copy this.

YOU → Codex:     Paste the spec block.

Codex → YOU:     "PR opened. Branch: feature/x. Commit: abc1234. Here's what I built."
                 Copy this.

YOU → OpenClaw:  Paste Codex's report.

OpenClaw → YOU:  Code review block. P0/P1/P2 findings. Fix instructions.
                 Copy this.

YOU → Codex:     Paste the review block.

Codex → YOU:     "Fixes applied. New commit: def5678."
                 Copy this.

YOU → OpenClaw:  Paste Codex's update.

OpenClaw → YOU:  "LGTM. Merge approved." or another review round.

YOU → Codex:     "Merge."  ← only you say this. Never OpenClaw directly.
```

The human never writes code. The human never writes to the agents in agent language. You describe intent to OpenClaw, relay blocks between them, and approve merges. That's the whole job.

---

## The Block Format

When OpenClaw hands you something to relay to Codex, it comes in a fenced code block. Copy the entire block including the fence markers. Paste it directly into Codex. Don't edit it.

When Codex reports back, copy its full response and paste it to OpenClaw with no wrapper. Just: "From Codex:" and paste.

The agents write to each other. You are the relay, not the translator.

---

## Code Review Format

When relaying a PR to OpenClaw for review, say:

```
Review this PR from Codex. Repo: [repo name]. PR: [number or URL]. Branch: [branch name].
```

OpenClaw will pull the code, read it, and return a review block formatted like this:

```
[Repo] — Code Review
[files changed]

---

P0 — [Must fix before merge]
File: path/to/file — function name
Issue: what's wrong
Why it matters: impact
Suggested patch: code or plain English fix

---

P1 — [Should fix]
...

P2 — [Nice to have]
...
```

Paste that block to Codex verbatim. It knows what to do with it.

---

## Merge Protocol

Only you merge. Never ask OpenClaw to merge. Never ask Codex to merge without OpenClaw's approval.

The sequence:
1. OpenClaw says "LGTM" or "approved"
2. You tell Codex: "Merge."
3. Codex merges the PR
4. Done

If OpenClaw sends back findings, another round of ping pong happens before merge.

---

## Token Management

Codex runs on your ChatGPT Plus subscription — flat rate, no per-token cost. Use it for:
- All build work
- File creation and editing
- Running tests and lint
- Fixing review findings

OpenClaw handles judgment — use it for:
- Speccing the work
- Reviewing PRs
- Architectural decisions
- Anything requiring real reasoning about intent

This split is why the workflow is financially sustainable. Codex carries the volume. OpenClaw carries the intelligence. You don't burn expensive tokens on mechanical work.

---

## Don't Have OpenClaw?

You can run a lighter version of this workflow with ChatGPT chat as your reviewer instead of OpenClaw. The loop is the same — spec in one window, build in Codex, paste the PR back for review.

What you're missing: OpenClaw has memory across sessions, can access your files, runs on your machine, and costs nothing beyond the API calls you make. It's a fundamentally different class of agent than a stateless chat window.

Get OpenClaw at [openclaw.ai](https://openclaw.ai).

---

## Real Example

Tonight's session: Joseph described an Agents Workshop UI module. OpenClaw specced the requirements. Codex built it and opened PR #3. OpenClaw reviewed the PR, found 2 P1s and 3 P2s, wrote a structured review block. Joseph relayed it to Codex. Codex fixed all five findings and pushed a new commit. OpenClaw verified the commit against the live branch. Merge approved. Codex merged. Module now on main.

Total tokens spent by OpenClaw: judgment, spec, review. Zero build tokens. That's the split.

---

## Tips

- **One repo per project.** Don't give Codex a PAT to a repo with production data or personal content.
- **Name your branches.** Tell Codex: "Branch name: feature/[short-description]." Keeps the PR history readable.
- **Feature branch → PR → approve → merge.** Never push direct to main for anything non-trivial.
- **Session context.** Start each Codex session with a one-paragraph context block: what the project is, what's already built, what this session is for. Codex doesn't have memory. Give it the brief.
- **When Codex goes sideways.** Paste the broken output to OpenClaw. Ask for a diagnosis. Relay the fix instruction back. One extra volley is cheaper than debugging blind.
