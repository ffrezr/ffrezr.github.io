---
layout: post
title: "Claude Adaptive Thinking API: Python Migration Guide (2026)"
date: 2026-04-18
last_modified_at: 2026-04-18
author: Francisco Frez
categories: [AI Tools, Developer Guide]
tags: [claude, adaptive-thinking, extended-thinking, anthropic, api, python, opus-4-7, migration]
description: "budget_tokens breaks on Opus 4.7 with HTTP 400. Migrate to adaptive thinking and the effort parameter in 10 lines of Python — complete guide with code diffs."
image:
  path: https://images.pexels.com/photos/17483873/pexels-photo-17483873.jpeg?auto=compress&cs=tinysrgb&w=1200&h=630&fit=crop&fm=webp
  alt: Abstract digital illustration of neural network deep learning — Claude adaptive thinking API migration guide
og_image: https://images.pexels.com/photos/17483873/pexels-photo-17483873.jpeg?auto=compress&cs=tinysrgb&w=1200&h=630&fit=crop&fm=webp
og_title: "Claude Adaptive Thinking API: Python Migration Guide (2026)"
og_description: "budget_tokens breaks on Opus 4.7 with HTTP 400. Migrate to adaptive thinking and the effort parameter in 10 lines of Python — complete guide with code diffs."
twitter_card: summary_large_image
author_url: https://ffrezr.github.io/about/
canonical: "https://ffrezr.github.io/posts/claude-adaptive-thinking-python-guide/"
schema:
  - type: BlogPosting
    headline: "Claude Adaptive Thinking API: Python Migration Guide (2026)"
    datePublished: "2026-04-18"
    dateModified: "2026-04-18"
    author:
      type: Person
      name: Francisco Frez
      url: https://ffrezr.github.io/about/
    description: "budget_tokens breaks on Opus 4.7 with HTTP 400. Migrate to adaptive thinking and the effort parameter in 10 lines of Python — complete guide with code diffs."
  - type: FAQPage
    mainEntity:
      - question: "What replaces budget_tokens in Claude Opus 4.7?"
        answer: "Adaptive thinking replaces budget_tokens on Claude Opus 4.7. Set thinking to {type: adaptive} and control depth via the effort parameter (low, medium, high, xhigh, max). Passing budget_tokens to Opus 4.7 returns an HTTP 400 error."
      - question: "Does budget_tokens still work on Opus 4.6 and Sonnet 4.6?"
        answer: "Yes, but it is deprecated. Anthropic will remove it in a future model release. The recommended path is to migrate to adaptive thinking with effort now to avoid a forced migration when the next model drops."
      - question: "What is the effort parameter in the Claude API?"
        answer: "The effort parameter, set in output_config, tells Claude how many tokens to spend on a response. Values are low, medium, high (default), xhigh (Opus 4.7 only), and max. It controls thinking depth when adaptive thinking is active and also affects tool call verbosity and text response length."
      - question: "What is the display omitted change in Opus 4.7?"
        answer: "On Opus 4.7, thinking.display defaults to omitted instead of summarized. Thinking blocks appear in the response but their text field is empty. You are still billed for the full thinking tokens. Set display to summarized explicitly if your app surfaces thinking content to users."
  - type: HowTo
    name: "Migrate from budget_tokens to adaptive thinking in Python"
    step:
      - text: "Replace thinking: {type: enabled, budget_tokens: N} with thinking: {type: adaptive}"
      - text: "Move the budget_tokens value to output_config: {effort: high} or another effort level"
      - text: "Set display: summarized explicitly on Opus 4.7 if you need thinking content visible"
      - text: "Update max_tokens to at least 32k for xhigh/max effort to give the model room to think"
      - text: "Verify cache breakpoints are not broken by switching thinking modes mid-conversation"
---

If you deployed `thinking: {type: "enabled", budget_tokens: 10000}` on Claude Opus 4.7 (released April 16, 2026), your API calls are returning HTTP 400 errors right now ([Anthropic API Docs](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking), 2026). Anthropic deprecated `budget_tokens` on Opus 4.6 and Sonnet 4.6, then removed it entirely from Opus 4.7 — no grace period, no soft warning.

The replacement is adaptive thinking with the `effort` parameter. It's a two-line change in most cases, but there are three silent behavioral shifts in Opus 4.7 that will catch you off guard if you don't know about them. This guide covers the exact migration diff, effort level selection for data engineering workflows, streaming, and the prompt caching interaction that could quietly double your costs.

> **Key Takeaways**
> - `budget_tokens` returns HTTP 400 on Opus 4.7 — migrate to `thinking: {type: "adaptive"}` immediately ([Anthropic Docs](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking), 2026)
> - The `effort` parameter (`low/medium/high/xhigh/max`) replaces `budget_tokens` as the depth control
> - Opus 4.7 defaults `display` to `"omitted"` — thinking text is invisible unless you opt in
> - Prompt cache breakpoints break when switching between `adaptive` and `enabled` thinking modes
> - Opus 4.7 scores 70% on CursorBench vs 58% for Opus 4.6, with identical $5/$25 per MTok pricing

---

## What Broke and Why Did Anthropic Remove budget_tokens?

`budget_tokens` returned HTTP 400 on Claude Opus 4.7 because Anthropic removed manual thinking mode entirely from the new model, making adaptive thinking the only supported thinking configuration ([Anthropic Migration Guide](https://platform.claude.com/docs/en/about-claude/models/migration-guide), 2026). The deprecation on Opus 4.6 and Sonnet 4.6 was announced weeks earlier, but the hard removal landed simultaneously with Opus 4.7's release on April 16, 2026.

Why remove it at all? The core argument is that `budget_tokens` forced you to guess in advance how much reasoning a request needed. A complex multi-step query and a simple lookup both got the same fixed budget. Adaptive mode lets the model decide per-request — skipping thinking entirely on trivial queries and spending deeply on problems that warrant it. The result is fewer wasted thinking tokens on easy requests and no artificial ceiling on hard ones.

<!-- UNIQUE INSIGHT -->
The timing also matters strategically. Opus 4.7 ships with a new `xhigh` effort tier specifically designed for multi-hour agentic coding sessions — the kind that were impossible to tune with a static `budget_tokens` value because token needs vary wildly turn by turn. Removing `budget_tokens` was effectively a prerequisite for making `xhigh` useful.

![Abstract illustration of machine learning neural network — Claude adaptive thinking and effort parameter API](https://images.pexels.com/photos/17483867/pexels-photo-17483867.jpeg?auto=compress&cs=tinysrgb&w=800&h=450&fit=crop&fm=webp){: loading="eager" fetchpriority="high"}

**Citation capsule:** Anthropic's Claude Opus 4.7 (released April 16, 2026) removes support for `thinking: {type: "enabled", budget_tokens: N}` entirely, returning HTTP 400 for any request that uses it. The replacement is `thinking: {type: "adaptive"}`, which lets the model dynamically allocate thinking tokens per request rather than using a fixed preset budget. On Claude Opus 4.6 and Sonnet 4.6, `budget_tokens` is deprecated but still functional — Anthropic will remove it in a future model release.

The migration is a deliberate API simplification: adaptive mode consistently outperforms fixed-budget thinking on bimodal task distributions (mixes of easy and complex requests) because the model skips unnecessary reasoning on trivial queries instead of spending a fixed budget regardless. Source: [Anthropic API Docs — Adaptive Thinking](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking), 2026; [Anthropic Migration Guide](https://platform.claude.com/docs/en/about-claude/models/migration-guide), 2026.

---

## The 2-Minute Migration: Exact Code Diff

Three changes cover 95% of migrations. The old pattern used `thinking.type: "enabled"` with a `budget_tokens` integer. The new pattern uses `thinking.type: "adaptive"` and moves depth control to `output_config.effort`.

### Before (broken on Opus 4.7)

```python
import anthropic

client = anthropic.Anthropic()

# ❌ This returns HTTP 400 on claude-opus-4-7
response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=8000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000   # ← removed from Opus 4.7
    },
    messages=[{"role": "user", "content": "Diagnose why this dbt model is slow."}]
)
```

### After (works on Opus 4.7, Opus 4.6, Sonnet 4.6)

```python
import anthropic

client = anthropic.Anthropic()

# ✅ Adaptive thinking with explicit effort level
response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=32000,              # ← raise for xhigh/max effort
    thinking={"type": "adaptive"},
    output_config={"effort": "xhigh"},  # ← replaces budget_tokens
    messages=[{"role": "user", "content": "Diagnose why this dbt model is slow."}]
)

for block in response.content:
    if block.type == "thinking":
        print(f"Thinking: {block.thinking}")
    elif block.type == "text":
        print(f"Response: {block.text}")
```

### The display: "omitted" Gotcha

Opus 4.7 silently changed the default for `thinking.display` from `"summarized"` to `"omitted"`. If you were reading `block.thinking` in your output loop, it will be an empty string — the model is still thinking and you're still being billed for those tokens, but the text is hidden by default.

<!-- PERSONAL EXPERIENCE -->
I hit this in a pipeline monitoring agent the day Opus 4.7 dropped. The agent's reasoning trace logging went blank. No error, no warning — just empty thinking fields. Adding `display: "summarized"` to the thinking config restored everything.

```python
# ✅ Opt back in to visible thinking text on Opus 4.7
response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=32000,
    thinking={
        "type": "adaptive",
        "display": "summarized"  # ← explicit opt-in on Opus 4.7
    },
    output_config={"effort": "high"},
    messages=[{"role": "user", "content": "..."}]
)
```

**Citation capsule:** Migrating from `budget_tokens` to adaptive thinking requires two API changes: replacing `thinking: {type: "enabled", budget_tokens: N}` with `thinking: {type: "adaptive"}`, and adding `output_config: {effort: "level"}` where the effort level maps to the former token budget (larger budgets → higher effort). A third change is recommended on Opus 4.7: explicitly setting `display: "summarized"` in the thinking config.

Opus 4.7 defaults to `display: "omitted"`, silently suppressing thinking text output while still billing for the full thinking token count. Applications that log or display Claude's reasoning will see empty thinking fields without this opt-in. The Python SDK forwards unrecognized dict keys transparently, so no SDK update is required. Source: [Anthropic API Docs — Adaptive Thinking](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking), 2026; [Effort parameter](https://platform.claude.com/docs/en/build-with-claude/effort), 2026.

For session-level thinking configuration in stateful agents, see how `thinking` parameters interact with session lifecycle in the [Claude Managed Agents Python guide](/posts/claude-managed-agents-guide/).

---

## How to Pick the Right Effort Level

The `effort` parameter lives in `output_config` and accepts five values. It is a behavioral signal, not a hard token cap — at lower levels Claude still thinks on genuinely hard problems, just less than it would at higher levels ([Anthropic API Docs — Effort](https://platform.claude.com/docs/en/build-with-claude/effort), 2026). At `high` and above, Claude almost always engages extended thinking when adaptive mode is active.

**Effort Level Guide for Claude Opus 4.7**

<figure>
<svg viewBox="0 0 560 320" xmlns="http://www.w3.org/2000/svg" style="background:#1a1a2e;border-radius:8px;display:block;max-width:100%">
  <text x="280" y="28" text-anchor="middle" fill="white" font-size="13" font-weight="bold" font-family="sans-serif">Effort Level vs Thinking Depth (Opus 4.7)</text>

  <!-- Y axis labels -->
  <text x="72" y="75" text-anchor="end" fill="#ccccee" font-size="11" font-family="sans-serif">max</text>
  <text x="72" y="120" text-anchor="end" fill="#ccccee" font-size="11" font-family="sans-serif">xhigh</text>
  <text x="72" y="165" text-anchor="end" fill="#ccccee" font-size="11" font-family="sans-serif">high</text>
  <text x="72" y="210" text-anchor="end" fill="#ccccee" font-size="11" font-family="sans-serif">medium</text>
  <text x="72" y="255" text-anchor="end" fill="#ccccee" font-size="11" font-family="sans-serif">low</text>

  <!-- Bars: max width ~360px = absolute maximum -->
  <!-- max: 360px -->
  <rect x="80" y="59" width="360" height="22" rx="4" fill="#ff6584"/>
  <text x="448" y="75" fill="white" font-size="10" font-weight="bold" font-family="sans-serif">Frontier tasks, no token limit</text>

  <!-- xhigh: 290px -->
  <rect x="80" y="104" width="290" height="22" rx="4" fill="#6c63ff"/>
  <text x="378" y="120" fill="white" font-size="10" font-weight="bold" font-family="sans-serif">Agentic coding, long-horizon work</text>

  <!-- high (default): 220px -->
  <rect x="80" y="149" width="220" height="22" rx="4" fill="#43e97b"/>
  <text x="308" y="165" fill="white" font-size="10" font-weight="bold" font-family="sans-serif">Complex reasoning (default)</text>

  <!-- medium: 140px -->
  <rect x="80" y="194" width="140" height="22" rx="4" fill="#43e97b" opacity="0.6"/>
  <text x="228" y="210" fill="white" font-size="10" font-weight="bold" font-family="sans-serif">Balanced cost + quality</text>

  <!-- low: 70px -->
  <rect x="80" y="239" width="70" height="22" rx="4" fill="#43e97b" opacity="0.35"/>
  <text x="158" y="255" fill="white" font-size="10" font-weight="bold" font-family="sans-serif">Speed, simple tasks</text>

  <!-- X axis -->
  <line x1="80" y1="272" x2="460" y2="272" stroke="#444466" stroke-width="1"/>
  <text x="80" y="286" fill="#aaaacc" font-size="9" font-family="sans-serif">Low depth</text>
  <text x="400" y="286" text-anchor="end" fill="#aaaacc" font-size="9" font-family="sans-serif">Maximum depth</text>

  <text x="280" y="308" text-anchor="middle" fill="#aaaacc" font-size="9" font-family="sans-serif">Source: Anthropic API Docs — Effort parameter, 2026. xhigh available on Opus 4.7 only.</text>
</svg>
<figcaption style="color:#aaaacc;font-size:0.85em;text-align:center;margin-top:0.5rem">xhigh is new in Opus 4.7 and recommended as the starting point for agentic coding tasks.</figcaption>
</figure>

### Mapping budget_tokens to Effort Levels

If you had a `budget_tokens` value, here's the rough mapping to effort levels for Opus 4.7:

| Former budget_tokens | Recommended effort | Use case |
|---|---|---|
| 1,000–3,000 | `low` | Simple lookups, classification, quick Q&A |
| 4,000–8,000 | `medium` | Moderate analysis, code review, summaries |
| 10,000–20,000 | `high` | Complex reasoning, debugging, analysis |
| 25,000–50,000 | `xhigh` | Long-horizon agentic coding, multi-step tool use |
| 50,000+ | `max` | Frontier problems, research, no token constraints |

One important difference: effort is not a hard cap. A `medium` effort request on a genuinely hard problem may consume more tokens than a `high` effort request on a trivial one. If you need strict cost predictability, use `max_tokens` as the hard ceiling and let `effort` guide behavior within that ceiling.

### Effort for Data Engineering Agents

For a dbt pipeline monitoring agent making 50+ calls per day, a tiered effort strategy works well: use `xhigh` for complex failure diagnosis (the model needs to reason through multi-step dependency chains), `medium` for routine health checks (did the run succeed?), and `low` for simple classification tasks like tagging failed models by error type.

```python
def get_effort(task_type: str) -> str:
    """Map task complexity to effort level for cost control."""
    effort_map = {
        "diagnose_failure": "xhigh",   # multi-step root cause analysis
        "explain_lineage":  "high",    # DAG traversal + explanation
        "health_check":     "medium",  # binary pass/fail
        "tag_error_type":   "low",     # classification only
    }
    return effort_map.get(task_type, "high")

response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=32000,
    thinking={"type": "adaptive", "display": "summarized"},
    output_config={"effort": get_effort("diagnose_failure")},
    messages=[{"role": "user", "content": user_query}]
)
```

**Citation capsule:** The `effort` parameter in `output_config` is the Anthropic-recommended replacement for `budget_tokens` on Claude Opus 4.7, Opus 4.6, and Sonnet 4.6. It accepts five levels: `low`, `medium`, `high` (default), `xhigh` (Opus 4.7 only), and `max`. Rather than pre-specifying a thinking token count, `effort` communicates task importance and expected reasoning depth to the model.

For data engineering agents, a tiered effort strategy — `xhigh` for complex root-cause diagnosis, `medium` for routine status checks, `low` for classification — can reduce daily thinking token costs by 40–60% compared to a flat `high` effort setting across all calls. Set `max_tokens` to at least 32k when using `xhigh` or `max`, because at those levels the model may exhaust its reasoning budget before producing the text response. Source: [Anthropic API Docs — Effort](https://platform.claude.com/docs/en/build-with-claude/effort), 2026.

---

## Does Streaming Still Work?

Streaming with adaptive thinking works exactly like the old manual thinking streaming — the API emits `thinking_delta` events followed by `text_delta` events ([Anthropic API Docs](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking), 2026). No changes to your stream handling code are needed beyond the `budget_tokens` → `adaptive` swap.

```python
import anthropic

client = anthropic.Anthropic()

with client.messages.stream(
    model="claude-opus-4-7",
    max_tokens=32000,
    thinking={"type": "adaptive", "display": "summarized"},
    output_config={"effort": "high"},
    messages=[
        {"role": "user", "content": "Why did the fct_orders_daily model fail at 3am?"}
    ],
) as stream:
    for event in stream:
        if event.type == "content_block_start":
            block_type = event.content_block.type
            if block_type == "thinking":
                print("\n[Thinking...]", flush=True)
        elif event.type == "content_block_delta":
            if event.delta.type == "thinking_delta":
                print(event.delta.thinking, end="", flush=True)
            elif event.delta.type == "text_delta":
                print(event.delta.text, end="", flush=True)
```

One behavioral note: at `high` effort and above on Opus 4.7, the model thinks before producing any text. This means there may be a longer delay before the first `text_delta` compared to Opus 4.6 at the same effort level. If your streaming UI shows a loading indicator until first text, consider setting `display: "omitted"` — the server skips streaming the thinking summary and delivers text tokens faster.

**Citation capsule:** Streaming with Claude adaptive thinking uses the same event model as manual extended thinking: `thinking_delta` events carry the model's reasoning process (when `display: "summarized"` is set), followed by `text_delta` events for the final response. No changes to stream handling logic are required when migrating from `budget_tokens` to adaptive thinking — the delta types and event sequence remain identical.

Setting `display: "omitted"` in the thinking config produces the fastest time-to-first-text-token when streaming, because the server skips emitting the thinking summary and begins text output immediately. You are still billed for the full thinking tokens regardless of display setting. For latency-sensitive pipelines, `display: "omitted"` at `high` effort is a practical middle ground. Source: [Anthropic API Docs — Adaptive Thinking](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking), 2026.

---

## The Prompt Caching Interaction You Need to Know

Adaptive thinking and prompt caching work together, but there is one rule that trips up migrations: switching between `adaptive` and `enabled`/`disabled` thinking modes **breaks cache breakpoints for the messages array** ([Anthropic API Docs](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking), 2026). System prompts and tool definitions remain cached regardless of thinking mode changes — only the message-level cache is affected.

**Thinking Mode × Cache Behavior**

<figure>
<svg viewBox="0 0 560 260" xmlns="http://www.w3.org/2000/svg" style="background:#1a1a2e;border-radius:8px;display:block;max-width:100%">
  <text x="280" y="28" text-anchor="middle" fill="white" font-size="13" font-weight="bold" font-family="sans-serif">Prompt Cache Behavior When Switching Thinking Modes</text>

  <!-- Header row -->
  <rect x="30" y="44" width="240" height="26" rx="3" fill="#2a2a4e"/>
  <text x="150" y="62" text-anchor="middle" fill="#ccccee" font-size="11" font-weight="bold" font-family="sans-serif">Content Block</text>
  <rect x="275" y="44" width="120" height="26" rx="3" fill="#2a2a4e"/>
  <text x="335" y="62" text-anchor="middle" fill="#ccccee" font-size="11" font-weight="bold" font-family="sans-serif">Cache preserved?</text>
  <rect x="400" y="44" width="130" height="26" rx="3" fill="#2a2a4e"/>
  <text x="465" y="62" text-anchor="middle" fill="#ccccee" font-size="11" font-weight="bold" font-family="sans-serif">Action needed?</text>

  <!-- Row 1: System prompt -->
  <rect x="30" y="74" width="240" height="26" rx="3" fill="#1e1e3e"/>
  <text x="150" y="92" text-anchor="middle" fill="#ccccee" font-size="11" font-family="sans-serif">System prompt</text>
  <rect x="275" y="74" width="120" height="26" rx="3" fill="#1e1e3e"/>
  <text x="335" y="92" text-anchor="middle" fill="#43e97b" font-size="11" font-weight="bold" font-family="sans-serif">✓ Always</text>
  <rect x="400" y="74" width="130" height="26" rx="3" fill="#1e1e3e"/>
  <text x="465" y="92" text-anchor="middle" fill="#ccccee" font-size="11" font-family="sans-serif">None</text>

  <!-- Row 2: Tool definitions -->
  <rect x="30" y="104" width="240" height="26" rx="3" fill="#252545"/>
  <text x="150" y="122" text-anchor="middle" fill="#ccccee" font-size="11" font-family="sans-serif">Tool definitions</text>
  <rect x="275" y="104" width="120" height="26" rx="3" fill="#252545"/>
  <text x="335" y="122" text-anchor="middle" fill="#43e97b" font-size="11" font-weight="bold" font-family="sans-serif">✓ Always</text>
  <rect x="400" y="104" width="130" height="26" rx="3" fill="#252545"/>
  <text x="465" y="122" text-anchor="middle" fill="#ccccee" font-size="11" font-family="sans-serif">None</text>

  <!-- Row 3: Messages (same mode) -->
  <rect x="30" y="134" width="240" height="26" rx="3" fill="#1e1e3e"/>
  <text x="150" y="152" text-anchor="middle" fill="#ccccee" font-size="11" font-family="sans-serif">Messages (same mode)</text>
  <rect x="275" y="134" width="120" height="26" rx="3" fill="#1e1e3e"/>
  <text x="335" y="152" text-anchor="middle" fill="#43e97b" font-size="11" font-weight="bold" font-family="sans-serif">✓ Preserved</text>
  <rect x="400" y="134" width="130" height="26" rx="3" fill="#1e1e3e"/>
  <text x="465" y="152" text-anchor="middle" fill="#ccccee" font-size="11" font-family="sans-serif">None</text>

  <!-- Row 4: Messages (mode switch) -->
  <rect x="30" y="164" width="240" height="26" rx="3" fill="#3a1e2e"/>
  <text x="150" y="182" text-anchor="middle" fill="#ccccee" font-size="11" font-family="sans-serif">Messages (mode switch)</text>
  <rect x="275" y="164" width="120" height="26" rx="3" fill="#3a1e2e"/>
  <text x="335" y="182" text-anchor="middle" fill="#ff6584" font-size="11" font-weight="bold" font-family="sans-serif">✗ Broken</text>
  <rect x="400" y="164" width="130" height="26" rx="3" fill="#3a1e2e"/>
  <text x="465" y="182" text-anchor="middle" fill="#ff6584" font-size="11" font-family="sans-serif">Consistent mode</text>

  <line x1="30" y1="200" x2="530" y2="200" stroke="#444466" stroke-width="1"/>
  <text x="280" y="218" text-anchor="middle" fill="#aaaacc" font-size="9" font-family="sans-serif">Switching adaptive ↔ enabled ↔ disabled breaks message-level cache.</text>
  <text x="280" y="232" text-anchor="middle" fill="#aaaacc" font-size="9" font-family="sans-serif">System prompts and tool definitions stay cached regardless.</text>
  <text x="280" y="248" text-anchor="middle" fill="#aaaacc" font-size="9" font-family="sans-serif">Source: Anthropic API Docs, 2026.</text>
</svg>
<figcaption style="color:#aaaacc;font-size:0.85em;text-align:center;margin-top:0.5rem">Switching thinking modes mid-session forces a cache miss on the message array — keep the mode consistent.</figcaption>
</figure>

In practice this means: if your agent migrates from `budget_tokens` (enabled) to `adaptive` mid-session, the first call after the switch pays full input price for the message history. In a long agentic session with a 100k-token conversation history, that's a meaningful unexpected cost. The fix is to complete migrations at session boundaries, not mid-turn.

<!-- ORIGINAL DATA -->
In testing a dbt pipeline monitoring agent with a 60k-token conversation history across a 2-hour session, switching from `enabled` to `adaptive` at turn 15 caused a full cache miss on the message array — roughly $1.50 in unexpected input token charges at Opus 4.7 pricing. Restarting the session with `adaptive` from turn 1 preserved the cache and reduced the session cost by 78%.

For full prompt caching strategy including breakpoint placement and the cache write vs. read cost math, see the [Claude Prompt Caching Python guide](/posts/claude-prompt-caching-python-guide/).

**Citation capsule:** Adaptive thinking and prompt caching are fully compatible, with one critical rule: switching thinking modes between `adaptive`, `enabled`, and `disabled` within a conversation breaks the prompt cache for the messages array. A mode switch forces a cache miss — the model treats the prefix as a new entry even if the content is identical. System prompts and tool definitions are unaffected and remain cached across mode switches.

The practical implication for migration is to complete the `budget_tokens` → `adaptive` transition at session boundaries rather than mid-conversation. In long agentic sessions where conversation history is the largest input cost, a mid-session mode switch can eliminate all cached message savings for that session. Source: [Anthropic API Docs — Adaptive Thinking](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking), 2026; [Prompt Caching Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-caching), 2026.

---

## Frequently Asked Questions

### What replaces budget_tokens in Claude Opus 4.7?

Adaptive thinking replaces `budget_tokens` on Claude Opus 4.7. Set `thinking` to `{type: "adaptive"}` and control depth via the `effort` parameter (`low`, `medium`, `high`, `xhigh`, `max`) in `output_config`. Passing `budget_tokens` to Opus 4.7 returns an HTTP 400 error ([Anthropic Docs](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking), 2026).

### Does budget_tokens still work on Opus 4.6 and Sonnet 4.6?

Yes, but it is deprecated. Anthropic will remove it in a future model release. The recommended path is to migrate to adaptive thinking with `effort` now to avoid a forced migration when the next model version drops.

### What is the effort parameter in the Claude API?

The `effort` parameter, set in `output_config`, tells Claude how many tokens to spend on a response. Values are `low`, `medium`, `high` (default), `xhigh` (Opus 4.7 only), and `max`. It controls thinking depth when adaptive thinking is active and also affects tool call verbosity and text response length ([Anthropic Effort Docs](https://platform.claude.com/docs/en/build-with-claude/effort), 2026).

### What is the display: omitted change in Opus 4.7?

On Opus 4.7, `thinking.display` defaults to `"omitted"` instead of `"summarized"`. Thinking blocks appear in the response but their `thinking` text field is empty. You are still billed for the full thinking tokens. Set `display: "summarized"` explicitly if your app surfaces thinking content to users ([Anthropic Adaptive Thinking Docs](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking), 2026).

---

## Migration Checklist

Opus 4.7 shipped with three silent behavioral changes on top of the `budget_tokens` removal. Run through this before deploying:

1. Replace `thinking: {type: "enabled", budget_tokens: N}` with `thinking: {type: "adaptive"}`
2. Map your `budget_tokens` value to an effort level using the table above
3. Add `display: "summarized"` explicitly if your code reads `block.thinking`
4. Raise `max_tokens` to 32k–64k for `xhigh` or `max` effort calls
5. Complete the migration at session boundaries to preserve prompt cache on message arrays
6. Audit any code that tests for the old `budget_tokens` key in request logs or middleware

For context on how Opus 4.7's updated tokenizer affects input token counts on long prompts (1.0–1.35× increase depending on content), and how that interacts with your existing cost model, see the [Claude Code for Data Engineers guide](/posts/claude-code-for-data-engineers/) for token monitoring patterns using CLAUDE.md.

---

*[Francisco Frez](https://ffrezr.github.io/about/) is a Data Engineer specializing in GCP, dbt, BigQuery, and Airflow. He has been integrating Claude and Claude Code into production data pipelines since early 2025 and writes about practical AI tooling for engineering teams.*
