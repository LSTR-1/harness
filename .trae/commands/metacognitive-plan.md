---
name: metacognitive-plan
description: Orchestrates the LSTR 6-step metacognitive reasoning chain and produces a decision-complete plan against verifiable constraints. Invoke for architecture decisions, multi-constraint trade-offs, incident triage, or any task requiring auditable reasoning. Trigger phrases "plan this", "/plan" (delegating to the reasoning chain), "apply the LSTR framework", "reason through this", or any query where constraints compete and a plan is needed.
version: 2026.10.14-alpha.1
---

# metacognitive-plan

Runs the LSTR 6-step chain (delegating the exact tool-argument contract to the global `reasoning-framework` skill), then adds constraint extraction and mental simulation before committing to a plan. The chain executes over the Florentin One Enterprise MCP Portal (`server_name: "mcp_mcp-portal"`) using Cloudflare Code Mode: Step 0 resolves tool names via `portal_codemode_search`, each step runs via `portal_codemode_execute`, and parameters MUST be nested inside `args`.

## When to Use

- Multi-step inference, competing constraints, non-trivial uncertainty, or material consequence (≥2 complexity-gate criteria).
- Any query where an auditable reasoning trail is required.

## Instructions

1. **Complexity gate** — evaluate against `10-reasoning-framework.md`. If <2 criteria, skip the chain and state "framework not warranted".
2. **Run the 6-step chain** via the reasoning MCP family (Florentin One Enterprise MCP Portal, `server_name: "mcp_mcp-portal"`), per `reasoning-framework`:
   - `reasoning_metacognitiveMonitoring` (knowledge-assessment) → `reasoning_sequentialthinking` (≥3 thoughts) → `reasoning_collaborativeReasoning` (≥2 divergent personas) → `reasoning_scientificMethod` + `reasoning_structuredArgumentation` (thesis + objection + synthesis) → `reasoning_constraintSolver` (numeric only).
   - EXCLUDE `reasoning_narrativePlanner`.
3. **Constraint extraction (DeepPlanning)** — classify every constraint as `local` (step-scoped) or `global` (cross-subtask budget / dependency). Record each constraint's verifiability: deterministic, rule-checkable, or qualitative.
4. **Mental simulation (Qwen-AgentWorld)** — before committing, rehearse the top candidate plan against the environment model; predict where it fails under resource limits or silent degradation; backtrack if a global constraint is violated.
5. **Emit a decision-complete plan** — ordered, dependency-aware steps, each with an exit condition. No calendar dates; use predecessor relationships.

## Output Contract

- Decision-complete plan (executor makes no additional choices).
- Local vs global constraint list, each with verifiability label.
- Reasoning record persisted per `reasoning-framework` contract.
- Confidence from `metacognitiveMonitoring.overallConfidence`.

## Constraints

- Do NOT fabricate a reasoning trace. A failed step is logged `failed`.
- Do NOT encode qualitative constraints as `constraintSolver` booleans.
