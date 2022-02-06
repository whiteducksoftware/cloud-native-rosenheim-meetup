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

Make sure to enable the corresponding feature gate up-front (don't miss it on the Kubelet!).

```bash
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: grpc-health
  name: grpc-health
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grpc-health
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: grpc-health
    spec:
      containers:
      - image: ghcr.io/nmeisenzahl/grpc-health/grpc-health:latest
        imagePullPolicy: Always
        name: grpc-health
        ports:
        - containerPort: 3000
        resources: {}
        livenessProbe:
            grpc:
                port: 3000
            initialDelaySeconds: 10
        readinessProbe:
            grpc:
                port: 3000
            initialDelaySeconds: 10
status: {}
EOF
```
