# Proxy Compatibility Reference

## Version Skew Classification

| Skew | Classification |
|------|---------------|
| Same minor version | PASS |
| +1 minor version | GOOD |
| +2 minor versions | WARNING |
| +3 minor versions | HIGH RISK — rolling restart required before cutover |
| > +3 minor versions | CRITICAL — upgrade blocked; must stage through intermediate versions |

**Rule**: Istio supports N-1 proxy/control-plane skew officially. N-2 is tolerated but unsupported. Beyond N-2, proxies may fail to connect to istiod.

## Commands

```bash
istioctl proxy-status
kubectl get pods -A -o jsonpath='{range .items[*]}{.metadata.namespace}/{.metadata.name}: {.spec.containers[*].image}{"\n"}{end}' | grep istio-proxy
```

## Decision Logic

1. Extract oldest proxy version from `istioctl proxy-status`.
2. Compute skew = `target_minor - oldest_proxy_minor`.
3. Apply the classification table above.
4. If skew ≥ 3: **block upgrade** — require rolling restart of workloads first to bring proxies to N-1.
5. If any proxy version predates the source control plane: classify as CRITICAL (stale proxy).

## Risk Triggers

- Pods with `istio-proxy` image from a revision that no longer has a live istiod → proxy cannot reconnect after control plane upgrade.
- DaemonSets (e.g. istio-cni) pinned to old node image → node-level injection failure after upgrade.
- Jobs/CronJobs with `istio-proxy` sidecars that are not restarted → permanent skew after upgrade.
