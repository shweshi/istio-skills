# Ambient Mesh Security Review — Logistics Platform

## Problem Description

A logistics company recently migrated their Kubernetes platform from sidecar-based Istio to ambient mode. The platform team believes their workloads are now protected by the same authorization policies they had before migration. A security consultant has been brought in to review whether this assumption is correct and to identify any protection gaps introduced by the migration.

The `inputs/` directory contains namespace enrollment status, gateway definitions, authorization policies, and pod manifests as captured from the environment. No live cluster access is available.

Produce a security assessment of the ambient mesh deployment in `ambient_assessment.md`. Determine which workloads are protected and to what degree, identify any protection gaps introduced by the migration, and provide a prioritized remediation plan.

## Output Specification

- `ambient_assessment.md`: A structured security assessment. Include an executive summary, a workload protection inventory mapping each workload to its observed mesh coverage, a findings section covering any policy enforcement gaps, and a remediation plan. Each finding must include its severity, confidence level, affected assets, evidence, the attack scenario it enables, business impact, remediation, and validation steps.
