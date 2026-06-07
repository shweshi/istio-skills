# Istio Skills

**istio-skills** is a collection of agent-ready Tessl plugins for Istio upgrade
assessment, security auditing, and service mesh operations.

Each plugin contains a focused `SKILL.md` workflow and supporting references with
commands, decision criteria, examples, and report templates.

---

## 🛠️ Available Plugins & Skills

### [istio-upgrade-skill](/istio-upgrade-skill/.tessl-plugin/plugin.json)
[![tessl](https://img.shields.io/endpoint?url=https%3A%2F%2Fapi.tessl.io%2Fv1%2Fbadges%2Fshweshi%2Fistio-upgrade-skill)](https://tessl.io/registry/shweshi/istio-upgrade-skill)

- **Version**: `0.1.4`
- **Skill document**: [SKILL.md](/istio-upgrade-skill/SKILL.md)
- **Use when**: Planning an Istio upgrade, checking version compatibility,
  performing pre-upgrade validation, or preparing a migration.
- **What it does**: Checks CRD compatibility, proxy version skew, revision and
  webhook state, EnvoyFilter and Wasm risks, east-west gateways, federation,
  security policy compatibility, Kubernetes support, and intermediate release
  changes. It produces readiness and confidence scores, a go/no-go decision,
  an upgrade sequence, and a rollback strategy.
- **Assumed environment**: Sidecar mode, multi-revision deployment,
  multi-primary multi-network topology, and mesh federation.

#### Assessment Workflow

| Step | Area | Reference |
|------|------|-----------|
| 1 | Cluster and Istio discovery | - |
| 2 | Multi-revision and webhook audit | - |
| 3 | Proxy version skew | [PROXY_COMPATIBILITY.md](/istio-upgrade-skill/references/PROXY_COMPATIBILITY.md) |
| 4 | CRD storage version compatibility | [CRD_ANALYSIS.md](/istio-upgrade-skill/references/CRD_ANALYSIS.md) |
| 5 | EnvoyFilter xDS and Wasm risk | [ENVOYFILTER_ANALYSIS.md](/istio-upgrade-skill/references/ENVOYFILTER_ANALYSIS.md) |
| 6 | East-west gateways and multi-cluster | [EAST_WEST_GATEWAY.md](/istio-upgrade-skill/references/EAST_WEST_GATEWAY.md) |
| 7 | Federation controller and trust bundle | [FEDERATION_ANALYSIS.md](/istio-upgrade-skill/references/FEDERATION_ANALYSIS.md) |
| 8 | Security: mTLS, AuthzPolicy, JWT | [SECURITY_ANALYSIS.md](/istio-upgrade-skill/references/SECURITY_ANALYSIS.md) |
| 9-10 | Traffic, telemetry, and release notes | - |
| - | Risk matrix and scoring | [SCORING_AND_RISK.md](/istio-upgrade-skill/references/SCORING_AND_RISK.md) |

### [istio-mesh-zero-trust-audit](/istio-mesh-zero-trust-audit/.tessl-plugin/plugin.json)
[![tessl](https://img.shields.io/endpoint?url=https%3A%2F%2Fapi.tessl.io%2Fv1%2Fbadges%2Fshweshi%2Fistio-mesh-zero-trust-audit)](https://tessl.io/registry/shweshi/istio-mesh-zero-trust-audit)

- **Version**: `0.1.0`
- **Skill document**: [SKILL.md](/istio-mesh-zero-trust-audit/SKILL.md)
- **Use when**: Auditing, reviewing, assessing, or hardening an Istio mesh;
  investigating mTLS, identity, authorization, lateral movement, ingress,
  egress, trust-domain, or ambient-mesh concerns; or preparing compliance
  evidence.
- **What it does**: Maps effective identity, mTLS, authentication,
  authorization, segmentation, ingress, egress, ambient, and multi-cluster
  controls. It validates configuration against data-plane behavior and
  produces evidence-backed findings, attack paths, maturity and confidence
  scores, and a prioritized remediation plan.
- **Evidence model**: Labels conclusions as `VERIFIED`, `INFERRED`, or
  `UNKNOWN` and requires positive and negative traffic tests for critical
  paths where testing is permitted.

#### Audit Resources

| Area | Reference |
|------|-----------|
| Read-only cluster discovery and validation commands | [EVIDENCE_COLLECTION.md](/istio-mesh-zero-trust-audit/references/EVIDENCE_COLLECTION.md) |
| Vulnerable and hardened Istio policy examples | [POLICY_EXAMPLES.md](/istio-mesh-zero-trust-audit/references/POLICY_EXAMPLES.md) |
| Severity, score, confidence, and maturity rules | [SCORING_AND_MATURITY.md](/istio-mesh-zero-trust-audit/references/SCORING_AND_MATURITY.md) |
| Evidence-backed audit report structure | [OUTPUT_TEMPLATE.md](/istio-mesh-zero-trust-audit/references/OUTPUT_TEMPLATE.md) |

---

## 🚀 How to Use These Plugins

These skills support agent workflows and manual operational reviews.

### For AI/Agent Integrations

1. Load the appropriate plugin's `.tessl-plugin/plugin.json`.
2. Let the agent activate the plugin's `SKILL.md` for matching requests.
3. Provide live read-only cluster access or redacted manifests and command
   output such as `kubectl version`, `istioctl proxy-status`, and
   `istioctl analyze --all-namespaces`.
4. State the mesh topology, scope, target outcome, and any business or
   compliance constraints.

### For Manual Operations

Follow the workflow in the selected `SKILL.md`, then open only the referenced
files needed for the assessment. Treat secrets, tokens, private keys, and full
certificate material as sensitive and redact them from collected evidence.

### Validate A Plugin

```bash
tessl skill lint ./istio-upgrade-skill
tessl skill lint ./istio-mesh-zero-trust-audit
```

---

## 🤝 Contributing

Contributions of new Tessl-compliant plugins and improvements to existing
skills are welcome. Keep the main skill concise, place detailed material in
directly linked `references/` files, and run `tessl skill lint` before
submitting changes.

---

## 📄 License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
