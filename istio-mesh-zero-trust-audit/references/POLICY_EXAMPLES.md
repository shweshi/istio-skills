# Policy Examples

Examples use `security.istio.io/v1`. Adapt namespaces, selectors, ports, identities, and paths
to verified application requirements.

## mTLS

Vulnerable: production namespace accepts plaintext and mTLS.

```yaml
apiVersion: security.istio.io/v1
kind: PeerAuthentication
metadata:
  name: default
  namespace: payments
spec:
  mtls:
    mode: PERMISSIVE
```

Hardened: require mTLS for the namespace.

```yaml
apiVersion: security.istio.io/v1
kind: PeerAuthentication
metadata:
  name: default
  namespace: payments
spec:
  mtls:
    mode: STRICT
```

Validate legacy and health-check traffic before changing modes. In ambient mode, `STRICT`
also helps reject traffic that bypasses mesh capture.

## Authorization Default Deny And Least Privilege

Vulnerable: an empty rule matches all requests.

```yaml
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: allow-everything
  namespace: payments
spec:
  rules:
  - {}
```

Hardened: establish namespace default deny, then allow one verified caller.

```yaml
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: default-deny
  namespace: payments
spec: {}
---
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: allow-checkout-to-payments
  namespace: payments
spec:
  selector:
    matchLabels:
      app: payments-api
  action: ALLOW
  rules:
  - from:
    - source:
        principals:
        - cluster.local/ns/checkout/sa/checkout-api
    to:
    - operation:
        methods: ["POST"]
        paths: ["/v1/charges"]
        ports: ["8080"]
```

An ALLOW policy with no rules denies access to its target. Confirm selector matches and
positive traffic still succeeds.

## JWT Authentication With Authorization

Weak: JWTs are validated when present, but anonymous requests may still be authorized.

```yaml
apiVersion: security.istio.io/v1
kind: RequestAuthentication
metadata:
  name: api-jwt
  namespace: storefront
spec:
  selector:
    matchLabels:
      app: public-api
  jwtRules:
  - issuer: https://identity.example.com/
    jwksUri: https://identity.example.com/.well-known/jwks.json
```

Hardened: require an authenticated principal for the protected route.

```yaml
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: require-jwt
  namespace: storefront
spec:
  selector:
    matchLabels:
      app: public-api
  action: ALLOW
  rules:
  - from:
    - source:
        requestPrincipals: ["https://identity.example.com/*"]
    to:
    - operation:
        methods: ["GET"]
        paths: ["/api/*"]
```

Add separate rules for public health or discovery endpoints when required.

## Safe Deny Rule

Risky: an HTTP-only attribute on TCP traffic can match unexpectedly in a DENY policy.

```yaml
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: risky-deny
  namespace: payments
spec:
  action: DENY
  rules:
  - to:
    - operation:
        methods: ["POST"]
```

Safer: constrain destination ports and workload scope.

```yaml
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: deny-admin-post
  namespace: payments
spec:
  selector:
    matchLabels:
      app: payments-api
  action: DENY
  rules:
  - to:
    - operation:
        methods: ["POST"]
        paths: ["/admin/*"]
        ports: ["8080"]
```

## Dry Run Before Enforcement

```yaml
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: deny-admin-post
  namespace: payments
  annotations:
    istio.io/dry-run: "true"
spec:
  selector:
    matchLabels:
      app: payments-api
  action: DENY
  rules:
  - to:
    - operation:
        methods: ["POST"]
        paths: ["/admin/*"]
        ports: ["8080"]
```

Keep dry run long enough to observe representative traffic. Review both ALLOW and DENY shadow
results, then remove the annotation and repeat positive and negative tests.

## Egress

`REGISTRY_ONLY` blocks unknown destinations handled by the proxy, but is not a firewall.

```yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
spec:
  meshConfig:
    outboundTrafficPolicy:
      mode: REGISTRY_ONLY
```

Pair it with scoped `ServiceEntry` resources, egress-gateway routing where required, and
Kubernetes or infrastructure network controls that block direct bypass.
