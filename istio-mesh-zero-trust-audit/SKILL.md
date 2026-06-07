---
name: istio-zero-trust-audit
description: Use when the user asks to audit, review, assess, or harden an Istio service mesh; investigate Zero Trust, mTLS, identity, authorization, lateral movement, ingress, egress, trust-domain, or ambient-mesh concerns; or prepare service-mesh security and compliance evidence. Produces evidence-backed findings, attack paths, maturity and confidence scores, and a prioritized remediation plan.
---

# Istio Zero Trust Audit

Audit actual enforcement, not configuration intent. Never invent cluster state or treat a
resource's existence as proof that it applies to traffic.

## Inputs

Accept live-cluster access, command output, manifests, architecture diagrams, traffic
requirements, or any combination. Establish:

- Istio and Kubernetes versions
- sidecar, ambient, or mixed data plane
- mesh root namespace and trust domain
- namespaces and workloads in scope
- ingress, egress, multi-cluster, and external authorization architecture
- regulatory or business constraints that change severity

If live access is available, read
[`references/EVIDENCE_COLLECTION.md`](references/EVIDENCE_COLLECTION.md) and collect the
minimum required evidence. If only supplied artifacts are available, inventory what is present
and mark missing evidence as `UNKNOWN`.

## Evidence Rules

- Label every conclusion `VERIFIED`, `INFERRED`, or `UNKNOWN`.
- Cite the resource, namespace, selector, workload, command output, or traffic test behind it.
- Resolve policy precedence and selector/`targetRefs` scope before judging coverage.
- Separate control-plane configuration from observed data-plane enforcement.
- Do not call `REGISTRY_ONLY` an egress firewall; it only controls unknown destinations known
  to the proxy and can be bypassed by traffic outside mesh capture.
- Do not call default ServiceAccount usage critical by itself. Raise severity when it creates
  shared identity, broad authorization, or a demonstrated attack path.
- Treat secrets, tokens, private keys, and full certificate material as sensitive; request
  metadata or redacted output.

## Workflow

### 1. Baseline The Mesh

Map control-plane revisions, injection or ambient enrollment, gateways, waypoint coverage,
proxy health, trust domains, and mesh-wide settings. Run static analysis before interpreting
individual policies.

**Checkpoint:** Stop and report `UNKNOWN` coverage if the workload inventory cannot be mapped
to a data plane. Do not score unenrolled workloads as protected.

### 2. Assess Identity And mTLS

Map workloads to ServiceAccounts and SPIFFE identities. Flag unrelated workloads sharing an
identity, production use of default identities when permissions are broad, trust-domain
alias expansion, certificate errors, and unexpected plaintext paths.

Resolve effective `PeerAuthentication` from mesh to namespace to workload and port. In ambient
mode, account for ztunnel mTLS and use `STRICT` to identify bypass paths; `DISABLE` is not
supported for ambient workloads.

**Checkpoint:** Before authorization analysis, document effective mTLS for every assessed
source/destination pair. Principal- and namespace-based authorization conclusions are
unreliable where peer identity is unavailable.

### 3. Assess Authentication And Authorization

Map `RequestAuthentication` and `AuthorizationPolicy` to workloads. Check:

- namespaces or sensitive workloads without an intentional default-deny posture
- empty rules, wildcard principals/namespaces/hosts, and broad service-account grants
- JWT validation without corresponding authorization requirements
- DENY rules that omit ports for TCP traffic or rely on HTTP-only attributes
- `CUSTOM` providers that are absent, fail open, or are not applied as intended
- selector or `targetRefs` mistakes, revision skew, and policies with no matching workload

Use [`references/POLICY_EXAMPLES.md`](references/POLICY_EXAMPLES.md) for complete vulnerable
and hardened examples. Recommend dry-run and traffic tests before enforcing disruptive policy.

**Checkpoint:** Compare intended communication with effective ALLOW, DENY, CUSTOM, and AUDIT
behavior. If policy intent is unavailable, classify excess access as probable rather than
verified.

### 4. Assess Segmentation And Attack Paths

Build a source-to-destination matrix for production, development, platform, monitoring, and
administrative workloads. Verify representative allowed and denied flows.

Construct plausible lateral-movement paths from exposed or lower-trust workloads to sensitive
assets. For each hop, record required identity, protocol, policy decision, and uncertainty.
Revisit identity or authorization findings when a path contradicts earlier assumptions.

### 5. Assess Ingress And Egress

For ingress, review gateway listeners, TLS modes, credential references, wildcard hosts,
routing reachability, client authentication, and gateway authorization.

For egress, review mesh and `Sidecar` outbound modes, `ServiceEntry` scope, egress gateways,
DNS behavior, network policies, and direct-IP or host-network bypasses. Verify whether external
destinations are merely registered, routed through a gateway, or actually restricted by an
independent network control.

### 6. Assess Ambient And Multi-Cluster Boundaries

When ambient is present, map namespace enrollment, ztunnel health, waypoint attachment, and
which policies require L7 enforcement. Identify traffic that receives only L4 enforcement.

For multi-cluster meshes, verify trust anchors, trust-domain aliases, east-west gateway
exposure, remote-cluster secrets, identity uniqueness, and cross-cluster policy assumptions.

### 7. Validate Findings

Run static analysis, inspect effective proxy configuration, and execute representative positive
and negative traffic tests where permitted. A negative test must show that forbidden traffic
fails; a positive test must ensure remediation does not break required traffic.

When evidence conflicts:

1. Prefer observed data-plane behavior over intended configuration.
2. Check revision, selector, attachment, and stale-proxy causes.
3. Keep the finding open and lower confidence until reconciled.

## Severity And Scoring

Read [`references/SCORING_AND_MATURITY.md`](references/SCORING_AND_MATURITY.md) before assigning
severity, Zero Trust score, maturity level, or confidence. Severity depends on reachability,
asset sensitivity, exploit prerequisites, blast radius, and compensating controls.

Do not award maturity credit for controls that are configured but not verified on in-scope
workloads.

## Output

Use [`references/OUTPUT_TEMPLATE.md`](references/OUTPUT_TEMPLATE.md). Every finding must include:

- severity and confidence
- affected assets
- evidence and enforcement status
- attack scenario and business impact
- specific remediation
- validation and rollback steps

Prioritize immediate containment, then 30-day and 90-day work. State assumptions, evidence
gaps, residual risk, and the tests required to close each unknown.
