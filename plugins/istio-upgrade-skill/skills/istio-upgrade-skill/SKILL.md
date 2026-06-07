---
name: istio-upgrade-skill
description: |
  Use when the user asks about upgrading Istio, checking Istio version compatibility, planning an Istio migration, performing pre-upgrade checks, preparing for a version bump, or creating an Istio upgrade plan. Checks CRD compatibility and storage version changes, validates sidecar proxy version skew against control-plane skew limits, reviews EnvoyFilter deprecated xDS API usage and Wasm ABI compatibility, analyzes east-west gateway upgrade ordering in multi-cluster environments, assesses federation controller compatibility and trust bundle exchange, identifies breaking changes across all intermediate Istio releases, and produces a scored upgrade readiness assessment with a go/no-go recommendation and rollback strategy.
---

# istio-upgrade-skill

## Environment (Pre-verified)

- **Mesh mode**: Sidecar (Ambient: disabled)
- **Deployment model**: Multi-Revision -- multiple istiod revisions coexist; namespaces pinned per revision
- **Cluster topology**: Multi-Primary, Multi-Network -- clusters communicate via East-West gateways
- **Federation**: Enabled -- treat as HIGH RISK until compatibility verified

---

## Upgrade Objective

Assess whether Istio can be safely upgraded from `<SOURCE_ISTIO_VERSION>` to `<TARGET_ISTIO_VERSION>`.

Produce: risk matrix, readiness score, confidence score, and a go/no-go recommendation.

---

## Analysis Rules

### Severity & Incompatibility Format

Classify issues by severity: **CRITICAL** (blocked), **HIGH RISK** (mitigate before upgrade), or **WARNING** (verify/remediate).

For each issue, report:
- **Issue**: <what breaks> (Severity: CRITICAL/HIGH/WARN | Impact: <outage type> | Trigger: <upgrade phase>)
- **Fix**: <remediation>

Separate findings into: **Verified** | **Probable** | **Possible** | **Unknown** (unknowns reduce confidence score).

---

## Step 1 -- Gather Cluster State

```bash
kubectl version && kubectl get nodes -o wide
istioctl version && istioctl x precheck
kubectl get ns --show-labels
istioctl proxy-status
```

Capture: Kubernetes version, Istio control plane version, all installed revisions, oldest/newest proxy version, version skew.

---

## Step 2 -- Multi-Revision & Webhook Audit

```bash
kubectl get ns --show-labels | grep 'istio.io/rev'
kubectl get mutatingwebhookconfigurations | grep istio
```

- Map namespaces -> revision labels.
- Flag orphaned revisions (revision label present, no live istiod for that revision).
- Flag duplicate webhooks for the same namespace (webhook conflict -> injection failure).

---

## Step 3 -- Proxy Skew

See [`references/PROXY_COMPATIBILITY.md`](references/PROXY_COMPATIBILITY.md) for classification thresholds and decision logic.

Key rule: skew >= N+2 minor versions -> **HIGH RISK**; skew > N+3 -> **CRITICAL** (upgrade blocked).

---

## Step 4 -- CRD Compatibility

```bash
kubectl get crd | grep istio.io
kubectl get crd -o json | jq '.items[] | select(.metadata.name | contains("istio.io")) | {name: .metadata.name, storedVersion: .spec.versions[] | select(.storage==true) | .name}'
```

See [`references/CRD_ANALYSIS.md`](references/CRD_ANALYSIS.md) for inventory checklist and risk classification.

Key rule: stored API version removed in target -> **CRITICAL**; new validation rules breaking existing resources -> **HIGH RISK**.

---

## Step 5 -- EnvoyFilter Analysis

```bash
kubectl get envoyfilter -A -o yaml
```

See [`references/ENVOYFILTER_ANALYSIS.md`](references/ENVOYFILTER_ANALYSIS.md) for xDS pattern detection and Wasm ABI rules.

Key rule: any `v2` proto reference -> **CRITICAL**; Wasm ABI mismatch -> **CRITICAL**; unverified filter -> **HIGH RISK**.

---

## Step 6 -- East-West Gateways & Multi-Cluster

```bash
kubectl get deploy -A | grep -E "east-west|eastwest"
kubectl get secret -A | grep -E "remote-secret|multiCluster"
istioctl remote-clusters
```

See [`references/EAST_WEST_GATEWAY.md`](references/EAST_WEST_GATEWAY.md) for upgrade ordering rules and decision logic.

Key rule: upgrade EW gateways **before** control plane; missing remote secret -> **CRITICAL**.

---

## Step 7 -- Federation

```bash
kubectl get crd | grep -iE "federation|serviceexport|serviceimport"
kubectl get serviceexport,serviceimport -A
```

See [`references/FEDERATION_ANALYSIS.md`](references/FEDERATION_ANALYSIS.md) for controller compatibility matrix and trust bundle rules.

Key rule: no published federation controller compatibility with target Istio -> **CRITICAL**; CA change -> **CRITICAL**.

---

## Step 8 -- Security

```bash
kubectl get peerauthentication,authorizationpolicy,requestauthentication -A -o yaml
```

See [`references/SECURITY_ANALYSIS.md`](references/SECURITY_ANALYSIS.md) for mTLS, AuthorizationPolicy version-specific breaking changes, and JWT rules.

Key rule: STRICT mTLS + proxy skew > N-1 -> **HIGH RISK**; `CUSTOM` action policy with missing provider -> **CRITICAL**.

---

## Step 9 -- Traffic, Telemetry & Kubernetes Compatibility

```bash
kubectl get virtualservice,destinationrule,serviceentry -A
istioctl analyze -A
```

- Run `istioctl analyze` against target CRD schema to surface invalid resources before upgrade.
- Confirm Kubernetes version is in the [Istio support matrix](https://istio.io/latest/docs/releases/supported-releases/) for the target version.

---

## Step 10 -- Release Notes (All Intermediate Versions)

For each release between source and target, review:
- Breaking changes
- Upgrade notes
- Deprecated / removed features
- Security advisories

**Do not skip intermediate releases.**

---

## Upgrade Strategy

Evaluate in this order:

1. **In-Place Upgrade** -- supported only for patch versions; not recommended for minor version jumps in multi-cluster environments.
2. **Revision Upgrade (Canary)** -- preferred strategy; installs new istiod revision alongside old, migrates namespaces incrementally.

### Recommended Upgrade Order (Canary)

1. Upgrade East-West Gateways on all clusters
2. Install new istiod revision
3. Validate new istiod (`istioctl proxy-status`)
4. Validate federation and cross-network discovery
5. Migrate non-critical namespaces; restart workloads
6. Validate traffic and telemetry
7. Migrate critical namespaces; restart workloads
8. Remove old revision

### Rollback Procedure

- Relabel namespaces back to old revision label.
- Restart workloads to pull old sidecar.
- Remove new istiod revision deployment.
- Rollback EW gateways if upgraded.

---

## Risk Matrix

See [`references/SCORING_AND_RISK.md`](references/SCORING_AND_RISK.md) for the evaluation template. Assess and complete the matrix for all 12 operational areas (Control Plane, Data Plane / Proxy Skew, Revisions/Webhooks, Sidecar Injection, CRDs, EnvoyFilters, East-West Gateways, Federation, Service Discovery, Security, Telemetry, and Kubernetes Compatibility).

---

## Scoring

See [`references/SCORING_AND_RISK.md`](references/SCORING_AND_RISK.md) for the scoring rubric and deduction values. Calculate the Readiness Score (0-100) and Confidence Score (0-100%) to determine the final upgrade decision.

---

## Executive Summary (Output Template)

```
UPGRADE DECISION:  APPROVED / CONDITIONAL / NOT RECOMMENDED

SOURCE VERSION:    <SOURCE_ISTIO_VERSION>
TARGET VERSION:    <TARGET_ISTIO_VERSION>
READINESS SCORE:   XX/100
CONFIDENCE:        XX%

CRITICAL ISSUES:   (list)
HIGH RISKS:        (list)
WARNINGS:          (list)

REQUIRED ACTIONS BEFORE UPGRADE:
  1.
  2.

FINAL RECOMMENDATION:
  <production-grade go/no-go with rollback strategy>
```
