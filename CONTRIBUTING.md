# Contributing to Studio Prime

Thanks for helping make Studio Prime the best public superprompt suite for autonomous, production-grade software engineering. This repo ships **prompts**, not application code — so "correctness" here means: an AI agent following a prompt reliably ships deployable, tested, documented software, on the platform the prompt targets, without hallucinating tools or stalling unattended.

## Repository layout

```
README.md                 # Landing page + platform chooser
studio_prime.md           # Universal prime — auto-detects the host & degrades gracefully
universal_setup.md        # Setup + deep-reference guide for the universal prime
<platform>_prime.md       # Platform-native superprompt (opencode/claudecode/codex/openclaw/antigravity)
<platform>_setup.md       # Setup + usage guide for that platform
LICENSE                   # MIT
```

Every change to a `*_prime.md` should keep its `*_setup.md` in sync, and changes that affect all platforms should land in the **universal** prime too (cross-platform parity is a hard requirement).

## The bar every prime must meet

1. **The Sleep Test.** Given a PRD + the required API keys/resources, a user triggers the agent, walks away, and returns to a **live** product: a working deployed URL when hosting credentials were provided (credentials supplied at intake ARE the deploy authorization), or a fully functional product with its localhost server still running (URL + PID documented) when they were not. A prime must carry the agent intake → 6-phase lifecycle → live handoff with **no stall, infinite loop, silent give-up, or human-wait that lacks an unattended fallback** — a deployable-but-dark artifact is not a passing Sleep Test. See the *Autonomous Execution Contract* in each prime.
2. **Production-grade output.** Deterministic DB migrations, Docker/CI-CD, OWASP + SAST + secrets scanning, **executed** unit + E2E/smoke tests (not stubs), structured logging, health probes, graceful shutdown, WCAG 2.1 AA, and a complete `HANDOFF.md` — each enforced by a concrete, executable, binary-gated **proof-of-work** check, never prose claims.
3. **No hallucinated tool signatures.** Every tool/command referenced must match the target platform's real, documented surface. Unverified signatures must be clearly hedged ("reverse-engineered — adjust to the live engine signature if rejected"), never asserted as hard contracts.
4. **Platform-native idioms.** Use each platform's own shell, sub-agent dispatch, task tool, web tool, and intake tool. Generic stack commands (docker/npm/pytest/gitleaks) run through the shell and may stay generic; show them in POSIX form with the documented POSIX↔PowerShell translation note.
5. **Cross-platform parity.** All six primes enforce the same 6-phase lifecycle, the same Apex Red Team 3-round protocol, the same proof-of-work discipline, and equivalent autonomy. Divergences must be justified by a real platform tool-surface difference.

## Proposing a change

1. Open an issue describing the gap (a tool signature that drifted, an autonomy break, a missing production mandate, a parity gap).
2. Branch, make the edit **surgically** — augment, don't gut; preserve the proof-of-work, repair-loop, and persistence machinery.
3. If you touch a tool signature, cite the platform's official docs in your PR. If docs are unavailable, hedge it.
4. Keep the prime, its setup guide, and (for cross-cutting changes) the universal prime consistent.
5. In your PR, state how the change affects the Sleep Test and whether parity is preserved.

## Reporting tool-surface drift

These platforms evolve fast. If a vendor changes a tool name, flag/CLI, or config path, open an issue with the source link — keeping the signatures accurate is the single most valuable contribution.
