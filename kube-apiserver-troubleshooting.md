# Kubernetes Control Plane Troubleshooting Guide

This guide covers troubleshooting scenarios related to kube-apiserver issues, missing kubeconfig files, kubelet problems, and how to recover when control plane components are down.

---

## 🔍 Problem: `kubectl` Fails with `The connection to the server ... was refused`

**Reason:** The `kube-apiserver` is not running or unreachable.

### Troubleshooting Steps

1. **Check if kube-apiserver is running (as static pod):**

```bash
ps aux | grep kube-apiserver
crictl ps -a | grep kube-apiserver
```
Or inspect the static pod manifest:
```bash
ls -l /etc/kubernetes/manifests/kube-apiserver.yaml
```

2. **Kubelet redeploys static pods from manifests. If file is deleted:**

- Check if the manifest file was accidentally deleted.
- Restore the file from backup or reinitialize using kubeadm.

3. **Restart kubelet to force reload static pod manifests:**

```bash
sudo systemctl restart kubelet
```

4. **Check Kubelet logs for errors loading pods:**

```bash
journalctl -u kubelet -f
```

5. **Validate control plane pod logs using crictl (if apiserver down):**

```bash
crictl ps -a | grep kube-apiserver
crictl logs <CONTAINER_ID>
```

---

## 🛠 Problem: `~/.kube/config` is missing

**Reason:** `kubectl` uses this file to access the cluster. It's typically copied from the control plane after kubeadm init.

### Recovery Steps

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## 🔁 Problem: Want to Reset Entire Cluster

### Reset cluster using kubeadm:

```bash
kubeadm reset --force
sudo rm -rf ~/.kube /etc/kubernetes /var/lib/etcd /var/lib/kubelet /etc/cni /opt/cni
```

Then reinitialize:

```bash
kubeadm init --pod-network-cidr=192.168.0.0/16
```

---

## 🚫 Kube-apiserver is unreachable, and all control components are static pods

### Manual Checks

1. **Inspect static pod manifests in** `/etc/kubernetes/manifests/`:

```bash
ls /etc/kubernetes/manifests
```
Make sure these files exist:
- `kube-apiserver.yaml`
- `kube-controller-manager.yaml`
- `kube-scheduler.yaml`
- `etcd.yaml`

2. **Check the logs of these pods via container runtime:**

```bash
crictl ps -a
crictl logs <container-id>
```

3. **Check kubelet status:**

```bash
sudo systemctl status kubelet
```

4. **Kubelet may fail to load static pods if:**

- Manifests are malformed
- Missing container images
- Container runtime not working

5. **Restart kubelet after fixing any static pod issues:**

```bash
sudo systemctl restart kubelet
```

6. **Verify etcd health (as it's backing apiserver):**

```bash
docker exec -it <etcd-container> etcdctl endpoint health --endpoints=127.0.0.1:2379
```

or via container runtime

```bash
crictl ps -a | grep etcd
crictl exec -it <etcd-container> etcdctl endpoint health --endpoints=127.0.0.1:2379
```

7. **Recreate apiserver manifest if deleted (example template available via kubeadm init logs)**

---

## ✅ Final Validation

After all components are up:

```bash
kubectl get nodes
kubectl get pods -n kube-system
```

Make sure API server is healthy and pods are running.

---
