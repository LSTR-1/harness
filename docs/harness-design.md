# LSTR Solo Harness — Architecture

Production-ready, zero-shot, self-discovering, metacognitive, self-evolving AI-agent harness for **TRAE IDE SOLO Mode**.

## 1. Purpose & Scope

The harness layers TRAE SOLO Mode's native primitives into one coherent, auditable control loop. It is **configuration, not code**: TRAE SOLO's built-in agent is the runtime; the harness supplies identity, reasoning discipline, deterministic guardrails, and a self-evolution policy.

The four required properties, defined precisely:

| Property | Definition | Mechanism |
| --- | --- | --- |
| **Zero-shot** | Operates from introspective capability discovery at session start; no warm-up, fine-tuning, or training | `self-discover` + `SessionStart` hook |
| **Self-discovering** | Builds a runtime inventory of its own skills, MCP tools, rules, and memory | `self-discover` skill; MCP `tools/list` |
| **Metacognitive** | Calibrates knowledge boundaries and confidence, reasons over verifiable constraints, gates its own output | LSTR 6-step chain; `metacognitive-plan`; `Stop` hook |
| **Self-evolving** | Reflects on task outcomes and distils reusable atomic rules into persistent state | `self-evolve` skill + evolution ledger |

## 2. The Metacognitive Loop

Six stages, each mapped to a TRAE primitive and a transferred paper mechanism.

| Stage | Action | TRAE primitive | Paper mechanism |
| --- | --- | --- | --- |
| 1. **Discover** | Capability inventory | `self-discover` skill; MCP `tools/list`; `SessionStart` hook | Zero-shot introspection |
| 2. **Plan** | 6-step reasoning + constraint planning + mental simulation | `metacognitive-plan` skill; MCP toolchain; `/plan` | DeepPlanning (local vs global constraints), Qwen-AgentWorld (simulation-as-thinking) |
| 3. **Act** | Guarded tool execution | `PreToolUse` hook; MCP `tools/call` | OccuBench (assume silent degradation) |
| 4. **Verify** | Fault-injection self-test, deterministic grading | `self-test` skill | OccuBench (fault injection), GDPevo (deterministic graders) |
| 5. **Evolve** | Reflection → distillation | `self-evolve` skill; `/reflect`; memory | GDPevo (reflect/fewshot/self supervision) |
| 6. **Record** | Audit trail + acceptance gate | `Stop` hook; reasoning records; `/audit` | Confident Decoding (don't trust final output unconditionally) |

## 3. Layer Map

Layering by context cost (from always-on to on-demand to deterministic):

```
rules (always-on)  →  skills (on-demand)  →  MCP (toolchain)  →  hooks (deterministic)  →  commands (entry)  →  memory (persistent state)
```

- **Rules** — always-on behavioral contract; loaded in full at chat start.
- **Skills** — on-demand capability modules; loaded only when triggered (reduces token consumption).
- **MCP** — the reasoning toolchain (lstr-reasoning) and any domain tools.
- **Hooks** — deterministic shell commands at lifecycle points; not subject to the model's judgment.
- **Commands** — human entry points (`/discover`, `/reflect`, `/audit`).
- **Memory** — persistent self-evolution state (local-only native memory + git-tracked ledger).

## 4. Reasoning Chain Spec

The LSTR 6-step chain is the runtime's cognitive core. Full argument contract lives in the global `lstr-reasoning-framework` skill; the project `metacognitive-plan` skill delegates to it.

| Step | Tool | Required args |
| --- | --- | --- |
| 1. Metacognitive Assessment | `metacognitiveMonitoring` | `task`, `stage`, `overallConfidence`, `uncertaintyAreas`, `recommendedApproach`, `monitoringId`, `iteration`, `nextAssessmentNeeded` |
| 2. Problem Decomposition | `sequentialthinking` | `thought`, `nextThoughtNeeded`, `thoughtNumber`, `totalThoughts` |
| 3. Multi-Perspective Analysis | `collaborativeReasoning` | `topic`, `personas`, `contributions`, `stage`, `activePersonaId`, `sessionId`, `iteration`, `nextContributionNeeded` |
| 4a. Hypothesis Testing | `scientificMethod` | `stage`, `inquiryId`, `iteration`, `nextStageNeeded` |
| 4b. Premise Validation | `structuredArgumentation` | `claim`, `premises`, `conclusion`, `argumentType`, `confidence`, `nextArgumentNeeded` |
| 5. Constraint Validation | `constraintSolver` | `variables`, `constraints` (numeric only) |

**EXCLUDE** `narrative-planner` (three-act story tool, not analytical output structuring).

## 5. Self-Evolution Policy

Supervision types (from GDPevo):

- `reflect` — **default**. Distil from the task's own scores (RL-like). Transfers most robustly across domains (GDPevo RQ2).
- `fewshot` — opt-in. Distil from gold answers (SFT-like). Reserved for canonical examples; overfits source domain.
- `self` — unsupervised. Exploratory tasks only.

Distillation gate: a rule is distilled only after its task passed the deterministic grader; the rule is atomic; provenance (task + score + supervision) is recorded; the change is additive and reversible. Never destructive or irreversible without approval.

Cost (tokens, turns, tool calls) is a first-class metric alongside accuracy — a useful evolution improves success without unbounded resource growth.

## 6. Compliance

- **GDPR Art. 28** — MCP calls transmit content to `https://mcp.beta.lstr.one/mcp`; secrets and PII MUST be stripped from tool arguments absent a processing agreement. Secrets EXCLUDED from hooks, memory, ledger, and reasoning records.
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

```
trae-harness/
├── AGENTS.md                          # portable identity + behavioral contract
├── docs/harness-design.md             # this document
└── .trae/
    ├── mcp.json                       # lstr-reasoning MCP server registry
    ├── rules/                         # always-on behavioral contract
    │   ├── 00-identity.md             # alwaysApply
    │   ├── 10-reasoning-framework.md  # alwaysApply
    │   ├── 20-output-spec.md          # alwaysApply
    │   ├── 30-compliance.md           # alwaysApply
    │   └── 40-evolution-policy.md     # intelligent (description-driven)
    ├── skills/                        # on-demand capability modules
    │   ├── self-discover/SKILL.md
    │   ├── metacognitive-plan/SKILL.md
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
