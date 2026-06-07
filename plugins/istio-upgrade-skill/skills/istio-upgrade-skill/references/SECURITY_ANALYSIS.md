# Security Analysis Reference

## Commands

```bash
kubectl get peerauthentication -A -o yaml
kubectl get authorizationpolicy -A -o yaml
kubectl get requestauthentication -A -o yaml
```

## mTLS Compatibility Rules

| Scenario | Severity |
|----------|----------|
| Global STRICT mTLS + proxy skew >= N+2 | HIGH RISK -- old proxy cannot complete mTLS handshake with new istiod cert |
| PERMISSIVE mode across upgrade window | PASS -- plaintext fallback absorbs handshake failures |
| DISABLE on any namespace + cross-namespace AuthorizationPolicy | WARNING -- policy enforcement may change with new proxy |
| Per-port STRICT mode with version-specific cipher changes | HIGH RISK -- verify cipher suite compatibility in target Envoy changelog |

**Rule**: If running STRICT mTLS, rolling restart of all proxies must complete within the N-1 skew window before removing the old revision.

## AuthorizationPolicy Changes by Version

### Known Breaking Patterns (check against target release notes)
- **Istio 1.22+**: `source.principal` matching is case-sensitive by default -- policies using mixed-case SPIFFE URIs may silently stop matching.
- **Istio 1.23+**: `action: CUSTOM` policies require explicit provider configuration in `meshConfig`; missing providers cause DENY by default.
- **Istio 1.24+**: L7 policies on TCP ports are rejected at admission (previously silently ignored) -- may block `kubectl apply` in CI.

## Decision Logic

1. Identify global and per-namespace PeerAuthentication modes.
2. If STRICT mode: verify proxy skew is <= N-1 before cutover or switch to PERMISSIVE for the upgrade window.
3. Audit AuthorizationPolicy `action` types: flag any `CUSTOM` policies and verify provider config exists in meshConfig for target version.
4. Check for `source.principal` patterns containing uppercase SPIFFE URIs if upgrading to 1.22+.
5. Check for L7 policies applied to TCP ports if upgrading to 1.24+.
6. Run `istioctl analyze -A` against target CRDs to surface validation failures before applying.

## JWT / RequestAuthentication Rules

- OIDC issuer URL must remain reachable from all sidecars during upgrade -- not an Istio concern but commonly overlooked during cluster-level changes.
- If JWKs cache TTL is short and OIDC endpoint is unavailable during upgrade: requests fail with 401 -> **WARNING**.
- `forwardOriginalToken: true` behaviour is unchanged across versions -- **PASS**.

## Risk Classification

| Scenario | Severity |
|----------|----------|
| STRICT mTLS + proxy skew > N-1 | HIGH RISK |
| CUSTOM action policy, no provider config | CRITICAL |
| Mixed-case SPIFFE URIs, upgrading to 1.22+ | HIGH RISK |
| L7 policy on TCP port, upgrading to 1.24+ | HIGH RISK |
| PERMISSIVE mode across all namespaces | LOW |
| No AuthorizationPolicies deployed | PASS |
