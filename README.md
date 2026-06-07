# Istio Skills

Welcome to **istio-skills**—a curated repository of expert-level, agent-ready skills and system prompts for Istio service mesh administration, upgrade assessment, and operations.

This repository is structured as a collection of **Tessl plugins** containing specialized operational workflows, verification procedures, and diagnostic guidelines.

---

## 🛠️ Available Plugins & Skills

### [istio-upgrade-skill](/istio-upgrade-skill/.tessl-plugin/plugin.json)
*   **Version**: `0.1.0`
*   **Skill Document**: [SKILL.md](/istio-upgrade-skill/SKILL.md)
*   **Use when**: The user asks about upgrading Istio, checking version compatibility, planning an Istio migration, performing pre-upgrade validation, or preparing for a version bump.
*   **What it does**: Checks CRD compatibility, validates sidecar proxy version skew, reviews EnvoyFilter deprecated API usage, analyzes east-west gateway and federation compatibility, identifies breaking changes across intermediate Istio releases, and produces a scored upgrade readiness assessment with a go/no-go recommendation.
*   **Environment**: Sidecar mode, Multi-Revision deployment, Multi-Primary Multi-Network topology, Mesh Federation.

#### Assessment Workflow
| Step | Area | Reference |
|------|------|-----------|
| 1 | Cluster & Istio discovery | — |
| 2 | Multi-revision & webhook audit | — |
| 3 | Proxy version skew | [PROXY_COMPATIBILITY.md](/istio-upgrade-skill/references/PROXY_COMPATIBILITY.md) |
| 4 | CRD storage version compatibility | [CRD_ANALYSIS.md](/istio-upgrade-skill/references/CRD_ANALYSIS.md) |
| 5 | EnvoyFilter xDS & Wasm risk | [ENVOYFILTER_ANALYSIS.md](/istio-upgrade-skill/references/ENVOYFILTER_ANALYSIS.md) |
| 6 | East-West gateways & multi-cluster | [EAST_WEST_GATEWAY.md](/istio-upgrade-skill/references/EAST_WEST_GATEWAY.md) |
| 7 | Federation controller & trust bundle | [FEDERATION_ANALYSIS.md](/istio-upgrade-skill/references/FEDERATION_ANALYSIS.md) |
| 8 | Security: mTLS, AuthzPolicy, JWT | [SECURITY_ANALYSIS.md](/istio-upgrade-skill/references/SECURITY_ANALYSIS.md) |
| 9–10 | Traffic, telemetry, release notes | — |
| — | Risk matrix & scoring | [SCORING_AND_RISK.md](/istio-upgrade-skill/references/SCORING_AND_RISK.md) |

---

## 🚀 How to Use These Plugins

These skills are optimized for integration into agentic workflows or manual change-management checklists.

### For AI/Agent Integrations
1.  Configure the agent workspace to load the plugin from [plugin.json](/istio-upgrade-skill/.tessl-plugin/plugin.json).
2.  Provide the contents of the [SKILL.md](/istio-upgrade-skill/SKILL.md) file as system context.
3.  Supply raw cluster dumps (e.g., `kubectl version`, `istioctl proxy-status`) and request a production-grade upgrade risk assessment report.

### For Manual Operations
*   Follow the standardized step-by-step diagnostic workflows in `SKILL.md` and dive into the relevant `references/` file for area-specific decision logic.

---

## 🤝 Contributing

We welcome additions of new Tessl-compliant plugins or enhancements to existing one.

---

## 📄 License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.