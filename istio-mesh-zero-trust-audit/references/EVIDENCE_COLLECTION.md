# Evidence Collection

Use read-only commands first. Narrow `-A` queries when the audit scope is smaller. Redact
tokens, private keys, credential data, and certificate bodies before sharing output.

## Baseline

```bash
kubectl version
istioctl version
istioctl proxy-status
istioctl analyze --all-namespaces
kubectl get namespaces --show-labels
kubectl get pods -A -o custom-columns='NAMESPACE:.metadata.namespace,NAME:.metadata.name,SERVICE_ACCOUNT:.spec.serviceAccountName,REVISION:.metadata.labels.istio\.io/rev,AMBIENT:.metadata.labels.istio\.io/dataplane-mode,PROXY:.spec.containers[*].name'
```

Capture Istio revisions and control-plane configuration:

```bash
kubectl get deploy,daemonset -A -l app=istiod -o wide
kubectl get mutatingwebhookconfiguration,validatingwebhookconfiguration
kubectl get configmap -n istio-system istio -o yaml
kubectl get configmap -n istio-system istio-sidecar-injector -o yaml
```

The control-plane namespace or ConfigMap names may differ. Record that as an evidence gap
instead of guessing.

## Identities And Security Policies

```bash
kubectl get serviceaccount -A
kubectl get peerauthentication,requestauthentication,authorizationpolicy -A -o yaml
kubectl get networkpolicy -A -o yaml
kubectl get pod -A -o json
```

For a representative workload, inspect effective state:

```bash
istioctl x describe pod <pod> -n <namespace>
istioctl proxy-config secret <pod> -n <namespace>
istioctl proxy-config listeners <pod> -n <namespace>
istioctl proxy-config clusters <pod> -n <namespace>
```

Use certificate metadata only. Do not include private key material.

## Ingress, Egress, And Trust Boundaries

```bash
kubectl get gateway,virtualservice,destinationrule,serviceentry,sidecar -A -o yaml
kubectl get gateways.gateway.networking.k8s.io,httproutes.gateway.networking.k8s.io,grpcroutes.gateway.networking.k8s.io -A -o yaml
kubectl get svc -A -o wide
kubectl get deploy,daemonset -A -o yaml
```

Search the mesh configuration for these fields:

```bash
kubectl get configmap -n istio-system istio -o yaml | grep -E 'rootNamespace|trustDomain|trustDomainAliases|outboundTrafficPolicy|extensionProviders'
```

## Ambient Mesh

```bash
kubectl get namespaces --show-labels
kubectl get pods -n istio-system -l app=ztunnel -o wide
kubectl get gateways.gateway.networking.k8s.io -A -o yaml
istioctl proxy-status
```

Map namespace enrollment, pod-level overrides, waypoint resources, and waypoint attachment.
Do not infer L7 enforcement from ambient enrollment alone.

## Validation Tests

For each critical path, define one expected allow and one expected deny from a real source
workload:

```bash
kubectl exec -n <source-ns> <source-pod> -c <app-container> -- \
  curl -sS -o /dev/null -w '%{http_code}\n' \
  http://<service>.<destination-ns>.svc.cluster.local:<port>/<path>
```

Prefer an existing diagnostic workload. Do not deploy tools or generate production traffic
without authorization. Capture timestamp, source identity, destination, expected result, and
actual result.

For authorization changes, use the `istio.io/dry-run: "true"` annotation where supported,
inspect shadow-policy logs/metrics, then repeat positive and negative tests before enforcement.

## Completion Check

Evidence is sufficient only when:

- every in-scope workload maps to sidecar, ambient, gateway, or unenrolled status
- effective mTLS and authorization are known for critical communication paths
- ingress and egress paths include non-Istio bypass controls
- critical findings have either observed behavior or a clearly marked inference
- contradictory analyzer, control-plane, and data-plane evidence is reconciled
