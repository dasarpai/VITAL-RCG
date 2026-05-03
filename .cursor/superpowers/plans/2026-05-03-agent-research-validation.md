# Research & Validation Agent — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Convert VITAL from a rich framework into **pragmatic, testable claims**: measurement quality, predictive lead time, calibration behaviour, and ethical participation—with artefacts researchers can extend and engineers can implement.

**Architecture:** A **hypothesis registry** drives **pilot protocols** and **analysis pipelines**. All definitions reference stable IDs that the Platform Agent will mirror in schemas. Outputs are **versioned reports** (CSV summaries + Markdown conclusions), not vague narratives.

**Tech Stack:** Markdown + CSV + optional Python 3.11+ in `research/` for reproducible stats (pandas only if needed; start with scipy-less notebooks optional). No proprietary data in repo—synthetic fixtures only.

**Agent charter:** The Research Agent owns **what we claim**, **how we measure**, and **whether evidence supports tightening thresholds**—not production SLAs or enterprise procurement.

---

## Coordination

| Provides to Enterprise Agent | Provides to Platform Agent |
|------------------------------|----------------------------|
| KPI definitions, stop rules for pilots | Minimum viable metrics API fields, evaluation datasets (synthetic) |
| Validated wording risks for “not sure” handling | Recommended defaults for dashboard uncertainty bands |

---

## File map

| Path | Responsibility |
|------|----------------|
| `docs/research/README.md` | How to contribute; ethics boundary |
| `docs/research/hypothesis-registry.v1.md` | Numbered falsifiable hypotheses |
| `docs/research/pilot-data-dictionary.v1.md` | Column meanings for exports |
| `docs/research/analysis-plan-core-functions.md` | Primary endpoints + precursors |
| `research/fixtures/synthetic_org_monthly.v1.csv` | Toy dataset for pipeline tests |
| `research/scripts/compute_participation_metrics.py` | CLI: participation + missingness |
| `research/scripts/compute_lead_time_proxy.py` | CLI: proxy metric (documented limitation) |
| `research/tests/test_participation_metrics.py` | Unit tests for CLI logic |
| `agents/research-validation/SYSTEM.md` | Executor agent boundaries |

---

### Task 1: Research spine and hypothesis registry

**Files:**
- Create: `docs/research/README.md`
- Create: `docs/research/hypothesis-registry.v1.md`

- [ ] **Step 1:** Create `docs/research/README.md` with: purpose, data ethics rule (**no individual exports**), how pilots submit **aggregated** CSV per `pilot-data-dictionary`, contribution workflow.

- [ ] **Step 2:** Create `hypothesis-registry.v1.md` with **three** starter hypotheses (verbatim):

```markdown
# Hypothesis Registry v1

## H1 — Participation sustainability
**Statement:** Aggregate participation remains ≥ target for 8 consecutive weeks after intro communications.
**Operationalisation:** Weekly active responding teams / eligible teams (aggregate).
**Failure mode:** Sharp decline after week 3 suggests trust or workload issue.

## H2 — Information-gap signal validity
**Statement:** Increased aggregate rate of “not sure” correlates with subsequent drops in self-reported information-flow health scores (same function), lag ≤ 4 weeks.
**Operationalisation:** Lagged correlation on monthly aggregates only.
**Failure mode:** No relationship after two pilot cycles → rethink question framing.

## H3 — Early-warning lead time (proxy)
**Statement:** At least one named correlation rule fires **before** a defined adverse KPI movement in ≥ X% of historical months in pilot archive (threshold X set per pilot).
**Operationalisation:** Compare first alert timestamp to KPI threshold crossing (both predefined).
**Limitation:** Proxy only; causal claims require controlled designs.

## Governance
Hypotheses are versioned. Changes require PR + changelog entry.
```

- [ ] **Step 3:** Commit

```bash
git add docs/research/README.md docs/research/hypothesis-registry.v1.md
git commit -m "docs(research): add spine and hypothesis registry v1"
```

---

### Task 2: Pilot data dictionary v1

**Files:**
- Create: `docs/research/pilot-data-dictionary.v1.md`

- [ ] **Step 1:** Document columns (copy table):

```markdown
# Pilot Data Dictionary v1

All files must be **aggregate**; `n` must respect minimum cell policy.

## monthly_org_metrics.csv
| column | type | description |
|--------|------|-------------|
| period | YYYY-MM | Month |
| function_id | string | Stable function identifier (see Platform schema) |
| tier | enum | CF,E,FF,RF |
| participation_rate | float | 0–1 |
| not_sure_rate | float | 0–1 |
| signal_stress_index | float | Normalised composite (define per pilot) |
| n_teams | int | Number of teams in aggregate |

## alerts.csv
| column | type | description |
|--------|------|-------------|
| fired_at | ISO8601 | Timestamp |
| rule_id | string | Named rule id |
| severity | int | Pilot-defined |
| context_summary | string | Non-identifying |

## kpis.csv (lagging)
| column | type | description |
|--------|------|-------------|
| period | YYYY-MM | Month |
| kpi_id | string | e.g. revenue_quality_proxy |
| value | float | |
```

- [ ] **Step 2:** Commit

```bash
git add docs/research/pilot-data-dictionary.v1.md
git commit -m "docs(research): add pilot aggregate data dictionary v1"
```

---

### Task 3: Analysis plan — core endpoints

**Files:**
- Create: `docs/research/analysis-plan-core-functions.md`

- [ ] **Step 1:** Write plan specifying primary endpoints for first pilot cycle:

```markdown
# Analysis Plan — Core Functions Pilot

## Primary endpoints (Month 3 and Month 6)
1. Participation trajectory (H1)
2. Not-sure ↔ information-flow lagged association (H2)
3. Lead-time proxy distribution (H3)

## Pre-registration discipline
- KPI thresholds for “adverse movement” frozen before viewing alert outcomes where feasible.
- Document any deviations in `docs/research/changelog.md` (create when first deviation occurs).

## Reporting
- Produce `reports/YYYY-MM-DD-pilot-summary.md` summarising accept/reject per hypothesis at alpha=0.05 **only if** sample size justification exists; otherwise descriptive only.
```

- [ ] **Step 2:** Commit

```bash
git add docs/research/analysis-plan-core-functions.md
git commit -m "docs(research): add core functions analysis plan"
```

---

### Task 4: Synthetic fixture + participation CLI + tests

**Files:**
- Create: `research/fixtures/synthetic_org_monthly.v1.csv`
- Create: `research/scripts/compute_participation_metrics.py`
- Create: `research/tests/test_participation_metrics.py`

- [ ] **Step 1:** Create CSV fixture:

```csv
period,function_id,tier,participation_rate,not_sure_rate,signal_stress_index,n_teams
2026-01,CF_FINANCE,CF,0.72,0.08,0.31,12
2026-02,CF_FINANCE,CF,0.68,0.11,0.35,12
2026-03,CF_FINANCE,CF,0.61,0.14,0.40,12
```

- [ ] **Step 2:** Write failing test first (`research/tests/test_participation_metrics.py`):

```python
import csv
from pathlib import Path

from compute_participation_metrics import load_monthly_org_metrics, participation_trend_slope


def test_participation_trend_slope_declining():
    fixture = Path(__file__).resolve().parent.parent / "fixtures" / "synthetic_org_monthly.v1.csv"
    rows = load_monthly_org_metrics(fixture)
    slope = participation_trend_slope(rows, function_id="CF_FINANCE")
    assert slope < 0
```

- [ ] **Step 3:** Run test expecting failure:

```bash
cd research
python -m pytest tests/test_participation_metrics.py::test_participation_trend_slope_declining -v
```

Expected: import error or collection error until implementation exists.

- [ ] **Step 4:** Implement `research/scripts/compute_participation_metrics.py`:

```python
"""Aggregate-only participation analytics helpers."""
from __future__ import annotations

import csv
from pathlib import Path
from statistics import linear_regression
from typing import List, Dict


def load_monthly_org_metrics(path: Path) -> List[Dict[str, str]]:
    with path.open(newline="", encoding="utf-8") as f:
        return list(csv.DictReader(f))


def participation_trend_slope(rows: List[Dict[str, str]], function_id: str) -> float:
    subset = [r for r in rows if r["function_id"] == function_id]
    subset.sort(key=lambda r: r["period"])
    if len(subset) < 2:
        raise ValueError("need at least two periods")
    xs = list(range(len(subset)))
    ys = [float(r["participation_rate"]) for r in subset]
    slope, _intercept = linear_regression(xs, ys)
    return float(slope)


def main() -> None:
    print("use as library; CLI extensions in later task")


if __name__ == "__main__":
    main()
```

- [ ] **Step 5:** Fix import path for tests — use package layout or `sys.path`. Minimal fix: run pytest with `PYTHONPATH=research/scripts`:

```bash
cd research
set PYTHONPATH=scripts
python -m pytest tests/test_participation_metrics.py::test_participation_trend_slope_declining -v
```

Expected: PASS

- [ ] **Step 6:** Commit

```bash
git add research/fixtures/synthetic_org_monthly.v1.csv research/scripts/compute_participation_metrics.py research/tests/test_participation_metrics.py
git commit -m "feat(research): participation metrics helper with synthetic fixture"
```

---

### Task 5: Lead-time proxy CLI (documented limitation)

**Files:**
- Create: `research/scripts/compute_lead_time_proxy.py`
- Create: `research/tests/test_lead_time_proxy.py`

- [ ] **Step 1:** Write failing test `research/tests/test_lead_time_proxy.py`:

```python
from pathlib import Path

from compute_lead_time_proxy import first_alert_before_kpi_crossing_months


def test_first_alert_before_kpi_crossing_detected():
    root = Path(__file__).resolve().parent.parent / "fixtures"
    months = first_alert_before_kpi_crossing_months(
        alerts_csv=root / "synthetic_alerts.v1.csv",
        kpis_csv=root / "synthetic_kpis.v1.csv",
        kpi_id="revenue_quality_proxy",
        kpi_threshold=0.85,
    )
    assert months >= 1
```

- [ ] **Step 2:** Create fixtures `research/fixtures/synthetic_alerts.v1.csv`:

```csv
fired_at,rule_id,severity,context_summary
2026-01-15T10:00:00Z,CASH_ATTRITION_SPIRAL,3,aggregate-context-a
```

And `research/fixtures/synthetic_kpis.v1.csv`:

```csv
period,kpi_id,value
2026-01,revenue_quality_proxy,0.90
2026-02,revenue_quality_proxy,0.84
```

- [ ] **Step 3:** Implement `compute_lead_time_proxy.py`:

```python
from __future__ import annotations

import csv
from datetime import datetime
from pathlib import Path
from typing import Optional


def _parse_month_from_iso(ts: str) -> str:
    return datetime.fromisoformat(ts.replace("Z", "+00:00")).strftime("%Y-%m")


def first_alert_before_kpi_crossing_months(
    alerts_csv: Path,
    kpis_csv: Path,
    kpi_id: str,
    kpi_threshold: float,
) -> int:
    """Return whole months between first alert month and first KPI breach month.

    Limitation: uses monthly granularity only; not causal inference.
    """
    with alerts_csv.open(newline="", encoding="utf-8") as f:
        alerts = list(csv.DictReader(f))
    if not alerts:
        return 0
    first_alert_month = min(_parse_month_from_iso(a["fired_at"]) for a in alerts)

    with kpis_csv.open(newline="", encoding="utf-8") as f:
        kpis = [r for r in csv.DictReader(f) if r["kpi_id"] == kpi_id]
    kpis.sort(key=lambda r: r["period"])
    breach_month: Optional[str] = None
    for row in kpis:
        if float(row["value"]) < kpi_threshold:
            breach_month = row["period"]
            break
    if breach_month is None:
        return 0

    y1, m1 = map(int, first_alert_month.split("-"))
    y2, m2 = map(int, breach_month.split("-"))
    return (y2 - y1) * 12 + (m2 - m1)
```

- [ ] **Step 4:** Run:

```bash
cd research
set PYTHONPATH=scripts
python -m pytest tests/test_lead_time_proxy.py -v
```

Expected: PASS

- [ ] **Step 5:** Commit

```bash
git add research/fixtures/synthetic_alerts.v1.csv research/fixtures/synthetic_kpis.v1.csv research/scripts/compute_lead_time_proxy.py research/tests/test_lead_time_proxy.py
git commit -m "feat(research): lead-time proxy helper with fixtures"
```

---

### Task 6: Agent system prompt

**Files:**
- Create: `agents/research-validation/SYSTEM.md`

- [ ] **Step 1:**

```markdown
You are the Research & Validation Agent executor.

Mission: Pre-register analyses, compute aggregate metrics, and write cautious interpretations.

Hard rules:
- Never request individual-level survey rows.
- Label causal claims as unsupported unless design supports them.
- Every statistic references the hypothesis ID from `docs/research/hypothesis-registry.v1.md`.

Deliverables: Markdown reports in `reports/` with explicit limitations section.
```

- [ ] **Step 2:** Commit

```bash
git add agents/research-validation/SYSTEM.md
git commit -m "docs(research): add research agent system prompt"
```

---

## Self-review

1. **Coverage:** Hypotheses, dictionary, analysis plan, code paths for metrics — covered.
2. **Placeholders:** Numeric alpha thresholds flagged as conditional on sample size; pilot parameter X referenced honestly as pilot-specific.
3. **Consistency:** `function_id` and `rule_id` align with naming style Platform Agent can mirror.

---

## Success metrics

- Each hypothesis has an operationalisation tied to aggregate CSV fields; at least two automated tests green in CI (when CI added by Platform Agent).

---

## Execution handoff

**Plan complete:** `docs/superpowers/plans/2026-05-03-agent-research-validation.md`

**Two execution options:**

1. **Subagent-driven** — superpowers:subagent-driven-development  
2. **Inline** — superpowers:executing-plans  

**Which approach?**
