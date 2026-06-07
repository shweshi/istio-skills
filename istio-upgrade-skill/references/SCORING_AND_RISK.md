# Risk Matrix & Scoring Rubric

Use this document to evaluate the upgrade risks and compute final readiness and confidence scores.

## Risk Matrix Template

Fill out this matrix for all evaluated areas:

| Area | Status | Severity | Explanation |
|------|--------|----------|-------------|
| Control Plane | | | |
| Data Plane / Proxy Skew | | | |
| Revisions & Webhooks | | | |
| Sidecar Injection | | | |
| CRDs | | | |
| EnvoyFilters | | | |
| East-West Gateways | | | |
| Federation | | | |
| Service Discovery | | | |
| Security (mTLS / AuthzPolicy) | | | |
| Telemetry | | | |
| Kubernetes Compatibility | | | |

---

## Scoring Rubric

### Readiness Score (0-100)
Start at **100** points. Deduct points for each compatibility or risk finding based on its severity:
- **CRITICAL**: Deduct **20** points per finding
- **HIGH RISK**: Deduct **10** points per finding
- **WARNING**: Deduct **3** points per finding

| Score | Decision | Recommendation |
|-------|----------|----------------|
| 90-100 | **[OK] Ready** | Upgrade is safe to proceed. |
| 75-89 | **[WARN] Ready with remediation** | Upgrade can proceed after addressing warnings/remediations. |
| 50-74 | **[RISK] Significant risk** | Not recommended; significant risks or blockers must be resolved first. |
| 0-49 | **[NO] Not recommended** | Blocker issues exist; upgrade is unsafe. |

### Confidence Score (0-100%)
Start at **100%** confidence. Deduct to reflect unknowns or unverified configurations:
- Deduct **10%** for each unverified area (e.g., no access to config, unable to check).
- Deduct **5%** for each unknown/unidentified custom component.
- **Rule**: The confidence score must never exceed the available evidence.
