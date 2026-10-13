# Evolution Ledger

Git-tracked audit mirror of the harness's self-evolution state. The canonical runtime memory lives in `~/.trae/memory/projects/{project_path}/project_memory.md` (TRAE native memory, local-only). This file is the cross-device, version-controlled mirror and the provenance record.

## Schema

Each entry MUST carry: `timestamp`, `task_slug`, `supervision_type`, `grader_score`, `distilled_rule`, `provenance`, `status`.

- `supervision_type` ∈ `reflect` | `fewshot` | `self`
- `status` ∈ `proposed` | `accepted` | `rejected`
- `grader_score` ∈ 0.0–1.0, sourced from the deterministic grader (never LLM-as-judge)

## Governance

- Only distil after a task has passed its deterministic grader.
- `reflect` (score-based) is the default supervision; `fewshot` (gold-answer) is reserved for canonical examples; `self` (unsupervised) for exploratory tasks.
- Every evolved artifact MUST carry provenance (source task + score).
- No secrets or personal data in this file (GDPR Art. 28).

## Entries

| Timestamp | Task | Supervision | Score | Distilled Rule | Provenance | Status |
| --- | --- | --- | --- | --- | --- | --- |
| 2026-10-13 | `reasoning-framework-portal-remap-v0.2.4` | `reflect` | 1.00 (6/6 rules held) | Document an MCP transport by the runtime's actual `server_name`, and keep that name as the single remap point; do NOT hardcode a documented default that the host does not expose. | Source task: edit of `LSTR-1/skills/reasoning-framework` to v0.2.4 (stale documented portal name remapped to `mcp_mcp-portal`). Deterministic grader: 6/6 — `version: 0.2.4` present; no stale portal name in `SKILL.md`/`references/tool-args.md`; `mcp_mcp-portal` present in both; `reasoning_` prefix documented; identity (`DeepSeek-V4-Pro`) preserved. Cost: ~6 turns, ~11 tool calls. NOT auto-committed. | proposed |
