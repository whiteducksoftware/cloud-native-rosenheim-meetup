# Kubernetes 1.22 - Details

## Pod Security Admission Control

tbd

## Ephemeral Containers

tbd

## Verify your API versions

We will use `kubeconform` to validate our manifest.

Sample manifest with outdated API (`legacy-ingress.yaml`):

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          serviceName: test
          servicePort: 80
```

Code snippets to validate:

```bash
kubeconform -kubernetes-version=1.18.0 legacy-ingress.yaml
kubeconform -kubernetes-version=1.22.0 legacy-ingress.yaml
```

## kubectl convert

You can download the `kubectl convert` plugin [here](https://www.downloadkubernetes.com).

Code snippets to convert outdated manifest:

```bash
kubectl convert -f legacy-ingress.yaml --output-version networking.k8s.io/v1
```
