# OpenCode Prime

[← Back to README](README.md) · Setup guide for [`opencode_prime.md`](opencode_prime.md)

**The Studio Prime superprompt — perfectly optimized for OpenCode's native toolset.**

---

## What is OpenCode Prime?

OpenCode Prime is more than a prompt — it is a complete production engineering playbook. It transforms your standard AI coding assistant into a relentless, autonomous Principal Architect, UI/UX Designer, and Zero-Trust Red Teamer. 

While the Studio Prime architecture runs on multiple platforms, **OpenCode is the definitive environment for it.** This is because OpenCode natively supports true sub-agent dispatch (via the `Task` tool). When Studio Prime launches its Apex Red Team to audit code, OpenCode physically spins up a fresh, blinded context window. The Red Team doesn't know the developer's original assumptions, struggles, or compromises. This guarantees **true adversarial review**, free from the context contamination that plagues other platforms.

With OpenCode Prime, you achieve the ultimate "Sleep Test": Start a session, go to sleep, and wake up to a LIVE, rigorously audited application — a working deployed URL when you provided hosting credentials, or a fully functional product with its localhost server still running (URL + PID documented) when you did not.

> **Scope:** Studio Prime targets **web products** by construction — its terminal liveness condition is an HTTP-200 URL. For a non-web target (mobile, desktop, CLI/library, data pipeline) it detects the target-class in Phase 1 and either substitutes an appropriate terminal condition (build artifact + simulator/binary smoke) or checkpoint-exits with `[UNSUPPORTED_TARGET: non-web]` rather than forcing a web shape. The whole pipeline is re-run/resume-safe: a side-effect ledger (`.studio/state/effects.md`) guards deploys, migrations, and publishes against double-application on a "Continue Studio Prime" resume or a Northstar re-walk.

---

## 🚀 Setup Process

Follow these steps exactly to install OpenCode Prime and unleash its full capabilities.

### Option A: As a System Prompt (Recommended)
This method ensures Studio Prime is active by default every time you open a new OpenCode workspace.

1. **Open your terminal** and navigate to your home directory or project workspace.
2. **Create the OpenCode config folder** if it doesn't exist:
   ```bash
   mkdir -p .opencode
   ```
3. **Copy the superprompt** into the system prompt file:
   ```bash
   cp opencode_prime.md .opencode/system.md
   ```
4. **Launch OpenCode**:
   ```bash
   opencode
   ```

### Option B: As an On-Demand Skill
This method allows you to load Studio Prime modularly only when you explicitly want to use it.

1. **Create the skills directory**:
   ```bash
   mkdir -p .opencode/skills/studio-prime
   ```
2. **Copy the superprompt** into the skill file:
   ```bash
   cp opencode_prime.md .opencode/skills/studio-prime/SKILL.md
   ```
3. **Launch OpenCode**:
   ```bash
   opencode
   ```

### Triggering the Agent

Once OpenCode is running, trigger the system by typing:

```
"Start Studio Prime"
```

**The Intake Gate:** Upon invocation, OpenCode Prime will NOT dump a massive wall of text. Instead, it leverages OpenCode's native `question` tool to present a clean TUI menu asking if you want to start a **NEW PROJECT** (guided discovery) or work on an **EXISTING CODEBASE** (add features or transform).

---

## ✨ Core Features

OpenCode Prime leverages OpenCode's native APIs to enforce rigorous software engineering standards, transforming the LLM from a stochastic text generator into a deterministic engineering environment:

*   **Proof-of-Work Anti-Hallucination (disk-anchored; writer ≠ grader):** LLMs inherently struggle to differentiate between intended outcomes and actual execution results. To counteract this, OpenCode Prime enforces a strict empirical verification protocol that cannot be satisfied from memory. Before asserting task completion, the agent generates a `<prediction>`, then EXECUTES the command tee-captured to disk (`.studio/state/pow/*.log`) with an unguessable wall-clock-nanosecond + random NONCE line appended by the same invocation. The `<stdout>` field is populated by RE-READING that log file — never typed — and the divergence analysis compares prediction against the re-read log. A `<stdout>` with no corresponding `.log` on disk, or a missing/implausible NONCE, is auto-classified `[UNVERIFIED]` and BLOCKS. The Apex reviewer (a separate, blinded context) takes the on-disk log as its ONLY evidence, so the context that wrote the code is never the one that grades its proof.
*   **The 2026 Impeccable Design Standard:** Studio Prime categorically rejects the homogenized "AI Starter Pack" aesthetic. It enforces a strict mathematical approach to UI engineering, dictating the use of the `OKLCH` color space for perceptual uniformity and accessibility. It mandates physics-based, 120fps GPU-accelerated motion (utilizing libraries such as `motion/react` for fluid, spring-based interactions), while explicitly prohibiting ubiquitous anti-patterns like absolute blacks (`#000000`), nested card hierarchies, and unstructured glassmorphism.
*   **TUI Synchronization via Context-Aware Tooling:** OpenCode Prime deeply integrates with the host platform's Terminal User Interface (TUI). By interfacing directly with the native `todowrite` tool, the agent dynamically maps its internal Directed Acyclic Graph (DAG) state to the external UI task panel. This bidirectional synchronization ensures the human operator maintains perfect, real-time observability of the agent's phase progression without needing to parse the raw context stream.
*   **Autonomous Escalation & State-Preserving Rollback:** Brittle debugging loops are a primary failure vector for autonomous agents. To resolve this, OpenCode Prime implements a bounded, 5-cycle recovery protocol. When confronted with an error, the agent is granted three direct remediation attempts. Upon failure, it is mathematically forced to pivot: executing targeted web research via the `webfetch` tool to inject external context for the specific error, followed by one final retry. This cycle can repeat up to five times (yielding 20 total iterations) before the agent triggers a hard ceiling. After exhausting the repair budget (5 cycles per failure, 20 iterations max), the agent executes a safe `git stash`, logs forensic data to `.studio/blocked.md`, then auto-isolates the failing module — logging it as a `[PRIORITY:H]` TECH_DEBT entry — and continues the pipeline with the remaining scope. Human-as-a-Service (HaaS) is invoked only when the isolated module is a critical-path dependency; in unattended (Sleep Test) mode the agent FIRST maximizes the largest live-serving subset (feature-flag the unmet feature off, confirm HTTP 200) and only then checkpoint-exits non-zero — a DEGRADED outcome, not full success — instead of blocking on a human. There is no token/$ cap the engine can observe, but optional `STUDIO_MAX_WALLCLOCK_HRS` / `STUDIO_MAX_TOOL_CALLS` guards (tracked in `.studio/state/budget.md`) checkpoint-exit on breach; set a provider-side spend cap before you walk away (see the README cost note). A credentialed deploy is NOT a checkpoint-exit gate — deploy/hosting credentials provided at intake are standing authorization, so the agent deploys live and hands over a working URL; on no-credential runs it instead leaves a detached localhost server running (URL + PID documented) and exits 0, never dark.
*   **Zero-Gap Phase Chaining:** Upon receiving a passing verdict from the Apex Red Team, the agent autonomously transitions to the next phase without pausing for human approval. The agent operates as a continuous pipeline, only halting for explicit blockers. The final Northstar Validation Gate never halts to ask whether to deploy — when deploy credentials were provided the deploy is pre-authorized and executes autonomously; on no-credential runs the gate requires a live localhost server (URL + PID) before it passes. Asking "Should I deploy?" / "Ready to go live?" when standing deploy authorization exists is a Contract violation.
*   **Parallel Build Swarm & Proactive Context Engineering:** The build interior of Phase 3 (scaffolding) and Phase 4 (implementation) is parallelized — the agent partitions the work into an ownership-disjoint DAG (`swarm_plan.md`) and fans out multiple `Task` units in a single message, while shared surfaces (schema, routing, lockfiles, config) stay sequential and main-agent-owned. No worker's green is trusted: after merge the main agent re-runs the full phase Proof-of-Work suite, and the Apex Red Team still reviews the merged artifact single-threaded (it degrades cleanly to sequential when `Task` is unavailable). Context is managed proactively rather than reactively: the agent tracks GREEN/AMBER/RED budget tiers, runs a compaction ladder (archive completed todos → distill `decisions.md` → offload heavy reads to a `Task` sub-agent that returns only the distilled result → write a heartbeat `context_checkpoint.md` for minimal-loss, re-derivable resume — written at every boundary event, not only at RED), then lets OpenCode's native auto-compaction reclaim the window.
*   **Design-Skills Acceleration:** As its first step, Phase 5 enumerates the awesome-design-skills registry NON-INTERACTIVELY by fetching its `index.json` directly (`curl -fsSL .../skills/index.json` — never the interactive `typeui.sh list`, which hangs headless), matches a skill to the project archetype, and pulls the chosen slug's `SKILL.md`. If the CLI is used it MUST be the fully-flagged non-interactive form (`npx -y typeui.sh pull <slug> -f skill -p <provider> --dry-run < /dev/null`, timeout-wrapped). Registry skills routinely ship banned hex and zero OKLCH, so the agent EXPECTS to TRANSFORM, not adopt: every pulled hex is converted to `oklch()` and pure black/white is clamped before merge into `design-system/MASTER.md`. This is a probed, default accelerator — only if the registry genuinely can't be reached does it log a conditional gate and fall back to the built-in 2026 Impeccable Standard — and Studio Prime's banned-pattern rules always WIN over any pulled style (glassmorphism, pure black/white, card-ception are stripped).
*   **Deployment Briefing at Sign-Off:** At the end of a successful run the agent emits a concise, platform-aware Deployment Briefing as its final message (and persists it to `.studio/state/deployment_briefing.md` + HANDOFF §10): the verified live access point (deployed URL or `http://localhost:<port>` + PID), the exact deploy/redeploy/rollback operations for the resolved target (or the `deploy_ready.sh` go-live steps + ranked host recommendation on no-credential runs), proactive next-step suggestions, and the "Continue Studio Prime" resume command.
*   **Northstar Validation Gate:** Before final sign-off in Phase 6, the agent compares the original Phase 1 requirements (`northstar.md` — immutable, never re-captured) against the final deliverables and writes every gap to a failure-findings file (`northstar_gap_analysis.md`). Isolated gaps trigger surgical remediation: only the phase(s) that own each gap are re-entered (a missing API re-enters P3/P4, an a11y miss P5, a deploy miss P6). A "deploy miss" with standing deploy credentials is remediated by re-entering P6 and DEPLOYING (never by asking a human); without credentials it is remediated by ensuring a live localhost server (`[LOCAL_LIVE]`). If the failure is systemic — smoke tests fail, most requirements are unmet, or the blueprint/architecture itself is implicated — it escalates: the agent re-enters Phase 1 and re-walks the entire pipeline with the failure findings as mandatory input to every phase. The two tiers have INDEPENDENT budgets (a larger surgical cap, a small systemic cap) plus a no-progress convergence guard. When the budget is reached, the agent FIRST guarantees the largest LIVE subset is serving (feature-flag the unmet feature off, confirm HTTP 200) — never leaving a dark process when a reduced-scope live product was achievable — then defers non-critical gaps to `TECH_DEBT`; only genuine critical-path gaps escalate to a human (interactive) or checkpoint-exit (unattended), and only after the bounded deploy retries exhaust. A liveness re-probe after any redeploy is a hard, non-downgradable gate before sign-off.

---

## ⚙️ Function: Orchestration & The Gating System

The operational architecture of OpenCode Prime is defined by a strictly sequential, 6-Phase lifecycle. Progression through these phases is not determined by heuristic guessing, but rather governed by a deterministic, mathematically verifiable gating system. 

### The Scratchpad DAG & Transition Checklists
Before the system is permitted to transition from Phase *N* to Phase *N+1*, it must construct a Directed Acyclic Graph (DAG) in its scratchpad memory (`<phase_gate_checklist>`). This enforces a mandatory sequential evaluation of:
1.  **Prerequisite Verification:** Ensuring all artifact dependencies from Phase *N* exist on the filesystem.
2.  **Proof-of-Work Compilation:** Gathering the raw `stdout` from testing and verification scripts.
3.  **Apex Red Team Adjudication:** Submitting the collected evidence to an isolated sub-agent for adversarial review.

### The Apex Red Team (Multi-Agent Debate)
The Apex Red Team operates as an isolated sub-agent spawned via OpenCode's `Task` tool. It executes a 3-round Divergent Persona Protocol:
*   **Round 1 (Steelman):** The sub-agent adopts the persona of the original developer, tasked with vigorously defending the implementation and citing specific passing tests.
*   **Round 2 (Adversarial):** The persona shifts to a zero-trust security auditor operating under the absolute assumption that a vulnerability *does* exist. It attacks the assertions made in Round 1.
*   **Round 3 (Synthesis):** A principal engineer persona adjudicates the debate, classifying findings into `[BLOCKER]` (halts execution, mandates a safe `git stash` rollback, and triggers HaaS), `[TECH_DEBT]` (logs to `.studio/todos.md` but permits progression), or `[GREEN_FLAG]` (clean pass). A product that is undeployed despite standing deploy credentials, or has no running server at handoff, is auto-remediated by deploying / (re)starting the server — never converted into a human checkpoint; and a logged `[LOCAL_LIVE]` localhost server on a no-credential run satisfies the deploy gate (the reviewer must accept it, not flag the absence of a cloud deploy as a fresh `[BLOCKER]`).

### The 6-Phase Lifecycle

1.  **Phase 1: Blueprint & Baseline**
    *   **Function:** Establishes the foundational architectural constraints. Initiates mandatory web fetches (min 3, recommended 5-10+) via the `webfetch` tool to anchor the model in current, verified documentation rather than parametric memory.
    *   **Output:** Generates `northstar.md` (immutable requirements), `decisions.md` (the architectural state machine), and an initial `data_contracts.md` (refined in Phase 2).
2.  **Phase 2: Link**
    *   **Function:** Defines the integration seams. This phase enforces the "Data-First Rule," establishing database migration strategies, external API wiring, and identity management before any business logic is written.
    *   **Output:** Complete `data_contracts.md` and integration blueprints.
3.  **Phase 3: Architecture & Scaffolding**
    *   **Function:** Translates the blueprint into syntactically valid code structures. Prohibits business logic. Focuses on strict types, interface boundaries, TDD stubs, **validated multi-stage Docker configs** (`docker build --check` + `docker-compose config -q`), **CI/CD workflows validated for syntax and required test/lint/security jobs**, and **migrations verified via dry-run** — file existence alone is not accepted.
4.  **Phase 4: Implement & Verify**
    *   **Function:** Executes business logic to satisfy TDD stubs. Enforces 80%+ code coverage, **executed E2E/integration tests** (implemented and run with captured pass/fail stdout — not stubs), a **binary security gate** (HIGH/CRITICAL dependency-CVE audit + secrets-leak scan), **validated JSON stdout logs** (parsed to confirm `timestamp`/`level`/`message`), and strict **OWASP security hardening** (schema validation, audited HttpOnly/Secure/SameSite cookies, safe JWT RS256 signing, and API rate-limiting per IP).
5.  **Phase 5: Stylize (UI/UX Polish)**
    *   **Function:** Transforms functional interfaces into production-grade aesthetics. Enforces the 2026 Impeccable Design Standard, ensuring **WCAG 2.1 AA accessibility verified by an executed axe-core/pa11y audit** (run against the live app, zero critical/serious violations gated as BLOCKER), a verifiable component state matrix (missing focus ring → BLOCKER), fluid motion physics, and LCP (<2.5s) / CLS (<0.1) metrics.
6.  **Phase 6: Release**
    *   **Function:** The final operational check that ends with the product LIVE. Integrates a **tested graceful OS-signal shutdown** (SIGTERM sent to a *disposable* instance, asserting clean drain within the window — the drain test never leaves the product dead; the handoff server is started fresh afterwards), production **DB migrations (dry-run before apply)**, **synthetic-error → alert-propagation verification** (confirms the event reaches Sentry/Datadog *and* the on-call channel via a guarded, non-prod-exposed error path built in Phase 3/4 — but when NO error-tracker DSN / alert webhook was supplied at intake this gate degrades to a `[CONDITIONAL_GATE]` `[PRIORITY:H]` TECH_DEBT the reviewer must accept, so a successfully-deployed app is never failed over a missing monitoring nicety; it is a BLOCKER only when monitoring creds WERE provided and propagation still fails), and a **wall-clock-measured rollback dry-run (<5min)**. The deploy step is keyed on credentials: **if deploy credentials were provided at intake the agent autonomously deploys live (the credentials ARE the authorization) and hands over a working production URL**; if none were provided it builds the artifact, runs a **detach-survival probe** (proving a background process actually outlives a shell exit on this host — preferring daemon-owned supervision like `docker compose up -d`), and only on a PASSING probe starts a **detached localhost server that stays running for handoff** (URL + PID recorded in `.studio/state/local_live.md`) and emits `deploy_ready.sh` for going live later. If the probe FAILS on the host (the OS tears down the process tree at session exit — common on some sandboxed Windows/PowerShell shells), it does NOT claim LIVE: it emits `[DEPLOY_READY: localhost cannot outlive this session]`, states it in the Deployment Briefing, and exits honestly rather than leaving a stale PID. Either way it executes **automated post-deploy smoke tests** against the live URL (production or `http://localhost:<port>`) with rollback wired to a real command on any 5xx, **re-probes the live URL for HTTP 200 immediately before sign-off**, and concludes with a comprehensive 17-section `HANDOFF.md` consolidation (placeholder-free, grep-gated, with the verified live URL + PID) and the final **Northstar Validation Gate**.

---

## 📥 Expected Input/Output (I/O)

To effectively orchestrate the development lifecycle, OpenCode Prime expects structured inputs and generates deterministic outputs. Understanding this I/O contract is critical for maximizing the system's efficacy.

### **Input Modalities**
*   **Intake Triggers:** Natural language commands such as `"Start Studio Prime"` or `"Continue Studio Prime"`.
*   **Requirement Documents (PRDs):** Structured text outlining the core problem, target audience, and functional requirements. 
*   **Design Assets & Constraints:** Optional guidelines, reference URLs, or brand tokens.
*   **Human-as-a-Service (HaaS) Feedback:** Explicit approvals, architectural overrides, or *reactive* credential provisions when the system halts at an enforced gate. (Note the distinction: hosting/deploy credentials supplied at intake — in the brief, `.env`, env vars, or secret store — are NOT reactive gap-fillers but STANDING authorization to deploy live; their presence fires no gate at all.)

### **Output Modalities**
*   **Terminal Execution (`stdout`/`stderr`):** Verifiable proof of execution, testing, and deployment.
*   **Persistent State Files (`.studio/`):** Markdown and JSON artifacts that maintain the agent's memory, architectural decisions, and active task lists.
*   **Production Code:** Fully typed, tested, and linted source code adhering to established conventions.
*   **Structured UI Prompts:** Interactive questions and options rendered in the OpenCode interface via the `question` tool.
*   **Adversarial Review Logs:** Detailed markdown reports from the Apex Red Team, classifying the phase's output into `[GREEN_FLAG]`, `[TECH_DEBT]`, or `[BLOCKER]`.

---

## 📁 File Structure & Reasoning

Studio Prime writes its state to the disk to cure "LLM Amnesia" and survive context limits.

```text
.studio/
├── todos.md                # Active task list (mirrored in the OpenCode TUI)
├── archive.md              # Flushed tasks to keep the active context small
├── blocked.md              # Detailed logs of failed escalation attempts
├── state/
│   ├── northstar.md            # The immutable original requirements
│   ├── phase*_research.md      # Raw search findings
│   ├── swarm_plan.md           # Parallel Build Swarm ownership DAG (P3/P4)
│   ├── context_checkpoint.md   # Heartbeat context handoff (minimal-loss resume; written at every boundary)
│   ├── deploy_target.md        # Resolved deploy target + standing-auth marker
│   ├── deploy_ready.sh         # Exact go-live commands (no-creds path)
│   ├── local_live.md           # {url, pid, start/stop cmd} for the running localhost server
│   ├── critical_journeys.json  # {id, slug, acceptance_criterion} — journey↔test mapping (P1)
│   ├── effects.md              # Side-effect ledger (idempotency) for safe re-walk/resume
│   ├── budget.md               # Optional wall-clock / tool-call spend guard
│   ├── pow/                    # Per-gate tee-captured proof-of-work logs
│   └── deployment_briefing.md  # Final platform-aware briefing (also in chat + HANDOFF §10)
└── checklists/             # Mandatory DAG gate checkpoints

architecture/
├── decisions.md            # Core architecture decisions
├── data_contracts.md       # DB schemas and API contracts
├── integration_plan.md     # Integration blueprint
└── phase_snapshots/        # Hard checkpoints allowing safe rollback

# Synthesized, deduplicated research lives at .studio/state/phase[N]_research.md
# (the legacy architecture/research_spike.md name is deprecated)

design-system/
└── MASTER.md               # Global design tokens (OKLCH, typography)

.tmp/
├── verify.sh / .ps1        # Local CI/CD test script
└── research_*.md           # Parallelized research (prevents context contamination)
```

---

## 🔬 Credits & Scientific Foundation

Studio Prime's mechanics are not arbitrary; they are implementations of peer-reviewed AI and LLM research:

*   **Apex Red Team (Multi-Agent Debate):** Based on research like *ChatEval: Towards Better LLM-based Evaluators through Multi-Agent Debate (Chen et al., 2023)*. Studies prove that assigning distinct, isolated adversarial roles reduces LLM hallucinations and false positives far better than single-agent self-critique. OpenCode's `Task` tool makes this physical isolation possible.
*   **Proof-of-Work (Verbal Reinforcement):** Rooted in *Reflexion: Language Agents with Verbal Reinforcement Learning (Shinn et al., 2023)*. Converting raw environmental feedback (terminal stdout) into a linguistic self-reflection allows the agent to break out of repetitive failure loops.
*   **Context Compaction (`.studio/`):** Based on architectures like *MemGPT: Towards LLMs as Operating Systems (Packer et al., 2023)*. Writing state to a persistent filesystem and retrieving it Just-In-Time (`grep`/`read`) prevents the "Lost in the Middle" context degradation phenomenon.

---
*Architected by Suraj Kuncham. Designed to push the boundaries of autonomous software engineering.*
