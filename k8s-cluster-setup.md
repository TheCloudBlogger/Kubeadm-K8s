# Kubernetes Cluster Setup (Control + 2 Workers with Containerd & Calico)

> Date: May 03, 2025

This guide walks you through setting up a Kubernetes cluster with:

- 1 Control Node
- 2 Worker Nodes
- Containerd as the CRI
- Calico as the CNI
- kubeadm, kubelet, kubectl (v1.29)

---

## 🚀 Control Node Setup

### 1. System Preparation

```bash
sudo apt update && sudo apt upgrade -y
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

---

### 2. Install Containerd

```bash
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
sudo sed -i 's|sandbox_image = .*|sandbox_image = "registry.k8s.io/pause:3.9"|' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

---

### 3. Install Kubernetes Components

```bash
sudo rm -f /etc/apt/keyrings/kubernetes-archive-keyring.gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

---

### 4. Initialize the Control Plane

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

> Save the join command for worker nodes.

---

### 5. Configure kubectl Access

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

### 6. Install Calico CNI

```bash
curl https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```

---

## 🧩 Worker Node Setup (worker1 & worker2)

### 1. System Preparation

```bash
sudo apt update && sudo apt upgrade -y
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

---

### 2. Install Containerd

```bash
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
sudo sed -i 's|sandbox_image = .*|sandbox_image = "registry.k8s.io/pause:3.9"|' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

---

### 3. Install Kubernetes Components

```bash
sudo rm -f /etc/apt/keyrings/kubernetes-archive-keyring.gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm
sudo apt-mark hold kubelet kubeadm
```

---

### 4. Join the Cluster

```bash
sudo kubeadm join <control-plane-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

To generate a new token (on control node):

```bash
kubeadm token create --print-join-command
```

---

## ✅ Verify Setup (On Control Node)

```bash
kubectl get nodes
kubectl get pods -A
```

All nodes should show `Ready` and pods in `Running` state.

---
