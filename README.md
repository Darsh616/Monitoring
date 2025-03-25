# Monitoring Setup with Grafana & Prometheus on Minikube

This guide walks through installing and configuring Prometheus & Grafana on Minikube using Helm.

## 📌 Prerequisites
Ensure the following are installed on your system:
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [Helm](https://helm.sh/docs/intro/install/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Chocolatey](https://chocolatey.org/install) (for Windows users)

## 🚀 Installation Steps
### 1️⃣ Install Helm
```sh
choco install kubernetes-helm
```
Or manually install it:
```sh
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### 2️⃣ Add Prometheus Helm Repository
```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### 3️⃣ Install Prometheus Stack
```sh
helm install monitoring prometheus-community/kube-prometheus-stack
```

## 🔍 Verify Installation
Check the services:
```sh
kubectl get svc -n default
```
Look for `monitoring-grafana` and `monitoring-kube-prometheus-prometheus-esvc` services.

## 🛠 Accessing Grafana

### 1️⃣ Check Service Type
```sh
kubectl get svc monitoring-grafana
```
If the type is **ClusterIP**, change it.

### 2️⃣ Change Service Type to NodePort
```sh
kubectl edit svc monitoring-grafana
```
Modify the `spec` section:
```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 3000
      nodePort: 32045
```
Save and exit the editor.

### 3️⃣ Get Minikube IP
```sh
minikube ip
```
Example output:
```
192.168.49.2
```

### 4️⃣ Access Grafana
Try accessing via terminal:
```sh
curl http://192.168.49.2:32045
```
Or open in your browser:
```
http://192.168.49.2:32045
```

### 5️⃣ Alternative Shortcut
```sh
minikube service monitoring-grafana --url
```
This will automatically open the Grafana UI in your default browser.

## 🔑 Retrieving Grafana Admin Password
Grafana credentials are stored in a secret. Get the password using:
```sh
kubectl get secret -n default monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```
Default username: **admin**

## 🛠️ Troubleshooting

### 🚨 Problem: Minikube Can't Access NodePort
- **Reason:** Windows host cannot access Minikube’s VM private IP.
- **Fix:** Use Minikube's service command:
  ```sh
  minikube service monitoring-grafana --url
  ```

### 🚨 Problem: "Connection refused" when querying Prometheus API
- **Fix:** Ensure Prometheus is running and accessible.
  ```sh
  kubectl port-forward svc/monitoring-kube-prometheus-prometheus 9090 -n default
  ```
  Access Prometheus at:
  ```
  http://localhost:9090
  ```

### 🚨 Problem: External access still not working?
Try running:
```sh
minikube tunnel
```
This enables LoadBalancer services and may resolve network issues.

## 🎯 Conclusion
This guide sets up a complete monitoring stack using Minikube, Prometheus, and Grafana. If issues persist, ensure services are running and accessible using `kubectl get svc -n default`.

Happy monitoring! 🚀

