---
title: "Opus 4.5 changed things"
publishDate: "9 February 2026"
description: "A field report on what changed when I stopped using AI to write code and started managing it like software engineers."
tags: []
---

> I stopped using AI to write code and started treating it like a team of engineers.That single shift changed everything else for me.

Over the last few weeks, I deliberately stopped using AI agents to _write code_ and started treating them as _software engineers_. That sounds subtle. It isn't. It changes how work gets planned, how it gets reviewed, what tooling matters, and where humans should spend their time.

This post tells that story chronologically — what broke, what I fixed, and what I learned along the way — with the lessons embedded where they were learned.

A few things up front so the context is clear:

- This is a real product, live in beta, treated like production. Not an eval, not a toy repo.
- We have five other engineers working in the repo. They kept chugging along while I went off on this exploration.
- Today, I routinely run **eight agents in parallel**, each in its own isolated devcontainer, all working concurrently.
- I enjoy writing code. I like hand-building systems and learning new technologies. None of that went away. What changed is where my time creates the most leverage.

In the last seven days alone, I merged **46 pull requests**, with roughly `+29,358 / -4,619` lines changed. It's not close to anything else I've done in a similar time window.

I also spent about **$1,000 in Cursor** during that period — mostly on **Claude Opus 4.5**. That context matters later.

> The job isn’t writing code anymore. It’s orchestrating work across an elastic team of software engineers.

## What changed (the six lessons)

I didn't start with a theory. These were learned by running the system and fixing it repeatedly.

1. **Engineers are becoming engineering leaders of elastic teams.** The job is no longer typing — it's orchestration.

2. **Tooling speed suddenly matters a lot.** Locally and in CI. When you're running multiple agents in parallel, feedback loops must be fast or everything collapses.

3. **You have to trust agents, but strong observability and low mean-time-to-detection and recovery must be part of the system.** Agents will make mistakes. The system should catch them quickly.

4. **The real work is planning and continuously improving the rules and constraints the agents operate under.** This is where most of the leverage lives.

5. **If your inference bill is low, you're probably not using AI.** Stop optimizing for cost. Optimize for results. The speed and quality gains dwarf the API costs.

6. **The agent's output will be as good as the plan you build with it.** Planning quality is the single strongest predictor of execution quality.

> I went from ‘AI helps me code’ to ‘AI does end-to-end software engineering. ’That distinction ended up mattering more than any benchmark.

Everything below is how those lessons showed up in practice.

## Opus 4.5 unlocked software engineers

Looking back, I think there have been three distinct eras so far, and a fourth emerging.

> A real coding agent isn’t defined by how well it writes code.It’s defined by whether it can operate across the entire software lifecycle.

**Era 1: VS Code + GitHub Copilot with tab-complete.** This felt novel. It was our first taste of "oh, sometimes these LLMs can be right." Often they weren't. We laughed at plenty of the suggestions and wondered where some of them came from. But even when tab-complete was right only about half the time, I found it useful and kept it on.

**Era 2: Cursor editor + agent chats.** A real step up. You could talk to the AI with the context of the codebase right there — far better than copy-pasting into ChatGPT. Then it became clear the agent could write decent code. Then multi-file changes. Then larger tasks. The progression was fast and exciting. The mental model shift is that you're no longer "in the game" alongside the agent. You align on a plan, let them play, and review the game footage afterward. You're the coach, not the player.

**Era 3: Coding agents (agentic engineering).** This is where things broke open. In this mode, you're not writing code. You're managing software engineers who happen to be AI, doing fully end-to-end work: investigation, planning, implementation, testing, CI, documentation. The mental model shift is that you're no longer "in the game" alongside the agent. You align on a plan, let them play, and review the game footage afterward. You're the coach, not the player. Andrej Karpathy calls this agentic engineering. For me, Opus 4.5 was the moment it clicked — the first model I trusted to do real software engineering, not just write code.

**Era 4 (emerging): Autonomous codebases.** This is obviously where everything points. Cursor has written about [self-driving codebases](https://cursor.com/blog/self-driving-codebases), and I'm not there yet, but I'm trying to get there as fast as I can. It sounds incredible.

I went from "AI helps me code" to "AI does software engineering work end to end." It's still early, but the difference is real.

That shift required both technical and cultural change. This post is the story of what that looked like.

## What a real coding agent actually is

A real coding agent isn't defined by how well it writes code. It's defined by whether it can operate across the full software development lifecycle.

> Rules and skills aren’t documentation — they’re part of the system. This is where most of the leverage has been for us.

In practice, that means it can investigate a problem, reproduce it when possible, trace code paths, and look for prior art. It can write an implementation plan, work test-first, implement changes, run formatting, linting, type checks, and tests, and then open a pull request with a real description. From there, it watches CI, fixes failures, and keeps going until everything is green. When it makes sense, it validates behavior in a real environment — actually using the product in a browser (both desktop and mobile), with full access to Chrome DevTools, reading console output, navigating the UI, and driving interactions end to end while watching service stdout and stderr for errors. It has full access to the data layer: connecting to Postgres via `psql`, running ad-hoc queries to debug, pulling the latest staging database dump, rebuilding the database locally. It operates with the same level of access a human engineer would have.

When an agent can't do most of that on its own, it ends up feeling more like autocomplete with extra steps.

## Context: the system we're building

For context, we're a seed-stage startup with a beta live in Texas.

The system has three major components: a Qwik + Fastify web server (the application and primary Postgres owner), a FastAPI service that hosts the agent system (with TimescaleDB as the vector database for RAG and Neo4j connected for a custom-built knowledge graph of the user's home), and Celery workers for longer-running inference workflows and scheduled tasks. We use LangGraph for the agent and our general inference work in FastAPI.

Everything runs in Docker devcontainers on a Debian-based environment that closely mirrors production. Locally, agents operate against the full stack.

Environment parity matters. Agents need to work in the same world humans do.

## Why Cursor

There are a lot of coding-agent tools right now: Codex, Codex CLI, Claude Code CLI, Cursor IDE, Cursor CLI, OpenCode, and many more.

The honest answer for why I use Cursor is simple: I started there, and it stuck. I like the IDE model. I also live in a terminal (I only use `psql` with Postgres, I use Ghostty constantly, I do most things with Unix commands on Debian and macOS — I'm not inherently a GUI lover). But for orchestrating agents, searching past conversations, and keeping context anchored to the code, the convenience of an IDE helps.

The biggest reason, though, is browser control. My agents can start all three of our services, open the app in a browser, navigate the site end to end, and validate behavior while watching service stdout and stderr at the same time. That's huge, and I haven't seen it done as well elsewhere. Granted, I haven't really checked. Some of the engineers I work with use Claude Code. And that's cool. I just haven't spent much time with it.

I also like the direction Cursor is taking with local and cloud agents. Cloud agents are ultimately where this goes. I don't really want to be running multiple machines at home long-term. The idea that I can start an agent in the cloud, take it over locally, and send it back to the cloud is incredibly compelling.

## Cloud agents vs. local agents

I tried Cursor cloud agents early. The idea is compelling and they do a decent job.

But without our full environment — databases, CLIs, browsers, logs — cloud agents behave like coders, not engineers. They can write code, but they can't be software engineers in our environment with all of our tools.

So I went local. Local devcontainers are the same environment humans use. That matters.

Right now, getting our full dev environment working in Cursor's cloud agents has been tricky, and cloud agents don't appear to have browser access yet. So for the moment, my preferred setup is local Cursor agents, running the same project cloned into four separate devcontainers per machine.

## The hard rule

I started with a rule: **all work goes through coding agents**.

Not the editor with inline AI help. The agent screen. By default, you don't even see the code.

This forced the system to show me its weaknesses quickly.

## Early results were bad

At first, results weren't good.

Agents didn't reliably use optional skills. Optional rules were ignored. Planning was shallow. Output was inconsistent.

> Rules and skills are a living system. This is where most of the leverage lives.

That started a continuous feedback loop of observing what agents did, identifying patterns of failure, and updating the system to prevent those failures from recurring.

### Software taste and standards

I care a lot about code quality. I really don't like bad code — regressions, subtle bugs, unnecessary latency, data corruption, spaghetti architectures. I wasn't willing to accept AI slop just to move faster. A lot of the work here has been about making my expectations explicit and enforceable.

I had an agent go through years of my GitHub pull request reviews across all my repos using the `gh` CLI. It extracted before/after code examples, copied my comments, grouped them by theme, and produced a 1,600+ line markdown file. The categories it identified:

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

The distilled version lives directly in our Cursor rules and skills, with links back to the full document when more context is needed. Code quality improved noticeably after this. My next step is having an agent convert this feedback into custom linter rules.

### Prior art and feedback loops

I discovered the value of prior art almost by accident. At one point, I told an agent I had worked on something recently and it should find that PR. It came back saying it had found "prior art" — and when it used that prior art, I noticed the plan was noticeably better. That's when it clicked.

I modified `planning.mdc` and forced agents to always search for prior art as part of Phase 0 investigation. The results have been incredible since.

Even when I'm reviewing AI-written code, I leave feedback on the pull request. Not in chat. In GitHub.

GitHub is our datastore of prior art. Pull request descriptions, review comments, CI output, test failures, fixes — all of it lives there together. That history is how agents (and humans) learn how this codebase wants to be worked on.

If feedback happens only inside a chat session, it disappears. When it happens in GitHub, it becomes part of the record. That's why I push so much of the workflow through PRs.

There's a specific step in `planning.mdc` (which is always applied) that forces the agent to search pull requests and Linear issues using the Linear MCP and `gh` CLI as a prior art step. This ensures agents don't reinvent patterns or miss important context from previous work.

## Rules and skills (and why planning dominates)

All guidance lives in `.cursor/rules` and `.cursor/skills`. Important rules are marked with `alwaysApply: true`, which means every agent sees them on every task.

> Serial work became unbearable once I saw what parallelism unlocked.

Rules and skills are a living system. This is where most of the leverage lives.

I started adding rules and skills each day to fix things, moving guidance from optional skills into always-applied rules, adjusting how many rules were auto-applied, and changing wording, structure, and examples constantly — often daily — based on what I observed agents doing.

Early on, results were mixed. Regardless of model, agents tended to ignore optional skills and optional guidance. That forced a practical distinction: if something mattered, it had to be a rule; if it was optional, it was often skipped.

Planning became the center of gravity.

For non-trivial work, I'll go back and forth with an agent **15–20 times** refining a plan. That sounds heavy, but agents iterate on plans extremely quickly while I'm doing other things. The quality of the plan is the single strongest predictor of output quality.

> The agent's output will be as good as the plan you build with it.

Once the work ships, we don't stop. We update the plan with hindsight — what worked, what didn't, what surprised us — and promote it into the rules as a reference example via `planning.mdc`. The guidance in `planning.mdc` lists example plans by topic like dependency upgrades, Sentry bugs, CI optimization, and so on. The agent knows to identify which example most closely matches its current task and reads it. The examples we put in are very high quality plans because we update them with hindsight and also spend a good chunk of time doing the planning with the LLM. The agent's output will be as good as the plan you build with it.

My job now, as I see it, is mostly looking for problems with the system: is CI getting backed up? Are agents making bad schema decisions? Is code quality slipping in a particular area? When I spot a pattern, I change the rules, the skills, or the planning examples. That's what "retraining" looks like. I'm constantly tuning the system. I'm sure it will taper off over time, but right now this is where most of my time goes.

The core loop: observe failures or friction, change the system, not just the output. Though I do change the output when needed, the real work is updating rules and skills.

One more takeaway here: almost never do I do this level of planning when I'm working with my engineers. And that's fine. But the plans the agent generates are truly awesome to look at and make me feel really, really good once we land on a good plan. This is a true enjoyment for me and something I don't get when working with human engineers.

Here's what the rules and skills system looks like today:

### `.cursor/rules/`

#### Rules (`.mdc` files)

| File                                 | Title                                 | Lines | `alwaysApply` | `globs` | Description                                                                                                                                                                                           |
| ------------------------------------ | ------------------------------------- | ----: | :-----------: | :-----: | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `project-overview.mdc`               | Project Overview                      |    71 |    `true`     |    —    | High-level overview of the HINT (Home Intelligence) project architecture and technology stack.                                                                                                        |
| `software-architecture.mdc`          | Software Architecture                 |   235 |    `true`     |    —    | Software architecture patterns and code organization for the HINT project, including DDD, data access patterns, naming conventions, and directory structure for all three namespaces in the monorepo. |
| `software-development-lifecycle.mdc` | Software Development Lifecycle (SDLC) |   217 |    `true`     |    —    | Step-by-step workflow for planning, implementing, testing, and submitting code changes via pull requests.                                                                                             |
| `planning.mdc`                       | Implementation Planning               |   341 |    `true`     |    —    | Guide for creating implementation plans that follow the SDLC. Reference this when creating plans.                                                                                                     |
| `outcomes.mdc`                       | Outcomes Over Output                  |    95 |    `true`     |    —    | Focus on achieving real outcomes, not just shipping code.                                                                                                                                             |

All five rules have `alwaysApply: true` with no `globs`, meaning they are injected into every conversation regardless of which files are open.

#### Plan Examples (plain `.md` — no frontmatter)

These have no `.mdc` frontmatter (no `alwaysApply`, no `globs`, no `description`). They are referenced by `planning.mdc` as linked files in its markdown body, so they get pulled in when the planning rule is active.

| File                         | Title                            | Lines |
| ---------------------------- | -------------------------------- | ----: |
| `example-sentry-error.md`    | Example: Sentry Error Triage     |   317 |
| `example-backend-bug.md`     | Example: Backend Bug Fix         |   258 |
| `example-ci-optimization.md` | CI Performance Optimization Plan |   203 |

### `.cursor/skills/`

- `alerting/SKILL.md` — Alerting — 53 lines
- `api-design/SKILL.md` — REST API Design Guidelines — 168 lines
- `aws/SKILL.md` — AWS Cloud Infrastructure — 53 lines
- `browsing-app/SKILL.md` — Browsing the App Website — 116 lines
- `database/SKILL.md` — Database & Models — 109 lines
- `development-commands/SKILL.md` — Development Commands Reference — 183 lines
- `error-handling-logging/SKILL.md` — Error Handling & Logging — 149 lines
- `langsmith/SKILL.md` — LangSmith Configuration — 117 lines
- `linear-ticket/SKILL.md` — Linear Tickets — 115 lines
- `package-management/SKILL.md` — Dependencies — 39 lines
- `pull-request/SKILL.md` — Pull Request Descriptions — 188 lines

**Total: 19 files · 3,027 lines**

Here's what each one actually does:

**Rules:**

- **Project Overview** — Defines the system architecture (Qwik web server, FastAPI backend, Celery workers), database ownership boundaries, data flow between components, and package management requirements (pnpm for TS, uv for Python).
- **Software Architecture** — Covers DDD principles, the Unit of Work pattern for DAOs, data access layer conventions (DAOs for TimescaleDB, client classes for Qwik private API), dependency injection, auth patterns, naming conventions, and directory structure for all three namespaces in the monorepo.
- **Software Development Lifecycle (SDLC)** — The end-to-end workflow: check for a Linear ticket, plan the work, follow TDD (red-green-refactor), run code quality checks, browser-test UI changes, create a PR, and wait for CI to go green before considering the work done. After an agent pushes code, it watches CI and explicitly waits for bug bot feedback using sleep-and-poll loops. When bug bot comments appear, the agent reads and addresses them immediately. The bug bot is genuinely good, and having agents respond to that feedback without human intervention saves a lot of time.
- **Implementation Planning** — Structured approach for creating plans. Emphasizes a thorough Phase 0 investigation (reproduce the bug, research prior art, trace the codebase) before proposing solutions, then walks through branching, TDD, local verification, PR creation, and CI monitoring.
- **Outcomes Over Output** — Guiding philosophy: solve the actual problem rather than shipping code to close tickets. Principles include "slow is smooth, smooth is fast," consistency over correctness, do the least work possible, and solve today's problem instead of overengineering for the future. Agents are explicitly instructed that the goal is to solve the actual problem, not ship code to close a ticket. The rule forces them to slow down during investigation, stop if something feels off, prefer consistency over novelty ("if the codebase has 10 patterns done one way, don't introduce an 11th"), bias toward the simplest solution, and verify behavior by observing code running — not just trusting green tests.
- `example-sentry-error.md` (plan example) — A real plan where a Sentry error was triaged end-to-end. Service worker caching caused stale JS bundles after deployments. Shows how to trace through HTTP headers, Qwik's prefetching, and Cloudflare caching.
- `example-backend-bug.md` (plan example) — A real plan for a backend bug where document indexing returned 502s because nginx routed `/private/*` to the wrong port. Also covers fixing error-swallowing anti-patterns and adding Sentry observability.
- `example-ci-optimization.md` (plan example) — A real plan for speeding up CI. E2E tests produced excessive stdout and took too long; fixed by upgrading runners, removing redundant build steps, parallelizing tests, and reducing log verbosity for a 34% reduction in CI time.

**Skills:**

- **Alerting** — Alerting philosophy (proactive over reactive), alert sources (Sentry + CloudWatch), the unified `#alerts` Slack channel, and patterns for monitoring Celery Beat jobs.
- **REST API Design** — Conventions for Qwik (Zod validation, ServiceLocator, response envelope) and FastAPI (Pydantic models, APIRouter). Documents the private APIs for service-to-service communication.
- **AWS Cloud Infrastructure** — Maps AWS resource naming to internal environment names. Lists all ECS clusters/services for production, staging, and eval.
- **Browsing the App** — Step-by-step guide for testing locally: start all dev servers, find the host URL (with DevContainer port-forwarding notes), create test accounts, use test addresses for different utility configurations.
- **Database & Models** — PostgreSQL schema conventions: required columns, UUID vs auto-increment, cascade behavior, soft deletes, JSONB over JSON, Drizzle migration practices, and an explicit "don't add indexes" policy.
- **Development Commands** — Cheat sheet for every dev command across both codebases: testing, linting, formatting, type checking, database operations, dev servers, pre-commit workflow.
- **Error Handling & Logging** — Core rule: don't catch exceptions unless you can recover from them. Proper Sentry integration, log levels, verifying logs by observing them, anti-patterns like bare `except:` and exception swallowing.
- **LangSmith Configuration** — Reference for the LangSmith CLI tool, tracing project names per environment, all evaluation datasets with example counts, common queries for finding errors and slow traces.
- **Linear Tickets** — Template for well-structured tickets: title format, required sections, and the cross-referencing traceability chain from ticket → PR → branch → commit.
- **Dependencies** — Short and strict: always use exact versions, `pnpm add pkg@x.y.z` for TypeScript and `uv add pkg==x.y.z` for Python, never edit manifest files manually.
- **Pull Request Descriptions** — PR title format, branch naming, required sections (Problem, Linear Issue, Sentry Issues, Root Cause, Solution, Changes, To Test). Includes a real-world example.

### TDD enforcement example

One concrete example of enforcement: around TDD, we require agents to verify that tests can actually fail. The agent intentionally breaks the implementation in a way that should cause the test to fail, runs the test to confirm it does fail, then reverts the change and continues with the real implementation. This catches a surprising number of fake-green tests. Agents, like humans, can write tests that always pass. I've seen it happen many times.

This was an iterative process. I made many, many changes to these files every time I noticed an agent not doing something to my liking. This is still happening daily.

## Serial work is unbearable

At first, I ran one agent at a time. I'd plan, watch it work, review output.

> Parallelism collapses when feedback loops are slow.Speed stops being a preference and becomes structural.

It was excruciating. I generally never look at Instagram or X while I'm working, but I found myself picking up my phone while the agent was developing. That was the signal.

So I tried running two agents — one working while I planned with the other.

## The setup wasn't built for parallelism

Everything broke.

> CI didn’t slowly become a problem — it failed almost immediately under load.

Speed stopped being a preference and became a requirement. If feedback loops aren't short, parallelism collapses.

Our devcontainer setup assumed a single instance. We had a hardcoded Docker Compose project name (`name: "hint"`), hardcoded port bindings, and aggressive file watching everywhere. As soon as I tried to run a second devcontainer from a clone of the repo, I hit port conflicts, container name collisions, and Cursor getting confused — sometimes triggering 30+ reinitialization cycles overnight because it couldn't distinguish between two identically-named containers.

Fixing this took real systems work, spread across several PRs. Agents did all of the work to fix these problems (I of course worked with them on the planning), and overall the work was completed in probably 20 minutes of actual working time across a 24-hour period, while working on other stuff. Incredible. It would take me at least a full day debugging and working on all of that myself.

**Dynamic Compose project names.** We removed the hardcoded `name` from `compose.yml`. An `initializeCommand` now generates a `HINT_CLONE_ID` environment variable from the parent directory name and writes it to `.devcontainer/.env`, which Docker Compose uses as the project name. Each clone gets a unique identity: `hint-monorepo-1`, `hint-monorepo-2`, etc.

**Dynamic port forwarding.** We removed all hardcoded Docker port bindings for dev services (Qwik, FastAPI, nginx, pgbouncer). VS Code/Cursor auto-detects and forwards these ports with automatic conflict resolution. We stopped forwarding ports of services like Postgres and Neo4j automatically and only expose the web app and API, since we don't really connect to them from the host machine.

**Unique devcontainer names.** Changed the devcontainer `name` field from a hardcoded string to `"hint-${localWorkspaceFolderBasename}"` so each clone gets its own identity in Cursor.

**Shared CLI credentials.** Added shared Docker volumes for GitHub CLI and Sentry CLI credentials, and later simplified further with a whitelist-based shell script that auto-exports AWS keys, `GH_TOKEN`, and `SENTRY_AUTH_TOKEN` from `.env` files on startup. Single source of truth, zero manual authentication after the devcontainer builds.

**Persistent Cursor plan files.** Added a named volume `cursor-plans-${localWorkspaceFolderBasename}` for `/root/.cursor/plans/` so plan files survive container rebuilds.

**Bind-mount I/O optimization.** This was the biggest one. We were seeing 147–181 GB of block I/O reads per container session. The root cause was three-fold: heavy directories on bind mounts (`node_modules` at 833 MB, `.pnpm-store` at 845 MB, `.venv` at 783 MB — about 2.5 GB total sitting on macOS's VirtioFS/gRPC FUSE layer), multiple file watchers independently scanning everything (Vite, uvicorn, TypeScript, watchmedo, VS Code's file watcher, Pylance, ESLint, Biome), and tool caches causing continuous metadata churn that pushed CPU above 160%.

We fixed this at three layers:

1. **Moved heavy directories to Docker named volumes** — `node_modules`, `.pnpm-store`, and `.venv` came off the bind mount entirely. This alone was the highest-impact change.
2. **Tightened every file watcher** — Added ignore patterns to Vite's `server.watch.ignored`, uvicorn's `--reload-exclude`, TypeScript's `watchOptions.excludeDirectories`, VS Code's `files.watcherExclude`, Pylance (switched to `openFilesOnly`), and both `.eslintignore` and `biome.json`.
3. **Moved tool caches to `/tmp`** — mypy, ruff, and pytest caches went container-local. The tradeoff is they're lost on restart, but they regenerate fast and the I/O savings are significant.

Together, those changes eliminated the majority of blocking I/O and made parallel work stable.

Each devcontainer runs a full, identical stack: Postgres (primary and test), Redis (primary and test), TimescaleDB, Neo4j, FastAPI, Celery workers, and the Qwik/Fastify web server. When I'm running four agents on a machine, I'm running four complete copies of that stack side by side, all isolated from each other. Nothing is shared accidentally. That isolation is what lets agents freely create migrations, reset databases, and run destructive operations without stepping on each other.

To make this concrete, here's a snapshot from `docker stats` and `htop` while four agents are actively working:

[IMAGE: `docker stats` output showing resource usage]

[IMAGE: `htop` output showing CPU and memory distribution]

Devcontainers typically sit around ~6–16% CPU and ~3–6 GB RAM each. Datastores (Postgres, Redis, TimescaleDB, Neo4j) are mostly idle: ~0–1% CPU. Across the machine: roughly ~40–45% of 24 logical CPUs and ~23 GB / ~60 GB RAM.

In practice, I can run meetings, Slack, music — normal work — with all of this running. It's boring, which is exactly what you want.

## Four agents → eight agents

Once that worked, things clicked.

I brought up four devcontainers on one machine and gave each agent a task. While one worked, I planned with the next. I'd spend 5–10 minutes planning with one agent, start the build, then cycle to the next devcontainer and do the same. Agents would run anywhere from 5 minutes to an hour on their tasks.

Then I hit the next bottleneck.

## CI collapsed under parallelism

We went from a few deployments per day to double digits.

> Agents will break things. Humans do too.The goal isn’t perfection — it’s fast detection and recovery.

CI jobs queued up. Fifteen-minute deploys stacked back-to-back. What used to be tolerable suddenly blocked everything. I was looking at around 2 hours for all my changes to deploy one-by-one.

That kicked off another optimization pass. This was all done in another 24-hour period, by agents, across a few pull requests, probably 30 minutes of active work time. Pretty cool. I had agents focus specifically on parallelizing E2E tests, upgrading to 4-core runners, removing redundant build steps, and tightening local checks so failures surfaced before they ever hit CI.

Speed stopped being a preference and became a requirement. If feedback loops aren't short, parallelism collapses.

## Scaling myself

Even with that, I wasn't done.

I added a second machine (a 16" MacBook Pro with an M4 Pro and 64 GB RAM), wired up a KVM switch, and mirrored the setup. My main machine is a Mac Studio with an M3 Ultra and 96 GB RAM.

macOS is amazing at this actually. I don't have to push the button to switch machines. It has a setting that, when I move my cursor to the edge of the screen, it just pops over to the other screen, switching computers, my keyboard, and my trackpad. My MacBook is hooked up to its own monitor and my Studio is hooked up to its own monitor. Incredibly seamless.

Now I run **eight agents in parallel** — four per machine — planning with them, checking in, and shipping continuously.

## Another detour: model choice

At this point, I was running eight agents and the bill started climbing. I had spent $600 over a few days and felt anxious about it.

I tried switching between models constantly. Sonnet here, Opus there. Thinking mode for this, fast mode for that. I'd trim context windows to save a few dollars. It was exhausting and counterproductive.

I stopped optimizing for cost and started optimizing for results. From then on, I left Opus 4.5 with thinking mode and max context on. I let the agents have as much reasoning headroom as they want.

I remember going to my partner Isaiah and saying "I've spent $600 in coding agent inference over the past few days." His response: "And? It's worth it."

That was the epiphany.

I stopped optimizing for cost and started optimizing for results. From then on, I left Opus 4.5 with thinking mode and max context on. I let the agents have as much reasoning headroom as they want.

There's a lot of AI hype on X right now. A new model drops and within hours you see evals, benchmarks, and proclamations about which is "better." None of that has been useful to me. I don't care how well a model draws a pelican (hi, Simon! no shade — I love your work!). I care how it performs inside our codebase, with our rules, our tooling, and our problems.

If I went by my X feed alone, I'd think Codex was the second coming. And I believe people when they say it works well for them — I'm not claiming it's a bad model. But in our environment, in Cursor, with our specific workflow and rules, Opus is consistently better at the work that actually dominates real engineering: investigation, planning, careful tool use, writing tickets, writing pull requests, and following process. Codex may well be better in the Codex macOS or CLI apps — I wouldn't know. But in our work, it's not close.

The winning pattern for us: Opus runs the workflow. For hard technical problems and planning, Opus asks Codex for a second opinion via a simple Python script wrapper around the API. That cross-model critique produces noticeably stronger plans.

The main point: you have to test models in your own codebase, with your own rules and constraints. What you find might surprise you.

I'm now trying out 4.6 and it seems good as well. But I've stopped worrying about the bill. I've said this for years: if your inference bill is low, you're probably not really using intelligence. You're not actually doing AI.

## Another roadblock: team dynamics

Traditional PR review doesn't scale here.

Once I was running eight agents, how on earth could everyone review the code? If everyone runs multiple agents, each person ends up with multiple PRs per day — easily 8 or more. There's no way a team lead can review 40+ pull requests in a day under the traditional model.

So I changed it. The orchestrator is the reviewer. That counts as code review. Experts get pulled in during planning, not after code is written.

For this to work, **95–100% of agent-written code must be mergeable** most of the time. That only happens if planning is excellent. Nothing is worse than getting a big PR back where half the code is wrong. The entire point is to spend all your time on planning so the execution is solid.

## Agents can do bad things

Agents, like humans, make mistakes.

> Once execution became cheap, learning accelerated dramatically.

There is no such thing as a system where things won't go wrong. Humans do things that are wrong all the time. I do wrong things all the time. Our system should reduce that occurrence, but it'll never be zero. The key is to give yourself strong observability and low mean time to resolution.

With full AWS CLI access, agents changed environment variables, merged PRs prematurely, and shipped regressions. The agents broke some of our environments, including features in production. We're rolling out Terraform in the next few days to make infrastructure changes safer, but the core problem was clear: I needed better visibility into what was happening.

Trusting agents like this only works with strong observability.

We invested heavily in making logs readable for humans and agents, emitting CloudWatch metrics for INFO/WARN/ERROR levels, and wiring alarms directly into Slack. Any abnormal behavior shows up quickly in `#alerts`, where we have Slack, Linear, and Cursor bots we can use to respond immediately.

Sentry is everywhere: client, web server, API, workers. Exceptions are formatted clearly and piped into the same alerts channel. Most days, the last 24 hours of production show zero exceptions. When they don't, I hand the issue to an agent and usually have a fix merged within minutes.

We also rely on Sentry traces to catch latency regressions early. Latency is often the first signal of a bug or bad code path.

We track critical metrics on a CloudWatch dashboard: ECS Fargate services, RDS Postgres health, background job completion, scheduled task execution. Problems get piped straight to Slack.

The observability investment makes me more comfortable. Problems will happen, but we know about them quickly and understand exactly what's going on. This paired with beefing up our automated checks (tests, extremely strict linting and type checking, end-to-end tests, etc.) is what gives us comfort.

There is no such thing as a system where things won't go wrong. Humans do things that are wrong all the time. I do wrong things all the time. Our system should reduce that occurrence, but it'll never be zero. The key is to give yourself strong observability and low mean time to resolution. Agents finally make that mean time to recovery single digit minutes in some cases. That has never been the case in my entire experience.

I know we should've had better observability to start. In other more mature systems that I've worked in, we had way better observability. But the point is, lower observability can work in the early days when you have 6 engineers. But it doesn't work when your 6 engineers each have 8 agents.

You need to invest in top-notch observability in the beginning now. It's no longer a later-stage (say Series A) problem.

## What becomes possible

Speed and parallelism are the obvious changes. The more interesting shift is what becomes _feasible_.

> The fun was never really about typing.It was about understanding systems.

How this technology gets used matters a lot. If you're using agents purely to ship more code, I think you're missing the point.

There are plenty of things I would normally avoid because they don't seem worth the time: internal tooling, cleanup projects, observability improvements, performance work, better documentation, more rigorous eval pipelines. With agents, those stop being big decisions and start being small tasks.

One example: I was working on an agent eval and had a dataset in LangSmith. I couldn't quickly find a LangSmith CLI — and Google results for anything LangChain/LangGraph/LangSmith are a mess. I knew there was an API, so I had an agent code up a CLI for our workflow. It took about three minutes. Now agents can search through traces to find examples to add to datasets, change dataset structure, and pull examples for evaluation. You can, in fact, just do things.

The threshold for "is this worth doing" drops dramatically. Refactoring that would take two hours suddenly takes ten minutes. Adding comprehensive logging to a module you've been meaning to instrument? Done in a chat. Building a one-off script to analyze production data patterns? Trivial.

This changes how you think about technical debt and incremental improvements. Things that used to require scheduling, planning meetings, and carving out focused time can now happen in the gaps between other work.

## Where we go from here

This approach works. Now the question is: how do we scale it across the team?

We're rolling this out to all engineers. That means getting engineers more hardware so they can replicate this setup — more RAM, more cores, possibly dedicated machines for running multiple devcontainers. We're also working hard to get cloud agents working properly with our full environment. If we can offload the compute to the cloud, engineers won't need beefy local machines, and we can push parallelism even further.

On the tooling side, we're having agents migrate us to even faster tools. This coming week, agents will switch us from node to bun, prettier/biome to oxfmt, eslint to oxlint, tsc to tsgo, mypy to pyx. We were already using uv and ruff. The goal is quick feedback on ideas and hypotheses. I want everything — formatting, linting, unit tests, E2E tests — to run in roughly two minutes or less.

The faster the feedback loop, the more agents we can run in parallel without getting blocked.

## Where this is going

I could be wrong — I often am — but a few things seem clear.

Engineers are becoming engineering managers of AI teams. Speed matters. Planning matters. Observability matters. And once execution becomes cheap, learning accelerates dramatically.

For the market: I think this will make it genuinely harder for entry-level and junior engineers to get hired. A lot of the work that used to justify those roles can now be done by agents under the supervision of experienced engineers. That's uncomfortable.

There's a paradox, though. If younger engineers don't get hired and don't get experience, who manages all of these agents and autonomous codebases in ten or twenty years? Someone has to learn how systems actually behave in the real world. And younger engineers bring something hard to replace — many of them are willing to try things without fear. They haven't been beaten down by bad SaaS launches, painful outages, or messy cloud migrations. They're not jaded. That lack of baggage is a strength, and every team needs it.

As for the rest of us — we are engineering managers now. We manage teams of AI software engineers.

How this technology gets used matters a lot. If you're using agents purely to ship more code, I think you're missing the point. I don't ship code I don't understand. When agents build something, I ask a lot of questions. Sometimes I worry I'm filling the context window with questions instead of code — but that's actually the best part.

The real silver lining is learning. The fun was never really about typing. It was about understanding systems. With agents, I can learn fifteen new things in a day about software, infrastructure, architecture, and tradeoffs — without waiting on a senior engineer, digging through search results, or trawling Stack Overflow.

My brain feels like a sponge again. I'm a student.
