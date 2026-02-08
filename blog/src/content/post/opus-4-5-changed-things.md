---
title: "Opus 4.5 Changed Things"
publishDate: "9 February 2026"
description: "The third big shift in AI and Software Engineering"
tags: []
---

This is a detailed post to share what I’ve learned over the last few weeks as we’ve deliberately stopped using AI agents to _write code_ and instead started treating them as _software engineers_. Today, that means I routinely run **eight agents in parallel**, each in its own isolated devcontainer, all working concurrently.

That shift sounds subtle, but it isn’t. It changes how work gets planned, how it gets reviewed, what tooling matters, and where humans should spend their time. I’ll go into a lot of detail below, because the details are the whole point.

It’s also worth being explicit about scope: this isn’t an eval, a toy repo, or a synthetic benchmark. This is an actual product we’re building, live in beta, and treated as a production application. The agents are working against real services, real data, real CI, and real users.

One quick disclaimer, because it’s easy to misread posts like this: I actually enjoy writing code. I like hand-building systems, learning new technologies, and even learning entirely new programming languages. None of that has gone away for me, and this isn’t a rejection of that part of the job.

In the last seven days, I shipped more production code than any other seven-day stretch in my nearly twenty-year career. In that window I merged **46 pull requests**, with roughly `+29,358 / -4,619` lines changed. It’s not close.

I also spent about **$1,000 in Cursor** during that same period—almost all on **Claude Opus 4.5** (and some 4.6). I ran **thinking models only**, with **Deep mode** and **Max context** enabled.

The speed and parallelism are the obvious part. The bigger change is that a bunch of things I’d normally never bother doing—internal tooling, cleanup work, observability tweaks, performance improvements, better documentation, eval plumbing—suddenly become reasonable.

One small example: I was working on an agent eval and had a dataset in LangSmith. I couldn’t quickly find a LangSmith CLI (and honestly, Google results for anything LangChain / LangGraph / LangSmith are a mess). I knew there was an API, so I had an agent code up a CLI for our workflow. It took about three minutes. Now agents can search through traces to find examples to add to the dataset, change the data structure of the dataset, and generally do everything I wanted to do inside LangSmith. You can, in fact, just do things.

## The Shift

For a long time, LLM-assisted coding still meant I was the executor. The model wrote pieces of code, I stayed closely involved, and progress was mostly linear.

Looking back, I think there have been four distinct eras.

The first was **VS Code + GitHub Copilot with tab-complete**. When that showed up, it felt novel. It was our first real taste of “sometimes these LLMs can be right.” Often they weren’t. We laughed at plenty of suggestions and wondered where some of them came from. Still, even when tab-complete was right only about half the time, I found it useful and kept it on.

The second era was **Cursor as an editor with agent chats**. This was a real step up. You could talk to the AI while you were working, ask questions about the codebase you were in, and get answers with real context. That alone was far better than copy-pasting into ChatGPT. Then it quickly became clear that you could ask the agent to write code, and it would do a decent job. Then multiple files. Then larger changes.

The third era is the one we’re in now, and it’s a much bigger shift. These are **coding agents**—or what Andrej Karpathy has called _agentic engineering_. This is the first time I’ve felt a real mental-model break.

In this mode, you’re not writing code at all. You’re managing software engineers. The engineers happen to be AI, but they’re doing fully end-to-end work: investigation, planning, implementation, testing, CI, and documentation. The key shift is that you’re no longer “in the game” alongside the agent. You align up front on a plan, let them play the game, and then review the game footage afterward.

You’re not the player anymore. You’re the coach.

For me, **Opus 4.5** was the moment this clicked. That model came out last month, and it was the first time I felt confident that we could use an LLM to do real software engineering—not just write code.

I think there’s a fourth era coming next, and it’s fairly obvious where this all leads: **autonomous codebases**.

Cursor has written about this explicitly in their post on self-driving codebases: https://cursor.com/blog/self-driving-codebases

I’m not there yet. But this is very clearly the direction I’m moving in, and the direction I’m optimizing for. Getting from “coding agents” to something closer to autonomous codebases feels like the next real step. And yes—it sounds pretty incredible.

## Why Cursor

There are a lot of coding-agent tools right now: Codex, Codex CLI, Claude Code CLI, Cursor IDE, Cursor CLI, OpenCode, and plenty more.

The honest answer for why I use Cursor is simple: I started there, and it stuck. I like the IDE model. I also live in a terminal (I only use `psql` with Postgres, I use Ghostty constantly, I do most things with Unix commands on Debian, and even on macOS I use Ghostty to manage things — I’m not inherently a GUI lover). But for orchestrating agents, searching past conversations, and keeping context anchored to the code, the convenience of an IDE really helps.

The biggest reason, though, is browser control. My agents can start all of our services (Qwik/Fastify, FastAPI/Uvicorn, and Celery), open the app in a browser, navigate the site end to end, and validate behavior while watching service stdout and logs at the same time. That combination is huge, and I haven’t seen it done as well elsewhere.

I also like the direction Cursor is taking with local and cloud agents. Cloud agents are ultimately where this goes. I don’t really want to be running multiple machines at home long-term. The idea that I can start an agent in the cloud, take it over locally, and then send it back to the cloud is incredibly compelling.

Right now, getting our full dev environment working in Cursor’s cloud agents has been tricky, and cloud agents don’t appear to have browser access yet. So for the moment, my preferred setup is local Cursor agents, running the same project cloned into four separate devcontainers per machine.

## What Counts as a Real Coding Agent

For me, a useful agent isn’t defined by how well it writes code in isolation. It’s defined by whether it can operate across the software development lifecycle.

In practice, that means it can investigate a problem, reproduce it when possible, trace code paths, and look for prior art. It can write an implementation plan, work test-first, implement changes, run formatting, linting, type checks, and tests, and then open a pull request with a real description. From there, it watches CI, fixes failures, and keeps going until things are green.

When it makes sense, it deploys and validates by actually using the product in a browser—sometimes even driving our in-app chat agent—while watching service stdout/logs for errors. It also has full access to the data layer: connecting to Postgres via `psql`, running ad-hoc queries to debug issues, pulling down the latest staging database dump, rebuilding the database locally, and generally operating with the same level of access a human engineer would have.

## What This Looks Like in Practice

### Technology stack

To ground all of this, here’s what the actual system looks like.

HINT is a three-component system — a web server, an API server, and background workers — running on AWS ECS and developed inside Docker devcontainers on Debian Bookworm.

The **web server** is Qwik with Fastify. It’s the user-facing frontend and its server-side rendering layer, running in production and via Vite in development. This service owns the primary PostgreSQL application database and handles authentication. Logging is via Pino and errors are captured with Sentry.

The **API server** is the AI backend, built with FastAPI on Python 3.11 and running behind Uvicorn (and nginx in production). It hosts the LangGraph-based agent system. TimescaleDB is used as the vector database for RAG workloads. Errors are captured in Sentry and traces are sent to LangSmith.

The **background workers** are Celery. We use them for longer-running inference workflows and scheduled tasks via Celery Beat.

Infrastructure-wise, we use Redis for Celery brokering and caching, Neo4j for graph data, and deploy everything to AWS ECS in `us-east-2` across production, staging, and eval environments. Observability is a combination of Sentry (errors), LangSmith (agent traces), CloudWatch (metrics and alarms), and Slack for alerting.

Development happens entirely inside Docker devcontainers.

### Rules and skills

A lot of what makes this work lives in Cursor rule and skill files under `.cursor/rules` and `.cursor/skills`. The important rules are marked with `alwaysApply: true`, so they’re injected into every agent task automatically.

Planning is the center of gravity. For non-trivial work, I’ll often go back and forth with an agent on a plan many times—sometimes 15–20 messages just refining the approach. That might sound heavy, but the agent can iterate on plans extremely quickly while I’m doing other things.

Once we agree on a plan, the agent executes it end-to-end. When the work is shipped, I come back and do something that turns out to be incredibly valuable: we update the plan with hindsight. We capture what worked, what didn’t, what we learned, and how we’d approach it next time. Then we move that plan out of the transient plan directory and into the rules as an example, and add a reference to it from `planning.mdc`.

Right now we only have a handful of example plans, but over time we’ll likely be adding several a day. Looking back at them after the fact has been surprisingly rewarding—they really are works of art.

#### What’s actually in `.cursor/rules` and `.cursor/skills`

```
.cursor/
├── rules/
│   ├── project-overview.mdc              # Project Overview (71 lines)
│   ├── software-architecture.mdc         # Software Architecture (235 lines)
│   ├── software-development-lifecycle.mdc# SDLC (217 lines)
│   ├── planning.mdc                      # Implementation Planning (341 lines)
│   ├── outcomes.mdc                      # Outcomes Over Output (95 lines)
│   ├── example-sentry-error.md           # Sentry error triage plan (317 lines)
│   ├── example-backend-bug.md            # Backend bug fix plan (258 lines)
│   └── example-ci-optimization.md        # CI optimization plan (203 lines)
│
├── skills/
│   ├── alerting/SKILL.md                 # Alerting (53 lines)
│   ├── api-design/SKILL.md               # REST API Design (168 lines)
│   ├── aws/SKILL.md                      # AWS Infrastructure (53 lines)
│   ├── browsing-app/SKILL.md             # Browsing the App (116 lines)
│   ├── database/SKILL.md                 # Database & Models (109 lines)
│   ├── development-commands/SKILL.md     # Dev Commands (183 lines)
│   ├── error-handling-logging/SKILL.md   # Error Handling & Logging (149 lines)
│   ├── langsmith/SKILL.md                # LangSmith (117 lines)
│   ├── linear-ticket/SKILL.md            # Linear Tickets (115 lines)
│   ├── package-management/SKILL.md       # Dependencies (39 lines)
│   └── pull-request/SKILL.md             # Pull Requests (188 lines)
```

That’s **19 files** and roughly **3,027 lines** of always-on guidance.

**Rules** define the global shape of the system and how work proceeds:

- **Project Overview** — Defines the HINT system architecture (Qwik web server, FastAPI backend, Celery workers), database ownership boundaries, data flow between components, and package management requirements (`pnpm` for TypeScript, `uv` for Python).
- **Software Architecture** — Covers DDD principles, the Unit of Work pattern for DAOs, data access layer conventions (DAOs for TimescaleDB, client classes for the Qwik private API), dependency injection, auth patterns, naming conventions, and directory structure.
- **Software Development Lifecycle (SDLC)** — The end-to-end workflow: check for a Linear ticket, plan the work, follow TDD (red–green–refactor), run code quality checks, browser-test UI changes, create a PR, and wait for CI to go green.
- **Implementation Planning** — Structured planning with a real Phase 0 investigation (reproduce the bug, research prior art, trace the codebase) before proposing solutions, followed by branching, TDD, local verification, PR creation, and CI monitoring.
- **Outcomes Over Output** — Guiding philosophy: solve the actual problem rather than shipping code to close tickets; slow is smooth, smooth is fast; consistency over correctness; do the least work possible.
- **Plan examples** — Real, battle-tested plans that agents can read and pattern-match against.

**Skills** are more tactical and task-oriented:

- **Alerting** — Alerting philosophy, Sentry + CloudWatch, and monitoring scheduled Celery Beat jobs.
- **REST API Design Guidelines** — Conventions for Qwik and FastAPI endpoints, including validation and response models.
- **AWS Cloud Infrastructure** — AWS naming, environment mapping, and ECS services.
- **Browsing the App Website** — How agents start services and validate flows in a real browser.
- **Database & Models** — PostgreSQL schema rules, migrations, and modeling conventions.
- **Development Commands Reference** — Cheat sheet for tests, linting, formatting, DB operations, and dev servers.
- **Error Handling & Logging** — Don’t catch exceptions unless you can recover; correct Sentry usage; logging patterns.
- **LangSmith Configuration** — LangSmith CLI usage, datasets, and common queries.
- **Linear Tickets** — Ticket structure and traceability.
- **Dependencies** — Exact versions only; no manual edits.
- **Pull Request Descriptions** — PR format, required sections, and real examples.

### Proving tests can fail

One concrete thing we require around TDD is verifying that tests can actually fail. Agents will intentionally modify the source code in a way that should break the behavior, run the test to confirm it fails, then revert the change and proceed with the real implementation. This sounds obvious, but just like humans, agents can write tests that always pass. I’ve seen it happen many times.

### Isolating agents like real engineers

Right now I’m typically running **eight agents at once** in parallel: four devcontainers on my main machine and four on a second machine. This parallelism is core to how this setup works.

My main machine is a Mac Studio with an **M3 Ultra** chip and **96 GB of RAM**. The other is a recent MacBook Pro with an **M4 Pro and 64 GB of RAM**. Both machines are allowed to give devcontainers access to all CPUs and all memory. I don’t artificially constrain them.

Each agent gets its own devcontainer, its own copy of the repo, and its own database and local services. Without this, things fall apart quickly once multiple agents are working in parallel.

Getting this stable on macOS required real work. Early on, devcontainers would intermittently disconnect and enter a “reconnecting” state. Investigation showed extreme bind-mount I/O—on the order of **hundreds of gigabytes of block reads per container session**. The root cause wasn’t CPU or memory; it was filesystem churn.

The fixes fell into three buckets:

- **Move heavy directories off the bind mount.** Large directories like `node_modules`, `.pnpm-store`, and Python virtualenvs were moved into Docker named volumes. These accounted for the majority of I/O and were the single biggest win.
- **Stop file watchers from scanning everything.** Every watcher was tightened: Vite, uvicorn, TypeScript, Python reloaders, and other tools were explicitly configured to ignore large and irrelevant directories. This prevented multiple independent tools from repeatedly walking the same trees.
- **Keep tool caches container-local.** Language server and tooling caches (mypy, ruff, pytest, etc.) were moved to `/tmp` inside the container. The caches are ephemeral, but they regenerate quickly and eliminating constant metadata churn on the bind mount made a huge difference.

Together, those changes eliminated the majority of the blocking I/O patterns. Once unnecessary host↔container filesystem traffic was removed, I could run four devcontainers at the same time with agents actively doing work in them without problems.

Getting to that point also required making the devcontainer setup explicitly support _multiple clones of the same repo_ on one machine.

Originally, the devcontainer assumed a single instance: hard‑coded Docker Compose project names, hard‑coded port bindings, and a fixed devcontainer name. Cloning the repo twice and opening both in Cursor led to port conflicts, container name collisions, and Cursor repeatedly re‑running initialization because it couldn’t distinguish between instances.

The fixes were incremental but important:

- **Dynamic Compose project names.** We removed the hard‑coded `name:` from `compose.yml`. An `initializeCommand` now derives a `HINT_CLONE_ID` from the parent folder name and writes it into `.devcontainer/.env`, which Docker Compose uses as the project name. Each clone becomes its own isolated stack.
- **Dynamic port forwarding.** All hard‑coded host port bindings for dev services (Qwik, FastAPI, nginx, pgbouncer) were removed. Cursor/VS Code auto‑detects and forwards ports with conflict resolution. Databases use random host ports; internal container‑to‑container networking stays stable via Docker service names.
- **Unique devcontainer names.** The devcontainer `name` is generated from the workspace folder (e.g. `hint-monorepo-1`, `hint-monorepo-2`) so Cursor can reliably track multiple running instances.
- **Shared and automated CLI credentials.** GitHub, Sentry, and AWS credentials are loaded automatically from whitelisted `.env` variables on startup, so agents don’t require per‑container manual authentication.
- **Persistent Cursor plan volumes.** Cursor’s plan directory is backed by a named Docker volume keyed off the clone name, so plans survive container rebuilds and remain isolated per agent.

With dynamic naming, dynamic ports, and persistent per‑clone state, running several identical devcontainers side‑by‑side stops being fragile and starts feeling boring — which is exactly what you want when you’re managing multiple agents.

To put some concrete numbers on it: most of the time, with four full devcontainers running on a machine, overall container CPU usage sits around ~40–45% of available cores (out of 24 logical CPUs), and memory usage is roughly ~23 GB out of ~60 GB available. In other words, there’s plenty of headroom.

In practice, this setup has virtually no noticeable impact on my day‑to‑day machine performance. I’m on Zoom calls, Slack, listening to music, and doing normal work at the same time. I could probably run more devcontainers if I wanted to, but for now four per machine has been a comfortable balance.

### The coding-agent stack

Once agents are running end-to-end, waiting time dominates. Not just CI, but local formatting, linting, type checks, unit tests, and end-to-end tests.

I’ve become a true believer in fast tools, because fast feedback lets you test ideas quickly. Our stack is biased toward things that run fast: `uv`, `bun`, `biome`, `ruff`, `vitest`, and similar tools. We’ll likely have agents trial switching us to things like `tsgo`, `pyx`, and `oxlint` soon.

My goal is that everything—formatting, linting, unit tests, and end-to-end tests—runs in roughly two minutes or less.

CI needed the same treatment. Our E2E tests were slow enough that GitHub Actions queues backed up once multiple agents were active. We switched to four-core runners and parallelized E2E tests across cores.

We also rely heavily on Cursor’s bug bot. After an agent pushes code, it waits for bug bot feedback and watches CI. Agents run simple sleep-and-poll loops so they can read bug bot comments as they appear and address them immediately. The bug bot is genuinely good.

More broadly, this is what my job looks like now.

I spend most of my time looking for signals that something in the _system_ is off. Are CI queues backing up? Then I need to optimize workflows or speed up tests. Are agents making sloppy database schema changes? That’s a rules or planning problem. Are PRs annoying to review? Then I need better plans, better examples, or stricter expectations.

When I see patterns like that, I don’t fix individual PRs over and over — I change the system. I update rules, skills, planning templates, or example plans so future agents don’t repeat the same mistakes. In practice, that feels a lot like retraining a team.

Right now, I’m making a lot of changes to the setup because the system is still young. Over time, I expect that to taper off. But this is the core job now: continuously improving the efficiency and quality of a parallel team of software engineers.

### Team dynamics and code review

This way of working has changed our team dynamics pretty fundamentally.

We used to work like most teams do: one engineer takes a task, opens a PR, gets code review, addresses comments, and eventually merges. That flow doesn’t really scale when each person is orchestrating multiple agents in parallel.

If everyone on the team is running something like **eight agents at once**, they’re each going to have multiple PRs landing every day. Expecting traditional, line-by-line peer review on all of that simply isn’t realistic.

So we’ve shifted the model. The person orchestrating the agents is also the primary code reviewer, and that review counts as code review. They’re responsible for understanding what was built, why it was built that way, and whether it meets our standards.

If there’s an area they don’t feel confident about, they’ll pull in the team expert on that topic. In practice, that usually happens during **planning**, not after the code is written. Planning is where we want the collective brain power of the team.

We still review the code and ask for changes when needed, but the goal is that those changes are small. Nothing is worse than finishing a task and then discovering that 50% of the code isn’t mergeable. That’s a failure of planning.

For this to work, it has to be true that **95–100% of the code an agent produces is acceptable most of the time**. The only way to get there is to invest heavily in planning. The better the agents are at planning, the better they are at everything else.

### Prior art lives in GitHub

GitHub is effectively our datastore of prior art. Pull request descriptions, review comments, CI output, test failures, and fixes all live there together. If feedback only happens in a chat session, it disappears. When it happens in GitHub, it becomes part of the record.

### Observability is non‑negotiable

Running agents like this only works if you have extremely good observability. We’ve taken a lot of care to make sure logs, metrics, and traces are readable and actionable by both humans and agents.

All application logs are structured and intentionally readable. We emit CloudWatch metrics for log levels (INFO, WARN, ERROR, etc.) and wire alarms off those metrics directly into Slack. Any abnormal behavior shows up quickly in an `#alerts` channel where we can talk to Cursor and Linear in real time.

We also rely heavily on Sentry. Exceptions are captured consistently, formatted clearly, and piped into the same alerts channel. Most days, when I look at the last 24 hours across all production services, there are zero exceptions. On days when there are a few, I’ll immediately hand them to an agent and usually have a fix merged within minutes.

Beyond errors, we use Sentry traces for performance monitoring. Latency regressions often signal bugs or bad code paths before users complain. We also maintain CloudWatch dashboards and alarms for ECS Fargate services, RDS/Postgres health, and background job completion. Any deviation from expected behavior surfaces quickly.

Between logs, metrics, traces, and alerts, we’ve given ourselves enough visibility into the system that running coding agents this way feels safe rather than reckless.

### Software taste and standards

I care a lot about code quality. I really don’t like bad code: regressions, subtle bugs, unnecessary latency, data corruption, or sprawling spaghetti architectures. I wasn’t willing to accept AI slop just to move faster.

To make standards concrete, I had an agent go through years of my GitHub PR reviews and extract before/after examples into a single markdown file that’s now **1,600+ lines long**, organized into:

1. Error Handling and Exceptions
2. Logging Best Practices
3. Documentation Standards
4. Architecture and Design
5. Code Quality
6. Naming Conventions
7. Type Safety
8. Database Schema Design
9. Environment Variables
10. Simplicity Principles
11. LLM/AI Agent Patterns

## Model choice

That spend is completely worth it to me. We’re a seed-stage startup, pre-revenue, and I still don’t hesitate here.

I’ve said this for years: if your inference bill is low, you’re probably not really using intelligence. You’re not actually doing AI.

For context, I spent about $1k on inference in the last seven days. I’m sure that’s nowhere near the high end of what some teams are doing. The point is simply that, for me, it’s been entirely worth it.

I run thinking models with deep mode and max context all the time. I don’t constantly switch between Sonnet and Opus, or between thinking and non-thinking, or trim context windows to save a few dollars. I let the agents have as much context and reasoning headroom as they want.

There’s also a lot of AI hype online. A new model drops and within hours you see proclamations about which model is “better.” None of that has been very useful for me.

I don’t care how well a model draws a pelican (hi, Simon! no shade here — I love your work!). I care how it performs inside our codebase, with our rules, our tooling, and our problems.

If I went by my X feed alone, I’d think Codex was the second coming. I believe people when they say it works well for them. I’m not claiming it’s a bad model. But in our environment, for end-to-end software engineering, it hasn’t come close to the Opus thinking models.

## How This Might Change the Market

I could be wrong here — I often am — but it’s worth speculating.

I think this will make it harder for entry-level and junior engineers to get hired. A lot of the work that used to justify junior roles can now be done by agents under the supervision of fewer experienced engineers.

There’s a paradox: if younger engineers don’t get hired and don’t get experience, who manages all of these agents and autonomous codebases ten or twenty years from now?

I also think younger engineers bring something that’s hard to replace. Many are willing to try things without fear. They don’t yet have all the scars from bad SaaS, painful outages, messy migrations, and brittle systems. That lack of baggage is a strength.

As for the rest of us, I think the shift is pretty clear: we’re engineering managers now. Even if our titles don’t change, the job does. We’re managing teams of AI software engineers.

How you use this technology matters. If you’re using agents purely to ship more code, I think you’re missing the point. I don’t ship code I don’t understand. When agents build something, I ask a lot of questions.

The real silver lining for me is learning. With agents, I can learn fifteen new things in a day without waiting on a senior engineer, digging through search results, or trawling Stack Overflow.

My brain feels like a sponge again. I’m a student.

How you use this technology matters. If you’re using agents purely to ship more code, I think you’re missing the point. I don’t ship code I don’t understand. When agents build something, I ask a lot of questions.

The real silver lining for me is learning. With agents, I can learn fifteen new things in a day without waiting on a senior engineer, digging through search results, or trawling Stack Overflow.

My brain feels like a sponge again. I’m a student.

## Appendix: Merged PRs (Last 7 Days)

_This is included for concreteness, not as a brag._

**Total:** 46 PRs · **+29,358 additions** / **-4,619 deletions**

- #843 docs: add dependency upgrade example to planning rules — +257 / -0
- #842 feat: upgrade Fastify v5, Qwik 1.19.0, Pino 10.3.0, Vite 7.3.1, Vitest 4.0.18 — +1,579 / -1,103
- #838 perf: optimize e2e tests and CI parallelization — +104 / -87
- #837 fix: report file processing errors to Sentry — +53 / -2
- #836 ci: add paths-ignore to deployment and code quality workflows — +114 / -0
- #835 docs: add categorized example plans to planning documentation — +360 / -34
- #826 feat: add LangSmith CLI tool for trace exploration — +782 / -0
- #821 fix: route /private/\* to FastAPI and fix error swallowing — +411 / -42
- #820 docs: add investigation phase to planning skill and create outcomes rule — +464 / -25
- #816 perf: optimize devcontainer I/O to prevent disconnections — +100 / -3
- #815 feat: add CloudWatch monitoring to expire_incentives task — +360 / -10
- #814 feat: add Celery Beat infrastructure for scheduled tasks — +197 / -1
- #809 feat: automated database backup with CloudWatch monitoring — +162 / -17
- #805 feat: consolidate deployment workflows to prevent race conditions — +807 / -198
- #800 chore: enable stricter ruff linting rules — +590 / -592
- #798 docs: enhance Cursor rules/skills and add PR feedback examples — +2,006 / -25
