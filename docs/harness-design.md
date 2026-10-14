# LSTR Harness (Vertical AI Integration) — Architecture

Production-ready, zero-shot, self-discovering, metacognitive, self-evolving AI-agent harness for **LSTR One Alpha**, positioned on **Vertical AI Integration**.
Harness version: `2026.10.14-alpha.1` (ISO 8601 CalVer, alpha).

## 1. Purpose & Scope

The harness layers LSTR's native primitives into one coherent, auditable control loop. It is **configuration, not code**: LSTR's built-in agent is the runtime; the harness supplies identity, reasoning discipline, deterministic guardrails, and a self-evolution policy.

The four required properties, defined precisely:

| Property | Definition | Mechanism |
| --- | --- | --- |
| **Zero-shot** | Operates from introspective capability discovery at session start; no warm-up, fine-tuning, or training | `self-discover` + `SessionStart` hook |
| **Self-discovering** | Builds a runtime inventory of its own skills, MCP tools, rules, and memory | `self-discover` skill; MCP `tools/list` |
| **Metacognitive** | Calibrates knowledge boundaries and confidence, reasons over verifiable constraints, gates its own output | LSTR 6-step chain; `10-reasoning-framework.md` rule; global `reasoning-framework` skill; `Stop` hook |
| **Self-evolving** | Reflects on task outcomes and distils reusable atomic rules into persistent state | `self-evolve` skill + evolution ledger |

## 2. The Metacognitive Loop

Six stages, each mapped to a LSTR primitive and a transferred paper mechanism.

| Stage | Action | LSTR primitive | Paper mechanism |
| --- | --- | --- | --- |
| 1. **Discover** | Capability inventory | `self-discover` skill; MCP `tools/list`; `SessionStart` hook | Zero-shot introspection |
| 2. **Plan** | 6-step reasoning + constraint planning + mental simulation | `10-reasoning-framework.md` rule; global `reasoning-framework` skill; MCP toolchain | DeepPlanning (local vs global constraints), Qwen-AgentWorld (simulation-as-thinking) |
| 3. **Act** | Guarded tool execution | `PreToolUse` hook; MCP `tools/call` | OccuBench (assume silent degradation) |
| 4. **Verify** | Fault-injection self-test, deterministic grading | `self-test` skill | OccuBench (fault injection), GDPevo (deterministic graders) |
| 5. **Evolve** | Reflection → distillation | `self-evolve` skill; `/reflect`; memory | GDPevo (reflect/fewshot/self supervision) |
| 6. **Record** | Audit trail + acceptance gate | `Stop` hook; reasoning records; `/audit` | Confident Decoding (don't trust final output unconditionally) |

## 3. Layer Map

Layering by context cost (from always-on to on-demand to deterministic):

```text
rules (always-on)  →  skills (on-demand)  →  MCP (toolchain)  →  hooks (deterministic)  →  commands (entry)  →  memory (persistent state)
```

- **Rules** — always-on behavioral contract; loaded in full at chat start.
- **Skills** — on-demand capability modules; loaded only when triggered (reduces token consumption).
- **MCP** — the reasoning toolchain, served through the Florentin One Enterprise MCP Portal (`server_name: "mcp_mcp-portal"`), and any domain tools.
- **Hooks** — deterministic shell commands at lifecycle points; not subject to the model's judgment.
- **Commands** — human entry points (`/discover`, `/reflect`, `/audit`).
- **Memory** — persistent self-evolution state (local-only native memory + git-tracked ledger).

## 4. Reasoning Chain Spec

The LSTR 6-step chain is the runtime's cognitive core. It executes over the Florentin One Enterprise MCP Portal (`server_name: "mcp_mcp-portal"`) using Cloudflare Code Mode: a mandatory Step 0 resolves tool names via `portal_codemode_search` (`codemode.tools()`); each step then runs via `portal_codemode_execute` (`codemode.<tool>(args)`) with parameters nested inside `args`; at most 4 upstream calls run concurrently. Full argument contract lives in the global `reasoning-framework` skill; the Plan stage is served directly by the always-on `10-reasoning-framework.md` rule and the global `reasoning-framework` skill.

| Step | Code Mode tool | Required args |
| --- | --- | --- |
| 0. Tool Discovery (MANDATORY) | `portal_codemode_search` | `code` (JS async arrow fn) |
| 1. Metacognitive Assessment | `reasoning_metacognitiveMonitoring` | `task`, `stage`, `overallConfidence`, `uncertaintyAreas`, `recommendedApproach`, `monitoringId`, `iteration`, `nextAssessmentNeeded` |
| 2. Problem Decomposition | `reasoning_sequentialthinking` | `thought`, `nextThoughtNeeded`, `thoughtNumber`, `totalThoughts` |
| 3. Multi-Perspective Analysis | `reasoning_collaborativeReasoning` | `topic`, `personas`, `contributions`, `stage`, `activePersonaId`, `sessionId`, `iteration`, `nextContributionNeeded` |
| 4a. Hypothesis Testing | `reasoning_scientificMethod` | `stage`, `inquiryId`, `iteration`, `nextStageNeeded` |
| 4b. Premise Validation | `reasoning_structuredArgumentation` | `claim`, `premises`, `conclusion`, `argumentType`, `confidence`, `nextArgumentNeeded` |
| 5. Constraint Validation | `reasoning_constraintSolver` | `variables`, `constraints` (numeric only) |

**EXCLUDE** `reasoning_narrativePlanner` (three-act story tool, not analytical output structuring).

## 5. Self-Evolution Policy

Supervision types (from GDPevo):

- `reflect` — **default**. Distil from the task's own scores (RL-like). Transfers most robustly across domains (GDPevo RQ2).
- `fewshot` — opt-in. Distil from gold answers (SFT-like). Reserved for canonical examples; overfits source domain.
- `self` — unsupervised. Exploratory tasks only.

Distillation gate: a rule is distilled only after its task passed the deterministic grader; the rule is atomic; provenance (task + score + supervision) is recorded; the change is additive and reversible. Never destructive or irreversible without approval.

Cost (tokens, turns, tool calls) is a first-class metric alongside accuracy — a useful evolution improves success without unbounded resource growth.

## 6. Compliance

- **GDPR Art. 28** — MCP calls transmit content to `https://mcp.api.florentin-one.de/mcp`; secrets and PII MUST be stripped from tool arguments absent a processing agreement. Secrets EXCLUDED from hooks, memory, ledger, and reasoning records.
- **EU AI Act** — no prohibited practices (Art. 5); no sensitive-attribute inference without a lawful basis.
- **German data sovereignty** — default on-device/local storage; prefer local skills and rules over remote services where equivalent; flag cross-border egress.

## 7. Failure Modes & Mitigations

| Failure mode | Mitigation |
| --- | --- |
| MCP unreachable | Retry with backoff (max 3); then HALT and offer an unaudited, explicitly-labelled direct answer |
| Hook infinite loop | `Stop` hook sets a bounded loop limit; acceptance gate fails open to a diagnostic message |
| Evolution degradation | Grader pass is a hard precondition; `reflect` (not `fewshot`) is default to avoid overfitting; every distillation is provenance-tracked and reversible |
| Context bloat | Skills load on demand; rules are lean; memory is summarized, not verbose |
| Silent environment degradation | `self-test` injects implicit faults (truncated/missing fields), which are the hardest class to detect (OccuBench) |
| Over-trusting final output | Confidence gate — the final answer is validated against `metacognitiveMonitoring.overallConfidence`; the `Stop` hook blocks spec-incomplete output (Confident Decoding principle) |

## 8. File Inventory

```text
trae-harness/
├── AGENTS.md                          # portable identity + behavioral contract
├── docs/harness-design.md             # this document
└── .trae/
    ├── mcp.json                       # Florentin One Enterprise MCP Portal registry
    ├── rules/                         # always-on behavioral contract
    │   ├── 00-identity.md             # alwaysApply
    │   ├── 10-reasoning-framework.md  # alwaysApply
    │   ├── 20-output-spec.md          # alwaysApply
    │   ├── 30-compliance.md           # alwaysApply
    │   └── 40-evolution-policy.md     # intelligent (description-driven)
    ├── skills/                        # on-demand capability modules
    │   ├── self-discover/SKILL.md
    │   ├── self-test/SKILL.md
    │   └── self-evolve/SKILL.md
    ├── commands/                      # slash-command entry points
    │   ├── discover.md
    │   ├── reflect.md
    │   └── audit.md
    ├── hooks.json                     # deterministic guardrails + acceptance gate
    └── memory/
        └── evolution-ledger.md        # git-tracked evolution mirror
```

## 9. Vertical AI Integration

The harness is positioned on **vertical AI** — AI systems tailored to a specific industry, process, or value-chain segment — following the BMFTR funding directive *"Richtlinie zur Förderung von Leitprojekten für die Nutzung von KI in Wertschöpfungsketten"* (Bundesanzeiger, 31.07.2026).

- **Vertical vs. horizontal:** BMFTR distinguishes *industrielle, domänenspezifische KI* (deep domain knowledge; specialised terminology, industrial datasets and formats; tailored to a vertical value chain) from horizontal, general-purpose LLM chatbots (broad applicability, but limited accuracy and precision in specialised fields such as medicine or mechanical engineering).
- **Programme context:** part of the Hightech-Agenda Deutschland (HTAD); complementary EU instruments named by BMFTR are the 8ra initiative, AI Factories/Gigafactories, the Apply-AI strategy, and IPCEI-AI.
- **Priority sectors:** industrial manufacturing & machinery, mobility & automotive, chemistry & cleantech, medtech, agriculture, ICT.
- **Methods:** the directive assesses Physics-Informed Neural Networks, RAG + Knowledge Graphs, Agentic Workflows, Federated Learning, Model Compression/Edge AI, Data Augmentation, Transfer Learning, MLOps, Explainable AI, formal verification, adversarial robustness, automated red teaming, reliability/out-of-distribution checks, Human-in-the-Loop, Data Provenance & Lineage, and standards-compatible test procedures positively.
- **Sovereignty:** local/edge processing reduces dependence on external providers and lowers data-protection and IT-security risk; funded results MUST be used only in Germany, the EEA, or Switzerland.
- **Exclusions:** pure data-centre hardware investment; generalist LLMs without a sector focus; AI hardware; AI in HR, marketing, customer support; predictive maintenance; civil security; robotic care systems.

The user-facing explanation lives in [`../README.md`](../README.md) → "Vertical AI — AI in Value Chains".

> **Source:** BMFTR, Bekanntmachung, Bundesanzeiger vom 31.07.2026 — <https://www.bmftr.bund.de/SharedDocs/Bekanntmachungen/DE/2026/07/2026-07-31-bekanntmachung-ki-wertschoepfungsketten.html>
