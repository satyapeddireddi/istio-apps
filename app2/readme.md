```bash
kubectl create namespace istio-lab
kubectl label namespace istio-lab istio-injection=enabled

# Deploy Frontend, Backend, and a mock External Service
kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/httpbin/httpbin.yaml -n istio-lab
# (Assume frontend and backend deployments are applied here)
```

```bash
kubectl patch svc frontend-svc -n istio-lab --type='json' -p='[
  {"op": "replace", "path": "/spec/type", "value": "NodePort"},
  {"op": "replace", "path": "/spec/ports/0/nodePort", "value": 30012}
]'
```
```bash
kubectl patch svc gateway-istio -n istio-lab --type='json' -p='[
  {"op": "replace", "path": "/spec/type", "value": "NodePort"},
  {"op": "replace", "path": "/spec/ports/1/nodePort", "value": 30010}
]'
```