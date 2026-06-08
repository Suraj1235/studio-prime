# 🌌 Studio Prime — Universal Edition Setup & Reference

[← Back to README](README.md) · Setup guide for [`studio_prime.md`](studio_prime.md)

**The omni-executioner prime with runtime platform auto-detection — one prompt, six platforms.**

**Covering OpenCode, Claude Code, OpenClaw, Codex CLI, Antigravity, and Cursor (degraded fallback).**

> **The Sleep Test:** hand the agent a PRD + your API keys, walk away, and wake up to a **LIVE** product — a working deployed URL when hosting credentials were provided, or a fully functional product with its localhost server still running (URL + PID documented) when they were not. Every prime enforces an *Autonomous Execution Contract* so it never stalls, loops, or silently gives up unattended — and a production-grade *proof-of-work* gate (migrations, Docker/CI-CD, OWASP + secrets scanning, executed E2E/smoke tests, structured logging, health probes, graceful shutdown, WCAG 2.1 AA, a 17-section `HANDOFF.md`) so what comes back is real, not hallucinated.

> **💰 Before you walk away — set a provider-side spend cap.** A non-trivial PRD is *many millions of tokens* across 6 phases × research fan-out × a parallel swarm × an adversarial gate at every phase, plus up to two northstar re-walks. Studio Prime can observe wall-clock and tool-call counts (optional `STUDIO_MAX_WALLCLOCK_HRS` / `STUDIO_MAX_TOOL_CALLS` guards that checkpoint-exit on breach) but it **cannot** see your dollar/token balance — set a hard usage/spend limit in your provider console (Anthropic / OpenAI / Google all support this) *before* an unattended run.

> **🔌 The no-creds "still-running localhost" promise is host-dependent and PROVEN, not assumed.** A detached localhost server only counts as live if it survives the agent session ending on *your* host. Studio Prime runs a **detach-survival probe** before claiming `[LOCAL_LIVE]` and prefers daemon-owned supervision (`docker compose up -d`, `pm2`, a Scheduled Task on Windows) over bare `nohup`/`Start-Process`. If your host tears down the process tree at session end and no supervisor is available, you get an honest `[DEPLOY_READY]` (a one-command go-live script) instead of a false "it's live" — see the per-host localhost-survival notes in the Platform Support Matrix below.

> **🌐 Scope: Studio Prime targets web products.** The terminal liveness condition is an HTTP-200 URL by construction, so mobile/desktop/CLI/library/data-pipeline targets are not first-class — for a non-web PRD the agent records a substituted terminal condition or checkpoint-exits with `[UNSUPPORTED_TARGET: non-web]` rather than forcing a web shape.

### ⚡ 2-Minute Quickstart

1. **Pick your host** and copy its prime into the host's system-prompt/instructions location:
   | Host | File | Install target |
   |---|---|---|
   | OpenCode | `opencode_prime.md` | `.opencode/system.md` |
   | Claude Code | `claudecode_prime.md` | `CLAUDE.md` |
   | OpenClaw | `openclaw_prime.md` | `AGENTS.md` |
   | Codex CLI | `codex_prime.md` | `AGENTS.md` |
   | Antigravity | `antigravity_prime.md` | `AGENTS.md` (or `GEMINI.md`) |
   | Cursor / unknown | `studio_prime.md` | host's system-prompt area |
2. **Trigger:** type `Start Studio Prime`.
3. **Answer the one intake question** (New Project or Existing Codebase), hand over your PRD + any API keys, and **walk away**.

See the per-platform install + setup blocks below (each has Option A: system prompt, Option B: on-demand skill), and the matching `*_setup.md` for full platform details.

---

## What is Studio Prime?

Studio Prime is more than a prompt — it is a complete production engineering playbook in a single Markdown file. It transforms a standard AI coding assistant into a relentless, autonomous **Principal Architect, Senior DevOps Engineer, Master UI/UX Designer, and Zero-Trust Red Teamer** — all within a single, unified runtime. The four-persona mandate is not metaphorical: at any given moment the agent is operating with the semantic intuition of a principal-level architect, the operational discipline of a senior DevOps engineer, the visual precision of a master UI/UX designer, and the adversarial paranoia of a dedicated Red Team.

What makes the universal edition distinct is its **runtime platform adaptation**. Rather than shipping platform-specific prompts tuned to individual hosts, the universal prime is **one prompt that adapts at session start** — now covering 6 platforms (OpenCode, Claude Code, OpenClaw, Codex CLI, Antigravity, and Cursor) via auto-detection. It physically probes each tool with a minimal no-op invocation, builds an internal capability map, and routes its execution strategy through a degradation matrix. The same prompt that gives you a full "Sleep Test" experience on OpenCode will gracefully degrade to a Markdown-checklist workflow on a host with no native task tool — never silently pretending capabilities exist that do not.

The universal edition is the right entry point if (a) you do not yet know which host you will run the agent on, (b) you maintain projects across multiple hosts and want behavioral parity, or (c) you want to read the full architecture before picking a host-specific prime.

### Platform Coverage: Codex CLI + Antigravity

Studio Prime covers two platforms with fundamentally different agent paradigms alongside the original hosts:
- **OpenAI Codex CLI** — the 2025 Rust-rewrite terminal coding tool. Studio Prime dispatches each blinded Apex Red Team round as its own fresh-context sub-agent via `spawn_agents_on_csv` (the rounds run SEQUENTIALLY — Round 2 attacks Round 1's assertions, Round 3 adjudicates both — so they are not independent parallel workers; `spawn_agents_on_csv`'s true parallelism is reserved for independent research fan-out). It uses custom-agent-as-TOML for the Apex Reviewer, `apply_patch` V4A diffs for safe file mutation, and `codex exec` for unattended Sleep-Test runs.
- **Google Antigravity 2.0** — the agent-first standalone agent-orchestration desktop application (launched Google I/O 2026; NOT an IDE, no built-in editor) plus the `agy` CLI and Python SDK, defaulting to Gemini 3.5 Flash (also Gemini 3 Pro / Claude Sonnet 4.5 / GPT-OSS). It exposes two execution modes (Planning Mode and Fast Mode) and four autonomy modes (Secure / Review-driven / Agent-driven / Custom), durable Artifacts (Task List, Implementation Plan, Walkthrough, Code Diffs, Screenshots, Browser Recordings), Skills, native isolated subagent spawn via `invoke_subagent` (Git-worktree isolation), and the `browser_subagent` (invoked via `/browser`) for autonomous visual verification of UI work.

The Platform Auto-Detection block probes 5 sub-agent dispatchers and 5 shell-tool variants, with explicit derivation rules for Codex CLI and Antigravity. The prime set spans five native variants (OpenCode, Claude Code, OpenClaw, Codex CLI, Antigravity) plus the universal prime, which carries 6-platform auto-detection covering all five native hosts plus Cursor as a degraded fallback. The degradation logic distinguishes between dispatcher-style sub-agents (OpenCode `Task`, Claude `Agent`, OpenClaw `sessions_spawn`, Codex `spawn_agents_on_csv`) and native isolated subagent spawn (Antigravity `invoke_subagent` — async, clean-context, optional Git-worktree isolation; `browser_subagent` is the browser-scoped variant invoked via `/browser`, with parallel multi-agent orchestration available through the Agent Manager).

### The Core Autonomy Engine

The engine is built for autonomy, safety, and host-agnosticism. Key mechanics:

*   **Intelligent platform detection** — runtime tool probing replaces parametric guessing. The agent does not read its system prompt and assume it is on OpenCode; it runs `echo platform_check_$(date +%s)` (or the host-equivalent), tests sub-agent dispatch with a no-op label, and builds an evidence-based capability map.
*   **3-tier verdict system** — `GREEN_FLAG` (clean pass, auto-proceed), `TECH_DEBT` (log to `.studio/todos.md`, auto-proceed), and `BLOCKER` (halt + HaaS + safe rollback) replace the earlier binary RED/GREEN verdict. This eliminates the previous problem where minor stylistic critiques were inflating into hard halts.
*   **Autonomous web-search recovery before HaaS escalation** — when a bug resists three direct fixes, the agent now performs a targeted web search for the exact error message + stack context, applies the result, and retries. The full cycle (3 attempts + web search + 1 retry) can repeat up to **5 cycles × 20 iterations** before any human is bothered.
*   **Scratchpad DAG XML enforcement** — host-agnostic structured reasoning replaces earlier Python daemon hacks. The `<phase_gate_checklist>` is now an inline XML scratchpad that works identically across all 6 supported platforms — OpenCode, Claude Code, OpenClaw, Codex CLI, Antigravity, and Cursor (via the universal degradation fallback). The XML templates are platform-agnostic by design — they're scratchpad memory, not tool invocations.
*   **Apex Red Team Divergent Persona Protocol** — the 3-round adversarial review (steelman → adversarial → synthesis) is now formally specified and runs in a true isolated sub-agent context on all hosts that support dispatch.

---

## ✨ Core Features

Studio Prime enforces rigorous software-engineering standards on every host, transforming the LLM from a stochastic text generator into a deterministic engineering environment:

*   **Proof-of-Work Anti-Hallucination:** LLMs inherently struggle to differentiate between *intended* outcomes and *actual* execution results. To counteract this, Studio Prime enforces a strict three-part empirical verification protocol. Before asserting task completion, the agent must generate a `<prediction>` of the expected terminal output, execute the command via the host-native shell, and synthesize a `<divergence_analysis>`. By comparing the expected state against the actual `stdout`, the agent is forced to linguistically process any discrepancies. Furthermore, if a successful output is indistinguishable from a generic response, the system injects a deliberately malformed flag into a subsequent execution to empirically verify the terminal's responsiveness, thereby eliminating blind assumptions.

*   **The 2026 Impeccable Design Standard:** Studio Prime categorically rejects the homogenized "AI Starter Pack" aesthetic. It enforces a strict mathematical approach to UI engineering, dictating the use of the `OKLCH` color space for perceptual uniformity and accessibility. It mandates physics-based, 120fps GPU-accelerated motion (utilizing libraries such as `motion/react` for fluid, spring-based interactions), while explicitly prohibiting ubiquitous anti-patterns like absolute blacks (`#000000`), nested card hierarchies, and unstructured glassmorphism. Every interactive component must explicitly style the full state matrix: default, hover, focus (with visible ring), active/pressed, and disabled.

*   **Platform-Native TUI Synchronization:** When running on platforms with native task tools — OpenCode's `todowrite`, Claude Code's `TaskCreate`/`TaskUpdate`, Codex CLI's `update_plan`, Antigravity's auto-generated Task List artifact (`task.md`) — the agent dynamically mirrors its internal Directed Acyclic Graph (DAG) state to the external UI task panel. This bidirectional synchronization ensures the human operator maintains perfect, real-time observability of the agent's phase progression without needing to parse the raw context stream. On platforms with no native task tool (OpenClaw, Cursor), the agent maintains a `.studio/todos.md` Markdown checklist on disk, which the human can monitor via their file watcher of choice. Critically, **`.studio/todos.md` is the cross-platform-portable canonical state log on every host regardless of which native task tool is also being driven** — so the `.studio/` markdown tree resumes on any host with **minimal, re-derivable loss**. Native task stores (Claude Code `TaskList`, Codex `update_plan`) and Antigravity Knowledge Items are **host-local and do NOT migrate**; on a cross-host resume they are reconstructed from `.studio/` (the agent rebuilds the native task store from `.studio/todos.md`, and re-seeds Antigravity Knowledge Items from `.studio/decisions.md`). The cross-platform guarantee covers the `.studio/` subset, not the full host-native memory layer.

*   **Autonomous Escalation & State-Preserving Rollback:** Brittle debugging loops are a primary failure vector for autonomous agents. To resolve this, Studio Prime implements a bounded, 5-cycle recovery protocol. When confronted with an error, the agent is granted three direct remediation attempts. Upon failure, it is mathematically forced to pivot: executing targeted web research to inject external context for the specific error, followed by one final retry. After exhausting the repair budget (5 cycles per failure, 20 iterations max), the agent executes a safe git stash, logs forensic data to `.studio/blocked.md`, then auto-isolates the failing module — logging it as a `[PRIORITY:H]` TECH_DEBT entry — and continues the pipeline with the remaining scope. Human-as-a-Service (HaaS) is invoked only when the isolated module is a critical-path dependency; in unattended (Sleep Test) mode the agent checkpoint-exits non-zero instead of blocking on a human. This exit path does NOT apply to deployment when hosting credentials were provided — their provision IS standing authorization, so the agent deploys live and exits `0` with a working URL; if no creds were provided it still hands over with the localhost server running. A non-zero checkpoint-exit is reserved for genuinely unresolved blockers, never for a deploy step it is already authorized to perform. *Studio Prime will NEVER autonomously execute a hard reset.*
*   **Zero-Gap Phase Chaining:** Upon receiving a passing verdict from the Apex Red Team, the agent autonomously transitions to the next phase without pausing for human approval. The agent operates as a continuous pipeline, only halting for explicit blockers or the final Northstar Validation Gate.
*   **Northstar Validation Gate:** Before final sign-off in Phase 6, the agent compares the original Phase 1 requirements (`northstar.md` — immutable, never re-captured) against the final deliverables and writes every gap to a failure-findings file (`northstar_gap_analysis.md`). Isolated gaps trigger surgical remediation: only the phase(s) that own each gap are re-entered (a missing API re-enters P3/P4, an a11y miss P5, a deploy miss P6). If the failure is systemic — smoke tests fail, most requirements are unmet, or the blueprint/architecture itself is implicated — it escalates: the agent re-enters Phase 1 and re-walks the entire pipeline with the failure findings as mandatory input to every phase. A "deploy miss" with standing deploy credentials re-enters P6 and DEPLOYS (the creds already authorize it — never a human escalation); a "deploy miss" with no creds ensures the localhost server is left running (`[LOCAL_LIVE]`). Both paths share a 2-cycle cap, after which non-critical gaps are deferred to `TECH_DEBT` and only critical-path gaps escalate to a human — and only after the bounded retries exhaust.

---

## ⚙️ Function: Orchestration & The Gating System

The operational architecture of Studio Prime is defined by a strictly sequential, 6-Phase lifecycle. Progression through these phases is not determined by heuristic guessing, but rather governed by a deterministic, mathematically verifiable gating system.

### The Scratchpad DAG & Transition Checklists
Before the system is permitted to transition from Phase *N* to Phase *N+1*, it must construct a Directed Acyclic Graph (DAG) in its scratchpad memory (`<phase_gate_checklist>`). This enforces a mandatory sequential evaluation of:
1.  **Prerequisite Verification:** Ensuring all artifact dependencies from Phase *N* exist on the filesystem.
2.  **Proof-of-Work Compilation:** Gathering the raw `stdout` from testing and verification scripts.
3.  **Apex Red Team Adjudication:** Submitting the collected evidence to an isolated sub-agent (or a blinded persona-swap on degraded platforms) for adversarial review.

### The Apex Red Team (Multi-Agent Debate)
The Apex Red Team executes a 3-round Divergent Persona Protocol. On hosts with native sub-agent dispatch (OpenCode `Task`, Claude Code `Agent`, OpenClaw `sessions_spawn`), it runs in a fully isolated context window — the reviewer has *no access* to the developer's conversation history, justifications, or compromises. This guarantees true adversarial review free from context contamination.

*   **Round 1 (Steelman):** The sub-agent adopts the persona of the original developer, tasked with vigorously defending the implementation and citing specific passing tests.
*   **Round 2 (Adversarial):** The persona shifts to a zero-trust security auditor operating under the absolute assumption that a vulnerability *does* exist. It attacks the assertions made in Round 1.
*   **Round 3 (Synthesis):** A principal engineer persona adjudicates the debate, classifying findings into `[BLOCKER]` (halts execution, mandates a safe `git stash` rollback, and triggers HaaS), `[TECH_DEBT]` (logs to `.studio/todos.md` but permits progression), or `[GREEN_FLAG]` (clean pass).

### The 6-Phase Lifecycle

1.  **Phase 1: Blueprint & Baseline** — Establishes the foundational architectural constraints. Initiates mandatory web searches (min 3, recommended 5-10+) to anchor the model in current, verified documentation rather than parametric memory. **Output:** Generates `northstar.md` (immutable requirements) and `decisions.md` (the architectural state machine).
2.  **Phase 2: Link** — Defines the integration seams. This phase enforces the "Data-First Rule," establishing database migration strategies, external API wiring, and identity management before any business logic is written. **Output:** Complete `data_contracts.md` and integration blueprints.
3.  **Phase 3: Architecture & Scaffolding** — Translates the blueprint into syntactically valid code structures. The agent is strictly prohibited from writing business logic. It focuses entirely on defining strict types, interface boundaries, Test-Driven Development (TDD) stubs, production-ready multi-stage **Docker templates (`Dockerfile`, `docker-compose.yml`)**, **automated CI/CD pipelines**, and **reproducible database migration environments**.
4.  **Phase 4: Implement & Verify** — Executes the core business logic to satisfy Phase 3 stubs. Enforces 80%+ code coverage, **EXECUTED E2E/integration tests covering the critical user journeys (failing critical-path tests block the phase)**, a **dependency CVE audit + secrets-leak scan (binary gates)**, **JSON-formatted stdout logging validated through a JSON parser**, and strict **OWASP security hardening** (Zod/Pydantic schema validation, secure HttpOnly/Secure/SameSite cookie configurations, safe JWT structures, and API rate-limiting per IP).
5.  **Phase 5: Stylize (UI/UX Polish)** — Transforms functional interfaces into production-grade aesthetics. Enforces the 2026 Impeccable Design Standard, ensuring **WCAG 2.1 AA accessibility verified via an EXECUTED automated Axe-core/pa11y audit (critical/serious violations are BLOCKERs)**, comprehensive component state matrices, fluid motion physics, and LCP (<2.5s) / CLS (<0.1) benchmarking.
6.  **Phase 6: Release** — The final pre-flight operational check, ending with the product **LIVE**. The kill-based gates run FIRST against a disposable instance — **TESTED graceful OS signal shutdown (timed `SIGTERM` drain ≤35s)**, production **DB migrations (dry-run gated)**, a **timed rollback dry-run (<5min)** — then the FINAL handoff server is started and left running. When hosting/deploy credentials were provided at intake, **their provision IS the authorization**: the agent performs the REAL platform deploy (not just a dry-run), keeps it serving after smoke tests pass, and documents the production URL; rollback fires only on failure (5xx → autonomous safe rollback + HaaS notify), after which it re-attempts or escalates. When no deploy credentials were provided, the agent emits `deploy_ready.sh` for going live later, then BUILDS and STARTS the production artifact as a DETACHED localhost server that survives the session and **leaves it running** (URL + PID recorded in `.studio/state/local_live.md`). It runs **synthetic-error → alert-propagation verification** and **automated Playwright/Cypress smoke tests** against the live environment (deployed URL or `http://localhost:<port>`), re-probes the live URL for HTTP 200 immediately before sign-off, and concludes with a comprehensive 17-section `HANDOFF.md` consolidation (validated placeholder-free, recording the live access point) and the final **Northstar Validation Gate**.

---

## 🔍 Platform Auto-Detection & Degradation

This is the heart of the universal edition. At session start — before Phase 1 even begins — the agent physically probes each tool with a no-op call rather than reading its system prompt and guessing. The probe sweep tries each capability against **five platform variants per slot, sometimes with multiple tool names per platform** (Claude Code has both `WebSearch` and `WebFetch`; Antigravity pairs a web-search capability with `read_url(domain)`): sub-agent dispatch is tested against `Task` / `Agent` / `sessions_spawn` / `spawn_agents_on_csv` / `invoke_subagent` (with the browser-scoped `browser_subagent` variant); shell against `bash` / `Bash` / `exec` / `shell` / `run_command`; web against `webfetch` / `WebSearch` / `WebFetch` / `web_search` / Antigravity's web-search + `read_url(domain)` capability (exact tool names reverse-engineered — adjust to the live engine signature if rejected); task tracking against `todowrite` / `TaskCreate` / `update_plan` / Antigravity's auto-generated Task List artifact (`task.md`) / file-based `.studio/todos.md`; and structured intake against `question` / `AskUserQuestion` / `request_user_input` / plan-checkpoint pattern / Markdown-letter fallback. The capability map produced by this probe routes every subsequent execution decision.

### Degradation Matrix

| Tool | Available | Unavailable (Fallback) |
|---|---|---|
| Sub-agent dispatcher (`Task` / `Agent` / `sessions_spawn` / `spawn_agents_on_csv` / `browser_subagent`) | Full isolated sub-agent for Apex Red Team — true context isolation, the developer's history is invisible to the reviewer. On Codex, the Apex Reviewer is a TOML-defined custom agent in `~/.codex/agents/`. On Antigravity, the Apex Red Team spawns natively via `invoke_subagent` (async, clean context, optional Git-worktree isolation) — never a main-thread persona swap; `browser_subagent` (via `/browser`) handles browser-scoped visual review, and the Agent Manager orchestrates parallel agent teams | Inline persona-swap with explicit **Attention Shift** instruction: heavily discount conversation history, evaluate only against PRD + phase artifacts + checklist + raw stdout |
| Native TUI tasks (`todowrite` / `TaskCreate` / `update_plan` / Antigravity Task List artifact `task.md` / file-based) | Mirror DAG state to TUI in real time so the human sees phase progression without parsing the context stream. `.studio/todos.md` is dual-written on every platform regardless of which native tool is also driven | Maintain `.studio/todos.md` Markdown checklist on disk; human monitors via file watcher |
| Web research (`webfetch` / `WebSearch` / `WebFetch` / `web_search` / Antigravity web-search + `read_url(domain)`) | Execute research gate normally — minimum 3 targeted searches per phase, each producing at least one `<assumption_update>` | Ask human for URLs or pasted documentation; if human unavailable, proceed and tag work `[TECH_DEBT: Unverified Assumptions]` |
| Shell (`bash` / `Bash` / `exec` / `shell` / `run_command`) | Full proof-of-work verification with `<prediction>` → `<execution>` → `<divergence_analysis>` | **CRITICAL DEGRADATION.** Every artifact is appended with `[UNVERIFIED - NO SHELL]`. The agent cannot honestly claim "tests pass" without a shell |
| Structured questions (`question` / `AskUserQuestion` / `request_user_input` / plan-checkpoint / none) | JSON option arrays rendered as a clean TUI menu (or, on Antigravity, surfaced via `/grill-me` native clarifying-questions or as an Implementation-Plan approval checkpoint in Planning Mode that the human reviews before execution) | Markdown letter-list (`A.` / `B.` / `C.`) inline in chat |

### Platform Support Matrix

| Platform | Sub-Agent | Terminal | Web | TUI Tasks | Intake | Sleep Test |
|---|---|---|---|---|---|---|
| **OpenCode** | ✅ `Task` (native XML) | ✅ `bash` | ✅ `webfetch` | ✅ `todowrite` | ✅ `question` | ✅ FULL |
| **Claude Code** | ✅ `Agent` (via Task tool) | ✅ `Bash` | ✅ `WebSearch` + `WebFetch` | ✅ `TaskCreate` / `TaskUpdate` | ✅ `AskUserQuestion` | ⚠️ Partial (manual `/compact`) |
| **OpenClaw** | ✅ `sessions_spawn` (JS) | ✅ `exec` (`pty:true`) | ✅ `web_search` | ⚠️ File-based (`.studio/todos.md`) | ⚠️ Markdown letters | ⚠️ Partial (PTY required) |
| **Codex CLI** | ✅ `spawn_agents_on_csv` (CSV-batch — true parallelism reserved for independent research fan-out; Apex rounds dispatched sequentially) + custom agents (`~/.codex/agents/*.toml`) | ✅ `shell` (PTY-backed) | ✅ `web_search` | ✅ `update_plan` + `.studio/todos.md` dual-write | ✅ `request_user_input` | ✅ FULL (`codex exec` for Sleep Test) |
| **Antigravity** | ✅ `invoke_subagent` (async, clean-context, Git-worktree isolation; `browser_subagent` for browser-scoped review; Agent Manager for parallel teams) | ✅ `run_command` | ✅ web-search + `read_url(domain)` | ✅ Task List artifact (`task.md`) + `.studio/todos.md` | ⚠️ `/grill-me` or Planning-Mode plan-approval checkpoint | ✅ FULL (Agent-driven autonomy + `/goal` + `agy -p` headless) |
| **Cursor** | ❌ Manual | ❌ Script required | ❌ Manual | ❌ Manual | ⚠️ Markdown letter list (universal fallback) | ❌ NO |

The "Sleep Test" column measures whether you can start a session, walk away, and return to a complete, audited application. **Three hosts pass the Sleep Test fully:** OpenCode (true sub-agent isolation + automatic context compaction + native HaaS UI), Codex CLI (`codex exec --sandbox workspace-write -a never` for fully unattended runs with TOML-defined custom agents), and Antigravity (Agent-driven autonomy + `/goal` run-to-completion + headless `agy -p` with `--dangerously-skip-permissions` for CI + durable Artifacts that survive session crashes). Claude Code and OpenClaw run Partial Sleep Tests because they require either manual `/compact` checkpoints or PTY-tethered execution; Cursor cannot run the Sleep Test at all and falls back to a semi-autonomous, human-in-the-loop pattern.

**Localhost-survival note (per host — the no-creds `[LOCAL_LIVE]` path):** a detached localhost server only counts as `[LOCAL_LIVE]` if it survives the agent session/process teardown on THIS host — which is NOT guaranteed and is PROVEN by a detach-survival probe (spawn a sleeper via the same backgrounding idiom, exit the parent shell, confirm from a fresh shell that it still answers), never assumed. **OpenCode / Claude Code:** bare `nohup`/`setsid` children may live in a session/cgroup the harness tears down — prefer `docker compose up -d` (daemon-owned) or a `pm2`/`tmux`/`screen` supervisor. **OpenClaw:** PTY-tethered; a bare background process commonly dies with the PTY session, so daemon-owned supervision is required. **Codex CLI:** sandbox/session-scoped `shell`; same caveat. **Antigravity:** the `run_command` background handle is engine-managed; verify survival from a fresh invocation. **Windows/PowerShell hosts:** `Start-Process` children are killed when the parent job object closes — use a Scheduled Task / detached `cmd /c start` wrapper / `nssm`/`pm2`, or `docker compose up -d`. If the probe FAILS on your host, you get an honest `[DEPLOY_READY: localhost cannot outlive this session — start via deploy_ready.sh]` instead of a false LIVE claim.

### How Detection Actually Runs

Concretely, the first thing the universal prime does on a fresh trigger is emit a `<tool_availability_check>` block. Inside it:

1.  **Shell probe.** The agent attempts a minimal `echo` with a timestamped token, trying in this order: `bash`, `Bash`, `exec` (`pty:true`), `shell`, `run_command`. If any one returns `stdout` containing the token, shell is `YES`; otherwise it is marked `CRITICAL` and every subsequent artifact will carry the `[UNVERIFIED - NO SHELL]` warning.
2.  **Sub-agent probe.** The agent attempts to dispatch a sub-agent with a one-line task: "Reply with `PONG_<timestamp>`." It tries, in this order: `Task` (OpenCode), `Agent` (Claude Code), `sessions_spawn` (OpenClaw), `spawn_agents_on_csv` (Codex CLI), and `invoke_subagent` (Antigravity — native async isolated spawn; the `browser_subagent` variant is browser-scoped). If the reply arrives within the dispatch timeout, isolated review is enabled; otherwise the Apex Red Team will run inline with the **Attention Shift** instruction.
3.  **Task-tool probe.** The agent attempts to create a single ephemeral task entry titled `__platform_check__` and then deletes it, trying in this order: `todowrite`, `TaskCreate`/`TaskUpdate`, `update_plan` (Codex), Antigravity's auto-generated Task List artifact (`task.md`). If any round-trip succeeds, TUI mirroring is enabled and `.studio/todos.md` is dual-written. Otherwise the agent falls back to `.studio/todos.md` as the sole canonical state log.
4.  **Web probe.** The agent attempts a single search for a known stable URL (e.g., a well-known documentation page), trying in this order: `webfetch`, `WebSearch` + `WebFetch`, `web_search`, Antigravity's web-search + `read_url(domain)` capability (exact tool names reverse-engineered — adjust to the live engine signature if rejected). If results return, the research gate runs normally; otherwise the gate degrades to human-supplied URLs or `[TECH_DEBT: Unverified Assumptions]`.
5.  **Question-tool probe.** The agent inspects whether structured questions are available, trying in this order: `question` (OpenCode), `AskUserQuestion` (Claude Code), `request_user_input` (Codex), Antigravity's `/grill-me` clarifying-questions / Planning-Mode plan-approval checkpoint. If none are available, all intake gates render as Markdown letter lists.

Probe results are persisted to `.studio/state/platform_capabilities.md`. On Resume Protocol, the file is re-read; if `probed_at` is missing or >24h old, a fresh probe runs. The persistent capability map is the cache; a fresh probe is the source of truth when the cache is stale — this guards against host upgrades that have silently added or removed tools.

---

## 🚀 Installation

Pick the installation block for your host. Each block offers **Option A (system prompt)** and **Option B (on-demand skill)**.

### OpenCode (Recommended)

```bash
# Option A: System prompt — always active
mkdir -p .opencode
cp opencode_prime.md .opencode/system.md

# Option B: On-demand skill
cp opencode_prime.md .opencode/skills/studio-prime/SKILL.md

opencode
# Trigger: "Start Studio Prime"
```

### Claude Code

```bash
# Option A: Project-level system prompt
cp claudecode_prime.md CLAUDE.md

# Option B: On-demand skill
cp claudecode_prime.md .claude/skills/studio-prime/SKILL.md

# Configure .claude/settings.local.json to allow:
# Bash, Read, Write, Edit, Glob, Grep, TaskCreate, TaskUpdate, TaskList, TaskGet, AskUserQuestion, WebSearch, WebFetch, Agent
claude
# Trigger: "Start Studio Prime"
```

### OpenClaw

```bash
# Option A: Workspace AGENTS file
cp openclaw_prime.md AGENTS.md

# Option B: On-demand skill
cp openclaw_prime.md .openclaw/skills/studio-prime/SKILL.md

openclaw agent --message "Start Studio Prime"
```

### Codex CLI (OpenAI)

```bash
# Install Codex CLI itself
npm i -g @openai/codex   # OR: brew install --cask codex

# Auth: before first interactive use, run `codex login` (browser OAuth flow)
#       OR set CODEX_API_KEY env var (see Sleep-Test below).

# Project-level system prompt (recommended)
cp codex_prime.md AGENTS.md
# OR global: cp codex_prime.md ~/.codex/AGENTS.md
# OR skill:  cp codex_prime.md ~/.codex/skills/studio-prime/SKILL.md

# IMPORTANT: bump Codex's doc-size cap — both codex_prime.md and the universal
# studio_prime.md exceed Codex's 32 KiB default (project_doc_max_bytes),
# which would silently truncate the prime. In ~/.codex/config.toml set:
#   project_doc_max_bytes = 131072

# Sleep-Test (unattended) mode
#   FIRST run `codex login status` (or `codex login --help`) ONCE to confirm which env
#   var your installed Codex honors, and which key is present — do NOT start an overnight
#   run on a guessed env-var name. Studio Prime detects and records this at Self-Setup; if
#   NEITHER CODEX_API_KEY nor OPENAI_API_KEY is set, it HALTS at setup with a clear message
#   ("no Codex auth key found — set CODEX_API_KEY and restart") rather than launching a run
#   that cannot authenticate.
CODEX_API_KEY=sk-... codex exec --sandbox workspace-write -a never "Start Studio Prime"
# Some Codex CLI versions honor OPENAI_API_KEY=sk-... as a fallback instead — the detected,
# recorded env var from setup is authoritative.

# Interactive
codex
```

Codex CLI uses `~/.codex/agents/*.toml` for custom-agent definitions (the Apex Reviewer ships as a TOML), `~/.codex/hooks.json` for pre/post-tool hooks, and `apply_patch` V4A diffs for every file mutation. The universal prime auto-detects the Codex toolset by probing `spawn_agents_on_csv`, `shell`, `update_plan`, `request_user_input`, and `web_search` at session start.

### Antigravity (Google)

```bash
# Install Antigravity desktop app from antigravity.google/download
# CLI (agy) is bundled — run setup wizard to install
# Auth: setup wizard prompts for Google sign-in on first launch
#       (applies to both the `agy` CLI and the desktop app).

# Project-level cross-tool rules file (primary)
cp antigravity_prime.md AGENTS.md
# Antigravity/Gemini-specific (coexists; legacy back-compat)
cp antigravity_prime.md GEMINI.md
# Global rules: cp antigravity_prime.md ~/.gemini/GEMINI.md

# Note: AGENTS.md is the primary cross-tool project-rules file (shared with
# Cursor, Claude Code, Codex CLI). Reported precedence is AGENTS.md > GEMINI.md >
# defaults, though this ordering is contested across sources — adjust if the live
# engine disagrees. GEMINI.md is the coexisting Antigravity/Gemini-CLI file kept
# for back-compat.

# Launch
agy                         # CLI TUI
agy -p "Start Studio Prime" # Headless one-shot
# OR open the Antigravity desktop app
```

Antigravity 2.0 ships with two execution modes (Planning Mode and Fast Mode) and four autonomy modes (Secure / Review-driven / Agent-driven / Custom) plus per-action policies (Terminal Execution allow/deny, JavaScript Browser policy) — these are the real gating surface for Studio Prime's phase progression and destructive-command control, not engine-enforced phase modes. (Planning / Execution / Verification remain Studio Prime's own workflow PHASES, not Antigravity engine modes.) Durable Artifacts (Task List, Implementation Plan, Walkthrough, Code Diffs, Screenshots, Browser Recordings — WebP format unconfirmed) survive session restarts, and `browser_subagent` (invoked via `/browser`; requires a Chrome extension) autonomously verifies UI work by clicking through your live app and capturing screenshots/recordings. Skills packages in `<repo>/.agents/skills/<name>/` (each with a required `SKILL.md`) provide persistent, queryable lore across sessions. For unattended Sleep-Test runs, use Agent-driven autonomy + `/goal` run-to-completion + headless `agy -p` (with `--dangerously-skip-permissions` for CI). When deploy credentials were provided, keep Terminal Execution set to allow for the deploy commands so the live-deploy step is not interactively gated — deploy commands executed under standing authorization are not "destructive commands"; the unattended green-flag criteria for P6 INCLUDE the live end-state (a deployed URL or a running `[LOCAL_LIVE]` localhost server).

### Cursor / Other Hosts (Universal mode)

```bash
# Use the universal prompt — it will auto-detect what is available.
# The canonical universal file is studio_prime.md.
cp studio_prime.md <wherever-your-host-loads-system-prompts>

# Trigger: "Start Studio Prime"
# The prompt will physically probe its toolbelt and degrade gracefully.
```

### The Intake Gate

Once the agent is running and you trigger it, Studio Prime will **NOT** dump a wall of text. It performs platform detection first, then opens an intake gate asking whether you want to start a **NEW PROJECT** (guided discovery) or work on an **EXISTING CODEBASE** (add features or transform). The gate uses the platform's structured-question tool if available, falling back to a Markdown letter list otherwise.

---

## 🛡️ Safety Features

### Human-as-a-Service (HaaS)

Studio Prime is fully autonomous. The human is treated as a blocking API, invoked only when the agent genuinely cannot proceed alone. Authorized, credentialed deployment is never a HaaS trigger — when deploy creds are provided at intake the agent deploys live on its own; only unrequested destructive/network actions and the categories below gate. HaaS triggers fall into five categories (matching the five enumerated HaaS gates in the prime):

1.  **Destructive & Network Gates** — The following commands ALWAYS require human authorization: `rm -rf`, `npm publish`, database drops, `force-push`, `chmod 777`, any `curl`/`wget`/`nc`/`netcat` sending data to external hosts, any `curl|wget` piping to `sh`/`bash`/`python`, and port-scanning tools like `nmap`/`masscan`. Legitimate API calls are allowed without authorization, but suspicious patterns are flagged. **Deploy carve-out:** when hosting/deploy credentials were supplied at intake, deployment commands targeting the user-supplied host with those creds (registry push, the Vercel/Netlify/Fly/Render CLIs, SSH push, and any `npm publish` / prod migration that is part of the authorized deploy) are PRE-AUTHORIZED and do NOT trip this gate — the provision of credentials IS the authorization. The gate still fires for unrequested publishes or exfiltration.
2.  **Credential Gaps** — API keys, OAuth tokens, or external service accounts that the agent cannot auto-generate trigger an immediate HaaS request with a clear list of what is needed and why. Conversely, hosting/deploy credentials supplied at intake constitute explicit STANDING AUTHORIZATION to deploy live without any further human gate — they are not a gap and never trigger a deploy checkpoint.
3.  **Unresolved PRD Conflicts** — When the agent detects that two requirements point to materially different product outcomes (e.g., "real-time" vs. "eventually consistent"), it halts and presents the conflict for human decision rather than guessing.
4.  **Architecture / Research Overrides** — An architecture or research gate that requires explicit human authorization to proceed (a deployment override only when **no** deploy credentials were provided — credentialed deploys are pre-authorized and never gate).
5.  **Repair Limit Reached** — After exhausting the repair budget (5 cycles per failure, 20 iterations max), the agent executes a safe git stash, logs forensic data to `.studio/blocked.md`, then auto-isolates the failing module — logging it as a `[PRIORITY:H]` TECH_DEBT entry — and continues the pipeline with the remaining scope. HaaS is invoked only when the isolated module is a critical-path dependency; in unattended (Sleep Test) mode the agent checkpoint-exits non-zero instead of blocking on a human.

### Zero-Trust Security

*   **Input Validation:** Every external input is validated before processing. No "I trust this came from our frontend" — boundary inputs are typed, range-checked, and sanitized.
*   **Secrets Handling:** API keys live in `.env`. Never committed. Pre-commit hooks check for credential leakage.
*   **Auth Checks:** Every protected endpoint has its authentication verified during the Apex Red Team review. The reviewer specifically searches for missing middleware on routes that handle PII or financial data.

---

## 📁 File Structure & Reasoning

Studio Prime writes its state to disk to cure "LLM Amnesia" and survive context limits. The same canonical tree is created on every host:

```text
.studio/
├── todos.md                # Active task list (mirrored in TUI on supported hosts)
├── decisions.md            # Cross-platform canonical mirror of architecture/decisions.md (universal prime)
├── archive.md              # Flushed tasks to keep the active context small
├── blocked.md              # Detailed logs of failed escalation attempts
├── state/
│   ├── northstar.md        # The immutable original requirements
│   └── phase*_research.md  # Raw search findings
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

### Platform-Specific Path Addenda

Beyond the canonical `.studio/`, `architecture/`, `design-system/`, and `.tmp/` trees (which are identical on every host), each platform adds its own host-specific configuration roots. Studio Prime writes to these only when it detects the corresponding platform — never speculatively.

**Codex CLI:**

```text
~/.codex/
├── AGENTS.md               # Global system prompt (overridable by project-level AGENTS.md)
├── agents/
│   └── apex-reviewer.toml  # The Apex Red Team as a TOML-defined custom agent
├── skills/
│   └── studio-prime/SKILL.md  # On-demand skill variant
└── hooks.json              # Pre/post-tool execution hooks for verification gates
```

**Antigravity:**

```text
~/.gemini/
├── GEMINI.md               # Global rules (Antigravity/Gemini-CLI; legacy back-compat)
└── antigravity/
    └── skills/             # Global Skills packages

<project>/
├── AGENTS.md               # Primary cross-tool rules (shared with Claude Code / Codex / Cursor; precedence over GEMINI.md is reported but contested)
├── GEMINI.md               # Coexisting Antigravity/Gemini rules file
└── .agents/
    ├── skills/<name>/SKILL.md   # Workspace Skills packages (required SKILL.md)
    ├── workflows/          # Workspace workflows
    └── rules/              # Workspace rules
```

The universal prime never writes to these locations unless its probe confirms the matching toolset. If you migrate a project between platforms, only the canonical `.studio/` tree is required to be preserved; the host-specific roots are regenerated automatically.

---

## 🎯 Choosing Your Platform

If you are evaluating which host to run Studio Prime on, the following decision tree maps the most common selection criteria to the recommended platform. All six paths converge on the same six-phase lifecycle and the same `.studio/` canonical state log — they differ only in their native dispatchers, sleep-test fidelity, and editor surface.

*   **Pick OpenCode** if you want the most autonomous Sleep-Test experience with native sub-agent isolation. The `Task` tool gives the Apex Red Team a truly clean context window, and `todowrite` mirrors phase state directly into the TUI.
*   **Pick Claude Code** if you want Claude Sonnet/Opus models + hooks-driven verification + `AskUserQuestion`-style structured prompts. Claude Code's `Agent` dispatcher plus `.claude/settings.local.json` permission-scoping make it the most operator-controllable host.
*   **Pick OpenClaw** if you need multi-channel invocation (Discord/Slack/CLI/HTTP) for team workflows. OpenClaw's `sessions_spawn` runs the Red Team in a JS-isolated session, and its PTY-backed `exec` keeps shell verification rigorous.
*   **Pick Codex CLI** if you want the most CI/CD-friendly Sleep-Test mode (`codex exec --sandbox workspace-write -a never`), parallel CSV-batch agents (`spawn_agents_on_csv`), and OpenAI/local model flexibility. Codex's `apply_patch` V4A diffs make file mutation auditable down to the byte.
*   **Pick Antigravity** if you want a standalone agent-orchestration desktop cockpit (no built-in editor — dual-wield with your preferred editor) with visual UI verification via `browser_subagent` (`/browser`), durable Artifacts (Task List, Implementation Plan, Walkthrough, Code Diffs, Screenshots, Browser Recordings) that survive crashes, native isolated subagents via `invoke_subagent` with Git-worktree isolation, and unlimited parallel agent teams in the Agent Manager. Its four autonomy modes (Secure / Review-driven / Agent-driven / Custom) plus per-action policies give the finest-grained gating of any host, and Agent-driven autonomy + `/goal` + `agy -p` deliver fully unattended Sleep-Test runs.
*   **Use the universal `studio_prime.md`** (the canonical universal file) on any platform via auto-detection — including Cursor (degraded semi-autonomous mode). This is the right choice if you do not yet know which host you will run on, if you maintain projects across multiple hosts, or if you want behavioral parity for a team that is split across editors.

---

## 🔄 Resumption

To resume an interrupted session, relaunch your host's agent and use one of these trigger phrases:

*   `"Continue Studio Prime"`
*   `"resume"`
*   `"pick up where we left off"`

The agent then executes the **Resume Protocol**:

1.  **Check for `.studio/` directory.** If missing, warn the user and offer to start fresh.
2.  **Re-read `platform_capabilities.md`** (re-probe if `probed_at` is missing or >24h old). `execution_mode` is **sticky-downward**: once recorded as `unattended`, a re-probe MUST NOT silently flip it back to interactive unless a positive interactive signal is present (an explicit human message in the resume trigger, or `STUDIO_INTERACTIVE=1`) — this prevents a re-probe from reintroducing a blocking stdin wait on a headless resume.
3.  **Read `context_checkpoint.md` FIRST if present**, and resume from its `next_concrete_action` as the authoritative restart pointer (discard it as stale if its `current_phase` is earlier than the latest committed state). Then read `todos.md` + `decisions.md`, `state/*`, and run `git status` to re-orient.
4.  **Session Coherence Check.** Reads `architecture/decisions.md` **completely** — this is the ONE protocol-named full read that is an explicit exception to Retrieval Discipline ("targeted grep-then-read for recall; full reads only where a protocol step names one"), kept safe by the standing read-size cap. It checks for drift across four axes: tech stack conflicts, data model conflicts, auth conflicts, and PRD alignment. If drift is detected, the agent presents the specifics and waits for resolution rather than silently overwriting.
5.  **Check for pending Apex Red Team verdicts.** If a phase ended without a recorded verdict, the Red Team is invoked retroactively before any new work begins.
6.  **Resume from last position**, consulting the side-effect ledger so a re-walk/resume never re-applies an already-completed non-idempotent action (deploy, migration, publish) without an idempotency check.

---

## 📥 Expected Input/Output (I/O)

To effectively orchestrate the development lifecycle, Studio Prime expects structured inputs and generates deterministic outputs. Understanding this I/O contract is critical for maximizing the system's efficacy.

### Input Modalities
*   **Intake Triggers:** Natural language commands such as `"Start Studio Prime"` or `"Continue Studio Prime"`.
*   **Requirement Documents (PRDs):** Structured text outlining the core problem, target audience, and functional requirements.
*   **Design Assets & Constraints:** Optional guidelines, reference URLs, or brand tokens. Studio Prime will extract OKLCH tokens from uploaded brand assets if provided.
*   **Human-as-a-Service (HaaS) Feedback:** Explicit approvals, architectural overrides, or reactive credential provisions when the system halts at an enforced gate (destructive command detection, Red Team blocker, repair-limit exhaustion, PRD conflict). (Distinct from intake-time hosting/deploy credentials, which are STANDING deploy authorization — they cause no halt and require no feedback; the agent deploys live with them autonomously.)

### Output Modalities
*   **Terminal Execution (`stdout`/`stderr`):** Verifiable proof of execution, testing, and deployment — pasted raw into `<phase_gate_checklist>` blocks, never summarized.
*   **Persistent State Files (`.studio/`):** Markdown and JSON artifacts that maintain the agent's memory, architectural decisions, and active task lists.
*   **Production Code:** Fully typed, tested, and linted source code adhering to established conventions.
*   **Structured UI Prompts:** Interactive questions rendered via the host's structured-question tool when available, or as Markdown letter lists otherwise.
*   **Adversarial Review Logs:** Detailed markdown reports from the Apex Red Team, classifying each phase's output into `[GREEN_FLAG]`, `[TECH_DEBT]`, or `[BLOCKER]`, persisted to `.studio/apex_red_team/reviews/phase[N]_verdict.md`.

---

## 🔬 Credits & Scientific Foundation

Studio Prime's mechanics are not arbitrary; they are implementations of peer-reviewed AI and LLM research:

*   **Apex Red Team (Multi-Agent Debate):** Based on research like *ChatEval: Towards Better LLM-based Evaluators through Multi-Agent Debate (Chen et al., 2023)*. Studies prove that assigning distinct, isolated adversarial roles reduces LLM hallucinations and false positives far better than single-agent self-critique. The universal prime's native sub-agent dispatch on OpenCode (`Task`), Claude Code (`Agent`), OpenClaw (`sessions_spawn`), Codex CLI (`spawn_agents_on_csv` + TOML custom agents), and Antigravity (`invoke_subagent` native isolated spawn with Git-worktree isolation + `browser_subagent` for visual review + Agent Manager for parallel teams) makes this physical isolation possible across all five dispatcher-equipped hosts. Each blinded review round is dispatched as its own fresh-context sub-agent, but the three rounds run SEQUENTIALLY — Round 2 attacks Round 1's assertions and Round 3 adjudicates both, so they cannot be independent parallel workers; the parallel-dispatch primitives (`spawn_agents_on_csv` CSV batches, the Agent Manager) are reserved for independent research fan-out. On Cursor and other dispatcher-less hosts, the prime falls back to the inline persona-swap with explicit Attention Shift.
*   **Proof-of-Work (Verbal Reinforcement):** Rooted in *Reflexion: Language Agents with Verbal Reinforcement Learning (Shinn et al., 2023)*. Converting raw environmental feedback (terminal stdout) into a linguistic self-reflection (`<divergence_analysis>`) allows the agent to break out of repetitive failure loops.
*   **Context Compaction (`.studio/`):** Based on architectures like *MemGPT: Towards LLMs as Operating Systems (Packer et al., 2023)*. Writing state to a persistent filesystem and retrieving it Just-In-Time (`grep`/`read`) prevents the "Lost in the Middle" context degradation phenomenon that plagues long autonomous sessions.

---

## 📜 License & Contributing

Studio Prime is released under the [MIT License](LICENSE) — use it, fork it, ship with it, commercially or otherwise. Attribution is appreciated.

Contributions are welcome: tool-surface drift fixes, autonomy/Sleep-Test improvements, new platform targets, and parity fixes. Read [CONTRIBUTING.md](CONTRIBUTING.md) for the bar every prime must meet (the Sleep Test, production-grade proof-of-work, no hallucinated tool signatures, cross-platform parity) and the PR workflow.

---
*Architected by Suraj Kuncham. Designed to push the boundaries of autonomous software engineering.*
