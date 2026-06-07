# Service Mesh Security Scorecard — Healthcare Platform

## Problem Description

A healthcare organization is preparing for an annual security review and needs a formal Zero Trust maturity assessment of their Istio service mesh deployment. The environment spans two namespaces — `clinical` and `admin` — running on Kubernetes with sidecar injection. The organization has provided a snapshot of their mesh configuration in `inputs/`.

The security team needs a scored assessment that stakeholders and auditors can use to understand the organization's Zero Trust posture. The CTO has requested a scorecard that rates each security dimension, a calculated overall score, a maturity level assignment, a confidence percentage, and an overall verdict. The mesh configuration has been captured from their environment but there is no live cluster access available.

Produce a full Zero Trust maturity assessment report in `assessment_report.md`.

## Output Specification

- `assessment_report.md`: A Zero Trust maturity assessment. Include an executive summary with Verdict, Zero Trust score, Maturity level, Confidence percentage, Scope, top exposure, and an immediate decision recommendation. Include a Scope and Evidence table, a Scorecard table rating each dimension 0-5 with weighted scores, a list of key findings, and a Residual Risk section listing what tests or evidence would be needed to improve confidence.
