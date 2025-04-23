# 🐳 Kubernetes-Based Canary Deployment with K3s and Istio

This project demonstrates how to perform **canary deployments** in a lightweight Kubernetes environment using **K3s** and **Istio**. It simulates real-world scenarios where traffic is gradually shifted between stable and new versions of an application.

---

## 🚀 Project Overview

- **K3s**: A lightweight Kubernetes distribution ideal for development and testing.
- **Istio**: A service mesh that provides advanced traffic management, security, and observability features.
- **Canary Deployment**: A strategy that introduces a new version of an application to a subset of users to mitigate risks and ensure stability.

---

## 📁 Repository Structure

```
├── gateway/                   # Istio Gateway configurations
├── istio/                     # Istio installation manifests and configurations
├── metrics/                   # Monitoring setup with Prometheus and Grafana
├── screenshots/               # Visual documentation of the deployment
├── split/                     # Traffic splitting configurations for canary deployments
└── README.md                  # Project documentation
```

---

## 🛠️ Prerequisites

- **Operating System**: Windows with WSL2 enabled
- **K3s**: Installed via [k3sup](https://github.com/alexellis/k3sup) or official script
- **kubectl**: Command-line tool for interacting with Kubernetes clusters
- **Helm**: Kubernetes package manager
- **Istioctl**: Command-line tool for Istio operations
- **Git**: Version control system

---

## 🔧 Installation Steps

### 1. Clone the Repository

```bash
git clone https://github.com/lokesh-matha/Kubernetes-Based-Canary-Deployment-with-K3s-and-Istio.git
cd Kubernetes-Based-Canary-Deployment-with-K3s-and-Istio
```

### 2. Install K3s

```bash
curl -sfL https://get.k3s.io | sh -
```

Ensure `kubectl` is configured to interact with your K3s cluster.

### 3. Install Istio

```bash
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH=$PWD/bin:$PATH
istioctl install --set profile=demo -y
```

### 4. Deploy Sample Applications

```bash
kubectl apply -f split/stable-deployment.yaml
kubectl apply -f split/canary-deployment.yaml
```

### 5. Configure Istio Gateway and Virtual Services

```bash
kubectl apply -f gateway/gateway.yaml
kubectl apply -f split/virtual-service.yaml
```

### 6. Deploy Monitoring Tools

```bash
kubectl apply -f metrics/prometheus.yaml
kubectl apply -f metrics/grafana.yaml
```

To access Grafana:

```bash
kubectl port-forward svc/grafana 3000:3000 -n monitoring
```

Then visit [http://localhost:3000](http://localhost:3000) in your browser.

---
Absolutely! Here's your full list of **Istio + K3s Canary Deployment commands** in regular, clean format (not inside a code block), ready to read, copy, and use:

---

### 🛠️ Setup and Gateway Deployment

1. View the Istio Gateway YAML:
   ```
   cat k8s/istio/gateway/app/istio.yaml
   ```

2. Create a namespace and enable Istio injection:
   ```
   kubectl create namespace go-demo-7
   kubectl label namespace go-demo-7 istio-injection=enabled
   ```

3. Apply the Istio gateway configuration:
   ```
   kubectl --namespace go-demo-7 apply --filename k8s/istio/gateway --recursive
   ```

4. Wait for the primary deployment rollout:
   ```
   kubectl --namespace go-demo-7 rollout status deployment go-demo-7-primary
   ```

5. Check resources:
   ```
   kubectl --namespace go-demo-7 get pods
   kubectl --namespace go-demo-7 get virtualservices
   kubectl --namespace go-demo-7 describe virtualservice go-demo-7
   ```

6. Run a quick curl test from an Alpine pod:
   ```
   kubectl run curl --image alpine -it --rm -- sh -c "apk add -U curl && curl go-demo-7.go-demo-7/demo/hello"
   ```

7. View gateway and ingress:
   ```
   kubectl --namespace go-demo-7 get ingress
   kubectl --namespace go-demo-7 get gateways
   kubectl --namespace go-demo-7 describe gateway go-demo-7
   ```

---

### 🌐 Ingress Configuration

- **If using Minikube**:
   ```
   export INGRESS_PORT=$(kubectl --namespace istio-system get service istio-ingressgateway --output jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
   export INGRESS_HOST=$(minikube ip):$INGRESS_PORT
   ```

- **If using EKS**:
   ```
   export INGRESS_HOST=$(kubectl --namespace istio-system get service istio-ingressgateway --output jsonpath="{.status.loadBalancer.ingress[0].hostname}")
   ```

- Confirm the host:
   ```
   echo $INGRESS_HOST
   ```

- Test access:
   ```
   curl -v -H "Host: go-demo-7.acme.com" "http://$INGRESS_HOST/demo/hello"
   curl -v -H "Host: something-else.acme.com" "http://$INGRESS_HOST/demo/hello"
   ```

- Delete gateway configuration:
   ```
   kubectl --namespace go-demo-7 delete --filename k8s/istio/gateway --recursive
   ```

---

### 🚀 First Release Testing

```
for i in {1..10}; do 
    curl -H "Host: go-demo-7.acme.com" "http://$INGRESS_HOST/version"
done
```

---

### 🆕 Canary Release

1. Review and compare manifests:
   ```
   cat istio/split/exercise/app-0-0-2-canary.yaml
   diff istio/gateway/app/deployment.yaml istio/split/exercise/app-0-0-2-canary.yaml
   ```

2. Deploy canary version:
   ```
   kubectl --namespace go-demo-7 apply --filename istio/split/exercise/app-0-0-2-canary.yaml
   kubectl --namespace go-demo-7 rollout status deployment go-demo-7-canary
   ```

3. Load test:
   ```
   for i in {1..100}; do 
       curl -H "Host: go-demo-7.acme.com" "http://$INGRESS_HOST/version"
   done
   ```

4. Validate state:
   ```
   kubectl --namespace go-demo-7 get deployments
   kubectl --namespace go-demo-7 describe service go-demo-7
   kubectl --namespace go-demo-7 describe virtualservice go-demo-7
   kubectl --namespace go-demo-7 describe gateway go-demo-7
   ```

---

### 🔀 Splitting Traffic

1. Apply host config:
   ```
   cat istio/split/exercise/host20.yaml
   kubectl --namespace go-demo-7 apply --filename istio/split/exercise/host20.yaml
   ```

2. Test:
   ```
   for i in {1..100}; do 
       curl -H "Host: go-demo-7.acme.com" "http://$INGRESS_HOST/version"
   done
   ```

3. Remove host routing:
   ```
   kubectl --namespace go-demo-7 delete --filename istio/split/exercise/host20.yaml
   ```

4. Apply split config (must total 100%):
   ```
   cat istio/split/exercise/split20.yaml
   kubectl --namespace go-demo-7 apply --filename istio/split/exercise/split20.yaml
   ```

5. Test again:
   ```
   for i in {1..100}; do 
       curl -H "Host: go-demo-7.acme.com" "http://$INGRESS_HOST/version"
   done
   ```

---

### 🔁 Rolling Forward

1. Apply increased traffic splits:
   ```
   cat istio/split/exercise/split40.yaml
   kubectl --namespace go-demo-7 apply --filename istio/split/exercise/split40.yaml
   ```

   ```
   cat istio/split/exercise/split60.yaml
   kubectl --namespace go-demo-7 apply --filename istio/split/exercise/split60.yaml
   ```

2. Test traffic after each:
   ```
   for i in {1..100}; do 
       curl -H "Host: go-demo-7.acme.com" "http://$INGRESS_HOST/version"
   done
   ```

---

### ✅ Finalize Deployment

1. Deploy the stable version of new release:
   ```
   cat istio/split/exercise/app-0-0-2.yaml
   diff istio/gateway/app/deployment.yaml istio/split/exercise/app-0-0-2.yaml
   kubectl --namespace go-demo-7 apply --filename istio/split/exercise/app-0-0-2.yaml
   kubectl --namespace go-demo-7 rollout status deployment go-demo-7-primary
   ```

2. Load test final state:
   ```
   for i in {1..100}; do 
       curl -H "Host: go-demo-7.acme.com" "http://$INGRESS_HOST/version"
   done
   ```

3. Route all traffic to new version:
   ```
   cat istio/split/exercise/split100.yaml
   kubectl --namespace go-demo-7 apply --filename istio/split/exercise/split100.yaml
   ```

4. Confirm deployment:
   ```
   for i in {1..100}; do 
       curl -H "Host: go-demo-7.acme.com" "http://$INGRESS_HOST/version"
   done
   ```

5. View final deployments:
   ```
   kubectl --namespace go-demo-7 get deployments
   ```

---
## 📸 Screenshots

Check the [`screenshots/`](./screenshots/) directory for visual references on:

- Deployment steps
- Traffic distribution
- Monitoring dashboards

---

## 🧹 Cleanup

To remove all deployed resources:

```bash
kubectl delete -f split/
kubectl delete -f gateway/
kubectl delete -f metrics/
istioctl uninstall --purge
```

---

## 📚 References

- [Istio Canary Deployments](https://istio.io/latest/docs/tasks/traffic-management/canary-deploy/)
- [Canary Deployment with Istio in Kubernetes](https://istio.io/latest/blog/2020/canary-deployments/)
- [Canary Deployment with Kubernetes and Istio - FP Complete](https://tech.fpcomplete.com/blog/canary-deployment-istio)

---

🧠 **Maintained by [lokesh-matha](https://github.com/lokesh-matha)** • Contributions welcome!
