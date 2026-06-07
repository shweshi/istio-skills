# Proxy Configuration Observations

## istioctl proxy-config listeners catalog-7f9d-x2kp1.storefront

Observed listener configuration for the catalog pod shows:
- Port 8080: TLS mode = DISABLE (plaintext accepted)
- FilterChain: no mTLS filter present
- Note: Pod label shows `istio.io/rev: canary` (different from the control plane revision `stable`)

## istioctl x describe pod catalog-7f9d-x2kp1 -n storefront

Output excerpt:
```
Pod is NOT part of mesh.
No Istio sidecar found.
```

Wait — this contradicts the next observation. This pod appears to have a proxy container but istioctl does not recognize it as part of the mesh.

## istioctl proxy-config clusters cart-6bc4-j3mn2.storefront

Output shows catalog service endpoint:
- Destination: catalog.storefront.svc.cluster.local:8080
- TLS mode: DISABLE (plaintext)
- UpstreamTlsContext: absent

## kubectl get pod catalog-7f9d-x2kp1 -n storefront -o yaml (excerpt)

```yaml
metadata:
  labels:
    app: catalog
    istio.io/rev: canary
  annotations:
    sidecar.istio.io/status: '{"revision":"canary"}'
spec:
  containers:
  - name: catalog
    image: catalog:v2.1
  - name: istio-proxy
    image: docker.io/istio/proxyv2:1.20.0
```

## kubectl get mutatingwebhookconfiguration (excerpt)

Only `istio-revision-tag-stable` webhook is present. No `istio-revision-tag-canary` or `istiod-canary` webhook found.
