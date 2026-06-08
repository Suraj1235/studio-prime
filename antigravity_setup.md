# Antigravity Prime

[← Back to README](README.md) · Setup guide for [`antigravity_prime.md`](antigravity_prime.md)

**The Studio Prime superprompt — optimized for Google Antigravity's standalone agent orchestration desktop application and development platform (designed for dual-wielding with your preferred editor): isolated-subagent adversarial review, `browser_subagent` for visual verification, durable Artifacts (Task List/Plan/Diffs/Walkthrough/Screenshots/recordings), and persistent Knowledge Items for cross-session memory.**

---

## What is Antigravity Prime?

Antigravity Prime is more than a prompt — it is a complete production engineering playbook running natively on **Google Antigravity 2.0** — Google's standalone agent-first desktop application and orchestration platform (launched at Google I/O on 2026-05-19; default model Gemini 3.5 Flash, with Gemini 3 Pro / Claude Sonnet 4.5 / GPT-OSS selectable). It transforms your standard AI coding assistant into a relentless, autonomous Principal Architect, UI/UX Designer, and Zero-Trust Red Teamer — backed by Antigravity's two user-facing execution modes (Planning / Fast), its autonomy modes (Secure / Review-driven / Agent-driven / Custom), and its multi-agent Agent Manager / Mission Control orchestrator. Under 2.0's "dual-wielding" paradigm, you act as the system director from the Antigravity cockpit (which has NO built-in code editor — reinforcing that it is an agent-orchestration platform, not an IDE) while coding in your preferred external editor (VS Code, Cursor, etc.).

> Note: **Planning / Execution / Verification** as used throughout this document are *Studio Prime's own workflow phases*, not Antigravity engine states. Antigravity exposes only two user-facing execution modes (Planning / Fast). There is no "AGENTIC engine mode"; unattended Sleep-Test runs use **Agent-driven autonomy + `agy -p`** (optionally `/goal`).

While the Studio Prime architecture runs on multiple platforms, **Antigravity 2.0 is the definitive environment for autonomous, sleep-test workloads.** This is because Antigravity 2.0 ships with native asynchronous subagent orchestration (clean isolated contexts, optional Git-worktree isolation, nesting ~10 deep) and CLI task scheduling (`/schedule in` / `/schedule every`) out of the box: the primary agent can instantiate isolated background subagents for security, validation, or design audits, while durable **Artifacts** (Task List, Implementation Plan, Diffs, Walkthroughs, Screenshots, browser recordings) provide a complete audit trail of everything the agent did while you were away. The application's first-class `browser_subagent` (invoked via `/browser`, requires the Antigravity Chrome extension) further enables sandboxed, visual verification that no other Studio Prime host can match.

With Antigravity Prime, you achieve the ultimate "Sleep Test": Start a session under Agent-driven autonomy, schedule background tasks, dispatch isolated subagents, go to sleep, and wake up to a **LIVE** product — a working deployed URL when you provided hosting credentials, or a fully functional product with its localhost server still running (URL + PID documented) when you did not — with a full video walkthrough and screenshot reel of every verified state.

> **Cost profile — before you walk away:** an unattended overnight run fans out across 6 phases × research × a build swarm × an adversarial gate per phase × repair loops × up to 2 northstar re-walks — i.e. many millions of tokens on a non-trivial PRD. **Set a provider-side hard spend cap (Anthropic / OpenAI / Google all support monthly/usage limits) before an unattended run** — Studio Prime cannot observe a token/$ ceiling itself; it only tracks the optional observable proxies `STUDIO_MAX_WALLCLOCK_HRS` / `STUDIO_MAX_TOOL_CALLS` (in `.studio/state/budget.md`), which checkpoint-exit on breach.
> **Scope — web products:** Studio Prime's terminal liveness condition (a content-aware HTTP-200 on a deployed URL or localhost) is WEB-shaped by construction, so web apps and HTTP APIs are the first-class target. For a non-web PRD (mobile / desktop / CLI/library / data-pipeline) Phase 1 detects the target class and either records a substituted terminal condition (e.g. mobile "live" = build artifact + simulator smoke; CLI/library "live" = executed binary/import smoke + installable artifact) or checkpoint-exits with `[UNSUPPORTED_TARGET: non-web]` — it never silently forces a non-web product into a web shape.
> **Localhost survival caveat (no-creds path):** a left-running localhost server only satisfies the Sleep Test if it can OUTLIVE the agent session on your host. Studio Prime runs a detach-survival PROBE before claiming `[LOCAL_LIVE]` and PREFERS daemon-owned supervision (`docker compose up -d`, `systemd --user`, `pm2`, a Scheduled Task) over a bare `nohup`/`Start-Process`. On Windows a bare `Start-Process` child commonly dies when the session/job closes — prefer Docker or a Scheduled Task. If the probe fails on your host, the agent honestly emits `[DEPLOY_READY: localhost cannot outlive this session]` instead of falsely claiming a live localhost.

---

## 🚀 Setup Process

Follow these steps exactly to install Antigravity Prime and unleash its full capabilities.

### Option A: As Project-Level `AGENTS.md` (Recommended — primary cross-tool rules file)
This method uses the cross-tool `AGENTS.md` convention so the same prompt is shared with Cursor, Claude Code, Codex, and Antigravity simultaneously. Per agentpedia, Antigravity reads precedence as **AGENTS.md > GEMINI.md > defaults**, making AGENTS.md the primary project-rules file. *(Precedence is CONTESTED — official docs were unavailable during research; verify on your engine.)*

1. **Open your terminal** and navigate to your project workspace.
2. **Copy the superprompt** into the workspace `AGENTS.md`:
   ```bash
   cp antigravity_prime.md AGENTS.md
   # Launch the Antigravity desktop app or run: agy
   ```

### Option B: As Project-Level `GEMINI.md` (Antigravity / Gemini-CLI legacy, coexists)
This method uses the `GEMINI.md` file — the legacy Antigravity & Gemini-CLI rules file. It coexists with AGENTS.md and remains fully supported; choose it if you are not targeting other tools.

```bash
cp antigravity_prime.md GEMINI.md
```

### Option C: As Global `~/.gemini/GEMINI.md`
This method installs Studio Prime as a user-wide default that activates for every Antigravity workspace.

```bash
mkdir -p ~/.gemini
cp antigravity_prime.md ~/.gemini/GEMINI.md
```

### Option D: As an On-Demand Skill
This method allows you to load Studio Prime modularly only when you explicitly want to use it.

```bash
mkdir -p .agents/skills/studio-prime
cp antigravity_prime.md .agents/skills/studio-prime/SKILL.md
```

### Red Team Workflow
Studio Prime installs a `red_team.md` workflow file at `.agents/workflows/red_team.md` during first-trigger Self-Setup (before Phase 1). Invoke it as a slash command at any phase boundary:

```
/red_team [phase]
```

### Autonomy Modes & Permissions (Recommended)
Antigravity's autonomy mode (Secure / Review-driven / Agent-driven / Custom) and per-action policies are the real gating surface. Tune them for autonomous operation — the settings below balance velocity against safety:

*   **Autonomy mode:** `Agent-driven` for unattended Sleep-Test runs (combine with `agy -p` and optionally `/goal`); `Review-driven` for supervised sessions.
*   **File writes (`write_file` / `edit_file`):** allow via the `write_file(/path)` permission so the agent applies patches without prompting on every edit. Studio Prime's Apex Red Team gate provides the safety net.
*   **Terminal Execution (trusted commands like `npm`, `pnpm`, `pytest`, `git`):** allow via `command(prefix)` (e.g. `command(npm)`, `command(git)`) — required for the Sleep Test to actually run unattended.
*   **Destructive Commands (`rm -rf`, `git push --force`, `DROP TABLE`):** `Ask` (deny by default) — Studio Prime additionally refuses these unless the user has explicitly approved them in the current session. **Deploy commands are NOT destructive:** a live production deploy under standing authorization (hosting/deploy credentials supplied at intake — e.g. `vercel --prod`, `fly deploy`, `kubectl apply`, a registry push) is explicitly carved OUT of this deny-by-default policy and requires NO in-session approval — the provided credentials ARE the authorization. Allow the deploy command prefixes via `command(prefix)` so the Sleep-Test deploy step never stalls.
*   **JavaScript Browser policy:** `Request review` — JS executed in `browser_subagent` against untrusted external pages should go through human review, because a malicious or malformed script could exfiltrate state from a sandboxed-but-authenticated tab. **Exempt the agent's own post-deploy verification** (same-origin state-mutating JS against the staging/prod origin it just deployed): in unattended runs this must NOT wait on a human, or the post-deploy walkthrough deadlocks. (This is a per-action *policy*, not a tool.)
*   **`read_url(domain)`:** allow trusted documentation domains for Phase 1 research; `mcp(server/tool)` per wired MCP server.

### Install & Triggering

Install the desktop app from `antigravity.google/download`. For headless / CI use, drive the same engine from the CLI:

*   **Desktop:** launch the Antigravity application; the workspace auto-loads `GEMINI.md` and any installed skills.
*   **CLI (interactive):** `agy` — opens a REPL bound to the current workspace.
*   **CLI (one-shot, headless):** `agy -p "Start Studio Prime"` — ideal for cron, CI, and overnight Sleep-Test runs.

Once Antigravity is running, trigger the system by typing:

```
"Start Studio Prime"
```

**The Intake Gate:** Upon invocation, Antigravity Prime will NOT dump a massive wall of text. Because Antigravity lacks a discrete `AskUserQuestion` primitive, the agent uses an **approval-gate + plan-checkpoint pattern** — it generates a Plan Artifact with numbered options in a plan-checkpoint format: `[1] NEW PROJECT — Guided discovery / [2] EXISTING CODEBASE — Add features or transform`, then halts at the Planning-Mode review boundary until the user picks a branch. (The `/grill-me` command is an alternative for eliciting clarifying questions before building.) The prompt forbids markdown letter-list ASCII boxes; numbered options inline are the canonical pattern. The Plan Artifact remains durable, so even if the session is closed and resumed later, the pending decision is exactly where the user left it.

### Model Selection

Antigravity exposes a model picker (`/model <name>`) that Studio Prime honors. The default is **Gemini 3.5 Flash**; **Gemini 3 Pro** is recommended for planning and high-stakes architectural decisions, with a Flash-class model for high-throughput Phase 3 scaffolding work. **Claude Sonnet 4.5** and **GPT-OSS** are available as alternate engines if the user prefers cross-model adversarial review during Phase 5 Red Team rounds.

---

## ✨ Core Features

Antigravity Prime leverages Antigravity 2.0's native tool surface to enforce rigorous software engineering standards, transforming the LLM from a stochastic text generator into a deterministic engineering environment:

*   **Proof-of-Work Anti-Hallucination:** LLMs inherently struggle to differentiate between intended outcomes and actual execution results. To counteract this, Antigravity Prime enforces a strict, three-part empirical verification protocol. Before asserting task completion, the agent must generate a `<prediction>` of the expected terminal output, execute the command via the `run_command` tool (which can run in the background and notify the agent on completion), and synthesize a `<divergence_analysis>`. By comparing the expected state against the actual `stdout`, the agent is forced to linguistically process any discrepancies.
*   **Isolated Subagent Orchestration (NEW in 2.0):** Main agents spawn specialized subagents in clean, isolated context windows via `invoke_subagent` (optionally typed via `define_subagent` where available; optional Git-worktree isolation; nesting ~10 deep). This prevents main-context window pollution and enables multi-persona developer collaboration (e.g. outsourcing deep code audits to an isolated zero-trust security subagent). Monitor running subagents via `/agents`. The Apex Red Team review runs here as a native isolated spawn — never a main-thread persona swap.
*   **Background Tasks & Scheduling (NEW in 2.0):** Long-running commands run in the background via `run_command` (the system notifies on completion). Recurring and deferred work is scheduled via the CLI: `/schedule every <interval> <prompt>` (recurring health checks / monitoring) and `/schedule in <time> <prompt>` (deferred one-shot, ~15-min cap), allowing the main developer agent to remain active and unblocked.
*   **Browser-Driven Visual Verification:** Antigravity is the only platform in the Studio Prime family with first-class browser-subagent support. During Phase 5 (Stylize), the agent dispatches `browser_subagent` (via `/browser`; requires the Antigravity Chrome extension) to drive a Chrome instance against the running app. The subagent captures full-page **Screenshots** at every viewport (mobile / tablet / desktop), records a **video Walkthrough** (WebP container reported — MEDIUM confidence) of the canonical user flow, runs DevTools Performance audits to verify the motion budget, reads console logs, and empirically measures OKLCH contrast ratios across the full component state matrix (hover / focus / active / disabled). All evidence is persisted as durable Antigravity Artifacts attached to the phase — no manual screenshotting, no "trust me, it works" claims. 
*   **Autonomous Escalation & State-Preserving Rollback:** Brittle debugging loops are a primary failure vector for autonomous agents. To resolve this, Antigravity Prime implements a bounded, 5-cycle recovery protocol. When confronted with an error, the agent is granted three direct remediation attempts. Upon failure, it is mathematically forced to pivot: executing targeted research via the `search_web` and `read_url_content` tools to inject external context for the specific error, followed by one final retry. After exhausting the repair budget (5 cycles per failure, 20 iterations max), the agent executes a safe `git stash` via `run_command`, logs forensic data to `.studio/blocked.md` (mirrored to a Knowledge Item for cross-session recall), then auto-isolates the failing module — logging it as a `[PRIORITY:H]` TECH_DEBT entry — and continues the pipeline with the remaining scope. Human-as-a-Service (HaaS) is invoked only when the isolated module is a critical-path dependency; in unattended (Sleep Test) mode the agent checkpoint-exits non-zero instead of blocking on a human. A deploy step with provided credentials is never a "blocked critical-path module" — the credentials ARE the authorization, so it auto-deploys rather than checkpoint-exiting; and on a no-credentials run the agent leaves the localhost server running at handoff rather than exiting dark.
*   **Zero-Gap Phase Chaining:** Upon receiving a passing verdict from the Apex Red Team, the agent autonomously transitions to the next phase without pausing for human approval. The agent operates as a continuous pipeline, only halting for explicit blockers or the final Northstar Validation Gate. The deploy/handoff step is never a human checkpoint: when hosting credentials were provided at intake they constitute standing authorization, so asking "should I deploy?" / "ready to go live?" is itself a Zero-Gap violation — the agent deploys and hands over a live URL.
*   **Proactive Context Engineering:** Context is managed before it degrades, not after. The agent tracks a GREEN/AMBER/RED budget (tool-call count, large `read_file` reads, phase length) and climbs a compaction ladder — archive completed tasks, summarize stale decisions, then **offload context-heavy subtasks to a clean-context `invoke_subagent`** ("read big, return small"; Antigravity's native compaction path, alongside the Knowledge Subagent). It writes `context_checkpoint.md` on a heartbeat — after every phase and before every subagent dispatch, not only at RED — for a minimal-loss, re-derivable resume across compaction or crash (the checkpoint captures the last command + its exit code so resume can detect duplicate side effects; it is not a literal zero-loss guarantee).
*   **Parallel Build Swarm (P3/P4):** The heavy build work inside Architecture (P3) and Implement (P4) is parallelized across an ownership-partitioned swarm of Git-worktree-isolated `invoke_subagent` workers, orchestrated by the Agent Manager / Mission Control. Each worker owns disjoint files; shared surfaces stay sequential and main-agent-owned; the main agent is the sole merge authority and RE-RUNS the full phase proof-of-work against the merged whole — speed of construction, never trust in unverified output. Degrades cleanly to sequential when subagent dispatch is unavailable. (The research gate and Apex Red Team gate remain always-sequential barriers.)
*   **Design-Skills Acceleration (P5):** Before hand-authoring tokens, Phase 5 by default (its mandatory first step) fetches the awesome-design-skills registry NON-INTERACTIVELY — `curl` the registry `index.json` and the chosen slug's `SKILL.md` (NOT the interactive `typeui.sh list`, which hangs / silently exits empty headless) — then **color-converts** the skill's hex into the built-in OKLCH system (registry skills routinely ship `#000000`/`#FFFFFF` and zero OKLCH, so the agent TRANSFORMS rather than adopts) and reconciles structure/typography/spacing into `design-system/MASTER.md`. It is a default accelerator on top of the 2026 Impeccable Standard — Studio Prime's banned-pattern bans always win, and only if the registry genuinely can't be reached does the agent fall back to the built-in standard with a logged conditional gate.
*   **Northstar Validation Gate:** Before final sign-off in Phase 6, the agent compares the original Phase 1 requirements (`northstar.md` — immutable, never re-captured) against the final deliverables and writes every gap to a failure-findings file (`northstar_gap_analysis.md`). Isolated gaps trigger surgical remediation: only the phase(s) that own each gap are re-entered (a missing API re-enters P3/P4, an a11y miss P5, a deploy miss — undeployed despite standing creds, or no running localhost server on a no-creds run — re-enters P6 to DEPLOY or to leave the localhost server running, never to ask a human first). If the failure is systemic — smoke tests fail, most requirements are unmet, or the blueprint/architecture itself is implicated — it escalates: the agent re-enters Phase 1 and re-walks the entire pipeline with the failure findings as mandatory input to every phase. The two tiers use SEPARATE counters (a larger surgical Tier-1 budget, a small Tier-2 systemic budget) with a no-progress convergence guard; after the caps, non-critical gaps are deferred to `TECH_DEBT`, and for a critical-path gap the agent FIRST feature-flags it off and confirms the largest live-serving subset is up (content-aware 200) before any checkpoint-exit — so you wake to a live product missing one flagged feature, never a dark exit-1 process.

---

## ⚙️ Function: Orchestration & The Gating System

The operational architecture of Antigravity Prime is defined by a strictly sequential, 6-Phase lifecycle. Progression through these phases is not determined by heuristic guessing, but rather governed by a deterministic, mathematically verifiable gating system.

### The Scratchpad DAG & Transition Checklists
Before the system is permitted to transition from Phase *N* to Phase *N+1*, it must construct a Directed Acyclic Graph (DAG) in its scratchpad memory (`<phase_gate_checklist>`). The agent records the start/end of each phase in the auto-generated **Task List artifact** (`task.md`, mirrored to `.studio/todos.md`) so the engine's task-tracking surface stays in sync with the DAG. This enforces a mandatory sequential evaluation of:
1.  **Prerequisite Verification:** Ensuring all artifact dependencies from Phase *N* exist on the filesystem (verified via `read_file` / the engine's file-listing capability).
2.  **Proof-of-Work Compilation:** Gathering the raw `stdout` from testing and verification scripts (via `run_command`, including its background-completion capture).
3.  **Apex Red Team Adjudication:** Submitting the collected evidence to the `/red_team` workflow for adversarial review.

### The Apex Red Team (Isolated Subagent Review)
In Antigravity 2.0, the Apex Red Team review is executed inside a fully isolated background subagent (clean context window). This eliminates context contamination and ensures an objective audit — it is a native isolated spawn, never a main-thread persona swap:
*   **Auditor Definition (optional):** The main agent defines a zero-trust `RedTeamAuditor` subagent via `define_subagent` where available; otherwise it passes the same auditor system prompt inline to `invoke_subagent`.
*   **Subagent Audit:** The main agent spawns the subagent using `invoke_subagent`, passing the current phase's raw terminal output, and (in P5/P6) the `browser_subagent` screenshots and recording.
*   **Multi-Persona Analysis:** The auditor subagent debates itself under Steelman, Adversarial, and Synthesis personas in isolation, returning a formal verdict containing either `[GREEN_FLAG]`, `[TECH_DEBT]`, or `[BLOCKER]`.
*   **Gating:** A `[GREEN_FLAG]` or `[TECH_DEBT]` classification lets Studio Prime advance to the next phase's Planning checkpoint. A `[BLOCKER]` halts forward progress and triggers an autonomous git rollback and HaaS alignment gate (Planning-Mode Decision Checkpoint). (Gating is enforced by Studio Prime's own Verification workflow phase, not by an engine mode.) A P6 "undeployed despite standing credentials" or "no running server at handoff" condition is NOT raised as a `[BLOCKER]` checkpoint — it is auto-remediated by deploying (creds path) or relaunching the localhost server (no-creds path); a logged `[LOCAL_LIVE]` server satisfies the deploy gate on no-creds runs and the reviewer must accept it.

### The 6-Phase Lifecycle

1.  **Phase 1: Blueprint & Baseline**
    *   **Function:** Establishes the foundational architectural constraints. Initiates mandatory web research (min 3, recommended 5-10+) via `search_web` + `read_url_content` to anchor the model in current, verified documentation rather than parametric memory.
    *   **Output:** Generates `northstar.md` (immutable requirements), `decisions.md` (the architectural state machine), and `architecture/security_baseline.md` (OWASP A01–A10 applicability matrix, verified complete before GREEN_FLAG). (The `red_team.md` workflow file is installed earlier, during first-trigger Self-Setup before Phase 1.)
2.  **Phase 2: Link**
    *   **Function:** Defines the integration seams. This phase enforces the "Data-First Rule," establishing database migration strategies, external API wiring, and identity management before any business logic is written.
    *   **Output:** Complete `data_contracts.md` and integration blueprints.
3.  **Phase 3: Architecture & Scaffolding**
    *   **Function:** Translates the blueprint into syntactically valid code structures using `write_file` for new files and `edit_file` for surgical edits. Prohibits business logic. Focuses on strict types, interface boundaries, TDD + E2E stubs, **multi-stage Docker configs (`Dockerfile`, `docker-compose.yml`)**, **automated CI/CD workflows**, and **database migrations** — each VALIDATED via `run_command` (`docker build`, `docker compose config`, migration dry-run, CI YAML + required-job check), not merely created. Non-zero exit or a missing test/lint/security job is a BLOCKER.
4.  **Phase 4: Implement & Verify**
    *   **Function:** Executes business logic to satisfy TDD stubs. Enforces 80%+ code coverage (verified via `run_command`), **EXECUTED E2E integration tests** (not stubs — failing critical-path test is a BLOCKER), a **dependency vulnerability audit + secrets-leak scan** via `run_command` (any HIGH/CRITICAL CVE or secret match is a BLOCKER), **JSON-log validation** (sample log piped to `jq`/`json.tool` for timestamp/level/message), and strict **OWASP security hardening** (schema validation, secure HttpOnly/Secure/SameSite cookies, safe JWT RS256 signing, and API rate-limiting per IP).
5.  **Phase 5: Stylize (UI/UX Polish)**
    *   **Function:** Transforms functional interfaces into production-grade aesthetics. Enforces the 2026 Impeccable Design Standard, ensuring **WCAG 2.1 AA accessibility verified by an EXECUTED automated audit** (`npx pa11y --standard WCAG2AA` or axe-core via Playwright — critical/serious violations are a BLOCKER), a **verifiable Component State Matrix** (every interactive element enumerated against all 5 states; missing focus ring is a BLOCKER), fluid motion physics, and LCP (<2.5s) / CLS (<0.1) metrics — all empirically verified by a dispatched `browser_subagent` whose screenshots/WebP timestamps the Apex reviewer must cite.
6.  **Phase 6: Release**
    *   **Function:** The final pre-flight operational check. Integrates and TESTS **graceful OS signal shutdowns** (sends `SIGTERM` via `run_command` against a DISPOSABLE instance and asserts a clean exit within the drain window — this kill-based gate never leaves the product dead at handoff), **health-check curls** (expect 200), a **timed rollback dry-run** (< 5min, measured), **synthetic-error → alert-propagation verification** (confirms a Sentry/Datadog event AND a Slack/Discord message within a timeout), production **DB migration dry-run** before apply, and **EXECUTED post-deploy smoke tests** with rollback-on-5xx wired to a real command. All kill-based gates run FIRST; then the FINAL persistent server start happens — when hosting credentials were provided the agent deploys live (the credentials ARE the authorization, in interactive AND unattended mode) and verifies the production URL returns `200`; when they were not, the agent leaves a DETACHED localhost production server running (recording its URL + PID + stop/start commands) **— but ONLY after a detach-survival probe proves the backgrounding idiom outlives a fresh shell on this host (preferring `docker compose up -d` / `systemd --user` / `pm2` / a Scheduled Task over a bare `nohup`/`Start-Process`); if the probe fails it emits `[DEPLOY_READY: localhost cannot outlive this session]` instead of a false live claim** — and emits `deploy_ready.sh` for going live later. Immediately before sign-off it re-probes that live access point with a **content-aware** check (asserts a northstar marker in the body, not a blind `200`) and re-runs the critical-journey smoke suite. The synthetic-error → alert-propagation gate is CONDITIONAL: when no error-tracker DSN / alert webhook was supplied it downgrades to TECH_DEBT (a deployed product never goes dark over an observability nicety the PRD never required); it is a hard BLOCKER only when monitoring creds WERE supplied and propagation fails. It concludes with a comprehensive 17-section `HANDOFF.md` that is validated for zero placeholders, full env-var coverage, AND a reachable deployment URL (creds path) OR a still-running localhost URL+PID (no-creds path), plus the final **Northstar Validation Gate**.
    *   **Output:** A LIVE product — a working deployed URL (creds path) or a still-running localhost server with documented URL+PID + `deploy_ready.sh` (no-creds path) — a deployment receipt, and a final consolidated Knowledge Item summarizing every architectural decision made across the run — invaluable for the next session's Planning-Mode boot. At sign-off the agent emits a concise **Deployment Briefing** as its final user-facing message (also persisted to `.studio/state/deployment_briefing.md`, HANDOFF §10, and a KI): verified live status, deploy-target operations (redeploy/rollback/logs commands, or `deploy_ready.sh` + a ranked host recommendation on the no-creds path), proactive next steps, and the exact `agy -p "Continue Studio Prime"` resume command.

---

## 📥 Expected Input/Output (I/O)

To effectively orchestrate the development lifecycle, Antigravity Prime expects structured inputs and generates deterministic outputs — with **Artifacts as the primary output channel**. Understanding this I/O contract is critical for maximizing the system's efficacy.

### **Input Modalities**
*   **Intake Triggers:** Natural language commands such as `"Start Studio Prime"` or `"Continue Studio Prime"`, issued via the Antigravity desktop chat or the `agy` CLI.
*   **Requirement Documents (PRDs):** Structured text outlining the core problem, target audience, and functional requirements.
*   **Design Assets & Constraints:** Optional guidelines, reference URLs (consumed via `read_url_content`), or brand tokens.
*   **On-Disk Brand Assets:** During Phase 1 Design System Intake, Studio Prime scans the working directory for on-disk brand assets (`.png` / `.svg` / `.figma`, plus design-system specs in `design/`, `assets/`, or the project root) and derives the Phase 1 design baseline from them. If none are found, it auto-generates a design system from the 2026 Impeccable Design Standard defaults.
*   **Human-as-a-Service (HaaS) Feedback:** Explicit approvals, architectural overrides, or credential provisions delivered through Antigravity's plan-checkpoint pattern when the agent halts at a Planning-Mode Decision Checkpoint. Always formatted as a simple numbered reply so it works equally well from desktop chat or the `agy` CLI. (Note: hosting/deploy credentials supplied at intake — brief, `.env`, env vars, secret store, or a CLI already logged in — are NOT reactive checkpoint provisions; they constitute STANDING authorization to deploy live, so no Planning-Mode checkpoint fires for the deploy step when such creds are already present.)

### **Output Modalities**
*   **Antigravity Artifacts (primary):** Durable Task List, Implementation Plan, Diffs, Walkthrough, Screenshots, and video-recording artifacts attached to each phase. These are the canonical, user-facing record of the agent's work — not transient chat messages. Artifacts survive session restarts.
*   **Terminal Execution (`stdout`/`stderr`):** Verifiable proof of execution, testing, and deployment captured via `run_command` (including background-completion output).
*   **Persistent State Files (`.studio/`):** Markdown and JSON files that maintain the agent's memory, architectural decisions, and active task lists. Edits use `write_file` / `edit_file`.
*   **Knowledge Items:** Cross-session memory entries written in parallel with `.studio/` (see the Knowledge Items section below).
*   **Task List Updates:** Every phase transition is recorded in the auto-generated Task List artifact (mirrored to `.studio/todos.md`) so Mission Control's task-tracking surface stays in sync.
*   **Plan-Checkpoint Prompts:** In place of a discrete `AskUserQuestion` tool, structured numbered decision points are embedded in the Plan Artifact, and the agent halts at a Planning-Mode boundary for human input.
*   **Adversarial Review Logs:** Detailed markdown reports from the Apex Red Team workflow, classifying the phase's output into `[GREEN_FLAG]`, `[TECH_DEBT]`, or `[BLOCKER]`.

---

## 📁 File Structure & Reasoning

Studio Prime writes its state to the disk to cure "LLM Amnesia" and survive context limits. On Antigravity, the on-disk `.studio/` tree is the **portable** source of truth, while **Knowledge Items** are the engine-native cross-session recall layer (see `🧠 Knowledge Items` below).

```text
.studio/
├── todos.md                # Active task list (mirrored to the Task List artifact)
├── decisions.md            # The "Brain": prevents the agent from contradicting past choices
├── data_contracts.md       # API schemas enforced before UI implementation
├── archive.md              # Flushed tasks to keep the active context small
├── blocked.md              # Detailed logs of failed escalation attempts
├── state/
│   ├── northstar.md           # The immutable original requirements
│   ├── phase*_research.md     # Raw search findings
│   ├── swarm_plan.md          # Parallel Build Swarm ownership DAG (P3/P4)
│   ├── context_checkpoint.md  # Heartbeat context handoff (minimal-loss, re-derivable resume across compaction/crash)
│   ├── effects.md             # Side-effect ledger (deploy/migration/publish) — idempotency guard for resume/re-walk
│   ├── critical_journeys.json # {id, slug, acceptance_criterion} journey↔test mapping
│   ├── budget.md              # Optional wall-clock / tool-call budget tracking (cost guard)
│   ├── deploy_target.md       # Resolved deploy target + standing-auth marker
│   ├── deploy_ready.sh        # Exact go-live commands (no-creds path)
│   ├── local_live.md          # {url, pid, start/stop cmd} for the left-running localhost server
│   └── deployment_briefing.md # Final deployment briefing (chat + HANDOFF §10 + KI)
└── checklists/             # Mandatory DAG gate checkpoints

architecture/
├── decisions.md            # Core architecture decisions
├── data_contracts.md       # DB schemas and API contracts
├── integration_plan.md     # Integration blueprint
├── research_spike.md       # Synthesized, deduplicated research
└── phase_snapshots/        # Hard checkpoints allowing safe rollback

design-system/
└── MASTER.md               # Global design tokens (OKLCH, typography)

.tmp/
├── verify.sh / .ps1        # Local CI/CD test script
└── research_*.md           # Parallelized research (prevents context contamination)
```

### **Antigravity-Specific Artifacts**
*   `AGENTS.md` — primary cross-tool project-rules file (also picked up by Cursor / Claude Code / Codex). Reported precedence AGENTS.md > GEMINI.md > defaults *(CONTESTED — verify on your engine)*.
*   `GEMINI.md` — Antigravity / Gemini-CLI legacy rules file; coexists with AGENTS.md and is fully supported.
*   `.agents/skills/studio-prime/SKILL.md` — on-demand skill installation path (YAML frontmatter `name` + `description` + Markdown body). Global skills live at `~/.gemini/antigravity/skills/<name>/SKILL.md`.
*   `.agents/workflows/red_team.md` — workflow file invoked as `/red_team` for the Apex Red Team. Auto-installed at first-trigger Self-Setup (before Phase 1). (Global workflows: `~/.gemini/antigravity/global_workflows/`.)
*   `.agents/rules/` — workspace rules directory.
*   `.agents/mcp_config.json` — MCP server configuration. **Gotcha:** the field is `serverUrl`, not the deprecated `url`/`httpUrl` — using the wrong key silently disables the server. Unified config also lives at `~/.gemini/config/mcp_config.json`; per-surface auth at `~/.gemini/antigravity/mcp_config.json` (desktop) and `~/.gemini/antigravity-cli/mcp_config.json` (CLI).
*   `~/.gemini/GEMINI.md` — global, user-wide Antigravity rules. Applies to every workspace.
*   Hooks (`PreToolUse` / `PostToolUse` / `SessionStart` / `SessionEnd`) — *assumed from the Gemini-CLI lineage; unverified for 2.0.* Studio Prime does not require these but respects any user-installed hooks where supported.

---

## 🔁 Workflow Phases & Autonomy Modes

Studio Prime's **workflow phases** (Planning / Execution / Verification — Studio Prime vocabulary) map onto Antigravity's real controls: its two user-facing **execution modes** (Planning / Fast) and its **autonomy modes** (Secure / Review-driven / Agent-driven / Custom). These are NOT four co-equal engine modes, and there is no "AGENTIC engine mode":

*   **Planning phase ↔** Intake Gate + Phase 1 / Phase 2 plan steps, run under Antigravity's **Planning Mode**. The agent halts here for explicit human approval at branch decision points (Planning-Mode Decision Checkpoint), and the Plan Artifact is the durable record of what was agreed.
*   **Execution phase ↔** Phase 2 / 3 / 4 implementation work — file edits via `write_file` / `edit_file`, and command execution via `run_command` with background-completion capture. Runs faster under **Fast Mode**.
*   **Verification phase ↔** Apex Red Team gates + Phase 5 / 6 `browser_subagent` dispatch. Studio Prime refuses to advance until the verdict is `[GREEN_FLAG]` or `[TECH_DEBT]`.
*   **Unattended Sleep-Test ↔ Agent-driven autonomy + `agy -p`** (optionally `/goal` run-to-completion, `--dangerously-skip-permissions` in CI), with the Agent Manager / Mission Control monitoring parallel background agent teams. Studio Prime tightens the gating criteria when running unattended — Red Team verdicts must be `[GREEN_FLAG]` (not merely non-`[BLOCKER]`) to auto-advance. The unattended P6 `[GREEN_FLAG]` criteria INCLUDE the live end-state: a successful live deploy (when hosting credentials were provided) OR a running localhost server with documented URL+PID (when they were not). Achieving that end-state is what EARNS the green flag — so the deploy/handoff step advances the run rather than stalling it.

---

## 🌐 Multi-Surface Operation

Antigravity is a dual-surface product, and Studio Prime detects which surface it is running in and adapts accordingly:

*   **Desktop App (Agent Manager / Mission Control):** Standalone agent-orchestration cockpit with NO built-in code editor. Orchestrates unlimited concurrent agent teams in parallel (each with its own subagent tree + progress feed), inline Artifact rendering, live Walkthrough playback, the Knowledge Items panel, and **Gemini Audio voice input**. Best for interactive sessions and parallel feature development.
*   **`agy` CLI:** Headless, Sleep-Test-friendly invocation via `agy` (interactive) or `agy -p "instruction"` (one-shot, scriptable; flags `--continue`, `--conversation <id>`, `--add-dir`, `--output-format json`, `--dangerously-skip-permissions`). Best for CI, scheduled runs, and unattended overnight workloads. Artifacts are still produced and persisted — they render in the desktop app the next time you open the workspace.
*   **Python SDK / Managed Agents (NEW in 2.0):** `pip install google-antigravity` (`from google.antigravity import Agent, LocalAgentConfig`; async context manager; `agent.chat()`; `CapabilitiesConfig`). The Managed Agents API provisions isolated Linux environments with persistent state — an ideal headless/CI substrate.

The agent introspects the runtime on its first turn and tunes its output formatting and approval-gating cadence to whichever surface it's running on. In the desktop surface, Studio Prime is more aggressive about dispatching parallel `browser_subagent` runs because Mission Control surfaces their progress visually; in the CLI surface, it serializes those calls to keep the log readable for post-hoc review.

### MCP Configuration

Antigravity supports the full MCP (Model Context Protocol) spec, configured via `.agents/mcp_config.json` (or the unified `~/.gemini/config/mcp_config.json`; per-surface auth lives at `~/.gemini/antigravity/mcp_config.json` for desktop and `~/.gemini/antigravity-cli/mcp_config.json` for the CLI). The config uses an `mcpServers` object; OAuth servers set `authProviderType`. Studio Prime auto-detects the file and surfaces any configured MCP servers (filesystem, GitHub, Postgres, etc.) as additional tool surfaces during Phase 1 research and Phase 4 implementation. **Critical gotcha:** the field name is `serverUrl`, not the deprecated `httpUrl` / `url` used by other MCP hosts — using the wrong key will silently disable the server and the agent will quietly fall back to its built-in tool surface without an explicit error. MCP auth is per-surface.

---

## 🧠 Knowledge Items

Antigravity ships a first-class **Knowledge Items** layer for cross-session memory — the engine surfaces relevant prior knowledge to future agent runs automatically. Studio Prime uses a **dual-write pattern** to get the best of both worlds:

*   **Write to `.studio/`** for portability — the on-disk markdown tree travels with the repository, is diffable, and works on every Studio Prime host (OpenCode, OpenClaw, Antigravity, Codex, Claude Code).
*   **Write to a Knowledge Item** for engine-native cross-session recall — Antigravity will surface the same content automatically in future sessions without the agent having to `read_file` it back in.

Critical decisions, blocked-state forensics, and architectural invariants are always written to both. This ensures Studio Prime remains portable across hosts while still benefiting from Antigravity's native memory layer. The dual-write pattern also acts as a redundancy mechanism: if a Knowledge Item is accidentally pruned, the `.studio/` tree is the authoritative reconstruction source, and vice versa.

The Knowledge Items Studio Prime maintains include:

*   **Architectural invariants** — never-violate rules established in Phase 1 (e.g., "all state mutations go through the `useStore` hook").
*   **Blocked-state forensics** — the contents of `.studio/blocked.md` so the next session can resume escalation triage without re-reading the full failure log.
*   **Design-system tokens** — the resolved OKLCH palette and motion budget so future sessions don't have to re-derive them.
*   **Phase summaries** — one-paragraph distillations of each completed phase, written at the end of Phase 6 for fast cross-session orientation.

---

## 🔧 Tool Surface Reference

Antigravity exposes a focused, opinionated tool surface. Studio Prime calls these throughout the lifecycle. **HIGH** = verified; *italic + (reverse-engineered)* = best-effort, adjust to the live engine signature if rejected:

*   **File I/O (HIGH):** `read_file`, `write_file`. *(`view_file` / `edit_file` / `create_file` are reverse-engineered alternates on some builds.)*
*   **Command Execution (HIGH):** `run_command` (background-capable; param schema is contested across sources — pass only the params your engine accepts). *(`manage_task` / `command_status` for background handles are reverse-engineered, single-source; `command_status` has a known stuck-`RUNNING` bug.)*
*   **Isolated Subagents:** `invoke_subagent` (HIGH; async, clean context, optional Git-worktree isolation, nesting ~10), `define_subagent` *(MEDIUM — reverse-engineered; if unavailable, pass the role's system prompt + tool allowlist inline to `invoke_subagent`)*. Monitor via `/agents`. *(No verified `send_message` / `manage_subagents` kill tool — do not assume one.)*
*   **Scheduling (HIGH, CLI):** `/schedule in <time> <prompt>`, `/schedule every <interval> <prompt>`, `/agent <task>`, `/agents`.
*   **Research:** web search + URL read (HIGH capability; governed by the `read_url(domain)` permission). Exact tool names are reverse-engineered (referred to as `search_web` / `read_url_content` in the prompt for readability).
*   **Visual Verification (HIGH):** `browser_subagent` via `/browser` (requires the Chrome extension; tools include click/scroll/type/`read_console_logs`/DOM capture/screenshot/video).
*   **Task Tracking (HIGH):** the auto-generated Task List artifact (`task.md`). *(There is no verified `task_boundary` engine tool.)*
*   **Autonomy / Permissions (HIGH):** modes Secure / Review-driven / Agent-driven / Custom; `/goal`; the JavaScript Browser policy; the `action(target)` permission grammar (`command(prefix)`, `read_file(/path)`, `write_file(/path)`, `read_url(domain)`, `mcp(server/tool)`).

In Antigravity 2.0, general-purpose subagents are supported via `invoke_subagent` (optionally typed via `define_subagent`), which enables isolated parallel agent execution and eliminates the need for main-thread persona-swaps during adversarial review.

---

## 🔬 Credits & Scientific Foundation

Studio Prime's mechanics are not arbitrary; they are implementations of peer-reviewed AI and LLM research:

*   **Apex Red Team (Multi-Agent Debate):** Based on research like *ChatEval: Towards Better LLM-based Evaluators through Multi-Agent Debate (Chen et al., 2023)*. Studies prove that assigning distinct, isolated adversarial roles reduces LLM hallucinations and false positives far better than single-agent self-critique. Antigravity's native isolated subagents (`invoke_subagent`) + AGENTS.md/GEMINI.md rules, combined with Studio Prime's Verification workflow phase, make this adversarial isolation enforceable.
*   **Proof-of-Work (Verbal Reinforcement):** Rooted in *Reflexion: Language Agents with Verbal Reinforcement Learning (Shinn et al., 2023)*. Converting raw environmental feedback (terminal stdout) into a linguistic self-reflection allows the agent to break out of repetitive failure loops.
*   **Context Compaction (`.studio/` + Knowledge Items):** Based on architectures like *MemGPT: Towards LLMs as Operating Systems (Packer et al., 2023)*. Writing state to a persistent filesystem and retrieving it Just-In-Time (`read_file` + Knowledge Item recall) prevents the "Lost in the Middle" context degradation phenomenon.

---
*Architected by Suraj Kuncham. Designed to push the boundaries of autonomous software engineering.*
