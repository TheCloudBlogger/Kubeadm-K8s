# Containerd Troubleshooting for Kubernetes

This document lists common containerd issues encountered during Kubernetes cluster setup and the recommended troubleshooting steps.

---

## 1. Check if containerd socket exists

```bash
ls -l /run/containerd/containerd.sock
```

✅ If the file exists and shows proper permissions like:
```
srw-rw---- 1 root root 0 May  3 08:10 /run/containerd/containerd.sock
```

❌ If it does not exist, containerd is not running or not installed correctly.

---

## 2. Restart and enable containerd

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

---

## 3. Configure CRI runtime endpoint for crictl

```bash
sudo crictl config --set runtime-endpoint=unix:///run/containerd/containerd.sock
```

Check crictl works:

```bash
crictl ps -a
```

If you see:
```
FATA validate service connection: validate CRI v1 runtime API for endpoint ... code = Unimplemented ...
```
Then the CRI plugin is not running properly.

---

## 4. Verify CRI plugin is loaded in containerd

```bash
ctr plugins list | grep cri
```

Expected output:
```
io.containerd.grpc.v1 cri linux/amd64 ok
```

If it says `error`:
- Your containerd config may be invalid or the CRI plugin is disabled.
- Regenerate default config:

```bash
containerd config default > /etc/containerd/config.toml
```

Then edit it to ensure:

```toml
[plugins."io.containerd.grpc.v1.cri"]
  sandbox_image = "registry.k8s.io/pause:3.8"
```

Restart containerd again:

```bash
sudo systemctl restart containerd
```

---

## 5. Common containerd configuration command

Set sandbox image explicitly:

```bash
sudo sed -i 's|sandbox_image = .*|sandbox_image = "registry.k8s.io/pause:3.8"|' /etc/containerd/config.toml
```

---

## 6. Validate containerd service status

```bash
sudo systemctl status containerd
```

Ensure it's running and not erroring out on config.

---

## 7. Check for containerd logs

```bash
journalctl -u containerd -f
```

Useful to debug CRI or config parsing failures.

---

## 8. Validate CRI API with crictl again

```bash
crictl ps -a
```

You should see containers and not a FATA error.

---

## 9. Restart kubelet after fixing containerd

```bash
sudo systemctl restart kubelet
```

Ensure kubelet uses containerd now.

---

## 10. Verify Kubelet uses containerd

Check flags in:
```bash
cat /var/lib/kubelet/kubeadm-flags.env
```

Look for:
```
--container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock
```
