# Istio Skills

Welcome to **istio-skills**—a curated repository of expert-level system prompts, instructions, and runbooks designed for AI agents, platform engineers, and mesh administrators operating Istio service meshes in production.

This repository compiles formal assessment guidelines, operational scripts, and verification procedures to ensure stable, secure, and reliable service mesh management.

---

## 🛠️ Available Skills

### 1. [istio-canary-upgrade](file:///Users/shashiprakashgautam/projects/istio-skills/istio-canary-upgrade/SKILL.md)

A comprehensive framework to guide a Principal Istio Service Mesh Architect, Kubernetes Platform Engineer, and Production Reliability Reviewer through a multi-revision canary upgrade process.

*   **Objective**: Perform compatibility, readiness, and risk assessments before upgrading Istio from a source version to a target version.
*   **Mode**: Sidecar mode with Multi-Revision deployment, Multi-Primary Multi-Network topology, and Mesh Federation.
*   **Key Steps Covered**:
    1.  **Cluster Discovery**: Check Kubernetes versions, node wide output, api-resources.
    2.  **Istio Inventory**: Control plane deployment, proxy-status check, and revision mapping.
    3.  **Compatibility Checks**: Envoy proxy skew, CRD serve/storage versions, EnvoyFilter analysis.
    4.  **Network & Security Review**: East-west gateway stability, cross-cluster endpoint discovery, federation compatibility, PeerAuthentication, and AuthorizationPolicies.
    5.  **Simulated Execution**: Detailed step-by-step canary execution planning and rollback scenario modeling.
    6.  **Risk Matrix & Scoring**: Standard templates for calculating a Readiness Score (0-100) and a Confidence Score.

For detailed guidelines, view the [SKILL.md](file:///Users/shashiprakashgautam/projects/istio-skills/istio-canary-upgrade/SKILL.md) file.

---

## 🚀 How to Use These Skills

These templates are designed to be ingested by AI agents (like Antigravity or other LLMs) or followed manually by platform engineers.

### For AI Agents
1.  Provide the contents of the [SKILL.md](file:///Users/shashiprakashgautam/projects/istio-skills/istio-canary-upgrade/SKILL.md) file as system instructions or few-shot context.
2.  Provide your cluster information (output of the commands listed in Step 1 & 2 of the skill).
3.  Ask the agent to generate a production-ready **Upgrade Readiness Assessment** following the mandatory rules.

### For Platform Engineers
*   Use the checklists and CLI commands in [SKILL.md](file:///Users/shashiprakashgautam/projects/istio-skills/istio-canary-upgrade/SKILL.md) as a standardized checklist during change reviews or upgrade planning meetings.

---

## 🤝 Contributing

Contributions of new skills or improvements to existing ones are highly welcome!
1.  Fork this repository.
2.  Create a directory for your skill (e.g., `istio-ambient-migration`).
3.  Add a `SKILL.md` file following the template structure.
4.  Submit a Pull Request.

---

## 📄 License

This project is licensed under the MIT License. See [LICENSE](file:///Users/shashiprakashgautam/projects/istio-skills/LICENSE) for details.