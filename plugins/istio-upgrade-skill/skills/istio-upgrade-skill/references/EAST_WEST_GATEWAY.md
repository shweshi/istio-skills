# East-West Gateway & Multi-Cluster Reference

## Commands

```bash
# List east-west gateways and their versions
kubectl get deploy -A -o json | jq '.items[] | select(.metadata.name | test("east-west|eastwest")) | {ns: .metadata.namespace, name: .metadata.name, image: .spec.template.spec.containers[0].image}'

# Check remote cluster secrets
kubectl get secret -A -l istio/multiCluster=true -o wide
kubectl get secret -A | grep -E "istio-remote-secret|remote-secret"

# Verify cross-cluster endpoint discovery
istioctl remote-clusters
istioctl proxy-config endpoints <east-west-gw-pod> | grep cross-cluster
```

## Upgrade Ordering Rules

**East-west gateways must be upgraded before or in sync with the control plane.**

| Rule | Reason |
|------|--------|
| Upgrade EW gateways first | Gateway is the TLS termination point for cross-cluster traffic; old gateway cannot decrypt new proxy's mTLS certificates if cipher suites diverge |
| Do not run EW gateways N+2 behind control plane | Discovery push from istiod to gateway uses xDS; large skew causes endpoint sync failures |
| Upgrade EW gateways on all clusters before migrating any namespace | Cross-cluster traffic must remain functional throughout canary phase |

## Decision Logic

1. Identify EW gateway pods: match deployment names containing `east-west` or `eastwest`, or gateways with `topology.istio.io/network` label.
2. Check gateway image version vs. target control plane version.
3. If skew > 1 minor version between EW gateway and target istiod → **HIGH RISK**.
4. Verify `istio-remote-secret` exists for every remote cluster and that the API server endpoint is reachable → missing secret = **CRITICAL** (cluster invisible to control plane).
5. After control plane upgrade, run `istioctl remote-clusters` — all remote clusters must show `SYNCED`. If any shows `TIMEOUT` or `NOT READY` → rollback trigger.

## Multi-Primary Specific Rules

- Each cluster runs its own istiod; upgrade them independently but within the same maintenance window.
- Do not let clusters diverge by more than 1 minor version during the upgrade window.
- Remote secrets reference the API server of each cluster; certificate rotation on the API server during upgrade requires secret refresh.

## Risk Classification

| Scenario | Severity |
|----------|----------|
| EW gateway version skew > N+1 from target | HIGH RISK |
| Remote cluster secret missing or expired | CRITICAL |
| Cross-cluster endpoint count drops after upgrade | HIGH RISK (rollback trigger) |
| EW gateway upgraded before control plane | PASS (correct order) |
| EW gateway upgraded after control plane (skew = 0) | PASS |
| Two clusters diverge by > 1 minor version during upgrade | HIGH RISK |
