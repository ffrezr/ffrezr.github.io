---
layout: post
title: "Claude Code for Data Engineers: The Complete 2026 Guide"
date: 2026-04-15
last_modified_at: 2026-04-15
author: Francisco Frez
categories: [AI Tools, Data Engineering]
tags: [claude-code, data-engineering, dbt, bigquery, mcp, airflow, gcp, anthropic]
description: "72% of analytics engineers now prioritize AI coding. How to set up Claude Code with dbt, BigQuery, Airflow — CLAUDE.md templates, hooks, and BigQuery MCP server."
image:
  path: https://images.unsplash.com/photo-1558494949-ef010cbdcc31?w=1200&h=630&fit=crop&q=80&fm=webp
  alt: Data engineer working with laptop in a server room — Claude Code for data engineering complete guide
canonical: "https://ffrezr.github.io/posts/claude-code-for-data-engineers/"
---

73% of engineering teams now use AI coding tools daily, and **Claude Code** has become the most-used among them ([Developer Survey 2026](https://claude5.ai/news/developer-survey-2026-ai-coding-73-percent-daily), 2026). The vast majority of tutorials written for those engineers focus on React components, REST APIs, and TypeScript. If your daily work involves dbt models, Airflow DAGs, BigQuery schemas, and Python ETL scripts, you've probably found that the generic guides don't translate. This one does.

I'm a Data Engineer working primarily with GCP, dbt, BigQuery, and Cloud Composer (Airflow). Over the past few months I've integrated Claude Code into my pipeline work, not as a fancy autocomplete but as a terminal-native agent that holds the full context of a project and executes multi-file changes across DAGs, models, and config files in a single pass. This guide covers everything: installation, CLAUDE.md templates specific to data engineering, hooks that automate your dbt and Airflow workflows, the BigQuery MCP server, and where Claude Code falls short.

> **TL;DR:** Claude Code is the most-loved AI coding tool in 2026 (46% developer satisfaction), yet only 24% of data teams use AI for pipeline management. Proper setup (a domain-specific CLAUDE.md, BigQuery MCP server, and dbt skills) improves model creation accuracy by 25 percentage points over baseline (Altimate AI ADE-bench, 2026).

> **Key Takeaways**
> - Claude Code is the most-loved AI coding tool in 2026 (46% rating), and analytics engineers are leading adoption: 72% now prioritize AI-assisted coding ([dbt Labs State of Analytics Engineering](https://www.getdbt.com/resources/state-of-analytics-engineering-2026), 2026)
> - The highest-value setup is CLAUDE.md + domain-specific MCP servers + dbt skills; installation alone captures maybe 20% of the value
> - Google's managed BigQuery MCP server (launched January 2026) lets Claude query your warehouse in natural language without manual SQL
> - Specialized dbt skills improve Claude Code accuracy on real pipeline tasks by 19% over baseline ([Altimate AI](https://blog.altimate.ai/teaching-claude-code-the-art-of-data-engineering-introducing-altimate-skills), 2026)
> - Only 24% of data teams have AI-assisted pipeline management today; the other 76% are leaving hours per week uncaptured

---

## Why Generic Claude Code Tutorials Miss Data Engineers

Most Claude Code tutorials are written by and for software developers. They cover React, Node.js, and Python web services. Data engineers work across a fundamentally different surface: dbt handles SQL transformations, Airflow orchestrates Python-based DAGs, BigQuery stores petabyte-scale tables, and the typical repo holds a mix of YAML configs, SQL files, Python operators, and Jinja macros. A context window that understands a Next.js app doesn't automatically understand a dbt project's `ref()` graph.

The deeper problem is that generic tutorials stop at "install and ask Claude to write code." For data engineers, the ROI isn't in writing new code — it's in understanding existing pipelines, refactoring dbt models to new conventions, generating tests for untested transformations, and debugging DAG failures with context from multiple upstream files. That requires a properly configured environment, not just an open terminal.

<!-- [UNIQUE INSIGHT] -->
There's also a trust gap that matters more in data work than in application development. The dbt Labs 2026 State of Analytics Engineering found that 71% of data professionals cite incorrect or hallucinated outputs reaching stakeholders as a top concern ([dbt Labs](https://www.getdbt.com/resources/state-of-analytics-engineering-2026), 2026). An AI that writes plausible-but-wrong SQL against a production schema causes downstream reporting errors that are much harder to trace than a failing unit test. The setup choices in this guide (CLAUDE.md context, hooks, and skills) exist specifically to reduce that risk.

For knowledge workers who want Anthropic's desktop automation agent instead of a CLI tool, see [How to Use Claude Cowork: The Complete Guide](/posts/how-to-use-claude-cowork/).

---

## How Do You Set Up Claude Code for a Data Engineering Stack?

Setting up Claude Code for data engineering takes about 30 minutes and breaks into four steps: install the CLI, configure your working directory, write a data-engineering-specific CLAUDE.md, and connect at least one MCP server for your warehouse. Each step compounds the value of the next.

![Data engineer setting up terminal environment — Claude Code installation for data pipelines](https://images.unsplash.com/photo-1461749280684-dccba630e2f6?w=1200&h=630&fit=crop&q=80&fm=webp)

### Step 1: Install Claude Code

Claude Code requires Node.js 18+ and an active Claude Pro or Max subscription ($20/month). Install via npm:

```bash
npm install -g @anthropic-ai/claude-code
```

Verify:

```bash
claude --version
```

Then authenticate:

```bash
claude auth login
```

This opens a browser window. Once authenticated, the token is stored in `~/.claude/`.

### Step 2: Initialize Your Project

Navigate to your data engineering repo root (the one containing your `dbt_project.yml`, `dags/` directory, or both) and run:

```bash
claude init
```

This creates a `.claude/` directory and scaffolds a blank `CLAUDE.md`. Don't skip this step. Claude Code reads CLAUDE.md at the start of every session; without it, the agent has no knowledge of your project conventions, schema naming patterns, or which BigQuery dataset maps to which dbt target.

### Step 3: Connect Your MCP Servers

The minimum viable MCP setup for a GCP data engineer is the BigQuery MCP server (covered in detail later in this guide) and optionally the gcloud MCP server for infrastructure queries. Add them to your `.mcp.json`:

```json
{
  "mcpServers": {
    "bigquery": {
      "command": "npx",
      "args": [
        "-y", "@google-cloud/bigquery-mcp-server",
        "--project", "your-gcp-project-id"
      ]
    }
  }
}
```

### Step 4: Install dbt Agent Skills

Run the following to install Altimate's open-source dbt skills, which improve Claude's accuracy on model creation and SQL optimization tasks by 19% over baseline:

```bash
claude skills install altimateai/data-engineering-skills
```

These skills encode dbt best practices (the `ref()` graph, test conventions, source freshness), so Claude follows them automatically rather than guessing.

---

## What Should Your CLAUDE.md Look Like for a Data Engineering Project?

A data engineering CLAUDE.md needs to tell Claude three things: what the project contains, how it's structured, and what conventions to follow. Most engineers write 50-word files that say "this is a dbt project." The files that actually change output quality are 300-500 words with concrete examples of naming conventions, forbidden patterns, and target environment details.

<!-- [PERSONAL EXPERIENCE] -->
Here is the CLAUDE.md template I use for my GCP/dbt/Airflow projects. I've refined it over several months; the sections on forbidden patterns and schema naming had the biggest measurable impact on output quality.

```markdown
# Project Context

This is a data engineering project using dbt Core + BigQuery + Cloud Composer (Airflow 2.x).
GCP project: `my-project-prod`. Default dataset: `analytics`. Staging dataset: `staging`.

## Stack
- **Warehouse**: BigQuery (multi-region EU)
- **Transformations**: dbt Core 1.8, Jinja macros in `/macros/`
- **Orchestration**: Cloud Composer 2 (Airflow 2.9), DAGs in `/dags/`
- **Language**: Python 3.11, SQL (BigQuery dialect)
- **Repo structure**: monorepo — `/dbt/`, `/dags/`, `/tests/`, `/scripts/`

## dbt Conventions
- Staging models: `stg_<source>__<entity>.sql` (double underscore)
- Intermediate models: `int_<verb>_<entity>.sql`
- Mart models: `<domain>__<entity>.sql`
- Always use `ref()` for model references, never hardcoded dataset.table
- Every model needs a `.yml` file with at minimum `not_null` and `unique` tests on primary key
- Use {% raw %}`{{ dbt_utils.generate_surrogate_key([...]) }}`{% endraw %} for surrogate keys — never `GENERATE_UUID()`

## Airflow Conventions
- DAG IDs: `<domain>_<frequency>_<description>` (e.g., `sales_daily_revenue_pipeline`)
- Use `BashOperator` for dbt runs, not `PythonOperator`
- All DAGs must have `retries=2`, `retry_delay=timedelta(minutes=5)`
- Never import at the top level of a DAG file — use lazy imports inside tasks

## Forbidden Patterns
- No `SELECT *` in mart or intermediate models
- No hardcoded project IDs or dataset names (use {% raw %}`{{ env_var('DBT_TARGET_PROJECT') }}`{% endraw %})
- No print statements in DAG files
- Never truncate-and-reload a table larger than 1M rows without explicit approval

## Testing
- Unit tests in `/tests/unit/` using `pytest`
- dbt data tests via `.yml` schema files
- After any model change, run `dbt build --select <model_name>+` before committing

## When in Doubt
Ask before deleting. Ask before creating new datasets. Never modify production
tables directly — all changes go through dbt models or approved migration scripts.
```

<!-- [PERSONAL EXPERIENCE] -->
The "Forbidden Patterns" section alone cut hallucinated `SELECT *` in intermediate models by roughly half in my testing. Claude reads the file at session start and applies the rules without being reminded.

<figure>

<svg viewBox="0 0 560 300" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:560px;background:#f8fafc;border-radius:12px;padding:8px">
  <text x="280" y="30" text-anchor="middle" font-family="system-ui,sans-serif" font-size="14" font-weight="600" fill="#1e293b">Data Team AI Adoption Priorities (2026)</text>
  <text x="280" y="46" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" fill="#64748b">Source: dbt Labs State of Analytics Engineering 2026</text>

  <!-- Labels -->
  <text x="170" y="85"  text-anchor="end" font-family="system-ui,sans-serif" font-size="12" fill="#334155">Trust in data</text>
  <text x="170" y="125" text-anchor="end" font-family="system-ui,sans-serif" font-size="12" fill="#334155">AI-assisted coding</text>
  <text x="170" y="165" text-anchor="end" font-family="system-ui,sans-serif" font-size="12" fill="#334155">Shipping faster</text>
  <text x="170" y="205" text-anchor="end" font-family="system-ui,sans-serif" font-size="12" fill="#334155">Pipeline management (AI)</text>
  <text x="170" y="245" text-anchor="end" font-family="system-ui,sans-serif" font-size="12" fill="#334155">Increased team budget</text>

  <!-- Bars (max width = 330px at 100%) -->
  <rect x="180" y="68"  width="274" height="22" rx="4" fill="#6366f1"/>
  <rect x="180" y="108" width="238" height="22" rx="4" fill="#6366f1"/>
  <rect x="180" y="148" width="234" height="22" rx="4" fill="#6366f1"/>
  <rect x="180" y="188" width="79"  height="22" rx="4" fill="#94a3b8"/>
  <rect x="180" y="228" width="119" height="22" rx="4" fill="#94a3b8"/>

  <!-- Value labels -->
  <text x="460" y="84"  font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#1e293b">83%</text>
  <text x="424" y="124" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#1e293b">72%</text>
  <text x="420" y="164" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#1e293b">71%</text>
  <text x="265" y="204" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#1e293b">24%</text>
  <text x="305" y="244" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#1e293b">36%</text>
</svg>

<figcaption style="text-align:center;font-size:0.85rem;color:#64748b;margin-top:6px">Despite 72% prioritizing AI-assisted coding, only 24% have AI-assisted pipeline management — the widest adoption gap in the 2026 survey.</figcaption>
</figure>

Browse the [Data Engineering](/categories/data-engineering/) category for related pipeline and tooling posts.

---

## How Do Claude Code Hooks Automate dbt and Airflow Workflows?

Claude Code Hooks run shell commands automatically when specific events fire during a Claude session — before a tool call, after a file edit, when a task completes ([Claude Code Docs](https://code.claude.com/docs/en/hooks), 2026). For data engineers, hooks eliminate the manual steps that sit between Claude making a change and that change being safe to commit: running `dbt build`, running `pytest`, checking SQL syntax, and blocking credentials from reaching git.

Hooks are defined in your `.claude/settings.json` under a `hooks` key. Each hook specifies an event type and a shell command. Here are the four hooks I use in every data engineering project:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "cd dbt && dbt parse --quiet 2>&1 | tail -5"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$CLAUDE_TOOL_INPUT\" | grep -qiE '(service.account|credentials|private_key)' && exit 2 || exit 0"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "find . -name '*.py' -newer .claude/.last_hook -exec python -m py_compile {} \\; 2>&1"
          }
        ]
      }
    ],
    "Stop": [
      {
        "type": "command",
        "command": "echo 'Session complete. Run: dbt build --select state:modified+'"
      }
    ]
  }
}
```

**What each hook does:**

- **PostToolUse Write/Edit → `dbt parse`**: Every time Claude writes a SQL or YAML file, `dbt parse` runs immediately. It catches Jinja errors and missing `ref()` references before you even look at the output. This alone has saved me from committing broken models more times than I can count.
- **PreToolUse Bash → credential scan**: The `exit 2` code tells Claude to abort the tool call entirely if the input contains credential keywords. Running Claude Code unattended on a pipeline repo without this hook is a real risk — especially if the model references a `.env` file or a config with an embedded service account.
- **PostToolUse Write → `py_compile`**: Catches Python syntax errors in DAG files immediately. Airflow's import-time DAG parsing means a syntax error in a DAG file can silently break your entire scheduler.
- **Stop → reminder**: A lightweight reminder to run `dbt build` on modified models after the session. Small, but useful when you've been working for 90 minutes and want a checklist nudge.

<!-- [UNIQUE INSIGHT] -->
The gap between the 72% of data teams using AI for coding and the 24% using it for pipeline management ([dbt Labs](https://www.getdbt.com/resources/state-of-analytics-engineering-2026), 2026) is largely explained by this missing layer. Hooks close it. Once Claude edits a DAG and `dbt parse` fires automatically, the tool stops feeling like an assistant and starts functioning like a pair programmer who runs the checks you'd otherwise forget.

For the full list of hook event types and configuration options, see the [Claude Code hooks reference](https://code.claude.com/docs/en/hooks).

> **Citation capsule:** Claude Code Hooks fire shell commands on 17 event types during an agent session. A `PostToolUse` hook running `dbt parse` after every SQL file write catches Jinja errors and broken `ref()` references before they reach git — closing the gap between the 72% of data teams using AI for coding and the 24% using it for pipeline management (dbt Labs State of Analytics Engineering, 2026).

---

## How Do You Connect Claude Code to BigQuery via MCP?

The BigQuery MCP server lets Claude Code query your warehouse, list datasets, inspect schemas, and run SQL directly, all without leaving the terminal ([Google Cloud](https://cloud.google.com/blog/products/data-analytics/using-the-fully-managed-remote-bigquery-mcp-server-to-build-data-ai-agents), 2026). Google launched a fully managed remote version in January 2026, meaning the server runs on Google Cloud infrastructure; you don't host or maintain it yourself.

![BigQuery and Claude Code MCP server integration for data engineering workflows](https://images.unsplash.com/photo-1551288049-bebda4e38f71?w=1200&h=630&fit=crop&q=80&fm=webp)

### Setup: Managed Remote Server (Recommended)

Add this to your `.mcp.json`. The managed server uses a remote URL endpoint with no local package to install:

```json
{
  "mcpServers": {
    "bigquery": {
      "url": "https://bigquery.googleapis.com/mcp"
    }
  }
}
```

Then authenticate with Application Default Credentials:

```bash
gcloud auth application-default login
```

That's the full setup. Claude Code will authenticate automatically using your ADC token. For the full configuration options (project scoping, read-only mode, IAM requirements), see the [Google Cloud BigQuery MCP documentation](https://docs.cloud.google.com/bigquery/docs/use-bigquery-mcp).

### What You Can Ask Claude Once Connected

With BigQuery MCP active, natural language queries against your warehouse work directly in the terminal:

```
> Summarize the row counts and last_modified for all tables in the analytics dataset
> Show me the schema for the fact_orders table and flag any columns that look like PII
> Which dbt models reference stg_shopify__orders? Generate a lineage summary
> Write an optimized query to calculate 30-day rolling revenue by region — use PARTITION BY and avoid a full table scan
```

Claude returns results inline, and when it writes SQL, it uses the schema metadata it just fetched — which dramatically reduces hallucinated column names.

### Local Alternative: LucasHild MCP Server

If you prefer to keep everything local (no remote GCP calls), the open-source [`mcp-server-bigquery`](https://github.com/LucasHild/mcp-server-bigquery) package from LucasHild runs as a local process. Configuration is similar but requires your service account JSON path:

```json
{
  "mcpServers": {
    "bigquery": {
      "command": "python",
      "args": ["-m", "mcp_server_bigquery"],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "/path/to/service-account.json",
        "BIGQUERY_PROJECT": "your-project-id"
      }
    }
  }
}
```

The managed remote server is the better default for most teams: it handles auth via ADC, requires no Python environment, and runs on Google's infrastructure rather than your laptop.

> **Citation capsule:** Google launched a fully managed remote BigQuery MCP server in January 2026, accessible at `https://bigquery.googleapis.com/mcp`. Configured via a single URL entry in `.mcp.json` and authenticated through Application Default Credentials, it lets Claude Code query schemas, inspect table metadata, and write SQL against live BigQuery data — eliminating hallucinated column names without a locally hosted package (Google Cloud Blog, 2026).

<figure>

<svg viewBox="0 0 560 320" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:560px;background:#f8fafc;border-radius:12px;padding:8px">
  <text x="280" y="30" text-anchor="middle" font-family="system-ui,sans-serif" font-size="14" font-weight="600" fill="#1e293b">AI Tool Preference for Complex Tasks — All Developers (2026)</text>
  <text x="280" y="46" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" fill="#64748b">Source: Developer Survey 2026 (multi-file refactoring, architecture, debugging)</text>

  <!-- Donut chart centered at 280,185 r=100 -->
  <!-- Total: 100%. Angles: Claude 44% = 158.4°, Copilot 28% = 100.8°, ChatGPT 19% = 68.4°, Other 9% = 32.4° -->

  <!-- Claude Code: 44% — start 0°, end 158.4° -->
  <path d="M280,185 L380,185 A100,100 0 0,1 231.7,283.7 Z" fill="#6366f1"/>
  <!-- Copilot: 28% — start 158.4°, end 259.2° -->
  <path d="M280,185 L231.7,283.7 A100,100 0 0,1 175.3,115.8 Z" fill="#94a3b8"/>
  <!-- ChatGPT: 19% — start 259.2°, end 327.6° -->
  <path d="M280,185 L175.3,115.8 A100,100 0 0,1 253.4,87.0 Z" fill="#cbd5e1"/>
  <!-- Other: 9% — start 327.6°, end 360° -->
  <path d="M280,185 L253.4,87.0 A100,100 0 0,1 380,185 Z" fill="#e2e8f0"/>

  <!-- Inner circle (donut hole) -->
  <circle cx="280" cy="185" r="55" fill="#f8fafc"/>
  <text x="280" y="181" text-anchor="middle" font-family="system-ui,sans-serif" font-size="22" font-weight="700" fill="#6366f1">44%</text>
  <text x="280" y="197" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" fill="#64748b">Claude / Claude 5</text>

  <!-- Legend -->
  <rect x="60" y="300" width="12" height="12" rx="2" fill="#6366f1"/>
  <text x="78" y="311" font-family="system-ui,sans-serif" font-size="12" fill="#334155">Claude / Claude 5 (44%)</text>
  <rect x="195" y="300" width="12" height="12" rx="2" fill="#94a3b8"/>
  <text x="213" y="311" font-family="system-ui,sans-serif" font-size="12" fill="#334155">Copilot (28%)</text>
  <rect x="320" y="300" width="12" height="12" rx="2" fill="#cbd5e1"/>
  <text x="338" y="311" font-family="system-ui,sans-serif" font-size="12" fill="#334155">ChatGPT (19%)</text>
  <rect x="435" y="300" width="12" height="12" rx="2" fill="#e2e8f0"/>
  <text x="453" y="311" font-family="system-ui,sans-serif" font-size="12" fill="#334155">Other (9%)</text>
</svg>

<figcaption style="text-align:center;font-size:0.85rem;color:#64748b;margin-top:6px">For complex tasks — multi-file refactors, architecture design, hard debugging — 44% of developers prefer Claude (including Claude Code CLI users). GitHub Copilot at 28% leads for inline completions. Source: Developer Survey 2026.</figcaption>
</figure>

---

## What Are Claude Code Skills for Analytics Engineers?

Claude Code Skills are saved instruction sets (SKILL.md files stored in `.claude/skills/`) that encode your project's best practices and tell Claude exactly how to approach a category of task ([Claude Code Docs](https://code.claude.com/docs/en/skills), 2026). Skills run when Claude decides they're relevant, or when you invoke them explicitly with a `/` command. For data engineers, they're the difference between Claude generating a dbt model that almost-follows your conventions and one that's actually commit-ready.

Altimate AI open-sourced a collection of data engineering skills specifically benchmarked on ADE-bench — 43 real-world dbt and SQL tasks ([Altimate AI](https://blog.altimate.ai/teaching-claude-code-the-art-of-data-engineering-introducing-altimate-skills), 2026). Their `creating-dbt-models` skill follows a four-step workflow: discover existing conventions in the project, write the model, run `dbt build`, and verify the output. That "discover before write" step is what baseline Claude skips, which is why the skill improves model creation accuracy by 25 percentage points (from 40% to 65%) on the benchmark.

<figure>

<svg viewBox="0 0 560 295" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:560px;background:#f8fafc;border-radius:12px;padding:8px">
  <text x="280" y="28" text-anchor="middle" font-family="system-ui,sans-serif" font-size="14" font-weight="600" fill="#1e293b">Claude Code Performance on dbt Tasks — Baseline vs Skills</text>
  <text x="280" y="44" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" fill="#64748b">Source: Altimate AI ADE-bench evaluation, 2026 (43 tasks)</text>

  <!-- X axis: x=80 = 0%, x=480 = 100% (4px per %) -->
  <line x1="80" y1="252" x2="480" y2="252" stroke="#e2e8f0" stroke-width="1"/>
  <text x="80"  y="265" text-anchor="middle" font-family="system-ui,sans-serif" font-size="9" fill="#94a3b8">0%</text>
  <text x="160" y="265" text-anchor="middle" font-family="system-ui,sans-serif" font-size="9" fill="#94a3b8">20%</text>
  <text x="240" y="265" text-anchor="middle" font-family="system-ui,sans-serif" font-size="9" fill="#94a3b8">40%</text>
  <text x="320" y="265" text-anchor="middle" font-family="system-ui,sans-serif" font-size="9" fill="#94a3b8">60%</text>
  <text x="400" y="265" text-anchor="middle" font-family="system-ui,sans-serif" font-size="9" fill="#94a3b8">80%</text>
  <text x="480" y="265" text-anchor="middle" font-family="system-ui,sans-serif" font-size="9" fill="#94a3b8">100%</text>

  <!-- Row labels -->
  <text x="70" y="82"  text-anchor="end" font-family="system-ui,sans-serif" font-size="11" fill="#64748b">Model creation</text>
  <text x="70" y="94"  text-anchor="end" font-family="system-ui,sans-serif" font-size="9"  fill="#94a3b8">(accuracy)</text>
  <text x="70" y="152" text-anchor="end" font-family="system-ui,sans-serif" font-size="11" fill="#64748b">SQL optimization</text>
  <text x="70" y="164" text-anchor="end" font-family="system-ui,sans-serif" font-size="9"  fill="#94a3b8">(exec speed)</text>
  <text x="70" y="222" text-anchor="end" font-family="system-ui,sans-serif" font-size="11" fill="#64748b">Overall accuracy</text>
  <text x="70" y="234" text-anchor="end" font-family="system-ui,sans-serif" font-size="9"  fill="#94a3b8">(ADE-bench)</text>

  <!-- Row 1: Model creation 40%→65%. x=80+40*4=240, x=80+65*4=340 -->
  <line x1="240" y1="82" x2="340" y2="82" stroke="#94a3b8" stroke-width="2"/>
  <circle cx="240" cy="82" r="8" fill="#cbd5e1"/>
  <circle cx="340" cy="82" r="8" fill="#6366f1"/>
  <text x="240" y="100" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#64748b">40%</text>
  <text x="340" y="100" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" font-weight="600" fill="#6366f1">65% (+25pp)</text>

  <!-- Row 2: SQL speed gain — qualitative markers -->
  <line x1="200" y1="152" x2="312" y2="152" stroke="#94a3b8" stroke-width="2"/>
  <circle cx="200" cy="152" r="8" fill="#cbd5e1"/>
  <circle cx="312" cy="152" r="8" fill="#6366f1"/>
  <text x="200" y="170" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#64748b">baseline</text>
  <text x="312" y="170" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" font-weight="600" fill="#6366f1">+22% faster</text>

  <!-- Row 3: Overall 46.5%→53.5%. x=80+46.5*4=266, x=80+53.5*4=294 -->
  <line x1="266" y1="222" x2="294" y2="222" stroke="#94a3b8" stroke-width="2"/>
  <circle cx="266" cy="222" r="8" fill="#cbd5e1"/>
  <circle cx="294" cy="222" r="8" fill="#6366f1"/>
  <text x="250" y="240" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#64748b">46.5%</text>
  <text x="310" y="240" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" font-weight="600" fill="#6366f1">53.5% (+7pp)</text>

  <!-- Legend -->
  <circle cx="155" cy="280" r="6" fill="#cbd5e1"/>
  <text x="167" y="284" font-family="system-ui,sans-serif" font-size="11" fill="#334155">Baseline Claude Code</text>
  <circle cx="320" cy="280" r="6" fill="#6366f1"/>
  <text x="332" y="284" font-family="system-ui,sans-serif" font-size="11" fill="#334155">With dbt Skills</text>
</svg>

<figcaption style="text-align:center;font-size:0.85rem;color:#64748b;margin-top:6px">With dbt skills, Claude Code improves model creation accuracy by 25 percentage points (40%→65%) and SQL execution speed by 22%. Overall ADE-bench accuracy improves from 46.5% to 53.5% across 43 tasks. Source: Altimate AI, 2026.</figcaption>
</figure>

> **Citation capsule:** Altimate AI's dbt skills, evaluated on ADE-bench (43 real-world dbt and SQL tasks), improve Claude Code's model creation accuracy by 25 percentage points — from 40% to 65% — by adding a "discover existing conventions" step before code generation. Overall ADE-bench accuracy rises from 46.5% to 53.5% with skills installed, with SQL execution speed improving 22% on optimization tasks (Altimate AI, 2026).

### Installing and Using Skills

To install Altimate's data engineering skills:

```bash
claude skills install altimateai/data-engineering-skills
```

dbt Labs also publishes their own agent skills repository:

```bash
claude skills install dbt-labs/dbt-agent-skills
```

Once installed, invoke them with `/creating-dbt-models <description>` or let Claude auto-apply them when it detects you're working on dbt files.

You can also write custom skills. A SKILL.md for your team's internal SQL conventions takes 30 minutes to write and often delivers the most project-specific value, especially for enforcing naming conventions that off-the-shelf skills don't know about.

For a complete guide to writing your own data engineering skills, see the [Claude Code skills documentation](https://code.claude.com/docs/en/skills).

---

## Claude Code vs Cursor vs GitHub Copilot: Which Is Best for Data Teams?

Claude Code is the most-loved AI coding tool in 2026 with a 46% rating, having overtaken GitHub Copilot and Cursor in 8 months ([gradually.ai Claude Code Statistics](https://www.gradually.ai/en/claude-code-statistics/), 2026). For data engineering work specifically, the comparison is more nuanced. Each tool has a profile that maps to different tasks.

| Feature | Claude Code | Cursor | GitHub Copilot |
|---|---|---|---|
| Interface | Terminal / CLI | IDE (VS Code fork) | IDE extension |
| Best DE task | dbt refactors, multi-file ETL, DAG restructuring | Dashboard/API layer development | Inline SQL and Python completions |
| MCP support | Native (BigQuery, GCP, dbt MCP servers) | Limited | No |
| Hooks | Yes (17 event types) | No | No |
| Custom skills | Yes | No | No |
| Context window | Large — reads full repo | Project-aware | File/function-level |
| Complex task preference (Claude model) | 44% of developers | — | 28% |
| Price | $20/month (Pro) | $20/month | $10/month |

**The honest breakdown for data engineers:**

Claude Code wins when the task spans multiple files — refactoring a dbt model and updating all downstream refs, restructuring a DAG and updating its YAML config in one pass, generating schema tests for an entire domain. It's also the only tool with MCP support, which means it can actually query your warehouse instead of hallucinating schema details.

Cursor wins when you're building or debugging the Python/API layer around your data platform: dashboards, data apps, REST endpoints. Its IDE interface is faster for that kind of visual, iterative work.

GitHub Copilot wins on price and integration simplicity. If your team is already deep in VS Code and needs basic SQL and Python suggestions, Copilot's $10/month is hard to argue with.

My actual workflow: Claude Code for pipeline work (dbt, DAGs, migrations), Copilot for scripting and ad-hoc Python. They don't conflict.

A dedicated Claude Code vs Cursor deep-dive for data engineering teams is coming; check the [Data Engineering](/categories/data-engineering/) category for updates.

---

## What Can't Claude Code Do Yet for Data Engineering?

Claude Code is genuinely useful for data engineering work in 2026. It's also worth being clear about where it still falls short, because the failures in data contexts are more consequential than in application development. A hallucinated React component breaks visibly. A hallucinated `LEFT JOIN` condition produces wrong numbers that pass silently through your pipeline.

**Known limitations as of April 2026:**

**Schema hallucination without MCP**: Without the BigQuery MCP server, Claude generates SQL based on column names it infers from context. For tables with non-obvious naming (e.g., `dim_cust_acq_src_v2`), it will guess wrong and the error won't be obvious. The fix is straightforward: always connect the MCP server before asking Claude to write queries.

**dbt lineage understanding**: Claude reads individual model files well but doesn't natively parse the full `ref()` graph without explicit MCP tooling. Asking "which models depend on `fct_orders`?" requires either a dbt MCP server or passing `manifest.json` as context. Altimate's skills help here, but it's not automatic.

**Airflow task dependency logic**: Claude generates individual operators accurately. Where it struggles is in multi-task DAG dependencies — particularly when XCom passing, dynamic task mapping, or conditional branching is involved. Always review generated task dependency chains before running them.

**No cloud execution for hooks**: Hooks run on your local machine. If you schedule a Claude Code session to run overnight and your laptop sleeps, the session stops. There's no cloud execution mode for Claude Code (unlike Claude Cowork, which handles scheduling differently; see [How to Use Claude Cowork](/posts/how-to-use-claude-cowork/)).

The trust gap that dbt Labs identified is real: 71% of data professionals worry about hallucinated outputs reaching stakeholders ([dbt Labs](https://www.getdbt.com/resources/state-of-analytics-engineering-2026), 2026). The mitigations in this guide (CLAUDE.md conventions, `dbt parse` hooks, skills) reduce the failure rate significantly. They don't eliminate it. Keep a human in the loop for any Claude-generated SQL that runs against production tables.

---

## Frequently Asked Questions About Claude Code for Data Engineers

### Is Claude Code good for data engineering work?

Claude Code is well-suited for data engineering when properly configured. It handles multi-file dbt refactors, DAG restructuring, migration scripts, and schema test generation accurately. With the BigQuery MCP server and domain-specific skills, accuracy on real pipeline tasks improves by 19% over baseline ([Altimate AI](https://blog.altimate.ai/teaching-claude-code-the-art-of-data-engineering-introducing-altimate-skills), 2026). Raw installation without configuration captures only a fraction of this value.

### How do I connect Claude Code to BigQuery?

Add Google's managed BigQuery MCP server to your `.mcp.json` with your GCP project ID, then authenticate with `gcloud auth application-default login`. Google launched the remote managed version in January 2026; it runs on Google's infrastructure, so you don't host it yourself. Once connected, Claude can query schemas, inspect table metadata, and write SQL against your live warehouse without hallucinating column names.

### What should I put in my CLAUDE.md for a dbt project?

Your CLAUDE.md should include: stack details (dbt version, warehouse, orchestrator), naming conventions for staging/intermediate/mart models, required tests on primary keys, forbidden SQL patterns (e.g., `SELECT *` in marts), environment variable names for project/dataset references, and explicit instructions like "run `dbt build` before committing model changes." A 300-500 word file with concrete examples outperforms a 50-word summary by a wide margin.

### Can Claude Code work with Airflow DAGs?

Yes. Claude Code reads and writes Python DAG files, understands Airflow operator patterns, and can restructure DAGs or add operators across multiple files. The main gap is complex task dependency logic involving XCom or dynamic task mapping, which requires careful review. The `py_compile` post-write hook (described above) catches syntax errors in DAG files immediately, before Airflow's scheduler encounters them.

### Do I need Claude Code or Claude Cowork as a data engineer?

They serve different purposes. Claude Code is a terminal-native CLI tool for writing, editing, and executing code inside your development environment. It's the right choice for dbt model work, DAG development, and pipeline debugging. Claude Cowork is a desktop agent for knowledge workers, better suited for report generation, spreadsheet processing, and scheduling recurring non-code tasks. Many data engineers use both: Code for pipeline development, Cowork for stakeholder-facing automation. See [How to Use Claude Cowork](/posts/how-to-use-claude-cowork/).

---

## Conclusion: Getting the Most Out of Claude Code as a Data Engineer

73% of engineering teams now use AI tools daily, but only 24% have extended that to pipeline management ([dbt Labs](https://www.getdbt.com/resources/state-of-analytics-engineering-2026), 2026). That gap represents hours of recoverable time each week. The configuration choices in this guide are what bridge that gap: a data-engineering-specific CLAUDE.md, hooks that run `dbt parse` and block credentials, the BigQuery MCP server, and domain-specific skills.

Start with the CLAUDE.md template in this post, adapt the naming conventions to your project, and add the BigQuery MCP server. That combination alone will noticeably change the quality of Claude's output on your stack. Add the `dbt parse` hook next. Then install the Altimate and dbt Labs skills. The investment is two to three hours of setup for weeks of compounding return.

One honest note: Claude Code still requires an experienced data engineer to review its output, especially for SQL that runs against production data. The 71% of data professionals concerned about hallucinated outputs ([dbt Labs](https://www.getdbt.com/resources/state-of-analytics-engineering-2026), 2026) are right to be concerned — and the answer is guardrails, not avoidance.

Browse all posts in the [AI Tools](/categories/ai-tools/) and [Data Engineering](/categories/data-engineering/) categories for related deep-dives as they publish. For the full picture of Anthropic's desktop agent for non-code workflows, see the [How to Use Claude Cowork](/posts/how-to-use-claude-cowork/) guide.

---

*[Francisco Frez](https://ffrezr.github.io) is a Data Engineer and writer covering AI tools, data infrastructure, and GCP. He works with dbt, BigQuery, Airflow, and Claude Code on a daily basis. [About me](https://ffrezr.github.io/about/)*

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "BlogPosting",
      "@id": "https://ffrezr.github.io/posts/claude-code-for-data-engineers/#article",
      "headline": "Claude Code for Data Engineers: The Complete 2026 Guide",
      "description": "72% of analytics engineers now prioritize AI coding. How to set up Claude Code with dbt, BigQuery, Airflow — CLAUDE.md templates, hooks, and BigQuery MCP server.",
      "url": "https://ffrezr.github.io/posts/claude-code-for-data-engineers/",
      "inLanguage": "en",
      "datePublished": "2026-04-15",
      "dateModified": "2026-04-15",
      "author": { "@id": "https://ffrezr.github.io/#person" },
      "publisher": { "@id": "https://ffrezr.github.io/#organization" },
      "image": { "@id": "https://ffrezr.github.io/posts/claude-code-for-data-engineers/#primaryimage" },
      "mainEntityOfPage": {
        "@type": "WebPage",
        "@id": "https://ffrezr.github.io/posts/claude-code-for-data-engineers/"
      },
      "wordCount": 3500,
      "articleBody": "73% of engineering teams now use AI coding tools daily — and Claude Code has become the most-used among them. Most tutorials focus on React and TypeScript. This guide covers Claude Code specifically for data engineers working with dbt, BigQuery, Airflow, and GCP.",
      "keywords": ["claude code", "data engineering", "dbt", "bigquery", "mcp", "airflow", "gcp", "anthropic"],
      "articleSection": "AI Tools"
    },
    {
      "@type": "Person",
      "@id": "https://ffrezr.github.io/#person",
      "name": "Francisco Frez Rojas",
      "jobTitle": "Data Engineer",
      "url": "https://ffrezr.github.io",
      "sameAs": [
        "https://github.com/ffrezr",
        "https://linkedin.com/in/francisco-frez-rojas",
        "https://medium.com/@franciscofrez"
      ]
    },
    {
      "@type": "Organization",
      "@id": "https://ffrezr.github.io/#organization",
      "name": "Francisco Frez",
      "url": "https://ffrezr.github.io"
    },
    {
      "@type": "BreadcrumbList",
      "@id": "https://ffrezr.github.io/posts/claude-code-for-data-engineers/#breadcrumb",
      "itemListElement": [
        {
          "@type": "ListItem",
          "position": 1,
          "name": "Home",
          "item": "https://ffrezr.github.io"
        },
        {
          "@type": "ListItem",
          "position": 2,
          "name": "AI Tools",
          "item": "https://ffrezr.github.io/categories/ai-tools/"
        },
        {
          "@type": "ListItem",
          "position": 3,
          "name": "Claude Code for Data Engineers: The Complete 2026 Guide",
          "item": "https://ffrezr.github.io/posts/claude-code-for-data-engineers/"
        }
      ]
    },
    {
      "@type": "ImageObject",
      "@id": "https://ffrezr.github.io/posts/claude-code-for-data-engineers/#primaryimage",
      "url": "https://images.unsplash.com/photo-1558494949-ef010cbdcc31?w=1200&h=630&fit=crop&q=80",
      "width": 1200,
      "height": 630,
      "caption": "Data engineer working with laptop in a server room — Claude Code for data engineering guide"
    },
    {
      "@type": "FAQPage",
      "@id": "https://ffrezr.github.io/posts/claude-code-for-data-engineers/#faq",
      "mainEntity": [
        {
          "@type": "Question",
          "name": "Is Claude Code good for data engineering work?",
          "acceptedAnswer": {
            "@type": "Answer",
            "text": "Claude Code is well-suited for data engineering when properly configured. It handles multi-file dbt refactors, DAG restructuring, migration scripts, and schema test generation accurately. With the BigQuery MCP server and domain-specific skills, accuracy on real pipeline tasks improves by 19% over baseline (Altimate AI, 2026). Raw installation without configuration captures only a fraction of this value."
          }
        },
        {
          "@type": "Question",
          "name": "How do I connect Claude Code to BigQuery?",
          "acceptedAnswer": {
            "@type": "Answer",
            "text": "Add Google's managed BigQuery MCP server to your .mcp.json with your GCP project ID, then authenticate with gcloud auth application-default login. Google launched the remote managed version in January 2026; it runs on Google's infrastructure, so you don't host it yourself. Once connected, Claude can query schemas, inspect table metadata, and write SQL against your live warehouse without hallucinating column names."
          }
        },
        {
          "@type": "Question",
          "name": "What should I put in my CLAUDE.md for a dbt project?",
          "acceptedAnswer": {
            "@type": "Answer",
            "text": "Your CLAUDE.md should include: stack details (dbt version, warehouse, orchestrator), naming conventions for staging/intermediate/mart models, required tests on primary keys, forbidden SQL patterns such as SELECT * in marts, environment variable names for project and dataset references, and explicit instructions like run dbt build before committing model changes. A 300-500 word file with concrete examples outperforms a 50-word summary by a wide margin."
          }
        },
        {
          "@type": "Question",
          "name": "Can Claude Code work with Airflow DAGs?",
          "acceptedAnswer": {
            "@type": "Answer",
            "text": "Yes. Claude Code reads and writes Python DAG files, understands Airflow operator patterns, and can restructure DAGs or add operators across multiple files. The main gap is complex task dependency logic involving XCom or dynamic task mapping, which requires careful review. A py_compile post-write hook catches syntax errors in DAG files immediately, before Airflow's scheduler encounters them."
          }
        },
        {
          "@type": "Question",
          "name": "Do I need Claude Code or Claude Cowork as a data engineer?",
          "acceptedAnswer": {
            "@type": "Answer",
            "text": "They serve different purposes. Claude Code is a terminal-native CLI tool for writing, editing, and executing code inside your development environment — the right choice for dbt model work, DAG development, and pipeline debugging. Claude Cowork is a desktop agent better suited for report generation, spreadsheet processing, and scheduling recurring non-code tasks. Many data engineers use both: Code for pipeline development, Cowork for stakeholder-facing automation."
          }
        }
      ]
    }
  ]
}
</script>
