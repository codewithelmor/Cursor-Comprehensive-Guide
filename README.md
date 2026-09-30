# The Complete Guide to Cursor and Cursor Rules

*Compiled September 30, 2026. Cursor changes fast and repriced itself several times in 2026, so verify pricing and model availability at cursor.com/pricing and cursor.com/docs before committing money or process to anything here. Where sources disagree, this guide says so.*

---

## How this document is organized

**Part I: Overview (sections 1–11)** covers what Cursor is, its features, getting started, pricing, rules fundamentals, rule examples, best practices, troubleshooting, the wider toolbox, alternatives, and a quick checklist.

**Part II: Deep dives (sections 12–26)** goes further on each area: workflows, pricing mechanics, rules architecture, ready-to-copy rule packs, AGENTS.md, Skills, Subagents, Hooks, MCP, Cloud Agents and CLI, Bugbot, security, team rollout, and a troubleshooting playbook.

**Appendices** include a glossary, a file-location cheat sheet, and sources.

### Table of contents

**Part I**
1. What Cursor is
2. Core features
3. Getting started
4. Pricing overview
5. Cursor Rules fundamentals
6. Rule examples
7. Rules best practices
8. Troubleshooting rules
9. The wider toolbox
10. Cursor vs. alternatives
11. New-project checklist

**Part II**
12. Deep dive: daily workflows
13. Deep dive: pricing mechanics and cost control
14. Deep dive: rules architecture
15. Rule packs (ready to copy)
16. Deep dive: AGENTS.md
17. Deep dive: Skills
18. Deep dive: Subagents
19. Deep dive: Hooks
20. Deep dive: MCP
21. Deep dive: Cloud Agents, CLI, and automation
22. Deep dive: Bugbot
23. Security and privacy
24. Team rollout playbook
25. Deep dive: prompting patterns that work
26. Master troubleshooting playbook

**Appendices**: A. Glossary · B. File-location cheat sheet · C. Sources

---

# PART I: OVERVIEW

## 1. What Cursor is

Cursor is an AI-first code editor and agent platform made by Anysphere. Its main product is an AI coding agent and development environment that can search a codebase, edit files, run terminal commands, and carry out multi-step programming tasks from natural-language instructions. It's built on VS Code, so extensions, themes, and keybindings mostly carry over.

The company reportedly passed $3 billion in annual recurring revenue by May 2026.

### How it has evolved

- **2023–24:** VS Code fork with Tab autocomplete, chat, and Cmd+K inline edit.
- **2025:** Agent mode. Cursor 2.0 (October 2025) added parallel agents using git worktrees or remote machines, and Cursor's own Composer model.
- **2026:** Agents for web, mobile, command-line, and cloud environments. Cursor 2.5 (February 2026) let subagents spawn their own subagents asynchronously. Composer 2 shipped in March 2026 and Composer 2.5 later in the year.

---

## 2. Core features

| Feature | What it does | Best for |
|---|---|---|
| **Tab** | Predictive multi-line autocomplete that suggests your next edit | Everyday typing, small refactors |
| **Inline Edit (Cmd/Ctrl+K)** | Edit a selected block with a prompt | Targeted, local changes |
| **Agent** | Autonomous multi-file work: reads code, edits, runs terminal commands | Features, bug fixes, refactors |
| **Plan Mode** | Agent drafts a plan for you to review before writing code (Shift+Tab) | Anything non-trivial |
| **Subagents** | Delegated agents with isolated context | Research, review, parallel exploration |
| **Cloud / Background Agents** | Agents that run remotely and open PRs | Long tasks while you do other work |
| **Bugbot** | Automated PR code review | Catching bugs before merge |
| **CLI** | Cursor agent in your terminal, including headless/CI use | Scripting, automation |
| **MCP** | Connect external tools (databases, Jira, Figma, etc.) | Giving the agent real-world context |
| **Skills, Plugins, Hooks** | Reusable workflows, shareable bundles, scripts at points in the agent loop | Team standardization, guardrails |

Cursor's own models matter for cost. Composer 2.5 is Cursor's first-party model. On Teams plans, Composer and Auto usage draws from a separate pool from third-party models such as Claude, GPT, and Gemini, so defaulting to Composer can noticeably reduce third-party credit consumption.

### The Customize area

Recent versions put everything configurable under **Customize** in the sidebar: Plugins, Rules, Skills, Subagents, Hooks, and MCP. Rules are one piece of a larger system.

---

## 3. Getting started

1. Download Cursor from cursor.com and sign in. You can import VS Code settings on first launch.
2. Open a project folder. Cursor indexes the codebase so the agent can search it semantically.
3. Try Tab on a file you're editing.
4. Open Agent and give it a task. Start in Plan Mode for anything bigger than a small fix.
5. Review the diff, then accept or reject each change.
6. Turn on **Privacy Mode** if you work on sensitive code. With it enabled, Cursor guarantees your code data isn't used for training by Cursor or its model providers.

### Prompting habits that work

- **Plan first, then execute.** Cursor's own guidance leans heavily on planning before coding.
- **Give the agent a verifiable goal**, such as failing tests or a type-check that must pass.
- **Use @-mentions** (`@file`, `@folder`, `@rule-name`) to point at exact context.
- **Start a new chat per task.** Long sessions accumulate noise and degrade results.
- **Encode repeated corrections as rules.**

---

## 4. Pricing overview (as of September 30, 2026)

Cursor rebuilt its pricing several times this year. Most plan tables online describe a product that no longer exists. Check cursor.com/pricing before buying.

| Plan | Price | Notes |
|---|---|---|
| **Hobby** | Free | Limited usage, good for trying it out |
| **Pro** | $20/mo | Unlimited Tab and Auto, plus $20 of frontier-model usage at API prices |
| **Pro+** | $60/mo | Buys about $70 of third-party model usage (roughly 3x Pro) |
| **Ultra** | $200/mo | Buys about $400 of usage (roughly 20x Pro) plus priority access to new features |
| **Teams Standard** | $40/user/mo | Shared billing, analytics, privacy mode, SSO |
| **Teams Premium** | $120/user/mo ($96 annual) | Five times Standard's included usage |
| **Enterprise** | Custom | Invoicing, pooled usage, advanced security |
| **India "Start"** | ₹649/mo | Regional plan, India only |

Key mechanics:

- You buy a credit pool, not a request count. The difference between Pro, Pro+, and Ultra is pool size, not features.
- Usage is metered at model API prices, so expensive frontier models drain a pool much faster than Cursor's own models.
- On-demand usage lets you continue past your included amount, billed in arrears.
- Flat-rate Auto was retired on August 24, 2026. Each Auto request now bills at the list price of the model it routes to.
- Teams and Enterprise pay an extra $0.25 per million tokens on third-party models.
- Annual billing takes 20% off paid tiers.
- Students: free-year offers have come and gone. As of September 2026 there's no standing offer, so check cursor.com/students.
- Bugbot: older articles say it's a $40/seat add-on, but that stopped being its pricing model in May 2026. Check the current Bugbot page.

---

## 5. Cursor Rules fundamentals

Language models don't remember anything between sessions. Rules supply persistent, reusable instructions, and their contents are placed at the start of the model's context when they apply.

### The four kinds of rules

| Type | Location | Scope |
|---|---|---|
| **Project Rules** | `.cursor/rules/*.mdc` | One repo, version-controlled |
| **User Rules** | Customize → Rules | All your projects, personal |
| **Team Rules** | Cursor dashboard (Teams/Enterprise) | Whole organization |
| **AGENTS.md** | Repo root or subfolders | Plain-markdown alternative to Project Rules |

When they overlap, precedence runs **Team Rules → Project Rules → User Rules**. All applicable rules are merged; earlier sources win conflicts.

### Anatomy of a Project Rule

```markdown
---
description: Conventions for React components
globs: src/components/**/*.tsx
alwaysApply: false
---

- Use named exports, never default exports
- Props interface goes at the top of the file
- Keep components under 200 lines; extract subcomponents beyond that

@component-template.tsx
```

### The four activation modes

| Mode (UI name) | `alwaysApply` | `description` | `globs` | When it loads |
|---|---|---|---|---|
| **Always Apply** | `true` | ignored | ignored | Every chat |
| **Apply to Specific Files** | `false` | optional | set | When a matching file is in context |
| **Apply Intelligently** | `false` | set | omitted | When the agent judges the description relevant |
| **Apply Manually** | `false` | omitted | omitted | Only when you type `@rule-name` |

### Glob patterns

| Pattern | Matches |
|---|---|
| `*.ts` | `.ts` files in the root only |
| `**/*.ts` | All `.ts` files anywhere |
| `src/**/*.tsx` | All `.tsx` files under `src/` |
| `docs/**/*.md, docs/**/*.mdx` | Multiple patterns, comma-separated |
| `tailwind.config.*` | Config file with any extension |

**A common trap:** the docs say to separate multiple patterns with commas. Community reports say YAML array syntax and quoted strings may silently fail to attach the rule. Write `globs: src/**/*.ts, src/**/*.tsx` as a bare, comma-separated string. Cursor gives no warning when a glob matches nothing.

### Creating rules

- **`/create-rule` in Agent chat:** describe what you want and the agent writes the file with correct frontmatter.
- **Customize → Rules → Add Rule** in the sidebar.
- **By hand:** create the `.mdc` file yourself.

A plain `.md` file inside `.cursor/rules` is ignored: no error, it just isn't treated as a rule.

### What rules do *not* affect

Rules don't affect Cursor Tab or other AI features. User Rules don't apply to Inline Edit (Cmd/Ctrl+K); they're used by Agent chat. Bugbot also ignores `.cursor/rules/*.mdc` (it has its own rules, see section 22).

---

## 6. Rule examples

### Always-on core rule (keep it short)

`.cursor/rules/core.mdc`
```markdown
---
alwaysApply: true
---

- Before editing unfamiliar code, read the surrounding files first
- Never edit generated output in `dist/` or `build/`
- Run `npm run typecheck` and `npm test` after changes; fix failures before finishing
- Ask before adding a new dependency
- Prefer small, focused diffs over sweeping rewrites
```

### File-scoped rule

`.cursor/rules/api-routes.mdc`
```markdown
---
globs: src/api/**/*.ts
alwaysApply: false
---

- Validate all request input with Zod at the route boundary
- Derive TypeScript types from the Zod schema; don't hand-write duplicates
- Return errors as `{ code, message }` objects, never raw strings
- Reference the canonical example: @src/api/users/route.ts
```

### Intelligently applied rule

`.cursor/rules/database-migrations.mdc`
```markdown
---
description: Use when creating or modifying database migrations or schema changes
alwaysApply: false
---

- Every migration needs both `up` and `down` steps
- Never change a column type in place: add a new column, backfill, then drop the old one in a later migration
- Test migrations against a copy of production-shaped data
```

The `description` is what the agent reads to decide whether to load the rule, so write it as a trigger condition ("Use when…"), not a vague label.

### Manual-only rule

`.cursor/rules/release-checklist.mdc`
```markdown
---
alwaysApply: false
---

When preparing a release:
1. Bump the version in package.json
2. Update CHANGELOG.md from merged PR titles
3. Run the full test suite and build
4. Summarize anything risky for the release notes
```

Invoke by typing `@release-checklist` in chat.

### AGENTS.md

```markdown
# Project Instructions

## Stack
- TypeScript, React 19, Node 22, PostgreSQL

## Conventions
- Functional components only
- snake_case for database columns, camelCase in code
- Business logic lives in `src/services/`, not in route handlers

## Commands
- `npm run dev` starts the dev server
- `npm test` runs the suite
```

### User Rule

Set in Customize → Rules:
```
Be concise. Skip preamble and summaries. When you change code, list the files touched and why in one line each.
```

### Recommended layout

```
.cursor/rules/
  core.mdc                  # alwaysApply, under 20 lines
  frontend/
    components.mdc          # globs: src/components/**
    styling.mdc             # globs: **/*.css, tailwind.config.*
  backend/
    api-routes.mdc          # globs: src/api/**
    database.mdc            # description-based
  workflows/
    release-checklist.mdc   # manual
AGENTS.md                   # cross-tool basics
```

---

## 7. Rules best practices

Cursor's official guidance:

- **Keep each rule under 500 lines** and split big ones into composable pieces.
- **Include concrete examples or reference real files.** Point at files with `@` instead of pasting their contents, so rules don't go stale.
- **Write like clear internal docs.** Vague advice ("write clean code") does nothing.
- **Start small.** Add a rule only when the agent makes the same mistake repeatedly.
- **Commit rules to git** so the team shares them.

**Leave out:** full style guides (use a linter), documentation of common commands (the agent knows npm, git, pytest), rare edge cases, and copies of code that already exists in the repo.

**Additional practical advice:**

- **Don't let `alwaysApply` sprawl.** Every rule someone marks "important" becomes always-on, and you end up with a huge implicit prompt spread across files nobody has seen in total. Aim for a handful of short always-on rules and use globs for the rest.
- **Audit globs after restructuring.** They match against the repo as it exists today.
- **Write positive instructions** ("Use Zod for validation") rather than long lists of prohibitions.
- **Encode your review comments.** If you leave the same PR comment three times, it belongs in a rule. You can tag `@cursor` on a GitHub issue or PR to have the agent update the rule.

### Team Rules

Admins on Teams and Enterprise create rules in the dashboard and choose whether each is enforced. Enforced rules can't be turned off by individual users. Team Rules are free-form text (no folder structure) but support glob patterns. The docs caution that AI guidance shouldn't be your only security control.

### Sharing rules across repos

Rules can't be imported directly from a GitHub repo. Package them into a plugin, publish it through a marketplace (the repo needs a `.cursor-plugin/marketplace.json`), and install the plugin. The rules then appear in Customize.

---

## 8. Troubleshooting rules that don't fire

1. **File extension:** is it `.mdc`, inside `.cursor/rules/`?
2. **Frontmatter delimiters:** does the file open and close frontmatter with `---`?
3. **Rule type matches the fields:** intelligent rules need a `description`, file-scoped rules need `globs`.
4. **Glob syntax:** bare comma-separated string; confirm the path exists.
5. **Is the file in context?** Glob rules only attach when a matching file is part of the conversation.
6. **Check Customize → Rules** for each rule's status.
7. **Rule too long or contradictory?** Trim it.
8. **Test in a new chat.**

---

## 9. The wider toolbox (summary)

- **MCP servers** connect the agent to outside systems; project config lives in `.cursor/mcp.json`.
- **Skills** are folders with a `SKILL.md` teaching a repeatable workflow.
- **Subagents** run tasks in isolated context (`.cursor/agents/`).
- **Hooks** are scripts that run at points in the agent loop (`.cursor/hooks.json`).
- **Bugbot** reviews pull requests automatically.
- **Cloud agents** run remotely and can open PRs.
- **Plugins** bundle rules, skills, and more so teams can share them.

Rules vs. skills vs. AGENTS.md: *Rules* are always-available conventions, scoped by file or relevance. *Skills* are step-by-step procedures loaded on demand. *AGENTS.md* is simple, portable, tool-agnostic instructions.

---

## 10. Cursor vs. alternatives

- **Claude Code / Codex CLI:** terminal-first agents, strong for autonomous long-running work, but no visual editor or Tab. Many developers use Cursor for interactive editing alongside one of these for delegated tasks.
- **GitHub Copilot:** cheaper and tightly tied to GitHub and VS Code, historically less agent-centric.
- **Windsurf and other AI IDEs:** similar concept; try each on your own codebase.

Honest tradeoff: Cursor is the most polished all-in-one, but usage-based pricing means heavy agent use can cost more than the sticker price suggests.

---

## 11. New-project checklist

- [ ] Create `.cursor/rules/core.mdc` with 5 or fewer always-on lines
- [ ] Add `AGENTS.md` with stack, commands, and conventions
- [ ] Add file-scoped rules for main areas (frontend, API, DB)
- [ ] Turn on Privacy Mode if the code is sensitive
- [ ] Set a spend alert or check the usage dashboard weekly
- [ ] Default to Plan Mode for multi-file tasks
- [ ] Add a rule each time you correct the agent for the same thing twice
- [ ] Commit `.cursor/` to git


---

# PART II: DEEP DIVES

## 12. Deep dive: daily workflows

### 12.1 The core loop

Every productive Cursor session follows roughly the same loop:

1. **Frame** the task in one or two sentences with a clear done-condition.
2. **Plan** (Plan Mode, Shift+Tab in Agent). Read the plan and correct it before any code is written.
3. **Execute** with Agent, ideally with a verifiable target (tests, typecheck, lint).
4. **Review** the diff yourself. Agent Review and Bugbot are additional layers, not replacements.
5. **Capture** anything you had to correct as a rule, skill, or AGENTS.md line.

The agent-facing docs also list dedicated modes and surfaces worth knowing: **Agents Window** (manage multiple agents), **Projects**, **Agent Review**, **Planning**, **Debugging** (Debug mode), and **Design Mode**. Interfaces evolve quickly, so treat menu names as approximate and use `Cmd/Ctrl+Shift+P` to search commands.

### 12.2 Choosing the right surface

| Situation | Use | Why |
|---|---|---|
| Finishing a line or function | **Tab** | Fastest, cheapest, no context switch |
| Rewriting one selected function | **Inline Edit (Cmd/Ctrl+K)** | Scoped, minimal blast radius |
| New feature across several files | **Agent + Plan Mode** | Needs exploration and coordination |
| "Where is X handled?" | **Agent or explore subagent** | Search-heavy, keeps main context clean |
| Mysterious runtime bug | **Debug mode** | Instrumentation-driven investigation |
| Long, well-specified task | **Cloud Agent** | Runs while you do other things |
| Every PR | **Bugbot** | Consistent second pair of eyes |
| Scripted or CI usage | **CLI (headless)** | Automation |

### 12.3 Plan Mode in practice

Plan Mode makes the agent research your codebase and propose an approach before touching files. Good practice:

- Ask it to list **files it will touch** and **risks**.
- Ask for **alternatives** when the task is architectural ("give me two approaches and tradeoffs").
- Edit the plan directly. Deleting a step is cheaper than reverting code.
- For large plans, ask it to **split into milestones** that each leave the repo green.

Prompt template:

```
Plan (don't implement yet):
Goal: <one sentence>
Constraints: <stack, patterns to follow, things not to touch>
Done when: <observable checks>
Output: files to change, step order, risks, and how we'll verify.
```

### 12.4 Test-driven agent loops

Tests give the agent an objective it can check itself. A reliable pattern:

1. Have the agent write failing tests from your spec (review them; they define the behavior).
2. Commit the tests.
3. Tell the agent to make them pass **without modifying the tests**.
4. Run the full suite plus lint and typecheck.

Add a one-line always-on rule such as "Run `npm test` before declaring a task done" so you don't have to repeat it.

### 12.5 Managing context

Context is the currency of agent quality.

- **@-mention specifically.** `@src/api/users/route.ts` beats "the users route."
- **Fresh chat per task.** Old conversation turns keep consuming context and can mislead.
- **Move discoveries into files.** If the agent spent ten minutes learning how your auth works, have it write a short `docs/auth.md`, then reference that file in future prompts or rules.
- **Use subagents for noisy work.** Codebase-wide searches and log analysis belong in isolated context.
- **Don't paste giant logs.** Point at a file or summarize the relevant 30 lines.

### 12.6 Working in parallel

Cursor 2.0 introduced parallel agents using git worktrees. Each agent works on its own copy of the repo, which avoids stomping on each other. Two cautions:

- **Assign file ownership up front.** Parallel agents touching the same files create merge conflicts when you integrate.
- **Parallelism multiplies cost.** N agents burn roughly N times the credits. Use it when the tasks are truly independent.

A common comparison workflow: run the same task with two different models and keep the better result. It's cost-heavy but useful for high-stakes changes.

### 12.7 Review discipline

Agents produce plausible-looking code quickly, which makes review the bottleneck. Suggested habits:

- Read diffs file by file; don't accept-all blindly.
- Ask the agent to **explain non-obvious changes** and to point out anything it wasn't sure about.
- Look specifically for: silently changed tests, new dependencies, swallowed errors, broad `try/catch`, hard-coded values, and edits to files you didn't expect.
- Keep commits small so bad changes are easy to revert.

### 12.8 Prompt anti-patterns

| Anti-pattern | Better |
|---|---|
| "Make it better" | "Reduce duplication in X; keep public API unchanged; tests must pass" |
| Mega-prompt with five unrelated tasks | One task per chat |
| Pasting 2,000 lines of code | `@`-mention the files |
| "Fix the bug" with no repro | Repro steps, expected vs. actual, failing test |
| Arguing with the agent for 20 turns | Restart with a better prompt and what you learned |

---

## 13. Deep dive: pricing mechanics and cost control

### 13.1 What you are actually buying

Cursor's paid tiers sell **usage pools** priced at model API rates. Three ideas matter:

1. **Pool size:** what your monthly fee covers. Per the pricing sources: Pro buys about $20 of third-party usage, Pro+ about $70, Ultra about $400. Pro is the only tier where the fee and included usage match; Pro+ and Ultra give more than you pay.
2. **Model rate:** each token is billed at the routed model's price, so a hard task on an expensive frontier model can consume in minutes what a cheap model would take hours to.
3. **Overage:** after the pool is spent you either stop or pay on-demand at the same rates, billed in arrears.

Tab is treated separately (unlimited on paid plans). Auto is no longer flat; since August 24, 2026 it bills at the list price of whichever model it picks.

### 13.2 Two pools

Since the 2026 changes, usage is split between:

- **Cursor models pool** (Composer, including Composer 2.5, and Auto routed to Cursor models). Reported rates for Composer 2.5 are cheaper than the old flat Auto rate (about $0.50 input / $2.50 output per million tokens according to one source).
- **Other models pool** (third-party models such as Claude, GPT, Gemini). Substantially more expensive per token.

On Teams and Enterprise there is a further **Cursor Token Rate** surcharge of $0.25 per million tokens on third-party usage, which also applies to bring-your-own-key usage.

Because dashboards were adjusted during the rollout, the exact pool sizes you see may differ from what articles quote. Your own dashboard is the source of truth.

### 13.3 Teams details

- **Standard** ($40/user/month; $32 annual) and **Premium** ($120/user/month; $96 annual). Premium includes five times Standard's usage at three times the price, so it only makes sense for heavy users.
- Seats can be mixed freely. Assign Premium to the few people who exhaust Standard.
- Admins get usage analytics, spend alerts (dollar thresholds routed to Slack or email), privacy mode controls, and SSO.
- The June 2026 update reportedly aimed to lower costs for most teams; renewing customers moved over July 1, 2026.

### 13.4 Illustrative scenarios

These use round numbers to show the logic. They are **not** quotes; real consumption depends on models and context size.

**Scenario A: Occasional user.** Uses Tab all day and Agent a few times a week for small edits, mostly on Auto/Composer. Pro at $20 is plenty; upgrading wastes money.

**Scenario B: Daily agent user.** Runs several multi-file agent tasks per day on frontier models. Pro's pool disappears mid-month and overage accumulates. If your overage regularly hits $20–40, Pro+ ($60) is usually cheaper than paying overage on Pro.

**Scenario C: Agent power user.** Runs parallel agents and cloud agents most of the day. Pro+ feels tight; Ultra's much larger pool (and priority access) can pay off, but only if you actually use it. Otherwise you're paying $200 for unused headroom.

**Scenario D: Five-person startup.** Four Standard seats plus one Premium seat for the heaviest user, with spend alerts set. Compare against giving everyone Pro individually: Teams adds centralized billing, privacy enforcement, and analytics.

### 13.5 Decision guide

```
Just exploring?                       → Hobby
Working developer, moderate agent use → Pro (watch usage for 2 billing cycles)
Regular overage above ~$20–40/mo      → Pro+
Overage still large on Pro+           → Ultra (or mix Cursor + a cheaper API for batch work)
3+ developers                         → Teams (Standard, add Premium for heavy users)
Compliance / invoicing / custom SSO   → Enterprise
```

### 13.6 Cost-control playbook

1. **Default to Composer/Auto** for routine work; reserve frontier models for genuinely hard problems.
2. **Plan first.** A wrong turn on an expensive model is the most common budget leak.
3. **Keep context tight.** Every extra file in context is paid for on every turn.
4. **Fresh chats.** Long conversations re-send history repeatedly.
5. **Use cheap models in subagents** for search and exploration (the built-in explore subagent uses a faster model by default).
6. **Avoid parallel agents** unless tasks are independent and valuable.
7. **Set spend alerts** and check the dashboard weekly.
8. **Watch cloud agents and Bugbot Autofix.** Autofix spawns cloud agents and bills at your plan's rates; it also requires on-demand usage to be enabled.
9. **Prefer rules over repetition.** A short rule saves tokens in every future prompt.
10. **Beware long always-on rules.** They are re-sent on every request.

### 13.7 Pricing FAQ

**Is there a free tier?** Yes, Hobby, with limited usage.

**Do features differ between Pro, Pro+, and Ultra?** Mostly no. The tiers differ by pool size (Ultra also gets priority access to new features).

**Does Auto save money?** It can, but it is no longer flat-rate, so it's only cheap when it routes to cheaper models.

**Can I bring my own API key?** Reportedly yes on some plans; on Teams and Enterprise the Cursor Token Rate still applies. Check current docs.

**Annual discount?** 20% off paid tiers.

**Student discount?** No standing offer as of September 2026; check cursor.com/students.

**Why do articles disagree on numbers?** Cursor changed pricing several times in 2026 and adjusted pool sizes during rollouts. Rely on cursor.com/pricing and your dashboard.

---

## 14. Deep dive: rules architecture

### 14.1 Mental model

Think of rules as **prompt fragments with routing logic**. Each rule is a small prompt; frontmatter decides when it's injected. Your goal is the minimum amount of always-present context plus precisely targeted extra context.

```
Always-on (tiny)        → non-negotiables and commands
File-scoped (globs)     → conventions for specific areas
Intelligent (description) → topic knowledge the agent pulls when relevant
Manual (@mention)       → templates, checklists, rare workflows
```

### 14.2 Choosing an activation mode

| Question | If yes → |
|---|---|
| Must this apply to literally every task? | **Always** (keep it to a few lines) |
| Does it apply whenever certain files are touched? | **Globs** |
| Is it topic knowledge the agent can recognize from a description? | **Intelligent** |
| Is it a template or a rarely used checklist? | **Manual** |

Decision heuristics:

- If you're tempted to mark something `alwaysApply` because "it's important," ask whether a glob captures it. Importance is not the criterion; universality is.
- Globs are deterministic. Descriptions rely on model judgment. Prefer globs where file paths reliably identify the domain.
- Use intelligent rules for cross-cutting topics with no clear path (security review, error handling philosophy, performance).

### 14.3 The three-layer stack

| Layer | Owns | Example |
|---|---|---|
| **User Rules** | Your personal style | "Be concise; no filler" |
| **Project Rules / AGENTS.md** | Repo-specific conventions | "Use Zod at API boundaries" |
| **Team Rules** | Org-wide standards | "Never log PII; use the shared logger" |

Precedence is Team → Project → User, with earlier sources winning conflicts. Practical implication: don't put personal preferences in project rules, and don't rely on user rules for anything the team must follow.

### 14.4 Writing rules that change behavior

**Be specific and checkable.**

| Weak | Strong |
|---|---|
| "Write clean code" | "Functions over 40 lines must be split" |
| "Handle errors properly" | "Wrap external calls; return `{code, message}`; never throw strings" |
| "Follow our patterns" | "Follow the pattern in @src/services/orders.ts" |
| "Use good naming" | "Booleans start with `is`/`has`/`should`" |

**Include the why for surprising rules.** A one-line reason ("legacy clients break on null, so return empty arrays") helps the agent generalize correctly and stops it "fixing" your rule.

**Show one canonical example** via `@file` rather than pasting code. It stays current as the code evolves.

**State commands explicitly** when they're non-obvious (custom scripts, monorepo filters): "Run tests with `pnpm --filter web test`".

**Order by importance** and keep bodies short. Long rules dilute attention.

**Prefer positive instructions**, and pair any prohibition with the alternative: "Don't use `any`; use `unknown` and narrow."

### 14.5 Size and budget

- Official guidance: under 500 lines per rule. In practice, aim for **under 50 lines** for most rules and **under 20** for always-on rules.
- Set an informal budget: total always-on content under about 50 lines across the project.
- Split by concern: one rule per topic, not one giant `standards.mdc`.

### 14.6 Referencing files and other rules

Inside a rule body, `@filename.ts` includes that file's content in the rule's context. In chat, `@rule-name` applies a rule manually. Reference instead of copy: it prevents drift and keeps rules short.

### 14.7 Rules in monorepos

Two approaches:

1. **Root `.cursor/rules/` with path-based globs** (`apps/web/**`, `packages/api/**`). Simple, one place to look.
2. **Nested `.cursor/rules/` or nested `AGENTS.md` per package.** Subdirectories can carry their own rules, and nested AGENTS.md files are combined with parent directories, with the more specific one taking precedence.

Keep root rules for repo-wide facts (package manager, CI commands, commit style) and package-level rules for language- or framework-specific behavior.

### 14.8 Migrating from `.cursorrules`

Older projects used a single `.cursorrules` file at the repo root. The modern format is `.cursor/rules/*.mdc`. To migrate:

1. Read the old file and group instructions by topic.
2. Create one `.mdc` per topic; decide each rule's activation mode.
3. Move truly universal lines into a short always-on `core.mdc`.
4. Ask Agent: "Split my .cursorrules into `.cursor/rules/*.mdc` files with proper frontmatter, keeping under 50 lines each" and review the result.
5. Delete the old file once you've verified behavior in a fresh chat.

### 14.9 Testing and maintaining rules

Rules fail silently, so test them:

- **Smoke test:** open a fresh chat, reference a file matching the glob, and ask "What conventions apply here?" A working rule shows up in the answer.
- **Behavioral test:** ask for a small change that would violate the rule if it were ignored.
- **Glob audit:** after refactors, list your rules' globs and confirm each still matches real files. A simple script that parses `.mdc` frontmatter and checks each glob against `git ls-files` catches drift.
- **Review cadence:** review rules quarterly. Delete any that the agent now follows unprompted.
- **Change control:** treat `.cursor/` like code: PR review, blame history, CODEOWNERS.

### 14.10 Rule anti-patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| 900 lines of always-on rules | Diluted attention, wasted tokens on every request | Split; convert to globs |
| Pasting the whole style guide | Agent already knows conventions; linter is better | Configure ESLint/Prettier/Ruff |
| Contradictory rules across files | Unpredictable behavior | One source of truth per topic |
| Rules that restate the obvious | Noise | Delete |
| Rules with stale code snippets | Agent copies outdated patterns | `@`-reference real files |
| No description on intelligent rules | Never triggers | Write "Use when…" descriptions |
| Wrong extension (`.md`) | Silently ignored | Rename to `.mdc` |
| Array/quoted globs | May silently not match | Bare comma-separated string |
| Relying on rules for security | Not a control | Use hooks, permissions, CI |

### 14.11 Rule governance

Rules are a shared, executable-ish artifact, so govern them:

- Add `.cursor/rules/` to CODEOWNERS.
- Require a PR description saying **what mistake this rule prevents**.
- Keep an `README` in the rules folder explaining the taxonomy (which rules are always-on, which are file-scoped).
- Use Team Rules with enforcement only for genuinely organization-wide standards.

---

## 15. Rule packs (ready to copy)

These are original starter packs. Adapt commands, paths, and libraries to your repo. In each pack, one or two files are always-on and everything else is scoped.

### 15.1 Pack: Next.js + TypeScript + Tailwind

`.cursor/rules/core.mdc`
```markdown
---
alwaysApply: true
---

- Stack: Next.js (App Router), TypeScript strict, Tailwind, Prisma
- Package manager: pnpm. Never use npm or yarn commands
- After changes run: `pnpm typecheck && pnpm lint && pnpm test`
- Ask before adding dependencies
- Small diffs; do not reformat unrelated code
```

`.cursor/rules/next-app-router.mdc`
```markdown
---
globs: src/app/**/*.ts, src/app/**/*.tsx
alwaysApply: false
---

- Components are Server Components by default; add `"use client"` only when state, effects, or browser APIs are required
- Fetch data in Server Components or route handlers, not in `useEffect`
- Mutations go through Server Actions or route handlers with Zod validation
- Colocate `loading.tsx` and `error.tsx` with routes that fetch data
- Never import server-only modules (db, secrets) into client components
```

`.cursor/rules/react-components.mdc`
```markdown
---
globs: src/components/**/*.tsx
alwaysApply: false
---

- Named exports only; one component per file
- Props typed with an interface declared at the top
- No prop drilling beyond two levels; use composition or context
- Accessibility: interactive elements are real buttons/links, images have alt text, inputs have labels
- Follow the structure in @src/components/ui/button.tsx
```

`.cursor/rules/tailwind-styling.mdc`
```markdown
---
globs: **/*.tsx, tailwind.config.*
alwaysApply: false
---

- Use Tailwind utilities; no inline `style` except for dynamic values
- Compose conditional classes with the `cn()` helper, not string concatenation
- Use design tokens from the Tailwind config; do not hard-code hex colors
- Mobile-first: base styles are mobile, add `sm:`/`md:`/`lg:` overrides
```

`.cursor/rules/prisma-db.mdc`
```markdown
---
description: Use when editing Prisma schema, migrations, or database access code
alwaysApply: false
---

- Access the database only from `src/server/` modules, never from components
- Schema changes require a migration; never edit applied migrations
- Select only needed fields; avoid unbounded `findMany` (always paginate)
- Wrap multi-step writes in `prisma.$transaction`
```

### 15.2 Pack: Python + FastAPI

`.cursor/rules/core.mdc`
```markdown
---
alwaysApply: true
---

- Python 3.12, FastAPI, SQLAlchemy 2.x, Pydantic v2, pytest
- Use `uv` for dependency management
- After changes run: `ruff check . && ruff format --check . && mypy . && pytest -q`
- Ask before adding dependencies
- Type hints required on all function signatures
```

`.cursor/rules/api-layer.mdc`
```markdown
---
globs: app/api/**/*.py
alwaysApply: false
---

- Route handlers stay thin: parse input, call a service, return a response model
- Every endpoint declares `response_model` and explicit status codes
- Use dependency injection (`Depends`) for DB sessions and auth
- Raise `HTTPException` only in the API layer; services raise domain exceptions
- Reference example: @app/api/users.py
```

`.cursor/rules/services-and-db.mdc`
```markdown
---
globs: app/services/**/*.py, app/models/**/*.py
alwaysApply: false
---

- Business logic lives in services, never in routes or models
- Use SQLAlchemy 2.x style (`select()`), not legacy `Query`
- Sessions are passed in; services do not create their own
- Avoid N+1 queries: use `selectinload`/`joinedload` deliberately
- Alembic migration for every schema change
```

`.cursor/rules/tests.mdc`
```markdown
---
globs: tests/**/*.py
alwaysApply: false
---

- pytest with fixtures; no test depends on execution order
- Use factories for data; do not share mutable module-level state
- Arrange / Act / Assert with blank lines between sections
- Mock only at system boundaries (HTTP, clock, external services)
- Name tests `test_<unit>_<behavior>_<condition>`
```

### 15.3 Pack: Node/Express or NestJS API

`.cursor/rules/core.mdc`
```markdown
---
alwaysApply: true
---

- Node 22, TypeScript strict, Vitest
- After changes run: `npm run typecheck && npm test`
- Never log secrets or PII; use the shared logger in `src/lib/logger.ts`
- Configuration comes from validated env (`src/config.ts`), never `process.env` directly
```

`.cursor/rules/http-layer.mdc`
```markdown
---
globs: src/routes/**/*.ts, src/controllers/**/*.ts
alwaysApply: false
---

- Validate body, params, and query with Zod before use
- Controllers call services; no business logic or DB access in controllers
- Errors flow through the central error middleware as typed `AppError`
- Follow RESTful naming and consistent status codes (201 create, 204 delete)
```

### 15.4 Pack: React Native / Expo

`.cursor/rules/core.mdc`
```markdown
---
alwaysApply: true
---

- Expo (managed workflow), TypeScript strict, Expo Router
- Run `npx expo lint` and `npx tsc --noEmit` after changes
- Ask before installing native modules (they may require a dev-client rebuild)
```

`.cursor/rules/rn-ui.mdc`
```markdown
---
globs: app/**/*.tsx, src/components/**/*.tsx
alwaysApply: false
---

- Use `StyleSheet.create` or the project styling library; no inline style objects in render
- Long lists use `FlatList`/`FlashList` with stable `keyExtractor`
- Respect safe areas; test on both iOS and Android sizing
- Avoid inline anonymous functions in list item props
```

### 15.5 Pack: Cross-cutting rules (any stack)

`.cursor/rules/git-and-prs.mdc`
```markdown
---
description: Use when committing, writing PR descriptions, or preparing changes for review
alwaysApply: false
---

- Conventional Commits: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`
- One logical change per commit; subject under 72 characters
- PR description: what changed, why, how it was tested, and risks
- Never commit secrets, `.env` files, or generated build output
```

`.cursor/rules/security-review.mdc`
```markdown
---
description: Use when handling authentication, authorization, user input, file uploads, or secrets
alwaysApply: false
---

- Validate and sanitize all external input at the boundary
- Use parameterized queries only; never build SQL by string concatenation
- Check authorization on the server for every protected action, not just authentication
- Never hard-code secrets; read from environment/secret manager
- Do not log tokens, passwords, or personal data
```

`.cursor/rules/error-handling.mdc`
```markdown
---
description: Use when adding error handling, retries, or logging around external calls
alwaysApply: false
---

- Catch errors at boundaries and convert to typed domain errors
- Never swallow errors silently; log with context and rethrow or return a failure
- Retries only for idempotent operations, with backoff and a cap
- User-facing messages are generic; details go to logs
```

`.cursor/rules/testing.mdc`
```markdown
---
globs: **/*.test.ts, **/*.test.tsx, **/*.spec.ts
alwaysApply: false
---

- Test behavior, not implementation details
- One assertion theme per test; descriptive names
- No network or real time in unit tests; inject clocks and fakes
- Never weaken or delete a failing test to make it pass; fix the code or explain why the test is wrong
```

`.cursor/rules/rule-authoring.mdc` (meta-rule so the agent writes good rules)
```markdown
---
description: Use when creating or editing Cursor rules, AGENTS.md, or skills
alwaysApply: false
---

- Rules are `.mdc` files in `.cursor/rules/` with frontmatter (`description`, `globs`, `alwaysApply`)
- Globs are a bare comma-separated string, not a YAML list
- Keep rules under 50 lines and focused on one topic
- Reference real files with `@path` instead of pasting code
- Intelligent rules need a "Use when…" description
```

### 15.6 Pack: Docs and content repos

`.cursor/rules/docs-style.mdc`
```markdown
---
globs: docs/**/*.md, docs/**/*.mdx
alwaysApply: false
---

- Sentence-case headings; second person ("you"); active voice
- Every page starts with a one-paragraph summary of what the reader will accomplish
- Code blocks specify a language and are runnable as written
- Link to related pages rather than repeating content
```

### 15.7 A short always-on baseline for any repo

If you only add one file, add this (edit the commands):

```markdown
---
alwaysApply: true
---

- Read relevant existing code before changing it; match surrounding patterns
- Run the project's tests, lint, and typecheck before declaring done
- Do not add dependencies or change unrelated files without asking
- Keep changes minimal; explain anything non-obvious
- If requirements are ambiguous, ask a question instead of guessing
```

---

## 16. Deep dive: AGENTS.md

### 16.1 What it is

`AGENTS.md` is a plain-markdown instruction file with no frontmatter or activation logic. Cursor reads it from the project root and from subdirectories. It follows a convention shared with other agent tools, which makes it the most portable option.

### 16.2 AGENTS.md vs. `.cursor/rules`

| | AGENTS.md | `.cursor/rules/*.mdc` |
|---|---|---|
| Format | Plain markdown | Markdown + YAML frontmatter |
| Activation control | Directory-based (nested files) | Always / globs / intelligent / manual |
| Portability | Works with multiple tools | Cursor-specific |
| Best for | Simple, readable, cross-tool basics | Fine-grained scoping, Cursor-specific workflows |
| Setup overhead | Minimal | Higher |

A good hybrid: **AGENTS.md for facts every tool should know** (stack, commands, architecture), **`.cursor/rules` for Cursor-specific scoped behavior**.

### 16.3 Nested AGENTS.md

```
project/
  AGENTS.md                # repo-wide
  frontend/
    AGENTS.md              # frontend-specific
    components/
      AGENTS.md            # component-specific
  backend/
    AGENTS.md              # backend-specific
```

Instructions from nested files are combined with those in parent directories, and the more specific file wins conflicts.

### 16.4 What to put in AGENTS.md

A useful structure:

```markdown
# AGENTS.md

## Overview
One paragraph: what this project is and who uses it.

## Stack
Languages, frameworks, versions.

## Commands
- Install: `pnpm install`
- Dev: `pnpm dev`
- Test: `pnpm test` (single file: `pnpm test path/to/file`)
- Lint/typecheck: `pnpm lint && pnpm typecheck`

## Architecture
- Where things live and why (short directory map)
- Key patterns (e.g., repository pattern, service layer)

## Conventions
- Naming, error handling, testing expectations

## Boundaries
- Never edit: generated files, vendored code, migrations already applied
- Ask first: new dependencies, schema changes, public API changes

## Definition of done
- Tests pass, lint clean, docs updated when behavior changes
```

### 16.5 Tips

- Keep it a page or two. Link to deeper docs (`docs/architecture.md`) instead of embedding them.
- Commands are the highest-value content. Agents fail most often by guessing how to run things.
- The "Boundaries" section is where you prevent the costly mistakes.
- If you also use Claude Code, keep `CLAUDE.md` and `AGENTS.md` consistent (or have one reference the other) so both tools see the same truths.

---

## 17. Deep dive: Skills

### 17.1 What a skill is

A skill is a folder containing a `SKILL.md` file: YAML frontmatter with a `name` and `description`, followed by markdown instructions. Skills follow an open "Agent Skills" standard, so the same skill can work in other tools. They package **procedures**: repeatable, multi-step workflows, optionally with scripts and reference material.

### 17.2 Skills vs. rules

| | Rules | Skills |
|---|---|---|
| Purpose | Conventions and constraints | Step-by-step procedures |
| Loading | Injected by frontmatter logic | Agent reads description and loads when relevant, or you invoke it explicitly |
| Can bundle scripts/assets | Not really | Yes (`scripts/`, `references/`, `assets/`) |
| Example | "Use Zod at API boundaries" | "How to cut a release, end to end" |

### 17.3 Where skills live

Cursor loads skills automatically from:

- `.cursor/skills/` and `.agents/skills/` (project)
- `~/.cursor/skills/` and `~/.agents/skills/` (personal, global on your machine)
- For compatibility: `.claude/skills/`, `.codex/skills/`, `~/.claude/skills/`, `~/.codex/skills/`

Cursor walks these folders recursively, so you can group skills in category folders (`shipping/`, `debugging/`); the skill's name comes from the folder containing `SKILL.md`. In a monorepo, a skill in `apps/web/.cursor/skills/` is only surfaced when the agent works on files under `apps/web/`. A `paths` frontmatter field can scope a skill similarly.

### 17.4 Anatomy

```
.cursor/skills/
  deploy-app/
    SKILL.md
    scripts/
      deploy.sh
      validate.py
    references/
      REFERENCE.md
    assets/
      config-template.json
```

Example `SKILL.md`:

```markdown
---
name: deploy-app
description: Use when the user asks to deploy, release, or ship the app to staging or production. Runs preflight checks, builds, deploys, and verifies health.
---

# Deploy the app

## Steps
1. Confirm the target environment (staging or production). Ask if unclear.
2. Run `./scripts/validate.py` and stop if it fails.
3. Run `./scripts/deploy.sh <env>`.
4. Check the health endpoint and report the result.

## Notes
- Never deploy to production without explicit confirmation.
- For rollback steps, read `references/REFERENCE.md`.
```

### 17.5 Frontmatter behaviors to know

- `name` should match the folder name; `description` is required and drives discovery, so include concrete trigger phrases ("Use when…").
- `disable-model-invocation: true` makes the skill run only when you explicitly invoke it (type `/skill-name`).
- `paths` scopes a skill to matching files.
- Skills with valid frontmatter can back a **Custom Mode**, keeping the skill in context for the whole session.
- Cursor ships a built-in `/create-skill` to scaffold skills.

### 17.6 Syncing to Cloud Agents

Personal skills in `~/.cursor/skills/` run locally. To use them in Cloud Agents, turn on **Sync Skills for Cloud Agents** (Settings → Agents → Context and Tools). Only `~/.cursor/skills/` syncs; project skills already travel with the repo, while `~/.agents/skills/` and other local files stay on your machine.

### 17.7 Writing good skills

- **Description is everything.** Vague descriptions never trigger. Include what it does and when to use it.
- **Keep `SKILL.md` focused;** move long reference material to separate files the skill points to.
- **Scripts for determinism.** If a step must be exact (deploy, migration, codegen), put it in a script and have the skill call it.
- **Make steps verifiable** ("run X, expect Y").
- **Test by asking naturally.** Try phrasing the request three different ways and see whether the skill triggers.

### 17.8 Skill ideas

| Skill | What it does |
|---|---|
| `land-it` | Rebase, run checks, open a PR with a templated description |
| `write-migration` | Create a reversible migration and test it |
| `triage-bug` | Reproduce, isolate, write a failing test, then fix |
| `release` | Version bump, changelog, tag, publish |
| `add-endpoint` | Scaffold route, validation, service, test from templates |
| `security-review` | Checklist-driven review of a diff |

---

## 18. Deep dive: Subagents

### 18.1 What they are

Subagents are specialized AI assistants that Cursor's main agent can delegate tasks to. Each runs in its own context window and returns its result to the parent. They work in the editor, the CLI, and Cloud Agents.

Why use them:

- **Context isolation:** noisy work (searching, log analysis) doesn't pollute the main conversation.
- **Parallelism:** launch several at once.
- **Specialization:** a focused prompt beats a general one for review, testing, or security.
- **Cost control:** run exploration on a faster, cheaper model.

Subagents do **not** automatically save tokens overall (they consume their own), so use them where isolation or parallelism pays for itself.

### 18.2 Built-in subagents

Three ship built in and require no setup:

- **explore:** codebase search and analysis (uses a faster model by default)
- **bash:** runs series of shell commands
- **browser:** controls a browser via MCP tools

### 18.3 Custom subagents

Create a markdown file with YAML frontmatter in:

- `.cursor/agents/` (project, committed to the repo)
- `~/.cursor/agents/` (user, all projects)

Cursor also reads `.claude/agents/` and `.codex/agents/`. When names collide, `.cursor/` wins.

Frontmatter fields:

| Field | Default | Meaning |
|---|---|---|
| `name` | required | Identifier |
| `description` | required | When the parent agent should delegate here. **Critical**: be specific |
| `model` | `inherit` | `inherit`, `fast`, or a specific model ID |
| `readonly` | `false` | If true, restricts write permissions |
| `is_background` | `false` | If true, runs without blocking the parent |

Example: read-only security reviewer

```markdown
---
name: security-auditor
description: Reviews code for security vulnerabilities. Use proactively when new endpoints, authentication logic, or data handling are added or changed.
model: inherit
readonly: true
is_background: false
---

You are a senior application security engineer.

When invoked:
1. Identify changed files (use git diff).
2. Check for injection, broken access control, secrets in code, unsafe deserialization, and missing input validation.
3. Report findings as: severity, file:line, explanation, suggested fix.
4. Do not modify files.
```

Example: verifier

```markdown
---
name: verifier
description: Independently verifies that a task is actually complete. Use after implementation to run tests, lint, and typecheck and confirm requirements are met.
model: fast
readonly: true
---

Run the project's test, lint, and typecheck commands. Compare the result to the stated requirements. Report exactly what passes, what fails, and anything that looks incomplete. Do not fix anything.
```

### 18.4 Behavior notes

- Subagents can be resumed: each run returns an agent ID you can use to continue with preserved context.
- Background subagents write state to `~/.cursor/subagents/`.
- Cloud subagents run on the repo's cloud environment and take MCP servers from your team's configuration, not your local session.
- **Nesting:** sources disagree. One community write-up says Cursor 2.5 lets subagents spawn their own subagents; older docs mirrors say nested subagents are unsupported. Check the current docs before designing deep hierarchies.

### 18.5 Design guidance

- Start with **2–3 focused subagents**, not twenty.
- Make `description` specific ("Use when implementing OAuth flows"), not "general helper."
- Prefer `readonly: true` for reviewers and researchers.
- Keep prompts short; long prompts are slower and harder to maintain.
- If a subagent is single-purpose and needs no context isolation, a skill or slash command may be simpler.
- Assign clear file ownership when running writers in parallel.

### 18.6 Common patterns

| Pattern | Setup |
|---|---|
| **Researcher → implementer** | explore gathers facts; main agent implements |
| **Implementer → verifier** | verifier independently runs checks and reports gaps |
| **Parallel reviewers** | security, performance, and style reviewers run at once on the same diff |
| **Planner → executor** | planner subagent proposes; you approve; main agent executes |

---

## 19. Deep dive: Hooks

### 19.1 What they are

Hooks are scripts Cursor spawns at defined points in the agent loop. They communicate over stdio using JSON in both directions. Use them for guardrails and automation: block dangerous commands, format files after edits, redact secrets, log activity, or notify you when a task finishes. Hooks were introduced in Cursor 1.7.

### 19.2 Configuration

Create a `hooks.json`:

- **Project:** `<project>/.cursor/hooks.json`
- **User (global):** `~/.cursor/hooks.json`
- **Enterprise/MDM-managed:** `/etc/cursor/hooks.json` on some setups

All matching hooks from all locations run. Cursor watches the files and reloads automatically (restart if something doesn't pick up). Project hooks are picked up by Cloud Agents; **user-level hooks are not available in Cloud Agents.** Cursor can also load hooks from third-party tools like Claude Code.

```json
{
  "version": 1,
  "hooks": {
    "afterFileEdit": [{ "command": ".cursor/hooks/format.sh" }],
    "beforeShellExecution": [{ "command": ".cursor/hooks/block-dangerous.sh" }],
    "stop": [{ "command": ".cursor/hooks/notify.sh" }]
  }
}
```

Path note: for user hooks (`~/.cursor/hooks.json`) relative paths resolve from `~/.cursor/`; for project hooks, write paths like `.cursor/hooks/script.sh`.

### 19.3 Available events (as documented)

| Event | Purpose |
|---|---|
| `sessionStart` / `sessionEnd` | Session lifecycle |
| `beforeSubmitPrompt` | Validate prompts before submission |
| `beforeShellExecution` / `afterShellExecution` | Control shell commands |
| `beforeMCPExecution` / `afterMCPExecution` | Control MCP tool usage |
| `beforeReadFile` | Control file access |
| `afterFileEdit` | React to edits |
| `afterAgentResponse` | React to agent output |
| `preToolUse` / `postToolUse` | Around tool calls |
| `subagentStart` / `subagentStop` | Subagent lifecycle |
| `preCompact` | Before context compaction |
| `stop` | Agent finished |
| `beforeTabFileRead` / `afterTabFileEdit` | Tab-specific hooks |

Event availability changes between versions; check cursor.com/docs/hooks for the current list. Some events are informational only (they can't block or message the agent), while "before" events can typically approve or deny.

### 19.4 Hook types

- **Command hooks:** run a script; it receives JSON on stdin and returns JSON on stdout. Exit code 0 means the hook succeeded and its JSON output is used.
- **Prompt hooks:** ask a model to make a judgment (for example, "Does this command look safe to execute?") without a script.

Example payload shape for `afterFileEdit` (fields as documented by community write-ups; confirm against current docs):

```json
{
  "conversation_id": "...",
  "generation_id": "...",
  "file_path": "README.md",
  "edits": [{ "old_string": "# OLD", "new_string": "# NEW" }],
  "hook_event_name": "afterFileEdit",
  "workspace_roots": ["/path/to/project"]
}
```

Hooks may also support a matcher (for example, matching the shell command text) and loop limits for `stop` hooks; consult the docs for exact fields.

### 19.5 Example: auto-format after edits

`.cursor/hooks.json`
```json
{
  "version": 1,
  "hooks": {
    "afterFileEdit": [{ "command": ".cursor/hooks/format.sh" }]
  }
}
```

`.cursor/hooks/format.sh`
```bash
#!/usr/bin/env bash
# Reads JSON from stdin, formats the edited file.
input=$(cat)
file=$(echo "$input" | python3 -c 'import sys,json; print(json.load(sys.stdin).get("file_path",""))')
case "$file" in
  *.ts|*.tsx|*.js|*.json|*.md) npx --no-install prettier --write "$file" >/dev/null 2>&1 ;;
  *.py) ruff format "$file" >/dev/null 2>&1 ;;
esac
exit 0
```

Make it executable with `chmod +x .cursor/hooks/format.sh`.

### 19.6 Example: block risky shell commands

`.cursor/hooks/block-dangerous.sh`
```bash
#!/usr/bin/env bash
input=$(cat)
cmd=$(echo "$input" | python3 -c 'import sys,json; print(json.load(sys.stdin).get("command",""))')

if echo "$cmd" | grep -Eq 'rm -rf /|git push --force|DROP TABLE|terraform destroy'; then
  # Deny with an explanation (check docs for the exact response schema for your version)
  echo '{"permission":"deny","userMessage":"Blocked by project policy","agentMessage":"That command is not allowed here. Propose a safer alternative."}'
  exit 0
fi

echo '{"permission":"allow"}'
exit 0
```

The response field names above illustrate the pattern; the authoritative schema is in the hooks reference for your Cursor version.

### 19.7 Example: notify when done (macOS)

```bash
#!/usr/bin/env bash
osascript -e 'display notification "Agent finished" with title "Cursor"'
exit 0
```

Wired to the `stop` event.

### 19.8 Ideas

| Goal | Event |
|---|---|
| Format/lint after every edit | `afterFileEdit` |
| Block destructive commands or force-pushes | `beforeShellExecution` |
| Redact secrets before prompts leave your machine | `beforeSubmitPrompt`, `beforeReadFile` |
| Audit log of agent actions | Multiple `after*` events |
| Run a verification pass when the agent finishes | `stop` |
| Restrict which MCP tools can run | `beforeMCPExecution` |

### 19.9 Cautions

- Hooks run local code with your privileges. Review anything you install.
- Keep hooks fast; slow hooks stall the agent.
- Make failures safe: decide whether a broken hook should block or allow.
- Version-control project hooks and require review for changes.

---

## 20. Deep dive: MCP

### 20.1 What MCP does

The Model Context Protocol lets Cursor's agent call external tools: query a database, read Jira tickets, fetch Figma designs, search docs, run browser automation, and so on. Each MCP server exposes tools; the agent decides when to call them, and you approve calls (or auto-approve trusted ones).

### 20.2 Configuration

Project-level: `.cursor/mcp.json`. User-level: `~/.cursor/mcp.json`. You can also manage servers in Customize/Settings.

Local (stdio) server:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "some-postgres-mcp-server"],
      "env": { "DATABASE_URL": "postgres://readonly@localhost/app" }
    }
  }
}
```

Remote (URL) server:

```json
{
  "mcpServers": {
    "docs": {
      "url": "https://example.com/mcp"
    }
  }
}
```

Package names and URLs above are placeholders. Use the vendor's documented values. Cursor supports referencing environment variables in config so you don't commit secrets; check the MCP docs for the exact interpolation syntax.

### 20.3 Good uses

| Use case | Benefit |
|---|---|
| Read-only database access | Agent writes queries against real schema |
| Issue tracker (Linear, Jira) | Pull requirements straight into the plan |
| Design tools (Figma) | Implement from actual designs |
| Documentation search | Current framework docs beyond training data |
| Browser automation | Verify UI changes, run end-to-end checks |
| Observability (logs, metrics) | Debug with real production signals |

### 20.4 Safety practices

- **Least privilege:** read-only credentials wherever possible.
- **Trust:** an MCP server is code that can act on your behalf. Install only from sources you trust.
- **Approvals:** keep manual approval on for anything that writes, deletes, or spends money.
- **Prompt injection:** content returned by tools (tickets, web pages, logs) can contain instructions aimed at the agent. Treat it as untrusted data.
- **Guardrails:** use `beforeMCPExecution` hooks to log or block sensitive tool calls.
- **Team control:** admins can manage MCP configuration for teams; cloud agents use team-level MCP config from the Cursor dashboard.

### 20.5 Tips

- Fewer, better servers. Each added tool consumes context and adds decision noise.
- Name and describe tools clearly if you build your own.
- Document in AGENTS.md which MCP tools exist and when to use them.

---

## 21. Deep dive: Cloud Agents, CLI, and automation

### 21.1 Cloud Agents

Cloud Agents (formerly Background Agents) run on remote machines instead of your laptop. You hand off a task, they work in an isolated environment, and they can open a pull request when done. The docs cover setup, builds, best practices, settings, an API, mobile access, and **self-hosted** runtimes so agents can run inside your own infrastructure.

Typical uses:

- Long, well-specified tasks (migrations, dependency upgrades, test backfills)
- Work you'd otherwise interrupt your day for (bug fixes from an issue)
- Fan-out tasks across many repos or files
- Fixes triggered from Slack, Linear, Jira, or GitHub

What they need to succeed:

- A reproducible **environment setup** (install, build, test commands) so the agent can verify its own work
- Clear task descriptions with a done-condition
- Project rules, AGENTS.md, and project hooks (these travel with the repo; user-level hooks do not)
- Team-level MCP configuration
- Synced personal skills if you rely on them (see section 17.6)

Cost note: cloud agents consume credits at your plan's rates, and features built on them (like Bugbot Autofix) draw from the same usage.

### 21.2 Automations and integrations

Cloud agent **Automations** can trigger agents from events. Integrations documented include Slack, Microsoft Teams, Jira, Linear, Notion, GitHub, GitLab, Azure DevOps, Bitbucket, plus JetBrains and Xcode support. Also listed: PR routing and approval agents, security agents, rollouts, and deeplinks.

### 21.3 The CLI

The Cursor CLI brings the agent to your terminal.

- **Interactive:** work in the terminal without the GUI.
- **Headless / CI:** run the agent non-interactively in scripts and pipelines.
- **Shell mode** and **ACP** (Agent Client Protocol) support for editor and tool integrations.
- It uses the same `.cursor/hooks.json` configuration and payload shape as the IDE.
- Other agent tools can delegate to it. For example, community MCP servers let Claude Code hand tasks to Cursor's CLI and models.

Illustrative CI idea (check the CLI docs for exact flags before using):

```bash
# Example concept: ask the agent to summarize risk in a diff during CI
cursor-agent -p "Review the diff on this branch for risky changes and summarize" 
```

### 21.4 SDK

Cursor documents **TypeScript** and **Python** SDKs, so you can build your own automation on top of agents (for example, custom workflows that spawn agents on internal events). See the SDK docs for the current API surface.

### 21.5 Best practices for delegation

1. Write the task like a ticket: goal, context, constraints, acceptance criteria.
2. Ensure the repo can be built and tested from a clean checkout.
3. Prefer small tasks; review PRs as you would a junior engineer's.
4. Keep secrets out of the repo; use the cloud environment's secret management.
5. Watch spend when launching many agents.

---

## 22. Deep dive: Bugbot

### 22.1 What it does

Bugbot reviews pull requests automatically and comments with bugs, security issues, and quality problems. It launched in July 2025 and supports GitHub (including GitHub Enterprise Server), GitLab (including self-hosted), Bitbucket, and Azure DevOps. On GitHub it appears as a check named **Cursor Bugbot**.

### 22.2 Setup

1. Connect your repository through the Cursor dashboard (Integrations).
2. Install the relevant GitHub/GitLab app or connection.
3. Configure behavior in **Bugbot Automations** (which PRs, when to run).
4. Optionally add rules (below).

Modes include running automatically on PRs or only when mentioned in a comment (for example `cursor review` or `bugbot run`). Individual users can override team defaults for their own PRs.

Check outcomes: `success` means no issues and no unresolved earlier comments; `neutral` means it found issues, was cancelled by a newer commit, or hit an error. Bugbot doesn't report a "skipped" state.

### 22.3 Rules for Bugbot

**Important:** Cursor project rules (`.cursor/rules/*.mdc`) do **not** apply to Bugbot runs. Bugbot has its own rule sources:

- **`.cursor/BUGBOT.md`** in your repo. Bugbot loads the root file plus any `.cursor/BUGBOT.md` found walking up from changed files, so nested files can add directory-specific rules.
- **Team rules** created in Bugbot Automations, applying to all team repositories.
- **Repository rules** (learned or manual), which apply when learning is enabled for the organization and repository.
- **Learned rules:** with learning enabled, rules are generated automatically from your team's activity on GitHub, or backfilled from repo history. You can teach Bugbot inline by commenting `@cursor remember [fact]` on a PR.

Rules have size limits; see Bugbot's "Rule limits" docs if a rule seems cut off.

Example `.cursor/BUGBOT.md`:

```markdown
# Bugbot review rules

## Always flag
- New endpoints without authentication or authorization checks
- SQL built with string concatenation
- Secrets, tokens, or keys in code or config
- Removed or weakened tests without explanation

## Project conventions
- API handlers must validate input with Zod
- Database access only from `src/server/`
- Every migration must be reversible

## Ignore
- Formatting-only changes (handled by Prettier)
- Files under `generated/`
```

### 22.4 Autofix

**Bugbot Autofix** spawns a Cloud Agent to fix bugs found in reviews. Per-user options include: use the installation default, off (use manual "Fix in Cursor"/"Fix in Web" links), create a new branch (recommended), or commit to the existing branch (capped at 3 attempts per PR to prevent loops).

Requirements and cost:

- On-demand usage pricing must be enabled.
- Storage must be enabled (not Legacy Privacy Mode).
- It uses Cloud Agent credits at your plan rates, with your Default agent model from Settings → Models.

### 22.5 Other entry points

- `/review` and `/review-bugbot` in Cursor 3.7+, at cursor.com/agents, and in the CLI. `/review-bugbot` stays in sync with Bugbot on your connected source-control provider.
- An API endpoint can queue a Bugbot review for a PR URL (with a dry-run option).
- Bugbot integrates with MCP servers so other AI tools can interact with it.

### 22.6 Pricing note

Older articles describe Bugbot as a $40/user/month add-on. Sources indicate that stopped being its model in May 2026. Confirm on the current pricing and Bugbot pages.

### 22.7 Getting value

- Treat Bugbot comments as a checklist, not gospel. Resolve or dismiss with a reason.
- Feed recurring false positives back via rules or `@cursor remember`.
- Keep `BUGBOT.md` short and specific, like `.cursor/rules`.
- Bugbot supplements human review; it does not replace it.

---

## 23. Security and privacy

### 23.1 Privacy Mode

Privacy Mode can be enabled in settings or enforced by a team admin. When it's on, Cursor states that code data is not used for training by Cursor or its model providers. Some features need storage (Bugbot Autofix, for example, requires storage to be enabled), so features and privacy settings interact.

### 23.2 Data flow to keep in mind

- Prompts, context files, and tool outputs are sent to model providers to generate responses. An outbound HTTPS connection is required.
- MCP servers and hooks execute on your machine (or in the cloud environment for cloud agents) and may access local data.
- Cloud Agents run your code on remote infrastructure; use self-hosted runtimes if data can't leave your network.

### 23.3 Practical safeguards

| Risk | Mitigation |
|---|---|
| Secrets sent to models | Keep secrets out of the repo; `.cursorignore`-style exclusions where supported; hooks to redact or block reads |
| Destructive commands | Manual approval for shell; `beforeShellExecution` hooks; avoid auto-run on production credentials |
| Malicious dependencies | Review dependency changes; hook to scan installs; Bugbot rules |
| Prompt injection via tools/web/issues | Treat tool output as untrusted; least-privilege MCP; approvals |
| Over-trusting generated code | Tests, review, Bugbot, human sign-off |
| Rules as "security" | They are guidance, not enforcement; use permissions, hooks, CI, and code review |

### 23.4 Enterprise controls

Teams and Enterprise offer SSO (SAML/OIDC), centralized privacy mode, usage analytics, model controls, admin dashboards, audit-oriented features, enforced Team Rules, and invoice billing. Enterprise adds custom terms and pooled usage.

### 23.5 Compliance mindset

Decide up front: which repositories may use cloud agents, which models are allowed, whether privacy mode is mandatory, who approves MCP servers, and how you audit agent activity. Write the answers into a short internal policy and enforce what you can technically.

---

## 24. Team rollout playbook

### Phase 0: Decide (week 0)

- Choose plan (Teams Standard for most; Premium seats for heavy users).
- Decide policies: privacy mode on, allowed models, MCP approval process, cloud-agent scope.
- Set spend alerts and assign an admin.

### Phase 1: Pilot (weeks 1–2)

- Pick 3–5 volunteers across skill levels.
- Create a minimal `AGENTS.md` and `core.mdc`.
- Have pilots log where the agent helped, failed, or cost too much.
- Turn every repeated correction into a rule.

### Phase 2: Standardize (weeks 3–4)

- Consolidate rules into a small, owned set; add CODEOWNERS for `.cursor/`.
- Add project hooks (formatter, dangerous-command guard).
- Add 2–3 skills for your most common workflows (release, add endpoint, triage bug).
- Enable Bugbot on a few repos with a short `BUGBOT.md`.
- Publish a one-page "how we use Cursor" doc.

### Phase 3: Scale (month 2+)

- Roll out to all engineers with a short training session.
- Package shared rules/skills into an internal plugin marketplace.
- Review spend weekly for the first month, then monthly.
- Adjust seats: move heavy users to Premium, light users stay Standard.
- Quarterly rule audit: prune, merge, fix stale globs.

### Team norms worth writing down

1. Humans own the code: agent output gets the same review as any PR.
2. Small PRs. Agents make it easy to create huge ones; don't.
3. Never paste secrets or customer data into prompts.
4. If the agent makes the same mistake twice, fix the rule.
5. Tests must pass before requesting review.
6. Disclose agent-generated migrations and security-sensitive changes in the PR description.

### Metrics

Track cycle time, PR size, review turnaround, defect escape rate, credit spend per developer, and rule/skill adoption. Beware vanity metrics like "lines generated."

---

## 25. Deep dive: prompting patterns that work

### 25.1 The anatomy of a strong task prompt

```
Goal:        <what outcome, in one sentence>
Context:     <where this lives; @-mention key files>
Constraints: <patterns to follow, things not to touch, performance/compat limits>
Done when:   <tests pass, endpoint returns X, no lint errors>
Approach:    <ask for a plan first if non-trivial>
```

### 25.2 Templates

**Feature**
```
Plan first, then implement.
Goal: add CSV export to the orders page.
Context: @src/app/orders/page.tsx, @src/server/orders.ts
Constraints: stream large exports; reuse existing auth checks; no new dependencies.
Done when: `pnpm test orders` passes and a manual export of 10k rows doesn't time out.
```

**Bug**
```
Bug: users see a 500 on /api/invoices when the customer has no address.
Repro: <steps>
Expected: empty address block. Actual: TypeError in @src/server/invoices.ts.
Write a failing test first, then fix. Don't change the public response shape.
```

**Refactor**
```
Refactor @src/services/billing.ts into smaller modules.
Behavior must not change. Keep existing exports working. Run the full test suite after each step and stop if anything fails.
```

**Review**
```
Review the current branch diff against main. Focus on correctness, security, and missing tests. List findings by severity with file and line. Don't modify anything.
```

**Explore**
```
Explain how authentication works in this repo: entry points, session handling, and where authorization is enforced. Cite files. Write a short summary to docs/auth.md.
```

### 25.3 Techniques

- **Ask for options** when unsure: "Propose two approaches with tradeoffs, recommend one."
- **Constrain scope explicitly:** "Only touch files under `src/billing/`."
- **Demand verification:** "Show me the command output that proves it works."
- **Have it state assumptions:** "List assumptions before coding."
- **Use checkpoints:** for large tasks, "Stop after step 2 and wait for approval."
- **Reset when stuck:** start a new chat with a distilled summary rather than arguing in a polluted one.

### 25.4 When output quality drops

| Symptom | Likely cause | Fix |
|---|---|---|
| Ignores conventions | Rule not attached, or too many rules | Check activation; trim |
| Invents APIs | Missing docs/context | Add MCP docs search or @docs; provide examples |
| Over-engineers | Vague goal | Add "simplest solution, minimal diff" |
| Breaks unrelated code | No scope limit | Constrain files; use tests |
| Loops or repeats | Context pollution | New chat with a summary |

---

## 26. Master troubleshooting playbook

### 26.1 Rules

See section 8, plus:

- **Rule works in one chat but not another:** intelligent rules depend on the model's judgment; make the description more specific or switch to globs.
- **Two rules conflict:** merge them or specify precedence in one file.
- **Team Rule seems ignored:** confirm it is enabled (not a draft) and not disabled by the user (unless enforced), and check its glob.
- **Bugbot ignoring my rules:** it doesn't read `.cursor/rules`; use `.cursor/BUGBOT.md`.

### 26.2 Skills and subagents

- **Skill never triggers:** rewrite the description with concrete "Use when…" phrases; confirm the folder contains `SKILL.md` and the name matches; restart Cursor; try explicit invocation (`/skill-name`). Check `disable-model-invocation`.
- **Skill scripts fail:** make them executable, use relative paths from the skill root, test outside Cursor.
- **Subagent not used:** its `description` is too vague; mention it explicitly in your prompt.
- **Wrong model used:** Cursor falls back to a compatible model if your plan or settings don't allow the one you specified.

### 26.3 Hooks

- **Hook doesn't run:** verify `hooks.json` location and `"version": 1`, script paths (relative to the right base), executable bit, and restart Cursor if needed.
- **Hook blocks everything:** check the script's exit code and JSON output; log stdin to a file to debug.
- **Works locally, not in Cloud Agent:** user-level hooks don't run in cloud agents; move them to the project.

### 26.4 MCP

- **Server not connecting:** run the command manually to see errors; check Node/Python availability and environment variables.
- **Tools not offered:** confirm the server is enabled for this chat/project and restart it.
- **Agent ignores tools:** mention the tool or server by name; document it in AGENTS.md.

### 26.5 Cost surprises

- Open the usage dashboard and check which model and which features consumed credits.
- Look for long-running chats, huge context, parallel or cloud agents, and Bugbot Autofix runs.
- Switch default to Composer/Auto, plan first, and set spend alerts.

### 26.6 Performance and indexing

- Very large repos: exclude build output and vendored code from indexing where supported, and keep unrelated folders out of the workspace.
- If search results feel stale, reindex the codebase from settings.
- If the app feels slow, disable heavy extensions (it's still VS Code underneath).

### 26.7 When to file feedback

Check the Cursor changelog and forum first; many behaviors changed rapidly in 2026 (pricing, Auto behavior, hooks, subagents). If something looks like a bug, capture the version, plan, model, and reproduction steps.

---

# APPENDICES

## Appendix A: Glossary

| Term | Meaning |
|---|---|
| **Agent** | Cursor's autonomous coding mode that reads, edits, and runs commands |
| **Auto** | Model routing mode where Cursor picks a model; billed at the routed model's rate since Aug 24, 2026 |
| **AGENTS.md** | Plain-markdown agent instructions file, supports nesting |
| **Bugbot** | Automated PR review service |
| **Cloud Agent** | Agent that runs on remote infrastructure |
| **Composer** | Cursor's first-party coding models (Composer 2, 2.5) |
| **Cursor Token Rate** | $0.25/million-token surcharge on third-party models for Teams/Enterprise |
| **`.mdc`** | Markdown-with-frontmatter rule file format |
| **Glob** | File-path pattern used to scope rules |
| **Hook** | Script run at a point in the agent loop |
| **MCP** | Model Context Protocol for external tools |
| **On-demand usage** | Paying for usage beyond the included pool, billed in arrears |
| **Plan Mode** | Agent proposes a plan before coding |
| **Privacy Mode** | Setting guaranteeing code isn't used for training |
| **Skill** | Folder with `SKILL.md` teaching a repeatable procedure |
| **Subagent** | Delegated agent with its own context window |
| **Tab** | Predictive autocomplete |
| **Worktree** | Separate git checkout used to isolate parallel agents |

## Appendix B: File-location cheat sheet

| Purpose | Project | User/global |
|---|---|---|
| Rules | `.cursor/rules/*.mdc` | Customize → Rules (User Rules) |
| Cross-tool instructions | `AGENTS.md` (root and nested) | n/a |
| Skills | `.cursor/skills/`, `.agents/skills/` | `~/.cursor/skills/`, `~/.agents/skills/` |
| Subagents | `.cursor/agents/*.md` | `~/.cursor/agents/*.md` |
| Hooks | `.cursor/hooks.json` + `.cursor/hooks/` | `~/.cursor/hooks.json` |
| MCP | `.cursor/mcp.json` | `~/.cursor/mcp.json` |
| Bugbot rules | `.cursor/BUGBOT.md` (nestable) | Team rules in dashboard |
| Team Rules | n/a | Cursor dashboard |
| Compatibility dirs read by Cursor | `.claude/skills`, `.claude/agents`, `.codex/skills`, `.codex/agents` | `~/.claude/...`, `~/.codex/...` |

## Appendix C: Sources and confidence

**Primary (Cursor documentation):** Rules, Skills, Subagents, Hooks, Bugbot, and Pricing pages at cursor.com (docs and pricing), which are the authority for exact schemas and current numbers.

**Secondary (third-party, used for pricing analysis and community practice):** pricing breakdowns from Flexprice, NoCode MBA, NxCode, Opslyft, Finout, and Emergent; hooks write-ups from InfoQ and GitButler; subagent guides from Morph and community authors; rules guidance from DEV Community and NVIDIA's NeMo toolkit documentation; company facts from Wikipedia.

**Where to be careful:**

- Pricing numbers and pool sizes changed several times in 2026 and vary between sources. Confirm on cursor.com/pricing.
- The exact JSON response schema for hooks and the complete event list evolve. Confirm on cursor.com/docs/hooks.
- Nested-subagent support is described inconsistently across sources.
- Some CLI flags, MCP variable-interpolation syntax, and UI menu names shown here are illustrative; verify against current docs.
- Bugbot's pricing model changed in May 2026; older articles are outdated.

*End of guide.*
