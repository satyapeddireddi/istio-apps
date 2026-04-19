```bash
kubectl create namespace istio-lab
kubectl label namespace istio-lab istio-injection=enabled

# Deploy Frontend, Backend, and a mock External Service
kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/httpbin/httpbin.yaml -n istio-lab
# (Assume frontend and backend deployments are applied here)
```
