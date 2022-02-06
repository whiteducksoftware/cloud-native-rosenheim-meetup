# Kubernetes 1.23 - Details

## Server Side Field Validation

Make sure to enable the corresponding feature gate upfront.

```bash
kubectl proxy &
```

```bash
curl -k -X POST -H 'Content-Type: application/json' --data '
{
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
        "name": "nginx-1"
    },
    "spec": {
        "containers": [
            {
                "name": "nginx",
                "image": "nginx:1.21",
                "unknownField": "i-was-here",
                "ports": [
                    {
                        "containerPort": 80
                    }
                ]
            }
        ]
    }
}
' "http://127.0.0.1:8001/api/v1/namespaces/default/pods"
```

```bash
kubectl get pods nginx-1 -oyaml
```

```bash
curl -k -X POST -H 'Content-Type: application/json' --data '
{
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
        "name": "nginx-2"
    },
    "spec": {
        "containers": [
            {
                "name": "nginx",
                "image": "nginx:1.21",
                "unknownField": "i-was-here",
                "ports": [
                    {
                        "containerPort": 80
                    }
                ]
            }
        ]
    }
}
' "http://127.0.0.1:8001/api/v1/namespaces/default/pods?fieldValidation=Strict"

```

## kubectl events

Currently in alpha and therefore available within the `kubectl alpha` sub-command.

```bash
kubectl alpha events -A

kubectl alpha events --for pods/nginx-1 -n default --watch
```


## Auto remove PVCs created by StatefulSet

tpd

## gRPC probes

Make sure to enable the corresponding feature gate upfront (don't miss it on the Kubelet!).

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: grpc-health-probe
spec:
  containers:
  - name: etcd
    image: k8s.gcr.io/etcd:3.5.1-0
    command: [ "/usr/local/bin/etcd", "--data-dir",  "/var/lib/etcd", "--listen-client-urls", "http://0.0.0.0:2379", "--advertise-client-urls", "http://127.0.0.1:2379", "--log-level", "debug"]
    ports:
    - containerPort: 2379
    livenessProbe:
      grpc:
        port: 2379
      initialDelaySeconds: 10
    readinessProbe:
      grpc:
        port: 2379
      initialDelaySeconds: 10
EOF
```