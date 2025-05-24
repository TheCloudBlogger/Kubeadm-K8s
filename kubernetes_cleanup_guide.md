
# 🧹 Kubernetes Cluster Cleanup Guide (kubeadm) – Ubuntu VMs

This guide walks you through completely cleaning up a `kubeadm`-based Kubernetes cluster with **1 control plane** and **2 worker nodes** on Ubuntu.

> ⚠️ **Warning**: This will completely remove all Kubernetes configurations, state, and workloads. Make sure to back up anything important.

---

## ✅ Steps to Reset Kubernetes on All Nodes

---

### 1. Reset Kubernetes Using `kubeadm`

Run this on **each node** (control plane and workers):

```bash
sudo kubeadm reset -f
```

---

### 2. Remove CNI (Container Network Interface) Configuration

```bash
sudo rm -rf /etc/cni/net.d
```

---

### 3. Delete Kubernetes-Related Files and Directories

```bash
sudo rm -rf ~/.kube
sudo rm -rf /etc/kubernetes
sudo rm -rf /var/lib/etcd
sudo rm -rf /var/lib/kubelet/*
sudo rm -rf /etc/systemd/system/kubelet.service.d
```

---

### 4. Restart System Services

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl restart containerd
```

Optionally reboot the node:

```bash
sudo reboot
```

---

### 5. (Optional) Remove Kubernetes Packages

If you want a fresh reinstallation:

```bash
sudo apt-get purge -y kubeadm kubectl kubelet kubernetes-cni kube*
sudo apt-get autoremove -y
```

---

### 6. (Optional) Remove Container Runtime (e.g., `containerd`)

```bash
sudo apt-get purge -y containerd
sudo apt-get autoremove -y
sudo rm -rf /var/lib/containerd
```

---

## ✅ Done!

Your system is now clean and ready for a fresh Kubernetes setup using `kubeadm`.

Would you like a guide for setting it up again from scratch?
