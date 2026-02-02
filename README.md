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
| [model_card_safety.md](docs/model_card_safety.md) | Model card safety section template |
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
blueoceanally@gmail.com

---

## License

CC BY-NC 4.0
