# Deploy Version 1 (Blue)
```bash
# Apply V1 Deployment
kubectl apply -f https://raw.githubusercontent.com/stefanprodan/podinfo/master/kustomize/deployment.yaml -n app

# Apply V1 Service
kubectl apply -f https://raw.githubusercontent.com/stefanprodan/podinfo/master/kustomize/service.yaml -n app

# Deploy Version 2 (Green) with a different color/version label
kubectl -n app create deployment podinfo-v2 --image=ghcr.io/stefanprodan/podinfo:6.3.5 --port=9898
kubectl -n app label deployment podinfo-v2 version=v2
kubectl -n app expose deployment podinfo-v2 --port=9898 --name=podinfo-v2


istioctl proxy-config cluster podinfo-849dbc4d6b-kk97c  -n app
istioctl proxy-config cluster podinfo-v2-644b5b8789-jcd2g  -n app
```

```bash
by deafult podinfo-gaetway-istio svc type is LB
kubectl port-forward svc/podinfo-gateway-istio -n istio-system 8081:80
```

or 

```bash
kubectl patch svc podinfo-gateway-istio -n istio-system -p '{"spec": {"type": "NodePort", "ports": [{"port": 80, "nodePort": 30008}]}}'
```

```bash
kubectl patch svc podinfo-gateway-istio -n app -p '{"spec": {"type": "NodePort", "ports": [{"port": 80, "nodePort": 30008}]}}'
```
```bash
while true; do 
  curl -s http://127.0.0.1:8081/ | grep -E "message|version"
  sleep 0.2
done


while true; do 
  curl -s http://172.29.8.197:30008/ | grep -E "message|version"
  sleep 0.2
done

or

while true; do    curl -s -o /dev/null -w "%{http_code} - %{time_total}s\n" http://172.29.8.197:30008/;   sleep 0.2; done

```
istioctl install --set meshConfig.enableTracing=true \
--set meshConfig.defaultConfig.tracing.sampling=100.0 \
--set meshConfig.defaultConfig.tracing.zipkin.address=jaeger-collector.istio-system:9411

kubectl exec -it jaeger-8489dc7cd5-bst7x -n istio-system -- netstat -tuln | grep 9411

kubectl get endpoints zipkin -n istio-system

kubectl rollout restart deployment istio-ingressgateway -n istio-system
kubectl rollout restart deployment podinfo -n app
kubectl rollout restart deployment podinfo-v2 -n app
