---
layout: post
title: "How to Use Claude Cowork: The Complete Guide (2026)"
date: 2026-04-15
last_modified_at: 2026-04-15
author: Francisco Frez
categories: [AI Tools, Productivity]
tags: [claude, cowork, anthropic, ai-productivity, automation]
description: "Complete guide to Claude Cowork: Anthropic's desktop AI agent. Learn setup, skills, 38+ connectors, and scheduling to automate hours of work each week."
image:
  path: https://ffrezr.github.io/assets/img/posts/how-to-use-claude-cowork/claude-cowork-og.png
  alt: Claude Cowork complete guide 2026 — Anthropic desktop AI agent
canonical: "https://ffrezr.github.io/posts/how-to-use-claude-cowork/"
---

Claude Cowork is Anthropic's autonomous desktop AI agent for knowledge workers, launched in January 2026. It runs inside the Claude Desktop app and handles multi-step tasks on your behalf — from drafting reports to processing spreadsheets and scheduling recurring workflows. You don't need to write code or use a terminal. If you can describe a task, the agent can likely handle it.

> **Key Takeaways**
> - Claude Cowork lives inside Claude Desktop under its own tab, separate from Chat and Code
> - Setup requires three context files and at least one connector to unlock its full potential
> - 92% of companies plan to increase AI investments in the next three years, yet only 1% report reaching AI maturity ([McKinsey Superagency, Jan 2025](https://www.mckinsey.com/capabilities/tech-and-ai/our-insights/superagency-in-the-workplace-empowering-people-to-unlock-ais-full-potential-at-work)) — Cowork targets that gap
> - Computer Use (browser automation) is available on both macOS and Windows (Windows support added April 3, 2026) for Pro and Max users, still in research preview
> - Scheduled tasks need your machine to be awake and running

---

## What Is Claude Cowork, and How Does It Differ from Claude Chat?

Claude Cowork is a desktop agent, not a chatbot. While 71% of organizations now regularly deploy generative AI ([McKinsey State of AI, 2025](https://www.mckinsey.com/capabilities/quantumblack/our-insights/the-state-of-ai)), most deployments rely on simple chat interfaces that require the user to do the follow-up work. Cowork changes that by executing multi-step tasks autonomously: reading your local files, calling connected apps, and returning finished output.

The Claude Desktop app has three tabs: Chat, Cowork, and Code.

![Claude Cowork main interface inside the Claude Desktop app showing the task input prompt and recent task history](/assets/img/posts/how-to-use-claude-cowork/claude-cowork-screenshot.png)
_The Cowork tab inside Claude Desktop — task input, recents, and Dispatch entry point._

- **Chat** is for back-and-forth conversation. You ask, it answers.
- **Cowork** is for knowledge workers who want results, not instructions.
- **Code** is a CLI-style interface for developers and technical users.

Think of Chat as a consultant you can question. Cowork, by contrast, is the assistant who actually does the work. For a broader look at tools in this space, see the [AI Tools](/categories/ai-tools/) category.

### Is Claude Cowork the Same as Claude Code?

No. Claude Code is a command-line tool aimed at software developers. It reads and writes code, runs tests, and integrates into developer workflows. Cowork targets non-technical users: marketers, analysts, HR managers, and operations teams. The two share the same underlying Claude model but serve entirely different audiences and use cases.

---

## How to Set Up Claude Cowork in Under 15 Minutes

Claude Cowork setup takes under 15 minutes: download Claude Desktop, open the Cowork tab, designate a working folder, and connect at least one integration. Cowork will run without any further configuration. The optional — but highly recommended — final step is creating three plain-text context files (about-me, brand-voice, working-preferences). Most users skip them, and that single omission is the most common reason the agent feels generic rather than personalized.

### Step-by-Step Setup

1. Download Claude Desktop from anthropic.com (Mac or Windows)
2. Open the app and click the **Cowork** tab
3. Designate a working folder on your local machine
4. Connect at least one integration via the **+** button
5. *(Recommended)* Create three core context files so the agent knows who you are — see below. Skipping this step is optional but is the main reason Cowork feels generic rather than personalized
6. *(Optional)* Assign Cowork to a Project for scoped context

![Cowork Projects view in Claude Desktop showing a project assigned for scoped context](/assets/img/posts/how-to-use-claude-cowork/claude-cowork-project.png)
_Assign Cowork to a Project so it loads scoped context files at every session._

### How to Create Context Files

Context files are plain Markdown documents you place in your working folder. The agent reads them at the start of every session. Without them, it has no memory of who you are or how you work.

Create these three files to start:

**`about-me.md`** — Your role, team, company, and the types of tasks you handle most. Keep it to 200–300 words.

**`brand-voice.md`** — Your writing style preferences: tone, vocabulary to avoid, formatting standards, and any brand guidelines. This file is critical if Cowork will write anything you publish.

**`working-preferences.md`** — Preferred output formats (tables vs. prose, Markdown vs. DOCX), how much detail you want, and whether you prefer the agent to ask clarifying questions or make assumptions and flag them at the end.

In practice, the `brand-voice.md` file has the highest return on investment. When I set up my own voice file with three annotated examples of good vs. bad writing, the revision cycle on drafted content dropped by roughly half. A one-paragraph description of tone helps, but concrete examples are dramatically more effective.

---

## What Can Claude Cowork Actually Do?

The agent's capabilities fall into four categories: file processing, app integration, automation, and browser control. The practical range is wider than most users expect. McKinsey estimates generative AI could automate [60–70% of existing work activities](https://www.mckinsey.com/~/media/mckinsey/business%20functions/mckinsey%20digital/our%20insights/the%20economic-potential-of-generative-ai-the-next-productivity-frontier/the-economic-potential-of-generative-ai-the-next-productivity-frontier.pdf), and Cowork is one of the first consumer-facing tools designed to act on that potential rather than just describe it.

### Built-in Skills: Excel, Word, PowerPoint, PDF

Cowork ships with native skills for the four most common office file types. You can ask it to:

- Summarize or restructure a PDF without opening it
- Reformat a spreadsheet, apply formulas, or generate a pivot table
- Draft a slide deck from a bullet list or a data file
- Merge, split, or extract sections from Word documents

All file processing happens in a sandboxed local VM. Your files don't leave your machine unless you explicitly direct Cowork to send them somewhere through a connector.

### Custom Skills via /skill-creator

You can build your own reusable skills using the `/skill-creator` command. A skill is a saved, named instruction set that the agent calls on demand. Useful examples include:

- A "weekly report" skill that pulls data from Snowflake, formats a summary table, and saves a DOCX
- A "prospect research" skill that queries HubSpot, checks LinkedIn context files, and drafts an outreach email

![Cowork Skills panel in Claude Desktop showing the skill-creator skill open for editing](/assets/img/posts/how-to-use-claude-cowork/claude-cowork-skills.png)
_Use `/skill-creator` to define reusable, named instruction sets the agent can call on demand._

Skills that combine two or more connectors tend to deliver the sharpest productivity gains. The step where most users stall is the connector configuration, not the skill logic itself — getting the OAuth handshake right for Google Drive or Salesforce is the friction point, not the task definition.

### Connectors: 38+ Integrations

Cowork connects to 38+ external services via the Model Context Protocol (MCP). MCP SDK downloads reached [approximately 97 million per month](https://www.digitalapplied.com/blog/mcp-97-million-downloads-model-context-protocol-mainstream) in March 2026 — up from roughly 2 million at launch in November 2024 — reflecting rapid adoption as an open integration standard.

![Cowork Connectors Directory showing popular integrations including Google Drive, Gmail, HubSpot, Linear, Zoom, Figma, and Notion](/assets/img/posts/how-to-use-claude-cowork/claude-cowork-connectors-directory.png)
_The in-app Connectors Directory — browse Anthropic & partner integrations, filter by category, and install with one click._

Current connector categories include:

| Category | Examples |
|---|---|
| Productivity | Microsoft 365, Google Drive, Notion |
| CRM and Sales | HubSpot, Salesforce |
| Communication | Slack, Gmail, Zoom |
| Project Management | Jira, Asana, Linear |
| Data Warehouses | Snowflake, BigQuery, Amplitude |

A plugin marketplace launched in February 2026 with department bundles for HR, Sales, Finance, and Engineering. These bundles pre-configure the most relevant connectors and skills for each function.

![Cowork Connectors panel showing Gmail integration with read and write tool permissions](/assets/img/posts/how-to-use-claude-cowork/claude-cowork-connectors.png)
_Each connector exposes granular tool permissions — read-only tools auto-allow, while write/delete tools require approval._

---

## How Do Scheduled Tasks Work in Claude Cowork?

Cowork supports recurring task automation using the `/schedule` command. This lets you run the same task on an hourly, daily, or weekly cadence without triggering it manually each time. As a result, repetitive reporting and data-processing workflows become something you configure once rather than babysit repeatedly.

### Setting Up a Scheduled Task

Type `/schedule` followed by your task description and cadence. Example:

```
/schedule every Monday at 8am: pull last week's Salesforce pipeline report, summarize wins and losses, save to /reports/weekly-pipeline.docx
```

Cowork registers the schedule and runs the task automatically at the specified time.

![Cowork Scheduled tasks view in Claude Desktop with the warning that tasks only run while the computer is awake](/assets/img/posts/how-to-use-claude-cowork/claude-cowork-shceduled-tasks.png)
_The Scheduled view warns up front: tasks only run while your computer is awake — there is no cloud fallback._

### Known Limitation: Machine Must Be Awake

Scheduled tasks skip silently if your machine is asleep or offline at run time. There's no catch-up mechanism. For more on building reliable [productivity workflows](/categories/productivity/), see the Productivity category. If you schedule a daily morning report and your laptop is closed, that day's run won't happen. Until Anthropic adds a cloud execution option, treat schedules as convenience automation rather than mission-critical pipelines.

---

## How Does Computer Use and Browser Automation Work?

Cowork's Computer Use feature lets it control your browser, click through web apps, and automate tasks that don't have an API connector. This is the most powerful feature in the product and, at the same time, the most restricted.

**Computer Use works on macOS and Windows** for Pro and Max users. Windows support shipped on April 3, 2026, bringing parity with the original macOS release. The feature is still labeled research preview on both platforms.

On either platform, the agent can:

- Navigate web apps and fill forms
- Extract data from sites without an API
- Chain browser actions across multiple tabs or services

Reliability varies by site complexity. Simple, well-structured web apps work well. Dynamic single-page apps with heavy JavaScript can cause errors. Treat Computer Use as a capable research preview, not a production-grade automation layer.

### What About Mobile?

Cowork's mobile control feature, called Dispatch, lets you send tasks to your desktop agent from your phone and return later to the completed work. It relies on your desktop being awake and signed in. Anthropic still labels Dispatch a research preview, so treat it as convenience automation rather than a dependency for time-sensitive or high-stakes workflows.

---

## Is Claude Cowork Safe for Sensitive Documents?

File security in Cowork operates on two layers: local sandboxing and enterprise audit gaps.

**Local sandboxing:** All file operations happen in a local VM. Nothing is sent externally unless you explicitly direct the agent to do so through a connector. For most users, this is the right level of protection.

**Enterprise compliance gap:** As of April 2026, Cowork activity is excluded from Enterprise Audit Logs. This means IT and compliance teams can't review what the agent accessed or actioned on behalf of employees. For regulated industries — finance, healthcare, legal — this is a meaningful gap.

C-suite leaders estimate that only 4% of employees use GenAI for 30% or more of their daily work, while employee self-reports put that number at 13% ([McKinsey Superagency, 2025](https://www.mckinsey.com/capabilities/tech-and-ai/our-insights/superagency-in-the-workplace-empowering-people-to-unlock-ais-full-potential-at-work)). The audit log gap may explain part of that disconnect: employees are using AI tools that IT teams can't fully see.

The audit log exclusion reflects how quickly Cowork shipped. Compliance tooling typically follows core product stabilization. Enterprises evaluating the tool today should treat this as a roadmap item to track, not necessarily a hard blocker, depending on their regulatory context.

---

## Claude Cowork vs. Microsoft Copilot and Gemini for Business

As of early 2026, the AI productivity assistant market has three serious desktop contenders: Claude Cowork, Microsoft 365 Copilot (Enterprise add-on at $30/user/month on top of an existing E3/E5 seat; Copilot Business starts at $21/user/month), and Google Workspace with Gemini bundled in (Business Standard at $16.80/user/month, Business Plus at $26.40/user/month). Cowork starts at $20/month and is the only one offering autonomous multi-step task execution, a plugin marketplace, and 38+ cross-platform connectors outside a single vendor's ecosystem. Here's how they compare on dimensions that matter most for knowledge workers.

| Feature | Claude Cowork | Microsoft 365 Copilot | Google Workspace + Gemini |
|---|---|---|---|
| Primary user type | Knowledge workers | Microsoft 365 users | Google Workspace users |
| Autonomous task execution | Yes (multi-step) | Partial (within M365 apps) | Partial (within Workspace) |
| File access | Local sandboxed VM | SharePoint and OneDrive | Google Drive |
| Scheduling | Yes (/schedule command) | Limited | Limited |
| Browser automation | Yes (macOS and Windows, research preview) | No | No |
| Custom skills | Yes (/skill-creator) | No | No |
| Connector count | 38+ | Deep M365 stack | Deep Workspace stack |
| Plugin marketplace | Yes (Feb 2026) | No | No |
| Enterprise audit logs | No (gap) | Yes | Yes |
| Starting price | $20/month (Pro) | $30/user/month (Enterprise add-on) | From $8.40/user/month (Gemini bundled) |
| Mobile control | Dispatch (research preview) | Yes (via M365 mobile) | Yes (via Workspace mobile) |

*Pricing as of April 2026. Google discontinued the standalone Gemini Business add-on in early 2026 and bundled Gemini into paid Workspace plans. Verify current rates at [Microsoft 365 Copilot pricing](https://www.microsoft.com/en-us/microsoft-365-copilot/pricing-new) and [Google Workspace pricing](https://workspace.google.com/pricing).*

**The practical summary:** Copilot and Gemini are deeper inside their own ecosystems. If you live in Microsoft 365 or Google Workspace, those tools have structural advantages. Cowork's edge is breadth of connectors, autonomous multi-step execution, and the scheduler — making it the stronger choice for users who span multiple platforms or need tasks completed without monitoring the process.

---

## What Plan Do You Need for Claude Cowork?

Claude Cowork is available on four paid Claude plans ranging from $20/month (Pro) to enterprise custom pricing. The Pro plan covers most individual knowledge workers. The Max plan at $100–$200/month adds higher task limits that reset every five hours — relevant if you run scheduled automation throughout the workday. Teams and Enterprise plans add shared workspaces and SSO.

| Plan | Price | Best For |
|---|---|---|
| Pro | $20/month | Individual users, moderate task volume |
| Max | $100–$200/month | Power users; higher limits, resets every 5 hours |
| Team | Managed access | Teams with shared workspaces |
| Enterprise | Custom | Organizations needing spend controls and SSO |

The Pro plan covers most individual use cases. The Max plan is worth considering if you run scheduled tasks throughout the day or use sub-agents for parallel batch processing. Usage resets every five hours on Max, so heavy users see meaningfully higher throughput.

---

## Frequently Asked Questions About Claude Cowork

### What is Claude Cowork used for?

Claude Cowork handles multi-step knowledge work tasks autonomously. Common uses include drafting and formatting documents, processing spreadsheets, scheduling recurring reports, querying connected databases, and automating browser-based workflows on macOS. It reads your local files, connects to 38+ external services, and returns finished output rather than instructions.

### Is Claude Cowork free?

No. Cowork requires a paid Claude plan starting at $20/month (Pro). There's no free tier for Cowork specifically, though Anthropic occasionally offers limited research preview access via Anthropic Labs. The Max plan at $100–$200/month provides higher usage limits with five-hour reset cycles.

### Does Claude Cowork work on Windows?

Yes. Cowork runs on both macOS and Windows, including Computer Use (browser automation) — Windows support for Computer Use shipped on April 3, 2026 for Pro and Max users and is still a research preview on both platforms. File processing, connectors, scheduling, and custom skills work identically across the two platforms.

### How do I schedule a recurring task in Claude Cowork?

Use the `/schedule` command followed by your task description and cadence (hourly, daily, or weekly). Your machine must be awake and online at the scheduled time, as there's no cloud fallback. Missed runs don't catch up automatically. For mission-critical recurring tasks, consider pairing scheduling with a cloud-based trigger as a backup.

### Is Claude Cowork safe for confidential files?

File processing happens in a local sandboxed VM, so files don't leave your machine unless you send them through a connector. The main enterprise concern is that Cowork activity is currently excluded from Enterprise Audit Logs. Regulated industries should review this compliance gap with their IT and legal teams before deploying the tool at scale.

---

## How Do You Get the Most Out of Claude Cowork?

Cowork is most effective when you invest five minutes upfront in the context file setup, one hour configuring two or three connectors for your most-used services, and one session building a custom skill for your most repetitive weekly task. That's roughly 90 minutes of setup for potentially hours of recovered time each week. For context: 47% of surveyed employees expect AI to replace at least 30% of their daily work within a year ([McKinsey Superagency, 2025](https://www.mckinsey.com/capabilities/tech-and-ai/our-insights/superagency-in-the-workplace-empowering-people-to-unlock-ais-full-potential-at-work)) — the window to build this fluency is now.

The product launched in January 2026 and is still labeled a research preview. The compliance gaps and the rough edges around Dispatch and research-preview Computer Use will improve over time. What exists today is already capable enough to handle real work — go in with realistic expectations about the edges.

If you're a data engineer or analyst, the Snowflake and BigQuery connectors combined with the `/schedule` command cover a genuinely useful set of data-to-report workflows. That's the best starting point. Browse all posts tagged [claude](/tags/claude/) for related deep-dives as they're published. You can also find more posts under the [automation](/tags/automation/) and [AI productivity](/tags/ai-productivity/) tags.

---

*[Francisco Frez](https://ffrezr.github.io) is a Data Engineer and writer covering AI tools, data infrastructure, and GCP.*

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "BlogPosting",
      "@id": "https://ffrezr.github.io/posts/how-to-use-claude-cowork/#article",
      "headline": "How to Use Claude Cowork: The Complete Guide (2026)",
      "description": "Complete guide to Claude Cowork: Anthropic's desktop AI agent. Learn setup, skills, 38+ connectors, and scheduling to automate hours of work each week.",
      "datePublished": "2026-04-15",
      "dateModified": "2026-04-15",
      "author": { "@id": "https://ffrezr.github.io/#person" },
      "publisher": { "@id": "https://ffrezr.github.io/#organization" },
      "image": { "@id": "https://ffrezr.github.io/posts/how-to-use-claude-cowork/#primaryimage" },
      "mainEntityOfPage": {
        "@type": "WebPage",
        "@id": "https://ffrezr.github.io/posts/how-to-use-claude-cowork/"
      },
      "wordCount": 2100,
      "articleBody": "Claude Cowork is Anthropic's autonomous desktop AI agent for knowledge workers, launched in January 2026. It runs inside the Claude Desktop app and handles multi-step tasks on your behalf — from drafting reports to processing spreadsheets and scheduling recurring workflows.",
      "keywords": ["claude cowork", "anthropic", "ai productivity", "automation", "desktop ai agent"]
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
      "url": "https://ffrezr.github.io",
      "logo": {
        "@type": "ImageObject",
        "url": "https://ffrezr.github.io/assets/img/avatar.jpg",
        "width": 400,
        "height": 400
      },
      "sameAs": [
        "https://github.com/ffrezr",
        "https://linkedin.com/in/francisco-frez-rojas"
      ]
    },
    {
      "@type": "BreadcrumbList",
      "@id": "https://ffrezr.github.io/posts/how-to-use-claude-cowork/#breadcrumb",
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
          "name": "How to Use Claude Cowork: The Complete Guide (2026)",
          "item": "https://ffrezr.github.io/posts/how-to-use-claude-cowork/"
        }
      ]
    },
    {
      "@type": "FAQPage",
      "@id": "https://ffrezr.github.io/posts/how-to-use-claude-cowork/#faq",
      "mainEntity": [
        {
          "@type": "Question",
          "name": "What is Claude Cowork used for?",
          "acceptedAnswer": {
            "@type": "Answer",
            "text": "Claude Cowork handles multi-step knowledge work tasks autonomously. Common uses include drafting and formatting documents, processing spreadsheets, scheduling recurring reports, querying connected databases, and automating browser-based workflows on macOS. It reads your local files, connects to 38+ external services, and returns finished output rather than instructions."
          }
        },
        {
          "@type": "Question",
          "name": "Is Claude Cowork free?",
          "acceptedAnswer": {
            "@type": "Answer",
            "text": "No. Cowork requires a paid Claude plan starting at $20/month (Pro). There's no free tier for Cowork specifically, though Anthropic occasionally offers limited research preview access via Anthropic Labs. The Max plan at $100–$200/month provides higher usage limits with five-hour reset cycles."
          }
        },
        {
          "@type": "Question",
          "name": "Does Claude Cowork work on Windows?",
          "acceptedAnswer": {
            "@type": "Answer",
            "text": "Yes. Cowork runs on both macOS and Windows, including Computer Use (browser automation) — Windows support for Computer Use shipped on April 3, 2026 for Pro and Max users and is still a research preview on both platforms. File processing, connectors, scheduling, and custom skills work identically across the two platforms."
          }
        },
        {
          "@type": "Question",
          "name": "How do I schedule a recurring task in Claude Cowork?",
          "acceptedAnswer": {
            "@type": "Answer",
            "text": "Use the /schedule command followed by your task description and cadence (hourly, daily, or weekly). Your machine must be awake and online at the scheduled time, as there's no cloud fallback. Missed runs don't catch up automatically. For mission-critical recurring tasks, consider pairing scheduling with a cloud-based trigger as a backup."
          }
        },
        {
          "@type": "Question",
          "name": "Is Claude Cowork safe for confidential files?",
          "acceptedAnswer": {
            "@type": "Answer",
            "text": "File processing happens in a local sandboxed VM, so files don't leave your machine unless you send them through a connector. The main enterprise concern is that Cowork activity is currently excluded from Enterprise Audit Logs. Regulated industries should review this compliance gap with their IT and legal teams before deploying the tool at scale."
          }
        }
      ]
    },
    {
      "@type": "ImageObject",
      "@id": "https://ffrezr.github.io/posts/how-to-use-claude-cowork/#primaryimage",
      "url": "https://ffrezr.github.io/assets/img/posts/how-to-use-claude-cowork/claude-cowork-og.png",
      "width": 1200,
      "height": 630,
      "caption": "Claude Cowork complete guide 2026 — Anthropic desktop AI agent"
    }
  ]
}
</script>
