# OpenCode Prime

[← Back to README](README.md) · Setup guide for [`opencode_prime.md`](opencode_prime.md)

**The Studio Prime superprompt — perfectly optimized for OpenCode's native toolset.**

---

## What is OpenCode Prime?

OpenCode Prime is not just a prompt; it is a **Production-Grade AI Operating System**. It transforms your standard AI coding assistant into a relentless, autonomous Principal Architect, UI/UX Designer, and Zero-Trust Red Teamer. 

While the Studio Prime architecture runs on multiple platforms, **OpenCode is the definitive environment for it.** This is because OpenCode natively supports true sub-agent dispatch (via the `Task` tool). When Studio Prime launches its Apex Red Team to audit code, OpenCode physically spins up a fresh, blinded context window. The Red Team doesn't know the developer's original assumptions, struggles, or compromises. This guarantees **true adversarial review**, free from the context contamination that plagues other platforms.

With OpenCode Prime, you achieve the ultimate "Sleep Test": Start a session, go to sleep, and wake up to a completed, rigorously audited application.

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

*   **Proof-of-Work Anti-Hallucination:** LLMs inherently struggle to differentiate between intended outcomes and actual execution results. To counteract this, OpenCode Prime enforces a strict, three-part empirical verification protocol. Before asserting task completion, the agent must generate a `<prediction>` of the expected terminal output, execute the command via the `<execution>` node, and synthesize a `<divergence_analysis>`. By comparing the expected state against the actual `stdout`, the agent is forced to linguistically process any discrepancies. Furthermore, if a successful output is indistinguishable from a generic response, the system injects a deliberately malformed flag into a subsequent execution to empirically verify the terminal's responsiveness, thereby eliminating blind assumptions.
*   **The 2026 Impeccable Design Standard:** Studio Prime categorically rejects the homogenized "AI Starter Pack" aesthetic. It enforces a strict mathematical approach to UI engineering, dictating the use of the `OKLCH` color space for perceptual uniformity and accessibility. It mandates physics-based, 120fps GPU-accelerated motion (utilizing libraries such as `motion/react` for fluid, spring-based interactions), while explicitly prohibiting ubiquitous anti-patterns like absolute blacks (`#000000`), nested card hierarchies, and unstructured glassmorphism.
*   **TUI Synchronization via Context-Aware Tooling:** OpenCode Prime deeply integrates with the host platform's Terminal User Interface (TUI). By interfacing directly with the native `todowrite` tool, the agent dynamically maps its internal Directed Acyclic Graph (DAG) state to the external UI task panel. This bidirectional synchronization ensures the human operator maintains perfect, real-time observability of the agent's phase progression without needing to parse the raw context stream.
*   **Autonomous Escalation & State-Preserving Rollback:** Brittle debugging loops are a primary failure vector for autonomous agents. To resolve this, OpenCode Prime implements a bounded, 5-cycle recovery protocol. When confronted with an error, the agent is granted three direct remediation attempts. Upon failure, it is mathematically forced to pivot: executing targeted web research via the `webfetch` tool to inject external context for the specific error, followed by one final retry. This cycle can repeat up to five times (yielding 20 total iterations) before the agent triggers a hard ceiling. After exhausting the repair budget (5 cycles per failure, 20 iterations max), the agent executes a safe `git stash`, logs forensic data to `.studio/blocked.md`, then auto-isolates the failing module — logging it as a `[PRIORITY:H]` TECH_DEBT entry — and continues the pipeline with the remaining scope. Human-as-a-Service (HaaS) is invoked only when the isolated module is a critical-path dependency; in unattended (Sleep Test) mode the agent checkpoint-exits non-zero instead of blocking on a human.
*   **Zero-Gap Phase Chaining:** Upon receiving a passing verdict from the Apex Red Team, the agent autonomously transitions to the next phase without pausing for human approval. The agent operates as a continuous pipeline, only halting for explicit blockers or the final Northstar Validation Gate.
*   **Northstar Validation Gate:** Before final sign-off in Phase 6, the agent compares the original Phase 1 requirements (`northstar.md` — immutable, never re-captured) against the final deliverables and writes every gap to a failure-findings file (`northstar_gap_analysis.md`). Isolated gaps trigger surgical remediation: only the phase(s) that own each gap are re-entered (a missing API re-enters P3/P4, an a11y miss P5, a deploy miss P6). If the failure is systemic — smoke tests fail, most requirements are unmet, or the blueprint/architecture itself is implicated — it escalates: the agent re-enters Phase 1 and re-walks the entire pipeline with the failure findings as mandatory input to every phase. Both paths share a 2-cycle cap, after which non-critical gaps are deferred to `TECH_DEBT` and only critical-path gaps escalate to a human.

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
*   **Round 3 (Synthesis):** A principal engineer persona adjudicates the debate, classifying findings into `[BLOCKER]` (halts execution, mandates a safe `git stash` rollback, and triggers HaaS), `[TECH_DEBT]` (logs to `.studio/todos.md` but permits progression), or `[GREEN_FLAG]` (clean pass).

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
    *   **Function:** The final pre-flight operational check. Integrates a **tested graceful OS-signal shutdown** (SIGTERM sent to the running process, asserting clean drain within the window), production **DB migrations (dry-run before apply)**, **synthetic-error → alert-propagation verification** (confirms the event reaches Sentry/Datadog *and* the on-call channel), a **wall-clock-measured rollback dry-run (<5min)**, and executes **automated post-deploy smoke tests** against production with rollback wired to a real command on any 5xx, concluding with a comprehensive 17-section `HANDOFF.md` consolidation (placeholder-free, grep-gated) and the final **Northstar Validation Gate**.

---

## 📥 Expected Input/Output (I/O)

To effectively orchestrate the development lifecycle, OpenCode Prime expects structured inputs and generates deterministic outputs. Understanding this I/O contract is critical for maximizing the system's efficacy.

### **Input Modalities**
*   **Intake Triggers:** Natural language commands such as `"Start Studio Prime"` or `"Continue Studio Prime"`.
*   **Requirement Documents (PRDs):** Structured text outlining the core problem, target audience, and functional requirements. 
*   **Design Assets & Constraints:** Optional guidelines, reference URLs, or brand tokens.
*   **Human-as-a-Service (HaaS) Feedback:** Explicit approvals, architectural overrides, or credential provisions when the system halts at an enforced gate.

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
│   ├── northstar.md        # The immutable original requirements
│   └── phase*_research.md  # Raw search findings
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
