# Internal Launch Checklist

This document provides a pre-release safety checklist, on-call procedures, and incident severity rubric for agentic system deployments.

---

## 1. Pre-Release Safety Checklist

### Phase 1: Development Complete

| # | Item | Owner | Status |
|---|------|-------|--------|
| 1.1 | All planned features implemented | Eng Lead | [ ] |
| 1.2 | Unit tests passing (>90% coverage) | Eng Lead | [ ] |
| 1.3 | Integration tests passing | Eng Lead | [ ] |
| 1.4 | Code review completed | Eng Lead | [ ] |
| 1.5 | Security review completed | Security | [ ] |

### Phase 2: Safety Evaluation

| # | Item | Owner | Status |
|---|------|-------|--------|
| 2.1 | Regression suite passing | Safety Eng | [ ] |
| 2.2 | Release gate verdict: OK or WARN | Safety Eng | [ ] |
| 2.3 | New scenarios added for changed behavior | Safety Eng | [ ] |
| 2.4 | Red-team evaluation completed | Red Team | [ ] |
| 2.5 | Failure budget within limits | Safety Lead | [ ] |

### Phase 3: Operational Readiness

| # | Item | Owner | Status |
|---|------|-------|--------|
| 3.1 | Monitoring dashboards configured | SRE | [ ] |
| 3.2 | Alerting rules updated | SRE | [ ] |
| 3.3 | Runbook updated for new features | SRE | [ ] |
| 3.4 | On-call briefed on changes | On-Call Lead | [ ] |
| 3.5 | Rollback procedure tested | SRE | [ ] |

### Phase 4: Launch Approval

| # | Item | Owner | Status |
|---|------|-------|--------|
| 4.1 | Safety Lead sign-off | Safety Lead | [ ] |
| 4.2 | Product Lead sign-off | Product Lead | [ ] |
| 4.3 | Eng Lead sign-off | Eng Lead | [ ] |
| 4.4 | Launch window confirmed | Release Mgr | [ ] |
| 4.5 | Rollback owner identified | Release Mgr | [ ] |

---

## 2. Launch Day Procedure

### T-24h: Final Checks

```
[ ] Regression suite green
[ ] No open S0/S1 issues
[ ] On-call coverage confirmed
[ ] Rollback tested in staging
[ ] Communication drafted (if needed)
```

### T-1h: Pre-Launch

```
[ ] Final regression run
[ ] Monitoring dashboards open
[ ] On-call channel active
[ ] Rollback command ready
```

### T-0: Launch

```
[ ] Deploy to canary (1%)
[ ] Monitor for 15 minutes
[ ] Check error rates, latency
[ ] Check safety metrics
[ ] If clean: expand to 10%
[ ] Monitor for 30 minutes
[ ] If clean: expand to 100%
```

### T+1h: Post-Launch

```
[ ] Confirm all metrics stable
[ ] Close launch ticket
[ ] Update status page (if applicable)
[ ] Notify stakeholders
```

---

## 3. On-Call Rotation

### Rotation Schedule

| Role | Primary | Secondary | Escalation |
|------|---------|-----------|------------|
| Safety On-Call | Rotating weekly | Rotating weekly | Safety Lead |
| SRE On-Call | Rotating weekly | Rotating weekly | SRE Lead |
| Eng On-Call | Rotating weekly | Per-team | Eng Lead |

### On-Call Responsibilities

**Safety On-Call:**
- Monitor safety dashboards
- Triage incoming safety alerts
- Escalate S0/S1 incidents immediately
- Document all incidents

**SRE On-Call:**
- Monitor system health
- Execute rollbacks if needed
- Coordinate with Safety On-Call

### Handoff Procedure

1. Review open incidents
2. Check recent deployments
3. Review failure budget burn rate
4. Confirm escalation contacts
5. Sign handoff log

---

## 4. Incident Severity Rubric

### Severity Definitions

| Severity | Criteria | Response Time | Escalation |
|----------|----------|---------------|------------|
| **SEV-0** | Active harm, legal risk, data breach | Immediate | Executive + Legal |
| **SEV-1** | Policy violation at scale, S1 budget exceeded | 15 min | Safety Lead + Eng Lead |
| **SEV-2** | Elevated failure rate, S2 budget at risk | 1 hour | Safety On-Call + SRE |
| **SEV-3** | Anomaly detected, investigation needed | 4 hours | Safety On-Call |
| **SEV-4** | Minor issue, no user impact | Next business day | Ticket queue |

### Severity Decision Tree

```
Is there active harm or legal risk?
  YES → SEV-0
  NO ↓

Is S1 failure budget exceeded?
  YES → SEV-1
  NO ↓

Is S2 failure budget at >80%?
  YES → SEV-2
  NO ↓

Is there an anomaly requiring investigation?
  YES → SEV-3
  NO → SEV-4
```

---

## 5. Incident Response Procedure

### SEV-0/SEV-1 Response

```
T+0min:  Alert received
         → Acknowledge alert
         → Page secondary on-call
         → Open incident channel

T+5min:  Initial assessment
         → Confirm severity
         → Identify affected scope
         → Begin rollback if needed

T+15min: Incident Commander assigned
         → Coordinate response
         → Draft initial communication

T+30min: Status update
         → Internal stakeholders
         → Customer communication (if needed)

T+1h:    Stabilization
         → Confirm mitigation in place
         → Monitor for recurrence

T+24h:   Initial post-mortem
         → Timeline
         → Root cause hypothesis
         → Immediate actions

T+72h:   Full post-mortem
         → Root cause confirmed
         → Remediation plan
         → Regression tests added
```

### SEV-2/SEV-3 Response

```
T+0:     Alert received
         → Acknowledge alert
         → Begin investigation

T+1h:    Initial assessment
         → Document findings
         → Determine if escalation needed

T+4h:    Resolution or escalation
         → Fix deployed OR
         → Escalate to SEV-1

T+24h:   Follow-up
         → Confirm resolution
         → Document learnings
```

---

## 6. Rollback Procedure

### Rollback Decision Criteria

| Condition | Action |
|-----------|--------|
| SEV-0 confirmed | Immediate rollback |
| SEV-1 with no fix in 30min | Rollback |
| S1 budget exceeded by 2x | Rollback |
| Customer-impacting errors >1% | Rollback |

### Rollback Command

```bash
# Standard rollback
kubectl rollout undo deployment/agent-service

# Verify rollback
kubectl rollout status deployment/agent-service

# Confirm metrics
curl -s http://metrics/safety_failure_rate
```

### Post-Rollback

1. Confirm metrics stabilized
2. Notify stakeholders
3. Create investigation ticket
4. Schedule post-mortem

---

## 7. Communication Templates

### Internal Incident Update

```
INCIDENT UPDATE - [SEV-X] [Title]

Status: [Investigating / Mitigating / Resolved]
Impact: [Description of user impact]
Start Time: [ISO timestamp]
Current Time: [ISO timestamp]

Summary:
[2-3 sentences describing the issue]

Actions Taken:
- [Action 1]
- [Action 2]

Next Steps:
- [Next action with owner and ETA]

Incident Commander: [Name]
Next Update: [Time]
```

### Customer Communication (SEV-0/SEV-1)

```
We identified an issue affecting [service description] starting at [time].

Impact: [Customer-facing description]

We have [mitigated/resolved] the issue as of [time].

We apologize for any inconvenience and are conducting a thorough review to prevent recurrence.

For questions, contact [support channel].
```

---

## 8. Post-Incident Review

### Post-Mortem Template

```markdown
# Post-Mortem: [Incident Title]

**Date:** [Date]
**Severity:** [SEV-X]
**Duration:** [Start] to [End]
**Author:** [Name]
**Reviewers:** [Names]

## Summary
[2-3 sentence summary]

## Impact
- Users affected: [N]
- Failure budget consumed: [X%]
- Revenue impact: [if applicable]

## Timeline
| Time | Event |
|------|-------|
| HH:MM | [Event] |

## Root Cause
[Detailed technical explanation]

## Contributing Factors
- [Factor 1]
- [Factor 2]

## Resolution
[How the incident was resolved]

## Action Items
| # | Action | Owner | Due Date | Status |
|---|--------|-------|----------|--------|
| 1 | [Action] | [Owner] | [Date] | [ ] |

## Lessons Learned
- What went well:
- What could be improved:

## Regression Tests Added
- [Test ID]: [Description]
```

---

## 9. Checklist Audit

### Weekly Audit

- [ ] All incidents properly documented
- [ ] Post-mortems completed within SLA
- [ ] Regression tests added for incidents
- [ ] On-call handoffs logged
- [ ] Failure budget burn rate reviewed

### Monthly Audit

- [ ] Checklist process reviewed
- [ ] On-call feedback collected
- [ ] Runbooks updated
- [ ] Training needs identified

---

*A launch without a checklist is a launch without a safety net.*
