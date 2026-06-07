# Istio Skills

Welcome to **istio-skills**—a curated repository of expert-level, agent-ready skills and system prompts for Istio service mesh administration, upgrade assessment, and operations.

This repository is structured as a collection of **Tessl plugins** containing specialized operational workflows, verification procedures, and diagnostic guidelines.

---

## 📂 Repository Structure

The repository is organized around a plugin-based architecture under the `plugins/` directory:

```text
istio-skills/
├── plugins/
│   └── istio-upgrade-skill/
│       ├── .tessl-plugin/
│       │   └── plugin.json             # Plugin metadata configuration
│       └── skills/
│           └── istio-upgrade-skill/
│               └── SKILL.md            # Structured skill/runbook definition
├── LICENSE
└── README.md
```

---

## 🛠️ Available Plugins & Skills

### [istio-upgrade-skill](file:///Users/shashiprakashgautam/projects/istio-skills/plugins/istio-upgrade-skill/.tessl-plugin/plugin.json)
*   **Version**: `0.1.0`
*   **Skill Document**: [SKILL.md](file:///Users/shashiprakashgautam/projects/istio-skills/plugins/istio-upgrade-skill/skills/istio-upgrade-skill/SKILL.md)
*   **Objective**: Perform compatibility, readiness, and risk assessments before upgrading Istio from a source version to a target version.
*   **Environment Assumptions**: Sidecar mode (Ambient disabled), Multi-Revision deployment, Multi-Primary Multi-Network cluster topology, and Mesh Federation.
*   **Key Assessment Steps**:
    1.  **Cluster & Istio Discovery**: Capture Kubernetes and control plane versions, node/network topologies, and version skews.
    2.  **Inventory & Revision Analysis**: Map active control plane and proxy revisions, identify namespace webhooks, and assess migration feasibility.
    3.  **Compatibility Analysis**: Check Envoy proxy skews, CRD versioning compatibility, and EnvoyFilter deprecations.
    4.  **Network, Security, & Federation**: Verify east-west gateway paths, remote cluster endpoint secrets, and federation controllers/CRDs.
    5.  **Simulation & Strategy**: Plan canary revision deployment, validation checkpoints, failure scenario responses, and rollbacks.

---

## 🚀 How to Use These Plugins

These skills are optimized for integration into agentic workflows or manual change-management checklists.

### For AI/Agent Integrations
1.  Configure the agent workspace to load the plugin from [plugin.json](file:///istio-skills/plugins/istio-upgrade-skill/.tessl-plugin/plugin.json).
2.  Provide the contents of the [SKILL.md](file:///istio-skills/plugins/istio-upgrade-skill/skills/istio-upgrade-skill/SKILL.md) file as system context.
3.  Supply raw cluster dumps (e.g., `kubectl version`, `istioctl proxy-status`) and request a production-grade upgrade risk assessment report.

### For Manual Operations
*   Follow the standardized step-by-step diagnostic workflows in the skill documents to review cluster readiness before applying mesh modifications.

---

## 🤝 Contributing

We welcome additions of new Tessl-compliant plugins or enhancements to existing ones:
1.  Fork this repository.
2.  Create a sub-folder under `plugins/` (e.g., `plugins/istio-ambient-migration`).
3.  Add `.tessl-plugin/plugin.json` describing your plugin.
4.  Add your runbook inside `skills/<your-skill-name>/SKILL.md` using the standard frontmatter structure.
5.  Submit a Pull Request.

---

## 📄 License

This project is licensed under the MIT License. See [LICENSE](file:///Users/shashiprakashgautam/projects/istio-skills/LICENSE) for details.