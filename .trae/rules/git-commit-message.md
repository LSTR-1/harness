---
alwaysApply: true
scene: git_message
standardID: GCM-LEDGER-1.0.0
scope: Every commit authored, generated, amended, or imported into this repository, by human or machine.
universality: Team-agnostic (lean MLOps, operational units, platform teams), platform-agnostic (cloud, on-prem, edge), data-type-agnostic (code, configuration, model artifacts, datasets, schemas, infrastructure-as-code, documentation), VCS-agnostic (Git, Mercurial, Subversion, Jujutsu).
---

# Git Commit Message

## 0. Prime Directive

A commit message IS the authoritative, off-platform evolution-ledger entry for the change. It MUST be a self-contained, standalone record: a future reader with access to NO external system — no issue tracker, no chat log, no cloud console, no CI dashboard, and no original author — MUST fully understand **what** changed, **why**, its **impact**, how it was **verified**, how to **reverse** it, and where it fits in the evolution of the system.

Non-negotiable properties:

- **Self-contained** — MUST NOT require external context to be interpretable. External references are supplementary, never load-bearing.
- **Dual-audience** — MUST be readable by a non-technical stakeholder AND a technical engineer.
- **Durable** — MUST remain interpretable after complete team turnover and across decades: absolute ISO-8601 dates, acronyms defined on first use, no transient codenames or rottable references as the sole context.
- **Auditable** — MUST support end-to-end audit, production troubleshooting, code-review validation, and logical traceability of system evolution.

## 1. Two-Layer Readability Contract

Every non-trivial entry MUST carry both layers:

| Layer | Block | Audience | Requirement |
| --- | --- | --- | --- |
| Plain-language | `[WHY]` | Non-technical stakeholders, operators, auditors, future maintainers | 1–3 sentences; no unexplained acronyms; states the operational/business reason and the user-visible effect. |
| Technical | `[WHAT]` | Engineers, reviewers, SRE/MLOps | Precise: components, modules, functions, data types, interfaces, and formats touched, and the mechanism of effect. |

A message readable by only one layer is non-conformant.

## 2. Ledger Entry Schema

**Header (line 1):** `<type>(<scope>): <summary>`

**Body blocks**, in fixed order, each introduced by an uppercase tag in square brackets at line start (machine-parseable):

```text
[LEDGER]   <ledger-id> | <ISO-8601-UTC timestamp> | <tier: T0|T1|T2>
[WHY]      plain-language rationale (non-technical layer)
[WHAT]     technical description of the change (technical layer)
[IMPACT]   affected systems / data / platforms; blast radius; compatibility
[EVIDENCE] verification performed (tests, metrics, manual checks)
[RISK]     failure modes introduced or mitigated
[ROLLBACK] reversal procedure
[TRACE]    provenance: parent revision, related commits, reasoning record, review reference
```

- `<type>` ∈ `feat | fix | perf | refactor | docs | test | build | ci | chore | data | model | config | infra | security | revert`.
- `<scope>` = affected component or domain, lowercase, hyphenated.
- `<summary>` = imperative mood, no trailing period, jargon-light; MUST be understandable without the body.
- `<ledger-id>` = stable, unique, immutable key; for a VCS that assigns a revision id, use it; otherwise use `<UTC-date>-<scope>-<seq>`.
- Block order is fixed; empty optional blocks MUST be omitted, never emitted blank.

## 3. Field Definitions

- **`[LEDGER]`** — identity of the record. Carries the ledger id, the absolute ISO-8601 UTC timestamp (`YYYY-MM-DDThh:mm:ssZ`), and the risk tier. No relative dates ("today", "yesterday") are permitted.
- **`[WHY]`** — the plain-language reason and the user-visible effect. Answers "why does this exist" for a non-technical reader.
- **`[WHAT]`** — the technical change: components, modules, functions, data types, formats, interfaces. Answers "what exactly changed" for an engineer.
- **`[IMPACT]`** — systems, data, and platforms affected; blast radius; backward/forward compatibility; SLA or throughput effects. MUST state if the change is irreversible.
- **`[EVIDENCE]`** — how the change was verified: automated test identifiers, metrics observed, manual checks. MUST NOT claim verification that was not performed.
- **`[RISK]`** — failure modes introduced or mitigated, with the trigger condition.
- **`[ROLLBACK]`** — the exact reversal procedure (revert command, migration down-step, artifact restore).
- **`[TRACE]`** — provenance: parent revision, related commits, the reasoning record path, and the review reference. Supplementary — the message MUST still be interpretable if every reference is dead.

## 4. Risk Tiers

The tier governs which blocks are mandatory. Tier is chosen by the highest-severity trigger present.

| Tier | Triggers | Mandatory blocks |
| --- | --- | --- |
| **T0** trivial | docs, comments, formatting; no behavioural change | `LEDGER`, `WHY`, `IMPACT`, `TRACE` |
| **T1** standard | routine code or configuration change; bounded blast radius | T0 + `WHAT`, `EVIDENCE` |
| **T2** high-impact | schema, dataset, model artifact, migration, infrastructure, security, or irreversible / wide-blast-radius change | T1 + `RISK`, `ROLLBACK` |

`WHY`, `IMPACT`, and `TRACE` form the **audit spine** and are mandatory at every tier, including T0.

## 5. VCS Rendering Rules

The schema above is VCS-neutral text. Rendering per host:

- **Git** — line 1 is the subject; keep it ≤50 characters (hard limit 72); a single blank line separates subject from body; wrap body lines at 72; each block tag begins a line.
- **Jujutsu** — identical to Git (description text).
- **Mercurial** — the whole entry is the changeset description; line 1 is the summary; blocks follow unchanged.
- **Subversion** — a single log message with no subject/body split; line 1 is the summary; blocks follow on subsequent lines. Do NOT rely on `svn:keywords` or external properties for ledger content.

## 6. Traceability & Audit

- Every ledger entry MUST be reconstructable in isolation: an auditor reading the commit alone MUST be able to state the change, its reason, its impact, its verification, and its reversal path.
- `[TRACE]` MUST link to the persisted reasoning record where one exists (e.g. `reasoning-records/<timestamp>-<slug>.md`) and to the parent revision, enabling a logical chain of evolution across commits.
- Ledger entries are append-only history: a superseding commit MUST reference the superseded one in `[TRACE]`, never rewrite history to erase a prior record.

## 7. Production Troubleshooting

During an incident, the on-call responder MUST be able to answer from the message alone:

1. What did this change do? → `[WHAT]`
2. Why does it exist? → `[WHY]`
3. What did it touch? → `[IMPACT]`
4. How was it proven? → `[EVIDENCE]`
5. What can fail, and how? → `[RISK]`
6. How do I undo it now? → `[ROLLBACK]`

A message that omits any of these at T1/T2 fails the troubleshooting gate.

## 8. Code Review Validation

A reviewer MUST verify, deterministically:

- The tier matches the change's blast radius.
- Every block mandatory for that tier is present and non-empty.
- `[EVIDENCE]` names concrete verification, not a claim.
- `[ROLLBACK]` (T2) is executable, not a description of intent.
- The message is self-contained — removing every external link leaves it interpretable.

## 9. Compliance & Secrets

- **GDPR Art. 28 / secrets hygiene** — the commit message MUST NOT contain credentials, tokens, keys, connection strings, personal data, or client-confidential material. Commit history is persistent and effectively immutable.
- **EU AI Act Art. 5** — no prohibited practice; no inference of sensitive attributes (race, health, political opinion) without a lawful basis. `[TRACE]` MUST NOT carry personal data.
- **Data sovereignty** — where a change moves data across borders, `[IMPACT]` MUST flag the cross-border egress explicitly.

## 10. Conformance Checklist (deterministic)

A commit message conforms when ALL hold:

- [ ] Header matches `<type>(<scope>): <summary>` with a valid `<type>`.
- [ ] `[LEDGER]` present with an ISO-8601 UTC timestamp and a tier in {T0, T1, T2}.
- [ ] All blocks mandatory for the declared tier are present and non-empty.
- [ ] Block order follows the schema; no blank mandatory blocks.
- [ ] Subject ≤72 characters; summary is imperative and jargon-light.
- [ ] No secrets, credentials, or personal data.
- [ ] Every acronym is defined on first use; all dates are absolute.
- [ ] The message is interpretable with all external references removed.

## 11. Worked Examples

### T0 — documentation-only change

```text
docs(readme): clarify local setup steps

[LEDGER] 2026-10-14-readme-001 | 2026-10-14T09:12:00Z | T0
[WHY] New contributors reported the setup steps were unclear; this
      makes local onboarding self-explanatory.
[IMPACT] Documentation only; no runtime behaviour changes.
[TRACE] parent: a1b2c3d; review: PR-142
```

### T2 — model artifact and schema change

```text
feat(pricing-model): retrain with corrected FX feature pipeline

[LEDGER] 2026-10-14-pricing-017 | 2026-10-14T11:40:00Z | T2
[WHY] The FX feature used a stale conversion source, producing a
      systematic bias in quoted prices for non-EUR orders.
[WHAT] Replaced fx_rate source in feature_pipeline.py; retrained
       pricing-model v2.4.0 (digest sha256:9f2c...); bumped the
       inference schema to v3 (added fx_rate_source field).
[IMPACT] Inference service and downstream quote API; requires
         coordinated rollout; NOT backward compatible with schema v2.
[EVIDENCE] offline MAPE 4.1% -> 2.3% on holdout; 312 unit tests pass;
           canary 5% for 24h with no SLA breach.
[RISK] If consumers do not upgrade to schema v3, requests fail closed
       with a validation error.
[ROLLBACK] Redeploy pricing-model v2.3.0 and revert schema to v2 via
           migration down-step m_2026_10_14_003.
[TRACE] parent: d4e5f6a; reasoning-records/2026-10-14T035618Z-pricing.md; review: PR-147
```

## 12. Failure Modes & Mitigations

| Failure mode | Mitigation |
| --- | --- |
| Message references a ticket/chat that later disappears | The audit spine (`WHY`/`IMPACT`/`TRACE`) carries the essential context inline; external links are supplementary only. |
| Tier misclassification hides mandatory blocks | The deterministic checklist (Section 10) rejects a missing block for the declared tier; a reviewer MUST re-tier if blast radius is understated. |
| Schema reduced to empty boilerplate | The checklist rejects empty mandatory blocks; `[EVIDENCE]` MUST name concrete verification, not a claim. |
| Git-centric tags break on SVN/Mercurial | The schema is VCS-neutral text with explicit per-VCS rendering (Section 5). |
| Secret or PII leaked into history | Section 9 prohibition; the pre-commit / commit-msg hook MUST scan for credential and PII patterns before the commit is accepted. |
| Decade-scale jargon rot | Absolute ISO-8601 dates, acronyms defined on first use, no transient codenames as sole context. |
