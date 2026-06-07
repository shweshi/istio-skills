# EnvoyFilter Risk Analysis Reference

## Commands

```bash
kubectl get envoyfilter -A -o yaml
```

## High-Risk Patterns

### xDS API Version References
Classify as **HIGH RISK** if the filter references any of the following deprecated/removed patterns:

| Pattern | Risk | Notes |
|---------|------|-------|
| `envoy.filters.http.lua` | WARNING | Lua filter stable but Envoy may rename configuration fields |
| `typed_config` with `@type: type.googleapis.com/envoy.config.filter.*v2.*` | CRITICAL | v2 xDS APIs removed; target version will reject |
| `typed_config` with `@type: type.googleapis.com/envoy.extensions.*v3.*` | PASS | v3 is stable |
| `route.v3.RouteAction` fields removed in target | HIGH RISK | Check Envoy changelog for target version |
| `applyTo: HTTP_FILTER` with `name: envoy.router` | WARNING | Renamed to `envoy.filters.http.router` in Envoy 1.14+ |
| `applyTo: NETWORK_FILTER` with custom Wasm plugin | HIGH RISK | Wasm ABI compatibility must be verified per target Envoy version |

### Wasm-Specific Rules
- If `vm_config.runtime: envoy.wasm.runtime.v8` — verify Wasm module was compiled for the target Envoy's V8 ABI version.
- If Wasm module is pulled from OCI: confirm the tag resolves and the image is accessible at upgrade time.
- ABI mismatch causes **silent proxy crash** on first request — classify as **CRITICAL**.

## Decision Logic

1. For each EnvoyFilter: scan `typed_config.@type` for `v2` proto paths → CRITICAL.
2. Check `applyTo` target and `match.listener/routeConfiguration/cluster` against Envoy changelog for target version.
3. If filter uses `name` referencing a built-in filter: verify the name was not renamed in the target Envoy release.
4. If any EnvoyFilter targets `istio-proxy` image and uses version-locked patch values: compare with target proxy config schema.
5. Unknown / unverified → default to **HIGH RISK** (never assume compatibility).

## Risk Classification

| Scenario | Severity |
|----------|----------|
| v2 xDS proto reference | CRITICAL |
| Wasm ABI mismatch | CRITICAL |
| Deprecated filter name | HIGH RISK |
| Removed route action field | HIGH RISK |
| Unverified custom filter | HIGH RISK |
| Stable v3 API, field unchanged | PASS |
