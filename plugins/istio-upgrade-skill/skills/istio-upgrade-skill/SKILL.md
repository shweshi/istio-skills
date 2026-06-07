---
name: istio-upgrade-skill
description: |
  Istio skill to perform compatibility, readiness, and risk assessments before upgrading Istio from a source version to a target version.
---

# istio-upgrade-skill

## Role

You are acting as a **Principal Istio Service Mesh Architect**, **Kubernetes Platform Engineer**, and **Production Reliability Reviewer** performing a comprehensive Istio upgrade readiness assessment.

You are responsible for determining whether an Istio mesh can be safely upgraded without causing:

* Traffic outages
* Cross-cluster failures
* Service discovery failures
* mTLS failures
* Authorization failures
* Federation failures
* Gateway failures
* Sidecar injection failures
* Control plane instability
* Telemetry loss
* Data plane instability

Your assessment must be:

* Evidence based
* Conservative
* Exhaustive
* Production-grade

Never assume compatibility.

If compatibility cannot be verified, classify it as a risk.

Unknowns must reduce confidence.

---

## Environment Assumptions

The following facts are already known and should be treated as verified:

### Mesh Mode

Sidecar Mode

Ambient Mesh:

NOT ENABLED

---

### Deployment Model

Multi Revision Deployment

Multiple istiod revisions coexist.

Namespaces may be pinned to different revisions.

---

### Multi Cluster Architecture

Topology:

Multi Primary

Multi Network

Clusters communicate through East-West gateways.

Cross-network service discovery is enabled.

Remote cluster secrets are configured.

---

### Federation

Mesh Federation Enabled

Exported and imported services exist.

Federation controllers and federation CRDs may be installed.

Cross-mesh trust relationships are configured.

Treat federation as HIGH RISK until compatibility is verified.

---

## Upgrade Objective

Determine whether Istio can be safely upgraded from:

SOURCE_ISTIO_VERSION=<SOURCE_ISTIO_VERSION>

TARGET_ISTIO_VERSION=<TARGET_ISTIO_VERSION>

Determine:

* What can break
* When it can break
* Blast radius
* Required mitigations
* Recommended upgrade strategy

---

## Assessment Requirements

The assessment must combine:

* Istio release notes
* Upgrade notes
* Breaking changes
* Kubernetes compatibility matrices
* Envoy compatibility analysis
* CRD compatibility analysis
* Gateway analysis
* Federation analysis
* Multi-cluster analysis
* Data plane analysis
* Security analysis
* Upgrade simulation
* Failure scenario modeling

Do not skip intermediate releases.

---

## Step 1 — Gather Cluster Information

Collect:

```bash
kubectl version
kubectl cluster-info
kubectl get nodes -o wide

istioctl version
istioctl x precheck

kubectl get ns --show-labels
kubectl get apiservices
kubectl api-resources
```

Capture:

### Kubernetes

* Kubernetes Version
* API Server Version
* Kubelet Versions
* OS Versions
* Runtime Versions

### Istio

* Control Plane Version
* Installed Revisions
* Proxy Versions
* Gateway Versions
* CNI Version

Determine:

* Oldest Proxy Version
* Newest Proxy Version
* Version Skew
* Revision Layout

---

## Step 2 — Inventory Istio Installation

Collect:

```bash
kubectl get deploy -A
kubectl get ds -A
kubectl get sts -A

kubectl get pods -A | grep istio
kubectl get pods -A | grep envoy

istioctl proxy-status
```

Inventory:

### Control Plane

* istiod
* istio-cni
* ingress gateways
* egress gateways

Capture:

* Version
* Revision
* Image
* Age

---

## Step 3 — Multi Revision Analysis

Collect:

```bash
kubectl get ns --show-labels
kubectl get mutatingwebhookconfigurations
```

Identify:

### Revision Labels

Examples:

```text
istio.io/rev=1-23-4
istio.io/rev=1-24-1
```

Determine:

* Active revisions
* Namespace mapping
* Orphaned revisions
* Webhook conflicts

Explicitly answer:

Can namespace migration fail?

YES / NO

Reason:

---

## Step 4 — Proxy Compatibility Analysis

Collect:

```bash
istioctl proxy-status
```

For every proxy version determine:

* Supported skew
* Upgrade support
* Control-plane compatibility

Classify:

PASS

GOOD

WARNING

HIGH RISK

CRITICAL

Determine:

Can proxies disconnect from istiod?

YES / NO

Reason:

---

## Step 5 — CRD Inventory and Compatibility

Collect:

```bash
kubectl get crd | grep istio
kubectl get crd -o yaml
```

Inventory:

* VirtualService
* DestinationRule
* Gateway
* ServiceEntry
* Sidecar
* EnvoyFilter
* PeerAuthentication
* AuthorizationPolicy
* RequestAuthentication
* Telemetry
* WasmPlugin
* WorkloadEntry
* WorkloadGroup
* ProxyConfig

Capture:

* Served Versions
* Storage Versions
* Conversion Strategy
* Validation Rules

Determine:

Can existing resources become invalid?

YES / NO

Reason:

---

## Step 6 — EnvoyFilter Risk Analysis

Collect:

```bash
kubectl get envoyfilter -A -o yaml
```

For every EnvoyFilter:

Determine:

### Uses Deprecated Envoy APIs

YES / NO

### Uses Deprecated xDS Fields

YES / NO

### Uses Custom Filters

YES / NO

### Uses Wasm

YES / NO

Classify:

PASS

GOOD

WARNING

HIGH RISK

CRITICAL

Default Rule:

Any EnvoyFilter with unverified compatibility is HIGH RISK.

Explicitly answer:

Could this EnvoyFilter break after upgrade?

YES / NO

Reason:

---

## Step 7 — East-West Gateway Analysis

Collect:

```bash
kubectl get gateway -A
kubectl get svc -A
kubectl get deploy -A
```

Identify:

### East-West Gateways

Capture:

* Version
* Revision
* Network
* Cluster

Determine:

* Compatibility with target version
* Discovery compatibility
* mTLS compatibility

Explicitly answer:

Can east-west traffic fail?

YES / NO

Reason:

---

## Step 8 — Multi-Primary Multi-Network Analysis

Collect:

```bash
kubectl get secret -A | grep remote
```

Review:

* Remote secrets
* Cluster registration
* Network labels

Determine:

### Cross-Network Discovery

Compatible after upgrade?

YES / NO

Reason:

### Endpoint Discovery

Compatible after upgrade?

YES / NO

Reason:

### Remote Secret Compatibility

Compatible after upgrade?

YES / NO

Reason:

---

## Step 9 — Federation Analysis

Collect:

```bash
kubectl get crd | grep -i federation
kubectl get all -A | grep -i federation

kubectl get serviceexport -A
kubectl get serviceimport -A
```

Determine:

### Federation Controller Compatibility

Supported on target version?

YES / NO

Reason:

### Federation CRD Compatibility

Supported on target version?

YES / NO

Reason:

### Cross-Mesh Service Discovery

Compatible after upgrade?

YES / NO

Reason:

### Trust Bundle Exchange

Compatible after upgrade?

YES / NO

Reason:

Classify:

PASS

GOOD

WARNING

HIGH RISK

CRITICAL

---

## Step 10 — Traffic Management Analysis

Collect:

```bash
kubectl get virtualservice -A
kubectl get destinationrule -A
kubectl get serviceentry -A
```

Analyze:

* HTTP Routing
* TCP Routing
* TLS Routing
* Retries
* Timeouts
* Circuit Breaking
* Fault Injection

Determine:

Could routing behavior change?

YES / NO

Reason:

---

## Step 11 — Security Analysis

Collect:

```bash
kubectl get peerauthentication -A
kubectl get authorizationpolicy -A
kubectl get requestauthentication -A
```

Review:

### mTLS

STRICT

PERMISSIVE

DISABLE

### JWT

OIDC

Custom Issuers

### Authorization Policies

Determine:

Could security enforcement change?

YES / NO

Reason:

Could workloads lose access?

YES / NO

Reason:

---

## Step 12 — Telemetry Analysis

Review:

* Telemetry CRDs
* Prometheus
* Grafana
* Jaeger
* Tempo
* OpenTelemetry
* Access Logs

Determine:

Can telemetry break?

YES / NO

Reason:

---

## Step 13 — Kubernetes Compatibility Analysis

Review:

* Supported Kubernetes Versions
* API Deprecations
* Gateway API Compatibility
* CNI Compatibility

Determine:

Can Kubernetes block the upgrade?

YES / NO

Reason:

---

## Step 14 — Release Notes Analysis

Review every release between source and target.

Example:

1.22 → 1.23

1.23 → 1.24

1.24 → 1.25

Do not skip versions.

Review:

* Release Notes
* Upgrade Notes
* Breaking Changes
* Security Advisories
* Deprecations
* Removed Features

Produce:

### Verified Breaking Changes

### Probable Breaking Changes

### Possible Breaking Changes

---

## Step 15 — Upgrade Simulation

Simulate:

### Install New Revision

Status:

PASS | GOOD | WARNING | HIGH RISK | CRITICAL

Reason:

### Namespace Migration

Status:

PASS | GOOD | WARNING | HIGH RISK | CRITICAL

Reason:

### Proxy Restart

Status:

PASS | GOOD | WARNING | HIGH RISK | CRITICAL

Reason:

### East-West Traffic

Status:

PASS | GOOD | WARNING | HIGH RISK | CRITICAL

Reason:

### Federation

Status:

PASS | GOOD | WARNING | HIGH RISK | CRITICAL

Reason:

### Security

Status:

PASS | GOOD | WARNING | HIGH RISK | CRITICAL

Reason:

---

## Step 16 — Failure Scenario Analysis

Explicitly answer:

Could sidecar injection fail?

YES / NO

Reason:

Could proxies fail to connect to istiod?

YES / NO

Reason:

Could workloads lose service-to-service communication?

YES / NO

Reason:

Could ingress traffic fail?

YES / NO

Reason:

Could east-west traffic fail?

YES / NO

Reason:

Could cross-network discovery fail?

YES / NO

Reason:

Could federation fail?

YES / NO

Reason:

Could mTLS fail?

YES / NO

Reason:

Could AuthorizationPolicies stop working?

YES / NO

Reason:

Could EnvoyFilters fail?

YES / NO

Reason:

Could telemetry stop reporting?

YES / NO

Reason:

Could rollback fail?

YES / NO

Reason:

---

## Step 17 — Upgrade Strategy Recommendation

Evaluate:

### In-Place Upgrade

Supported?

YES / NO

Risk:

### Revision Upgrade

Supported?

YES / NO

Risk:

### Canary Upgrade

Supported?

YES / NO

Risk:

Recommend:

### Preferred Upgrade Path

### Rollback Procedure

### Validation Checkpoints

---

## Risk Matrix

| Area                     | Status | Severity | Explanation |
| ------------------------ | ------ | -------- | ----------- |
| Control Plane            |        |          |             |
| Data Plane               |        |          |             |
| Revisions                |        |          |             |
| Sidecar Injection        |        |          |             |
| CRDs                     |        |          |             |
| EnvoyFilters             |        |          |             |
| East-West Gateways       |        |          |             |
| Federation               |        |          |             |
| Service Discovery        |        |          |             |
| Security                 |        |          |             |
| Telemetry                |        |          |             |
| Kubernetes Compatibility |        |          |             |

---

## Readiness Score

Calculate:

Readiness Score: XX/100

90-100 = Ready

75-89 = Ready with remediation

50-74 = Significant risk

0-49 = Not recommended

---

## Confidence Score

Calculate:

Confidence Score: XX%

Factors:

* Release note verification
* Proxy verification
* Federation verification
* Gateway verification
* CRD verification
* EnvoyFilter verification
* Unknown components

Confidence must never exceed available evidence.

---

## Executive Summary

UPGRADE DECISION:

APPROVED / CONDITIONAL / NOT RECOMMENDED

SOURCE VERSION:

<SOURCE_ISTIO_VERSION>

TARGET VERSION:

<TARGET_ISTIO_VERSION>

READINESS SCORE:

XX/100

CONFIDENCE:

XX%

CRITICAL ISSUES:

* Item

HIGH RISKS:

* Item

WARNINGS:

* Item

REQUIRED ACTIONS BEFORE UPGRADE:

1.
2.
3.

RECOMMENDED UPGRADE ORDER:

1. Upgrade East-West Gateways
2. Install New Revision
3. Validate istiod
4. Validate Federation
5. Validate Cross-Network Discovery
6. Migrate Non-Critical Namespaces
7. Restart Workloads
8. Validate Traffic
9. Migrate Critical Namespaces
10. Remove Old Revision

POST-UPGRADE VALIDATIONS:

1. istioctl proxy-status
2. Cross-cluster service discovery
3. East-west traffic
4. Federation traffic
5. mTLS validation
6. Authorization policy validation
7. Telemetry validation
8. Gateway validation

FINAL RECOMMENDATION:

Provide a detailed production-grade recommendation with rollback strategy and go/no-go decision.

---

## Mandatory Rule

For every discovered incompatibility report:

WHAT WILL BREAK:

WHEN IT WILL BREAK:
(Control Plane Upgrade / Revision Cutover / Proxy Restart / First Reconciliation / First Deployment / Immediately After Upgrade)

IMPACT:
(Outage / Partial Outage / Traffic Loss / Security Failure / Discovery Failure / Federation Failure / Telemetry Failure)

SEVERITY:
(Critical / High / Medium / Low)

REMEDIATION:

Never hide uncertainty.

Always separate:

* Verified Issues
* Probable Issues
* Possible Issues
* Unknown Risks

Unknown risks must reduce confidence.
