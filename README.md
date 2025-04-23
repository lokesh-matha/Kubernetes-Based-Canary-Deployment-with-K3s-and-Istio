Sure! Here's your complete `README.md` in markdown format, complete with emojis and code blocks ready for single-click copy on GitHub:

```markdown
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

## 🚦 Performing a Canary Deployment

### Initial Setup

Ensure the stable version (`v1`) of your application is running.

### Deploy Canary Version

Apply the deployment manifest for the canary version (`v2`).

### Configure Traffic Splitting

Update your `VirtualService` to split traffic:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: your-app
spec:
  hosts:
    - your-app
  http:
    - route:
        - destination:
            host: your-app
            subset: v1
          weight: 90
        - destination:
            host: your-app
            subset: v2
          weight: 10
```

Apply the changes:

```bash
kubectl apply -f split/virtual-service.yaml
```

### Monitor Performance

Use Grafana dashboards to track stability and performance of both versions.

### Gradual Rollout

If `v2` performs well, incrementally increase its traffic share by updating weights in `VirtualService`.

### Full Rollout

Once confident, route 100% of traffic to `v2` and decommission `v1`.

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
```

Let me know if you'd like a badge section or GitHub Actions integration added too!
