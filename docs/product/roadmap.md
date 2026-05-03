# VITAL product roadmap (implementation-facing)

This file is **canonical product context** for engineers and AI assistants. If you opened this repo in a **new Cursor project** (e.g. under WSL) and prior chat sessions are unavailable, **start here** before implementing features.

**Explicitly out of scope for now:** the formal **research / validation programme** (hypothesis registry, statistical proof of lead time, fleet of pilots). The product should **not** claim predictive validity until that programme exists and has evidence.

---

## Honest positioning (until pilots produce evidence)

Ship an **operational multi-tenant hub**: ingest organisational signals (manual → CSV → APIs), enforce **aggregate-only** policy, evaluate rules, route alerts, and present **exec-ready** and **developer-ready** surfaces.

Label demos honestly: **framework mechanics + configuration**, not “proven 30–60 day early warning” for arbitrary customers.

---

## Audiences and what they need

| Audience | Jobs to be done | Product surfaces |
|----------|-----------------|------------------|
| **CEO / exec sponsor** | See correlated drift; act early; trust it is not surveillance | Executive dashboard, alert summaries, period comparisons, plain-language briefs |
| **CTO / platform owner** | Security, tenancy, uptime, data boundaries | SSO-ready admin, RBAC, audit logs, retention, environments, backups/runbooks |
| **Developers** | Integrate safely | REST APIs, sandbox tenant, OpenAPI, connector patterns, staging Postgres |

---

## Technical backbone (single spine)

1. **PostgreSQL** — system of record: tenants, users/roles, calibration/policy, aggregate snapshots, alerts, audit events, connector metadata, ingestion jobs. **Staging** tables land raw-ish batches before promoted aggregates (when connectors exist).
2. **Backend API** — authentication/authorisation, **tenant isolation on every query**, idempotent ingest, read APIs for dashboards and integrations.
3. **Web applications** (one shell or separate apps)
   - **Exec experience:** read-mostly, polished UI.
   - **Admin / calibration:** scope, thresholds, routing; approvals later.
   - **Developer portal (minimum):** API keys, docs link, sample payloads.

Use **SQLite only for local developer convenience**, not as the long-term product database.

---

## Phased delivery

### Phase 0 — Product skeleton

- Tenancy: organisation + environment (e.g. demo / staging / production).
- Auth baseline (start simple; document SSO roadmap).
- Postgres schema: aggregates, alerts (stub acceptable), audit log, users/roles.
- Seed script + **demo tenant** with fixture data (no external APIs required).
- **Executive dashboard v1:** tier/function overview; participation/stress-style indicators — credible CEO demo.
- **Developer:** publish OpenAPI; document JSON shapes; batch ingest endpoint and read endpoint for dashboard data.

**Exit criteria:** One deployable environment; CEO walkthrough on realistic data; developer can POST aggregates and see the UI reflect them.

### Phase 1 — Organisation-specific configuration

- **Scope:** which functions/tiers are active per organisation.
- **Calibration:** thresholds and routing (forms/tables first; optional YAML export).
- **Policy enforcement in API:** aggregates-only, minimum cell size — not UI-only checks.
- **Connector registry:** connector types registered even if first implementation is CSV upload.

**Exit criteria:** Two tenants in one deployment with **different** scope/thresholds without code changes.

### Phase 2 — Staging ingest and connector pattern

- Pipeline: staging → normalisation → aggregate snapshots (clear lineage).
- **CSV / file upload** as first-class connector for early adopters.
- Per-tenant API keys, basic rate limits; optional webhooks for new alerts.
- Ops UI: freshness, last sync, error queue.

**Exit criteria:** CTO-visible operational health; developers integrate without coupling to executive UI internals.

### Phase 3 — Enterprise hardening

- SSO (SAML/OIDC); SCIM optional.
- Fine-grained RBAC (exec read vs admin configure vs developer ingest).
- Approval workflow for calibration changes (two-person rule, aligned with enterprise governance docs when present).
- Aggregate-only export packs for leadership materials.
- Exec UI polish: narrative templates, responsive summary, print/PDF-friendly views.

**Exit criteria:** Passes a reasonable security review for a cautious CTO.

### Phase 4 — Real system connectors

- Implement connectors **one at a time** (priority order decided per market).
- Same Postgres model: staging → aggregates; connectors as **plugins** behind one interface.

**Exit criteria:** Less reliance on manual CSV; research programme may resume separately when data exists.

---

## UI / design note

**Beautiful, cohesive UI** is required for CEO/CTO adoption and should follow **one design system** from Phase 0 so work is not duplicated. The reference implementation may start with a thin table; treat that as a **spike**, not the final exec experience.

---

## Relationship to other repo docs

- Framework philosophy and monitoring model: root **`README.md`** and **`docs/`** narrative conversions (e.g. framework description).
- Optional detailed agent execution plans (if present): **`docs/superpowers/plans/`** — align implementation tasks with **this roadmap**; research-heavy tasks stay parked unless revived.

---

## Working in WSL or a new clone

- Paths in plans may use Windows-style history; this repo under WSL uses normal Linux paths — content is the same.
- Prefer **branch + PR** workflow so context lives in commits and descriptions, not only in IDE chat.
- When asking an AI assistant to implement: point it at **`docs/product/roadmap.md`** and the current phase.

---

## Quick glossary

| Term | Meaning |
|------|--------|
| **Tenant** | One adopting organisation (strict data isolation). |
| **Aggregate snapshot** | Monthly (or periodic) roll-up per function/tier with policy-safe cell sizes. |
| **Calibration** | Org-specific thresholds and routing. |
| **Connector** | Ingest path (CSV, API, etc.) into staging then aggregates. |
