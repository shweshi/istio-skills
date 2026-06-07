# Federation Analysis Reference

## Commands

```bash
kubectl get crd | grep -iE "federation|serviceexport|serviceimport"
kubectl get serviceexport -A -o wide
kubectl get serviceimport -A -o wide
kubectl get all -A | grep -i federation
# If using Sail Operator or Admiral federation:
kubectl get admiral -A 2>/dev/null || true
kubectl get federatedservice -A 2>/dev/null || true
```

## Federation Controller Compatibility

### Known Compatibility Matrix

| Federation Mechanism | Istio Dependency | Risk on Upgrade |
|---------------------|-----------------|----------------|
| MCS (Multi-Cluster Services API) | Kubernetes Gateway API; Istio >= 1.17 | LOW if Kubernetes >= 1.24 |
| Admiral | Istio control plane xDS hooks | HIGH RISK -- verify Admiral release supports target Istio minor version |
| Sail Operator federation | Istio operator CRDs | HIGH RISK -- Sail must be upgraded in sync |
| Custom federation controllers | Unknown | CRITICAL until verified |
| Istio native ServiceEntry-based federation | No external controller | LOW -- verify ServiceEntry CRD schema unchanged |

## Decision Logic

1. Identify federation mechanism from installed CRDs.
2. Check federation controller's release notes for Istio compatibility of the target version.
3. If no official compatibility statement exists -> classify as **HIGH RISK**.
4. Verify exported `ServiceEntry` and imported services still resolve after control plane restart:
   - Run `istioctl proxy-config endpoints <federation-gateway-pod> | grep <exported-service>` before and after upgrade.
5. Check trust bundle exchange: if the root CA changes during upgrade, cross-mesh mTLS will fail until both sides re-exchange certs.

## Trust Bundle Rules

- If upgrading from self-signed Citadel CA to external CA (or vice versa): **CRITICAL** -- cross-mesh trust breaks immediately.
- If CA stays the same but intermediate cert rotates: verify federation peers are configured to accept new intermediate -> **HIGH RISK** without explicit validation.
- If `cacerts` Secret is unchanged: trust bundle exchange is unaffected -> **PASS**.

## Risk Classification

| Scenario | Severity |
|----------|----------|
| Federation controller has no published compatibility with target Istio | CRITICAL |
| Root CA changes during upgrade | CRITICAL |
| Federation controller supports target but requires its own upgrade | HIGH RISK |
| ServiceEntry-based federation with no external controller | WARNING |
| MCS with compatible Kubernetes version | LOW |
| All compatibility verified through release notes | PASS |
