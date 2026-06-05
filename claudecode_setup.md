# Claude Code Prime

[← Back to README](README.md) · Setup guide for [`claudecode_prime.md`](claudecode_prime.md)

**The Studio Prime superprompt — optimized for Claude Code's native Task tool, hooks, and structured questions.**

---

## What is Claude Code Prime?

Claude Code Prime is more than a prompt — it is a complete production engineering playbook. It transforms your standard AI coding assistant into a relentless, autonomous Principal Architect, UI/UX Designer, and Zero-Trust Red Teamer.

While the Studio Prime architecture runs on multiple platforms, **Claude Code is the most rigorously instrumented environment for it.** Claude Code natively supports true sub-agent dispatch via the `Task` tool with the `subagent_type` parameter, deterministic system-side hooks via `.claude/settings.json` (hooks are merged into the existing `.claude/settings.json` under a top-level `hooks` key), and structured user prompts via the `AskUserQuestion` tool. When Studio Prime launches its Apex Red Team to audit code, Claude Code physically spins up a fresh, blinded context window. The Red Team doesn't know the developer's original assumptions, struggles, or compromises. This guarantees **true adversarial review**, free from the context contamination that plagues other platforms.

With Claude Code Prime, you achieve the ultimate "Sleep Test": Start a session, go to sleep, and wake up to a LIVE product — a working deployed URL when hosting credentials were provided (providing the credentials IS the authorization), or a fully functional product with its localhost server still running (URL + PID documented) when they were not. **(Partial caveat: Claude Code does not auto-compact long sessions. Either the agent or the operator must periodically invoke `/compact` to free tokens — see "Context Flush Note" below.)**

---

## 🚀 Setup Process

Follow these steps exactly to install Claude Code Prime and unleash its full capabilities.

### Option A: As CLAUDE.md (Recommended, System-Level)
This method ensures Studio Prime is active by default every time you launch Claude Code in this project.

1. **Open your terminal** and navigate to your project workspace.
2. **Copy the superprompt** into the project-level system prompt file:
   ```bash
   cp claudecode_prime.md CLAUDE.md
   ```
3. **Launch Claude Code**:
   ```bash
   claude
   ```

### Option B: As an On-Demand Skill
This method allows you to load Studio Prime modularly only when you explicitly want to use it.

1. **Create the skills directory**:
   ```bash
   mkdir -p .claude/skills/studio-prime
   ```
2. **Copy the superprompt** into the skill file:
   ```bash
   cp claudecode_prime.md .claude/skills/studio-prime/SKILL.md
   ```
3. **Launch Claude Code**:
   ```bash
   claude
   ```

### Permissions (Required)
Configure `.claude/settings.local.json` to pre-approve the tool surface Studio Prime needs for full autonomy. Without these, Claude Code will halt to ask for permission on each invocation:

```json
{
  "permissions": {
    "allow": [
      "Bash", "Read", "Write", "Edit", "Glob", "Grep",
      "TaskCreate", "TaskUpdate", "TaskList", "TaskGet",
      "AskUserQuestion", "WebSearch", "WebFetch", "Task"
    ]
  }
}
```

### Triggering the Agent

Once Claude Code is running, trigger the system by typing:

```
"Start Studio Prime"
```

**The Intake Gate:** Upon invocation, Claude Code Prime will NOT dump a massive wall of text. Instead, it leverages Claude Code's native `AskUserQuestion` tool to present a clean structured menu asking if you want to start a **NEW PROJECT** (guided discovery) or work on an **EXISTING CODEBASE** (add features or transform).

---

## ✨ Core Features

Claude Code Prime leverages Claude Code's native APIs to enforce rigorous software engineering standards, transforming the LLM from a stochastic text generator into a deterministic engineering environment:

*   **Proof-of-Work Anti-Hallucination:** LLMs inherently struggle to differentiate between intended outcomes and actual execution results. To counteract this, Claude Code Prime enforces a strict, three-part empirical verification protocol. Before asserting task completion, the agent must generate a `<prediction>` of the expected terminal output, execute the command via the `Bash` tool, and synthesize a `<divergence_analysis>`. By comparing the expected state against the actual `stdout`, the agent is forced to linguistically process any discrepancies. Furthermore, if a successful output is indistinguishable from a generic response, the system injects a deliberately malformed flag into a subsequent execution to empirically verify the terminal's responsiveness, thereby eliminating blind assumptions.
*   **The 2026 Impeccable Design Standard:** Studio Prime categorically rejects the homogenized "AI Starter Pack" aesthetic. It enforces a strict mathematical approach to UI engineering, dictating the use of the `OKLCH` color space for perceptual uniformity and accessibility. It mandates physics-based, 120fps GPU-accelerated motion (utilizing libraries such as `motion/react` for fluid, spring-based interactions), while explicitly prohibiting ubiquitous anti-patterns like absolute blacks (`#000000`), nested card hierarchies, and unstructured glassmorphism.
*   **Hooks-Driven Verification (Claude-Specific):** Claude Code uniquely exposes a deterministic event hook surface via `.claude/settings.json`. Studio Prime exploits this by auto-generating two `PostToolUse` hooks during Phase 1: a `Write|Edit` matcher that runs `npx eslint --fix` against the just-touched file, and a `Bash` matcher that appends a verification line to `.studio/state/hooks.log` so every shell invocation leaves a proof-of-work audit trail. Additional gates such as `prettier --write` or type-checks are documented as drop-in additions you can wire into the same `PostToolUse` matcher, but only these two hooks ship auto-generated to keep the default surface fast and stack-agnostic. This shifts quality enforcement from the LLM's probabilistic judgement to the host OS's deterministic execution, eliminating an entire class of "the agent forgot to lint" failure modes while saving thousands of tokens per phase. Combined with Claude Code's strict permissions model, hooks form a system-side moat that adversarial or malformed agent output cannot bypass.
*   **Autonomous Escalation & State-Preserving Rollback:** Brittle debugging loops are a primary failure vector for autonomous agents. To resolve this, Claude Code Prime implements a bounded, 5-cycle recovery protocol. When confronted with an error, the agent is granted three direct remediation attempts. Upon failure, it is mathematically forced to pivot: executing targeted research via the `WebSearch` and `WebFetch` tools to inject external context for the specific error, followed by one final retry. This cycle can repeat up to five times (yielding 20 total iterations) before the agent triggers a hard ceiling. After exhausting the repair budget (5 cycles per failure, 20 iterations max), the agent executes a safe `git stash`, logs forensic data to `.studio/blocked.md`, then auto-isolates the failing module — logging it as a `[PRIORITY:H]` TECH_DEBT entry — and continues the pipeline with the remaining scope. Human-as-a-Service (HaaS) is invoked only when the isolated module is a critical-path dependency; in unattended (Sleep Test) mode the agent checkpoint-exits non-zero instead of blocking on a human. A deploy step with provided credentials is NEVER a "blocked critical-path module" — the credentials ARE standing authorization, so it auto-deploys live without re-prompting; the non-zero checkpoint-exit is reserved for genuinely unrecoverable failures, and even then a fully built app is left with its localhost server still running for handoff.
*   **Zero-Gap Phase Chaining:** Upon receiving a passing verdict from the Apex Red Team, the agent autonomously transitions to the next phase without pausing for human approval. The agent operates as a continuous pipeline, only halting for explicit blockers or the final Northstar Validation Gate. The Northstar Validation Gate never halts for deploy authorization — provided credentials ARE the authorization, so the deploy/handoff is part of sign-off, never its own checkpoint; with no creds the gate requires a live localhost server (URL+PID) before sign-off.
*   **Northstar Validation Gate:** Before final sign-off in Phase 6, the agent compares the original Phase 1 requirements (`northstar.md` — immutable, never re-captured) against the final deliverables and writes every gap to a failure-findings file (`northstar_gap_analysis.md`). Isolated gaps trigger surgical remediation: only the phase(s) that own each gap are re-entered (a missing API re-enters P3/P4, an a11y miss P5, a deploy miss P6). If the failure is systemic — smoke tests fail, most requirements are unmet, or the blueprint/architecture itself is implicated — it escalates: the agent re-enters Phase 1 and re-walks the entire pipeline with the failure findings as mandatory input to every phase. A "deploy miss" with standing credentials is auto-remediated by re-entering P6 and performing the live deploy (never escalated); without creds it is remediated by ensuring the localhost server is running (`[LOCAL_LIVE]`). Both paths share a 2-cycle cap, after which non-critical gaps are deferred to `TECH_DEBT` and only critical-path gaps escalate to a human — and only after the bounded retries are exhausted.

---

## ⚙️ Function: Orchestration & The Gating System

The operational architecture of Claude Code Prime is defined by a strictly sequential, 6-Phase lifecycle. Progression through these phases is not determined by heuristic guessing, but rather governed by a deterministic, mathematically verifiable gating system.

### The Scratchpad DAG & Transition Checklists
Before the system is permitted to transition from Phase *N* to Phase *N+1*, it must construct a Directed Acyclic Graph (DAG) in its scratchpad memory (`<phase_gate_checklist>`). This enforces a mandatory sequential evaluation of:
1.  **Prerequisite Verification:** Ensuring all artifact dependencies from Phase *N* exist on the filesystem.
2.  **Proof-of-Work Compilation:** Gathering the raw `stdout` from testing and verification scripts.
3.  **Apex Red Team Adjudication:** Submitting the collected evidence to an isolated sub-agent for adversarial review.

### The Apex Red Team (Multi-Agent Debate)
The Apex Red Team operates as an isolated sub-agent spawned via Claude Code's `Task` tool with an explicit `subagent_type` parameter — this guarantees the reviewer runs in a fresh, blinded context window with no conversational history from the main thread. It executes a 3-round Divergent Persona Protocol:
*   **Round 1 (Steelman):** The sub-agent adopts the persona of the original developer, tasked with vigorously defending the implementation and citing specific passing tests.
*   **Round 2 (Adversarial):** The persona shifts to a zero-trust security auditor operating under the absolute assumption that a vulnerability *does* exist. It attacks the assertions made in Round 1.
*   **Round 3 (Synthesis):** A principal engineer persona adjudicates the debate, classifying findings into `[BLOCKER]` (halts execution, mandates a safe `git stash` rollback, and triggers HaaS) or `[TECH_DEBT]` (logs to `.studio/todos.md` but permits progression); absence of either yields the overall verdict `[GREEN_FLAG]` (clean pass). In Phase 6, "app not yet deployed despite valid creds" or "no running server" are auto-remediated by deploying / relaunching — never emitted as a HaaS-triggering `[BLOCKER]`; a logged `[LOCAL_LIVE]` localhost server satisfies the deploy gate on no-creds runs and the reviewer must accept it.

### The 6-Phase Lifecycle

1.  **Phase 1: Blueprint & Baseline**
    *   **Function:** Establishes the foundational architectural constraints. Initiates mandatory `WebSearch` and `WebFetch` invocations (min 3, recommended 5-10+) to anchor the model in current, verified documentation rather than parametric memory. Generates the `.claude/settings.json` file that will enforce deterministic quality checks for the remainder of the build.
    *   **Output:** Generates `northstar.md` (immutable requirements), `decisions.md` (the architectural state machine), and `.claude/settings.json`.
2.  **Phase 2: Link**
    *   **Function:** Defines the integration seams. This phase enforces the "Data-First Rule," establishing database migration strategies, external API wiring, and identity management before any business logic is written.
    *   **Output:** Complete `data_contracts.md` and integration blueprints.
3.  **Phase 3: Architecture & Scaffolding**
    *   **Function:** Translates the blueprint into syntactically valid code structures. Prohibits business logic. Focuses on strict types, interface boundaries, TDD stubs, **multi-stage Docker configs (`Dockerfile`, `docker-compose.yml`)**, **automated CI/CD workflows**, and **database migrations**.
4.  **Phase 4: Implement & Verify**
    *   **Function:** Executes business logic to satisfy TDD stubs. Enforces 80%+ code coverage, **fully implemented AND executed E2E/integration tests covering 100% of critical user journeys (stub-only suites are a BLOCKER)**, structured **JSON stdout logs**, and strict **OWASP security hardening** (schema validation, secure HttpOnly/Secure cookies, safe JWT RS256 signing, and API rate-limiting per IP).
5.  **Phase 5: Stylize (UI/UX Polish)**
    *   **Function:** Transforms functional interfaces into production-grade aesthetics. Enforces the 2026 Impeccable Design Standard, ensuring **WCAG 2.1 AA accessibility verified via an executed runtime a11y audit (pa11y or axe-core against the running app); critical/serious violations are a BLOCKER, and a stub-only or audit-only submission is itself a BLOCKER**, comprehensive component state matrices, fluid motion physics, and LCP (<2.5s) / CLS (<0.1) metrics.
6.  **Phase 6: Release**
    *   **Function:** The final pre-flight operational check. Tests **graceful OS signal handling (`SIGTERM`/`SIGINT`)** against a throwaway instance (the handler is verified, then the server is restarted — the drain test never leaves the product dead at handoff), runs production **DB migrations** and **synthetic alert pipeline verification**, and handles the deploy keyed on credentials: **if deploy credentials were provided at intake, the agent autonomously deploys live (the credentials ARE the authorization) and hands over a working production URL; if none were provided, it leaves the production build running as a detached localhost server (URL+PID documented) so the product is still live at handoff.** It then executes **automated post-deploy smoke tests** against that live target, re-probes the live URL for HTTP 200 immediately before sign-off, and concludes with a comprehensive 17-section `HANDOFF.md` consolidation (recording the live URL+PID) and the final **Northstar Validation Gate**.

---

## 📥 Expected Input/Output (I/O)

To effectively orchestrate the development lifecycle, Claude Code Prime expects structured inputs and generates deterministic outputs. Understanding this I/O contract is critical for maximizing the system's efficacy.

### **Input Modalities**
*   **Intake Triggers:** Natural language commands such as `"Start Studio Prime"` or `"Continue Studio Prime"`.
*   **Requirement Documents (PRDs):** Structured text outlining the core problem, target audience, and functional requirements.
*   **Design Assets & Constraints:** Optional guidelines, reference URLs, or brand tokens.
*   **Human-as-a-Service (HaaS) Feedback:** Explicit approvals, architectural overrides, or credential provisions when the system halts at an enforced gate, delivered through Claude Code's `AskUserQuestion` tool as a structured multiple-choice prompt. Credentials provided at initiation constitute standing deploy authorization — the agent MUST deploy live without re-prompting; no gate fires to request deploy credentials when intake creds are already present.

### **Output Modalities**
*   **Terminal Execution (`stdout`/`stderr`):** Verifiable proof of execution, testing, and deployment, captured via the `Bash` tool.
*   **Persistent State Files (`.studio/`):** Markdown and JSON artifacts that maintain the agent's memory, architectural decisions, and active task lists.
*   **Production Code:** Fully typed, tested, and linted source code adhering to established conventions, with hook-enforced formatting on every write.
*   **Structured UI Prompts:** Interactive questions and options rendered in the Claude Code interface via the `AskUserQuestion` tool.
*   **Adversarial Review Logs:** Detailed markdown reports from the Apex Red Team, classifying the phase's output into `[GREEN_FLAG]`, `[TECH_DEBT]`, or `[BLOCKER]`.

---

## 📁 File Structure & Reasoning

Studio Prime writes its state to the disk to cure "LLM Amnesia" and survive context limits.

```text
.studio/
├── todos.md                # Active task list (mirrored via TaskCreate/TaskUpdate)
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

**Claude-Specific Artifacts:** In addition to the cross-platform `.studio/` and `architecture/` trees, Claude Code Prime maintains two host-level configuration files that are unique to this platform:
*   **`.claude/settings.json`** — generated in Phase 1; defines `PostToolUse` triggers (under a top-level `hooks` key) that run linters, formatters, and type-checks after every `Edit`/`Write`. This is what makes Claude Code's verification loop deterministic rather than probabilistic.
*   **`.claude/settings.local.json`** — pre-approves the tool surface listed in the Permissions section above so the agent runs without interruption.

---

## ⚠️ Context Flush Note (Claude-Specific)

Unlike some platforms that auto-compact long sessions, **Claude Code does not transparently summarize or evict conversational history**. As a Studio Prime session progresses through all six phases — with Red Team reviews, research gates, and proof-of-work logs — the context window will eventually saturate.

The agent itself cannot invoke slash commands programmatically; the `/compact` command must be issued by the human operator. Claude Code Prime mitigates this in two ways:
1.  **Proactive prompting:** The agent emits a `[CONTEXT NOTICE]` recommending `/compact` after every ~15 tool invocations or after reading any file larger than 500 lines.
2.  **Prime Directive #2 (Memory Persistence) as the safety net:** Every architectural decision, blocker, and phase verdict is written to `.studio/` immediately. Even if context is lost or `/compact` aggressively prunes history, the agent re-orients on resume by reading `architecture/decisions.md`, `.studio/todos.md`, and `.studio/state/northstar.md`. The filesystem is the source of truth, not the conversation.

In practice: when you see the `[CONTEXT NOTICE]`, type `/compact` and then `"Continue Studio Prime"`. The agent will pick up exactly where it left off.

---

## 🔬 Credits & Scientific Foundation

Studio Prime's mechanics are not arbitrary; they are implementations of peer-reviewed AI and LLM research:

*   **Apex Red Team (Multi-Agent Debate):** Based on research like *ChatEval: Towards Better LLM-based Evaluators through Multi-Agent Debate (Chen et al., 2023)*. Studies prove that assigning distinct, isolated adversarial roles reduces LLM hallucinations and false positives far better than single-agent self-critique. Claude Code's `Task` tool (with `subagent_type`) makes this physical isolation possible.
*   **Proof-of-Work (Verbal Reinforcement):** Rooted in *Reflexion: Language Agents with Verbal Reinforcement Learning (Shinn et al., 2023)*. Converting raw environmental feedback (terminal stdout) into a linguistic self-reflection allows the agent to break out of repetitive failure loops.
*   **Context Compaction (`.studio/`):** Based on architectures like *MemGPT: Towards LLMs as Operating Systems (Packer et al., 2023)*. Writing state to a persistent filesystem and retrieving it Just-In-Time (`grep`/`read`) prevents the "Lost in the Middle" context degradation phenomenon.

---
*Architected by Suraj Kuncham. Designed to push the boundaries of autonomous software engineering.*
