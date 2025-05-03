# Worker Node Troubleshooting Guide

This document covers comprehensive troubleshooting for Kubernetes worker node issues, including node join failures, kubelet misconfiguration, and runtime compatibility problems.

---

## 1. **Preflight Check Failures During `kubeadm join`**

### Common Errors:

* `error uploading crisocket: Unauthorized`
* `initial timeout of 40s passed`

### Causes:

* Control plane components not reachable (e.g., kube-apiserver down)
* kubelet not started or misconfigured
* Container runtime not configured correctly

### Solutions:

* Check that the control node’s kube-apiserver is running:

  ```bash
  ps -ef | grep kube-apiserver
  ```
* Verify that containerd is installed and running:

  ```bash
  systemctl status containerd
  ```
* Ensure containerd config is set correctly:

  ```bash
  sudo sed -i 's|sandbox_image = .*|sandbox_image = "registry.k8s.io/pause:3.8"|' /etc/containerd/config.toml
  sudo systemctl restart containerd
  ```
* Verify time is synchronized across control and worker nodes.

---

## 2. **`kubelet` Not Starting Properly**

### Indicators:

* No pods showing up with `crictl ps`
* Logs show errors regarding TLS bootstrap

### Solutions:

* Inspect logs:

  ```bash
  journalctl -xeu kubelet
  ```
* Check kubelet environment files:

  ```bash
  cat /var/lib/kubelet/kubeadm-flags.env
  cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
  ```
* Validate that the config file `/var/lib/kubelet/config.yaml` exists and is valid.

If missing, reset the node:

```bash
kubeadm reset -f
rm -rf /etc/kubernetes/* /var/lib/kubelet/* ~/.kube
```

Then attempt to join again.

---

## 3. **Container Runtime Issues (containerd)**

### Errors:

* `crictl ps` fails with runtime errors
* `ctr plugins list | grep cri` shows plugin with `error`

### Solutions:

* Verify containerd is installed:

  ```bash
  which containerd
  ```
* Ensure it is running:

  ```bash
  systemctl restart containerd
  systemctl enable containerd
  ```
* Re-generate containerd config if corrupted:

  ```bash
  containerd config default | sudo tee /etc/containerd/config.toml
  sudo sed -i 's|sandbox_image = .*|sandbox_image = "registry.k8s.io/pause:3.8"|' /etc/containerd/config.toml
  sudo systemctl restart containerd
  ```

---

## 4. **Joining the Node Again**

Ensure:

* Token is valid and not expired (valid for 24h by default):

  ```bash
  kubeadm token create --print-join-command
  ```
* CA cert hash is correct (check using `openssl` on control plane).

Then join:

```bash
kubeadm join <control-plane-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

---

## 5. **Verification After Join**

On the control node:

```bash
kubectl get nodes -o wide
```

Ensure new node appears and Ready status is shown.

Use:

```bash
crictl ps -a
```

To inspect running containers on worker node.

---

## 6. **Other Useful Tips**

* Ensure `iptables` rules and firewall do not block traffic on port 6443.
* Use `ping` and `telnet` to verify connectivity.
* If kubelet still fails, reboot the node.

---

## Final Note

Always match Kubernetes and container runtime versions across nodes to avoid subtle issues.
