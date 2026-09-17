---
name: ping-pong
version: 3.0.0
description: "Human clipboard relay between a Judgment agent and a Builder agent. Structured [AGENT_HANDOFF] blocks, PR review, human merge gate."
homepage: https://github.com/highnoonoffice/agent-ping-pong
source: https://github.com/highnoonoffice/agent-ping-pong
license: MIT
credentials:
  - name: GitHub access (Judgment agent)
    description: Fine-grained PAT or equivalent so the Judgment agent can read PRs and review diffs on the target repo (and port/push only if you choose that pattern). Contents + Pull Requests as needed. Optional until you wire GitHub review.
    required: false
  - name: GitHub access (Builder agent)
    description: Fine-grained PAT or equivalent so the Builder can push branches and open PRs on the target repo (or an optional sandbox repo — see Reference setups). Prefer least privilege. Optional until you wire builds.
    required: false
binaries: []
---

# Agent Ping Pong

A two-role coding workflow: a **Judgment** agent specs and reviews; a **Builder** agent implements and opens PRs; a **human** relays `[AGENT_HANDOFF]` blocks by copy-paste and is the sole merge authority. No direct agent-to-agent network required — two windows and a clipboard.

Vendor-neutral by default. OpenClaw + Codex/Claude Code is a worked **Reference setup** at the end, not the identity of the protocol.

---

## Execution Gates (core)

```xml
<skill_gates version="1.0" mode="mandatory_pre_execution" order="sequential" on_violation="stop_and_report">

  <gate id="block_terminator" priority="1" severity="hard" scope="pre_handoff">
    <condition>About to relay any [AGENT_HANDOFF] block</condition>
    <question>Does the block end with `Reply with a single [AGENT_HANDOFF] block. No prose outside the block.` as the last line before the closing tag?</question>
    <pass_action>Proceed.</pass_action>
    <fail_action>Stop. Add the terminator line. Without it, the other agent may return prose-wrapped output and the clipboard relay breaks.</fail_action>
  </gate>

  <gate id="no_secrets_in_blocks" priority="2" severity="hard" scope="pre_handoff">
    <condition>About to put a credential, API key, token, or secret in a handoff block</condition>
    <question>Is any raw secret value present in the block payload?</question>
    <pass_action>No secrets — proceed.</pass_action>
    <fail_action>Stop. Remove secrets. Clipboard contents can be read by other processes. Credentials stay in agent/config stores, never in blocks.</fail_action>
  </gate>

  <gate id="no_merge_without_approval" priority="3" severity="hard" scope="pre_merge">
    <condition>About to instruct anyone to merge a PR</condition>
    <question>Has the human explicitly approved the merge after seeing Judgment's verdict?</question>
    <pass_action>Proceed with the merge instruction.</pass_action>
    <fail_action>Stop. Never auto-merge. Judgment "LGTM" / "approved" is a recommendation, not authorization. Wait for explicit human approval.</fail_action>
  </gate>

  <gate id="single_artifact" priority="4" severity="hard" scope="pre_handoff">
    <condition>About to deliver a handoff for the human to copy</condition>
    <question>Is this one fenced artifact that starts with [AGENT_HANDOFF] and ends with [/AGENT_HANDOFF], with no prose inside that fence?</question>
    <pass_action>Proceed.</pass_action>
    <fail_action>Stop. Resend the entire corrected block as one copy-pasteable fence. No patches-only, no split messages, no commentary between the tags.</fail_action>
  </gate>

  <gate id="pr_before_default_branch" priority="5" severity="hard" scope="pre_land">
    <condition>Builder is about to land code on the default branch</condition>
    <question>Is the change going through a PR, with Judgment review and human merge approval — not a direct push to the default branch?</question>
    <pass_action>Proceed.</pass_action>
    <fail_action>Stop. Unapproved code must not land on the default branch. Open a PR; wait for review + human merge.</fail_action>
  </gate>

</skill_gates>
```

Vendor-specific gates (sandbox repo name, Telegram copy chat, "Build this for Cody", port-from-sandbox) live in **Reference setups** — not core.

---

## Roles

| Role | Job |
|------|-----|
| **Human** | Relay blocks; intercept when needed; sole merge authority |
| **Judgment** | Spec, review PRs, hold the quality bar; never treat LGTM as merge |
| **Builder** | Implement, open PRs, fix findings; never merge without explicit human "Merge." |

`from` / `to` in blocks are **roster names** (any agents). Examples: Gary, Maria, Cody, Anders, Magnus — or Judgment / Builder in minimal two-window setups.

---

## Default repo pattern (core)

**Direct PR on the target repo** (validated in multi-agent crews such as Drawbridge):

1. Builder opens a PR on the project repo you actually ship.
2. Human pastes the delivery block to Judgment.
3. Judgment reviews and returns a short or full review block.
4. Builder fixes if needed; loop until Judgment recommends merge.
5. **Human** says merge. Builder (or human) merges.

Least-privilege tokens are encouraged. A separate sandbox repo is **optional** — see Reference setups.

Invariant: *unapproved code must not land on the default branch; PR + human merge gate.*

---

## Two block shapes (guidance, not a linter)

Same wrapper always. Shape = which fields this volley usually needs. Missing a field does **not** invalidate a block — it may just burn a round. Prefer the thin shape when a stamp is enough.

### Full block

**Use for:** `spec` | `delivery` | `decision` | `diagnostic`

**Typical fields:**

```
[AGENT_HANDOFF]
type: spec
from: Judgment
to: Builder
thread: csv-summary
goal: outcome this advances
definition_of_done: observable pass/fail
scope: may change / must not change
constraints: merge, security, product, timing
assumptions: verified / inferred / unknown
confidence: high | medium | low
open_questions: blockers or choices
verification: tests, review lens, evidence, PR link when known
repo: owner/target-repo
branch: feature/short-name
Reply with a single [AGENT_HANDOFF] block. No prose outside the block.
[/AGENT_HANDOFF]
```

Useful optional full fields: `non_goals`, `success_intent`, `review_lens`, `publish_target`, `user_flow_must_pass`.

### Short block

**Use for:** `review` | `ack` | `status`

**Typical fields:**

```
[AGENT_HANDOFF]
type: review
from: Judgment
to: Builder
thread: csv-summary
verdict: go | no-go | need-changes | received | done
open_questions: none
pr: https://github.com/owner/repo/pull/1
note: optional one line
Reply with a single [AGENT_HANDOFF] block. No prose outside the block.
[/AGENT_HANDOFF]
```

Short blocks do **not** need goal / definition_of_done / scope / constraints / assumptions / confidence / long verification — those live on the full block that opened the thread.

### Type map (2.x → 3.0)

| Older type names | Shape | Notes |
|------------------|-------|-------|
| `spec` | full | unchanged idea |
| `delivery` | full | build/PR report |
| `decision` / `diagnostic` | full | product or debug briefs |
| `review_verdict` / review findings | short `review` | stamp + optional PR; attach long P0/P1 lists in a **separate** message if needed, or keep findings outside the short stamp |
| `acknowledgment` | short `ack` | received / confirmed |
| `schema_check` | full or short | prefer short if only confirming format |

Old blocks still parse. 3.0 clarifies thin vs thick; it does not reject 2.x field sets.

---

## The Aesthetic

Judgment speaks in blocks. Builder speaks in blocks. Blocks are addressed to each other — not to you.

**Judgment** may put context above/below the fenced block in its chat with you. You read that; you copy only the fence.

**Builder** should keep the relay message as the fence alone (or clearly one fence). Prose outside the fence gets copied into the other agent and pollutes the protocol.

**Every block requests a block back** via the terminator line.

---

## Quick Start

**1. You → Judgment:** describe what you want.

**2. Judgment → you:** full `spec` block. Copy the fence. Paste to Builder.

**3. Builder → you:** full `delivery` (PR URL, branch, commit, files). Copy. Paste to Judgment.

**4. Judgment → you:** short `review` (`verdict: go` or `need-changes`). If findings are long, put the P0/P1 detail in a following message; keep the stamped handoff copy-paste clean.

**5. You → Builder:** paste review (and findings if any). Builder fixes; short or full `delivery` again.

**6. Judgment recommends merge. You say:** `Merge.` Builder merges. Done.

---

## One-Click Rule

Every handoff is one copy-pasteable artifact: starts `[AGENT_HANDOFF]`, ends `[/AGENT_HANDOFF]`, one fence, zero prose inside the fence.

1. Deliver the fence in its own message when possible.
2. Never split one handoff across messages.
3. On correction, resend the **entire** block — not a patch note.
4. Commentary for the human stays outside the fence.

---

## Code Review

Trigger Judgment with something like:

```
Review this PR from Builder. Repo: [repo]. PR: [number or URL]. Branch: [branch].
```

Judgment returns a short `review` stamp for relay, plus (as needed) structured findings:

```
P0 — must fix before merge
File: path — symbol
Issue: ...
Why it matters: ...
Suggested fix: ...

P1 — should fix
...
```

Paste the stamp (and findings) to Builder. Builder never merges on findings alone.

---

## Human in the Loop

**Relay mode (default):** copy fences without deep reading; you're the wire.

**Review mode:** read before paste; edit direction; add your own instruction. Design, not a break.

You always hit send. Merge is only your call.

---

## Merge Protocol

1. Judgment recommends (`verdict: go` / LGTM).
2. **You** tell Builder: `Merge.`
3. Builder merges the PR (or you merge in the UI).
4. Done.

Findings ⇒ another ping-pong round before merge.

---

## Security

- No secrets in blocks.
- Prefer fine-grained, least-privilege GitHub tokens.
- Default branch is protected by PR + human merge — not by hoping agents behave.

---

## Tips

- One PR per feature; keep scope tight.
- If Builder is unsure of scope, it asks before building (full block with `open_questions`).
- Prefer short acks for "got it" / "claimed" / "merged."
- Prefer full specs when a wrong guess costs a round trip.
- Thread ids (`thread: n12-join-new`) keep volleys aligned across agents.

---

## Session Learnings (preserved)

### 2026-05-05
Thin volleys that still carry thick fields burn clipboard time. Say when a stamp is enough. Preventable rounds often come from missing `definition_of_done` or `non_goals` on the opening full spec — not from missing ceremony on the ack.

### 2026-04-21
Both ends must enforce the terminator line. Schema negotiation belongs in a block. Human merge gate is non-negotiable even when Judgment is trusted.

---

## Reference setups (appendix — not core)

### A. Direct PR on target repo (default)

Builder PAT: Contents + PRs on `owner/project`. Judgment PAT: read PRs (write only if it also ports). Human merges. Matches multi-agent crews on a shared product repo.

### B. Sandbox → port (legacy OpenClaw + Codex pattern)

Optional when the Builder must never touch production:

- Sandbox repo (historically `codex-repo`) for all Builder writes.
- Production repo receives code only after review + human approval, usually ported by Judgment/OpenClaw.
- Separate fine-grained PATs: Builder = sandbox only; Judgment = sandbox + production.
- Historical OpenClaw-specific ops (Telegram copy chat, "Build this for Cody" trigger, port-after-sandbox-merge) stay here if you still run that harness — they are **not** protocol requirements.

### C. Example vendors

Judgment examples: OpenClaw, Hermes, Grok Bot/CLI, any orchestrator that can spec/review/gate.  
Builder examples: Codex, Claude Code, Grok Build, other coding harnesses that can open PRs and return a clean block.

---

## Trigger phrases (optional house style)

Teams may define their own. One historical High Noon phrase: **"Build this for Cody"** meant "discussion over — emit the first handoff now." Use roster-neutral equivalents as you like ("Build this for Builder", "Ship the handoff").

---

@version 3.0.0 — Agent-agnostic roles; short/full guidance; direct-PR default; sandbox→port moved to appendix; core gates unchanged in spirit (terminator, no secrets, human merge, single artifact, PR before default branch).
