# Kubernetes 1.23 - Details

## Server Side Field Validation

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
