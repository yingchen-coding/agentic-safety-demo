> **Portfolio**: [Safety Memo](https://yingchen-coding.github.io/safety-memos/) · [when-rlhf-fails-quietly](https://github.com/yingchen-coding/when-rlhf-fails-quietly) · [agentic-misuse-benchmark](https://github.com/yingchen-coding/agentic-misuse-benchmark) · [agentic-safeguards-simulator](https://github.com/yingchen-coding/agentic-safeguards-simulator) · [safeguards-stress-tests](https://github.com/yingchen-coding/safeguards-stress-tests) · [scalable-safeguards-eval-pipeline](https://github.com/yingchen-coding/scalable-safeguards-eval-pipeline) · [model-safety-regression-suite](https://github.com/yingchen-coding/model-safety-regression-suite) · [agentic-safety-incident-lab](https://github.com/yingchen-coding/agentic-safety-incident-lab)

# Agentic Safety Demo

> A unified end-to-end demonstration of the closed-loop safety system: Stress Testing → Regression → Release Gate → Incident Replay.

## Motivation

Individual safety components are necessary but not sufficient. Production-grade safety requires a **closed-loop system** where:

1. **Discovery**: Red-teaming surfaces delayed failures
2. **Regression**: Failures become permanent tests
3. **Gating**: Release decisions are automated
4. **Learning**: Incidents feed back into the system

This demo walks through the complete loop in 5 minutes.

---

## 5-Minute End-to-End Demo

### Step 0: Setup

```bash
git clone https://github.com/yingchen-coding/agentic-safety-demo
cd agentic-safety-demo
pip install -r requirements.txt
```

Or run everything with one command:

```bash
make demo
```

---

### Step 1: Discover Delayed Failures (Stress Tests)

```bash
python scripts/step1_run_stress_tests.py --rollouts 50
```

**What happens:**
- Runs adaptive red-teaming against target model
- Discovers delayed policy erosion failures
- Generates failure distribution over turns

**Output:** `artifacts/stress_failures.json`

---

### Step 2: Generate Regression Tests from Failures

```bash
python scripts/step2_generate_regression.py --input artifacts/stress_failures.json
```

**What happens:**
- Converts discovered failures into structured regression tests
- Tags each test with failure taxonomy and severity
- Creates permanent test cases for CI/CD

**Output:** `artifacts/regression_tests.json`

---

### Step 3: Gate a Candidate Model Release

```bash
python scripts/step3_run_release_gate.py --baseline v1 --candidate v2
```

**What happens:**
- Runs regression suite against baseline and candidate
- Computes statistical significance of any regressions
- Produces OK / WARN / BLOCK verdict

**Output:** `artifacts/gate_report.html`, exit code (0=OK, 1=WARN, 2=BLOCK)

---

### Step 4: Replay a Production Incident

```bash
python scripts/step4_replay_incident.py --incident artifacts/incident_example.json
```

**What happens:**
- Replays a simulated production incident
- Labels root cause and estimates blast radius
- Emits new regression tests to prevent recurrence

**Output:** Incident summary, new tests added to regression suite

---

## Expected Demo Output

```
$ make demo

=== Step 1: Stress Testing ===
Running 50 adaptive rollouts...
Discovered 12 delayed failures
Saved to artifacts/stress_failures.json

=== Step 2: Regression Generation ===
Generated 8 regression tests from 12 failures
Saved to artifacts/regression_tests.json

=== Step 3: Release Gate ===
Running regression suite...
  Baseline (v1): 94.2% pass rate
  Candidate (v2): 87.5% pass rate
  Delta: -6.7% (p=0.003)
Release verdict: BLOCK
Report saved to artifacts/gate_report.html

=== Step 4: Incident Replay ===
Replaying incident INC-2024-001...
  Root cause: policy_erosion
  Blast radius: 1,247 affected conversations
  New regression tests: 2
Incident loop complete.

=== Demo Complete ===
Total time: 47 seconds
```

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    CLOSED-LOOP SAFETY SYSTEM                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────────────┐         ┌──────────────────┐             │
│   │  Stress Tests    │────────▶│  Regression Gen  │             │
│   │  (Discovery)     │         │  (Conversion)    │             │
│   └──────────────────┘         └────────┬─────────┘             │
│                                         │                        │
│                                         ▼                        │
│   ┌──────────────────┐         ┌──────────────────┐             │
│   │  Incident Lab    │◀────────│  Release Gate    │             │
│   │  (Learning)      │         │  (Gating)        │             │
│   └────────┬─────────┘         └──────────────────┘             │
│            │                                                     │
│            └──────────────────────────────────────────────────┐ │
│                              │                                 │ │
│                              ▼                                 │ │
│                    [ REGRESSION SUITE ]◀───────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Repository Structure

```
agentic-safety-demo/
├── scripts/
│   ├── step1_run_stress_tests.py    # Discovery
│   ├── step2_generate_regression.py # Conversion
│   ├── step3_run_release_gate.py    # Gating
│   └── step4_replay_incident.py     # Learning
├── artifacts/
│   ├── stress_failures.json         # Step 1 output
│   ├── regression_tests.json        # Step 2 output
│   ├── gate_report.html             # Step 3 output
│   └── incident_example.json        # Sample incident
├── configs/
│   └── demo.yaml                    # Demo configuration
├── Makefile                         # One-command demo
├── requirements.txt
└── README.md
```

---

## Live Demo (Terminal Recording)

Run the demo yourself:
```bash
make demo
```

Or record your own terminal session:
```bash
pip install asciinema
asciinema rec demo.cast
make demo
exit
```

Playback:
```bash
asciinema play demo.cast
```

---

## CI/CD Integration

This demo is designed for CI/CD integration:

```yaml
# .github/workflows/safety-gate.yml
name: Safety Release Gate

on:
  pull_request:
    branches: [main]

jobs:
  safety-gate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Release Gate
        run: |
          python scripts/step3_run_release_gate.py \
            --baseline main \
            --candidate ${{ github.head_ref }}
      - name: Upload Report
        uses: actions/upload-artifact@v3
        with:
          name: safety-report
          path: artifacts/gate_report.html
```

---

## Documentation

| Document | Description |
|----------|-------------|
| [exec_summary.md](docs/exec_summary.md) | Executive summary for leadership |
| [threat_model.md](docs/threat_model.md) | Adversaries, capabilities, coverage gaps |
| [failure_budget.md](docs/failure_budget.md) | Release thresholds, failure budget, anti-gaming |
| [launch_checklist.md](docs/launch_checklist.md) | Pre-release checklist, accountability matrix |
| [oncall_playbook.md](docs/oncall_playbook.md) | Incident response runbook |
| [metrics_glossary.md](docs/metrics_glossary.md) | Safety metrics definitions |
| [security_review_checklist.md](docs/security_review_checklist.md) | ProdSec / Infra sign-off checklist |
| [security_release_workflow.md](docs/security_release_workflow.md) | Security review -> release sign-off flow |
| [release_readiness_checklist.md](docs/release_readiness_checklist.md) | Hard gates + stop-the-line conditions |
| [oncall_incident_playbook.md](docs/oncall_incident_playbook.md) | 10-minute incident containment runbook |
| [model_card_safety.md](docs/model_card_safety.md) | Model card safety section template |
| [safety_metrics_dashboard_spec.md](docs/safety_metrics_dashboard_spec.md) | Grafana/Datadog dashboard spec |
| [residual_risk_acceptance_memo.md](docs/residual_risk_acceptance_memo.md) | VP/Legal risk sign-off template |
| [safety_slo_error_budget.md](docs/safety_slo_error_budget.md) | SRE-style safety SLOs and error budgets |
| [board_level_safety_update.md](docs/board_level_safety_update.md) | Board of Directors safety briefing |
| [regulator_auditor_briefing.md](docs/regulator_auditor_briefing.md) | 10-min compliance brief for auditors |
| [post_incident_external_disclosure.md](docs/post_incident_external_disclosure.md) | Public incident disclosure template |
| [ai_safety_regulatory_mapping.md](docs/ai_safety_regulatory_mapping.md) | EU AI Act / NIST RMF mapping |
| [kill_switch_and_rollback_policy.md](docs/kill_switch_and_rollback_policy.md) | Emergency shutdown policy |
| [model_decommissioning_policy.md](docs/model_decommissioning_policy.md) | Model retirement workflow |
| [capability_risk_register.md](docs/capability_risk_register.md) | Feature-level risk tracking |
| [safety_exception_process.md](docs/safety_exception_process.md) | Controlled gate override policy |
| [third_party_risk_policy.md](docs/third_party_risk_policy.md) | External dependency governance |
| [safety_governance_org_raci.md](docs/safety_governance_org_raci.md) | Org chart + RACI matrix |
| [red_team_program_charter.md](docs/red_team_program_charter.md) | Annual red-team program plan |
| [ai_safety_risk_heatmap.md](docs/ai_safety_risk_heatmap.md) | Capabilities x Harms x Controls |
| [annual_safety_roadmap.md](docs/annual_safety_roadmap.md) | Strategic investment plan |
| [safety_investment_roi_model.md](docs/safety_investment_roi_model.md) | Cost of incidents vs safeguards |
| [pre_mortem_12_month_failure.md](docs/pre_mortem_12_month_failure.md) | How this system fails |
| [safety_vs_velocity_tradeoff_framework.md](docs/safety_vs_velocity_tradeoff_framework.md) | Speed vs safety decision framework |
| [what_we_will_not_build_capability_boundaries.md](docs/what_we_will_not_build_capability_boundaries.md) | Explicit no-go capabilities |
| [first_90_days_safety_strategy.md](docs/first_90_days_safety_strategy.md) | New product safety bootstrap |
| [safety_principles_codex.md](docs/safety_principles_codex.md) | Foundational safety principles |
| [interview_onepager.md](docs/interview_onepager.md) | Portfolio system overview |

---

## Related Repositories

| Repository | Role in System |
|------------|----------------|
| [safeguards-stress-tests](https://github.com/yingchen-coding/safeguards-stress-tests) | Step 1: Discovery |
| [agentic-safety-incident-lab](https://github.com/yingchen-coding/agentic-safety-incident-lab) | Steps 2 & 4: Conversion & Learning |
| [model-safety-regression-suite](https://github.com/yingchen-coding/model-safety-regression-suite) | Step 3: Gating |

---

## Citation

```bibtex
@misc{chen2026agenticsafetydemo,
  title  = {Agentic Safety Demo: Closed-Loop Safety System Integration},
  author = {Chen, Ying},
  year   = {2026}
}
```

---

## Contact

Ying Chen, Ph.D.
yingchen.for.upload@gmail.com

---

## License

CC BY-NC 4.0
