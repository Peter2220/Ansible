# AWX Installation on Minikube (Rocky Linux 10)

This guide walks through installing AWX on Minikube using Rocky Linux 10 with the Docker driver.

---

## Prerequisites

* Rocky Linux 10
* Root or sudo access
* Internet connectivity

### Recommended Resources

* CPU: 6 cores
* Memory: 8 GB

---

## System Preparation

```bash
sudo dnf clean all
sudo dnf makecache
```

---

## Install Required Tools

### 1. Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

kubectl version --client
```

---

### 2. Install Helm

```bash
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

helm version
```

---

### 3. Install Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

---

## Install Docker (Minikube Driver)

```bash
sudo dnf remove -y podman*

sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo systemctl enable --now docker
```

### Configure User Access

```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

## Start Minikube Cluster

```bash
minikube start --driver=docker --cpus=6 --memory=8192

minikube addons enable ingress
```

---

## Create Namespace

```bash
kubectl create namespace awx

kubectl get namespaces | grep awx
```

---

## Install AWX Operator

### Add Helm Repository

```bash
helm repo add awx-operator https://ansible-community.github.io/awx-operator-helm/

helm repo update

helm search repo awx-operator
```

---

### Install Operator

```bash
helm install awx-operator awx-operator/awx-operator -n awx
```

---

### Verify Operator

```bash
kubectl get pods -n awx -w
```

Wait until:

```
awx-operator-controller-manager-xxxxx   2/2   Running
```

---

## Deploy AWX Instance

```bash
cat <<EOF | kubectl apply -f -
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
  namespace: awx
spec:
  service_type: NodePort
  nodeport_port: 30052
EOF
```

---

### Verify Deployment

```bash
kubectl get awx -n awx

kubectl get pods -n awx -w
```

---

### Expected Pods

```
awx-operator-controller-manager-xxxxx   2/2 Running
awx-postgres-15-0                       1/1 Running
awx-web-xxxxx                           3/3 Running
awx-task-xxxxx                          4/4 Running
```

Note: Initial deployment may take up to an hour depending on resources and network.

---

## Monitor Deployment

```bash
kubectl logs -f deployment/awx-operator-controller-manager -n awx -c awx-manager

kubectl get pods -n awx

kubectl get svc -n awx
```

---

## Expose AWX Service

### Open Firewall Ports

```bash
sudo firewall-cmd --add-port=30000-32767/tcp --permanent
sudo firewall-cmd --reload
```

---

### Verify Port

```bash
ss -tlnp | grep 30052
```

---

### Get Service URL

```bash
minikube service awx-service -n awx --url
```

---

## Get Admin Password

```bash
kubectl get secret awx-admin-password -n awx -o jsonpath='{.data.password}' | base64 --decode
```

---

## Configure Hostname (awx-server)

### 1. Add Entry to `/etc/hosts`

```bash
sudo vi /etc/hosts
```

Example:

```
192.168.49.2   awx-server
```

---

### 2. Access AWX

```
http://awx-server:30052
```

---

## Optional: Create Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: awx-ingress
  namespace: awx
spec:
  rules:
  - host: awx-server
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: awx-service
            port:
              number: 80
```

---

## Cluster Validation

```bash
minikube status

kubectl get nodes
```

---

## Productivity Tip

```bash
echo "alias k='kubectl'" >> ~/.bashrc
source ~/.bashrc
```

---

## Final Access

```bash
echo "AWX URL: http://awx-server:30052"
```

---
