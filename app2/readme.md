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
```bash
if its not wokring use annotations 

kubectl annotate gateway gateway-istio  -n istio-lab "networking.istio.io/service-type=NodePort" --overwrite
```
```bash
# Replace 172.29.8.197 with your actual Node IP
while true; do 
  curl -s -o /dev/null -w "Status: %{http_code} | Time: %{time_total}s\n" http://172.29.8.197:30012/
  sleep 0.5
done
```
```bash
# 1. Get a Backend Pod IP
BACKEND_IP=$(kubectl get pod -l app=backend,version=v1 -n istio-lab -o jsonpath='{.items[0].status.podIP}')

# 2. Try to curl it from the Frontend
kubectl exec -it $(kubectl get pod -l app=frontend -n istio-lab -o jsonpath='{.items[0].metadata.name}') -n istio-lab -- curl -I http://$BACKEND_IP:8080/status/200
HTTP/1.1 200 OK
access-control-allow-credentials: true
access-control-allow-origin: *
content-type: text/plain; charset=utf-8
date: Sun, 19 Apr 2026 09:36:18 GMT
x-envoy-upstream-service-time: 1
server: istio-envoy
x-envoy-decorator-operation: backend-svc.istio-lab.svc.cluster.local:80/*
transfer-encoding: chunked
```
```bash
for i in {1..20}; do 
  kubectl exec -it $(kubectl get pod -l app=frontend -n istio-lab -o jsonpath='{.items[0].metadata.name}') -n istio-lab -- curl -s -o /dev/null http://backend-svc.istio-lab.svc.cluster.local/status/200
  echo "Traffic pulse $i sent..."
done
```
or

```bash
while true; do 
  kubectl exec -it $(kubectl get pod -l app=frontend -n istio-lab -o jsonpath='{.items[0].metadata.name}') -n istio-lab -- curl -s -o /dev/null -w "Connect to Backend: %{http_code}\n" http://backend-svc.istio-lab.svc.cluster.local/status/200
  sleep 0.5
done
```

```bash
service entry test
[root@LAPTOP-AV0J9JU7 app2]# istioctl pc clusters frontend-58cdd886f5-2fnv8 -n istio-lab | grep google.com
google.com                                                           443       -          outbound      STRICT_DNS       
[root@LAPTOP-AV0J9JU7 app2]# istioctl pc clusters frontend-58cdd886f5-2fnv8 -n istio-lab | grep hashicorp.com
hashicorp.com                                                        443       -          outbound      STRICT_DNS       
```

```bash
istioctl pc endpoint $(kubectl get pod -l app=frontend -n istio-lab -o jsonpath='{.items[0].metadata.name}') -n istio-lab --cluster "outbound|80||backend-svc.istio-lab.svc.cluster.local"
```



