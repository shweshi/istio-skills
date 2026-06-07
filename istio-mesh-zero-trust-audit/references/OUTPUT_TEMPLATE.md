# Audit Report Template

## Executive Summary

- **Verdict:** STRONG / MODERATE / WEAK / HIGH RISK / INCONCLUSIVE
- **Zero Trust score:** `<0-100 or N/A>`
- **Maturity level:** `<0-7 or N/A>`
- **Confidence:** `<0-100%>`
- **Scope:** `<clusters, namespaces, workloads, time>`
- **Top exposure:** `<one sentence>`
- **Immediate decision:** `<acceptable, conditional, or unacceptable risk>`

## Scope And Evidence

| Item | Value |
|---|---|
| Istio/Kubernetes versions | |
| Data-plane mode | |
| Clusters/namespaces | |
| Evidence collected | |
| Missing evidence | |
| Assumptions | |

## Scorecard

| Dimension | Rating (0-5) | Weighted score | Evidence status | Key gap |
|---|---:|---:|---|---|
| Workload identity | | | VERIFIED/INFERRED/UNKNOWN | |
| mTLS | | | | |
| Request authentication | | | | |
| Authorization | | | | |
| Segmentation | | | | |
| Ingress | | | | |
| Egress | | | | |
| Auditability | | | | |

Explain confidence deductions and the first unmet maturity level.

## Prioritized Findings

### `[ID] Title`

- **Severity / confidence:** `<severity>` / `<high, medium, low>`
- **Status:** `VERIFIED / INFERRED / UNKNOWN`
- **Affected assets:** `<resources and workloads>`
- **Evidence:** `<resource, selector, command output, or test>`
- **Enforcement observed:** `<yes/no/not tested>`
- **Attack scenario:** `<source -> hops -> target>`
- **Business impact:** `<availability, confidentiality, integrity, compliance>`
- **Remediation:** `<specific change>`
- **Validation:** `<positive and negative tests>`
- **Rollback:** `<how to restore service safely>`
- **Owner / target date:** `<when known>`

Order findings by severity, then exploitability and blast radius.

## Attack Paths

| Source | Identity | Path | Target | Existing controls | Result | Confidence |
|---|---|---|---|---|---|---|
| | | | | | ALLOWED/DENIED/UNKNOWN | |

## Remediation Plan

### Immediate

Contain active or critical exposure without breaking required traffic.

### 30 Days

Close high-risk identity, mTLS, authorization, ingress, and egress gaps.

### 90 Days

Complete least privilege, segmentation, policy testing, and operational ownership.

### Strategic

Automate drift detection, attack-path tests, exception expiry, and audit evidence.

## Residual Risk And Retest

List accepted exceptions, compensating controls, unresolved contradictions, and exact commands
or tests needed to close each unknown. End with a concise evidence-backed final assessment.
