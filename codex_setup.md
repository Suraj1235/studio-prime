# Codex Prime

[← Back to README](README.md) · Setup guide for [`codex_prime.md`](codex_prime.md)

**The Studio Prime superprompt — optimized for OpenAI Codex CLI's native toolset (`shell`, `apply_patch`, `spawn_agents_on_csv`, `request_user_input`, `update_plan`) and its hierarchical `AGENTS.md` instruction chain.**

---

## What is Codex Prime?

Codex Prime is more than a prompt — it is a complete production engineering playbook. It transforms your standard AI coding assistant into a relentless, autonomous Principal Architect, UI/UX Designer, and Zero-Trust Red Teamer.

While the Studio Prime architecture runs on multiple platforms, **Codex CLI is the most battle-hardened environment for unattended autonomous runs.** Codex is written in Rust with zero Node dependencies, ships a `codex exec` headless mode purpose-built for CI and cron, auto-compacts silently at `context - 13k tokens`, and loads its system prompt from a hierarchical `AGENTS.md` chain (global `~/.codex/AGENTS.md` merged down through every parent directory to the project root). Its native `spawn_agents_on_csv` tool dispatches a CSV-defined batch of independent sub-agents in a single call, with each worker reporting back via `report_agent_job_result` — true parallel research fan-out, plus a fresh, physically-isolated blinded context for each adversarial review round.

With Codex Prime, you achieve the ultimate "Sleep Test": Start a session, go to sleep, and wake up to a LIVE product — a working DEPLOYED URL when you provided hosting/deploy credentials (the provision of the credentials IS the deploy authorization), or a fully functional application with its localhost server STILL RUNNING (URL + PID documented, started detached so it survives the session) when you did not. **Codex CLI passes this test natively because `codex exec` is engineered from day one for unattended runs (CI, cron, pre-merge hooks) — no human in the loop required, and the handoff/deploy step is never a human checkpoint.**

---

## 🚀 Setup Process

Follow these steps exactly to install Codex Prime and unleash its full capabilities.

> **⚠️ CRITICAL — set the doc cap BEFORE copying (Options A & B):** `codex_prime.md` is ~111 KB, but Codex's default `project_doc_max_bytes = 32 KiB` will silently truncate ~70% of it (dropping the Glossary and tool-mapping notes from the bottom). BEFORE running the `cp` in Option A or B, raise the cap in `~/.codex/config.toml`: `project_doc_max_bytes = 131072`. This applies to BOTH the global (`~/.codex/AGENTS.md`) and project-level (`AGENTS.md`) scopes — global scope shares the same 32 KiB cap and does NOT escape truncation. Option C (Skill install) is NOT subject to the `AGENTS.md` doc cap and needs no config change.

### Option A: As Project-Level `AGENTS.md` (Recommended for Project-Scoped Use)
This method scopes Studio Prime to a single repository. Codex walks up from the working directory and merges every `AGENTS.md` it finds, so a project-level file always wins on conflicts.

1. **Navigate to your project workspace.**
2. **Copy the superprompt** into the project root as `AGENTS.md`:
   ```bash
   cp codex_prime.md AGENTS.md
   codex
   ```

### Option B: As Global `~/.codex/AGENTS.md` (Loads in Every Workspace)
This method makes Studio Prime the default operating mode for every Codex session on the machine.

```bash
mkdir -p ~/.codex
cp codex_prime.md ~/.codex/AGENTS.md
codex
```

### Option C: As an On-Demand Skill
This method allows you to load Studio Prime modularly only when you explicitly want to use it.

```bash
mkdir -p ~/.codex/skills/studio-prime
cp codex_prime.md ~/.codex/skills/studio-prime/SKILL.md
codex
# Invoke with: $studio-prime
```

### Custom Reviewer Agent
Studio Prime installs a `reviewer` custom agent on first run for the Apex Red Team. The agent is auto-written to `~/.codex/agents/reviewer.toml` during Self-Setup (Phase 1 verifies it exists and re-creates it if missing — Self-Setup is the canonical writer). Codex is the only platform where reviewer personas are declared as a first-class TOML schema rather than embedded prose, which means the reviewer's permissions, sandbox profile, and tool surface are enforced by the host runtime, not the LLM's good intentions.

### Sandbox / Approval
For interactive development, launch with `codex --full-auto` (equivalent to `--sandbox workspace-write` + `--ask-for-approval on-request`). For unattended sleep-test runs, use `codex --sandbox workspace-write -a never` so the agent never blocks on approval prompts.

> **Deploy egress note:** `-a never` removes the approval-prompt surface, but the `workspace-write` sandbox confines network egress. A LIVE cloud deploy (network calls to Vercel/Fly/registry) may be blocked by the sandbox even with `-a never` — to deploy live unattended, launch with network access (or `--sandbox danger-full-access`) so the credential-backed deploy can reach the platform. If egress is blocked, Studio Prime logs `[CONDITIONAL_GATE: sandbox blocks deploy egress — LOCAL_LIVE fallback]` and instead starts the production server locally as a DETACHED process (`nohup`/`setsid`) so it OUTLIVES the `codex exec` process and is still running at handoff with its URL + PID recorded. Providing deploy credentials at intake IS the deploy authorization — no human gate fires for the deploy.

### Triggering the Agent

Once Codex is running, trigger the system by typing:

```
"Start Studio Prime"
```

**The Intake Gate:** Upon invocation, Codex Prime will NOT dump a massive wall of text. Instead, it leverages Codex's native `request_user_input` tool to present a structured choice asking if you want to start a **NEW PROJECT** (guided discovery) or work on an **EXISTING CODEBASE** (add features or transform).

---

## ✨ Core Features

Codex Prime leverages Codex CLI's native APIs to enforce rigorous software engineering standards, transforming the LLM from a stochastic text generator into a deterministic engineering environment:

*   **Proof-of-Work Anti-Hallucination:** LLMs inherently struggle to differentiate between intended outcomes and actual execution results. To counteract this, Codex Prime enforces a strict, three-part empirical verification protocol. Before asserting task completion, the agent must generate a `<prediction>` of the expected terminal output, execute the command via the `shell` tool, and synthesize a `<divergence_analysis>`. By comparing the expected state against the actual `stdout`, the agent is forced to linguistically process any discrepancies. Furthermore, if a successful output is indistinguishable from a generic response, the system injects a deliberately malformed flag into a subsequent execution to empirically verify the terminal's responsiveness, thereby eliminating blind assumptions.
*   **The 2026 Impeccable Design Standard:** Studio Prime categorically rejects the homogenized "AI Starter Pack" aesthetic. It enforces a strict mathematical approach to UI engineering, dictating the use of the `OKLCH` color space for perceptual uniformity and accessibility. It mandates physics-based, 120fps GPU-accelerated motion (utilizing libraries such as `motion/react` for fluid, spring-based interactions), while explicitly prohibiting ubiquitous anti-patterns like absolute blacks (`#000000`), nested card hierarchies, and unstructured glassmorphism.
*   **CSV-Batch Parallel Sub-Agents (Codex-Specific):** Codex CLI uniquely exposes `spawn_agents_on_csv`, a native tool that accepts a CSV of agent definitions and dispatches every row in parallel as physically isolated workers. Codex Prime exploits this for genuinely INDEPENDENT work — chiefly research fan-out, where several distinct questions are answered concurrently in their own blinded contexts, each reporting back via `report_agent_job_result` with no context contamination. The 3-round Apex Red Team protocol, by contrast, is SEQUENTIAL by design — Round 2 attacks Round 1's assertions and Round 3 adjudicates both, so each round consumes the prior round's output and cannot run as parallel workers. Studio Prime dispatches each blinded review round as its own sub-agent (a fresh `spawn_agents_on_csv` call per round) so the reviewer always runs in a clean, history-free context. Combined with the typed `reviewer` agent declared in `~/.codex/agents/reviewer.toml`, this gives Studio Prime a deterministic adversarial loop with runtime-enforced reviewer permissions that other platforms can only emulate via prose.
*   **Autonomous Escalation & State-Preserving Rollback:** Brittle debugging loops are a primary failure vector for autonomous agents. To resolve this, Codex Prime implements a bounded, 5-cycle recovery protocol. When confronted with an error, the agent is granted three direct remediation attempts. Upon failure, it is mathematically forced to pivot: executing targeted research via the `web_search` tool to inject external context for the specific error, followed by one final retry. This cycle can repeat up to five times (yielding 20 total iterations) before the agent triggers a hard ceiling. After exhausting the repair budget (5 cycles per failure, 20 iterations max), the agent executes a safe `git stash`, logs forensic data to `.studio/blocked.md`, then auto-isolates the failing module — logging it as a `[PRIORITY:H]` TECH_DEBT entry — and continues the pipeline with the remaining scope. Human-as-a-Service (HaaS) is invoked only when the isolated module is a critical-path dependency; in unattended (Sleep Test) mode the agent checkpoint-exits non-zero instead of blocking on a human. **The deploy/handoff step is excluded from this checkpoint-exit path:** when deploy credentials were provided at intake, deploying LIVE is standing-authorized and runs autonomously (never a gate, never a non-zero exit); when no credentials were provided, the agent keeps a detached localhost server RUNNING at handoff rather than exiting dark.
*   **Zero-Gap Phase Chaining:** Upon receiving a passing verdict from the Apex Red Team, the agent autonomously transitions to the next phase without pausing for human approval. The agent operates as a continuous pipeline, only halting for explicit blockers or the final Northstar Validation Gate — and even that final gate never halts for deploy authorization: when intake deploy credentials exist, deploying live is standing-authorized and proceeds without a checkpoint; when none exist, the gate requires a still-running localhost server before it passes. Asking "should I deploy?" / "ready to go live?" with standing credentials is itself a Zero-Gap violation.
*   **Northstar Validation Gate:** Before final sign-off in Phase 6, the agent compares the original Phase 1 requirements (`northstar.md` — immutable, never re-captured) against the final deliverables and writes every gap to a failure-findings file (`northstar_gap_analysis.md`). Isolated gaps trigger surgical remediation: only the phase(s) that own each gap are re-entered (a missing API re-enters P3/P4, an a11y miss P5, a deploy miss P6). If the failure is systemic — smoke tests fail, most requirements are unmet, or the blueprint/architecture itself is implicated — it escalates: the agent re-enters Phase 1 and re-walks the entire pipeline with the failure findings as mandatory input to every phase. A "deploy miss" with standing deploy credentials is auto-remediated by re-entering P6 and DEPLOYING (never a human escalation); without credentials it is satisfied by ensuring the localhost server is running. Both paths share a 2-cycle cap, after which non-critical gaps are deferred to `TECH_DEBT` and only critical-path gaps escalate to a human — and only after the bounded deploy/relaunch retries truly exhaust.

---

## ⚙️ Function: Orchestration & The Gating System

The operational architecture of Codex Prime is defined by a strictly sequential, 6-Phase lifecycle. Progression through these phases is not determined by heuristic guessing, but rather governed by a deterministic, mathematically verifiable gating system.

### The Scratchpad DAG & Transition Checklists
Before the system is permitted to transition from Phase *N* to Phase *N+1*, it must construct a Directed Acyclic Graph (DAG) in its scratchpad memory (`<phase_gate_checklist>`), mirrored to the operator-visible plan via Codex's `update_plan` tool. This enforces a mandatory sequential evaluation of:
1.  **Prerequisite Verification:** Ensuring all artifact dependencies from Phase *N* exist on the filesystem.
2.  **Proof-of-Work Compilation:** Gathering the raw `stdout` from testing and verification scripts.
3.  **Apex Red Team Adjudication:** Submitting the collected evidence to isolated sub-agents for adversarial review.

### The Apex Red Team (Multi-Agent Debate)
The Apex Red Team is dispatched via Codex's `spawn_agents_on_csv` tool — each round loading the `reviewer` persona defined in `~/.codex/agents/reviewer.toml` in its own sub-agent. The three rounds run SEQUENTIALLY (Round 2 consumes Round 1's output, Round 3 adjudicates both), each dispatched as a separate `spawn_agents_on_csv` call so every reviewer runs in a fresh, blinded context window with no conversational history from the main thread, reporting back via `report_agent_job_result`. It executes a 3-round Divergent Persona Protocol:
*   **Round 1 (Steelman):** The sub-agent adopts the persona of the original developer, tasked with vigorously defending the implementation and citing specific passing tests.
*   **Round 2 (Adversarial):** The persona shifts to a zero-trust security auditor operating under the absolute assumption that a vulnerability *does* exist. It attacks the assertions made in Round 1.
*   **Round 3 (Synthesis):** A principal engineer persona adjudicates the debate, classifying findings into `[BLOCKER]` (halts execution, mandates a safe `git stash` rollback, and triggers HaaS) or `[TECH_DEBT]` (logs to `.studio/todos.md` but permits progression); absence of either yields the overall verdict `[GREEN_FLAG]` (clean pass). A Phase 6 "undeployed despite standing credentials" or "no live server at handoff" is auto-remediated by deploying / relaunching (re-enter the deploy step), never raised as a `[BLOCKER]` that ends in a human checkpoint; and a logged `[LOCAL_LIVE]` localhost server is accepted as satisfying the deploy gate on no-credential runs.

### The 6-Phase Lifecycle

1.  **Phase 1: Blueprint & Baseline**
    *   **Function:** Establishes the foundational architectural constraints. Initiates mandatory `web_search` invocations (min 3, recommended 5-10+) to anchor the model in current, verified documentation rather than parametric memory. Verifies the `reviewer` custom agent at `~/.codex/agents/reviewer.toml` exists (written during Self-Setup; re-created only if missing) and optionally seeds `~/.codex/hooks.json` (an `apply_patch` envelope-validation hook + a `shell` invocation-logging hook) on user opt-in.
    *   **Output:** Generates `northstar.md` (immutable requirements), `decisions.md` (the architectural state machine), and the host-level Codex configuration artifacts.
2.  **Phase 2: Link**
    *   **Function:** Defines the integration seams. This phase enforces the "Data-First Rule," establishing database migration strategies, external API wiring, and identity management before any business logic is written.
    *   **Output:** Complete `data_contracts.md` and integration blueprints.
3.  **Phase 3: Architecture & Scaffolding**
    *   **Function:** Translates the blueprint into syntactically valid code structures. Prohibits business logic. Focuses on strict types, interface boundaries, TDD stubs, **multi-stage Docker configs (`Dockerfile`, `docker-compose.yml`)**, **automated CI/CD workflows**, and **database migrations**.
4.  **Phase 4: Implement & Verify**
    *   **Function:** Executes business logic to satisfy TDD stubs. Enforces 80%+ code coverage, **fully implemented AND executed E2E/integration tests covering 100% of critical user journeys (stub-only suites are a BLOCKER)**, structured **JSON stdout logs**, and strict **OWASP security hardening** (schema validation, secure HttpOnly/Secure cookies, safe JWT RS256 signing, and API rate-limiting per IP). All file mutations flow through `apply_patch` using V4A diff format.
5.  **Phase 5: Stylize (UI/UX Polish)**
    *   **Function:** Transforms functional interfaces into production-grade aesthetics. Enforces the 2026 Impeccable Design Standard, ensuring **WCAG 2.1 AA accessibility verified via an executed runtime axe-core/pa11y audit against the running app (critical/serious violations are a BLOCKER, and a stub-only or audit-only submission is itself a BLOCKER)**, comprehensive component state matrices, fluid motion physics, and LCP (<2.5s) / CLS (<0.1) metrics.
6.  **Phase 6: Release**
    *   **Function:** The final pre-flight operational check. Tests **graceful OS signal shutdowns (`SIGTERM`/`SIGINT`)** against a DISPOSABLE instance (the drain handler is verified, then the product is (re)started — the kill never leaves the app dark at handoff), runs production **DB migrations**, verifies the **synthetic alert pipeline**, and — when deploy credentials were provided at intake (the credentials ARE the authorization) — **autonomously deploys LIVE and runs post-deploy smoke tests against the production URL**; when no credentials were provided, it starts the production server locally as a DETACHED process that survives the session and runs the smoke suite against `http://localhost:<port>`. A final liveness re-probe asserts the URL returns HTTP 200 immediately before sign-off. Concludes with a comprehensive 17-section `HANDOFF.md` consolidation (recording the live URL or the running localhost URL + PID) and the final **Northstar Validation Gate**.

---

## 📥 Expected Input/Output (I/O)

To effectively orchestrate the development lifecycle, Codex Prime expects structured inputs and generates deterministic outputs. Understanding this I/O contract is critical for maximizing the system's efficacy.

### **Input Modalities**
*   **Intake Triggers:** Natural language commands such as `"Start Studio Prime"` or `"Continue Studio Prime"`.
*   **Requirement Documents (PRDs):** Structured text outlining the core problem, target audience, and functional requirements.
*   **Design Assets & Constraints:** Optional guidelines, reference URLs, or brand tokens. Visual references can be attached directly via `codex -i screenshot.png "prompt"`.
*   **Human-as-a-Service (HaaS) Feedback:** Explicit approvals, architectural overrides, or credential provisions when the system halts at an enforced gate, delivered through Codex's `request_user_input` tool. Note: deploy/hosting credentials provided AT INTAKE are NOT a reactive provision after a halt — they are recorded as STANDING deploy authorization (`[DEPLOY_AUTH: standing]`), so no gate ever fires to request them when they are already present and the live deploy runs autonomously.

### **Output Modalities**
*   **Terminal Execution (`stdout`/`stderr`):** Verifiable proof of execution, testing, and deployment, captured via the `shell` tool.
*   **File Mutations:** All edits and new files are emitted as V4A diffs through the `apply_patch` tool — never raw overwrites — so every change is reviewable, atomic, and reversible.
*   **Persistent State Files (`.studio/`):** Markdown and JSON artifacts that maintain the agent's memory, architectural decisions, and active task lists.
*   **Production Code:** Fully typed, tested, and linted source code adhering to established conventions.
*   **Structured UI Prompts:** Interactive questions and options rendered in the Codex TUI via the `request_user_input` tool, with the operator-visible task list synchronized through `update_plan`.
*   **Adversarial Review Logs:** Detailed markdown reports from the Apex Red Team, classifying the phase's output into `[GREEN_FLAG]`, `[TECH_DEBT]`, or `[BLOCKER]`.

---

## 📁 File Structure & Reasoning

Studio Prime writes its state to the disk to cure "LLM Amnesia" and survive context limits.

```text
.studio/
├── todos.md                # Active task list (mirrored via update_plan)
├── decisions.md            # The "Brain": prevents the agent from contradicting past choices
├── data_contracts.md       # API schemas enforced before UI implementation
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

**Codex-Specific Artifacts:** In addition to the cross-platform `.studio/` and `architecture/` trees, Codex Prime relies on host-level configuration files that are unique to this platform:
*   **`~/.codex/AGENTS.md`** — the global system prompt loaded into every Codex session; merged hierarchically with any project-level `AGENTS.md` walking down to the working directory.
*   **`~/.codex/agents/reviewer.toml`** — the typed custom agent definition for the Apex Red Team. Codex enforces this agent's tool surface and sandbox profile at the runtime level.
*   **`~/.codex/hooks.json`** — optional hooks (opt-in only). Studio Prime's defaults are a `PreToolUse` hook on `apply_patch` running `validate-patch-envelope.sh` and a `PostToolUse` hook on `shell` running `log-shell-invocation.sh`. Lint/format/typecheck verification hooks are NOT defaults — they are optional user-added hooks you can wire in yourself.
*   **`~/.codex/config.toml`** — main Codex configuration (model, sandbox defaults, approval mode).
*   **`~/.codex/skills/studio-prime/SKILL.md`** — on-demand skill installation path (Option C).

---

## 🛡️ Sandbox & Approval Modes

Codex separates **sandbox** (what the agent can touch) from **approval** (when it must ask permission). Three of each, paired deliberately:

*   **Sandbox modes:** `read-only` (inspection only), `workspace-write` (writes confined to the working directory and `.tmp/`), and `danger-full-access` (no fence — only use in disposable VMs).
*   **Approval modes:** `untrusted` (every tool call requires approval), `on-request` (the agent decides when to ask), and `never` (fully autonomous; never blocks for input).

Recommended pairings: **interactive dev** → `workspace-write` + `on-request` (the `--full-auto` preset); **headless CI / pre-merge hooks** → `workspace-write` + `never` (the agent runs to completion or fails hard); **sleep-test unattended runs** → `workspace-write` + `never` via `codex exec` with `--json --output-last-message` so a wrapper script can capture the verdict.

---

## 🚀 Headless / Sleep-Test Mode

Codex CLI's killer differentiator is `codex exec` — a non-interactive entrypoint designed for unattended execution. There is no TUI, no readline prompt, no spinner; just stdin/stdout and a structured JSON event stream. This is what makes Studio Prime's Sleep Test real on Codex rather than aspirational.

```bash
CODEX_API_KEY=sk-... codex exec --sandbox workspace-write -a never \
  --json --output-last-message out.txt "Start Studio Prime"
# Note: CODEX_API_KEY is the primary env var; some Codex CLI versions also honor
# OPENAI_API_KEY as a fallback — check `codex login --help` for current auth model.
```

The agent runs the full 6-phase lifecycle, dispatches each sequential Red Team round as its own blinded sub-agent via `spawn_agents_on_csv` (and fans research questions out in true parallel CSV batches), auto-compacts silently when the context approaches the `context - 13k` token threshold, and writes the final verdict to `out.txt`. Wire this into a cron job, a GitHub Action, or a pre-merge hook and you have a genuinely autonomous engineering loop.

---

## 🔬 Credits & Scientific Foundation

Studio Prime's mechanics are not arbitrary; they are implementations of peer-reviewed AI and LLM research:

*   **Apex Red Team (Multi-Agent Debate):** Based on research like *ChatEval: Towards Better LLM-based Evaluators through Multi-Agent Debate (Chen et al., 2023)*. Studies prove that assigning distinct, isolated adversarial roles reduces LLM hallucinations and false positives far better than single-agent self-critique. Codex CLI's `spawn_agents_on_csv` tool, combined with the typed `reviewer` agent at `~/.codex/agents/reviewer.toml`, makes this physical isolation possible — a fresh, blinded sub-agent per sequential review round (with parallel CSV fan-out reserved for independent research).
*   **Proof-of-Work (Verbal Reinforcement):** Rooted in *Reflexion: Language Agents with Verbal Reinforcement Learning (Shinn et al., 2023)*. Converting raw environmental feedback (terminal stdout) into a linguistic self-reflection allows the agent to break out of repetitive failure loops.
*   **Context Compaction (`.studio/`):** Based on architectures like *MemGPT: Towards LLMs as Operating Systems (Packer et al., 2023)*. Writing state to a persistent filesystem and retrieving it Just-In-Time (`grep`/`read`) prevents the "Lost in the Middle" context degradation phenomenon.

---
*Architected by Suraj Kuncham. Designed to push the boundaries of autonomous software engineering.*
