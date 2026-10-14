# LSTR Harness (Vertical AI Integration)

> Production-ready, **zero-shot**, **self-discovering**, **metacognitive**, **self-evolving** AI-agent harness for [LSTR ONE](https://lstr.one/) **Alpha**, positioned on **Vertical AI Integration** — domain-specific AI along industrial value chains.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Platform: LSTR One](https://img.shields.io/badge/Platform-LSTR%20One-blue)](https://florentin-one.de/)
[![Version: 2026.10.14-alpha.1](https://img.shields.io/badge/Version-2026.10.13--alpha.1-orange)](https://github.com/LSTR-1/harness)

The harness is **configuration, not code**. It layers LSTR's native primitives — rules, skills, MCP, hooks, slash commands, and memory — into one coherent, auditable metacognitive control loop. LSTR's built-in agent is the runtime; this repository supplies the identity, reasoning discipline, deterministic guardrails, and self-evolution policy that make that runtime production-ready.

---

## Table of Contents

1. [The Four Capabilities](#the-four-capabilities)
2. [How It Works — The Six-Stage Loop](#how-it-works--the-six-stage-loop)
3. [Repository Structure](#repository-structure)
4. [Requirements](#requirements)
5. [Installation](#installation)
6. [Usage](#usage)
7. [The Reasoning Framework](#the-reasoning-framework)
8. [Vertical AI — AI in Value Chains](#vertical-ai--ai-in-value-chains)
9. [Self-Evolution Policy](#self-evolution-policy)
10. [Compliance & Data Sovereignty](#compliance--data-sovereignty)
11. [Research Foundation](#research-foundation)
12. [Roadmap](#roadmap)
13. [Contributing](#contributing)
14. [License](#license)

---

## The Four Capabilities

| Capability | Definition | Mechanism |
| --- | --- | --- |
| **Zero-shot** | Operates from introspective capability discovery at session start — no warm-up, fine-tuning, or training | `self-discover` skill + `SessionStart` hook |
| **Self-discovering** | Builds a runtime inventory of its own skills, MCP tools, rules, and memory | `self-discover` skill; MCP `tools/list` |
| **Metacognitive** | Calibrates knowledge boundaries and confidence, reasons over verifiable constraints, and gates its own output | LSTR 6-step reasoning chain; `Stop` acceptance hook |
| **Self-evolving** | Reflects on task outcomes and distils reusable atomic rules into persistent state | `self-evolve` skill + evolution ledger |

## How It Works — The Six-Stage Loop

Every non-trivial task proceeds through a deterministic loop. Each stage maps to a LSTR primitive and to a mechanism transferred from peer-reviewed research.

| Stage | Action | LSTR primitive | Research mechanism |
| --- | --- | --- | --- |
| 1. **Discover** | Build a capability inventory | `self-discover` skill; MCP `tools/list`; `SessionStart` hook | Zero-shot introspection |
| 2. **Plan** | 6-step reasoning + constraint planning + mental simulation | `10-reasoning-framework.md` rule; global `reasoning-framework` skill; MCP toolchain | DeepPlanning (local vs global constraints); Qwen-AgentWorld (simulation-as-thinking) |
| 3. **Act** | Guarded tool execution | `PreToolUse` hook; MCP `tools/call` | OccuBench (assume silent environment degradation) |
| 4. **Verify** | Fault-injection self-test with deterministic grading | `self-test` skill | OccuBench (fault injection); GDPevo (deterministic graders, not LLM-as-judge) |
| 5. **Evolve** | Reflection → distillation into skills/memory | `self-evolve` skill; `/reflect`; memory | GDPevo (reflect / fewshot / self supervision) |
| 6. **Record** | Audit trail + acceptance gate | `Stop` hook; reasoning records; `/audit` | Confident Decoding (do not over-trust the final output) |

## Repository Structure

```text
lstr-solo-harness/
├── AGENTS.md                          # Portable identity + behavioral contract (AGENTS.md-compatible IDEs)
├── README.md                          # This file
├── LICENSE                            # MIT
├── docs/
│   └── harness-design.md              # Full architecture document
└── .trae/
    ├── mcp.json                       # Florentin One Enterprise MCP Portal registry
    ├── rules/                         # Always-on behavioral contract
    │   ├── 00-identity.md             #   alwaysApply: true
    │   ├── 10-reasoning-framework.md  #   alwaysApply: true
    │   ├── 20-output-spec.md          #   alwaysApply: true
    │   ├── 30-compliance.md           #   alwaysApply: true
    │   └── 40-evolution-policy.md     #   intelligent (description-driven)
    ├── skills/                        # On-demand capability modules
    │   ├── self-discover/SKILL.md
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

- **LSTR IDE** with **SOLO Mode** enabled. See [SOLO mode overview](https://docs.trae.ai/ide/solo-mode).
- Network reachability to the reasoning MCP server `https://mcp.api.florentin-one.de/mcp` (the LSTR 6-step reasoning chain — mandated by the always-on `10-reasoning-framework.md` rule and executed via the global `reasoning-framework` skill — depends on it).
- The global skill **`reasoning-framework`** installed under `~/.trae/skills/` — it supplies the exact tool-argument contract for the reasoning chain. See the [skill source](https://github.com/LSTR-1/reasoning).
- `python3` on the system PATH (used by the `PreToolUse` and `Stop` hooks).

> **Note:** This is a declarative-configuration project. There is no build step, no test suite, and no runtime binary. Verification is performed by TRAE IDE itself: skills appear under **Settings → Skills & Commands**, hooks log to the **Output → Agent Hooks** panel, and MCP tools appear after the server is connected.

## Installation

### Option A — Use as a project template

1. Clone the repository:

   ```bash
   git clone https://github.com/LSTR-1/harness
   cp -R harness/.trae .
   cp harness/AGENTS.md .
   ```

2. Open the folder as a project in TRAE IDE and switch to **SOLO Mode**.
3. Confirm the MCP server is connected (Settings → MCP) and enable **Rules**, **Skills**, and **Hooks** as prompted.

### Option B — Drop into an existing project

Copy the `.trae/` directory and `AGENTS.md` into your existing project root, then open it in TRAE IDE SOLO Mode.

```bash
cp -R harness/.trae /path/to/your/project/
cp harness/AGENTS.md /path/to/your/project/
```

> TRAE reads project-scoped configuration from `$PROJECT_FOLDER/.trae/` and `$PROJECT_FOLDER/AGENTS.md`.

## Usage

Three slash commands — `/discover`, `/reflect`, and `/audit` — drive the harness from the chat input box:

| Command | Skill | Purpose |
| --- | --- | --- |
| `/discover` | `self-discover` | Print a zero-shot capability inventory (skills, MCP tools, rules, memory) with confidence per capability |
| `/reflect` | `self-evolve` | Reflect on the just-completed task and distil reusable rules into the evolution ledger |
| `/audit` | — | Validate the newest reasoning record against the output contract and report gaps |

Skills also load automatically: the agent reads each `SKILL.md` description and invokes the relevant skill when the task matches (e.g. "test this", "check robustness" → `self-test`).

## The Reasoning Framework

The cognitive core is the **LSTR 6-step Zero-Shot Chain-of-Thought**, executed over the **Florentin One Enterprise MCP Portal** (`server_name: "mcp_mcp-portal"`) using **Cloudflare Code Mode**.

A mandatory **Step 0** resolves the exact tool names and schemas via `portal_codemode_search` (`codemode.tools()`) before any execution. Each framework step then runs via `portal_codemode_execute` (`codemode.<tool>(args)`). Parameters MUST be nested inside `args`. At most 4 upstream tool calls run concurrently. Reasoning tool names carry the `reasoning_` prefix.

| Step | Code Mode tool | Purpose |
| --- | --- | --- |
| 0 | `portal_codemode_search` | Discover reasoning tool names and schemas (MANDATORY, once per session) |
| 1 | `reasoning_metacognitiveMonitoring` | Assess knowledge boundaries and calibrate confidence |
| 2 | `reasoning_sequentialthinking` | Decompose the problem into ≥3 ordered thoughts |
| 3 | `reasoning_collaborativeReasoning` | Evaluate ≥2 genuinely divergent perspectives |
| 4 | `reasoning_scientificMethod` + `reasoning_structuredArgumentation` | Hypothesis testing + thesis/antithesis/synthesis |
| 5 | `reasoning_constraintSolver` | Validate numeric constraints only |
| 6 | `reasoning_metacognitiveMonitoring` (reflection) | Finalize confidence and structure the output |

The complexity gate in `10-reasoning-framework.md` decides whether the chain runs: it is mandatory only when **two or more** of (multi-step inference, competing constraints, non-trivial uncertainty, material consequence, explicit demand) are satisfied. Confidence is **never invented** — it is sourced from `metacognitiveMonitoring.overallConfidence`.

> `reasoning_narrativePlanner` is deliberately EXCLUDED from the chain: its schema generates three-act story outlines and MUST NOT structure analytical output.

## Vertical AI — AI in Value Chains

The harness is positioned on **vertical AI**: AI systems tailored to a specific industry, process, or value-chain segment, as opposed to horizontal, general-purpose assistants. The framing follows the German Federal Ministry for Research, Technology and Space (BMFTR) funding directive *"Richtlinie zur Förderung von Leitprojekten für die Nutzung von KI in Wertschöpfungsketten"* (Bundesanzeiger, 31.07.2026).

### Domain-specific industrial AI vs. horizontal general-purpose LLMs

| Dimension | Domain-specific industrial AI (vertical) | General-purpose LLM chatbots (horizontal) |
| --- | --- | --- |
| Data | Domain corpora, industrial datasets and formats, specialised terminology | General, cross-domain corpora |
| Strength | Deep domain knowledge; reliable processing of domain-typical data; support in workflows | Broad applicability; text generation, image analysis, general Q&A |
| Limitation | Narrower applicability outside the target domain | Limited accuracy and precision in specialised fields such as medicine or mechanical engineering |
| Cost profile | Lower compute footprint; local/edge processing feasible | Higher compute and inference cost for comparable reliability |

BMFTR states the core argument directly: a strong focus on a single, currently widespread flavour — chatbots based on general large language models — is *not sufficient* to exploit AI's potential as a cross-cutting technology for pattern recognition and data analysis across many industries. The deficit stems substantially from a shortage of specialised, high-quality, purpose-updated data (specialised language, sector-specific legal frameworks, domain knowledge).

### Programme context

The directive sits under the **Hightech-Agenda Deutschland (HTAD)**, whose flagship measures span six priority key technologies; AI is one. BMFTR points to complementary EU-level instruments — the **8ra initiative**, the **AI Factories and Gigafactories**, the **Apply-AI strategy**, and in particular **IPCEI Artificial Intelligence (IPCEI-AI)** — as momentum to build on.

### Priority sectors

BMFTR targets sectors with a high value-added share in Germany:

- Industrial manufacturing and mechanical engineering (process optimisation, autonomous reconfiguration of production lines on product changeover, resilient supply-chain control, AI-based product and production engineering)
- Mobility and the automotive industry (specialised environmental perception, logistics optimisation)
- Chemical industry and cleantech (materials research, sustainable process control, AI-driven laboratory environments)
- Medical technology (AI-supported diagnostics)
- Agriculture (precision farming, intelligent nutrient management)
- Information and communications technology

### Method catalogue (BMFTR-positive approaches)

The directive states that using one or more of the following approaches is assessed positively:

- Physics-Informed Neural Networks — embed physical laws (thermodynamics, fluid mechanics) into models
- Retrieval-Augmented Generation and Knowledge Graphs — couple models in real time to structured domain knowledge and in-house knowledge bases
- Agentic Workflows — systems that plan multi-step workflows autonomously and interact with external databases or tools
- Federated Learning — cooperative training across distributed actors without sharing sensitive data
- Model Compression — run capable models directly on end devices; reduce cloud dependence; ensure real-time capability and energy efficiency
- Data Augmentation — enrich real data with synthetic data where data volume is scarce
- Transfer Learning — adapt existing models efficiently to specific industrial target tasks
- MLOps — automated training/testing/deployment pipelines with continuous monitoring (e.g. drift detection)
- Explainable AI — make algorithm decisions interpretable for domain experts; ease certification in regulated domains
- Formal verification — prove mathematically that models act within defined safety bounds
- Adversarial robustness — protect systems against targeted manipulation
- Automated red teaming — systematically identify weaknesses in domain-specific models
- Reliability checks — detect and signal inputs outside the training distribution instead of extrapolating
- Human-in-the-Loop — hand over to a human expert under high uncertainty in critical decisions
- Data Provenance & Lineage — make origin, quality and processing history of all training data auditable
- Standards-compatible test procedures — accelerate approval against existing industry norms

BMFTR is explicit that applying specific methods is not decisive by itself; what matters is a traceable justification that the chosen approaches fully address industrial requirements for reliability, safety, and certifiability within a deep vertical integration.

### Sovereignty, edge processing, and data protection

Because specialised systems often need substantially less compute, input data can be processed locally (**Edge AI**). This reduces dependence on external providers and lowers data-protection and IT-security risk. A remaining hurdle is the preparation of frequently unstructured and heterogeneous data; standards, quality assurance, and cooperative interoperability are prerequisites for transferable solutions. The directive further requires that funded results be used only in the Federal Republic of Germany, the EEA, or Switzerland.

### Explicit exclusions

The directive explicitly excludes: pure investment in data-centre hardware; development of generalist language models without a specific sector focus; development of AI hardware; and the use of AI in HR, marketing, or customer support, predictive maintenance, civil security, or robotic systems for care — because these are addressed by complementary national and European programmes.

> **Source:** BMFTR, *Bekanntmachung — Richtlinie zur Förderung von Leitprojekten für die Nutzung von KI in Wertschöpfungsketten*, Bundesanzeiger vom 31.07.2026 — <https://www.bmftr.bund.de/SharedDocs/Bekanntmachungen/DE/2026/07/2026-07-31-bekanntmachung-ki-wertschoepfungsketten.html>

## Self-Evolution Policy

Supervision types (from GDPevo):

- **`reflect`** — *default*. Distil from the task's own scores (reinforcement-learning-like). Transfers most robustly across domains.
- **`fewshot`** — *opt-in*. Distil from gold answers (supervised-like). Reserved for canonical examples; overfits the source domain.
- **`self`** — *unsupervised*. Exploratory tasks only.

A rule is distilled only when the task passed its **deterministic grader** (never an LLM-as-judge); the rule is atomic; provenance (task + score + supervision) is recorded in `.trae/memory/evolution-ledger.md`; and the change is additive and reversible. Cost (tokens, turns, tool calls) is a first-class metric alongside accuracy.

## Compliance & Data Sovereignty

- **GDPR Art. 28** — reasoning MCP calls transmit content to `https://mcp.api.florentin-one.de/mcp`. Secrets and personal data are stripped from tool arguments absent a processing agreement covering that endpoint.
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

Open an issue or pull request on GitHub. For substantive reasoning changes, run the LSTR 6-step reasoning chain — always-on `10-reasoning-framework.md` rule and global `reasoning-framework` skill — first and attach the reasoning record.

## License

[MIT](LICENSE) © 2026 [Florentin One](https://florentin-one.de/), founded by Florentin Sakwiset (Hannover, Germany).

---

*LSTR Harness (Vertical AI Integration) is a Florentin One project. "LSTR" is the reasoning identity; "TRAE" and "SOLO" are trademarks of their respective owners. TRAE product naming is referenced for platform compatibility only.*
