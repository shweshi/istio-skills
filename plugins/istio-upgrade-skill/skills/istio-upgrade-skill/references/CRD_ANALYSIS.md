# CRD Compatibility Reference

## Commands

```bash
kubectl get crd -o json | jq '.items[] | select(.metadata.name | contains("istio.io")) | {name: .metadata.name, stored: .spec.versions[] | select(.storage == true) | .name, served: [.spec.versions[] | select(.served == true) | .name]}'
```

## CRD Inventory Checklist

| CRD | Risk Area |
|-----|-----------|
| `virtualservices.networking.istio.io` | HTTP routing behaviour changes |
| `destinationrules.networking.istio.io` | Load balancing / circuit breaker changes |
| `envoyfilters.networking.istio.io` | xDS API breakage — highest risk |
| `gateways.networking.istio.io` | Ingress/egress config changes |
| `serviceentries.networking.istio.io` | External service discovery |
| `peerauthentications.security.istio.io` | mTLS mode enforcement changes |
| `authorizationpolicies.security.istio.io` | Access control breakage |
| `requestauthentications.security.istio.io` | JWT validation changes |
| `telemetries.telemetry.istio.io` | Observability pipeline changes |
| `wasmplugins.extensions.istio.io` | Wasm runtime compatibility |
| `sidecars.networking.istio.io` | Proxy scope changes |
| `workloadentries.networking.istio.io` | VM workload registration |
| `proxyconfigs.networking.istio.io` | Per-proxy overrides |

## Decision Logic

1. For each CRD, compare `spec.versions[*].storage: true` between source and target Istio release.
2. If stored version changes (e.g. `v1alpha3` → `v1beta1`): existing stored objects must be migrated — classify as **HIGH RISK** if a conversion webhook is not in place.
3. If a served version is **removed** in the target (check upgrade notes): any in-flight controllers or GitOps tools using that version will break — classify as **CRITICAL**.
4. Validation rule additions in target may reject previously-valid resources on first `kubectl apply` — classify as **WARNING**.

## Risk Classification

| Scenario | Severity |
|----------|----------|
| Stored API version removed | CRITICAL |
| Served version removed, version still in use by controllers | CRITICAL |
| Storage version changed, conversion webhook absent | HIGH RISK |
| New validation rules added that existing resources violate | HIGH RISK |
| New required fields added (with defaults) | WARNING |
| Cosmetic schema changes, all versions still served | PASS |
