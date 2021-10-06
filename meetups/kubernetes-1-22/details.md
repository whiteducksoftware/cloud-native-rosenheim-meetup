# Kubernetes 1.22 - Details

## Prerequisites

* Kubernetes 1.22
* containerd >1.5
* feature gate `PodSecurity` enabled on
  * kube-api-server
  * kube-controller-manager
  * kube-scheduler
* feature gate `EphemeralContainers` enabled on
  * kube-api-server
  * kube-controller-manager
  * kube-scheduler
  * kubelet

## Pod Security Admission Control

Example with Namespace labels:

```bash
# Check if feature gate is activated
kubectl -n kube-system get pods -o json | jq -r '.items[].spec.containers[].command'
# Create NS
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: demo-enforce
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: v1.22
---
apiVersion: v1
kind: Namespace
metadata:
  name: demo-audit 
  labels:
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/audit-version: v1.22
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/warn-version: v1.22
EOF
# Create privileged pod
kubectl -n demo-enforce apply -f https://raw.githubusercontent.com/BishopFox/badPods/main/manifests/priv/pod/priv-exec-pod.yaml
# Create privileged pod
kubectl -n demo-audit apply -f https://raw.githubusercontent.com/BishopFox/badPods/main/manifests/priv/pod/priv-exec-pod.yaml
```

Example with AdmissionConfiguration:

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: PodSecurity
  configuration:
    apiVersion: pod-security.admission.config.k8s.io/v1alpha1
    kind: PodSecurityConfiguration
    # Defaults applied when a mode label is not set.
    #
    # Level label values must be one of:
    # - "privileged" (default)
    # - "baseline"
    # - "restricted"
    #
    # Version label values must be one of:
    # - "latest" (default) 
    # - specific version like "v1.22"
    defaults:
      enforce: "privileged"
      enforce-version: "v1.22"
    exemptions:
      # Array of authenticated usernames to exempt.
      usernames: []
      # Array of runtime class names to exempt.
      runtimeClassNames: []
      # Array of namespaces to exempt.
      namespaces: [kube-system]
```

## Ephemeral Containers
### Example
```bash
# start a pod called demo
kubectl run demo --image=google/pause --restart=Never -- sleep 1d
# inject a ephemeral container into the pod
# the target parameter targets the process namespace of another container
kubectl debug demo -it --image=busybox --target=demo
```

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
