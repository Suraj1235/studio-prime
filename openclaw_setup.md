# OpenClaw Prime

[← Back to README](README.md) · Setup guide for [`openclaw_prime.md`](openclaw_prime.md)

**The Studio Prime superprompt — optimized for OpenClaw's native `sessions_spawn`, PTY-backed exec, and multi-channel (Discord/Slack/CLI) invocation.**

---

## What is OpenClaw Prime?

OpenClaw Prime is more than a prompt — it is a complete production engineering playbook running on a multi-channel gateway. It transforms your standard AI coding assistant into a relentless, autonomous Principal Architect, UI/UX Designer, and Zero-Trust Red Teamer accessible from Discord, Slack, or your terminal — all backed by the same underlying agent instance.

While the Studio Prime architecture runs on multiple platforms, **OpenClaw is the definitive environment for distributed, multi-surface orchestration.** This is because OpenClaw natively supports true sub-agent dispatch via `sessions_spawn` *and* exposes PTY-backed shell execution through its `exec` tool. When Studio Prime launches its Apex Red Team to audit code, OpenClaw dispatches the reviewer via `sessions_spawn` under `agentId: "reviewer"`, and the spawn task body enforces a strict BLINDED protocol: the reviewer receives ONLY the North Star, the phase artifacts, the checklist, and the raw stdout — no conversation history, no developer notes, no prior attempts. The Red Team doesn't know the developer's original assumptions, struggles, or compromises. This guarantees **true adversarial review**, free from the context contamination that plagues simpler platforms.

With OpenClaw Prime, you achieve the ultimate "Sleep Test" (**Partial**): Start a session, go to sleep, and wake up to a LIVE product — a working deployed URL when hosting credentials were provided (the credentials ARE the deploy authorization, so it deploys without asking), or a fully functional product with its localhost server still running (URL + sessionId/PID documented) when they were not. The "Partial" caveat applies because long-running background jobs (dev servers, watchers, daemons) require PTY allocation via `exec pty:true background:true`, and a PTY-tethered session is not guaranteed to outlive the agent on every host. So on the no-creds path the agent first runs a **detach-survival probe** — it launches a throwaway sleeper via the same backgrounding idiom, lets the parent shell close, and from a fresh shell confirms the process is still alive — and only a PASSING probe licenses the `[LOCAL_LIVE]` claim. It prefers daemon-owned supervision (`docker compose up -d`, `systemd --user`, `pm2`, detached `tmux`/`screen`; on Windows a Scheduled Task or `nssm`) over a bare backgrounded session. At handoff it leaves that verified-surviving server running, records its `sessionId`/PID + URL in `.studio/state/local_live.md`, and never kills it. If the probe shows localhost cannot survive on this host, the agent says so honestly (`[DEPLOY_READY: localhost cannot outlive this session]`) rather than claiming a live server that will be dead by morning.

> **Before you walk away — set a spend cap.** An unattended run fans out web research across 6 phases, a parallel build swarm, an adversarial reviewer at every gate, and up to two full pipeline re-walks — a non-trivial PRD is many millions of tokens. Studio Prime cannot observe a token/dollar total the engine doesn't expose, so **set a provider-side hard spend limit (Anthropic/OpenAI/Google all support this) before an overnight run.** Optionally export `STUDIO_MAX_WALLCLOCK_HRS` / `STUDIO_MAX_TOOL_CALLS` and the agent tracks them in `.studio/state/budget.md`, checkpoint-exiting non-zero on breach — these are observable proxies, not a money cap.

> **Scope: web products.** The Sleep Test's terminal liveness condition (an HTTP-200 against a deployed URL or localhost) is web-shaped by construction, so web products are the default supported target. For a non-web PRD (mobile / desktop / CLI / library / data-pipeline) the agent detects the target-class in Phase 1 and either records a substituted terminal condition in `decisions.md` (e.g. mobile = built artifact + emulator smoke run; CLI/library = executed binary/import smoke + installable artifact) or checkpoint-exits with `[UNSUPPORTED_TARGET: non-web]` — it never silently forces a non-web project into a web shape.

> **Re-run / resume safety.** "Continue Studio Prime" and the Northstar re-walks are idempotency-guarded: a side-effect ledger (`.studio/state/effects.md`) records deploy ids, applied migration versions, publish versions, and synthetic-error injections, so a resume or re-walk SKIPS or guards already-applied effects rather than double-deploying, re-migrating, or re-firing real alerts. Resume reads `context_checkpoint.md` first (the authoritative restart pointer) and relaunches any dead background job rather than assuming it can re-attach to a prior PTY-tethered session.

---

## 🚀 Setup Process

Follow these steps exactly to install OpenClaw Prime and unleash its full capabilities.

### Option A: As AGENTS.md (Recommended, System-Level)
This method ensures Studio Prime is active by default every time you open a workspace, and travels with the repository for the whole team.

1. **Open your terminal** and navigate to your project workspace.
2. **Copy the superprompt** into the workspace agents file:
   ```bash
   cp openclaw_prime.md AGENTS.md
   ```
3. **Trigger the Agent**:
   ```bash
   openclaw agent --message "Start Studio Prime"
   ```

### Option B: As an On-Demand Skill
This method allows you to load Studio Prime modularly only when you explicitly want to use it.

1. **Create the skills directory**:
   ```bash
   mkdir -p .openclaw/skills/studio-prime
   ```
2. **Copy the superprompt** into the skill file:
   ```bash
   cp openclaw_prime.md .openclaw/skills/studio-prime/SKILL.md
   ```
3. **Install and trigger**:
   ```bash
   openclaw skills install
   /subagents spawn --task "Start Studio Prime"
   ```

### Multi-Channel Triggers

Because OpenClaw is a gateway, the same agent can be invoked from multiple surfaces — and Studio Prime is designed to recognize all of them:

*   **CLI:** `openclaw agent --message "Start Studio Prime"`
*   **Discord:** `/subagents spawn --task "Start Studio Prime"`
*   **Slack:** `/subagents spawn --task "Start Studio Prime"`
*   **HTTP API:** e.g. `POST /v1/sessions/spawn` (your OpenClaw gateway's spawn route) with body `{ "task": "Start Studio Prime" }`

On its first response, the agent echoes a canonical, channel-safe bracketed header so the user knows which OpenClaw instance is replying — this matters when running multiple workspaces:

```
[OpenClaw | channel=<CLI|Discord|Slack|API|HTTP> | agent=studio-prime | session=<short-id>]
```

Pipe separators are used instead of bullets (`•`) because bullets render inconsistently in Slack and Discord proportional fonts.

### Triggering the Agent

Once OpenClaw is running, trigger the system by typing:

```
"Start Studio Prime"
```

**The Intake Gate:** Upon invocation, OpenClaw Prime will NOT dump a massive wall of text. It FIRST auto-classifies the project as NEW vs EXISTING from workspace markers (scanning for `package.json`, `pyproject.toml`, `.git/`, `src/`, and similar) and logs the result as `[AUTO-INTAKE: ...]`. ONLY when that classification is ambiguous does it present a clean **markdown letter-list** asking if you want to start a **A. NEW PROJECT** (guided discovery) or work on an **B. EXISTING CODEBASE** (add features or transform) — ASCII boxes are explicitly forbidden because they render broken in Slack and Discord. Unattended (Sleep Test) runs never show the letter-list: they always proceed on the auto-classification (defaulting to the safe, non-destructive EXISTING path when ambiguous). That "safe, non-destructive default" bias governs intake routing only — it does NOT apply to deploying: when hosting/deploy credentials were provided, deploying live IS the authorized safe path (the credentials are standing authorization), so the agent deploys rather than declining.

---

## ✨ Core Features

OpenClaw Prime leverages OpenClaw's native APIs to enforce rigorous software engineering standards, transforming the LLM from a stochastic text generator into a deterministic engineering environment:

*   **Proof-of-Work Anti-Hallucination (disk-anchored; writer ≠ grader):** LLMs inherently struggle to differentiate between intended outcomes and actual execution results. To counteract this, OpenClaw Prime enforces a strict empirical verification protocol that cannot be satisfied from memory. The agent writes a `<prediction>` BEFORE running, executes each gate command captured TO DISK (`<cmd> 2>&1 | tee .studio/state/pow/p{N}_c{K}.log; echo RC=$? TS=$(date -u +%s%N)`), then populates `<divergence_analysis>` by RE-READING that log file — never typing stdout. Each capture carries an unguessable wall-clock-nanosecond `RC/TS` liveness nonce tied to the same invocation; a `<proof_of_work>` block with no corresponding on-disk `.log` is auto-classified `[UNVERIFIED]` → BLOCKER. The dispatched Apex reviewer (a separate, clean context) grades ONLY the on-disk logs, so the writer and the grader are never the same pass.
*   **The 2026 Impeccable Design Standard:** Studio Prime categorically rejects the homogenized "AI Starter Pack" aesthetic. It enforces a strict mathematical approach to UI engineering, dictating the use of the `OKLCH` color space for perceptual uniformity and accessibility. It mandates physics-based, 120fps GPU-accelerated motion (utilizing libraries such as `motion/react` for fluid, spring-based interactions), while explicitly prohibiting ubiquitous anti-patterns like absolute blacks (`#000000`), nested card hierarchies, and unstructured glassmorphism. To accelerate this, Phase 5 by default (its mandatory first step) selects a curated design-skill matching the project archetype from the awesome-design-skills registry — enumerated NON-interactively by fetching the registry index (`curl -fsSL .../skills/index.json`), never the interactive `typeui.sh list` selector (which hangs or silently no-ops headless). The chosen skill's `SKILL.md` is pulled directly via `curl` (or the fully-flagged non-interactive `npx -y typeui.sh pull <slug> -f skill -p <provider> --dry-run < /dev/null`), and its structure/typography/spacing reconciled into `design-system/MASTER.md`. Because registry skills routinely ship banned hex (`#000`/`#fff`) and zero OKLCH, the agent expects to color-CONVERT (hex→OKLCH, clamp pure black/white), not adopt — Studio Prime's banned-pattern list always wins. Only if the registry genuinely can't be reached does the agent fall back to the built-in standard with a logged `[CONDITIONAL_GATE]` (never blocking).
*   **PTY-Backed Execution & Process Monitoring:** OpenClaw is the only platform in the Studio Prime family that runs *every* shell command through a true pseudo-terminal. The agent invokes `exec({ command, pty: true, ... })` for synchronous work, and `exec({ command, pty: true, background: true })` for long-running jobs (dev servers, watchers, build daemons). Those background sessions are then tracked through `process({ action: "list" })` and tailed via `process({ action: "log", sessionId, offset, limit })`. This is what enables OpenClaw to handle daemons, dev servers, and long builds without blocking the agent or losing output across channel boundaries — a class of work that breaks non-PTY platforms entirely. This same mechanism delivers the live handoff: on a no-credentials run, the FINAL Phase 6 server is started with `exec({ command, pty: true, background: true })`, its `sessionId` + URL recorded in `.studio/state/local_live.md`, and left RUNNING at sign-off — it is exempt from the graceful-kill that disposable gate instances receive.
*   **Autonomous Escalation & State-Preserving Rollback:** Brittle debugging loops are a primary failure vector for autonomous agents. To resolve this, OpenClaw Prime implements a bounded, 5-cycle recovery protocol. When confronted with an error, the agent is granted three direct remediation attempts. Upon failure, it is mathematically forced to pivot: executing targeted research via the `web_search` tool to inject external context for the specific error, followed by one final retry. This cycle can repeat up to five times (yielding 20 total iterations) before the agent triggers a hard ceiling. After exhausting the repair budget (5 cycles per failure, 20 iterations max), the agent executes a safe `exec({ command: "git stash push -m studio-prime-recovery", pty: true })`, logs forensic data to `.studio/blocked.md`, then auto-isolates the failing module — logging it as a `[PRIORITY:H]` TECH_DEBT entry — and continues the pipeline with the remaining scope. Human-as-a-Service (HaaS) is invoked only when the isolated module is a critical-path dependency; in unattended (Sleep Test) mode the agent checkpoint-exits non-zero instead of blocking on a human. A deploy step with credentials provided at intake is NEVER a "blocked critical-path module" — it auto-deploys live (the credentials are the authorization), so it never rides the checkpoint-exit path; and on a no-credentials run the localhost server is left running for handoff rather than the session ending dark. Checkpoint-exit is reserved for genuinely unrecoverable build failures.
*   **Parallel Build Swarm (P3/P4):** Phase boundaries and review gates stay strictly sequential, but the heavy build work INSIDE Phase 3 (scaffolding) and Phase 4 (implementation) is parallelized across an ownership-partitioned swarm of `sessions_spawn` workers — each owns a disjoint set of files, runs its own module-scoped proof-of-work, and returns a manifest. The main agent is the sole merge authority and RE-RUNS the full phase proof-of-work against the integrated whole before the Apex Red Team reviews it, so speed never buys trust in unverified output. With no dispatch tool the swarm gracefully degrades to sequential execution — same gates, same correctness, just no speedup.
*   **Proactive Context Engineering:** Studio Prime treats the context window as scratch space and the `.studio/` tree as durable memory (the MemGPT model), actively managing context pressure across GREEN/AMBER/RED budget tiers rather than waiting to "lose the middle." As pressure rises it runs a compaction ladder — flushing completed todos to `.studio/archive.md`, summarizing old decisions, and offloading context-heavy subtasks to `sessions_spawn` workers ("read big, return small"). It writes `context_checkpoint.md` (recording live `sessionId`s, since OpenClaw is PTY-tethered) on a HEARTBEAT — after every phase, before every dispatch, and after any large read, not only at RED — so a crash at any tier has a recent anchor for a **minimal-loss, re-derivable** resume (the checkpoint is an anchor, not a full transcript). OpenClaw has no in-session `/compact`, so under genuine pressure the durable path is to checkpoint-exit cleanly and resume via "Continue Studio Prime".
*   **Zero-Gap Phase Chaining:** Upon receiving a passing verdict from the Apex Red Team, the agent autonomously transitions to the next phase without pausing for human approval. The agent operates as a continuous pipeline, only halting for explicit blockers or the final Northstar Validation Gate. At sign-off — one of the few legitimate stop states — it emits a final **Deployment Briefing** to the channel (and persists it to `.studio/state/deployment_briefing.md` + HANDOFF §10): the verified live status (deployed URL or running `http://localhost:<port>` + `sessionId`), the exact redeploy/rollback/log/scale commands for the target, proactive next-step suggestions, and the OpenClaw resume command.
*   **Northstar Validation Gate:** Before final sign-off in Phase 6, the agent compares the original Phase 1 requirements (`northstar.md` — immutable, never re-captured) against the final deliverables and writes every gap to a failure-findings file (`northstar_gap_analysis.md`). Isolated gaps trigger surgical remediation: only the phase(s) that own each gap are re-entered (a missing API re-enters P3/P4, an a11y miss P5, a deploy miss P6). If the failure is systemic — smoke tests fail, most requirements are unmet, or the blueprint/architecture itself is implicated — it escalates: the agent re-enters Phase 1 and re-walks the entire pipeline with the failure findings as mandatory input to every phase. The two paths use SEPARATE budgets — a cheap surgical budget (cap 5) and a small systemic re-walk budget (cap 2), plus a no-progress guard that escalates or breaks early if a cycle fails to reduce the unmet-criteria count. After the caps, non-critical gaps are deferred to `TECH_DEBT`; a CRITICAL-path gap never exits to a dark process while any live-serving build is achievable — the agent feature-flags the unmet feature OFF, redeploys/restarts, confirms HTTP 200, and only then (unattended) checkpoint-exits non-zero, so you wake to a LIVE product missing one flagged feature rather than nothing.

---

## ⚙️ Function: Orchestration & The Gating System

The operational architecture of OpenClaw Prime is defined by a strictly sequential, 6-Phase lifecycle. Progression through these phases is not determined by heuristic guessing, but rather governed by a deterministic, mathematically verifiable gating system.

### The Scratchpad DAG & Transition Checklists
Before the system is permitted to transition from Phase *N* to Phase *N+1*, it must construct a Directed Acyclic Graph (DAG) in its scratchpad memory (`<phase_gate_checklist>`). This enforces a mandatory sequential evaluation of:
1.  **Prerequisite Verification:** Ensuring all artifact dependencies from Phase *N* exist on the filesystem.
2.  **Proof-of-Work Compilation:** Gathering the raw `stdout` from testing and verification scripts.
3.  **Apex Red Team Adjudication:** Submitting the collected evidence to an isolated sub-agent for adversarial review.

### The Apex Red Team (Multi-Agent Debate)
The Apex Red Team operates as a sub-agent dispatched via OpenClaw's `sessions_spawn({ task, label, agentId })` under `agentId: "reviewer"`, with the spawn task body enforcing a strict BLINDED protocol — the reviewer receives ONLY the North Star, the phase artifacts, the checklist, and the raw stdout (no conversation history, developer notes, or prior attempts), which is what makes the adversarial review meaningful. It executes a 3-round Divergent Persona Protocol:
*   **Round 1 (Steelman):** The sub-agent adopts the persona of the original developer, tasked with vigorously defending the implementation and citing specific passing tests.
*   **Round 2 (Adversarial):** The persona shifts to a zero-trust security auditor operating under the absolute assumption that a vulnerability *does* exist. It attacks the assertions made in Round 1.
*   **Round 3 (Synthesis):** A principal engineer persona adjudicates the debate, classifying findings into `[BLOCKER]` (halts execution, mandates a safe `git stash` rollback, and triggers HaaS), `[TECH_DEBT]` (logs to `.studio/todos.md` but permits progression), or `[GREEN_FLAG]` (clean pass). One thing is NEVER routed to HaaS as a `[BLOCKER]`: a product that is "not deployed despite standing credentials" or "no running server at handoff" is auto-remediated by DEPLOYING (re-enter the deploy step), not by asking a human; and a logged `[LOCAL_LIVE]` localhost server satisfies the deploy gate on no-credentials runs — the reviewer accepts it rather than flagging the missing cloud deploy.

### The 6-Phase Lifecycle

1.  **Phase 1: Blueprint & Baseline**
    *   **Function:** Establishes the foundational architectural constraints. Initiates mandatory web searches (min 3, recommended 5-10+) via the `web_search` tool to anchor the model in current, verified documentation rather than parametric memory.
    *   **Output:** Generates `northstar.md` (immutable requirements) and `decisions.md` (the architectural state machine).
2.  **Phase 2: Link**
    *   **Function:** Defines the integration seams. This phase enforces the "Data-First Rule," establishing database migration strategies, external API wiring, and identity management before any business logic is written.
    *   **Output:** Complete `data_contracts.md` and integration blueprints.
3.  **Phase 3: Architecture & Scaffolding**
    *   **Function:** Translates the blueprint into syntactically valid code structures. Prohibits business logic. Focuses on strict types, interface boundaries, TDD stubs, **multi-stage Docker configs (`Dockerfile`, `docker-compose.yml`)**, **automated CI/CD workflows**, and **database migrations** — each VALIDATED, not merely created (docker build + `docker compose config` + CI YAML/required-job check + migration dry-run, with raw stdout captured into the proof-of-work block).
4.  **Phase 4: Implement & Verify**
    *   **Function:** Executes business logic to satisfy TDD stubs. Enforces 80%+ code coverage, **executed E2E integration tests** (implemented and RUN with captured pass/fail, not stubs), validated **structured JSON stdout logs**, a **dependency CVE audit + secrets-leak scan (binary gates, pre-commit hook installed)**, and strict **OWASP security hardening** (schema validation, secure HttpOnly/Secure cookies, safe JWT RS256 signing, and API rate-limiting per IP).
5.  **Phase 5: Stylize (UI/UX Polish)**
    *   **Function:** Transforms functional interfaces into production-grade aesthetics. Enforces the 2026 Impeccable Design Standard, ensuring **WCAG 2.1 AA accessibility verified by an EXECUTED axe-core/pa11y audit** (critical/serious violations are a BLOCKER), verifiable component state matrices (every interactive element proven to carry all 5 states with a mandatory focus ring), fluid motion physics, and LCP (<2.5s) / CLS (<0.1) metrics.
6.  **Phase 6: Release**
    *   **Function:** The final pre-flight operational check. Integrates a **tested graceful OS-signal shutdown** (SIGTERM is actually sent against a DISPOSABLE throwaway instance and a clean exit within the drain window is asserted, not just a registered listener — then the server is (re)launched detached for handoff, never left dead), a **migration dry-run before prod apply**, **synthetic-error → confirmed alert propagation**, a **wall-clock-measured rollback dry-run (<5min)**, and a **credential-keyed deploy + executed smoke tests**: if deploy credentials were provided at intake the agent autonomously deploys LIVE (the credentials are the authorization) and runs the post-deploy smoke suite against the real production URL; if none were provided it starts the verified artifact as a detached `background: true` localhost server (`sessionId` + URL recorded, left running) and runs the smoke suite against `http://localhost:<port>`. All kill-based gates run first against throwaway instances; the FINAL persistent server is left live and re-probed (HTTP `200`) immediately before sign-off. This concludes with a comprehensive 17-section `HANDOFF.md` consolidation (validated to be free of placeholders and with every section filled, recording the live deployed URL or the still-running localhost URL + `sessionId`/PID) and the final **Northstar Validation Gate**.

---

## 📥 Expected Input/Output (I/O)

To effectively orchestrate the development lifecycle, OpenClaw Prime expects structured inputs and generates deterministic outputs across multi-channel environments. Understanding this I/O contract is critical for maximizing the system's efficacy.

### **Input Modalities**
*   **Intake Triggers:** Natural language commands such as `"Start Studio Prime"` or `"Continue Studio Prime"`, deliverable via CLI, Discord/Slack `/subagents spawn`, or HTTP.
*   **Requirement Documents (PRDs):** Structured text outlining the core problem, target audience, and functional requirements.
*   **Design Assets & Constraints:** Optional guidelines, reference URLs, or brand tokens.
*   **Human-as-a-Service (HaaS) Feedback:** Explicit approvals, architectural overrides, or credential provisions when the system halts at an enforced gate. Always formatted as a simple `A/B/C` reply so it works equally well from a terminal or a mobile chat app. Note: hosting/deploy credentials supplied at intake are STANDING AUTHORIZATION, not a reactive HaaS provision — no enforced deploy gate fires for them, and the agent deploys live without waiting for a reply.

### **Output Modalities**
*   **Terminal Execution (`stdout`/`stderr`):** Verifiable proof of execution, testing, and deployment streamed via the PTY-backed `exec` tool.
*   **Persistent State Files (`.studio/`):** Markdown and JSON artifacts that maintain the agent's memory, architectural decisions, and active task lists.
*   **Production Code:** Fully typed, tested, and linted source code adhering to established conventions.
*   **Markdown Letter-List Prompts:** Interactive menus rendered as plain markdown letter-lists (`A. ... B. ...`) — never ASCII boxes — so they render cleanly across CLI, Discord, and Slack.
*   **Adversarial Review Logs:** Detailed markdown reports from the Apex Red Team, classifying the phase's output into `[GREEN_FLAG]`, `[TECH_DEBT]`, or `[BLOCKER]`.

---

## 📁 File Structure & Reasoning

Studio Prime writes its state to the disk to cure "LLM Amnesia" and survive context limits. On OpenClaw, where there is no native TUI task panel, the file system **is** the canonical interface — `.studio/todos.md` in particular is the source of truth for active work, not an ephemeral UI widget.

```text
.studio/
├── todos.md                # Canonical task list (THE source of truth — no native TUI mirror)
├── decisions.md            # The "Brain": prevents the agent from contradicting past choices
├── data_contracts.md       # API schemas enforced before UI implementation
├── archive.md              # Flushed tasks to keep the active context small
├── blocked.md              # Detailed logs of failed escalation attempts
├── state/
│   ├── northstar.md            # The immutable original requirements
│   ├── phase*_research.md      # Raw search findings
│   ├── background_jobs.md      # sessionId log for every backgrounded `exec` (mandatory)
│   ├── swarm_plan.md           # Parallel Build Swarm ownership DAG (P3/P4)
│   ├── context_checkpoint.md   # Heartbeat resume anchor — minimal-loss, re-derivable (records live sessionIds)
│   ├── effects.md              # Side-effect ledger (deploy/migration/publish keys — idempotent re-runs)
│   ├── budget.md               # Optional wall-clock / tool-call budget guard
│   ├── deploy_target.md        # Resolved deploy target + standing-auth marker
│   ├── deploy_ready.sh         # Exact go-live commands (no-creds path)
│   ├── local_live.md           # {url, sessionId, start/stop cmd} for the left-running localhost server
│   └── deployment_briefing.md  # Final platform-aware deployment briefing (also in-channel + HANDOFF §10)
└── checklists/                 # Mandatory DAG gate checkpoints (e.g., arch_red_team.md for Phase 2)

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

> **Note on `.studio/todos.md`:** Because OpenClaw does not have a native TUI task panel (unlike OpenCode's `todowrite`), the markdown checklist in this file *is* the task list. Treat it as authoritative — every transition through the 6-phase lifecycle reads and writes it, and humans interacting via chat channels rely on the agent quoting it back to inspect status.

---

## 🌐 Multi-Channel Operation

OpenClaw is a gateway, not a single-surface CLI. The same agent instance accepts work from several invocation surfaces, and Studio Prime adapts its output formatting to whichever channel it detects:

*   **CLI (`openclaw agent --message "..."`):** Full ANSI rendering. Tables and code blocks render normally; markdown is interpreted by the terminal frontend.
*   **Discord (`/subagents spawn --task "..."`):** Markdown letter-lists only — no ASCII boxes, no wide tables. Long code blocks are kept under Discord's 2000-character message limit by splitting into multiple messages when needed.
*   **Slack (`/subagents spawn --task "..."`):** Same rules as Discord. ASCII art is explicitly banned because Slack's proportional fonts mangle column alignment.
*   **HTTP API (e.g. `POST /v1/sessions/spawn` — your OpenClaw gateway's spawn route):** Raw markdown payload returned in the response body. The agent assumes the caller will render it.

The agent declares which channel it's responding on in its first message using the canonical bracketed header (e.g., `[OpenClaw | channel=Discord | agent=studio-prime | session=<short-id>]`) so the user can correlate the response with the right workspace when multiple OpenClaw instances are running concurrently.

---

## 🔬 Credits & Scientific Foundation

Studio Prime's mechanics are not arbitrary; they are implementations of peer-reviewed AI and LLM research:

*   **Apex Red Team (Multi-Agent Debate):** Based on research like *ChatEval: Towards Better LLM-based Evaluators through Multi-Agent Debate (Chen et al., 2023)*. Studies prove that assigning distinct, isolated adversarial roles reduces LLM hallucinations and false positives far better than single-agent self-critique. OpenClaw's `sessions_spawn` tool makes this possible: each reviewer is dispatched under `agentId: "reviewer"` with a spawn task body that enforces a strict BLINDED protocol (the North Star, phase artifacts, checklist, and raw stdout only — no conversation history, developer notes, or prior attempts).
*   **Proof-of-Work (Verbal Reinforcement):** Rooted in *Reflexion: Language Agents with Verbal Reinforcement Learning (Shinn et al., 2023)*. Converting raw environmental feedback (terminal stdout) into a linguistic self-reflection allows the agent to break out of repetitive failure loops.
*   **Context Compaction (`.studio/`):** Based on architectures like *MemGPT: Towards LLMs as Operating Systems (Packer et al., 2023)*. Writing state to a persistent filesystem and retrieving it Just-In-Time (`grep`/`read`) prevents the "Lost in the Middle" context degradation phenomenon.

---
*Architected by Suraj Kuncham. Designed to push the boundaries of autonomous software engineering.*
