# Demo Artifacts

This directory contains the outputs from each step of the closed-loop safety demo.

## Artifact Flow

| Step | Script | Artifact | Purpose |
|------|--------|----------|---------|
| 1 | `step1_run_stress_tests.py` | `stress_failures.json` | Discovered delayed failures from red-teaming |
| 2 | `step2_generate_regression.py` | `regression_tests.json` | Tests generated from failures |
| 3 | `step3_run_release_gate.py` | `gate_report.html` | Release verdict with metric diffs |
| 4 | `step4_replay_incident.py` | `incident_replay.json` | Root cause analysis + new tests |

## Why This Matters

These are not just "demo outputs" — they are **auditable artifacts** that would exist in a real production safety pipeline:

1. **stress_failures.json** → Evidence of what the red-team discovered
2. **regression_tests.json** → Permanent tests that prevent recurrence
3. **gate_report.html** → Decision record for release approval
4. **incident_replay.json** → Post-incident learning documentation

## Schema Examples

### stress_failures.json
```json
{
  "run_id": "stress_20260130_143052",
  "total_rollouts": 50,
  "failures": [
    {
      "failure_id": "F001",
      "attack_family": "decomposition",
      "first_failure_turn": 4,
      "severity": "high"
    }
  ]
}
```

### regression_tests.json
```json
{
  "tests": [
    {
      "test_id": "REG_F001",
      "source_failure": "F001",
      "expected_verdict": "BLOCK",
      "max_safe_turn": 3
    }
  ]
}
```

### gate_report.html
Human-readable report with:
- Metric comparison table
- Regression severity score
- Final verdict with reasons
- Approval/override signature fields

### incident_replay.json
```json
{
  "incident_id": "INC-2024-001",
  "root_cause": "policy_erosion",
  "blast_radius": {
    "affected_conversations": 1247,
    "risk_level": "systemic"
  },
  "new_regression_tests": 2
}
```

## Connection to Other Repos

| Artifact | Consumed By |
|----------|-------------|
| stress_failures.json | ⑦ incident-lab (for pattern matching) |
| regression_tests.json | ⑥ regression-suite (for release gating) |
| gate_report.html | CI/CD, Human reviewers |
| incident_replay.json | ⑦ incident-lab (for promotion) |
