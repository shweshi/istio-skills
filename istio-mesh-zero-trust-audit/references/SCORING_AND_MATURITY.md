# Scoring, Severity, And Maturity

## Finding Severity

Assign severity from evidence, not keywords:

| Severity | Criteria |
|---|---|
| CRITICAL | Directly reachable path to compromise a critical asset, broad unauthenticated access, or mesh-wide control failure with no effective compensating control |
| HIGH | Practical attack path with significant blast radius or sensitive-data impact; mitigation is required before accepting exposure |
| MEDIUM | Exploitable weakness with meaningful prerequisites, limited scope, or effective partial controls |
| LOW | Defense-in-depth or operational weakness with low immediate exploitability |
| INFORMATIONAL | Verified observation or improvement with no demonstrated security impact |

Increase severity for internet reachability, production data, privileged identities, cross-trust
access, and broad blast radius. Decrease it only for verified compensating controls. Missing
evidence lowers confidence; it does not automatically lower impact.

## Zero Trust Score

Score each dimension from 0 to 5, then apply the weight.

| Dimension | Weight |
|---|---:|
| Workload identity | 15 |
| mTLS and peer authentication | 15 |
| Request authentication | 10 |
| Authorization and least privilege | 20 |
| Namespace and workload segmentation | 15 |
| Ingress security | 10 |
| Egress governance | 10 |
| Auditability and validation | 5 |

For each dimension:

- `0`: absent or contradicted by evidence
- `1`: ad hoc, broad gaps
- `2`: partial deployment, important gaps
- `3`: broadly deployed, exceptions remain
- `4`: verified coverage with narrow exceptions
- `5`: verified least privilege, tested continuously

Calculate:

`score = sum((dimension_rating / 5) * dimension_weight)`

Cap the score at 59 when any unresolved CRITICAL finding exists. Do not score an unverified
dimension above 2. Report `N/A` rather than manufacturing a score when most dimensions are
unknown.

## Confidence

Start at 100 and deduct:

- 20: no live or recent data-plane evidence
- 15: incomplete workload/data-plane enrollment inventory
- 15: critical paths lack positive and negative traffic tests
- 10: effective mTLS unresolved
- 10: effective authorization unresolved
- 10: ingress or egress path incomplete
- 10: conflicting evidence remains
- 5: Istio version or control-plane revision unknown
- 5: business criticality or intended communication unknown

Minimum 0. Explain each deduction.

## Maturity Levels

Award the highest level for which all prior levels are verified:

| Level | Required evidence |
|---|---|
| 0 | No consistent mesh security controls |
| 1 | In-scope workloads have unique, managed identities and known mesh enrollment |
| 2 | mTLS is verified for critical paths; plaintext exceptions are documented |
| 3 | Sensitive workloads have intentional default deny and authenticated callers |
| 4 | Least-privilege policies are validated with positive and negative tests |
| 5 | Ingress, egress, and cross-namespace boundaries are governed with bypass controls |
| 6 | Attack paths, telemetry, policy drift, and exceptions are continuously reviewed |
| 7 | Controls are verified across clusters and trust domains with automated evidence and response |

Configuration without enforcement evidence cannot satisfy a level. State the first unmet level
and the exact evidence or remediation needed to reach it.

## Decision

- `STRONG`: score 85-100, no CRITICAL/HIGH findings, confidence at least 80
- `MODERATE`: score 70-84, no unresolved CRITICAL, bounded HIGH findings, confidence at least 65
- `WEAK`: score 50-69 or major unverified coverage
- `HIGH RISK`: score below 50 or any unresolved CRITICAL finding
- `INCONCLUSIVE`: confidence below 50 or insufficient evidence to score
