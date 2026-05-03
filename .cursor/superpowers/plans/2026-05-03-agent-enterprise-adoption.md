# Enterprise Adoption Agent — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make VITAL adoptable inside enterprises through repeatable governance (roles, approvals, audit), pilot playbooks, and optional AI assistants that **draft** configuration—never bypass policy.

**Architecture:** Treat “enterprise practicality” as a **workflow + accountability layer** on top of the VITAL framework docs. Agents produce **artifacts** (YAML policies, runbooks, RACI matrices, calibration proposals) that humans approve; a deterministic policy engine (even if initially manual checklists in Git) enforces separation of duties.

**Tech Stack:** Markdown/Git for canonical artifacts; optional later: OPA/Rego or equivalent only if you outgrow YAML gates. AI layer: prompts + tool schemas bound to allowed file edits (no silent prod changes).

**Agent charter (summary):** The Enterprise Adoption Agent owns **who may change what**, **how pilots start/end**, **how alerts route**, and **evidence packs for executives**—not correlation mathematics or dashboard UI code.

---

## Coordination with other agents

| Needs from Research Agent | Needs from Platform Agent |
|---------------------------|---------------------------|
| Validated pilot KPI definitions (lead time, false-alert tolerance, participation metrics) | Stable IDs for functions, signals, rules (schema/version), export formats for audit |

---

## File map (this workstream creates)

| Path | Responsibility |
|------|----------------|
| `docs/enterprise/README.md` | Entry point for adoption materials |
| `docs/enterprise/raci-template.md` | Roles vs framework activities |
| `docs/enterprise/pilot-protocol.md` | 90-day pilot steps aligned with research protocol |
| `docs/enterprise/policy/policy-schema.v1.yaml` | Machine-readable gates (draft → approve → publish) |
| `docs/enterprise/policy/example-tenant-policy.v1.yaml` | Reference tenant |
| `docs/enterprise/runbooks/incident-response-drill.md` | Tabletop exercise for alert flooding |
| `docs/enterprise/ai-assistants/configuration-copilot-spec.md` | Allowed tools, forbidden actions, approval UX |
| `agents/enterprise-adoption/SYSTEM.md` | Boundaries + escalation for an executor agent |

---

### Task 1: Adoption documentation spine

**Files:**
- Create: `docs/enterprise/README.md`
- Create: `docs/enterprise/raci-template.md`

- [ ] **Step 1:** Create `docs/enterprise/README.md` with sections: Purpose, Prerequisites (legal/ER sign-off), Pilot overview link, Policy files link, AI assistant scope link, FAQ (“Is this surveillance?”).

- [ ] **Step 2:** Create `docs/enterprise/raci-template.md` using this table (copy verbatim, then customise names per org):

```markdown
# VITAL Pilot RACI (template)

| Activity | Sponsor (CxO) | VITAL Owner | Data/IT | Legal/ER | People/HR | Participants |
|----------|---------------|-------------|---------|----------|-----------|--------------|
| Approve pilot charter | A | R | C | C | C | I |
| Approve threshold calibration | A | R | C | I | C | I |
| Approve alert routing | A | R | R | I | C | I |
| Operate connectors (technical) | I | A | R | I | I | I |
| Communicate “trust architecture” | A | R | I | C | R | I |
| Review aggregated dashboards only | A | R | I | I | C | I |

Legend: R Responsible, A Accountable, C Consulted, I Informed.
```

- [ ] **Step 3:** Commit

```bash
git add docs/enterprise/README.md docs/enterprise/raci-template.md
git commit -m "docs(enterprise): add adoption spine and RACI template"
```

---

### Task 2: Policy schema v1 (human-gated changes)

**Files:**
- Create: `docs/enterprise/policy/policy-schema.v1.yaml`
- Create: `docs/enterprise/policy/example-tenant-policy.v1.yaml`

- [ ] **Step 1:** Write `policy-schema.v1.yaml` documenting allowed keys (comments in YAML):

```yaml
# policy-schema.v1.yaml — documents keys for example-tenant-policy.v1.yaml
# version: semver of policy bundle
# tenant_id: stable org identifier
# roles: named principals mapped to enterprise identities (filled at deploy time)
# approvals_required: list of change categories requiring two-person rule
# connectors_allowed: allowlist of connector types
# data_access: maximum visibility envelopes (aggregates-only, minimum cell sizes, etc.)
# retention: log and raw response retention (if any)
version: "1.0.0"
tenant_id: "REPLACE_ME"
roles: {}
approvals_required: []
connectors_allowed: []
data_access: {}
retention: {}
```

- [ ] **Step 2:** Write `example-tenant-policy.v1.yaml` with concrete example values:

```yaml
version: "1.0.0"
tenant_id: "example-msme-001"
roles:
  vital_owner: ["group:vital-admins"]
  security_reviewer: ["group:security"]
  executive_reader: ["group:cxo-readonly"]
approvals_required:
  - change_category: "threshold_calibration"
    rule: "two_person"
    approvers_any_of: ["role:security_reviewer", "role:vital_owner"]
  - change_category: "new_connector"
    rule: "two_person"
    approvers_any_of: ["role:security_reviewer"]
connectors_allowed:
  - type: "csv_hr_headcount_monthly"
  - type: "survey_vital_sdq_webhook"
data_access:
  aggregates_only: true
  minimum_cell_size: 5
  forbid_individual_response_export: true
retention:
  audit_logs_days: 365
  raw_survey_payload_days: 0
```

- [ ] **Step 3:** Commit

```bash
git add docs/enterprise/policy/policy-schema.v1.yaml docs/enterprise/policy/example-tenant-policy.v1.yaml
git commit -m "docs(enterprise): add v1 policy schema and example tenant"
```

---

### Task 3: Pilot protocol (enterprise-operable)

**Files:**
- Create: `docs/enterprise/pilot-protocol.md`

- [ ] **Step 1:** Create `docs/enterprise/pilot-protocol.md` containing these phases (fill stakeholder names at pilot kickoff):

```markdown
# VITAL 90-Day Pilot Protocol (Enterprise)

## Phase 0 — Charter (Week 0)
- Signed pilot charter: scope (functions in-scope), success metrics, stop conditions.
- Legal/ER acknowledgement of aggregation-only visibility model.

## Phase 1 — Instrumentation (Weeks 1–3)
- Deploy allowed connectors per policy allowlist.
- Baseline participation rate target (set with Research Agent metrics).

## Phase 2 — Signal stabilisation (Weeks 4–6)
- No threshold tightening without two-person approval (per policy).
- Weekly ops review: connector freshness, missing data reasons.

## Phase 3 — Alert rehearsal (Weeks 7–8)
- Run tabletop using `runbooks/incident-response-drill.md`.
- Track false-positive burden proxy: alerts requiring >X hours/week executive attention.

## Phase 4 — Decision (Weeks 9–12)
- Continue / expand / stop based on pre-committed KPIs.
```

- [ ] **Step 2:** Commit

```bash
git add docs/enterprise/pilot-protocol.md
git commit -m "docs(enterprise): add 90-day pilot protocol"
```

---

### Task 4: Incident response drill runbook

**Files:**
- Create: `docs/enterprise/runbooks/incident-response-drill.md`

- [ ] **Step 1:** Write runbook with scripted scenario:

```markdown
# Incident Response Drill — Correlated Alert Flood

## Scenario
Three correlated alerts fire within 48 hours across CF and E tiers.

## Roles
- Incident commander: VITAL Owner
- Comms: Sponsor (CxO delegate)
- Technical: Data/IT

## Steps (timed)
1. T+0: Acknowledge alerts in audit log; freeze calibration changes.
2. T+4h: Classify: operational drift vs connector fault vs true organisational stress.
3. T+24h: Decision tree:
   - If connector fault: disable connector (two-person approval), communicate downtime.
   - If stress signal: activate remediation playbook (HR/finance/ops as per RACI).
4. T+48h: Post-incident review: update routing thresholds only via approved change request.

## Success criteria
- No individual-level data accessed for investigation.
- Written outcome stored with incident ID.
```

- [ ] **Step 2:** Commit

```bash
git add docs/enterprise/runbooks/incident-response-drill.md
git commit -m "docs(enterprise): add correlated alert drill runbook"
```

---

### Task 5: Configuration Copilot — agent specification

**Files:**
- Create: `docs/enterprise/ai-assistants/configuration-copilot-spec.md`
- Create: `agents/enterprise-adoption/SYSTEM.md`

- [ ] **Step 1:** Write `configuration-copilot-spec.md`:

```markdown
# Configuration Copilot — Specification

## Allowed
- Propose edits to calibration YAML (thresholds, routing) as **pull requests** or **draft patches**.
- Summarise policy violations against `example-tenant-policy.v1.yaml`.
- Generate executive briefing from **aggregated** exports provided by Platform Agent.

## Forbidden
- Requesting or processing individual survey rows.
- Applying changes without two-person approval markers in audit metadata.
- Expanding connector allowlist without security reviewer role.

## Tools (recommended bindings)
- Read: policy YAML, pilot protocol, RACI
- Write: only `drafts/` branch paths or PR descriptions

## Output format
Every proposal must include: Risk section, Rollback section, Approvers list.
```

- [ ] **Step 2:** Write `agents/enterprise-adoption/SYSTEM.md`:

```markdown
You are the Enterprise Adoption Agent executor.

Mission: Produce governance artifacts and pilot readiness materials for VITAL.

Hard rules:
- Never advise circumventing aggregation-only or minimum-cell rules.
- Treat `docs/enterprise/policy/example-tenant-policy.v1.yaml` as the reference constraint object unless tenant supplies another approved file.
- When suggesting AI automation, separate drafting from approval.

Escalate to human: legal interpretations, union contexts, law-enforcement deployments.
```

- [ ] **Step 3:** Commit

```bash
git add docs/enterprise/ai-assistants/configuration-copilot-spec.md agents/enterprise-adoption/SYSTEM.md
git commit -m "docs(enterprise): specify configuration copilot and agent system prompt"
```

---

## Self-review (plan vs intent)

1. **Spec coverage:** Governance, pilots, RACI, policy gates, drill, AI bounds — each has tasks.
2. **Placeholder scan:** No TBD steps; org-specific names marked as kickoff fill-ins only in templates.
3. **Consistency:** Policy schema keys match example tenant file.

---

## Success metrics (agent/workstream)

- Pilot charter completion rate; time-to-first-approved-calibration; drill completion; audit trail presence for every threshold change.

---

## Execution handoff

**Plan complete:** `docs/superpowers/plans/2026-05-03-agent-enterprise-adoption.md`

**Two execution options:**

1. **Subagent-driven** — dispatch per task with superpowers:subagent-driven-development  
2. **Inline** — superpowers:executing-plans with checkpoints after Tasks 2 and 5  

**Which approach?**
