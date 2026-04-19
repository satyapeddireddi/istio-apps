# Istio Observability & E-Commerce Microservices Lab

This repository contains the consolidated configuration and operational steps for deploying the **Google Online Boutique** microservices demo within an **Istio Service Mesh**. This setup is optimized for learning distributed tracing with **Jaeger**, metrics with **Prometheus/Grafana**, and service graphing with **Kiali**.

---

## 🚀 1. Application Deployment

First, create the namespace and enable Istio's automatic sidecar injection to ensure every microservice is part of the mesh.

```bash
# Create the namespace
kubectl create namespace ecom

# Enable Istio sidecar injection
kubectl label namespace ecom istio-injection=enabled

# Deploy the 11-microservice stack
kubectl apply -f [https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/main/release/kubernetes-manifests.yaml](https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/main/release/kubernetes-manifests.yaml) -n ecom

The complete README.md file for your Istio E-commerce lab is provided below in Markdown format.Markdown# Istio Observability & E-Commerce Microservices Lab

This repository contains the consolidated configuration and operational steps for deploying the **Google Online Boutique** microservices demo within an **Istio Service Mesh**. This setup is optimized for learning distributed tracing with **Jaeger**, metrics with **Prometheus/Grafana**, and service graphing with **Kiali**.

```
#### 🌐 2. Network & External AccessTo access the application from a browser outside the cluster, we patch the services to use NodePorts.Configure Gateway Entrypoint (NodePort 30010)

```bash
kubectl patch svc ecom-gateway-istio -n ecom --type='json' -p='[
  {"op": "replace", "path": "/spec/type", "value": "NodePort"},
  {"op": "replace", "path": "/spec/ports/0/nodePort", "value": 30010}
]'
Configure Direct Frontend Access (NodePort 30011)Bashkubectl patch svc frontend-external -n ecom --type='json' -p='[
  {"op": "replace", "path": "/spec/type", "value": "NodePort"},
  {"op": "replace", "path": "/spec/ports/0/nodePort", "value": 30011}
]'
```

#### 📊 3. Observability SetupTraffic GenerationObservability tools require active traffic to populate data. Use this loop to simulate continuous user activity.Bash# Continuous loop with status code and latency tracking

```bash

while true; do 
  curl -s -o /dev/null -w "%{http_code} - %{time_total}s\\n" [http://172.29.8.197:30010/](http://172.29.8.197:30010/)
  sleep 0.2
done

```

#### Jaeger Data Source (Grafana Configuration)When adding Jaeger as a Data Source in Grafana, use the internal ClusterIP URL:
Name: 
JaegerURL: http://tracing.istio-system:80Query Timeout: 60s🛠 
4. Chaos Engineering (Fault Injection)Test your observability dashboards by injecting a 5-second delay into the Product Catalog service.

```bash 
cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: productcatalog-delay
  namespace: ecom
spec:
  hosts:
  - productcatalogservice
  http:
  - fault:
      delay:
        percentage:
          value: 100.0
        fixedDelay: 5s
    route:
    - destination:
        host: productcatalogservice
EOF
```
#### To Undo the Fault Injection:
```bash
kubectl delete virtualservice productcatalog-delay -n ecom
```
```bash

🔗 5. Service Map & Access URLs

ComponentAccess URL
PurposeStorefronthttp://172.29.8.197:30010
Main E-com UIJaeger UIhttp://172.29.8.197:30009
Distributed Tracing UI
Grafana http://localhost:30007Metrics 
& DashboardsKiali http://localhost:30005
Service Mesh Topology

```