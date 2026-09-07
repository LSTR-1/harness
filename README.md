# LSTR Solo Harness

> Production-ready, **zero-shot**, **self-discovering**, **metacognitive**, **self-evolving** AI-agent harness for [TRAE IDE](https://www.trae.ai/) **SOLO Mode**.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Platform: TRAE IDE](https://img.shields.io/badge/Platform-TRAE%20IDE%20SOLO-blue)](https://docs.trae.ai/ide/solo-mode)

The harness is **configuration, not code**. It layers TRAE SOLO Mode's native primitives — rules, skills, MCP, hooks, slash commands, and memory — into one coherent, auditable metacognitive control loop. TRAE SOLO's built-in agent is the runtime; this repository supplies the identity, reasoning discipline, deterministic guardrails, and self-evolution policy that make that runtime production-ready.

---

## Table of Contents

1. [The Four Capabilities](#the-four-capabilities)
2. [How It Works — The Six-Stage Loop](#how-it-works--the-six-stage-loop)
3. [Repository Structure](#repository-structure)
4. [Requirements](#requirements)
5. [Installation](#installation)
6. [Usage](#usage)
7. [The Reasoning Framework](#the-reasoning-framework)
8. [Self-Evolution Policy](#self-evolution-policy)
9. [Compliance & Data Sovereignty](#compliance--data-sovereignty)
10. [Research Foundation](#research-foundation)
11. [Roadmap](#roadmap)
12. [Contributing](#contributing)
13. [License](#license)

---

## The Four Capabilities

| Capability | Definition | Mechanism |
| --- | --- | --- |
| **Zero-shot** | Operates from introspective capability discovery at session start — no warm-up, fine-tuning, or training | `self-discover` skill + `SessionStart` hook |
| **Self-discovering** | Builds a runtime inventory of its own skills, MCP tools, rules, and memory | `self-discover` skill; MCP `tools/list` |
| **Metacognitive** | Calibrates knowledge boundaries and confidence, reasons over verifiable constraints, and gates its own output | LSTR 6-step reasoning chain; `Stop` acceptance hook |
| **Self-evolving** | Reflects on task outcomes and distils reusable atomic rules into persistent state | `self-evolve` skill + evolution ledger |

## How It Works — The Six-Stage Loop

Every non-trivial task proceeds through a deterministic loop. Each stage maps to a TRAE primitive and to a mechanism transferred from peer-reviewed research.

| Stage | Action | TRAE primitive | Research mechanism |
| --- | --- | --- | --- |
| 1. **Discover** | Build a capability inventory | `self-discover` skill; MCP `tools/list`; `SessionStart` hook | Zero-shot introspection |
| 2. **Plan** | 6-step reasoning + constraint planning + mental simulation | `metacognitive-plan` skill; MCP toolchain; `/plan` | DeepPlanning (local vs global constraints); Qwen-AgentWorld (simulation-as-thinking) |
| 3. **Act** | Guarded tool execution | `PreToolUse` hook; MCP `tools/call` | OccuBench (assume silent environment degradation) |
| 4. **Verify** | Fault-injection self-test with deterministic grading | `self-test` skill | OccuBench (fault injection); GDPevo (deterministic graders, not LLM-as-judge) |
| 5. **Evolve** | Reflection → distillation into skills/memory | `self-evolve` skill; `/reflect`; memory | GDPevo (reflect / fewshot / self supervision) |
| 6. **Record** | Audit trail + acceptance gate | `Stop` hook; reasoning records; `/audit` | Confident Decoding (do not over-trust the final output) |

## Repository Structure

```
lstr-solo-harness/
├── AGENTS.md                          # Portable identity + behavioral contract (AGENTS.md-compatible IDEs)
├── README.md                          # This file
├── LICENSE                            # MIT
├── docs/
│   └── harness-design.md              # Full architecture document
└── .trae/
    ├── mcp.json                       # lstr-reasoning reasoning MCP server registry
    ├── rules/                         # Always-on behavioral contract
    │   ├── 00-identity.md             #   alwaysApply: true
    │   ├── 10-reasoning-framework.md  #   alwaysApply: true
    │   ├── 20-output-spec.md          #   alwaysApply: true
    │   ├── 30-compliance.md           #   alwaysApply: true
    │   └── 40-evolution-policy.md     #   intelligent (description-driven)
    ├── skills/                        # On-demand capability modules
    │   ├── self-discover/SKILL.md
    │   ├── metacognitive-plan/SKILL.md
    │   ├── self-test/SKILL.md
    │   └── self-evolve/SKILL.md
    ├── commands/                      # Slash-command entry points
    │   ├── discover.md
    │   ├── reflect.md
    │   └── audit.md
    ├── hooks.json                     # Deterministic guardrails + acceptance gate
    └── memory/
        └── evolution-ledger.md        # Git-tracked mirror of self-evolution state
```

## Requirements

- **TRAE IDE** with **SOLO Mode** enabled. See [SOLO mode overview](https://docs.trae.ai/ide/solo-mode).
- Network reachability to the reasoning MCP server `https://reasoning.lstr.one/mcp` (the `metacognitive-plan` skill depends on it).
- The global skill **`lstr-reasoning-framework`** installed under `~/.trae/skills/` — it supplies the exact tool-argument contract for the reasoning chain. See the [skill source](https://github.com/florentin-one-cloud/mcp).
- `python3` on the system PATH (used by the `PreToolUse` and `Stop` hooks).

> **Note:** This is a declarative-configuration project. There is no build step, no test suite, and no runtime binary. Verification is performed by TRAE IDE itself: skills appear under **Settings → Skills & Commands**, hooks log to the **Output → Agent Hooks** panel, and MCP tools appear after the server is connected.

## Installation

### Option A — Use as a project template

1. Clone the repository:

   ```bash
   git clone https://github.com/florentin-one/LSTR-solo-harness
   cp -R LSTR-solo-harness/.trae .
   cp LSTR-solo-harness/AGENTS.md .
   ```

2. Open the folder as a project in TRAE IDE and switch to **SOLO Mode**.
3. Confirm the MCP server is connected (Settings → MCP) and enable **Rules**, **Skills**, and **Hooks** as prompted.

### Option B — Drop into an existing project

Copy the `.trae/` directory and `AGENTS.md` into your existing project root, then open it in TRAE IDE SOLO Mode.

```bash
cp -R LSTR-solo-harness/.trae /path/to/your/project/
cp LSTR-solo-harness/AGENTS.md /path/to/your/project/
```

> TRAE reads project-scoped configuration from `$PROJECT_FOLDER/.trae/` and `$PROJECT_FOLDER/AGENTS.md`.

## Usage

Three slash commands drive the harness from the chat input box:

| Command | Skill | Purpose |
| --- | --- | --- |
| `/discover` | `self-discover` | Print a zero-shot capability inventory (skills, MCP tools, rules, memory) with confidence per capability |
| `/plan` | `metacognitive-plan` | Run the 6-step reasoning chain and emit a decision-complete plan against verifiable constraints |
| `/reflect` | `self-evolve` | Reflect on the just-completed task and distil reusable rules into the evolution ledger |
| `/audit` | — | Validate the newest reasoning record against the output contract and report gaps |

Skills also load automatically: the agent reads each `SKILL.md` description and invokes the relevant skill when the task matches (e.g. "test this", "check robustness" → `self-test`).

## The Reasoning Framework

The cognitive core is the **LSTR 6-step Zero-Shot Chain-of-Thought**, executed over the reasoning MCP server.

| Step | Tool | Purpose |
| --- | --- | --- |
| 1 | `metacognitiveMonitoring` | Assess knowledge boundaries and calibrate confidence |
| 2 | `sequentialthinking` | Decompose the problem into ≥3 ordered thoughts |
| 3 | `collaborativeReasoning` | Evaluate ≥2 genuinely divergent perspectives |
| 4 | `scientificMethod` + `structuredArgumentation` | Hypothesis testing + thesis/antithesis/synthesis |
| 5 | `constraintSolver` | Validate numeric constraints only |
| 6 | `metacognitiveMonitoring` (reflection) | Finalize confidence and structure the output |

The complexity gate in `10-reasoning-framework.md` decides whether the chain runs: it is mandatory only when **two or more** of (multi-step inference, competing constraints, non-trivial uncertainty, material consequence, explicit demand) are satisfied. Confidence is **never invented** — it is sourced from `metacognitiveMonitoring.overallConfidence`.

> `narrative-planner` is deliberately EXCLUDED from the chain: its schema generates three-act story outlines and MUST NOT structure analytical output.

## Self-Evolution Policy

Supervision types (from GDPevo):

- **`reflect`** — *default*. Distil from the task's own scores (reinforcement-learning-like). Transfers most robustly across domains.
- **`fewshot`** — *opt-in*. Distil from gold answers (supervised-like). Reserved for canonical examples; overfits the source domain.
- **`self`** — *unsupervised*. Exploratory tasks only.

A rule is distilled only when the task passed its **deterministic grader** (never an LLM-as-judge); the rule is atomic; provenance (task + score + supervision) is recorded in `.trae/memory/evolution-ledger.md`; and the change is additive and reversible. Cost (tokens, turns, tool calls) is a first-class metric alongside accuracy.

## Compliance & Data Sovereignty

- **GDPR Art. 28** — reasoning MCP calls transmit content to `https://reasoning.lstr.one/mcp`. Secrets and personal data are stripped from tool arguments absent a processing agreement covering that endpoint.
- **EU AI Act** — no prohibited practices (Art. 5); no sensitive-attribute inference without a lawful basis.
- **German data sovereignty** — on-device/local storage by default; local skills and rules are preferred over remote services where functionally equivalent; cross-border egress is flagged explicitly.

## Research Foundation

The harness transfers six mechanisms from peer-reviewed work (papers supplied in the design phase):

| Paper | Transferred mechanism |
| --- | --- |
| **GDPevo** (arXiv:2608.03764) | Agent self-evolution under `reflect`/`fewshot`/`self` supervision; deterministic rule-based graders; cost as a first-class metric |
| **OccuBench** (arXiv:2604.10866) | Language Environment Simulation; three-class fault injection (explicit / implicit / mixed) for robustness |
| **DeepPlanning** (arXiv:2601.18137) | Local vs global constrained optimization; verifiable constraints; backtracking under resource limits |
| **Qwen-AgentWorld** (arXiv:2606.24597) | Mental simulation of candidate plans before committing ("simulation as a thinking pattern parallel to reflection") |
| **WebWorld** (arXiv:2602.14721) | Lightweight trajectory memory via the evolution ledger |
| **Confident Decoding** (arXiv:2606.21906) | Do not over-trust the final output; gate on calibrated intermediate confidence |

Full mapping and rationale: [`docs/harness-design.md`](docs/harness-design.md).

## Roadmap

- [ ] Install as **global** harness (`~/.trae/`) so it applies across all projects.
- [ ] Add a `PostToolUse` hook to auto-append execution traces to the evolution ledger.
- [ ] Ship a `self-test` grader library with pre-written deterministic checkers for common task types.
- [ ] Add a `Notification` hook for completion alerts.

## Contributing

Contributions follow the harness's own discipline:

1. Every change to rules/skills MUST carry provenance and be additive (no destructive evolution).
2. Grading changes MUST be deterministic, not LLM-as-judge.
3. No secrets or personal data in any committed artifact.

Open an issue or pull request on GitHub. For substantive reasoning changes, run `/plan` first and attach the reasoning record.

## License

[MIT](LICENSE) © 2026 [Florentin One](https://florentin-one.de/), founded by Florentin Sakwiset (Hannover, Germany).

---

*LSTR Solo Harness is a Florentin One project. "LSTR" is the reasoning identity; "TRAE" and "SOLO" are trademarks of their respective owners. TRAE product naming is referenced for platform compatibility only.*
