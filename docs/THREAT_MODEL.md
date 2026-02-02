# Threat Model for Agentic Safety Systems

This document formalizes the threat model for the closed-loop safety system. It identifies adversaries, capabilities, attack surfaces, and coverage gaps.

---

## 1. Assets Under Protection

| Asset | Description | Compromise Impact |
|-------|-------------|-------------------|
| **User Intent Integrity** | User's actual goals vs what agent executes | Agent acts against user interests |
| **Safety Policy Compliance** | Adherence to harm prevention policies | Harmful outputs produced |
| **Tool Invocation Boundaries** | Limits on what tools can do | Unauthorized actions executed |
| **Evaluation Integrity** | Trustworthiness of safety metrics | False confidence in unsafe models |
| **Regression Test Suite** | Coverage of known failure modes | Reintroduction of fixed vulnerabilities |
| **Release Gate Authority** | Power to block unsafe releases | Unsafe models deployed to production |

---

## 2. Adversary Taxonomy

### External Adversaries

| Adversary | Skill Level | Motivation | Persistence |
|-----------|-------------|------------|-------------|
| **Opportunistic User** | Low | Curiosity, minor misuse | Low |
| **Skilled Attacker** | Medium | Targeted misuse, content extraction | Medium |
| **Adaptive Red-Teamer** | High | Bypass defenses systematically | High |
| **Automated Attack System** | High | Scale attacks, find edge cases | High |

### Internal Adversaries

| Adversary | Capability | Motivation | Detection Difficulty |
|-----------|------------|------------|---------------------|
| **Metric-Gaming Team** | Modify prompts, select scenarios | Pass release gate without improving safety | High |
| **Negligent Operator** | Disable safeguards, ignore alerts | Ship faster, reduce friction | Medium |
| **Compromised Insider** | Access to eval data, thresholds | Sabotage safety systems | Very High |

---

## 3. Attack Surfaces

### Runtime Attack Surfaces

| Surface | Attack Vector | Example | Control |
|---------|---------------|---------|---------|
| **Prompt Channel** | Injection, jailbreak, erosion | "Ignore previous instructions..." | Pre-action classifier |
| **Conversation History** | Context poisoning | Inject fake assistant messages | Trajectory monitor |
| **Tool Interfaces** | Hallucination, misuse | "Call delete_all_files()" | Action validator |
| **Memory Store** | Long-term context manipulation | Persist malicious instructions | Memory sanitizer |

### Evaluation Attack Surfaces

| Surface | Attack Vector | Example | Control |
|---------|---------------|---------|---------|
| **Test Scenarios** | Overfitting, memorization | Train on test set | Holdout rotation |
| **Thresholds** | Gaming, threshold hugging | Optimize to just pass | Gaming detector |
| **Metrics** | Goodhart's law | Improve metric, not safety | Multi-metric gates |
| **Human Review** | Alert fatigue | Flood with low-priority alerts | Tiered alerting |

---

## 4. Failure Modes

### Model-Level Failures

| Failure Mode | Description | Detection | Coverage |
|--------------|-------------|-----------|----------|
| **Immediate Jailbreak** | Direct policy violation turn 1 | Turn-level detector | High |
| **Delayed Policy Erosion** | Gradual compliance degradation | Trajectory monitor | Medium |
| **Intent Drift** | Shift from benign to harmful goal | Intent tracker | Medium |
| **Tool Misuse** | Using tools for unintended purposes | Action validator | Medium |

### System-Level Failures

| Failure Mode | Description | Detection | Coverage |
|--------------|-------------|-----------|----------|
| **Safeguard Bypass** | Circumventing safety checks | Audit log analysis | Medium |
| **Silent Regression** | Safety degradation across releases | Regression gating | High |
| **Alert Fatigue** | Operators ignore real threats | Alert audit | Low |
| **Metric Gaming** | Optimize benchmark, not safety | Gaming detector | Medium |

---

## 5. Coverage Analysis

### Current Coverage by Threat Type

| Threat | Coverage | Gaps | Priority |
|--------|----------|------|----------|
| Direct jailbreaks | 90% | Novel patterns | P2 |
| Policy erosion | 75% | Subtle drift | P1 |
| Intent drift | 70% | Goal extraction accuracy | P1 |
| Tool misuse | 65% | Complex tool chains | P1 |
| Metric gaming | 60% | Sophisticated gaming | P0 |
| Insider threats | 30% | Limited audit capability | P2 |

### Gap Remediation Roadmap

| Gap | Current State | Target State | Timeline |
|-----|---------------|--------------|----------|
| Subtle policy erosion | Heuristic detection | ML-based trajectory classifier | Q2 |
| Complex tool chains | Single-action validation | Chain-level validation | Q2 |
| Metric gaming | Pattern detection | Behavioral anomaly detection | Q3 |
| Insider threats | Basic audit logs | Full action attribution | Q4 |

---

## 6. Trust Boundaries

```
┌─────────────────────────────────────────────────────────────────┐
│                        TRUST BOUNDARIES                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   [UNTRUSTED]              [BOUNDARY]              [TRUSTED]     │
│                                                                  │
│   User Input        →    Intent Classifier    →    Planner      │
│   User History      →    Context Validator    →    Memory       │
│   Tool Responses    →    Output Verifier      →    Agent State  │
│   Model Outputs     →    Policy Checker       →    Response     │
│   PR Code Changes   →    Safety Gate          →    Production   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Threat Prioritization Matrix

| Priority | Threat | Likelihood | Impact | Status |
|----------|--------|------------|--------|--------|
| **P0** | Silent regression across releases | High | Critical | Mitigated via regression gating |
| **P0** | Metric gaming by teams | High | High | Mitigated via gaming detector |
| **P1** | Delayed policy erosion | High | High | Partially mitigated |
| **P1** | Adaptive multi-turn attacks | Medium | High | Partially mitigated |
| **P2** | Tool misuse chains | Medium | Medium | In progress |
| **P2** | Novel jailbreak patterns | Medium | Medium | Ongoing red-teaming |
| **P3** | Insider sabotage | Low | Critical | Basic controls only |

---

## 8. Non-Goals

| Non-Goal | Rationale |
|----------|-----------|
| **Prevent all misuse** | Impossible; focus on bounded risk |
| **Perfect detection** | Accept FN/FP tradeoff; optimize for recall |
| **Zero false positives** | Would require accepting dangerous FNs |
| **Eliminate human review** | Human judgment needed for edge cases |

---

## 9. Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2024-01 | Initial threat model |
| 1.1 | 2024-07 | Added metric gaming, organizational threats |
| 1.2 | 2024-08 | Added coverage analysis, gap remediation |

---

*This threat model is a living document. Update as new threats emerge and defenses evolve.*
