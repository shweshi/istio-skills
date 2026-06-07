# Mesh Audit — Reconciling Configuration and Observed Behavior

## Problem Description

An e-commerce company's security team is auditing their Istio mesh after a recent incident report flagged unexpected plaintext traffic between two services that were believed to be mTLS-protected. The team has collected two types of evidence: static manifests from the cluster, and a snapshot of observed proxy behavior captured from `istioctl` commands.

The `inputs/` directory contains the authorization policy manifests, PeerAuthentication resources, and `istioctl proxy-config` command outputs for two workloads. In several places the observed proxy configuration contradicts what the manifests suggest should be enforced.

The security team needs a written analysis in `conflict_analysis.md` that reconciles these contradictions, explains the most likely cause of each discrepancy, and provides a prioritized set of findings with remediation steps. The team's primary concern is whether any real protection gaps exist in the current state.

## Output Specification

- `conflict_analysis.md`: A structured conflict analysis and audit report. Include: an executive summary, a section listing each evidence contradiction and the most likely cause, a findings section where each finding has its severity, confidence, evidence, attack scenario, business impact, remediation, and validation steps. Include a section explaining what evidence the team would need to definitively resolve each remaining unknown. Use VERIFIED / INFERRED / UNKNOWN labels throughout.
